# 【模型路由】01-OrchestraLLM：高效协调对话状态跟踪语言模型

原文链接：https://zhuanlan.zhihu.com/p/700883845

---

​

目录

大型语言模型（LLM）已成为一种多功能工具，只需训练少量示例就能处理各种任务。然而，其规模的不断扩大带来了计算需求的不断攀升。相比之下，更高效的小语言模型（SLM）往往需要大量的微调数据才能真正有效。它针对的是只有有限的特定任务数据可用，使得微调小语言模型的可靠性降低。**本研究要解决的问题是开发一个路由框架，协调 SLM 和 LLM，在提高任务性能的同时降低计算成本。** 在结构化知识提取任务中，**SLM 和 LLM 表现出互补优势** ，在这一发现的推动下，通过协调 SLM/LLM 两种特性来提升任务性能成为可能。本篇论文《[OrchestraLLM](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=OrchestraLLM&zhida_source=entity): Efficient Orchestration of Language Models for Dialogue State Tracking》提出了一个[动态路由框架](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%E6%A1%86%E6%9E%B6&zhida_source=entity) OrchestraLLM，实现大小模型的协调，以此提升任务性能。

## 技术方案

### 概述

本文提出了一个动态路由框架 OrchestraLLM（如图1所示），它利用了小型（微调）和大语言模型专家。假设语义嵌入相似实例的难度相同，根据测试实例与专家池中实例之间的嵌入距离选择合适的专家。专家池中的实例代表了不同语言模型能提供更可靠答案的对话语境类型。在检索出最接近的 k 个实例后，根据多数票选出实例的路由。

### 主要组成部分

**[对话状态跟踪](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=%E5%AF%B9%E8%AF%9D%E7%8A%B6%E6%80%81%E8%B7%9F%E8%B8%AA&zhida_source=entity) DST**

这部分讲解如何将通用 LLM 和特定任务 SLM 结合起来，以提高对话状态跟踪（DST）的效率。DST 的目标是从用户-系统话语中提取与任务相关的信息作为结构化的表征（对话状态），以便相应地满足用户的请求。  
通常基于特定任务 SLM 的 DST 模型是通过全参数更新进行微调的，而使用 LLM 的 DST 模型则是通过少量的上下文学习来实现的。作者分别讨论了这两种不同的DST模型。

**LLM DST** ：[IC-DST](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=IC-DST&zhida_source=entity) 是一种上下文学习（ICL）框架，可通过 LLMs 实现少量 DST。预测的是每一轮的变化，而不是累积的对话状态。要获得累积的对话状态，需要汇总各回合的变化。前一轮的对话状态被用作上下文历史的总结，这样可以容纳更多的示例。

**SLM DST** ：作者用 SLM 开发了一个基于提示的 DST 模型（称为 [Prompt-DST](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=Prompt-DST&zhida_source=entity)）。Prompt-DST 的输入与 IC-DST 相似，只是排除了上下文中的示例。

### 路由方法

  * **将 OrchestraLLM 应用于 DST 任务的路由选择方法** ：整体框架如图 1 所示。作者将不同的 DST 模型称为专家。给定一个新的输入实例，OrchestraLLM 首先计算其语义嵌入，将其与每个专家库中的实例进行比较，并检索前 K 个示例。路由器根据多数票将输入分配给专家。
  * **专家库构建方法** ： 对于小范围保留集中的每个对话，SLM 和 LLM 专家分别预测每个用户回合的 TLB（ ）。如果两位专家都能正确预测 TLB，那么实例就会被纳入 SLM 池。如果只有一位专家正确预测了 TLB，实例就会被分配到该专家的池中。未被正确预测的实例不会被用于任何一个池中。



## 实验方法及结果

### 实验方法

作者对 DST 进行少量采样设置，分别从 [MultiWOZ](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=MultiWOZ&zhida_source=entity)（一个面向任务的多领域对话数据集）和 [SGD](https://zhida.zhihu.com/search?content_id=243882377&content_type=Article&match_order=1&q=SGD&zhida_source=entity)（一个面向任务的对话数据集） 中随机抽取 5% 的训练数据，用于训练专家模型。  
在MWOZ 实验中，从保留的对话集中随机抽取 100 个对话作为 SLM 池和 LLM 池；在 SGD 实验中则抽取 300 个。设置 =10 ，在票数相同时以 SLM 为优先。

### 实验结果  


  
IC-DST 在少数几个回合的情况下优于 Prompt-DST，**这表明 LLM 比微调 SLM 更具通用性** 。OrchestraLLM 的表现也优于 IC-DST，同时节省了 60% 的 LLM 调用，**这证明了本文所提出框架的有效性** 。  
尽管 Prompt-DST 体积小巧，但它可以**根据特定领域知识和特定任务进行微调** 。与此相对，IC-DST 在 LLM 的预训练阶段获得了丰富的知识库，使其**具备了上下文推理能力** ，并增强了对常识性知识的掌握。**由于这两个模型是互补的，有效的整合可以超越 IC-DST 模型的性能** 。  


  
该表格数据表明，对目标进行微调后，效率提高了 5%，TLB 和 DST 分数也都有所提高。更多的轮次被路由到 SLM，从而提高了 TLB 得分。这表明**路由器具有足够的通用性，可以支持跨域分配，并成功提高了系统精度** 。

### **实验分析**

**SLM、LLM与OrchestraLLM对比**

  
数据说明：蓝色条Prompt-DST表示基于**SLM** 开发的DST模型；橙色条IC-DST表示基于**LLM** 实现的DST模型；绿色条表示使用不同检索器的 **OrchestraLLM** 。

**代表性示例展示**

数据说明：红色文本为错误回答。

为了更好地理解 LM 的互补性，作者在上表中提供了专家库中具有代表性的示例。LLM 常犯的一个错误是没有遵守模式。在这个例子中，LLM 简单地复制了转折中的文本（"affordable"）作为 DST 预测，而 SLM 则能够以模式特定的格式（"cheap"）来确定值。

不过，与 SLM 相比，我们发现 **LLM 有两个优势** ：

  1. **它在处理常识性知识方面表现出色** 。例如，它可以根据上下文（"我和我妈妈"）推断出住在宾馆的客人的正确人数。
  2. **它能熟练地进行长语境推理** 。当跨域引用以前的上下文时，LLM 始终能做出正确的推理，而 SLM 则经常忽略上下文并产生随机值。



## 总结与展望

本篇论文介绍的 OrchestraLLM 是一个路由框架，它无缝集成了 SLM 和 LLM，并由基于检索的路由器进行协调。在推理过程中，动态路由器会利用 SLM 和 LLM 的能力，将实例引导至其中一个 LM。评估表明，**OrchestraLLM 的性能优于基于 LLM 的系统，同时还节省了 50% 以上的计算成本** 。这项研究标志着语言模型的高效协作迈出了重要一步，尤其是在多轮人机交互系统（如任务导向对话）中。

尽管该研究证明了将 SLM 和 LLM 结合使用的好处，即在提高任务性能的同时控制计算成本，尤其是在对话状态跟踪任务中。但是，本文的方法可能无法无缝扩展到更广泛的 NLP 领域中的所有任务类型。此外，目前的框架主要利用 SLM 和 LLM。然而，现实世界中的应用往往涉及各种各样的任务，每个任务都可能需要具有不同专业知识的 LM。作为未来的研究方向，同时协调多个 LM 的问题是有必要的。

## 相关链接

Chia-Hsuan Lee, Hao Cheng, Mari Ostendorf. OrchestraLLM: Efficient Orchestration of Language Models for Dialogue State Tracking. arXiv:2311.09758

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【模型路由技术洞察】02-RouteLLM：利用偏好数据学习路由LLMS](https://zhuanlan.zhihu.com/p/21408688093)
