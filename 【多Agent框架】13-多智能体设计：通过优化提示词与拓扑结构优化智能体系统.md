# 【多Agent框架】13-多智能体设计：通过优化提示词与拓扑结构优化智能体系统

原文链接：https://zhuanlan.zhihu.com/p/1910732602626797678

---

​

目录

## 1 概述

近来，随着智能体（Agent）在解决复杂任务上的表现，各大厂商/初创公司持续发力智能体（Agent），相继推出如[Deep Research](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=Deep+Research&zhida_source=entity)、Manus等产品。尤其是[多智能体系统](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E7%B3%BB%E7%BB%9F&zhida_source=entity)（MAS）在协同解决复杂任务方面，更是具有突出表现。MAS通过定制化提示词定义多个独立的智能体角色，每个Agent都拥有其独特角色定义、领域知识和资源，然后基于拓扑结构编排协作流程，共同完成复杂的决策任务，能够显著提升任务处理能力。

然而，设计高效MAS往往极具挑战，如下：

  * 提示词设计存在敏感性，即使细微调整也可能引发显著性能波动，且MAS中，级联使用的智能体会放大该问题。
  * 拓扑结构设计空间庞大，人工试错成本高昂，而现有自动化方法对提示词与拓扑的协同作用缺乏系统认知，难以平衡组合爆炸的搜索效率与性能优化。



为此，本文提出多智能体系统搜索（Mass）框架，通过三阶段交错优化实现自动化设计和优化： 

  1. 对单智能体进行模块级提示词局部优化； 
  2. 在剪枝后的拓扑空间中优化[工作流拓扑](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=%E5%B7%A5%E4%BD%9C%E6%B5%81%E6%8B%93%E6%89%91&zhida_source=entity)结构； 
  3. 基于最优拓扑开展全局（工作流级）提示词调优。



该方法通过分层解耦的关键设计要素，在多个基准任务中显著超越传统方案，并为高效MAS构建提供了"提示词优先、拓扑精简化"的实践原则。

图1 多智能体系统搜索（Mass）框架优化示意图

如图1所示，Mass通过交错的[提示词优化](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=%E6%8F%90%E7%A4%BA%E8%AF%8D%E4%BC%98%E5%8C%96&zhida_source=entity)与拓扑优化，在可定制的多智能体设计空间（左侧为关键组件示意）中发现高效的多智能体系统设计（右侧为优化后的拓扑与提示词）。

## 2 Mass关键技术解析

### 2.1 多智能体系统设计方法论

在MAS中，将智能体（或等效的构建模块）的**结构化排列** 定义为**智能体的拓扑结构** 。**工作流** $W$定义为用于构建MAS的**跨拓扑的逻辑序列** 。因此，MAS的设计可分解为两个层级：模块级设计与工作流级编排。

  * **模块级设计** ：关注通过更好的提示词设计构建高效的单个智能体，以便更好地执行其预期角色。
  * **工作流级编排** ：关注智能体的类型、数量，以及如何最有效的编排它们（即拓扑优化）。



### 2.2 Mass：多智能体系统搜索框架

图2 [Mass框架](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=Mass%E6%A1%86%E6%9E%B6&zhida_source=entity)示意，展示搜索空间与多阶段优化流程

如图2所示，搜索空间涵盖提示词(指令、示例)与可配置模块(Aggregate、Reflect、Debate、Summarize、Tool-use)。优化流程分为：（1）[模块级提示优化](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9D%97%E7%BA%A7%E6%8F%90%E7%A4%BA%E4%BC%98%E5%8C%96&zhida_source=entity)：独立优化各模块提示；（2）工作流拓扑优化：基于第1阶段各模块最佳提示词，从加权设计空间中采样有效配置，并融合各模块提示词；（3）[全局提示优化](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=%E5%85%A8%E5%B1%80%E6%8F%90%E7%A4%BA%E4%BC%98%E5%8C%96&zhida_source=entity)：以第2阶段最佳工作流为基础，进行全局提示词优化。

算法1

Mass框架突破了现有仅优化拓扑结构的局限。研究表明，经过优化的提示词和精心设计的搜索空间，使MAS设计的效果远超单一拓扑优化。Mass框架如算法1和图2所示，其设计直觉是“由局部到全局、由模块到流程”的渐进优化策略，通过分阶段优化，有效应对组合优化的复杂性，具体流程如下：

**1 模块级提示词优化**

在集成智能体系统之前，对各智能体实施精细化的提示词优化，确保每个智能体都拥有最优指令。首先，使用智能体自动提示优化对初始预测器进行预热，记为 a_{0}^{\ast}\leftarrow O_{D }(a_{0}) ，即由模块优化器 O 联合优化指令的示例。其次，继续优化最小智能体的各种拓扑，如由2个预测器和1个辩论器构成的基础辩论拓扑 a_{0}^{\ast}\leftarrow O_{D }(a_{i}|a_{0}^{\ast}) 。该策略既降低优化复杂度，又为后续扩展奠定基础。记录所有优化模块的验证性能，作为后续拓扑优化的参考依据。此阶段虽为预热过程，却能有效规避人工提示词在复杂系统中的级联负面影响。

**2 工作流拓扑优化**

在此阶段，聚焦于优化整体MAS拓扑，确定智能体间最有效的排列与连接方式。有效的拓扑仅占全部设计空间的一小部分。因此，该阶段目标是将高性能拓扑的本质提炼到剪枝空间内，提高工作流级拓扑搜索效率。具体做法：用增量影响力 I_{a_{i} } =\varepsilon(a_{i}^{\ast})/\varepsilon(a_{0}^{\ast}) 量化拓扑 a_{i} 相对于基线 a_{0} 的增益。

同时，遵循有影响力的维度伴随着更高的选择概率的直觉，如果  u>p_{a} ，则激活相应的拓扑维度 a ，其中  ∼U(0,1) 且 p_{a}=Softmax(I_{a}, ) 。引入基于规则的拓扑编排顺序[summarize, reflect, debate, aggregate]将不同的拓扑组合成一个统一的空间，从而实现通过约束工作流，降低优化复杂度。 并通过在预定义设计空间内集成拒绝采样，过滤掉停用或超出最大智能体数预算$ $的无效拓扑组合。

**3 全局提示词优化**

在确定最优拓扑后，实施全局提示优化 (W^{\ast}) =O_{D}(W_{c}^{\ast}) 。此阶段通过自适应调优，解决模块级优化可能存在的协同偏差问题。

## 3 实验

**模型**

模型：[Gemini 1.5](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=Gemini+1.5&zhida_source=entity)和Claude 3.5 Sonnet。

**评测数据**

1\. [Hendryck’s MATH](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=Hendryck%E2%80%99s+MATH&zhida_source=entity)，用于推理能力评测； 2）HotpotQA、MuSiQue、[2WikiMultiHopQA](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=2WikiMultiHopQA&zhida_source=entity)，来自LongBench，用于长文本理解； 3）[MBPP](https://zhida.zhihu.com/search?content_id=258263947&content_type=Article&match_order=1&q=MBPP&zhida_source=entity)、HumanEval以及LiveCodeBench（LCB），用于代码能力评测。

所有方法推理时的智能体数均少于10。

**基线方法**

  1. CoT：零样本提示下的链式思维推理； 
  2. CoT-SC：利用自洽性从多样化推理轨迹中选出最一致答案； 
  3. Self-Refine：反思型智能体，验证并自我改进预测结果； 
  4. 多智能体辩论：智能体为答案辩护并从其他智能体聚合信息； 
  5. ADAS：自动智能体设计框架，通过基于LLM的元智能体迭代提出新智能体； 
  6. AFlow：基于蒙特卡洛树搜索的自动工作流设计。



所有基线的最大智能体数量均限制为10。

**实验设置**

Mass集成了提示词优化器 MIPRO，通过贝叶斯智能体模型为每个智能体优化指令和示例。每个智能体最多自举3个示例，10个指令候选，共10轮。模型温度T设为0.7，最大输出token为4096，Softmax中t设为0.05，用于加大每个搜索维度选择概率的锐化。所有阶段均用同一LLM作为评估器和优化器。

**实验结果**

表1 Gemini 1.5 Pro和Gemini 1.5 Flash在评测集上的结果

表1显示Mass平均性能显著优于基线：Gemini 1.5 Pro/Flash分别达78.8%/74.3%，Claude 3.5结果见表4。ADAS因未优化提示词仅实现有限提升，AFlow在部分任务表现接近Mass但缺乏提示优化环节，导致MATH等任务表现欠佳。

图3 左：Mass在Gemini 1.5 Pro上8项评测任务各优化阶段的平均表现；右：在HotpotQA上对未剪枝拓扑优化（2TO）和无前置提示词优化（1PO）的消融对比。

优化阶段消融研究。图3显示各阶段贡献：模块级优化较单智能体提升6%，拓扑优化再增3%。剪枝策略与提示预热缺一不可。全局提示调优带来额外2%增益，证明智能体协同优化价值。

图4 Mass与自动智能体设计基线在DROP验证集每轮优化轨迹对比

成本效益分析。图4对比优化轨迹：Mass通过交错优化实现稳定提升，AFlow因MCTS特性波动较大，ADAS易陷入复杂拓扑陷阱。

图5 Mass 在MATH上的优化轨迹

最佳架构与设计准则。图5展示MATH任务优化路径：阶段1优选辩论拓扑，阶段2转向并行聚合，阶段3全局调优达成最优。由此提炼设计准则：1) 先优化单体后组网；2) 优选高效拓扑；3) 协同调优提升系统效能。

## 4 结论

通过系统性分析多智能体系统的设计空间，揭示了提示词优化的关键作用与高效拓扑结构的稀疏分布特征。基于此，提出了多阶段优化框架Mass，该框架在剪枝后的设计空间内实施提示词与拓扑结构的交错优化，高效生成高性能MAS系统。实验表明，经Mass优化的多智能体系统在多项任务中显著超越人工设计与自动优化基线。最终，通过分析Mass发现的最优系统配置，并提炼出指导未来LLM-MAS开发的核心设计准则，为构建高效智能协作系统提供理论依据与实践路径。

## 相关链接

  1. Multi-Agent Design: Optimizing Agents with Better Prompts and Topologies [https://arxiv.org/pdf/2502.02533](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2502.02533)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】12-PydanticAI关键技术分析](https://zhuanlan.zhihu.com/p/1899832254022263668)
