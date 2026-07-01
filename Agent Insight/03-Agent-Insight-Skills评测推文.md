# 别让质量黑洞吞掉你的Agent：Agent Insight让Skill随模型同步进化

:::tip 背景
你的团队写了一个Skill，功能是把用户上传的CSV自动做清洗、分列、输出标准化报表。上线两个月，用户好评如潮。

然后底层模型升级了。

没有人改过这个Skill的任何一行代码，但生产日志里开始出现诡异的迹象，某些CSV格式不再被正确路由到这个Skill，触发率从 92% 静默滑落到 67%；偶尔触发成功的请求，输出报表的列顺序也和之前不一致。**没有报错，没有告警，只有用户开始抱怨"好像没以前好用了"**。

问题出在哪？不是Skill变了，而是模型对Skill的"理解面"变了。团队没有任何机制能探测到这种变化。
:::

> 上面这个场景并非假设。Skill 不是写完就稳定的代码，而是一个需要被持续度量、监控、有时还要被退役的工件。没有可信的评分，"该不该上线""该不该灰度""什么时候必须回滚"只能靠经验和勇气回答。这正是 Agent Insight 要解决的问题。

![alt text](overall.png)

2025年10月Anthropic推出Agent Skills；半年内业界看到了三个独立信号：skill-creator在2026年3月升级，把**trigger eval + quality eval直接编进Skill创作流程**，Anthropic自己跑了一轮描述优化，6个公共Skill里5个触发率提升；Snyk在2月发布ToxicSkills研究，**扫描 3,984 个公共Skill，36.82% 至少含一个安全问题，13.4%含Critical级漏洞**；学术界则在 4 月给出SICA，一个能自己修改自己源码、在SWE-Bench上把自身性能从 17% 推到 53% 的self-improving coding agent。

三个信号指向同一件事：**Skill不是写完就稳定的代码，而是一个需要被持续度量、监控、有时还要被退役的工件**。底层模型每次升级都会改变Skill的"理解面"，模型在进化，没有评测兜底的Skill在原地退化。Agent Insight正是为此而设计，它提供一套以 4 维加权 + 置信加权 + Smart Run自动复测为核心的Skill评测能力。本文将详细介绍这套评测体系的设计思路与Agent Insight中的具体实现。

:::tip Agent Insight
Agent Insight的目标是成为一个框架无关的统一AgentOps平台，用「观测 → 评测 → 诊断 → 优化」的能力闭环，让生产级Agent行为可观测、故障可定位、质量可度量、能力可进化，赋能Agent越用越好。

本文聚焦Agent Insight在Skill评测方面提供的四种核心能力：**静态合规、触发分析、用例分析、A/B测试**。

![Agent Insight Skills分析界面](images/评测分析.png)
:::

# 一、Skill开发者路径：评测是上线生产的硬基线

Anthropic在2025年10月推出Agent Skills后，Skill正在成为继Prompt、Tool之后第三种Agent能力单位。一个Skill的核心交付物是SKILL.md加上配套资源（`scripts/`、`references/`等），模型先读front matter决定是否加载，再执行任务。

但Skill写完不等于结束。它有一个完整的生命周期，Agent Insight的评测能力贯穿其中：

![Skill生命周期](images/fig01-skill生命周期.png)

*Skill生命周期：开发一次，评测⇄优化反复发生，Agent Insight的评分穿透到生产监控*

这条路径里，开发只发生一次（或少数几次重写），评测和优化是反复发生的螺旋，评分穿透整条路径。Skill第一次写出来时几乎一定"看起来合理但实际不工作"，所以Anthropic的官方建议很直白："Create evaluations BEFORE writing extensive documentation"。在这条路径上，Agent Insight的评测同时承担四个角色：

1. 质量守门人：决定能不能进入灰度、能不能推全
2. 优化指南针：静态合规、触发分析、用例分析、A/B测试四个维度的分数各有对应的优化方向，告诉团队下一步该改哪里
3. 回归探测器：Smart Run让静默退化变成自动告警
4. 退役判官：A/B显著为负或长期低于阈值时，给出"该退役"的客观信号

要让评测真正扮演这四个角色，本文按两条线展开：先把Agent Insight的评测系统拆开看（总览 → 综合健康分 → 四个维度的细节），再把它装回开发者路径里看（评测 ⇄ 优化的螺旋 → 长期价值与演进定位）。两条线最终汇到同一个结论：评测不只是打分工具，而是Skill工程化的基础设施。

# 二、Agent Insight评测方案总览：4个维度 + 1个综合健康分

前面给出了评测要扮演的 4 个角色：质量守门、优化指南、回归探测、退役判官。要让一套打分系统同时承担这 4 个角色，Agent Insight在设计上必须满足两个约束：**指标足够细，能定位到每一类失败模式应该修改Skill的哪一部分；输出足够聚合，能直接和上线阈值比较，给出二元决策**。这就要求一组分项指标 + 一个综合分。本节先讲分项怎么拆、再讲聚合规则和阈值。

## 2.1 为什么是这4个维度

"打几个分、各自衡量什么"。Agent Insight把它拆成4维，对应Skill走向生产时必须连续通过的 4 道关卡：

![Agent Insight的4个评测维度](images/fig02-agent-insight的4个评测维度.png)

*Agent Insight 4个评测维度的前序关系：前一关不过，后面全部失效；离用户价值越近，权重越大*

少任何一道都会留下盲区：

| 缺失维度 | 会发生什么 |
| --- | --- |
| **缺静态合规** | Skill结构问题导致永远无法被正确加载，但其他维度可能误报"工作正常" |
| **缺触发分析** | Skill看起来工作得很好，但生产中只在 30% 该用到的场景被触发 |
| **缺用例分析** | 触发率100%，但每次触发都把事情做错 |
| **缺A/B测试** | Skill分数都高，但其实不用它表现一样好甚至更好 |

这套拆分在三个独立来源上得到交叉印证。Anthropic官方实践：skill-creator 2.0的trigger eval + quality eval对应Agent Insight中间两维，Agent Insight前后各补一维覆盖官方留下的盲区。**软件工程测试金字塔**：静态合规 ≈ lint、触发 ≈ 单测、用例 ≈ 集测、A/B ≈ 对照实验，每层解决前一层解决不了的问题。**学术界自我进化框架**：ASG-SI把验证器奖励分解为tool-use correctness、outcome validity、skill reuse、composition integrity四类，与Agent Insight的 4 维高度同构。

## 2.2 方案结构：4个维度 → 1 个综合健康分

**权重按"影响Skill真实价值的因果距离"递增**：静态合规离用户价值最远（10%）、A/B 离用户价值最近（40%）。这同时也对齐了反馈成本。权重低的维度反馈快、修复便宜，权重高的维度反馈慢、修复昂贵。

4 个维度对应Skill生命周期的 4 个本质问题，也对应**开发者路径上 4 类不同的优化动作 + 4 道不同的质量基线**：

| 维度 | 它在回答的问题 | 分数低时优化什么 | 对应的质量基线 |
| --- | --- | --- | --- |
| **静态合规** | Skill写得规不规范？能不能被正确路由？ | 改Skill的描述、结构、上下文预算 | 最底层基线，不达标连灰度都进不去 |
| **触发分析** | 该触发的时候触发了吗？ | 改front matter关键词、调描述触发性 | 灰度阶段持续监控的关键指标 |
| **用例分析** | 触发之后，做得怎么样？ | 改主体内容、加示例、调指令措辞 | 推全前最重要的质量证据 |
| **A/B测试** | 这版本/这Skill真的带来增益了吗？ | 决定是否推全、是否退役 | 推全/回滚/退役决策的唯一刚性依据 |

![4 个维度汇聚为综合健康分](images/fig03-综合健康分.png)

*4个评测维度经「置信加权 × 覆盖度折扣」汇聚为 1 个综合健康分，既能逐维定位问题，又输出一个可直接决策的标量*

# 三、综合健康分：从 4 个数字到 1 个可决策的分数

4 维分数各自指向不同的优化动作，但**上线决策不能由 4 个数字同时投票** —— 团队需要一个能直接和阈值比较的标量，否则"该不该灰度"会再次回到"凭感觉"。所以Agent Insight要把 4 个分数聚合成一个综合健康分，但聚合方式不能是简单加权，下面就要解决这件事。

4 个维度的分数 `S₁..S₄` 都已归一化到 [0, 100]。综合健康分先算原始加权分，再做置信加权 + 覆盖度折扣：

1. ①原始加权分：$S_{raw}=\sum \omega_{i} \times S_{i}$，$\omega = [0.10, 0.20, 0.30, 0.40]$
2. ②置信加权（每个维度的样本量决定权重缩放）：$S_{conf} = Σ(wᵢ · cᵢ · Sᵢ) / Σ(wᵢ · cᵢ)$ ，$cᵢ ∈ [0,1] = min(nᵢ / n*ᵢ, 1)$
3. ③覆盖度折扣（缺失维度不能"假装"自己是 100 分）：$S_{health} = S_{conf} × (有数据维度的权重和)$

第①步是直觉做法，但有两个致命缺陷，第②③步分别堵这两个缺陷。②**让有数据维度之间公平对比**（样本量不足的维度不能"假装"自己很可信），③**惩罚维度缺失**（少一个维度，综合分按比例打折）。两步针对的是不同问题，不要混为一谈。

## 3.1 为什么必须有后两项

如果只用$S_{raw}$，**只跑了 1 个维度的Skill也能拿到高分**。一个只跑过静态合规、其他维度没数据的Skill，可能假装到 85 分上线，等于让一个完全没经过质量验证的Skill进入生产。置信加权 + 覆盖度折扣强制这种Skill拿到一个低分，**用低分把它挡在生产之外，逼团队补齐评测维度才能放行**。

这是Agent Insight把"评测覆盖度"内化为综合分的一部分。一个Skill想拿高分，不仅要在跑过的维度上表现好，**还必须把 4 个维度都跑过。覆盖度折扣是综合分给"半成品上线"上的锁**。

![综合健康分三档阈值带](images/fig04-阈值带.png)

*4维权重分布与三档阈值带：分数不只是数字，是直接的上线决策*

# 四、Agent Insight四个评测维度详解

综合分回答了"够不够上线"，但要把分数推上去，必须知道 4 维各自怎么算、低分时改什么、在生产链路里对应哪一道质量基线。下面按权重从小到大逐个展开，并展示Agent Insight中每个维度的实际界面。

## 4.1 静态合规（w = 10%）

Agent Insight的静态合规对Skill的全部组成（SKILL.md及其配套资源如 `scripts/`、`references/` 等）做离线检查，基于**六项分析标准**进行全面的代码质量检查：目的适配性、结构规范性、指令适配性、内容一致性、运维可靠性、脚本及参考文档质量。同时承担**安全扫描**职责，检测Skill是否含有恶意内容。两者共同决定一个Skill能否进入后续评测和上线流程。

规范这一层的重要性来自Anthropic的路由机制：模型先只读front matter决定是否加载，**front matter写不好 = Skill永远不会被触发**。Anthropic 自己跑过一次描述优化，6 个公共Skill里有 5 个触发率提升。安全这一层则被严重低估。Snyk在 2026 年 2 月发布的ToxicSkills研究中，扫描结果是**36.82% 的Skill至少含一个安全问题，13.4% 含Critical级漏洞**。Skill继承Agent的完整权限，其内容本身就是模型读取并执行的指令源，任何被注入的内容都会被当作可信指令执行。

:::info
安全扫描覆盖Snyk ToxicSkills研究中识别出的几类典型攻击模式：

- **Prompt injection**：描述里嵌入条件触发的隐藏指令
- **恶意脚本**：脚本读环境变量再外发，或执行未声明的shell命令
- **数据外泄通道**：诱导Agent把敏感数据写进gitcommit、邮件草稿等次级输出
- **越权范围**：声明的scope与`allowed-tools`不匹配
:::

生产基线方面，规范分低于 60 禁止进入灰度；任意Critical级安全告警直接熔断，不允许进入任何环境。这是布尔判定，不参与加权，类比CI流水线里CVE阻断构建。**低分时的优化动作**是改Skill的描述、结构、上下文预算；安全告警则必须先修复源问题，不能用调权重绕过。

![Agent Insight静态合规分析界面](images/静态合规.png)

*Agent Insight 静态合规分析界面*

## 4.2 触发分析（w = 20%）

Agent Insight的触发分析能力**基于Skill代码自动生成正反用例**，验证用例执行时Skill的触发行为是否与预期一致。它在一组标注好的触发集上replay当前Skill，看"该触发的时候触发了吗、不该触发的时候抑制住了吗"，最终输出一个 $F_β$ 分数。

$F_β · 偏向召回的调和平均（β > 1）$

$F_β = (1 + β²) · precision · recall / (β² · precision + recall)$

召回是Skill唯一的入口。触发不到，再好的执行也是 0。漏触发尤其危险，它是彻底沉默的失败，用户根本意识不到有这个Skill应该生效却没生效，所以这里用 `β > 1` 偏向召回，宁可误触发也不要漏触发。Anthropic在skill-creator 2.0里也把trigger eval单独拆成一个独立的eval-set。

:::info
Agent Insight在此基础上做了进一步自动化：**无需人工手写触发集**，系统根据Skill代码自动生成正例（应该触发）和反例（不应触发），同时也支持人工补充来自生产trace的样本。每次Skill修改后自动replay计算P/R。**低分时的优化动作**是改front matter关键词、调整描述的触发性，配套产出是一份"哪些query没被触发"的反例清单，它就是下一轮description优化的直接输入。**生产基线**把触发率放在灰度阶段做持续监控。如果灰度真实流量的触发率显著低于离线触发集，说明触发集分布有偏差，需要补样本而不是急着推全。
:::

![Agent Insight触发分析界面](images/触发分析.png)

*Agent Insight 触发分析界面*

## 4.3 用例分析（w = 30%）

如果说触发分析回答"找得到吗"，用例分析回答的是"找到之后用得怎么样"。Agent Insight的用例分析**基于评测数据集 + 评估器**，对Skill的实际执行过程做双维度评估：**结果评估**用LLM-as-Judge + Rubric给最终输出打分，**轨迹评估**则对tool call、参数、中间状态打分。

为什么必须做双维度？因为只看输出会漏掉大量过程性失败。Claw-Eval在 14 个前沿模型上做的对照实验显示，**纯输出评估漏掉了 44% 的安全违规和 13% 的鲁棒性失败**，这些都需要轨迹审计才能发现。一个最终回复看起来合理的Skill，可能其实用错了tool、漏了关键参数，再用模型推理"圆"回来。这种"输出对但过程错"的隐性失败只有轨迹分析能抓到。

LLM-as-Judge在这里靠谱的几个前提：给judge留"Unknown"出口、每个维度用独立judge、鼓励reasoning、关键样本做human calibration。**低分时的优化动作**是根据高偏离trace的标签定位问题。比如"tool call参数总是漏字段X"，对应改Skill里把这个字段从should改为MUST。**生产基线**：灰度阶段可接受平均分 70+，推全要求 85+，并额外加**一条高偏离trace占比不超过 5%**。一个平均分 90 但 10% 场景灾难性失败的Skill，不能推全。

![Agent Insight用例分析界面](images/用例分析.png)

*Agent Insight用例分析界面*

## 4.4 A/B测试（w = 40%）

A/B测试是唯一能直接回答"这个Skill到底有没有用"的维度。Agent Insight的A/B测试能力**基于评测数据集 + 评估器**，支持两种对比模式：**有Skill vs 无Skill**（验证Skill的增益价值）和**同一Skill不同版本**（验证迭代方向是否正确）。通过盲测和显著性检验给出结论，显著性检验采用Bradley-Terry 评级 + bootstrap 重采样算 95% CI，与Artificial Analysis的方法论一致。

A/B权重最高，因为前三个维度都在回答"做得好不好"，只有A/B回答"该不该做"。这个区分对Skill尤其关键。Anthropic在skill-creator升级时把Skill分成两类：
- **Capability Uplift**（让模型做它本来做不到的事）有过期日，模型变强后可能反而拖后腿；
- **Encoded Preference（编码团队偏好）**则随模型变强越有价值。

要区分一个Skill属于哪类、还该不该留着，只有A/B能给答案。当无Skill的baseline已经能通过eval时，意味着模型已经把这个Skill的能力内化了，Skill该退役了。

具体到生产基线，A/B是决策级的，对应三条规则：推全要求显著为正（p < 0.05）且 95% CI下界 > 0；已推全的Skill在Smart Run中转为显著为负立即回滚；连续 3 次`with_skill vs without_skill`无显著差异则触发退役评审。

![Agent Insight A/B测试界面](images/AB测试.png)

*Agent Insight A/B 测试界面*

:::info
# 五、评测 ⇄ 优化的螺旋如何运转

到这里Agent Insight的"评测系统长什么样"讲完了。但工具静态摆在那里没用，它必须嵌进开发者每天的工作节奏里。下一个问题是：怎么让"评测 → 优化 → 再评测"形成稳定的螺旋上升，而不是来回震荡？

Agent Insight的答案是把**版本管理**做成螺旋的物理载体。每一轮"评测 → 优化"完成后，Skill自动生成新版本，把Skill内容、运行链路（trace）、4 维评分一起冻结归档。Skill不再是一个会被反复覆盖的文件集，而是一条带评分历史的版本链。带来的直接收益有三个：每一步改动的效果都被量化记录（v1 的描述优化让综合分从 30 涨到 55，主要贡献在触发维度从 20 涨到 65，这种"分数归因"是后续优化的方向盘）；一键回滚到任何历史版本；任意两个历史版本天然可以做 A/B。**评测沉淀 trace、trace 驱动优化、优化产生新版本、新版本再被评测**，这就是螺旋上升的实际工作方式。

![评测 ⇄ 优化的螺旋](images/fig05-评测优化螺旋.png)

*评测 ⇄ 优化的螺旋：每转一圈（评测 → 沉淀 trace → 优化 → 新版本）沉淀一个版本，综合分逐版上升、依次跨过准入 / 灰度 / 推全三档基线*

**一键评测**是螺旋的加速器。Agent Insight将 4 个维度的评测并行执行，聚合成综合健康分报告。没有自动化时手动跑一次要 30–60 分钟，一键评测压到 3–5 分钟。效率提升的真正价值不只是省时间，更重要的是它**改变了迭代节奏**：手动评测下你会本能地把多个改动攒在一起跑，结果是改动之间相互混淆、分数变化无法归因；一键评测让"小步快跑"成为可能，改一个关键词跑一次、改一段示例跑一次，每次的分数差就是这次改动的净效应。

配合分层节奏（静态合规 + 触发分析作内循环小时级迭代，用例 + A/B 作外循环天级迭代，Smart Run 作持续监控），版本管理 + 一键评测 + 三档质量基线（60 准入 / 70 灰度 / 85 推全）让"何时停止迭代、何时上线"从"PM 拍板"变成"分数到位"。

螺旋同样延伸到生产期，由**Smart Run**自动复测驱动：综合分跌破 70 自动降级到灰度，跌破 60 回滚到上一基线版本，A/B 转为显著负立即回滚并触发修复评估，连续 3 次`with_skill vs without_skill`无显著差异则进入退役评审。同一套分数同时决定 Skill 能不能上线、上线后什么时候必须下线。
:::

# 六、Agent Insight评测框架在Skill演进路径上的定位

评测能力已经讲完，但还有一个更本质的问题没有回答：随着Skill生态成熟、Agent自治程度提高，这套打分系统会过时吗？答案取决于它在业界Skill评测演进路径上的位置。只有当它能同时服务于"现在团队怎么做评测"和"未来团队会怎么做评测"，长期投入才成立。先看演进路径本身，再看 Agent Insight 在路径上的角色。

## 6.1 三阶段演进：EDD与Self-Improving

业界Skill评测的实践正在沿一条路径演进：「凭感觉」→「评测驱动开发（EDD）」→「Skill自我进化（Self-Improving）」。

:::info
这条路径的核心逻辑是把人从"读分数 → 改Skill"的循环里逐步移出去：Skill数量少时人扛得住，到几十上百个Skill时人就是瓶颈。
1. **EDD**把评测分数确立为优化方向、停止条件和上线资格的依据，与Anthropic官方skill authoring推荐的"Create evaluations BEFORE writing extensive documentation"同源；
2. **Self-Improving**更进一步：让Agent基于失败案例自动回写Skill。EvolveR把交互轨迹蒸馏为可复用策略原则、AutoSkill把临时交互沉淀为显式Skill、SICA让Agent自我修改源码，ASG-SI则要求所有自动patch必须经验证器验证并留下可审计evidence。

![三阶段演进](images/fig06-三阶段演进.png)

*三阶段演进：Agent Insight的评测在EDD中是"标准答案"（告诉人该改什么），在Self-Improving中是"验收官"（决定Agent的自动修改能不能过），人逐步退场*

Agent Insight的 4 维评测 + 三档质量基线在两个阶段承担相同的角色。
- **EDD中充当"标准答案"**，人根据评分判断Skill哪里不行、该怎么改；
- **Self-Improving中充当"验收官"**，Agent自动提交修改后，由同一套评测决定这次修改能不能过。
:::

两者复用同一套打分规则：Self-Improving提patch → 4 维评测验证 → 综合分上升且 A/B 显著正则auto-commit进灰度，否则rollback。三个阶段之间是依赖关系而非替代关系：EDD依赖 4 维评测给出优化方向，Self-Improving依赖EDD沉淀的评测体系 + 基线作安全阀。这也呼应了Agent Insight两大核心能力的设计初衷：**Agent观测与自进化**提供数据飞轮和辅助决策，**Skill开发与自进化**提供全生命周期能力闭环，两者共同支撑从EDD到Self-Improving的演进。

## 6.2 与同类方案的对照

| 方案 | 定位 | 与 Agent Insight 的关系 |
| --- | --- | --- |
| Anthropic skill-creator (2026-03) | trigger eval + quality eval | Agent Insight前后各补一维（静态合规、A/B），并加置信加权 |
| Braintrust EDD | 通用LLM eval平台 | 更通用、生态更完整；Agent Insight面向Skill，4 维与生命周期紧耦合 |
| ASG-SI | 学术界自我进化 + 验证器框架 | 同源思想，Agent Insight把它工程化为可上线可回滚的生产闭环 |
| Artificial Analysis | 模型评测榜单（Bradley-Terry + bootstrap CI） | 方法论复用其A/B 维度；评的对象不同（模型 vs Skill增益） |

Agent Insight的差异集中在三点：
1. **置信加权 + 覆盖度折扣**让证据不足的Skill无法假装高分；
2. **4 维度天然映射到 4 类优化动作 + 4 道质量基线**，评测嵌入开发者路径而非旁路打分；
3. **评测在EDD中为人提供优化方向，在Self-Improving中为Agent的自动修改做验收**，把学术界自我进化研究和工业界评测平台打通。

此外，Agent Insight作为**Agent框架无关的统一AgentOps平台**，无论团队使用opencode、Hermes、OpenClaw还是其他框架构建Agent，都能接入同一套评测体系，获得统一的观测和优化能力。

# 七、结语

Skill的开发者路径是一条「开发 → 评测 ⇄ 优化 → 上线 → 持续监控」的螺旋，Agent Insight的评分穿透整条链路，既是优化的指南针，也是上线的质量基线、回归的探测器、Self-Improving阶段Agent自动修改的验收官。从「凭感觉」到「EDD」再到「Self-Improving」，是一条把人从循环里逐步移出去的路，每一步都依赖更可信的评测。Anthropic在 2025 年定义了Skill这种新的能力载体，后续核心工程问题是让Skill像微服务一样可治理、可灰度、可回滚。以及，在前面那一切都做对之后，安全地让它自我进化。Agent Insight提供的四种评测能力（静态合规、触发分析、用例分析、A/B 测试）正是这件事的工程起点。


## Agent Insight 已经开源啦！

嗨，各位伙伴！很高兴告诉大家，openEuler孵化的Agent Insight已经正式开源啦！🎉🎉🎉从现在起，项目的全部代码、文档都对内开放，每一位小伙伴都可以自由访问、使用，也欢迎你参与进来，一起把它变得更好。

- 🔗 仓库地址（点击阅读原文可访问）：[https://atomgit.com/openeuler/agent-insight](https://atomgit.com/openeuler/agent-insight)

- 📖 文档 / 使用指南：[https://atomgit.com/openeuler/agent-insight/blob/master/docs/user-guide/home.md](https://atomgit.com/openeuler/agent-insight/blob/master/docs/user-guide/home.md)

无论你擅长什么，这里都有你的舞台：

- 🐛 在实际使用中遇到问题？来提Issue，或直接反馈你的想法；
- ✨ 有新功能点子、想优化体验？欢迎提交设计或需求；
- 🔧 想动手写代码？认领任务、提交 Merge Request，你的代码会被看见；
- 📝 擅长文档或测试？完善说明、补充用例同样价值巨大。

开源不只是开放代码，更是把大家的能力和智慧连接起来。期待你的一条关注、一条反馈、一次贡献，让Agent Insight成为更强大的AgentOps平台 🚀 一起来玩，一起来造！

## 参考资料

1. Anthropic. *skill-creator (SKILL.md) — Create, Eval, Improve, Benchmark*. GitHub: anthropics/skills, 2026-03. [github.com/anthropics/skills/…](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
2. Anthropic. *Skill authoring best practices*. Claude API Docs. [platform.claude.com/docs/…](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
3. Anthropic. *Equipping agents for the real world with Agent Skills*. Anthropic Engineering, 2025-10. [anthropic.com/engineering/…](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
4. Snyk Labs. *Snyk Finds Prompt Injection in 36%, 1,467 Malicious Payloads in a ToxicSkills Study*. Snyk Blog, 2026-02-05. [snyk.io/blog/…](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)
5. Braintrust. *What is eval-driven development*. Braintrust Articles, 2026. [braintrust.dev/articles/…](https://www.braintrust.dev/articles/eval-driven-development)
6. Zhang Y. et al. *EvolveR: Self-Evolving LLM Agents through an Experience-Driven Lifecycle*. arXiv:2510.16079, 2025. [arxiv.org/abs/2510.16079](https://arxiv.org/abs/2510.16079)
7. Yang Y. et al. *AutoSkill: Experience-Driven Lifelong Learning via Skill Self-Evolution*. arXiv:2603.01145, 2026. [arxiv.org/abs/2603.01145](https://arxiv.org/abs/2603.01145)
8. Anonymous. *Audited Skill-Graph Self-Improvement for Agentic LLMs via Verifiable Rewards*. arXiv:2512.23760, 2025. [arxiv.org/abs/2512.23760](https://arxiv.org/abs/2512.23760)
9. Robeyns M. et al. *A Self-Improving Coding Agent (SICA)*. ICLR 2025 Workshop, arXiv:2504.15228, 2025. [arxiv.org/abs/2504.15228](https://arxiv.org/abs/2504.15228)
10. Claw-Eval Authors. *Claw-Eval: Toward Trustworthy Evaluation of Autonomous Agents*. arXiv:2604.06132, 2026. [arxiv.org/pdf/2604.06132](https://arxiv.org/pdf/2604.06132)
11. Artificial Analysis. *Intelligence Benchmarking Methodology*. [artificialanalysis.ai/methodology/…](https://artificialanalysis.ai/methodology/intelligence-benchmarking)