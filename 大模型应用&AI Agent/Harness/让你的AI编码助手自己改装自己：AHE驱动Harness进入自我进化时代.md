# 让你的AI编码助手自己"改装"自己：AHE驱动Harness进入自我进化时代

> 当所有人都在卷模型的时候，一群研究者把目光转向了模型外面的那层"脚手架"——而且他们让这层脚手架学会了自己进化。

# 一、Harness Engineering：2026年AI编程圈最火的概念

如果你关注AI编程领域，2026年肯定躲不开一个词：**Harness Engineering（驾驭工程）**。

事情的引爆点有好几个。先是OpenAI在2月发了一篇《Extreme Harness Engineering》实践报告，3个工程师在5个月内用Codex CLI指挥AI写出了**100 万行代码，人类一行都没写**。紧接着Anthropic的Claude Code因为一次.map文件事故导致51万行源码泄露，外界得以一窥顶级Coding Agent的harness架构设计，社区称之为"AI界的核泄漏事件"。Martin Fowler也亲自下场写了Harness Engineering的方法论博客。再加上OpenHands刚发布Agent Control Plane，Cursor推出Agent SDK，整个行业在从"卷模型"转向"卷工程"。

**但这里有一个关键问题：这些harness都是人类手工设计的。而手工设计面临一个结构性困境——基座模型迭代太快了。**

光2026年上半年，我们就看到了GPT-5.4、Claude Opus 4.6/4.7、DeepSeek-V4、Qwen 3.6、Kimi K2.6等一大波新模型。每个新模型的"最佳harness"都不一样：为GPT-5.4调好的harness换到DeepSeek-V4上可能反而降性能。手工重调的速度根本追不上模型发布的速度，这个Gap正在越拉越大。

**那能不能让Agent自己进化自己的harness？**

这就是复旦大学和北京大学联合上海奇绩智峰在最新论文中解决的问题。他们提出了**AHE（Agentic Harness Engineering）**，一个基于可观测性驱动的闭环系统，让编码 Agent 的 harness 自动进化。**10 轮迭代，Terminal-Bench2 的 pass@1 从 69.7% 提升到 77.0%，超过了人类手工设计的 Codex-CLI（71.9%）。**

<center>

![alt text](image.png)

图1 AHE在Terminal-Bench2上将纯Bash种子演进至超越所有人类设计与自演进基线的水平</center>

我读下来的感受是，这篇文章值得看，不是因为它刷了一个高分，而是它把一个工程直觉——"好的可观测性是自动优化的前提"——系统化地做成了可复现的闭环。比那些在prompt里加几句咒语就号称self-evolving的工作扎实太多了。

# 二、问题拆解：为什么不能让Agent自己改配置？

让Agent自己优化自己的harness，听起来很自然。真做起来有三个硬骨头：

**第一，动作空间太碎（Heterogeneous Action Space）**。一个典型的Coding Agent harness包含系统提示词、工具定义与实现、中间件、技能库、子Agent配置、长期记忆等七八个组件，格式千差万别——有的是自然语言，有的是YAML/JSON配置，有的是Python代码。进化Agent得同时理解和编辑所有这些，动作空间极其碎片化。

**第二，轨迹信息过载（Voluminous Trajectories）**。每次任务执行都产生数百万token的原始轨迹：工具调用、shell输出、错误堆栈、中间结果……进化Agent不可能通读这些原始日志，真正有价值的信号被埋在了海量噪声里。

**第三，编辑效果难以归因（Hard Attribution）**。你改了一个middleware，改了一个 tool，又改了system prompt，然后pass@1涨了 3 个点——到底是谁的功劳？下次又跌了，是谁的锅？没有归因，进化就会退化成瞎试。

这三个障碍的根因指向同一个东西：**observability（可观测性）不够**。AHE的核心洞察是，这个问题的瓶颈不在于进化Agent本身有多聪明，而在于你有没有给它提供**结构化的、可追溯的、可证伪的**观测信息。

这让我想起Sutton那篇《The Bitter Lesson》：利用算力和结构化搜索，往往比精巧的手工设计走得更远。AHE就是这个哲学在harness工程领域的实践。

# 三、AHE的三根支柱：把每次编辑变成"可证伪的合约"

AHE用三个可观测性支柱来逐个击破上面的障碍。

<center>

![alt text](image-1.png)

图2 AHE流水线</center>

## 3.1 支柱一：组件可观测性（Component Observability）

AHE基于NexAU 框架，将harness的7个组件类型（系统提示词、工具描述、工具实现、中间件、技能、子Agent配置、长期记忆）**全部解耦为独立文件**，挂在固定路径下。

这个设计看起来简单，但工程收益很大：

- **每个失败模式能精准映射到单个组件类**，而不是散落在几百行prompt散文里。
- **每次编辑是一次git commit**，天然获得file-level diff和rollback粒度。
- **进化Agent的动作空间变得干净**：该改工具就改工具文件，该加中间件就加中间件文件，不用在一个巨大prompt文本里做外科手术。

## 3.2 支柱二：经验可观测性（Experience Observability）

AHE用Agent Debugger框架处理每次rollout的海量轨迹数据：

1. 把每条轨迹消息存为独立文件，构建一个**可导航的文件环境**。
2. 用一个分析Agent遍历这些文件，提取**失败根因**和**成功模式**，生成per-task分析报告。
3. 把所有per-task报告聚合为一个**benchmark级概览**，作为每轮进化的入口文档。
4. 保留原始轨迹（去重和base64清理后），供进化Agent必要时钻取验证。

这里的核心技巧是**渐进式信息披露（Progressive Disclosure）**：进化Agent先看概览，发现可疑再下钻到具体任务报告，再下钻到原始轨迹。既控制了token消耗，又不丢信息。

## 3.3 支柱三：决策可观测性（Decision Observability）

这是AHE最让我兴奋的设计。每次进化Agent编辑harness 时，**必须提交一个变更清单（change manifest）**，写清楚：

- 引用的失败证据
- 推断的根因
- 针对性的修复方案
- **预测的修复效果**：哪些任务会变好（predicted fixes）
- **预测的回退风险**：哪些任务可能变差（predicted regressions）

下一轮评估后，系统把预测跟实测的task-level delta做交集，生成**per-edit裁决**。预测命中的编辑保留，预测失败的编辑**在文件粒度上回滚**。

**这就是AHE跟其他self-evolve方法最本质的区别**。别人的进化是"我觉得这样改应该更好"，AHE的进化是"我预测这样改会让任务 A、B、C变好但可能让任务X变差，下一轮我们看实测结果"。每次编辑是一份**可证伪的合约**，不是自我合理化的散文。

这种设计本质上是在把一个工程优化问题安上科学实验的骨架：假设、预测、验证、回滚。没有人类监督，方向也能保持正确。

# 四、实验结果：不是刷一个榜，是刷完了还能迁移

AHE在Terminal-Bench2（89个任务，easy/medium/hard三级）上跑了一轮10轮进化，GPT-5.4 High作为所有三个角色Agent（代码、调试、进化）的共享基座。

### 主结果：全面碾压人类手工和自动基线

<center>

![alt text](image-2.png)

</center>

几个细节值得注意：

- ACE在全部任务上**不升反降**（68.9% vs NexAU₀ 69.7%）。ACE的思路是在prompt里注入自然语言playbook，这些playbook是从terminal-bench轨迹里提炼的策略。但它只能编辑prompt，碰不到工具、中间件和记忆 —— 而后面这几个正好是AHE证明增益最大的部分。
- TF-GRPO比NexAU₀强（72.3%），但本质是强化工具调用序列偏好，同样编辑不了脚手架本身。
- AHE在Hard上略输Codex-CLI（53.3% vs 56.7%），论文分析是因为多个AHE组件在长周期任务上产生了冗余验证交互，吃掉了时间预算。

### 跨Benchmark迁移：SWE-bench-verified

你可能想：在Terminal-Bench2上优化，当然在那上面强，这不就是过拟合吗？

![alt text](image-3.png)

恰恰这是AHE最有说服力的结果。团队把**冻结的AHE harness（不再进化）**直接搬到 SWE-bench-verified（500个真实GitHub issue）：

- **总成功率75.6%，四组最高**（NexAU₀ 75.2%, ACE 74.6%, TF-GRPO 74.2%）。
- **总token消耗461K，四组最低**，比NexAU₀少12%，比ACE少32%。

ACE和TF-GRPO在跨benchmark场景下都**劣于NexAU₀**，因为它们注入prompt的terminal-bench策略变成了噪声，徒增token消耗又不改变底层策略。AHE的增益编码在工具、中间件和长期记忆里，不受prompt面迁移的影响。

### 跨模型迁移：+5.1 到 +10.1 pp

更意外的是跨模型家族的迁移。把AHE harness（在GPT-5.4 High上进化）直接给其他模型用：

![alt text](image-4.png)

**跨家族增益远大于同家族增益。** 论文给的解释很有意思：离饱和越远的基座模型，越依赖harness里编码好的协调模式；越强的模型自己就能从prompt推导出这些策略，边际收益就低。换个说法：**好的harness对不那么强的模型帮助更大** —— 这对实际工程选型是个重要信号。

# 五、增益解剖：到底谁在起作用？

论文做了一个很干净的组件消融实验：每行只从 AHE 完整配置中**换入一个组件**到种子 harness，其余三个保持种子默认值。

![alt text](image-5.png)

三个发现：

1. **工具、中间件、长期记忆各自独立贡献正值**（+3.3, +2.2, +5.6 pp），而**系统提示词单独换入反而降了 2.3 pp**。事实性的harness结构（工具怎么调、中间件怎么检查、记住了什么教训）能跨任务迁移，自然语言级别的策略指导则依赖上下文配合，单独拿出来是负收益。
2. **组件之间存在非加和性交互**。三个正向组件的单独增益加起来是 +11.1 pp，完整AHE只有 +7.3 pp。尤其在Hard上，仅长期记忆就超过了完整AHE —— 记忆、中间件和系统提示词都在往"验证式闭环"方向推，叠加出冗余的重复检查，吃掉了长周期任务的时间预算。
3. **ACE和TF-GRPO从未编辑的组件，恰好是增益所在。** 两个prompt-only基线为什么落后一目了然：真正有效的改进发生在它们够不到的层面。

# 六、进化Agent的自我认知：会归因修复，但看不到回归

AHE的change manifest机制让研究者可以直接评估进化Agent的"自我认知"水平。他们拿每轮的预测（"我会修复 A、B、C"）跟下一轮的实测Ground Truth对比，分fix和regression两个维度算精确率和召回率：

- **Fix预测**：precision 33.7%, recall 51.4%，约为随机基线（6.5%/10.6%）的**5倍**。进化Agent确实在基于证据做针对性修复，不是在随机尝试。
- **Regression预测**：precision 11.8%, recall 11.1%，仅为随机基线（5.6%/5.4%）的 **2倍**。进化Agent**几乎没有预测回归的能力**。它知道改完之后为什么某些任务会变好，但无法预见到同一个修改会让哪些任务变差。

**这就是AHE进化曲线出现非单调跳跃的根本原因，也是论文指出的最清晰的未来方向：regression foresight（回退预见）。**

拿软件工程打个比方：AHE现在就像一个只有单元测试没有回归测试的CI管道——它能确认修好了该修的，但没法在改之前就知道可能改坏了什么。要解决这个问题，可能得让进化Agent维护一张所有历史编辑效果的"影响图"，或者用小规模试探性rollout来做编辑的预验证。

# 七、几点判断

读完这篇论文，有几个想法——不一定对，但值得聊：

**第一，Harness正在变成AI编程的"编译器优化"层。** 把基座模型当CPU，harness就是编译器。同样的任务，不同harness配置，性能天差地别。AHE做的事，相当于让编译器自己调自己的 -O2/-O3 参数。这方向如果继续走下去，harness可能会长成类似LLVM那样的中间层基础设施。

**第二，"编辑prompt"可能是链条上最弱的一环。** AHE的组件消融给出了一个很直接的信号：真正可迁移的增益长在结构化的工程组件上（工具、中间件、记忆），而不是自然语言提示词。之前不少self-evolve工作只在prompt措辞上做文章，跟AHE的结果一比，差距明显。将来做Agent自进化的团队，精力应该放到可编程、可组合的结构化组件上。

**第三，可观测性是自动优化的前提——这个洞见不止于harness。** AHE三根支柱（解耦的组件表示、分层的经验蒸馏、可证伪的决策记录）本质上是一套通用方法。你在做RAG pipeline自动调优、multi-agent拓扑自动搜索、evaluation pipeline自动校准——这三个设计模式都值得认真看看。

**第四，别忽视局限性。** AHE目前只在两个benchmark上验证过，进化操作点（step budget、timeout）是针对GPT-5.4 High调的，跨模型存在耦合。更关键的是，它的自我修改治理还比较基础：有workspace隔离、manifest记录和file-level rollback，但缺长周期的harness清理和更强的误用防护。论文自己说得很直白，AHE是"受控的研究原型，而非完全成熟的自主自改进系统"。

# 八、总结

**AHE让我最兴奋的地方，不是77%那个数字，而是它证明了harness进化这件事是可行的。**

一个Coding Agent的harness能从自己的失败里学东西，把经验固化成工具、中间件和长期记忆，而且每次修改都经得起下一轮验证 —— 到这个程度，人类工程师就不用熬夜看轨迹、手写prompt修bug 了。省下来的时间可以做更有价值的事：定义进化的方向，而不是执行进化的细节。

# 相关链接

1. 论文代码 [GitHub](https://github.com/china-qijizhifeng/agentic-harness-engineering)
2. 论文原文 [arXiv](https://arxiv.org/abs/2604.25850)
