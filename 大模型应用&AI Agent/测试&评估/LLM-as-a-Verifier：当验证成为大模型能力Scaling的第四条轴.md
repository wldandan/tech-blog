# LLM-as-a-Verifier：当"验证"成为大模型能力Scaling的第四条轴

###### PART 01
# 一个被忽视的瓶颈

过去几年，大模型能力的提升遵循着一条清晰的路线图：**扩大预训练数据和算力、优化后训练策略、增加推理时的计算量**。这三条scaling轴几乎定义了整个领域的发展方向。

但有一个问题始终悬而未决：***模型产出的质量谁来保证？***

![alt text](LLM-as-a-Verifier：当验证成为大模型能力Scaling的第四条轴.png)

想想看，当我们让模型反复采样，它常常能在某一轮给出正确答案，只是我们认不出哪个解是对的。论文给了一个震撼的数字：在Terminal-Bench V2上，如果把多个模型的轨迹池化在一起，配合一个能"总能挑出最佳轨迹"的oracle verifier，**成功率可以冲到98.9%**，几乎把这个hard benchmark给"解"了。

问题在于，我们没有一个可靠的verifier来捕捉这个headroom。

![alt text](image.png)

图1 LLM-as-a-Verifier在三个领域四个benchmark上的表现一览

这就是Stanford和UC Berkeley联合团队在**LLM-as-a-Verifier**这篇论文中要解决的核心问题。他们做出了一个重要判断：**验证（verification）应当被视为大模型能力scaling的第四条独立维度**，而不是附属于生成的次要环节。

---

###### PART 02
# 为什么Judge不够用？

当前最常用的评估方案是**LLM-as-a-Judge**：给模型一个prompt，让它输出一个离散分数（比如 1-5 分），然后取概率最高的那个token作为最终得分。

听起来没问题，但仔细一想就发现一个关键缺陷：***这个操作把整个评分分布"坍缩"成了一个点***。

论文给了一个非常直观的案例（query-optimize 任务）：同一对轨迹，用离散的 1-5 分judges，**88/100次评估给出了平局**（两个轨迹都得 5 分）。但实际上，其中一条轨迹在验证方法论上有根本性缺陷，它在验证优化结果时做了一次"作弊式"的数据库替换，根本没有真正验证原始查询和优化查询的等价性。LLM在reasoning中确实识别出了这个问题，但措辞很含蓄："slightly cleaner"、"marginally more direct"。当这些nuanced的评估被压成一个整数时，所有细粒度的信息都丢了。

**离散分数 → 高tie rate → 无法区分方案 → 退化为随机选择。**这就是传统LLM Judge的死穴。

---

###### PART 03
# 核心方法：把"打分"变成一个概率问题

LLM-as-a-Verifier的解法出奇地简洁，也出奇地有效。

## 3.1 不取argmax，求期望

传统Judge做的事情是：

$$R_{\text{LM}}(x, \tau) = \arg\max_g p_\theta(v_g | x, \tau)$$

即取概率最高的token作为最终分数。而LLM-as-a-Verifier做的事情是：

$$R(x, \tau) = \frac{1}{CK} \sum_{c=1}^{C} \sum_{k=1}^{K} \sum_{g=1}^{G} p_\theta(v_g | x, c, \tau) \cdot \varphi(v_g)$$

***改为对评分token的概率分布求期望***。这个改动看起来不大，但它解开了一系列scaling的可能性。

> **关键直觉**：模型的评分logits分布包含了丰富的不确定性信息。直接取argmax等于丢弃了所有这些信息。取期望则完整保留了模型对候选方案质量的"信念分布"。

得到连续分数后，论文用Bradley-Terry模型将其转化为pairwise preference：

$$P(\tau_i \succ \tau_j | x) = \frac{1}{1 + \exp(-(R(x, \tau_i) - R(x, \tau_j)))}$$

这样，任意两条轨迹之间就不再是"分数是否相同"的离散比较，而是一个**连续的、有概率含义的偏好度**。

![alt text](image-1.png)

图2 LLM-as-a-Verifier框架总览

## 3.2 验证Scaling三条轴

一旦把验证变成了概率期望的形式，就自然暴露出了三个独立的scaling维度：

:::success
***轴一：评分粒度（Score Granularity）***

传统judge用 1-5 分。论文尝试了1-20 分。直觉上，分的数量本身不增加信息，但它给了模型一个**更细粒度的投影空间**，两个在 5 分制下都会被round到 5 分的方案，在 20 分制下可能分别是 17.2 和 15.6。

论文用了一个信号噪声比（SNR）度量来量化这个效果：

$$SNR(G) = \frac{\mathbb{E}[s_c - s_i]}{\sqrt{\text{Var}(s_c - s_i)}}$$

从G=1 到G=20，SNR从 0.775 提升到 0.799，配对验证准确率从 73.1% 提升到 77.5%。

回到query-optimize的例子：离散 1-5 分judges产生了 88% 的tie rate；改用 5 分制的连续期望后，tie rate归零，正确排序率达到 69%；进一步扩展到 **20分制，正确排序率达到77%**。

***轴二：重复评估（Repeated Evaluation）***

单次评估可能因为prompt的偶然特征或模型推理的随机性而产生偏差。一个自然的做法是：**独立评估 K 次，取平均**。

这是蒙特卡洛估计的标准逻辑：方差按 O(1/K) 缩小。实验显示，K=1 时准确率 74.7%，K=16 时达到 77.5%。更重要的是，**单次评估的verifier（K=1）已经匹配甚至超过了 K=16 的离散judge**，这说明连续概率信号本身就足够强，重复评估是锦上添花。

***轴三：准则分解（Criteria Decomposition）***

"这条轨迹正确吗？"这个问题对一个长horizon的agentic任务来说太compound 了。模型很容易被prompt中最salient的因素带偏。

论文的做法是把一个monolithic的评估分解为多个子准则。对代码agent，分解为三个维度：
- **Specification**：是否满足所有任务需求
- **Output**：最终输出格式是否匹配预期
- **Errors**：日志和工具输出中是否存在失败信号
:::

单个准则的准确率在 75.2%-76.4% 之间，**三者ensemble后达到 78.3%**。三个维度的增益几乎是可加的，说明它们捕获了互补的质量信号。

![alt text](image-2.png)

图3 验证Scaling的三维实验

---

###### PART 04
# 概率枢轴锦标赛：O(N²) 降到 O(Nk)

有了可靠的pairwise验证信号，下一个问题是：***如何从N个候选轨迹中选出最好的？***

最直接的方法是全轮赛（round-robin），但需要`O(N²)`次pairwise比较，费钱。论文提出了一个聪明的替代方案，**概率枢轴锦标赛（Probabilistic Pivot Tournament, PPT）**，把复杂度降到 `O(Nk)，k ≪ N`。

PPT分三步走：

:::success
***Step 1: Ring Pass（环形淘汰）***
随机生成一个Hamiltonian环，让每个候选轨迹在N个相邻对中各出现一次：**恰好一次在"A"位置，一次在"B"位置**。这一步有一个巧妙的设计意图：如果LLM对A/B位置有系统性偏好（position bias），环形结构能让这个bias在期望意义上消掉。

***Step 2: Pivot Selection（枢轴选择）***
按ring pass累积的归一化偏好得分排序，**选前 k 个作为"枢轴"**。这里的关键决策是把剩余预算集中在不确定的高分区，与其花资源区分"肯定不行"和"更不行"的候选，不如聚焦在那些"可能正确"的候选之间做精细比较。

***Step 3: Pivot Tournament（枢轴对战）***
非枢轴只和枢轴比，枢轴之间互相比。`总比较次数 = N + k(N-k) + C(k,2) = O(Nk)`。
:::

当`k=3`时，PPT（4,723 对比较）已经超越V1 baseline在 7,000 对比较下的表现。`k=9`时，只用全轮赛 73% 的预算就接近了全轮赛的准确率。

![alt text](image-3.png)

图4 PPT的五阶段流水线

---

###### PART 05
# 跨领域验证：一次方法，四个SOTA

LLM-as-a-Verifier最令人信服的地方在于它**完全不需要训练，也不做任何per-domain的微调**。同一套框架，同一个Gemini 2.5 Flash verifier（granularity=20, K=8, 三准则分解），直接应用到四个差异巨大的benchmark上：

| 领域 | Benchmark | 基座模型 Pass@1 | Oracle 上界 | LLM-as-a-Verifier | 提升 |
|------|-----------|----------------|-------------|-------------------|------|
| 编程 | Terminal-Bench V2 | 83.1% | 92.1% | **86.5%** | +3.4% → SOTA |
| 编程 | SWE-Bench Verified | 76.1% | 84.4% | **78.2%** | +2.1% → SOTA |
| 机器人 | RoboRewardBench | — | — | **87.4%** | +6.0% vs RoboReward-8B |
| 医疗 | MedAgentBench | 70.2% | 75.0% | **73.3%** | +3.1% → SOTA |

几个亮点值得展开说说：

**Terminal-Bench V2** 是shell-based长horizon任务，很多失败轨迹的终端状态看起来"语法上没问题"，但实际上不对。GPT-5.5的`pass@1`是 83.1%，`oracle pass@5`是 92.1%。LLM-as-a-Verifier把实际成功率拉到 86.5%，超越了包括Claude Mythos + Terminus-2 (82.0%) 和GPT-5.5 + NexAU-AHE (84.7%) 等强基线。

**SWE-Bench Verified** 的实验设置更有意思：候选池是**异构的**，来自Claude Opus 4.5、Gemini 3 Flash和MiniMax M2.5三个不同模型家族的方案混在一起。Verifier需要跨越不同代码风格和解题策略来判断"哪个patch更好"。78.2% 的结果说明这套方法对模型风格差异是鲁棒的。

**RoboRewardBench** 上的结果尤其值得注意。这里输入是**多帧视频而非文本**，verifier需要理解物理操作过程。Qwen 3.6 35B作为VLM verifier，零样本、不微调，就在偏好准确率上跑赢了专门用 ~45k robotics数据训练的RoboReward-8B（87.4% vs 81.4%）。这个结果直接挑战了"reward model必须per-domain训练"的假设。

---

###### PART 06
# 超出Best-of-N：验证信号的意外收获

论文最有洞察力的部分可能不在verifier本身，而在于**它产出的那个连续的、fine-grained的分数可以做更多事情**。

## 6.1 任务进度追踪

直觉上，一个好的verifier应该在成功的轨迹上给出**单调递增**的分数，越往后越接近目标，分数应该越高。论文用Value-Order Correlation（VOC）量化了这个想法：将轨迹的step序号和该step处verifier打分做Spearman秩相关。

结果非常漂亮：
- **代码生成**（Terminal-Bench V2）：成功轨迹的VOC达到 **0.848**，失败轨迹仅 0.769，两者之间有 0.08 的gap。
- **机器人操作**（RoboRewardBench）：VOC达到 **0.966**，远超RoboReward-8B (0.877) 和Robometer-4B (0.780)。

这个单调递增的性质有一个很实在的应用：**你可以把它做成一个实时监控面板**。当verifier分数在某个agentic任务中持续不涨甚至下降时，大概率是agent卡住了或者在干错误的事，这时候就应该人工介入，而不是坐等最终失败。

论文团队也正是这么做的。他们为Claude Code和Codex开发了**TurboAgent** 扩展，一个透明的推理时代理层，可以在不改动底层agent harness或模型API的情况下，实时展示verifier的分数和进度曲线。对于需要跑好几个小时的agentic任务来说，这相当于给开发者装了一个"心电图"。

## 6.2 稠密奖励，让RL少走弯路

验证器的连续信号还顺手解决了一个RL训练里的老大难问题：**信用分配**。一个稀疏的reward（比如只有任务完成才给 1，否则给 0）会导致梯度信号极少，训练效率低。LLM-as-a-Verifier的连续分数天然是稠密的。

论文在两个不同范式的RL上做了验证：

**Off-policy（DSRL-SAC + π₀ policy on LIBERO）**：用verifier对rollout中每个frame打分，得到一个per-step的progress curve $ρ_t$，叠加到环境reward 上：$r_t = r_env_t + λ·ρ_t$。效果是 **约 1.8 倍的采样效率提升**，同时最终成功率也从 0.69 提升到 0.76。

**On-policy（GRPO + Qwen3-8B on MATH）**：在GRPO训练早期，经常出现所有sampled responses都给出错误答案的情况，导致group-relative advantage坍缩为零、没有梯度。Verifier的reasoning reward可以在"最终答案都是错的"的情况下仍然区分出"谁的推理过程更好"，提供约 **1.1 倍的采样效率提升**。

> 一个可以类比的理解：传统的稀疏reward像只有"及格/不及格"的考试反馈；LLM-as-a-Verifier的稠密reward更像一个能告诉你"第几步做得不错，第几步有问题"的详细批改。后者对学习的帮助显然更大。

---

###### PART 07
# 七、局限与展望

论文坦率地列出了几个局限，也暗示了未来的方向：

- **Logits依赖**：需要能返回token logprobs的模型。虽然two-stage workaround解决了部分问题，但最优雅的方案还是希望更多模型API开放logprobs访问。
- **准则设计靠人工**：当前的criteria decomposition是手工设计的。如果能够**自动学习或动态生成per-domain的分解准则**，scalability会更好。
- **RL实验限于单轮**：目前的RL实验在单轮生成和较短horizon上进行。扩展到多轮、长horizon的agentic RL，让verifier为策略提供per-step reward来引导复杂环境中多步决策的信用分配，是一个很有前景的方向。
- **验证scaling的上限**：从 G=16 到 G=20，收益已经放缓。是否还有别的维度（比如多模型verifier的ensemble、动态分配评估预算）可以进一步push，值得探索。

---

###### PART 08
# 八、写在最后

LLM-as-a-Verifier这个工作的价值，我想可以归结为两点：

**第一，它重新定位了"验证"在LLM系统中的地位。** 过去验证是附属于生成的一个评估步骤，是"事后把关"。这篇论文论证了验证本身就是一条独立的scaling轴，你不只可以花更多compute去生成更好的候选，还可以花更多compute去更好地选择候选。两者协同，才能完整释放模型的能力。

**第二，它证明了概率化思维的力量。** 从"argmax取一个token"变成"对分布求期望"，这个改动只涉及几行代码，但它解锁了scoring granularity、repeated evaluation、criteria decomposition三条 scaling路径，衍生出了PPT排序算法，甚至让同一个分数既可以是verifier又可以是progress tracker又可以是RL reward。**好的抽象确实能让一个简单机制在不同场景下复用出多种价值。**

对于正在构建agentic系统的团队来说，LLM-as-a-Verifier提供了一个即插即用的方案：不训练任何模型，不改动现有harness，只是在候选轨迹评估这个环节换一种概率化的方式，就能获得可观的性能提升和额外的监控能力。

# 相关链接
1. 论文：https://arxiv.org/abs/2607.05391
2. 代码：https://github.com/llm-as-a-verifier/llm-as-a-verifier
3. 项目官网：https://llm-as-a-verifier.com

