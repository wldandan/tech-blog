# 【Skills】03-Skill-Insight：给 Agent 装上"后视镜"，让Skills开始进化

原文链接：https://zhuanlan.zhihu.com/p/2004914907972401040

---

## 引言：一场静默的革命正在发生

2025年11月，[Anthropic](https://zhida.zhihu.com/search?content_id=270245705&content_type=Article&match_order=1&q=Anthropic&zhida_source=entity) 正式发布了 **[Claude Skills](https://zhida.zhihu.com/search?content_id=270245705&content_type=Article&match_order=1&q=Claude+Skills&zhida_source=entity)** ——一个让开发者可以创建、分享和复用 AI 技能的开放平台。这标志着 Skills 从实验性功能走向规模化应用。

短短两个月后的2026年1月，Vercel 发布了 **[skills.sh](https://zhida.zhihu.com/search?content_id=270245705&content_type=Article&match_order=1&q=skills.sh&zhida_source=entity)** ——一个可以让开发者用一条命令就给 AI Agent 安装新技能的平台。几周时间，[SkillsMP.com" class=" external" target="_blank" rel="nofollow noreferrer">http://SkillsMP.com](https://link.zhihu.com/?target=http%3A//<span data-search-entity=) 上就汇聚了超过 **16万个 Skills** 。

**Skills 正成为 AI 时代的”新 App”** ——它们封装专业知识，连接 LLM 与真实世界，让 Agent 从”会聊天”进化到”会做事”。

这种模式的价值得到了最新研究的证实。2026年1月发表的研究《When Single-Agent with Skills Replace Multi-Agent Systems and When They Fail》显示：**对于小型技能库，单智能体 + Skills 的架构可降低 54% 的 Token 消耗和 50% 的延迟** ——通过消除多智能体协作的通信开销[^1]。

开发者们兴奋地涌入这个新世界：安装 Skill、加载 Skill、看着 Token 消耗、等待结果输出。一切看起来都很美好。

直到有一天，你开始问自己这些最基本的问题：

> “我刚才装的 Skill，Agent 到底用了没？”  
>  “换了 [DeepSeek 模型](https://zhida.zhihu.com/search?content_id=270245705&content_type=Article&match_order=1&q=DeepSeek+%E6%A8%A1%E5%9E%8B&zhida_source=entity)，为什么同一个 Skill 忽好忽坏？”  
>  “我优化了 Skill ，效果变好了没？”

然后你发现——**没人能回答这些问题。**

## 一、繁荣背后的尴尬：Skill 时代的”盲飞”状态

Skills 降低了任务完成的门槛，但没有降低任务优化的门槛。

你可以轻易从平台上下载一个”K8s 故障诊断” Skill，但你无法知道：

  * 它在你的环境中表现如何？
  * 它比另一个同类 Skill 好在哪里？
  * 你修改后的版本是否真的有改进？



**一种”凭感觉优化”的状态。**

一位开发者在 Reddit 上吐槽：”我花了一周优化 Skill 的提示词，最后发现 Agent 根本没在用它——它一直在用原始的 kubectl 命令。”

这不是个别现象。根据 Medium 上近期一项针对 AI Agent 开发者的调查，超过 **70% 的开发者表示无法准确评估 Skill 的实际效果** 。

澳大利亚 CSIRO Data61 的研究人员在 AgentOps 可观测性研究中指出：传统的 AI DevOps 框架已不足以支撑 AI Agent 系统，组织需要系统化的可观测性能力[^1]。研究团队用一句话形象地描述了当前的状态：**“我们在飞行，但没有仪表盘。”**

## 二、三大困境，困住了谁？

### 困境一：只看见过程，看不清价值

你打开 Agent 的执行日志，看到一长串的工具调用记录：

  * `kubectl get pods`
  * `kubectl logs`
  * `helm list`



**这些日志告诉你 Agent 做了什么，但没告诉你它做得怎么样。**

  * 这次任务成功了吗？
  * 耗时多少？是否可接受？
  * Token 消耗是否合理？
  * 和上周相比，是进步了还是退步了？



当前市面上的观测工具大多停留在”记录器”层面，缺乏真正的”分析器”能力。你淹没在数据中，却依然看不到答案。

### 困境二：只能靠”感觉”评估

“我觉得这个版本更好。”

“感觉应该优化了吧？”

**“感觉”成了评估的主要工具。**

这不是开发者的错——而是缺乏统一的评估标准。不同开发者对”好”的定义不同，评估结果无法复用，团队知识无法沉淀。

更致命的是**归因困难** ：即使发现问题，你也无法判断是模型的问题（Skill 写得很清楚，但模型理解错了）还是 Skill 的问题（Skill 本身就没说清楚）。

Towards AI 发表的一篇文章标题直击痛点：《How to Evaluate AI Agent Skills Without Relying on Vibes》——**我们太需要一个不依赖”感觉”的评估方法了。**

### 困境三：下载容易，管好很难

Skills.sh 让获取 Skill 变得像安装 App 一样简单。但**管理 Skills 却像管理一个没有分类的下载文件夹** ：

  * 三个功能相似的 Skill，该用哪个？
  * 优化后的 Skill 版本，如何和旧版本对比？
  * 哪些 Skill 是团队常用的？哪些已经废弃？
  * 不同框架的 Skill 配置方式不同，如何统一维护？



**当 Skills 数量突破 100 个，管理混乱就开始吞噬效率。**

## 三、我们的解法：给 Agent 装上”后视镜”和”仪表盘”

[SkillInsight](https://zhida.zhihu.com/search?content_id=270245705&content_type=Article&match_order=1&q=SkillInsight&zhida_source=entity) 的使命很简单：**让每一次 Skill 执行，都能被看见、被衡量、被优化。**

### 能力一：多维度对比，让差异无处藏身

SkillInsight 最直观的价值是——**让你看见差异** 。

  * 同一个任务，GPT-4 和 DeepSeek 谁更快？
  * 同一个 Skill，在 OpenCode 和 ClaudeCode 上表现一致吗？
  * 优化前后的两个版本，准确率提升了多少？



这些对比不再是”凭感觉”，而是基于3+1**个黄金指标** 的量化数据：

  * **效果（准确率）** ：问题真正解决了吗？
  * **效率（时延）** ：响应速度可接受吗？
  * **成本（Token）** ：算力消耗合理吗？
  * **安全（Security）** ：行为可信任吗？



### 能力二：智能裁判，不再依赖”感觉”

SkillInsight 内置了智能评判系统：

  * **自动评分** ：对比标准答案，对每一次执行打分
  * **Skill 召回验证** ：检查 Agent 是否真的用了你预期的 Skill
  * **失败归因** ：智能区分是”模型没做好”还是”Skill 没写好”



**最关键的是这个归因能力** ——它会告诉你：

> “这次失败是因为 Skill 没有说明需要先备份配置文件，建议添加 backup 步骤”

而不是让你去翻阅几百行日志自己猜。

### 能力三：统一管理，告别混乱

SkillInsight 提供统一的 Skill 管理中心：

  * **版本控制** ：V1、V2… 多版本并存，一键回滚
  * **跨框架统一** ：一套配置，适配 OpenCode、ClaudeCode、OpenHands
  * **效果对比** ：直观对比不同版本的表现，用数据做决策



**你不再需要记住”那个在 ClaudeCode 上好用的版本是哪个”——系统会告诉你。**

### 能力四：透明接入，零侵入部署

最让开发者头疼的是”接入成本”——改造代码、配置环境、调试兼容性…

SkillInsight 采用**透明代理技术** ：任何支持 [OpenAI SDK](https://zhida.zhihu.com/search?content_id=270245705&content_type=Article&match_order=1&q=OpenAI+SDK&zhida_source=entity) 的 Agent，只需配置一个 Base URL，即可无感接入。

没有代码侵入，没有复杂配置，像装一个”网络监听器”一样简单。

## 五、从”盲飞”到”数据驱动”

Google Cloud 的《AI Agent Trends 2026》报告显示，88% 的早期采用者正在看到积极的 ROI。但报告也指出：**“最大的障碍不是技术，而是无法证明价值。”**

SkillInsight 解决的就是这个”证明价值”的问题。

  * 对开发者：它让每一次优化都有数据支撑
  * 对团队：它让评估标准统一，知识可以沉淀
  * 对组织：它让 Agent 的投入产出变得可衡量



**我们相信，当 Agent 装上”后视镜”和”仪表盘”，Skills 的真正进化才刚刚开始。**

## 六、当前进展

Skill Insight 0.1版本功能已完成开发，欢迎体验：

模块| 功能| 状态  
---|---|---  
数据采集| 透明代理、会话记录、指标追踪| ✅ 已实现  
Skill 管理| Skill 仓库、版本控制、启用/禁用| ✅ 已实现  
评估引擎| 答案准确性、Skill 召回、失败归因| ✅ 已实现  
可视化| Dashboard、详情下钻、配置管理| ✅ 已实现  
  
Roadmap 中，Skill 质量评分、相似 Skill 聚类、文档驱动生成等能力正在开发中。

> 技术全景图：[Skill全生命周期管理-全景图](https://zhuanlan.zhihu.com/p/2025509340916794370)  
> 上一篇：[【Skills】02-集成Skills的单智能体，能否终结多智能体？](https://zhuanlan.zhihu.com/p/1999195993162396617)  
> 下一篇：[【Skills】04-Skill-insight：Agent skills从”感觉”到”量化”的进化之路](https://zhuanlan.zhihu.com/p/2014044608666019231)

* * *

**Sources:** [^1]: [arXiv - A Taxonomy of AgentOps for Enabling Observability of Foundation Model based Agents](https://link.zhihu.com/?target=https%3A//arxiv.org/html/2411.05285v1) \- CSIRO Data61, November 2024 [^2]: [arXiv - When Single-Agent with Skills Replace Multi-Agent Systems and When They Fail](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2601.04748) \- January 2026

  * [Towards AI - How to Evaluate AI Agent Skills Without Relying on Vibes](https://link.zhihu.com/?target=https%3A//pub.towardsai.net/how-to-evaluate-ai-agent-skills-without-relying-on-vibes-9a5764ad18c4)
  * [Medium - Demystifying AI Agent Evaluation](https://link.zhihu.com/?target=https%3A//medium.com/ai-simplified-in-plain-english/demystifying-ai-agent-evaluation-a-system-level-methodology-for-production-reliability-db2c3022a97a)
  * [IBM - What is AgentOps?](https://link.zhihu.com/?target=https%3A//www.ibm.com/cn-zh/think/topics/agentops)
  * [Vercel - Introducing Skills](https://link.zhihu.com/?target=https%3A//vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem)
  * [SkillsMP](https://link.zhihu.com/?target=https%3A//skillsmp.com/zh)
  * [Reddit - Skills Marketplace Discussion](https://link.zhihu.com/?target=https%3A//www.reddit.com/r/ClaudeCode/comments/1qa02h0/skills_marketplace_a_new_digital_economy/)


