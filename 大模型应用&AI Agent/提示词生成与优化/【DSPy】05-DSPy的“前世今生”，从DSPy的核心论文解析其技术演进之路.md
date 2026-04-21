# 【DSPy】05-DSPy的“前世今生”，从DSPy的核心论文解析其技术演进之路

原文链接：https://zhuanlan.zhihu.com/p/707184607

---

​

目录

## 概述

随着大模型(LMs)的出现，研究人员能够在更高的抽象层次和更低的数据需求下构建自然语言处理(NLP)系统。这推动了“提示”技术和轻量级微调的快速发展，然而，现有的LMs应用通常使用硬编码的“Prompt模板”来实现。这种方法虽然普遍，但不能很好地泛化到不同的语言模型、领域数据，脆弱且不易扩展。因此具备良好的泛化能力，且能够将其集成到复杂任务的流水线中的新的提示语言模型的技术，成为人们探索的新方向。[DSPy](https://zhida.zhihu.com/search?content_id=245283333&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)应运而生。

DSPy（Declarative Self-improving Language Programs（in Python）），即声明式自改进语言程序，是一个对语言模型Prompt和权重进行算法优化的框架。**DSPy强调通过编程而非硬编码Prompt构建基于LLM的应用。**

本文通过对DSPy的核心论文进行简述，解析其技术演进之路。在后续的文章中，将会对DSPy的核心论文进行详细的解读。

## DSP，伊始之作

2022年12月份，DSP诞生之时，聚焦于如何充分的发挥语言模型（LM）和检索模型（RM）的潜力，在论文《Demonstrate-Search-Predict: Composing retrieval and language models for knowledge-intensive NLP》中提出了[DEMONSTRATE-SEARCH-PREDICT](https://zhida.zhihu.com/search?content_id=245283333&content_type=Article&match_order=1&q=DEMONSTRATE-SEARCH-PREDICT&zhida_source=entity)（DSP）框架。**DSP可以通过传递自然语言文本在语言模型和检索模型之间建立复杂的流水线，** 从而系统地分解问题，使LM和RM能够更可靠、更高效地解决知识密集型NLP任务。其核心思想是通过自然语言文本在两个模型间相互传递信息，逐步分解和解决问题。

DSP程序解决多跳问答任务过程图

DSP框架包括**三个阶段：DEMONSTRATE、SEARCH和PREDICT** 。在每个阶段，DSP框架提供了简单的可组合函数，允许开发者通过自然语言文本作为媒介，将不同的预训练模型组合成复杂的系统，来解决知识密集型NLP任务。三个阶段说明如下：

  * **DEMONSTRATE** ：自动从训练数据中选择示例，并为其添加中间查询和检索的注释。
  * **SEARCH** ：使用检索模型(RM)来从知识库中检索与输入问题相关的文本段落。
  * **PREDICT** ：利用DEMONSTRATE阶段中的示例和SEARCH阶段中检索到的段落，来生成问题的答案。



具体来说，DEMONSTRATE阶段自动从训练数据中选择示例，并为其添加中间查询和检索的注释。SEARCH阶段使用检索模型(RM)来从知识库中检索与输入问题相关的文本段落。PREDICT阶段利用DEMONSTRATE阶段中的示例和SEARCH阶段中检索到的段落，来生成问题的答案。

DSP框架的一个**关键特点** 是**模块化和可组合的函数** ，这些函数允许开发者以自然语言文本作为媒介，将不同的预训练模型组合成复杂的系统，来解决知识密集型NLP任务。这种模块化和可组合性使得开发者可以轻松地构建复杂的程序，而无需深入了解每个组件的内部工作原理。

另一个**关键特点** 是**自动化演示标注** 。DSP框架可以通过简单的程序自动为复杂的流水线生成演示，而不需要为每个转换步骤进行人工标注。这种自动化演示标注使得开发者可以更轻松地构建复杂的程序，而无需花费大量时间进行人工标注。

实验结果：在实验评估中取得了新的上下文学习结果，相对于如基础的[GPT-3.5](https://zhida.zhihu.com/search?content_id=245283333&content_type=Article&match_order=1&q=GPT-3.5&zhida_source=entity)语言模型、标准的检索-阅读流程以及self-ask流程，它的效果分别提高了37%至120%，8%至39%，以及80%至290%。

## DSPy，将声明式语言模型调用编译成自我改进流水线

随着大语言模型的爆火，提示技术快速发展，人们开始探索新的提示优化技术。2023年10月，斯坦福大学NLP团队发布了DSPy，并在论文《DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines》中介绍了其技术实现细节。

DSPy是一个对语言模型Prompt和权重进行算法优化的框架，旨在简化基于LLM的应用的复杂构建过程。**DSPy强调通过编程而非硬编码Prompt构建基于LLM的应用。** 其将构建大模型流水线的过程从操纵基于字符串的提示技术转变为接近编程的方式。DSPy的设计灵感来自于神经网络抽象的共识，其中包括两个关键概念：

  * 建立一种通用的机制，并通过模块化组合的方式实现。
  * 模型权重可以使用优化器进行训练，而不是手工调整。



DSPy工作流，来源于领英DSPy: The Future of Programming Language Models

DSPy主要包含**签名（Signatures）** 、**模块（Modules）** 和**优化器（Optimizers）** 三个组件。其创新之处在于将签名、模块和优化器结合起来使用。签名为语言模型提供指导，而优化器则使用签名和一个度量或评估系统（可能是语言模型作为评判标准）来进行实验，以确定更理想的提示文本和最佳的少量示例集。因此，使用DSPy，开发者只需关注任务本身，而不必纠结于Prompt工程的具体细节，可以极大地提高了开发效率。

  * **签名（Signatures）** ：**签名为语言模型提供指导。是DSPy模块输入/输出行为的声明性规范。** 其目的是提供对任务或子任务以及输入和输出类型的最基本描述。对于简单的情况，签名可以是简短的字符串，也可以包含多个输入/输出字段。其参数名定义了输入/输出的语义角色。
  * **模块（Modules）** ：**封装了特定的文本转换功能，如问答或文本摘要，并可以学习适应不同任务。** （1）每个内置模块都抽象了一种提示技术（如Chain of Thought或ReAct）。每个模块都关联一个自然语言签名，并内部实现了相应的Prompt流程。（2）使其能够通过迭代选择更好的示例来学习其期望的行为。（3）DSPy模块还可以组成任意的流水线，从而组合成更复杂的程序。
  * **优化器（Optimizers）** ：**使用签名和一个度量或评估系统（可能是语言模型作为评判标准）来进行实验，以确定更理想的提示文本和最佳的少量示例集。** 用于改进DSPy程序的质量或成本，选择最佳的提示或微调策略。



DSPy解决了开发LLM应用时由Prompt脆弱性带来的问题。其强调Prompt的整体系统设计，并通过模块化的设计，DSPy将复杂的任务分解为多个模块，每个模块都有明确的职责和接口，使得系统更易于维护和扩展。

实验结果表明，经过编译后，DSPy允许GPT-3.5和[llama2-13b-chat](https://zhida.zhihu.com/search?content_id=245283333&content_type=Article&match_order=1&q=llama2-13b-chat&zhida_source=entity)自我启动流水线，这些流水线在标准少样本提示和专家创建的示例方面表现优于标准少样本提示（平均分别超过25%和65%）和专家创建的示例（高达5-46%和16-40%）。

## DSPy断言，定义了语言模型流水线的“规矩”

2023年12月，在论文《DSPy Assertions: Computational Constraints for Self-Refining Language Model Pipelines》中定义了**LM断言，一种用于表达LM应满足的计算约束的编程构造** 。

**LM断言** 被定义为**程序元素** ，它**定义了在语言模型流水线执行过程中必须遵守的某些条件或规则。** 这些约束可确保流水线的行为符合开发人员指定的不变量或准则，从而提高流水线输出的可靠性、可预测性和正确性。

将LM断言集成到DSPy编程模型中，除了作为传统的运行时监视器，LM断言还能实现多种新颖的断言驱动优化，以改进LM程序。三种断言方式如下：

  * **断言驱动的回溯：** LM断言可以在推理时促进LM流水线中的自我优化。当约束失败时，允许流水线回溯并重试失败的模块。LM断言会提供重试尝试的反馈，它们会将错误输出和错误消息注入提示，以便自我优化输出。
  * **断言驱动的示例引导：** LM断言可以在编译时启用引导提示优化器。与DSPy中现有的自动提示优化器集成，它们可以生成更难的少样本示例，从而指导LM程序进行具有挑战性的步骤。
  * **反例引导：** 在提示优化和示例引导过程中，LM断言的另一个重要贡献是开发包含失败示例和修复错误痕迹的演示。当反例与引导的少样本示例混合时，LM将会更有可能避免同样的错误，而无需断言驱动的回溯。



LM断言包含两种类型：**（硬性）断言** 和**（软性）建议** ，分别用Assert和 Suggest表示。**硬性断言表示临界条件** ，当在最大重试次数后仍被违反时，会导致LM流水线停止，表明这是不可协商的违背了要求。而**建议表示期望但非必要的属性** ，其违背会触发自我优化过程，但超过最大重试次数并不会停止流水线。相反，流水线会继续执行下一个模块。

将断言集成到DSPy，通过设计和实现三种新的断言驱动优化，使得DSPy程序能够自我优化，并生成符合特定准则的输出。它简化了调试，让开发人员更清楚地了解复杂流水线中的LM行为。此外，通过将LM断言与DSPy中的提示优化器结合使用，能够引导生成更好的少样本示例和反例，使流水线更稳健和高效。

作者在四个不同文本生成案例研究中应用了LM Assertions，使用DSPy编程模型，并发现LM断言不仅提高了对施加规则的遵从性，还提高了下游任务的性能，遵从约束的次数最多提高了164%，生成的高质量响应最多提高了37%。

## 极端多标签分类的上下文学习

2024年1月，在论文《In-Context Learning for Extreme Multi-Label Classification》中提出了**[Infer-Retrieve-Rank](https://zhida.zhihu.com/search?content_id=245283333&content_type=Article&match_order=1&q=Infer-Retrieve-Rank&zhida_source=entity) (IReRa)**，**一个用于极端多标签分类（≥ 10,000类）的高效上下文学习程序** 。其**定义了LM和检索器之间的多步交互，** 有效地解决了具有大量类别的多标签分类问题。其创新点为该方案不需要微调，适用于新任务，减轻了提示工程，并且只需要少量标记的示例。

Infer-Retrieve-Rank (IReRa)的关键见解是，**如果LM在上下文中学习如何预测相关查询并解释检索结果，则可以使这种冷冻检索器更加灵活** 。

如上图，在给定输入情况下，第一个上下文学习模块会预测查询结果，并将其发送至冻结检索器。检索到的文档由第二个上下文模块重新排序（步骤1）。给定一个最小提示（步骤2）后，无需任何训练的Teacher LM引导生成示例，以优化少样本的Student LM（步骤3）。使用约50个标记输入进行优化可以产生最先进的结果，只需使用约20次Teacher调用和约1,500次Student调用。这些发现表明，prompt和pipeline engineering的未来不一定是脆弱的。模块化程序一旦优化，就可以作为高效的通用解决方案。

## 优化多阶段语言模型程序的指令和演示

**语言模型程序，即模块化语言模型（LM）调用的复杂流水线** ，其正在不断推进自然语言处理任务的发展，但它们需要设计对所有模块都有效的提示。针对多阶段复杂任务的处理，多阶段语言模型程序（multi-stages LM程序）（即多阶段流水线）更是对提示优化技术提出了更高的要求。

示例其为包含两个LM模块的LM程序。通过给定问答对和指标，优化器为每个阶段提出新的指令并引导生成新的示例（未显示）

2024年6月，在论文《Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs》中提出了**一种无需模块级标签或梯度即可最大化下游指标的优化方法。** 其创新点在于**将问题分解为每个模块的自由形式指令和少样本演示的优化，并引入了策略来构建任务相关的指令和跨模块的信用分配导航。** 并在此基础上，提出了**一种名为[MIPRO](https://zhida.zhihu.com/search?content_id=245283333&content_type=Article&match_order=1&q=MIPRO&zhida_source=entity)的新型优化器，以研究多阶段LM程序的提示优化**。

**MIPRO** 特点是**通过将信用分配任务与提案任务分离，使得LM可以专注于提案任务** 。MIPRO算法包括初始化、提案、更新和提取优化集四个步骤。

  1. 在**初始化** 阶段，算法根据提案超参数提出一组指令和任务示例，并对所有贝叶斯模型中的潜在变量进行均匀先验进行初始化。
  2. **提案阶段** 利用树状Parzen估计器的采样规则来提出部分指令和示例赋值。
  3. **更新阶段** 在随机选择的小批量**B** 样本进行评分，并据此更新模型对于参数质量的先验。
  4. 在最后的**提取优化集阶段** ，在整个训练集上评估试验中平均得分最高的Φ的候选参数。试验结束时，经过全面评估的参数化的最高得分将作为最佳分配返回。



本文将优化语言模型 (LM) 程序的问题形式化为搜索可能的提示，分析了LM程序优化的两个主要挑战：提案生成和优化期间的信用分配，随后提出三种提案生成策略和三种信用分配策略来解决这些挑战，并开展了不同任务的基准测试。研究发现，优化少量示例演示非常强大，但对于复杂的任务规范，指令优化也可能至关重要，同时优化两者通常会产生最佳效果。

在实验阶段，作者开展了六个任务（即六组数据集、指标和LM程序）来评估LM程序优化器效果。结果表明，优化器MIPRO（Multi-prompt Instruction Proposal Optimizer）在使用Llama3-8B在多个项目上都得到了不错的效果，且最高可提升13%的准确率。

## 总结

DSPy通过优化LM的Prompt和权重，实现了更少的提示、更高的分数以及更系统化的方法，极大的简化了构建LLM应用解决各种复杂任务的过程。DSPy解决了开发LLM应用时由Prompt脆弱性带来的问题。其基于Prompt的整体系统设计思路、模块化的设计，有助于确保系统的维护性和扩展性。

DSPy的出现，标志着提示技术从人工写作、硬编码“Prompt模板”等方式逐步演进为自动化和系统化的优化过程。相信，随着DSPy编程模型及其自动优化策略的引入，提示技术在未来有望继续推动NLP领域的创新和发展。

## 参考资料

  1. Omar Khattab, Keshav Santhanam, Xiang Lisa Li, et al. Demonstrate-Search-Predict: Composing retrieval and language models for knowledge-intensive NLP. arXivpreprint arXiv:2212.14024
  2. Omar Khattab, Arnav Singhvi, Paridhi Maheshwari, et al. DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines. arXivpreprint arXiv:2310.03714
  3. Arnav Singhvi, Manish Shetty, Shangyin Tan, et al. DSPy Assertions: Computational Constraints for Self-Refining Language Model Pipelines. arXivpreprint arXiv:2312.13382 2023.
  4. Krista Opsahl-Ong, Michael J Ryan, Josh Purtell, et al. Optimizing Instructions and Demonstrations for Multi-Stage Language Model Programs. arXivpreprint arXiv:2406.11695.
  5. D'Oosterlinck K, Khattab O, Remy F, et al. In-Context Learning for Extreme Multi-Label Classification[J]. arXiv preprint arXiv:2401.12178, 2024.
  6. GitHub：https://github.com/stanfordnlp/dspy



## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】04-RAG坦途已现！DSPy，将会革命性改变RAG系统的构建方式](https://zhuanlan.zhihu.com/p/706135629)  
> 下一篇：[【DSPy技术洞察】06-Prompt或许的新未来， DSPy使用从0到1快速上手](https://zhuanlan.zhihu.com/p/707925423)
