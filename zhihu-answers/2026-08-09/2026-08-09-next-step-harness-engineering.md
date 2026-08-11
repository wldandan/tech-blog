# 标题占位

> 平台：知乎回答 | 状态：**草稿（待你网页发布）**
> 发布地址：https://www.zhihu.com/question/2020813873368884488

---

# 从长上下文工程到 Harness 工程，Agent 工程下个发力点可能在哪里？

> 平台：知乎回答 | 状态：**草稿（待 humanizer + 网页发布）**
> qid：2020813873368884488
> 题目：从长上下文工程到 Harness 工程,你认为 Agent 工程下个发力点可能是哪里？
> 差异化角度：把"上下文工程"和"Harness 工程"放进同一个工程钟，押注下一步不是哪个新框架，而是"骨架感知的评测"

---

## 一、先看清楚我们正卡在哪

知乎上现在关于 Agent 工程的讨论，关键词基本集中在三件事上：

- 长上下文工程：怎么在 200K 甚至 1M 上下文里塞进最有用的信息，避免"上下文污染"和"中间遗忘"。
- Harness Engineering：在模型外面套工程层（脚手架/编排/状态机/容错），让 Agent 能跑长任务。
- Multi-Agent 协作：多 Agent 分工、并行、消息通信、冲突解决。

这三件事每个都有大量文章、视频、教材。但有个尴尬的事实是：Agent 在生产里依然普遍拉胯。[Gartner 预测到 2027 年底超过 40% 的 Agentic AI 项目会被取消](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027)；CMU 的 [TheAgentCompany](https://arxiv.org/abs/2412.14161) 基准里，最强模型（Gemini 2.5 Pro）也只能自主完成 30.3% 的真实任务；[MIT NANDA 的调研](https://mlq.ai/media/quarterly_decks/v0.1_State_of_AI_in_Business_2025_Report.pdf)显示约 95% 的生成式 AI 试点没有可衡量回报。

为什么不是这三件事做不到位，而是我们评价 Agent 的方式过时了——这才是问题。

我们今天用的评测方法，大多是"这个任务 Agent 跑通了/没跑通"——一个 0/1 分。但 Agent 跑通不代表 Agent 跑对，更不代表 Agent 的 Harness 健康。模型换了、智能体框架改了、上下文变了，传统的"任务成功率"打不出问题出在哪一层。

所以我认为下一阶段真正的发力点，是把评测能力从"单点打分"挪到"骨架感知"和"轨迹级"。

## 二、发力点 1：骨架感知的评测（Scaffold-Aware Evaluation）

先说一个概念：最近一批 agent 评测研究（比如 [OctoBench](https://arxiv.org/html/2601.10343)）已经明确提出"骨架感知的评测"（scaffold-aware evaluation）——主张把围绕模型的整个控制栈纳入被评测的系统，而不是只测模型本身。

为什么这件事卡死了我们两年？因为 Agent 的失败，绝大多数不在模型上，而在骨架上。

典型例子（[NeurIPS 2025 的 MAST 多智能体失败分类法](https://proceedings.neurips.cc/paper_files/paper/2025/hash/b1041e52d3be19f0a9bc491657488e4a-Abstract-Datasets_and_Benchmarks_Track.html)，分析了 1600+ 条真实执行轨迹）：

- 系统设计失败：角色定义不清、任务边界重叠、上下文传递链路断。
- 协作错位失败：Agent 之间协议不一致、消息丢失、循环死锁。
- 验证缺失失败：Agent 跑完任务没有自动校验机制，错了也不知道。

这三类失败，传统评测一个都看不出来——你只看到一个"任务失败"的 0 分。

骨架感知的评测要干的事，是把每次 Agent 运行变成结构化轨迹，然后在轨迹上做归因：

- 哪一步决策错了？
- 哪一次工具调用错了？
- 哪个 Skill 失能了？
- 是上下文不够、还是上下文传递丢了？
- 是状态机卡死、还是终止条件没触发？

我自己在 [agent-insight](https://atomgit.com/openeuler/agent-insight) 里做的也是这件事。每次 Agent 跑完一条任务，不只记录"成功/失败"，而是把完整的轨迹、上下文快照、决策链都存下来，跑 MAST 那一套归因，把失败归到"规格 / 协作 / 验证"三类里。

这条路一开始很重，但只要落地，团队就能从"我们不知道 Agent 为什么挂着"变成"我能精准定位是哪一层在挂"。这一步，是大多数团队还没迈过去的那道坎。

---

## 三、发力点 2：上下文工程从"管 prompt"挪到"管骨架"

第二个发力点，是把"上下文工程"这个老话题重新审视一遍。

过去两年业界讲上下文工程，重点在怎么写好 prompt、怎么压缩 token、怎么选示例。但随着 Harness 框架（LangGraph、Claude Agent SDK、OpenAI Swarm、JiuwenSwarm）的普及，"上下文"这个词的边界已经变了。

它不再是"塞进模型窗口的那段文字"。它至少包含 5 层：

1. 系统提示（你的指令和约束）
2. 工具集定义（Agent 能调什么）
3. MCP / 协议层（跨 Agent 通信规范）
4. 外部数据（RAG 拉来的内容、技能库）
5. 会话历史（多轮记忆、子 Agent 上下文）

每一层都有自己的"上下文污染"和"上下文丢失"问题。传统的 prompt 工程只能管第 1 层，剩下 4 层全靠工程结构去管。

这就是为什么这两年我们开始看到一个新的工程方向——把"上下文"当作一个一等公民来设计：类似 Claude Skills 的渐进式披露（按需加载 Skill）、Anthropic 的上下文压缩（接近窗口上限时压缩摘要）、AET 这类系统里把每个阶段的产物（设计文档、开发计划、验证报告）固化成 Markdown 当下一阶段输入。

具体到工程发力点，三件事最重要：

- 结构化笔记：让 Agent 把"我现在知道什么、准备做什么、卡在哪"用结构化方式写下来，而不是塞进自由文本历史。
- 上下文压缩时优先保召回：宁可压缩后多留一点冗余，也不要为了精简丢掉关键细节——这一点我专门在 [上下文工程系列文章](https://zhuanlan.zhihu.com/p/1992637122281228256) 里写过。
- 子 Agent 上下文隔离：长任务里派一个子 Agent 出去跑局部，最好让它带着"自己的"上下文，回来时只回传结论，不污染主对话。

这些不是新概念，但把它们工程化、模板化、放进 Agent 框架作为默认值，是接下来一年最值得投入的方向。

---

## 四、发力点 3：Harness 本身要被优化

第三个发力点，单独拿出来讲，因为它容易被忽略。

**我们现在已经默认"模型要评测、要优化"（各种 Bench、各种 RLHF），但 Harness 本身基本没有被优化过。** 换个模型，跑一遍——这是现在大部分团队的做法。

但 Harness 是一个工程产物，它有结构、有耦合、有技术债。Skill 写了 50 个，有 20 个已经失效了；Check Point 设了 5 个位置，但其中 3 个根本不会被触发；终止条件写了 3 套，但只有 1 套真在用。没人在评测它。

下一阶段的发力点，是让 Harness 自己也进入"评测—优化"这条流水线。我做的 agent-insight 干的就是这件事：把 Skill 当一等公民，跑"生成→评测→优化→再评测"的回路。

这条流水线里关键的两件事：

第一，**评测不能只看"跑没跑通"**。在 SkillBench 里我做了四种评测模式（static / dynamic / hybrid / feedback），拿真实任务去跑，采集 token、成本、耗时、成功率，把优化前后的版本放一起打分。"代码跑得通"和"答得对"根本是两码事——生产里最坑的事故往往是脚本顺顺当当跑完，结果是错的，还没人发现。

第二，优化不能改完就算数，得自带把关。优化 Agent 改完 Skill 后自动过三道门：结构门（引用的脚本都在、能编译）、脚本真值门（算出来的数拿标准答案核对）、行为门（拿几道题真跑一遍，和旧版本比分）。任何一道发现变差就打回让它自己修（repair），三道全过了才轮到我拍板确认成新版本。

DSPy 生态的 [GEPA](https://arxiv.org/abs/2507.19457) 这类反思式优化已经能从执行轨迹中诊断"为什么失败"，再自动进化出更优 prompt，多个基准上以最多 35 倍更少的 rollout 次数超越强化学习（GRPO）；研究也开始把 Harness 本身当作搜索空间，让智能体自动改写自己的骨架。

---

## 五、所以，下一个发力点到底是什么？

回到问题本身：长上下文工程到 Harness 工程，下一个发力点在哪里？

**我的判断是三件事并行，而不是任何新概念：**

1. 把评测这件事挪到"骨架感知 + 轨迹级"上来做——这是整个行业跨过 Demo、进入生产之间最关键的一步。
2. 上下文工程这件事挪到"管 5 层上下文的工程结构"上做——长任务、长链路、多 Agent 场景里，这是工程能力的分水岭。
3. 把 Harness 本身纳入"评测—优化"这套流水线——Harness 会腐化，需要持续盯着它、评它、改它。

这三件事的共同点，是它们都不是新的炫技概念，而是工程纵深。AI Agent 行业再走两年就会发现："能跑通"和"能跑稳"之间隔着的那段距离，不是另一个框架，而是这些工程纵深。

> 长上下文工程、Harness 工程、Multi-Agent 都不是终态。它们是这一代 Agent 工程师必须掌握的三件基础工具，而这之上那一层——把工程本身当成可评测、可优化的对象——才是下一个发力点。

---

**文中项目都是 openEuler 社区开源：**

- witty-diagnosis-agent（智能诊断，假设-验证 + 六 Agent 协同）：https://atomgit.com/openeuler/witty-diagnosis-agent
- agent-insight（Agent/Skill 全生命周期观测·评测·优化）：https://atomgit.com/openeuler/agent-insight
- AET / agentic-engineering-team（全流程 AI 辅助研发底座）：https://atomgit.com/openeuler/agentic-engineering-team

**延伸阅读：**

- [Skill Insight → Agent Insight：一次以 Harness 为中心的演进](https://atomgit.com/openeuler/agent-insight)
- [为 AI Agent 高效构建上下文](https://zhuanlan.zhihu.com/p/1992637122281228256)
- [Harness Engineering 的本质是什么？](https://www.zhihu.com/question/2016648624256340425/answer/2037953527507629953)

_我是 Leon，openEuler SIG AI Agent Maintainer，关注 AI Agent 工程化与 Harness Engineering，欢迎评论区交流。_
