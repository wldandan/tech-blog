# 【测试/评估】03-Thinking-LLM-as-a-Judge：基于规划和推理学习的评估方法

原文链接：https://zhuanlan.zhihu.com/p/1908812520539534270

---

​

目录

## 1 概述

随着[大语言模型](https://zhida.zhihu.com/search?content_id=258037595&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLMs）的不断发展，如何可靠的评估其长文本输出面临巨大挑战。由于人工评估的成本高昂、耗时且容易出现偏差，因此催生出[LLM-as-a-Judge](https://zhida.zhihu.com/search?content_id=258037595&content_type=Article&match_order=1&q=LLM-as-a-Judge&zhida_source=entity)范式，将LLM自身作为评估者。LLM-as-a-Judge模型会通过生成[思维链](https://zhida.zhihu.com/search?content_id=258037595&content_type=Article&match_order=1&q=%E6%80%9D%E7%BB%B4%E9%93%BE&zhida_source=entity)（CoT）展示其评估推理过程，以提升评估透明度和准确性。然而LLM-as-a-Judge任然面临着两大问题：（1）人工标注的思维链数据稀缺导致训练困难；（2）现有方法依赖于人工编写各领域定制化的评估标准或流程（如安全与代码需不同规则），导致其扩展性差，难以适应多任务需求。

为了克服上述问题，Meta推出了[EvalPlanner](https://zhida.zhihu.com/search?content_id=258037595&content_type=Article&match_order=1&q=EvalPlanner&zhida_source=entity)。**EvalPlanner，一种用于构建Thinking-LLM-as-a-Judge模型的[偏好优化](https://zhida.zhihu.com/search?content_id=258037595&content_type=Article&match_order=1&q=%E5%81%8F%E5%A5%BD%E4%BC%98%E5%8C%96&zhida_source=entity)方法，旨在通过优化的计划-执行策略来提高其规划和推理能力，从而提升评估的准确性、可扩展性和透明度**。

图1 EvalPlanner的典型输入输出，输入为用户指令与待评估响应对，输出为结构化思维链推理序列：包括规划部分（评估规划）、推理部分（执行规划）及最终判决。

## 2 EvalPlanner技术解析

**EvalPlanner创新之处在于通过解耦式思维链设计** 将评估流程分离为“规划”与“推理”两阶段：

  1. **规划阶段** ：生成包含任务专属评估步骤的详细计划。
  2. **推理阶段** ：分步执行计划、分析响应，得出判断。



然后，通过[自改进循环](https://zhida.zhihu.com/search?content_id=258037595&content_type=Article&match_order=1&q=%E8%87%AA%E6%94%B9%E8%BF%9B%E5%BE%AA%E7%8E%AF&zhida_source=entity)，模型持续采样多组计划-执行过程，并对正确/错误的思维链三元组（评估计划、执行、判断）进行偏好优化，从而同步提升（1）根据任务需求生成优质计划（包含相关标准、评分规则、参考答案等）和（2）基于生成的计划正确执行的能力。

基于上述解耦的思想，EvalPlanner可以更好地协调评估目标和推理过程，从而得出更准确、可解释的判断。同时，通过不断的自改进循环，EvalPlanner确保了比现有LLM-as-a-Judge模型更可靠、可扩展的评估方法。

### 2.1 方法概述

针对复杂指令生成的长文本响应本质上属于规划+推理问题，即评估者首先需要制定评估方案，然后根据方案和响应进行推理，最终得出判断。在EvalPlanner中，假设有效的评估思维链包含三个要素，生成评估计划 z ，执行计划 e ，最终判决 y 。

  * **生成评估计划**z：对于给定输入指令 x ，评估计划详细的规定了如何评价该指令下的响应。
  * **执行计划**e：负责逐步按照评估计划，分析输入的响应对 a 和 b ，最终生成判决 y 。
  * **最终判决**y：假设LLM-as-a-Judge模型参数为 \theta ，其中规划 z 与执行 e 均为潜在变量，可将最终判决 y 的生成过程表述如下：



p_{\theta}(y|x,a,b)=\sum\limits_{z\in P}\sum\limits_{e\in \varepsilon}p_{\theta}(y|e,z,x,a,b)p_{\theta}(e|z,x,a,b)p_{\theta}(z|x)

EvalPlanner依据该生成过程构建思维链的偏好对用于模型训练，其工作流程如图2所示。

  1. 给定指令和种子模型，首先从多个规划 z\in P 中进行采样。
  2. 随后，针对给定的计划、指令和响应对，从计划的多个执行 e∈\varepsilon 中进行采样，这些执行可能导致正确或错误的最终判决。
  3. 利用这些数据，设计了一个自训练循环，通过联合优化方案和执行来训练LLM-as-a-Judge模型，从而提升判决质量。
  4. 在测试阶段，模型生成结构化思维链 \tilde{y}=(\tilde{z},\tilde{e},\tilde{y}) ，包含“计划-执行-判决”三部分。



图2 EvalPlanner：基于思考型LLM-as-a-Judge模型学习的规划与推理评估方法

### 2.2 生成合成训练数据

本节将介绍生成合成训练数据的流程，以及如何构建偏好对和思维链。

**提示词选择与响应对生成**

选择通用指令遵循及数学推理两类提示词。

  * 对于通用指令遵循提示词：先将原始指令修改为“噪声化”版本，再针对噪声指令生成响应。因此，对原始指令的响应被视为“被选中”响应，对噪声指令的响应则为“被拒绝”响应。
  * 对于数学推理提示词：采样多个响应，其中带来正确解答的为“被选中”响应，错误解答的为“被拒绝”响应。



**生成评估计划**

获得上述合成的偏好对后，需要生成评估计划。通过设计通用且无约束的计划生成提示词，根据输入指令向种子模型（如微调过的LLM）查询初始计划。计划生成提示词不包含响应条件。
    
    
    We want to evaluate the quality of the responses provided by AI assistants to the user question displayed
    below. For that, your task is to help us build an evaluation plan that can then be executed to assess the response quality. Whenever appropriate, you can choose to also include a step-by-step reference answer as part of the
    evaluation plan. Enclose your evaluation plan between the tags “[Start of Evaluation Plan]” and “[End of Evaluation Plan]”.
    [User Question]
    {instruction}

**执行评估计划**

在计划执行阶段，使用相同种子模型，基于指令、响应对和上步生成的计划进行推理并执行，得出判决结论。解耦规划和执行阶段有两个好处：（1）确保推理/执行过程必须遵循既定规划；（2）通过对同一规划的多个执行进行采样，使模型能兼顾规划与执行的多样性，获得丰富的评测数据。与初始规划类似，初始执行也将在后续自训练中得到优化。

如下为计划执行提示词。
    
    
    Please act as an impartial judge and evaluate the quality of the responses provided by two AI assistants to the user
    question displayed below. Your evaluation should be performed by following the provided evaluation plan step-by-step.
    Avoid copying the plan when doing the evaluation. Please also only stick to the given plan and provide explanation of
    how the plan is executed to compare the two responses. Avoid any position biases and ensure that the order in which
    the responses were presented does not influence your decision. Do not allow the length of the responses to influence
    your evaluation. Do not favor certain names of the assistants. Be as objective as possible. After providing your evaluation, output your final verdict by strictly following this format: “[[A]]” if assistant A is better, “[[B]]” if assistant B is better.
    
    [User Question]
    {instruction}
    
    [The Start of Assistant A’s Answer]
    {response A}
    [The End of Assistant A’s Answer]
    
    [The Start of Assistant B’s Answer]
    {response B}
    [The End of Assistant B’s Answer]
    
    [The Start of Evaluation Plan]
    {evaluation plan}
    [The End of Evaluation Plan]

**构建规划与执行的偏好对**

基于规划及其推理执行的偏好对，构建偏好数据集以优化思维链。对每个输入指令，采样 P 个计划；对每个计划，采样 E 次执行。最终每个指令产生 2PE 个思维链。若三元组（计划，执行，判断）得出正确判决，则认为该推理是正确的，否则为错误。基于此正确性标准，构建偏好调优数据集。

### 2.3 计划与执行的偏好优化

EvalPlanner的训练流程采用自训练循环机制：以种子模型 M_{0} （如指令微调的LLM）为起点，首先在部分“被选中”的思维链（CoT）上进行监督式微调（SFT），得到模型 M_{1}^{SFT} ，随后进行两轮思维链（CoT）偏好对的直接偏好优化（DPO，Direct Preference Optimization），依次得到 M_{1}^{DPO} 和 M_{2}^{DPO} 。

  * M_{1}^{SFT} ：以种子模型 M_{0} 和一部分输入指令及响应对为起点，依照第2.2章节的方法生成思维偏好对，记为数据集 D_{1} 。然后在 D_{1}^{c} （D1中“被选中“思维链子集）上进行微调，获得型正确遵循“计划-执行-判决”的思维链范式。具体而言，针对每个指令随机采样一个正确思维链（导向正确判决的样本），执行SFT训练后获得模型 M_{1}^{SFT} 。  

  * M_{1}^{DPO} ：以 M_{1}^{SFT} 为初始模型，在包含“被选中“和“被拒绝“思维链的数据集 D_{1} 上进行DPO。由于思维中包含计划和执行两个部分，DPO使模型能够区分正确与错误的思维，两者在计划和评估执行上都可能不同。最终得到模型 M_{1}^{DPO} 。  

  * M_{2}^{DPO} ：第二轮DPO，选取新的指令和响应对子集，用模型 M_{1}^{DPO} ，按同样流程生成新的CoT数据集。具体而言，对每个训练数据点，从 M_{1}^{DPO} 中采样 P 个CoT，提取出其中的计划，再用 M_{1}^{DPO} 为每个计划采样 E 个执行。将这轮CoT数据记为 D_{2} 。然后在新的输入与思维链上进行训练。  




## 3 实验

### 3.1 实验设置

**训练数据**

  * 提示词来源：WildChat和MATH  
对于WildChat，直接使用Self-Taught Evaluators生成的合成响应。对于MATH问题，使用Mixtral 22Bx8 Instruct模型生成多个候选答案，正确答案的响应作为“被选中”响应，错误答案的响应作为“被拒绝”响应。
  * 模型：Llama-3.1-70B-Instruct或Llama-3.3-70B-Instruct作为种子模型，采样温度为 0.8，top_p为 0.95。



**基线**

将EvalPlanner与多种模型对比，包括：（1）零样本作为评委的强大开源/闭源LLM，（2）可生成标量分数及点评的奖励模型，（3）RewardBench榜单上的最新生成式奖励模型。

### 3.2 实验结果

EvalPlanner在训练数据更少、完全由合成方式生成偏好对的情况下，超越了所有基线模型。如表1展示了在RewardBench上的结果。EvalPlanner的训练方法在两个Llama种子模型上表现相当，证明了初始训练数据的有效性以及该方法的可推广性。

表1 EvalPlanner与SOTA生成式奖励模型在RewardBench上的对比

EvalPlanner数据高效，受益于迭代式思维优化。如表2所示，EvalPlanner仅用5K偏好对即可取得92.3分，已与RewardBench上最优模型相当。同时，进行两轮DPO的效果，相较于同等数据量的一轮DPO仅略有提升（92.3→92.5）。这种迭代提升归因于模型在新数据点上用更新版CoT增强了数据集。

表2 EvalPlanner在RewardBench上两轮DPO与一轮DPO的比较

## 4 总结

Meta提出的EvalPlanner是一种面向Thinking-LLM-as-a-Judge模型的高鲁棒性、高数据效率的新范式，为AI评估领域树立了新的标准，证明了LLM通过规划和推理学习，可以显著提高评估质量。通过将评估流程解耦为“规划”与“推理”两阶段，EvalPlanner增强了推理过程的可解释性。同时，通过不断的自改进循环，EvalPlanner确保了比现有LLM-as-a-Judge模型更可靠、可扩展的评估方法。

## 5 附录示例

### 5.1 EvalPlanner为编码问题生成的计划示例

下图展示了EvalPlanner为编程问题生成的评估计划示例。该模型会生成多个测试用例，包含用于验证代码正确性的无效输入和边界案例。

### 5.2 EvalPlanner为数学问题生成的计划示例

下图展示了EvalPlanner为数学问题生成的评估计划示例。计划包含由评判模型生成的分步解答过程，以及多个用于对比响应质量的评估标准。

### 5.3 EvalPlanner为安全问题生成的计划示例

下图展示了EvalPlanner为安全问题生成的评估计划示例。计划包含多项评估标准、评估步骤（含确保符合伦理准则的反馈机制）、评分细则以及高质量的参考答案框架。

## 相关链接

  1. [https://arxiv.org/abs/2501.18099](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2501.18099)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【评估技术洞察】02-Agent-as-a-Judge: 用智能体评估智能体](https://zhuanlan.zhihu.com/p/13238264056)  
> 下一篇：[【评估技术洞察】04-LLM-based Agents评估综述](https://zhuanlan.zhihu.com/p/1911378309859739418)
