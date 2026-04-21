# 【单Agent框架】06-KG-Agent：一种基于知识图谱实现复杂推理的高效自治智能体框架

原文链接：https://zhuanlan.zhihu.com/p/692978901

---

​

目录

## 简介

本篇论文《KG-Agent: An Efficient Autonomous Agent Framework for Complex Reasoning over Knowledge Graph》提出了一种名为KG-Agent的自主智能体框架，以提高大型语言模型（LLM）对[知识图谱](https://zhida.zhihu.com/search?content_id=242125872&content_type=Article&match_order=1&q=%E7%9F%A5%E8%AF%86%E5%9B%BE%E8%B0%B1&zhida_source=entity)（KG）的推理能力。KG-Agent 集成了 LLM、多功能工具箱、基于 KG 的执行器和知识储存器，并使用迭代机制自主选择工具并更新内存以对 KG 进行推理。作者利用程序语言在KG上制定[多跳推理](https://zhida.zhihu.com/search?content_id=242125872&content_type=Article&match_order=1&q=%E5%A4%9A%E8%B7%B3%E6%8E%A8%E7%90%86&zhida_source=entity)过程，并合成基于代码的指令数据集来微调基础LLM。

实验结果表明，即使只有10K个样本进行调整，KG-Agent 也优于使用更大 LLM 或更多数据的最先进的方法。所提出的框架是第一个使用相对较小的 LLM 支持对 KG 进行复杂推理的框架。

## 背景介绍

尽管大型语言模型（LLM）在多种NLP任务下都表现优异，但仅基于其参数知识解决复杂任务的能力很有限，例如多跳推理和知识密集型任务。

知识图谱（KG）中以结构存储着大量知识三元组，目前许多研究都将其用于LLM的外部补充知识，可由于KG的数据量庞大与其格式化结构，LLM很难有效利用其信息。

基于 LLM 的 KG 推理的现有方法包括检索增强和协同增强，前一种方法检索并序列化与任务相关的三元组，作为LLM prompt的一部分，后一种方法设计了KG和LLM之间的信息交互机制，以迭代地找到问题的解决方案。

然而上述两种方法存在局限性：检索增强方法会丢失原始KG中的结构化信息，并可能检索冗余知识，从而限制LLM的理解；协同增强方法中，LLM 和 KG 之间的信息交互机制往往是预先定义的，无法灵活地适应各种复杂任务，且大多数方法依赖强大的LLM APIs(如：ChatGPT 和 GPT-4)，即使对LLM进行蒸馏，也仅限于特殊的任务设置或能力水平，可能不太适合指导这些较弱的模型。

## 技术方案

为了解决上述提到的限制和问题，本篇论文提出一个名为KG-Agent的自主智能体框架，基于LLM，应用于需要对任何KG进行推理的各种复杂任务。

此方案的目标分为：

  1. 设计自主推理方法，能够在推理过程中主动做出决策，无需人工协助；
  2. 使相对较小的模型（例如7B LLM）能够有效地执行复杂的推理，而不依赖于闭源的LLM APIs。



为了实现目标，贡献了三个主要的技术点：

  1. 通过策划一个多功能工具箱来扩展LLM处理结构化数据的能力，使LLM能够对KG数据和中间结果执行离散或高级操作。
  2. 利用现有的KG推理数据集来合成基于代码的指令数据来微调LLM，首先根据KG上的推理链生成程序，然后合成指令数据。
  3. 提出了一种基于工具选择和存储器更新的自主迭代机制，该机制集成了调谐的LLM，多功能工具箱，基于KG的执行器和知识存储器，用于在KG上进行自主推理。



KG-Agent架构

### KG工具箱

基于现有的工作，KG推理通常需要三种基础的操作：KG信息提取、基于问题语义的无关信息过滤、操作提取的信息。作者根据上述操作设计了三种工具。

  * **提取工具** ：共设计了五个工具，用于从KG中提取相关的信息，分别用于提取关系(get_relation)、提取头/尾实体(get_head_entity/get_tail_entity)、基于特定类型或约束提取实体(get_entity_by_type/get_entity_by_constraint)。
  * **逻辑工具** ：共设计了五个工具，用于对提取出的KG信息进行基本操作，分别用于实体计数(count)、实体集交集(intersect)、实体集并集(union)、条件验证(judge)、以当前实体作为最终回答(end)。
  * **语义工具** ：利用预训练模型实现了两个特定的函数，用于拓展对KG的基本操作，分别用于关系检索(retrieve_relation)、实体消歧(disambiguate_entity)。



### KG-Agent指令微调

为了实现Agent的核心推理能力，作者构建了一个高质量的指令数据集来微调一个轻量的LLM([LLaMA2-7B](https://zhida.zhihu.com/search?content_id=242125872&content_type=Article&match_order=1&q=LLaMA2-7B&zhida_source=entity))。

  1. **KG推理程序生成** ：作者没有使用蒸馏的方式提取闭源LLM的推理能力，而是使用现有的[KGQA数据集](https://zhida.zhihu.com/search?content_id=242125872&content_type=Article&match_order=1&q=KGQA%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)得到KG的推理流程。
  2. **KG推理指令合成** ：得到KG上与问题相关的推理程序后，作者进一步将其用于监督微调的指令数据集合成。



### KG自主推理

在上述准备工作结束后，作者设计了一个能够自主对KG进行多步推理的agent框架，主要包含四个组件：LLM、多功能工具箱、基于KG的执行器、知识记忆存储器。

  * **知识记忆存储器** ：用于记录上下文和当前有用的信息。
  * **工具选择规划器** ：基于当前的知识记忆，LLM将会在每一步选择一个与KG交互的工具。具体来说，当前知识记忆都将通过相应的prompt模板格式化为输入，然后通过从输入中选择工具及其参数来生成函数调用。
  * **记忆更新执行器** ：在函数调用执行后，知识记忆将会进行相应地更新。首先将当前函数调用添加到历史推理程序中。其次，如果调用的工具从KG中获取了新信息，执行器便将其添加到KG信息中。
  * **迭代自主的KG-Agent** ：KG-Agent框架自主迭代上述工具选择和记忆更新过程来执行逐步推理，其中知识记忆用于维护来自 KG的访问信息。Agent的多轮决策过程就像沿着关系链在KG上游走。一旦到达答案实体，Agent就会自动停止迭代过程。



## 实验结果

为了验证有效性，在域内和域外任务上对KG-Agent进行了评估，包括基于KG的问题回答（KGQA）和开放域问题回答（ODQA）。

使用更少的训练数据（10K样本）来调整较小的LLM（LLaMA-7B），此方法可以在域内数据集上优于竞争的基于LLM的基线（例如使用约36%和23%的原始训练集量，同时在CWQ和[GrailQA](https://zhida.zhihu.com/search?content_id=242125872&content_type=Article&match_order=1&q=GrailQA&zhida_source=entity)上分别获得7.5%和2.7%的F1相对改善）。在域外数据集上，KG-Agent的zero-shot性能优于竞争的全数据监督微调模型（例如在WQ-Freebase和TQ-Wiki上分别相对提高了9.7%和8.5%的精度）。

如表2和表3所示，分别显示了基于Freebase和Wikidata的域内数据集上的结果。

首先，在WebQSP和[KQA Pro](https://zhida.zhihu.com/search?content_id=242125872&content_type=Article&match_order=1&q=KQA+Pro&zhida_source=entity)上，基于LM（Language Model）的seq2seq生成方法可以获得比基于子图的推理方法更好的F1分数。实验表明，与传统的基于子图的推理方法相比，LM生成的SPARQL查询可以获得更完整的答案集，并且结构化查询可以更好地支持一些复杂的操作（例如，最大值，计数）。

其次，虽然LLM很强大，但直接使用Davinci-003、ChatGPT甚至GPT-4与WebQSP、GrailQA和KQA Pro中最好的微调方法相比仍然有很大的性能差距，这表明仅靠LLM回答复杂问题是困难的。

最后，KG-Agent在指示对混合数据进行调整后，在所有数据集中都大大优于所有其他竞争基线。通过不同数据集之间的相互增强，此方法在WebQSP、CWQ和Grailqa上分别实现了1.7%、7.5%和2.7%的F1提升。受益于自主推理机制，此方法可以在两个KG上进行推理，并在所有数据集上获得一致的改进。

在指令调整之后，直接评估了KG-Agent在域外数据集上的zero-shot性能。如表4所示，尽管使用完整数据进行了微调，但小型预训练语言模型（例如T5和BART）无法有效回答这些事实问题。

由于参数规模较大，Davinci-003和ChatGPT在NQ和TQ上表现良好，它们是基于维基百科构建的，它们可能已经在维基百科上进行了预训练。但是，它们在基于Freebase KG构建的WQ上表现不佳。

相比之下，KG-Agent只需要学习如何与KG交互，而不是记忆特定的知识。因此，它可以在zero-shot设置中利用外部KG，并且与经过微调的预训练语言模型相比，可以实现一致的改进。

## 总结

这篇论文提出了一个自主智能体框架，以协同LLM和KG来执行KG上的复杂推理，即KG-Agent。

在论文的方法中，作者首先为KG策划了一个工具箱，由三种类型的工具组成，以支持在KG上推理时的典型操作；然后，开发了一种基于工具选择和存储器更新的自主迭代机制，该机制集成了LLM，多功能工具箱，基于KG的执行器和知识存储器，用于在KG上进行推理；接下来，利用现有的KGQA数据集来合成基于代码的指令调整数据集。

最后，在只有10K调优样本的情况下，实现了依赖于较小的7B LLM的自治智能体，其性能大多优于基于全数据调优或较大LLM的最先进的基线。

在未来的工作中，作者将考虑扩展框架来处理更多类型的结构化数据，例如数据库和表。

## 参考资料

  1. Jiang J, Zhou K, Zhao W X, et al. KG-Agent: An Efficient Autonomous Agent Framework for Complex Reasoning over Knowledge Graph[J]. arXiv preprint arXiv:2402.11163, 2024



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【单Agent洞察】05-XAgent：采用双循环运转机制，自主解决复杂任务的通用智能体](https://zhuanlan.zhihu.com/p/681136067)  
> 下一篇：[【单Agent洞察】07-具有超长语境要点记忆功能的人类启发式阅读智能体](https://zhuanlan.zhihu.com/p/699566756)
