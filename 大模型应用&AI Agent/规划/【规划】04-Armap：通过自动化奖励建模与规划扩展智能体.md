# 【规划】04-Armap：通过自动化奖励建模与规划扩展智能体

原文链接：https://zhuanlan.zhihu.com/p/1918318256994911365

---

​

目录

## 1 概述

[大语言模型](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）在语言理解和生成任务上展现出卓越的能力，但在复杂环境中的多步决策、与环境交互（如在线购物、科学推理、数学问题求解）的任务中却力不从心，仍然面临着诸多挑战。与纯文本数据不同，大规模的采集决策数据难度大。此外，主流LLM仅能通过API访问，使得针对智能体任务进行微调的成本高、复杂度大。

因此，为解决上述挑战和问题，提出了**[Armap](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=Armap&zhida_source=entity) 框架，通过自动学习奖励建模（无需人工标注）和规划，来提升LLM智能体在各种任务中的决策能力，从而克服数据稀缺和API限制等问题，并实现更有效和可扩展的自主智能体**。该框架具备以下优点：

  * **有效性** ：提升各种LLM智能体在不同任务下的表现。
  * **灵活性** ：无需微调LLM本体，可在推理时灵活优化奖励目标，实现更可控的生成。
  * **实用性** ：自动奖励模型训练无需人工繁琐标注或SOTA LLM，应用更广泛。



图1 多步任务实现对比

图1(a)展示了LLM智能体在交互环境中生成多步计划，但生成一条正确的多步解决方案以抵达目标页面很难，因为需要预测多步动作并与环境互动；而图1(b)展示LLM学习奖励模型来评判轨迹是否符合任务指令，相对容易；图1(c)则展示了通过将奖励模型与基于LLM的智能体结合，并配合[蒙特卡洛树搜索](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=%E8%92%99%E7%89%B9%E5%8D%A1%E6%B4%9B%E6%A0%91%E6%90%9C%E7%B4%A2&zhida_source=entity)（MCTS）算法，可模拟并评估智能体任务的未来状态，提升策略优化能力。

## 2 ARMAP关键技术解析

ARMAP使用RL对LLM智能体的多步行为的奖励模型进行优化，并使用该奖励模型来指导LLM智能体对行为步骤的选择，使LLM智能体能够更有效地进行任务规划与决策。其主要创新包括：

  * **[自动奖励建模](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%A5%96%E5%8A%B1%E5%BB%BA%E6%A8%A1&zhida_source=entity)** ：利用LLM自身生成训练数据，通过合成轨迹数据学习奖励函数建模，无需人工标注数据集，显著降低了构建奖励模型的成本和复杂性。
  * **集成规划机制** ：将奖励模型与蒙特卡洛树搜索（MCTS）、Reflexion和[Best-of-N采样](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=Best-of-N%E9%87%87%E6%A0%B7&zhida_source=entity)等规划策略结合，提高了代理在多步骤决策任务中的表现。
  * **可定制的奖励目标** ：允许在推理过程中定制奖励目标，使得能够根据特定目标生成量身定制的行动计划。
  * **可扩展性与高效性** ：在多项基准任务中表现优异，对专有LLM API依赖极小。



### 2.1 ARMAP实现流程

ARMAP核心组件是经过训练的奖励模型 R ，该模型用于评估轨迹 h 是否成功完成任务：

r=R(x,h) (1)

  * r 为奖励模型预测的奖励；
  * x 为给定的任务指令；
  * h=\left\\{\left\\{a_{n} \right\\}_{n=1}^{N},\left\\{o_{n}\right\\}_{n=0}^{N}\right\\}_；_ \left\\{a_{n}\right\\}_{n=1}^{N} _为轨迹中的行动序列已完成任务目标；_ \left\\{o_{n}\right\\}_{n=0}^{N} _为环境观察；_ a_{n} 为第n步行动；每步动作后，环境转至状态 s_{n} ，智能体获得新观测 o_{n} 。



通过将奖励模型与LLM智能体结合，可在不同环境、不同规划算法下提升智能体的表现。

ARMAP框架流程包含：

  1. 使用LLM智能体在环境中进行导航，实现LLM生成的任务意图，同时收集大量行动轨迹示例。
  2. 使用LLM对收集的轨迹进行分析，为每条轨迹分配任务意图，并合成对应的正轨迹样本（达成目标）与负轨迹样本（未达成目标）。
  3. 基于收集到的合成数据（任务意图、正轨迹、负轨迹），训练定制化奖励模型，用以评估智能体行动轨迹是否满足用户意图。
  4. 训练完成的奖励模型可与各类LLM智能体协同优化任务规划，通过实时评估行动路径的有效性，显著提升复杂环境下的决策质量。



图2 ARMAP框架流程图

### 2.2 自动奖励数据生成

1\. **生成指令（生成任务意图）** ：利用LLM的上下文学习能力为给定观测生成任务指令。即在上下文中提供若干few-shot示例及环境状态观测，让LLM归纳观测内容并生成任务目标指令，最终收集合成文本指令集 \left\\{x_{m}^{raw}\right\\}_{m=1}^{M} ，其中M表示合成指令总数。具体提示词如下。
    
    
    Task Instruction: You are a helpful assistant to do some scientific experiments in an environment.    In the environment, there are several rooms: kitchen, foundry, workshop, bathroom, outside, living room, bedroom, greenhouse, art studio, and hallway.    The available actions are:    open OBJ: open a container    ...
    
    You will be given a task description and a corresponding trajectory. The task description concludes what you have done in this trajectory. You need to elaborate this description based on this environment by adding more details.
    
    Example:    Task Description: Your task is to grow an apple. You can find seeds in the kitchen. You should focus on the grown apple.    Corresponding Trajectory:    look around    This room is called the hallway. In it, you see:    ...    open door to kitchen    The door is already open.    go to kitchen    You move to the kitchen.    ...
    
    Refined Task Description: Your task is to grow an apple. This will require growing several plants, and them being crosspollinated to produce fruit. Seeds can be found in the kitchen. To complete the task, focus on the grown apple.
    
    Here is the task description you need to refine, and the corresponding trajectory is    also provided:    ...
    
    Refined Task Description:

2\. **采集轨迹** ：基于合成指令 x_{m}^{raw} 和环境，通过LLM智能体在环境中执行动作，生成完成任务的多样化轨迹 \left\\{x_{m}^{raw},h_{m}\right\\}_{m=0}^{M} 。此处 h_{m} 表示第m条历史轨迹，包含 N 个动作 \left\\{a_{n}\right\\}_{n=1}^{N} _和_ N+1 _个环境观测_ \left\\{o_{n}\right\\}_{n=0}^{N} 。

**3\. 构建正负轨迹对** ：为使奖励模型具备区分优劣轨迹的能力，还需采集负轨迹数据（即未达标轨迹）。通过修改 h_{m} 中的行动生成负轨迹 h_{m}^{-} ，将修正成功达标（正轨迹）的轨迹记为 \left\\{x_{m},h_{m}^{+}\right\\} ，未达标（负轨迹）的记为 \left\\{x_{m},h_{m}^{-}\right\\} 。这些成对数据将用于奖励模型的训练，使其能够评估环境中任意给定轨迹的奖励值。

### 2.3 奖励模型设计

**奖励模型架构**

奖励模型的目标是预测奖励分值以评估给定轨迹( x_{m},h_{m} )是否满足任务指令。可以采用任何能够接收视觉与文本序列输入的视觉-语言模型（VLM）作为奖励模型的基础模型。本文中选择了最新的[VILA模型](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=VILA%E6%A8%A1%E5%9E%8B&zhida_source=entity)作为奖励建模的基础。

**训练 &优化奖励模型**

基于已生成三元组数据（任务意图、正轨迹和负轨迹）来训练奖励模型，并为行动序列打分。

参考基于人类反馈强化学习的标准方法，通过二分类问题交叉熵损失优化奖励模型，确保模型能有效评估未来的行动方案。形式化定义为：

\mathcal{L}\left( \theta \right)=-E_{\left( x_{m},h_{m}^{+},h_{m}^{-} \right)}\left[ \log_{}{\sigma}\left( R_{\theta}\left( x_{m},h_{m}^{+} \right)-R_{\theta}\left( x_{m},h_{m}^{-} \right) \right) \right] (2)

其中 \sigma 为sigmoid函数， \theta 是奖励模型 R 的可学习参数。通过优化该目标，奖励模型被训练为对更接近任务指令目标的轨迹，给出更高评分。

### 2.4 结合奖励模型进行规划

获得评估轨迹与任务指令匹配度的奖励模型后，可将其与不同规划算法结合以提升LLM智能体的性能。以下总结了几种可采用的典型算法：

  * **Best of N** ：是一种利用奖励模型提升LLM智能体性能的简单算法。具体实现为：首先，提示LLM智能体独立生成n条不同轨迹；然后，选择奖励分数最高的那条轨迹作为最终预测。
  * **Reflexion** ：是一种使LLM可通过试错学习而无需微调的规划框架。该框架不更新模型参数，而是将任务结果转化为语言反馈，生成反思性总结并存储于情景记忆缓冲区以指导未来决策。Reflexion支持多种反馈类型，通过模拟人类自省式学习的语言强化机制，在决策、编程和推理任务中提升性能。
  * **MCTS** ：采用基于树搜索的蒙特卡洛树搜索（MCTS），用于寻找最优策略。算法构建的树结构中，每个节点代表一个状态，每条边表示一个行动。从根节点的初始状态出发，算法在状态空间中探索，根据奖励模型预测寻找高奖励行动-状态轨迹。 



算法追踪两个关键指标：（1）节点访问的频次；（2）记录在状态 s 采取动作 a 可获得最大预测奖励的价值函数。MCTS会优先访问和扩展具有较高价值（指向高预测奖励轨迹）或较低访问次数（探索不足）的节点。

## 3 实验

### 3.1 实验设置

**环境**

  * **Webshop** ：知名的在线购物测试平台，智能体需在网站上搜索和选择商品以获得最终结果。参照AgentBench中LLM评估配置，在验证集上测试模型，采用默认的匹配奖励作为评估指标。
  * **ScienceWorld** ：具身科学实验设计的交互式基准，智能体被置于文本模拟环境中，需要通过探索、操作物体和观察结果来完成基础科学实验，旨在验证AI模型是否具备科学知识应用能力而非单纯信息检索能力。
  * **24点游戏** ：数学逻辑测试任务，智能体需对给定的四个数字，通过加、减、乘、除运算得到24。



**LLM**

  1. 采用Llama3-70b-instruct模型合成自动奖励模型的训练数据。
  2. 为评估不同LLM智能体的表现，本地部署了Llama70B、Llama8B、[Mistral7B](https://zhida.zhihu.com/search?content_id=259192044&content_type=Article&match_order=1&q=Mistral7B&zhida_source=entity)和Phi3.8B，兼顾模型多样性和低成本。
  3. 奖励评估使用VILA模型。



**基线方案**

本研究测试了ARMAP的不同版本：

  * ARMAP-R：基于Reflexion的规划。
  * ARMAP-B：基于Best of N的规划。
  * ARMAP-M：基于MCTS的规划。



基线方法：Sampling（采样）和Greedy（贪心）

### 3.2 实验数据

表1 在不同基准上ARMAP的有效性

ARMAP框架在不同模型下均优于现有基线。相对于较弱模型，ARMAP提升较大；而对于强模型Llama70B，提交较小，因为弱模型会探索更多低奖励轨迹，从而给奖励模型带来更多提升空间。在三种规划算法中，MCTS平均表现最好。

表2 可控轨迹生成，动作预测模型为Llama70B

表2中展示了通过自定义奖励目标可生成更少步数、更低价格等可控轨迹。对ARMAP-M应用轨迹长度惩罚后，平均步数长度从4.5降至4，平均商品价格从97.9降至69.0，同时在默认匹配奖励上的表现几乎未受影响。ARMAP-B的表现类似。

ARMAP流程的另一大优势为可以在推理阶段自定义奖励目标，从而生成更可控的行动序列，而不仅仅是单纯最大化预测奖励。根据不同目标调整奖励目标的设计，例如加入步数Penalty，类似当前对长思考CoT长度的Penalty，引导智能体在更少的步数里面完成整个任务，或者在Webshop任务里面加入价格奖励，引导模型在购物搜索时寻找价格更低的产品。实现针对不同场景的智能体诉求，在已有任务目标的基础上，加入更多约束项。

## 4 总结

ARMAP提供了一种可扩展、低成本的替代传统强化学习的方法，旨在让LLM智能体能够自主管理多步决策与环境反馈的任务，为自主AI智能体开辟了新路径。该框架侧重奖励建模而非策略训练，提升了LLM智能体在复杂任务上的表现，绕开了昂贵微调和数据稀缺难题。未来发展方向包括： 

  * 整合常识推理的混合奖励模型。 
  * 针对用户个性化的自适应奖励学习。
  * 推动多智能体AI进入企业自动化等真实世界场景。
  * 增强奖励模型的可解释性，以便更好地理解和调试智能体的行为。 
  * 加强复杂指令的处理能力，降低智能体行为和指令意图产生的偏差。



## 相关链接

  1. [https://arxiv.org/abs/2502.12130](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2502.12130)
  2. [https://armap-agent.github.io](https://link.zhihu.com/?target=https%3A//armap-agent.github.io)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【规划】03-LLM整合多个函数调用的新思路！LLMCompiler：用于并行函数调用的LLM编译器](https://zhuanlan.zhihu.com/p/713091653)
