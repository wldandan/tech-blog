# 【PromptWizard】01-PromptWizard，Prompt工程上又一颗璀璨明星

原文链接：https://zhuanlan.zhihu.com/p/25290259323

---

​

目录

微软研究院最近刚刚开源了[PromptWizard](https://zhida.zhihu.com/search?content_id=254080237&content_type=Article&match_order=1&q=PromptWizard&zhida_source=entity)，一个用于自动化提示（Prompt）和示例优化的框架，它利用迭代反馈和合成过程不断改进提示。PromptWizard性能始终优于当前最先进的方法，同时还能显著降低计算成本，从而为各种任务和[LLM](https://zhida.zhihu.com/search?content_id=254080237&content_type=Article&match_order=1&q=LLM&zhida_source=entity)提供高效、可扩展的提示工程解决方案。

## 概述

LLM在各种任务中都取得了卓越的表现，其成功的核心在于提示。研究表明，提示会显著地影响LLM的性能，因此提示工程成为提升模型准确性的关键要素。

然而，设计有效的提示是一项劳动密集且领域特定的任务，通常需要具有专业的知识和主观判断。随着模型的演进和任务的变化，重复设计提示会带来大量繁重的工作。因此，引出一个关键问题：**提示工程能否实现自动化，以简化流程并提高可扩展性？**

## 背景

提示是LLM的核心能力！

**提示（Prompting）定义** ：通过提供输入指令，引导模型生成符合预期的输出。

**提示工程（Prompt Engineering）定义** ：设计和优化提示的过程。

**提示设计的挑战** ：

  * 劳动密集型：提示设计需要耗费大量时间和精力。
  * 领域特定：提示需要根据具体领域进行定制，才能发挥作用。
  * 专业知识：通常需要人类的专业知识、经验，具有主观性。
  * 重复设计：随着模型的升级和任务的变化，提示需要不断重新设计。



## 技术方案

PromptWizard是一个离散提示优化框架，采用自适应进化机制，允许LLM自主生成、批评和完善提示及示例，并通过迭代反馈和合成不断改进。

**三大技术方案**

  * **基于反馈的优化** ：LLM自主生成、批评和优化提示及示例，通过迭代反馈和合成不断改进。
  * **批评并合成多样化示例** ：生成具有鲁棒性、多样性且任务相关的合成示例，同时针对提示和示例进行同步优化。
  * **自生成的思维链（CoT）** ：结合正向、负向和合成示例，通过思维链提升模型解决问题的能力。



### **基于反馈的优化**

使用系统化的反馈驱动流程，通过批评组件提供反馈，从而指导并优化提示。在多次迭代中逐步完善提示。

以下步骤系统地实现这一过程：

  * **变异（Mutate）** ：基于初始问题描述和思维风格生成提示。
  * **评分（Scoring）** ：评估生成提示的性能，以确定最佳提示。
  * **批评（Critique）** ：通过分析模型表现不佳的案例，审查提示的成功与失败之处。
  * **合成（Synthesize）** ：利用批评反馈优化最佳提示。



### **批评并合成多样化示例**

  * PromptWizard可同时改进提示指令和少样本（Few-Shot）学习示例
  * 它利用自我反思合成多样化且任务相关的示例。
  * 通过迭代反馈循环，持续优化提示和[少样本示例](https://zhida.zhihu.com/search?content_id=254080237&content_type=Article&match_order=1&q=%E5%B0%91%E6%A0%B7%E6%9C%AC%E7%A4%BA%E4%BE%8B&zhida_source=entity)。



少样本示例优化：

  * 批评：分析之前选择的示例，并利用反馈确定如何改进示例。
  * 合成：生成新的合成示例，使其更加多样化、鲁棒性且与任务相关。
  * 提示指令优化：
  * 批评：识别提示中的缺点和待改进的地方。
  * 合成：利用批评的反馈进一步优化提示指令。



### **[思维链推理](https://zhida.zhihu.com/search?content_id=254080237&content_type=Article&match_order=1&q=%E6%80%9D%E7%BB%B4%E9%93%BE%E6%8E%A8%E7%90%86&zhida_source=entity)**

  * 纳入思维链（CoT）提升模型解决问题的能力。
  * 思维链推理针对选定的少样本示例，为每个示例生成详细的推理链，从而促进问题解决。
  * 使用LLM 检查示例的一致性和相关性。



## 实验数据

PromptWizard在所有任务中始终保持接近最佳的准确性。在多个数据集上，它的平均准确率达到 81.9%，最高可达 95.4%。

PromptWizard每项任务的成本仅为0.05美元，整体代币/成本降低了5-60倍。

## 技术方案思考

随着Dspy、PromptWizard等自我迭代的框架不断发展，提示的优化正在从低效的依赖专家经验演进到由框架通过反馈自动实现。虽然[DSPy](https://zhida.zhihu.com/search?content_id=254080237&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)和PromptWizard都是旨在优化大型语言模型（LLMs）提示词的框架，但它们在设计理念和实现机制上存在一些区别：

### **设计理念**

    * **DSPy** ：DSPy（Declarative Language Model Programming）强调通过编程而不是手动编写提示词来使用语言模型。它通过声明性编程来指定程序应该做什么，而不是如何做。DSPy的核心是将提问技巧转化为可以推理任意签名的模块，参数化这些模块，并通过迭代收集示例来学习实现签名所定义的变换行为。
    * **PromptWizard** ：PromptWizar通过自我演变和自我适应机制，迭代优化提示指令和上下文示例，提升LLMs在特定任务中的表现。其创新地引入了引导探索、序列优化、高效示例合成和错误分析等方法，提高了Prompt自生成的效率和质量。



### **实现机制**

    * **DSPy** ：DSPy使用自然语言签名声明每个模块的输入/输出行为，并将提示技术翻译为参数化的声明式模块，这些模块可以被组合形成管道。DSPy的编译器优化任意管道，常用策略包括Bootstrap少量示例、模型选择等。
    * **PromptWizard** ：PromptWizard具有独特的“反馈驱动”机制，它能够让AI模型在感知不同的任务的情况下，自主生成、评估和优化Prompt。这样可以大大提升模型在不同任务中的性能。



### **优化过程**

    * **DSPy** ：DSPy的优化过程涉及到收集训练数据、定义程序、选择评估方式和优化器等步骤，它通过模块化和自动编译，有效减少依赖手工设计提问模板。
    * **PromptWizard** ：PromptWizard的优化过程包括反馈驱动的迭代优化、联合优化指令和样例、自生成推理步骤，它通过合成多样且任务相关的样例，同时优化指令和上下文示例。



### **应用场景**

    * **DSPy** ：DSPy适用于需要构建高质量系统的场景，它允许利用更小型和更具成本效益的数据构建系统，同时支持对丰富的设计空间进行系统探索。
    * **PromptWizard** ：PromptWizard适用于需要大规模提示优化的场景，它在多个任务中显示出卓越的准确率，并大幅降低资源消耗和成本。



总的来说，DSPy更侧重于通过编程和模块化来优化提示，而PromptWizard则侧重于通过自我演变和自我适应机制来自动化和简化提示优化过程。两者都旨在提升LLMs在特定任务中的表现，但它们的实现方式和优化策略有所不同。

## 相关链接

  1. DSPy GitHub： _[https:// github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)_
  2. DSPy官网： _[https:// dspy.ai/](https://link.zhihu.com/?target=https%3A//dspy.ai/)_
  3. PromptWizar论文： _[https:// arxiv.org/pdf/2405.18369](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2405.18369)_
  4. PromptWizar GitHub： _[https:// github.com/microsoft/PromptWizard](https://link.zhihu.com/?target=https%3A//github.com/microsoft/PromptWizard)_



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【PromptWizard技术洞察】02-Prompt又一选择，从0-1快速上手PromptWizard](https://zhuanlan.zhihu.com/p/26154065107)
