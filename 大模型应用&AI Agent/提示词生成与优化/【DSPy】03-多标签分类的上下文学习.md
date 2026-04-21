# 【DSPy】03-多标签分类的上下文学习

原文链接：https://zhuanlan.zhihu.com/p/706118311

---

​

目录

## 简介

本篇轮文《In-Context Learning for [Extreme Multi-Label Classification](https://zhida.zhihu.com/search?content_id=245046182&content_type=Article&match_order=1&q=Extreme+Multi-Label+Classification&zhida_source=entity)》提出了一个名为**[Infer-Retrieve-Rank](https://zhida.zhihu.com/search?content_id=245046182&content_type=Article&match_order=1&q=Infer-Retrieve-Rank&zhida_source=entity) (IReRa)的通用程序**，它**定义了LM和检索器之间的多步交互，** 有效地解决了具有大量类别的多标签分类问题。作者使用**[DSPy](https://zhida.zhihu.com/search?content_id=245046182&content_type=Article&match_order=1&q=DSPy&zhida_source=entity) 编程模型**实现了该程序，该模型**以声明的方式指定上下文系统，并使用DSPy优化器通过引导几十个少数例子来针对特定数据集进行调整。**

这个主要极端分类程序分别针对每个任务进行优化，在三个基准测（use,Tech,TechWolf）获得了最先进的结果，将相同的程序应用于具有截然不同特征的基准测试，并获得具有竞争力的性能（[BioDEX](https://zhida.zhihu.com/search?content_id=245046182&content_type=Article&match_order=1&q=BioDEX&zhida_source=entity)）。与以前的工作不同，作者提出的解决方案**不需要微调，很容易适用于新任务，减轻了提示工程，并且只需要几十个标记的示例** 。

## 背景介绍

由于语言模型（LMs）可能缺乏关于精确类别或如何分配这些类别的先验知识，而且在提示中展示每个类别通常是不可行的，并且可用类的绝对数量（通常超过10,000个）通常意味着即使在提示符中演示每个类也是不可行的。因此极端的多标签分类（XMC）任务很难单独使用上下文学习来解决。

为了解决这个问题，最近的一些研究在推理时进行多个LM调用，而另一些则促使LMs生成用于微调的合成数据，这些方法可以配置得很好，但它们都有手动“旋钮”，如提示和其他超参数，使得将它们应用于新的数据集、度量或LMs具有挑战性。

## 技术方案

为了应对以上问题和挑战，本文展示了使用DSPy编程模型，编写的简单程序支持强大的、高度通用的XMC任务**。DSPy允许单独指定方法的模块化程序，以及它应该如何针对不同的数据集进行优化。** 作者们为XMC任务提出了一个简单的上下文程序，将程序称为Infer-Retrieve-Rank (IReRa)，如下图Step1所示：

首先，一个LM处理输入文档并猜测一组适用的术语（Infer）。

然后，检索器将每个预测项与实际标签空间（Retrieve）相关联。

最后，使用LM对检索到的标签进行重新排序（Rerank）。至关重要的是，本文使用冷冻的检索器和冷冻的LM。

Infer-Retrieve-Rank (IReRa)的关键见解是，**如果LM在上下文中学习如何预测相关查询并解释检索结果，则可以使这种冷冻检索器更加灵活** 。

底层的LM、检索器和提示被认为是IReRa程序的超参数，可以自动调整或轻松配置。仅使用10个未标记的训练输入和≈50个标记的验证示例，就能达到一流的性能，本文作者使用具有最小种子提示（seed-prompt）的zero-shot teacher LM为两个LM组件引导一个few-shot提示，即可实现prompt优化，而不是迭代提示工程来提高性能，如上图Step2所示。

DSPy的编译抽象可以很好地处理这个问题，**它采用已经定义的程序逻辑，用teacher LM对其进行实例化，处理未标记的训练样本，为每个程序步骤生成zero-shot标签，并根据验证性能选择最佳标签放入few-shot提示中** 。因为程序是由两个上下文内模块组成，所以作者建议按顺序引导它们，如上图Step3所示。

### Infer-Retrieve-Rank (IReRa)

Infer-Retrieve-Rank (IReRa)的程序如下方代码段所示（为简洁起见，对其进行了少量修改），首先，使用LM来预测给定输入的查询（Infer），检索器基于与查询的最大余弦嵌入相似性输出所有标签的排名（Retrieve），排名Top标签由另一个LM重新排名（Rerank）。

### Seed-prompt

要将Infer-Retrieve-Rank (IReRa)应用于数据集，最小的Seed-prompt需要定义每个上下文内模块的行为。下方代码段包含Infer模块的提示，该模块位于BioDEX数据集，使用DSPy Signature抽象化整齐地组织。

Seed-prompt在文档字符串中定义任务说明，并在输入和输出字段中定义说明和格式信息。Signature（签名）作为零触发和少触发提示的框架。

下方代码段给出了BioDEX Rank模块的提示。

作者对所有三个职位空缺数据集使用相同的提示，它们分别在Infer和Rank模块如下方代码段所示。请注意提示如何共享其大部分内容：适应Infer-Retrieve-Rank (IReRa)就像简洁地描述输入和输出字段一样简单。

## 实验结果

实验过程中，作者用[Llama-2-7b-chat](https://zhida.zhihu.com/search?content_id=245046182&content_type=Article&match_order=1&q=Llama-2-7b-chat&zhida_source=entity)模型实例化Infer模块，而用于自举的teacher模型是[GPT-3.5](https://zhida.zhihu.com/search?content_id=245046182&content_type=Article&match_order=1&q=GPT-3.5&zhida_source=entity)，Rank模块由GPT-4模型实例化和引导。而调整Infer-Retrieve-Rank (IReRa) 以适应新的数据集可以很简单，步骤如下：（1）写入新的zero-shot提示（2）配置使用哪些LMs（3）运行优化程序。

作者分别针对4个XMC数据集优化该程序：一个数据集涉及提取和编码生物医学文献中表达的不良药物事件和三个数据集涉及标记职位空缺片段及其表达的所需能力。

实验显示，程序在职位空缺数据集上获得了最先进的结果，并在更难的生物医学任务上获得了有意义的牵引力——**无需微调，无需提示工程，只需使用大约50个标记的例子** 。由此可知，优化是跨任务性能的一致驱动因素。

## 总结

本文作者提出了一个用于极端多标签分类的通用程序Infer-Retrieve-Rank (IReRa)，该程序使用一个冷冻检索器和两个上下文学习模块在三个基准上实现了最先进的结果。这些发现表明，prompt和pipeline engineering的未来不一定是脆弱的。模块化程序一旦优化，就可以作为高效的通用解决方案。

## 参考资料

  1. D'Oosterlinck K, Khattab O, Remy F, et al. In-Context Learning for Extreme Multi-Label Classification[J]. arXiv preprint arXiv:2401.12178, 2024.



## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】02-DSPy：将声明式语言模型调用编译为自我改进流水线](https://zhuanlan.zhihu.com/p/705291734)  
> 下一篇：[【DSPy技术洞察】04-RAG坦途已现！DSPy，将会革命性改变RAG系统的构建方式](https://zhuanlan.zhihu.com/p/706135629)
