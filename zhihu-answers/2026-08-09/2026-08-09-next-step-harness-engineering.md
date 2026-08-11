# 标题占位

> 平台：知乎回答 | 状态：**草稿（待你网页发布）**
> 发布地址：https://www.zhihu.com/question/2020813873368884488

---

# 从长上下文工程到 Harness 工程，Agent 工程下个发力点可能在哪里？

> 平台：知乎回答 | 状态：**草稿（待 humanizer + 网页发布）**
> qid：2020813873368884488
> 题目：从长上下文工程到 Harness 工程,你认为 Agent 工程下个发力点可能是哪里？
> 差异化角度：开篇直接回答"发力点在哪"——评测（骨架感知评测），再用翁荔《Harness Engineering for Self-Improvement》的自我改进链条（prompt→context→harness→optimizer）作证据：链条走多深，取决于评测跟不跟得上，它是 Meta-Harness 敢用于生产的前提，不是可有可无的收尾工作。

---

先说结论：我认为下一个发力点是评测——把评测能力从"任务跑没跑通"的单点打分，挪到骨架感知（scaffold-aware evaluation）和轨迹级归因。

不管走的是上下文工程这条路，还是 Harness 工程这条路，甚至再往前一步搞 Harness 自我改进，卡住继续往前走的都是同一件事：你怎么知道新版本真的比旧版本好。

这个判断是踩坑换来的。我这两年基本就在干一件事——把 AI Agent 往生产环境里推：openEuler 上的故障诊断（witty-diagnosis-agent，社区官方项目）、给 Agent 能力做全生命周期管理的平台（agent-insight）、一支 Agent 团队跑全流程研发（AET）。这三个场景的共同点是：错误代价是真实的，回路里没有人类随时纠偏。

这道题这几天格外热闹，因为翁荔（前 OpenAI 安全副总裁、现 Thinking Machines Lab 联创）刚发了篇新博客《[Harness Engineering for Self-Improvement](https://lilianweng.github.io/posts/2026-07-04-harness/)》，DeepSeek 研究员崔添翼也转发附议。

她梳理了近期约 35 篇论文后给出一条递进链条：自我改进的重心正在从 prompt 下沉，经过 structured context、workflow、harness code，最后落到 optimizer code。

最前沿的 [Meta-Harness](https://arxiv.org/abs/2603.28052)（Lee et al. 2026），让一个 coding agent（论文里用的是 Claude Code + Opus 4.6，中位数每轮读 82 个文件）去读之前所有候选 harness 的源码、分数和执行轨迹，自己提案、自己改写下一版 harness，在 Pareto 前沿上留最好的几个——跑出来的效果比 ACE、MCE 这些手工设计的基线高 7.7–8.6 分，还能省 4 倍 context。

这条链条我认同方向，而且它恰好印证了开头那句判断：链条每往下沉一层，"变得更好"这件事就更难判断一次——这正是评测要补上的缺口。

下面拆开讲。

---

## 一、链条往下沉一层，验证就难一级

prompt 变好，人读一眼就知道；换个说法顺不顺、清不清楚，直觉就能判断。

到了 harness code 这一层，判断就没那么直觉了：Meta-Harness 改的是"决定存什么、取什么、怎么呈现给模型"的代码本身，一次改动可能同时影响十几个下游任务的行为，靠人读 diff 已经看不出好坏。

到了 optimizer code 这一层——也就是"改 harness 这件事本身"的代码——判断难度又上一个台阶：你不是在评价一次执行结果，是在评价一套改进策略长期跑下去会不会把系统带偏。

Meta-Harness 论文用的是 Pareto 前沿加固定 benchmark 打分，这套办法在论文的封闭评测集里管用，但换到生产环境，没有现成 benchmark、没有 ground truth，"新版本是不是真的更好"这个问题谁来回答？

这正是我这两年在 agent-insight 里天天要处理的问题，也是我认为下一阶段真正的发力点：把评测能力从"单点打分"挪到"骨架感知"和"轨迹级"。

这件事应该是整条自我改进链条能往前走的前提，而不是收尾工作。

## 二、发力点 1：骨架感知的评测，是 Meta-Harness 能落地的闸门

先说一个概念：最近一批 agent 评测研究（比如 [OctoBench](https://arxiv.org/html/2601.10343)）已经明确提出"骨架感知的评测"（scaffold-aware evaluation）——主张把围绕模型的整个控制栈纳入被评测的系统，而不是只测模型本身。

为什么这件事卡死了我们两年？因为 Agent 的失败，绝大多数不在模型上，而在骨架上。

典型例子（[NeurIPS 2025 的 MAST 多智能体失败分类法](https://proceedings.neurips.cc/paper_files/paper/2025/hash/b1041e52d3be19f0a9bc491657488e4a-Abstract-Datasets_and_Benchmarks_Track.html)，分析了 1600+ 条真实执行轨迹）：

- 系统设计失败：角色定义不清、任务边界重叠、上下文传递链路断。
- 协作错位失败：Agent 之间协议不一致、消息丢失、循环死锁。
- 验证缺失失败：Agent 跑完任务没有自动校验机制，错了也不知道。

这三类失败，传统评测一个都看不出来——你只看到一个"任务失败"的 0 分。放进 Meta-Harness 的语境里看更扎眼：如果一个自动生成 harness 的系统本身没有"验证缺失失败"这道检测，它优化出来的新 harness，也可能只是在同一类盲区里换了个位置摔倒，Pareto 分数完全看不出来。

骨架感知的评测要干的事，是把每次 Agent 运行变成结构化轨迹，然后在轨迹上做归因：

- 哪一步决策错了？
- 哪一次工具调用错了？
- 哪个 Skill 失能了？
- 是上下文不够、还是上下文传递丢了？
- 是状态机卡死、还是终止条件没触发？

我自己在 [agent-insight](https://atomgit.com/openeuler/agent-insight) 里做的也是这件事。每次 Agent 跑完一条任务，不只记录"成功/失败"，而是把完整的轨迹、上下文快照、决策链都存下来，跑 MAST 那一套归因，把失败归到"规格 / 协作 / 验证"三类里。

这条路一开始很重，但只要落地，团队就能从"我们不知道 Agent 为什么挂着"变成"我能精准定位是哪一层在挂"。这一步，是 Meta-Harness 这类自动优化系统敢不敢放进生产的前提——没有它，只能眼看着 Pareto 分数涨，却不知道涨的是真本事还是 benchmark 打法。

---

## 三、发力点 2：上下文工程从"管 prompt"挪到"管骨架"

第二个发力点，呼应翁荔链条里 structured context 这一环。

过去两年业界讲上下文工程，重点在怎么写好 prompt、怎么压缩 token、怎么选示例。但随着 Harness 框架（LangGraph、Claude Agent SDK、OpenAI Swarm、JiuwenSwarm）的普及，"上下文"这个词的边界已经变了。

它不再是"塞进模型窗口的那段文字"。它至少包含 5 层：

1. 系统提示（你的指令和约束）
2. 工具集定义（Agent 能调什么）
3. MCP / 协议层（跨 Agent 通信规范）
4. 外部数据（RAG 拉来的内容、技能库）
5. 会话历史（多轮记忆、子 Agent 上下文）

每一层都有自己的"上下文污染"和"上下文丢失"问题。传统的 prompt 工程只能管第 1 层，剩下 4 层全靠工程结构去管。

这也是为什么翁荔的链条里 structured context 会被单独列成一层，而不是并入 prompt：ACE（Agentic Context Engineering）把上下文当成一个不断演进的 playbook，而不是越来越长的 prompt；MCE（Meta Context Engineering）在此基础上用双层优化去搜索更好的上下文管理方式本身。这条路径，和我在 AET 里把每个阶段产物（设计文档、开发计划、验证报告）固化成 Markdown 当下一阶段输入，是同一个方向的不同实现。

具体到工程发力点，三件事最重要：

- 结构化笔记：让 Agent 把"我现在知道什么、准备做什么、卡在哪"用结构化方式写下来，而不是塞进自由文本历史。
- 上下文压缩时优先保召回：宁可压缩后多留一点冗余，也不要为了精简丢掉关键细节——这一点我专门在 [上下文工程系列文章](https://zhuanlan.zhihu.com/p/1992637122281228256) 里写过。
- 子 Agent 上下文隔离：长任务里派一个子 Agent 出去跑局部，最好让它带着"自己的"上下文，回来时只回传结论，不污染主对话。

这些不是新概念，但把它们工程化、模板化、放进 Agent 框架作为默认值，是接下来一年最值得投入的方向。

---

## 四、发力点 3：Harness 本身要被优化——但要装刹车

第三个发力点，正面回应翁荔链条最前沿的那一段：harness code 和 optimizer code 要不要自动改。

我的立场是：方向对，但 Meta-Harness 现在这套"proposer 自由改、Pareto 前沿选"的模式，直接搬进生产环境会出事。论文里的 fitness 是固定 benchmark 上的 reward，现实里没有这么干净的目标函数——Skill 写了 50 个，有 20 个已经失效了；Check Point 设了 5 个位置，但其中 3 个根本不会被触发；终止条件写了 3 套，但只有 1 套真在用。这些技术债不会体现在 Pareto 分数上，但会在某次线上事故里体现出来。

下一阶段的发力点，不是照搬 Meta-Harness，而是让"harness 自己被优化"这件事也进入一条有把关的流水线。我做的 agent-insight 干的就是这件事：把 Skill 当一等公民，跑"生成→评测→优化→再评测"的回路，本质上是给 Meta-Harness 的自由提案装上刹车。

这条流水线里关键的两件事：

第一，评测不能只看"跑没跑通"。在 SkillBench 里我做了四种评测模式（static / dynamic / hybrid / feedback），拿真实任务去跑，采集 token、成本、耗时、成功率，把优化前后的版本放一起打分。"代码跑得通"和"答得对"根本是两码事——生产里最坑的事故往往是脚本顺顺当当跑完，结果是错的，还没人发现。

第二，优化不能改完就算数，得自带把关。优化 Agent 改完 Skill 后自动过三道门：结构门（引用的脚本都在、能编译）、脚本真值门（算出来的数拿标准答案核对）、行为门（拿几道题真跑一遍，和旧版本比分）。任何一道发现变差就打回让它自己修（repair），三道全过了才轮到我拍板确认成新版本。

DSPy 生态的 [GEPA](https://arxiv.org/abs/2507.19457) 这类反思式优化已经能从执行轨迹中诊断"为什么失败"，再自动进化出更优 prompt，多个基准上以最多 35 倍更少的 rollout 次数超越强化学习（GRPO）。这类反思式优化配上骨架感知的评测把关，才是 Meta-Harness 敢往生产里落的样子——不是"proposer 自由改、分数说了算"，而是"proposer 提案、评测拦截、人在关键节点拍板"。

---

## 五、所以，下一个发力点到底是什么？

回到问题本身：长上下文工程到 Harness 工程，下一个发力点在哪里？

翁荔那条 prompt → context → harness → optimizer 的链条，我认同是正确的演进方向。但我的判断是，这条链条能走多远，取决于每一层"变得更好"能不能被验证——而这件事，行业现在还没跟上。

三件事并行：

1. 把评测这件事挪到"骨架感知 + 轨迹级"上来做——这是 Meta-Harness 这类自动优化系统能不能安全落地生产的前提，也是整个行业跨过 Demo、进入生产之间最关键的一步。
2. 上下文工程这件事挪到"管 5 层上下文的工程结构"上做——长任务、长链路、多 Agent 场景里，这是工程能力的分水岭。
3. 把 Harness 本身纳入"评测—优化"这套流水线，而不是直接照搬"proposer 自由改"的模式——Harness 会腐化，也可能被自动优化带偏，两件事都需要持续盯着、评它、改它。

> 长上下文工程、Harness 工程都不是终态，向 optimizer code 自我改进也不是终点。翁荔指的方向没错，但下一个发力点，是给这条自我改进链条装上能验证"每一步真的变好了"的刹车——没有它，链条走得越深，摔得越不知道为什么。

---

**文中项目都是 openEuler 社区开源：**

- witty-diagnosis-agent（智能诊断，假设-验证 + 六 Agent 协同）：https://atomgit.com/openeuler/witty-diagnosis-agent
- agent-insight（Agent/Skill 全生命周期观测·评测·优化）：https://atomgit.com/openeuler/agent-insight
- AET / agentic-engineering-team（全流程 AI 辅助研发底座）：https://atomgit.com/openeuler/agentic-engineering-team

**延伸阅读：**

- [Skill Insight → Agent Insight：一次以 Harness 为中心的演进](https://atomgit.com/openeuler/agent-insight)
- [为 AI Agent 高效构建上下文](https://zhuanlan.zhihu.com/p/1992637122281228256)
- [Harness Engineering 的本质是什么？](https://www.zhihu.com/question/2016648624256340425/answer/2037953527507629953)
- [翁荔：Harness Engineering for Self-Improvement](https://lilianweng.github.io/posts/2026-07-04-harness/)
- [Meta-Harness: End-to-End Optimization of Model Harnesses（Lee et al., 2026）](https://arxiv.org/abs/2603.28052)

_我是 Leon，openEuler SIG AI Agent Maintainer，关注 AI Agent 工程化与 Harness Engineering，欢迎评论区交流。_
