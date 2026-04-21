# 【RAG】02-RAP：针对多模态LLM Agent的上下文记忆检索增强规划

原文链接：https://zhuanlan.zhihu.com/p/700207538

---

​

目录

随着基于LLM的Agent进展， [大语言模型](https://zhida.zhihu.com/search?content_id=243732149&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLMs）现在可以作为Agent部署在包括：机器人、游戏和API集成在内的日益复杂的决策应用中。然而，在当前决策过程中反映过去经验，这一人类的固有行为，仍然面临重大挑战。

为了解决这个问题，在论文《RAP: Retrieval-Augmented Planning with Contextual Memory for Multimodal LLM Agents》中，作者们提出了[检索增强规划](https://zhida.zhihu.com/search?content_id=243732149&content_type=Article&match_order=1&q=%E6%A3%80%E7%B4%A2%E5%A2%9E%E5%BC%BA%E8%A7%84%E5%88%92&zhida_source=entity)（Retrieval-Augmented Planning，RAP）框架，该框架旨在动态利用与当前情境和背景相关的过去经验，从而增强Agent的规划能力。

RAP的独特之处在于其多功能性：它在纯文本和多模态环境中都表现出色，使其适用于广泛的任务。实证评估表明，RAP在文本场景中达到了最先进的性能，并显著增强了多模态LLM Agent在实体任务中的表现。这些结果突显了RAP在提升LLM Agent在复杂现实应用中的功能和适用性的潜力。

## 背景介绍

最近的研究揭示了大语言模型（LLMs）作为Agent的高推理能力，表明其在各种领域如决策任务和机器人控制中的潜在应用。先前的工作如[ReAct](https://zhida.zhihu.com/search?content_id=243732149&content_type=Article&match_order=1&q=ReAct&zhida_source=entity)表明作为语言Agent，LLMs可以通过迭代执行Action和推理生成准确的Action。

与此同时，随着LLMs的快速发展，检索增强生成（RAG）已成为提高LLMs生成能力的流行技术。这种方法将外部知识融入生成过程中，从而丰富生成内容的背景和准确性。

**然而，将外部记忆集成以增强LLM Agent的规划，特别是在多样化环境中，面临着巨大挑战。** 现有工作如Reflexion，分析失败案例，和[ExpeL](https://zhida.zhihu.com/search?content_id=243732149&content_type=Article&match_order=1&q=ExpeL&zhida_source=entity)，提取见解并增强语言Agent的学习，在利用复杂环境中的综合过去信息方面有所欠缺。这凸显了一个关键的差距：缺乏一个综合框架来在LLM Agent规划中利用过去经验，限制了其在复杂现实场景中的适应性和应用性。

在本文中，作者们**引入了一个新的框架，检索增强规划（RAP），它体现了人类利用过去经验为当前任务服务的关键能力，并将其应用于LLM Agent** 。作者们的方法包括将过去的经验存储在记忆中，根据与当前情境的相似性适当地检索它们，并通过上下文学习生成后续Action，从而增强语言Agent的决策能力。

该框架的核心是LLMs从各种抽象模式中进行类比的能力。利用这一能力，作者们的记忆存储了每次经验的上下文和动作-观察轨迹。该方法有效地帮助从任务约束内的记忆示例中得出正确的Action。此外，通过在记忆中存储多模态信息并在检索过去经验时考虑这些信息，作者们的方法灵活地将多模态信息与LLMs和视觉-语言模型（VLMs）分开使用。

因此，作者们的方法证明了在文本和多模态环境中语言Agent进行决策和机器人任务时，记忆利用的有效性。具体来说，RAP在ALFWorld、Webshop、[Franka Kitchen](https://zhida.zhihu.com/search?content_id=243732149&content_type=Article&match_order=1&q=Franka+Kitchen&zhida_source=entity)和Meta World基准测试中分别比ReAct提高了33.6%、13.0%、18.2%和12.7%的性能。

总结一下，本篇论文的贡献如下：

  * 提出了RAP，一个增强LLM Agent规划能力的新框架。它通过存储过去的经验并根据当前情境智能地检索这些经验，战略性地丰富了决策过程。
  * RAP不仅适用于文本环境，还适用于多模态实体任务，标志着在利用记忆检索技术为多模态Agent服务方面的开创性努力，据作者们所知，这是该领域的首次尝试。
  * 作者们在文本和多模态基准测试中实证验证了RAP的有效性。RAP在这两种环境中均表现出显著的改进，超过了之前的最先进方法。



## 技术方案

### 总览

作者们的框架存储过去的经验，并根据当前情境进行检索。左图：在ALFWorld上的评估过程。ICL表示上下文学习。右图：在Franka Kitchen上的评估过程。

### 主要组成部分

RAP核心组件

  
RAP框架由以下四个核心组件组成：

**记忆模块（Memory）**

    * 存储过去成功的任务执行日志，包括任务信息、总体计划、动作-观察序列等。
    * 这些日志帮助Agent在面临类似任务时检索相关的经验。



**推理模块（Reasoner）**

    * 基于当前任务和环境生成总体计划和行动计划。
    * 根据当前任务状态，动态生成Action或行动计划。
    * 生成检索关键字，用于从记忆中检索相关的经验。



**检索模块（Retriever）**

    * 通过计算当前状态与存储的经验日志之间的相似性来检索最相关的记忆。
    * 相似性评分基于任务相似性、总体计划一致性和检索关键字的匹配度。
    * 检索结果用于指导Agent的后续行动。



**执行模块（Executor）**

    * 接收检索到的过去经验，并通过上下文学习生成下一步的动作。
    * 利用与当前情境相关的过去经验，做出准确的决策。



  
Retriever根据检索关键字，计算与Memory的相似性，在动作和观察之间动态切换。此图说明了基于观察计算相似性的过程。右图：Executor从Memory中接收相关经验，并在prompt中利用它们。

## 实验结果

为了验证框架在各种环境中的有效性，论文作者在四个基准测试中进行了评估。这些基准测试包括文本基础的多步骤任务ALFWorld和Webshop，以及包含文本和图像的机器人任务FrankaKitchen和Meta-World。  


下图5展示了ReAct和RAP在三次试验中的成功率和得分改进的比较，两种方法均使用[GPT-3.5](https://zhida.zhihu.com/search?content_id=243732149&content_type=Article&match_order=1&q=GPT-3.5&zhida_source=entity)。结果表明，作者们的方法在成功率和得分上都有显著的提升，表明来自其他任务的成功经验得到了有效利用。  


使用两个视觉语言模型LLaVA和CogVLM，并分别在有和没有论文作者提出的RAP方法的情况下进行了评估。**结果表明，带有RAP的方法显著提升了多模态LLM Agent在执行实体任务时的性能。**

**

**

## 总结

本篇论文提出了检索增强规划（RAP）框架，它通过存储过去的经验，从多模态信息（如文本和图像）中提取相关经验，并指导后续行动。论文中提到的框架在各种大型语言模型和四个不同的Agent和机器人基准测试中展示了优越的性能。通过这些结果，该框架使语言Agent能够灵活地利用当前情境中的过去经验，模拟人类的能力，从而增强决策能力。

## 相关链接

Tomoyuki Kagaya, Thong Jing Yuan, Yuxuan Lou, et al. RAP: Retrieval-Augmented Planning with Contextual Memory for Multimodal LLM Agents. arXiv:2402.03610

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【RAG技术洞察】01-浅谈RAG的十大挑战](https://zhuanlan.zhihu.com/p/696904158)  
> 下一篇：[【RAG技术洞察】03-Agent检索增强生成：突破传统RAG局限，构建更加智能、贴近事实的LLM应用！](https://zhuanlan.zhihu.com/p/678681627)
