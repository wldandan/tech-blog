# Skill-MAS：不训参数，只进化一份"编排说明书"

> 推理时搜索太贵，训练时微调绑死了小模型。Skill-MAS 找了第三条路：把多智能体编排经验写成一份持续进化的文档。

## 自动 MAS 卡在哪

LLM 多智能体系统（MAS）在复杂任务上效果好，但手工给每个任务设计 Agent 角色、通信拓扑和工作流，既累又难规模化。于是有了自动 MAS：让一个 Meta-agent 自动生成多智能体架构。

现有方案两派，各有各的问题。

推理时编排用冻结的前沿 LLM 做 Meta-agent，配合搜索算法迭代优化 MAS。AFlow 用 MCTS 探索工作流空间，AOrchestra 基于层次分解动态创建子 Agent，MAS-Zero 用自反射反馈优化拓扑。这些方法能利用最强模型的推理能力，但每处理一个新任务都从零开始搜。上一次失败的诊断经验带不到下一次。搜索是经验盲的。

训练时编排把编排能力参数化，在小模型（通常 7B）上微调。MAS2 训练模型自生成和自我修正工作流，MAS-Orchestra 把 MAS 构建建模成函数调用任务用 GRPO 优化。能通过梯度更新内化经验，但受限于小模型的能力天花板。大模型的微调成本高到不现实。

两派卡在一个死结上：**用强模型就学不会，能学会的模型不够强。** 经验保留和模型能力能不能拆开？

香港科大（广州）和蚂蚁集团的联合团队在 Skill-MAS 里给出了一个答案：把 Meta-agent 的编排能力写成一份 Meta-Skill，一份结构化的自然语言文档，在迭代中持续进化这份文档，不动模型参数。

## Meta-Skill 长什么样

三层结构。

Task Decomposition 规定了 Meta-agent 怎么拆用户查询：识别宏观目标和范围，拆成逻辑自洽的子任务序列，定义每个子任务的成功标准。Agent Engineering 规定了怎么为每个子任务设计子 Agent：分配角色、起草系统指令、指定输入上下文。Workflow Orchestration 规定了怎么选架构拓扑（顺序管线、路由型、层次型、黑板型），定义 Agent 间的 I/O 映射和数据流，输出可执行 MAS。

初始版本从 Anthropic 的 MAS 构建指南总结而来，通用、领域无关。关键是它是自然语言写的，任何 LLM 都能读。换底层 Meta-agent 模型不用丢编排经验。

## 怎么进化：多轨迹采样 + 选择性反思

两阶段闭环。

第一阶段，Multi-Trajectory Rollout。对验证集里每项任务，用当前 Meta-Skill 独立采 K 条轨迹（默认 5）。同任务跑多次不是为了刷通过率，是为了算两个分布量：uncertainty（不确定性），同任务多轨迹的分数标准差，反映指令在多大程度上模糊或不稳定；difficulty（困难度），负平均分数，反映系统性难度。

第二阶段，Selective Reflection。不是所有 N 项任务都值得平等反思。把 uncertainty 和 difficulty 归一化后合成优先级分数，降序排列，用二阶差分自动检测肘点，只选最易变、最困难的顶部子集做深度分析。避免诊断信号被大量简单任务稀释。

两层分析。层一，任务内对比。对每项任务，把 K 条轨迹按分数中位数分成高分组和低分组。LLM 反射器对比执行快照，识别分歧点（两组从哪步开始走不同路），提取成功因素，归类失败模式。输出结构化诊断和候选修复方案。层二，跨任务综合。把各任务诊断报告放一起，找跨任务的系统性弱点和通用成功模式。任务特定的观察被提升为领域无关的编排原则。

然后用证据包驱动 Skill Optimizer 把 Meta-Skill 从 S^(r) 更新到 S^(r+1)。修改必须基于反思证据，必须抽象为可泛化的原则而非任务特定补丁，严格保持三层模块结构。通过结构有效性检查才接受。所有轮次后选验证集最优的 S* 上测试集。

## 数据

四个基准、四个 LLM 做 Meta-agent。DeepResearchBench（深度研究报告）、HLE-Math（专家级数学推理）、BrowseComp-Plus（多跳动态问答）、VitaBench（真实世界交互场景，多工具）。

DeepSeek-V4-Flash 上，Skill-MAS-optimized 平均 41.05%，Skill-MAS-init 只有 33.72%，加了 7.33 个百分点。四个 LLM 上都是最优或接近最优，成本适中。

Training-time 方法（MAS2、MAS-Orchestra）成本最低但性能最差，小模型一次性生成 MAS 对复杂多变任务缺乏泛化。Inference-time 方法（AFlow、EvoAgent）性能更好但推理成本最高，每次查询都重新搜索。Skill-MAS 用一次性生成替代了逐样本搜索，用 Meta-Skill 保留了跨任务经验，卡在性能高、成本适中的位置。

## Meta-Skill 可以迁移

有意思的实验。把 DeepSeek-V4-Flash 上进化出来的 Meta-Skill 直接给 GPT-5.4-Nano 用，反之亦然。同任务跨 LLM 迁移，性能接近原生进化。跨任务同 LLM 迁移也有竞争力，说明进化后的 Meta-Skill 学会的是领域无关的编排策略，不是数据集特化技巧。跨任务跨 LLM 最弱但仍有增益。

消融方面。rollout 数从 3 到 5 到 7 持续提升，但边际递减。去掉优先级任务选择（全量验证或随机一半），性能下降但仍超多数 baseline，核心进化机制是稳健的。

## 进化后长什么样

论文附录里优化前后 Meta-Skill 的完整对比非常好看。初始版是简洁的通用框架，每条规则一两句话。进化后精确到令人发指。

DeepResearchBench 优化后，Task Decomposition 从"分析意图拆子任务"变成了强制钻石型拓扑（context-scoping root → parallel analytical branches → dedicated synthesis terminal node），硬约束最大两层深度。Agent Engineering 加了输出长度合约、两级验证（Strict Pass ≥60% token 预算加全领域术语覆盖，Soft Pass 40-60%）、带阈值递减的重试协议。Workflow Orchestration 加了全局/Local 双内存层、合成前覆盖率检查、优雅降级。

BrowseComp-Plus 的进化方向不同：加权约束满足协议、迭代搜索、交叉实体桥接、合并节点主动重新检索权限。HLE-Math 加了强制解释登记册（MIR）和小 n 校准、多路径验证。VitaBench 加了探索-评估-执行三阶段分解、决策代理结构化合约、"现实检查"门禁。

Meta-Skill 的进化不是堆更多规则，是根据各领域的失败模式长出适配策略。DeepResearchBench 的框架太松散导致输出质量不稳，进化方向是加结构约束。BrowseComp-Plus 的单一搜索路径证据不足，进化方向是加平行检索和交叉验证。HLE-Math 的术语歧义崩推理链，进化方向是强制解释登记和实证校准。VitaBench 的跨能力域单 Agent 过载，进化方向是按能力边界拆 Agent。

## 局限

有 ground-truth 标签时 Skill-MAS 效果好，弱监督或无监督场景会退化。论文提了可以集成 LLM-as-a-judge 做自监督评估。多任务联合学习消融喜忧参半，VitaBench 微涨，BrowseComp-Plus 下降，缺自适应域隔离机制。

## 我觉得最巧的地方

Skill-MAS 最巧妙的事不是进化算法本身，是它把"学习"定义成了"修改一份文档"而不是"修改模型权重"。经验保留和参数更新被彻底拆开了。

拆开之后三个好处。Meta-agent 可以随时换，今天 DeepSeek 明天 Qwen 后天 GPT，Meta-Skill 不变，编排经验不丢。进化完全可解释，diff 两份 Meta-Skill 就能精确看到每条原则怎么演化的。推理没额外开销，Meta-Skill 就是一段 system prompt 前缀，生成 MAS 一次性的，不迭代搜索。

搭多 Agent 系统的话，一个值得试的方向：把你手工调的编排经验写成结构化文档，在验证集上多跑几次观察方差，让强模型从失败轨迹里提取模式，把经验写回文档。迭代几轮，这份文档可能比你的模型参数值钱。

---

*论文：Lin et al., "Skill-MAS: Evolving Meta-Skill for Automatic Multi-Agent Systems," arXiv:2606.18837v1, 2026.*
