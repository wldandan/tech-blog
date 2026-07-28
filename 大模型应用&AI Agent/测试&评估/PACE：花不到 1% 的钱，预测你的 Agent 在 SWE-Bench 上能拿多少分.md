# PACE：花不到 1% 的钱，预测你的 Agent 在 SWE-Bench 上能拿多少分

# 一、Agent 评测为什么这么贵

LLM 的能力评估过去靠的是便宜又快的静态 benchmark。MMLU 测知识，HumanEval 测代码，IFEval 测指令跟随，一道题一次调用，几秒出结果，几分钱。

但 LLM 变成 Agent 之后，评估范式完全不一样了。SWE-Bench 要在真实代码仓库里定位 bug、改代码、跑测试；GAIA 要浏览网页、调工具、多步推理、输出精确答案；SWT-Bench 要为真实 bug 生成复现测试。这些 benchmark 的共同点是：**长周期、依赖工具和环境、基础设施复杂**。 单次评估上千美元、好几天。

结果很直接：团队在模型迭代中间跑不起完整 agentic eval。只能降低频率、在子集上报告、或者干脆不测。

但这里有个直觉上不对劲的地方。模型在 agentic benchmark 上的表现，依赖的是指令跟随、规划、工具使用、推理、代码生成这些基础能力，**这些能力便宜的 benchmark 早就分别在测了**。 SWE-Bench 考的核心是代码生成 + 规划 + 验证，LiveCodeBench、PlanBench、DebugBench 各自就在测这几件事，几分钱一次。

PACE 问的就是：能不能用一小撮精心挑选的非 agentic instance，可靠地预测模型在 agentic benchmark 上的分数？

答案是可以。**成本不到完整 agentic eval 的 1%**。

# 二、形式化：两个目标

PACE 定了两个目标：

**Goal A（分数预测）**： 给定预算 C（比如 100 个 source instance），从候选池里选 C 个 instance，训一个回归器，输入是模型在这 C 个 instance 上的得分向量，输出是它在目标 agentic benchmark 上的预测均分。回答"这个模型在 SWE-Bench 上大概能拿多少分"。

**Goal B（成对偏好预测）**： 用同样的 C 个 instance，判断任意两个模型谁在目标 benchmark 上更强。回答"A 和 B 谁在 GAIA 上更好"。

评估用的是留一法交叉验证（LOOCV）：14 个模型，每次留一个做测试，剩下 13 个用来选 instance 和拟合回归。这直接回答了一个关键问题：**PACE 能不能泛化到没见过的模型？**

# 三、实例筛选：双信号×双子集，与Bootstrap回归去噪

![alt text](image-3.png)

## 3.1 第一步：从 19 个benchmark里挑最有预测力的instance

候选池是19个非agentic benchmark，覆盖11种能力：指令跟随、长上下文聚合、错误恢复、规划、代码生成、信息检索、代码搜索、工具调用、推理、多模态理解、验证与测试。

问题在于怎么从成千上万个候选instance里挑出C个最有预测力的。直接做Lasso或Ridge回归选特征会过拟合——模型数量（14个）远小于instance数量（数万个），联合拟合严重欠定。

PACE的做法是**两个独立信号乘起来排序，取top-C**：

**局部信号（target relevance）：** 该instance在不同模型上的得分序列，跟target benchmark模型得分序列之间的Spearman秩相关性。相关性高说明模型在这个instance上表现好 ↔ 在目标benchmark上也表现好。

**全局信号（SVD leverage）：** 把所有模型在所有source instance上的得分堆成矩阵X，做SVD分解。每个instance的leverage score衡量它在X的行空间中贡献了多少结构信息——leverage高的instance在区分不同模型的特征向量上权重更大，去掉它信息损失更多。

两个信号乘起来：一个instance既要跟目标高度相关，又要在源分数结构中信息量大，才会被选中。单独用Spearman会选到冗余instance（两个跟目标高度相关的instance可能高度共线，多选不增加信息量），单独用SVD leverage可能选到信息量大但与目标完全不相关的instance。乘积同时约束两个维度。

## 3.2 第二步：回归 + Bootstrap 抗噪

选好C个instance后，每个模型用这C个instance的得分构成一个C维向量，做最小二乘回归预测目标benchmark的平均分。

但这里有一个实际问题：agentic benchmark的instance本身就少（SWE-Bench Verified 500个、GAIA 165个），从几百个instance算出来的模型均分是**有噪声的估计**——换一批target instance，同一模型的均分可能浮动好几个点。把有噪声的均值当精确标签做回归，权重会对单次采样过拟合。

PACE的处理方式是**bootstrap**：对target instance做B次有放回重复采样，每个模型产生B个不同的伪均值。回归训练时把这B个伪均值都当标签用，让模型看到同一模型在不同target instance子集上的波动。训练出的权重对target instance的具体构成不敏感，泛化到hold-out模型时更稳定。

对于"模型A和B谁更强"（Goal B），PACE用Bradley-Terry配对比较变体：将两个模型的C维分数向量之差作为输入，拟合逻辑回归预测谁会赢，同样用bootstrap稳定训练。

# 四、效果

14 个模型、4 个agentic benchmark（GAIA、SWE-Bench Multimodal、SWE-Bench Verified、SWT-Bench）、19 个source benchmark。只用 **100 个 proxy instance**：

![alt text](image-4.png)

- **留一法交叉验证MAE < 4%**（在 14 个模型上依次留一个做测试、其余做训练，平均绝对误差不到 4 个百分点）
- **Spearman秩相关性 > 0.80**（预测排名和真实排名高度一致）
- **成对模型排序准确率约 85%**（任意两个模型谁强谁弱，100 个instance能判断对 85% 的情况）

**成本不到完整agentic评测的1%。** 同等预测质量下，PACE的成本约是对target benchmark做随机子采样的 **1/100**。预算曲线显示：大约100个instance之后，边际提升开始饱和，再往上还能小幅改善但回报递减很陡。100个instance是一个高性价比甜点。

![alt text](image-5.png)

---

# 五、放在一起

PACE用两个信号（局部Spearman相关性 + 全局SVD leverage）从便宜的benchmark里挑100个instance，做线性回归就能以MAE < 4%的精度预测agentic benchmark分数。Bootstrap让回归在target instance稀缺时依然稳定。成本1%，精度够迭代中间用了。

Agent评测正在从"跑完看分数"的粗活，变成可以分层诊断、可以省钱的工程实践。


## 相关链接

1. 微服务诊断基准：https://arxiv.org/abs/2606.29193v1
2. 数据集：https://www.aiops.cn/gitlab/aiops-live-benchmark/agenticopseval
3. ACE：https://arxiv.org/abs/2607.02032v1
4. ACE代码：https://github.com/neulab/pace
