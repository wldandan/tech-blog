# 【DSPy】02-DSPy：将声明式语言模型调用编译为自我改进流水线

原文链接：https://zhuanlan.zhihu.com/p/705291734

---

​

目录

## 概述

随着大模型(LMs)的出现，研究人员能够在更高的抽象层次和更低的数据需求下构建自然语言处理(NLP)系统。这推动了“提示”技术和轻量级微调技术的快速发展，这些技术用于使LMs适应新任务、从LMs中引出系统性推理以及通过检索来源或工具增强LMs。然而，现有的LM管道通常使用硬编码的“提示模板”来实现，这些模板是通过试错发现的长字符串。这种方法虽然普遍，但可能脆弱且不可扩展。因为它不能很好地泛化到不同的管道、语言模型、数据领域或输入。

本文介绍了**[DSPy](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=DSPy&zhida_source=entity) ，这是一种新的编程模型，即[声明式自改进语言程序](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=%E5%A3%B0%E6%98%8E%E5%BC%8F%E8%87%AA%E6%94%B9%E8%BF%9B%E8%AF%AD%E8%A8%80%E7%A8%8B%E5%BA%8F&zhida_source=entity)。其强调编程而非Prompt，并将构建基于LM的管道从操作prompt转移到更贴近编程。**其用于构建由预训练大模型(LMs)和其他工具组成的人工智能系统。DSPy通过将大模型调用抽象成[文本转换图](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=%E6%96%87%E6%9C%AC%E8%BD%AC%E6%8D%A2%E5%9B%BE&zhida_source=entity)来优化它们的使用，这些图是命令式的计算图，其中语言模型是通过声明性模块被调用的。DSPy模块是参数化的，意味着它们可以通过创建和收集示例来学习如何应用提示、微调、增强和推理技术的组合。此外，设计了一个编译器来优化任何DSPy管道，以最大化给定的度量标准。

## 技术方案

### 概述

为了克服这些限制，本文提出了**DSPy，它将构建新的大模型管道的过程从操纵自由形式的字符串转变为更接近编程的方式——组合模块化操作符构建文本转换图。** 在这种方式中，编译器自动从程序生成优化的语言模型调用策略和提示。DSPy的设计灵感来自于神经网络抽象的共识，其中包括两个关键概念：(1) **许多通用层可以在任何复杂架构中模块化组合** ；(2) **模型权重可以使用优化器进行训练，而不是手工调整** 。

DSPy模型首先**将基于字符串的提示技术转换为带有自然语言类型签名的声明性模块** 。这些模块是任务自适应组件，类似于神经网络层，它们抽象了任何特定的文本转换，例如回答问题或总结文章。然后，**通过参数化每个模块，使其能够通过在管道内迭代自举有用的示例来学习其期望的行为。** 受[PyTorch](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=PyTorch&zhida_source=entity)抽象的直接启发，DSPy模块通过表达式定义的运行时计算图来使用。管道通过声明所需的模块和使用这些模块在任何逻辑控制流中逻辑连接模块来表达。

接着，本文开发了DSPy**编译器** ，它可以优化任何DSPy程序以提高质量或降低成本。**编译器的输入包括程序、一些可选标签的训练输入以及验证度量。** 编译器在输入上模拟程序的版本，并自举每个模块的示例跟踪以进行自我改进，使用它们构建有效的少次提示或微调小语言模型以实现管道的步骤。在DSPy中，优化是高度模块化的：它由teleprompters进行，这些是通用的优化策略，确定模块应如何从数据中学习。通过这种方式，编译器自动将声明性模块映射到提示、微调、推理和增强的高质量组合。

**核心概念：**

  * **签名（Signatures）** ：使用自然语言定义函数的输入和输出，而不是具体实现细节。
  * **模块（Modules）** ：封装了特定的文本转换功能，如问答或文本摘要，并可以学习适应不同任务。
  * **优化器（Teleprompters）：** 用于改进DSPy程序的质量或成本，通过选择最佳的提示或微调策略。



### 方法详解

DSPy通过两个案例研究展示了其有效性：**数学文字问题** 和**复杂问题回答** 。

### 数学文字问题

本文使用流行的[GSM8K](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=GSM8K&zhida_source=entity)数据集对小学数学问题进行评估（Cobbe等人，2021）。本文从官方训练集中抽取了200个和300个问题-答案对，分别用于训练和开发。本文的最终评估使用官方测试集中的1.3k个例子。为了避免在测试集上过拟合，本文在开发集上报告了广泛的比较。按照GSM8K上先前的工作，本文评估了LM输出中出现的最终数值的准确性。

对于这项任务，本文考虑了三个简单的DSPy程序：一步式Predict模块（vanilla）、两步式ChainOfThought模块（CoT），以及最后是多阶段ComparerOfThoughts模块（ThoughtReflection）。这些程序完全由以下代码定义：
    
    
    class ThoughtReflection(dspy.Module):
        def __init__(self, num_attempts):
            self.predict = dspy.ChainOfThought("question -> answer ", n=num_attempts)
            self.compare = dspy.MultiChainComparison(' question -> answer ', M=num_attempts)
        def forward(self, question):
            completions = self.predict(question=question).completions
            return self.compare(question=question, completions=completions)
    # GSM8K程序‘reflection’
    reflection = ThoughtReflection(num_attempts=5)
    # GSM8K程序‘vanilla’
    vanilla = dspy.Predict("question -> answer")
    # GSM8K程序‘CoT’
    CoT = dspy.ChainOfThought("question -> answer ")

在reflection中，从语言模型中抽取了五个推理链及其答案，并通过内置的MultiChainComparison模块并行比较它们，以生成新的综合答案。

DSPy程序可以编译成新的优化程序。本文评估了程序的零样本（无编译）以及多种编译策略。例如，LabeledFewShot编译如下：
    
    
    fewshot = dspy.LabeledFewShot(k=8).compile(program, trainset=trainset)

这里，program可以是任何DSPy模块。编译器从训练集中随机抽取8个示例，用于训练示例和签名共有的字段，例如问题和答案。

接下来，本文还考虑了使用随机搜索来引导少样本示例：

这个过程会为训练集中的例子生成示例链，并优化选择这些示例的过程，使用随机搜索，将选择示例视为一个需要优化的参数。
    
    
    # 使用随机搜索从训练集中引导示例，优化选择示例的过程
    tp = BootstrapFewShotWithRandomSearch(metric=gsm8k_accuracy)
    bootstrap = tp.compile(program, trainset=trainset, valset=devset)

此外，本文可以在DSPy中嵌套自举过程。具体来说，本文可以使用优化后的自举程序本身来进一步引导另一个程序。这在原始零样本程序表现相对较差时强相关。
    
    
    # 使用先前优化的程序作为教师，进一步自举另一个程序
    bootstrap2 = tp.compile(program, teacher=bootstrap, trainset=trainset, valset=devset)

最后，本文还考虑了将这些自举程序集成在一起：

GSM8K包含了人类推理链。在上面的trainset中，本文没有包括这些推理链。本文还评估了包括人类推理链的trainset，这通过在trainset中添加人类的推理字符串来扩展例子。这两个数据集可以作为trainset参数的值互换使用。本文注意到，编译通常只需要几分钟（或几十分钟），即使是更昂贵的设置也只需要运行程序几千次（例如，在150-300个验证例子上进行10-20次试验），并且可以并行进行。
    
    
    # 使用majority voting集成从自助编译运行中选出的前7个候选程序
    ensemble = Ensemble(reduce_fn=dspy.majority).compile(bootstrap.programs[:7])

### 复杂问题回答

在这项研究中，本文探索了使用[HotPotQA](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=HotPotQA&zhida_source=entity)数据集的多跳问题回答任务，在开放域的“fullwiki”设置中。对于检索，使用了HotPotQA官方2017年“摘要”转储的搜索索引。搜索是通过[ColBERTv2](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=ColBERTv2&zhida_source=entity)检索器进行的。HotPotQA的测试集是隐藏的，因此本文保留官方验证集用于本文的测试，并抽取了1000个例子用于此。本文在训练集（和验证集）中只保留了原始数据集中标记为“困难”的例子，这与官方验证集和测试集的指定相符。在训练和报告开发结果时，本文分别抽取了200个和300个例子。

最简单的基线是上文中在GSM8K上使用的“question -> answer”签名的vanilla程序，它在适当编译时适用于此任务（以及许多其他任务）。

本文将看到，这个程序在HotPotQA上并不出色，这激发了本文评估两个多跳程序的动机。

为此，本文首先测试了ReAct，这是一个用于工具使用的多步骤代理，它在DSPy中作为内置模块实现。在最简单的情况下，可以像下面这样在DSPy中声明特定签名的ReAct模块：
    
    
    react = dspy . ReAct (" question -> answer ", tools =[ dspy . Retrieve ( k =1) ] , max_iters =5)

本文还测试了以下自定义程序，它模拟了Baleen和IRRR中的信息流，并且与IRCoT有相似之处：
    
    
    class BasicMultiHop(dspy.Module):
        def __init__(self, passages_per_hop):
            self.retrieve = dspy.Retrieve(k=passages_per_hop)
            self.generate_query = dspy.ChainOfThought("context, question -> search_query")
            self.generate_answer = dspy.ChainOfThought("context, question -> answer")
        def forward(self, question):
            context = []
            for hop in range(2):
                query = self.generate_query(context=context, question=question).search_query
                context += self.retrieve(query).passages
            return self.generate_answer(context=context, question=question)
    multihop = BasicMultiHop(passages_per_hop=3)
    multihop_t5 = dspy.BootstrapFinetune(metric=answer_exact_match).compile(
        program, teacher=bootstrap, trainset=trainset, target='t5-large'
    )

本文还考虑了本文的teleprompters的两种组合。对于ReAct，本文考虑从ReAct程序的早期自举开始使用BootstrapFewShotWithRandomSearch进行自举。对于简单的多跳程序，本文还考虑了从该程序的早期自举开始使用[T5-Large](https://zhida.zhihu.com/search?content_id=244862611&content_type=Article&match_order=1&q=T5-Large&zhida_source=entity)进行微调。

## 实验结果概述

### 数学文字问题

表1中总结了结果，包括开发集的结果以及在测试集上对每种方法有前景的代表的评估。首先，vanilla程序的结果显示，GPT-3.5和llama2-13b-chat在没有使用推理链的情况下，直接预测数学文字问题的答案存在困难。然而，通过使用bootstrap编译和迭代这个过程，可以显著提高性能。CoT程序在专家人类推理链的辅助下表现更好，但即使没有这些推理链，通过bootstrap也能匹配或超越这种性能。ThoughtReflection程序虽然代码更长，但表现最佳。通过组合DSPy模块和teleprompters，不同LMs的准确率从4-20%提高到49-88%。

与现有文献中的其他方法相比，即使没有使用人类推理链，也能与使用更大模型的结果竞争。这表明DSPy通过自动编译程序，减少了对手工制作提示的依赖，提高了系统的模块化和可复现性。

### 复杂问题回答

表2总结了结果。与vanilla少样本提示相比，思维链和检索增强生成（CoT RAG）程序可以通过DSPy自举显著提高答案的精确匹配（EM）。然而，这完全依赖于ColBERTv2检索器直接从原始问题中找到相关段落，限制了其段落召回。这在react和multihop程序中得到了解决，它们将为检索器在多个迭代的“跳数”中生成查询。确实，总的来说，一个简单的多跳程序表现最佳，并且一般而言，自举再次被证明非常有效，可以提高其相对于其少样本变体的质量，无论是对于哪种LM。

特别是，我们可以看到自举（和/或自举×2）可以超越多跳的少样本提示（对于multihop）和专家人类推理（对于react；从Yao等人（2022）稍微调整以适应我们的检索设置）。也许最重要的是，我们可以通过简单地编译我们的程序，使llama2-13b-chat与GPT-3.5竞争。

为了评估DSPy的微调能力，本文还评估了上述定义的编译器multihop t5，它产生了一个T5-Large（770M参数）模型。该程序在开发集上的得分为39.3%的答案EM和46.0%的段落准确率，仅使用了200个标记输入和800个未标记问题。对于编译，本文使用了一个由两个multihop和llama2-13b-chat组成的集成（联合）作为教师程序。考虑到其极小的尺寸和本地可用性，与像GPT-3.5这样的专有LM相比，使用T5-Large的编译程序将带来几个数量级的低成本推理。

## 总结

本文介绍了DSPy，这是用于设计使用预训练语言模型（LMs）和其他工具的人工智能系统的新编程模型。本文展示了这个抽象中引入的三个新概念（DSPy签名、模块和teleprompters），并在两个非常不同的案例研究中表明，它支持使用相对较小的LMs快速开发高效的系统。作者已经维护了这个框架的开源版本将近一年。在这段时间里，作者见证了许多程序通过DSPy编译成高质量的系统，涵盖了从信息提取到低资源合成数据生成等任务。由于篇幅和本文合理范围的考虑，我们将在后续工作中报告这些任务在受控实验条件下的表现。尽管上下文学习在过去2-3年的LM研究中证明了其变革性，本文认为，这种新兴范式中真正的表达能力在于构建复杂的文本转换图，其中可组合的模块和优化器（teleprompters）以更系统和可靠的方式结合在一起，利用LMs。

## 参考资料

  1. Omar Khattab, Arnav Singhvi, Paridhi Maheshwari, Zhiyuan Zhang, Keshav Santhanam, Sri Vardhamanan, Saiful Haq, Ashutosh Sharma, Thomas T. Joshi, Hanna Moazam, Heather Miller, Matei Zaharia, Christopher Potts. DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines arXiv:2310.03714
  2. GitHub：[https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)



## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】01-Prompt工程或成为过去时，DSPy将会是拯救Prompt思维的利器](https://zhuanlan.zhihu.com/p/705107414)  
> 下一篇：[【DSPy技术洞察】03-多标签分类的上下文学习](https://zhuanlan.zhihu.com/p/706118311)
