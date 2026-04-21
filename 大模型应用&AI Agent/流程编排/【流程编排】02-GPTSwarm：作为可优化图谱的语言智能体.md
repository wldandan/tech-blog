# 【流程编排】02-GPTSwarm：作为可优化图谱的语言智能体

原文链接：https://zhuanlan.zhihu.com/p/6550590254

---

​

目录

## 问题背景

为了改进基于大型语言模型（LLMs）的问题求解器，人们提出了各种人工设计的提示工程技术，产生了许多不同的代码库。例如提供工程、单/多智能体等等方案。

  * 提示工程：通过设计合适的提示，引导 LLM 执行特定的任务。例如，Chain of Thought (COT) 和 ReAct 等方法通过结构化的提示，提高 LLM 的推理能力。
  * 单智能体框架：例如，AutoGPT 和 LangChain 等框架，利用 LLM 执行各种外部函数和工具，实现各种功能。
  * 多智能体系统：例如，NL-SOM 和 CAMEL 等系统，利用多个 LLM 执行不同的角色，并通过自然语言进行通信和协作。



但是，目前各种方案都存在一些问题，如智能体系统存在碎片化、效率低下和可扩展性差等问题：

  * 碎片化：各种不同的智能体系统通常采用不同的代码库和提示方案，难以集成和复用。
  * 效率低下：智能体系统通常需要大量的人工工程，难以自动化优化。
  * 可扩展性差：难以构建和优化大型智能体群体。



## 方案与关键技术

本文提出的**[GPTSwarm](https://zhida.zhihu.com/search?content_id=250332597&content_type=Article&match_order=1&q=GPTSwarm&zhida_source=entity) 方案核心思路是将语言智能体建模为计算图，并通过[节点优化](https://zhida.zhihu.com/search?content_id=250332597&content_type=Article&match_order=1&q=%E8%8A%82%E7%82%B9%E4%BC%98%E5%8C%96&zhida_source=entity)和边优化，自动改进智能体系统的提示和协作模式，提高其效率和性能。**

具体来说，GPTSwarm 的核心思路包括以下几个方面：

  * **计算图表示** ：将语言智能体建模为计算图，其中每个节点表示一个基本操作，例如 LLM 推理、工具使用、函数调用等。节点之间的边表示信息流动和操作顺序。
  * **节点优化** ：通过自动优化每个节点的提示，提高节点操作的效率和性能。
  * **边优化** ：通过自动优化节点之间的连接关系，发现更有效的协作模式和通信路径。
  * **模块化设计** ：GPTSwarm 提供模块化的设计，方便用户构建和复用各种智能体组件，并支持多种节点操作和智能体类型。



该方案具有层级化的特点，其最基础节点为node，包括llm、tool等，通过node组成graph，可以理解为一个agent，通过graph组成 composite graph，即组合图。  
其旨在优化点主要在节点和边，目标如下：

  * **节点** ：优化llm中的提示词。
  * **边** ：优化边的路由，使得信息更好的传递。



## 实验与数据

作者们在实验过程中采用的数据集如下：

  * [MMLU](https://zhida.zhihu.com/search?content_id=250332597&content_type=Article&match_order=1&q=MMLU&zhida_source=entity): 用于测试智能体在对抗性场景下的鲁棒性，以及协作场景下的性能提升。
  * [Mini Crosswords](https://zhida.zhihu.com/search?content_id=250332597&content_type=Article&match_order=1&q=Mini+Crosswords&zhida_source=entity): 用于测试边优化对标准智能体性能的提升。
  * [HumanEval](https://zhida.zhihu.com/search?content_id=250332597&content_type=Article&match_order=1&q=HumanEval&zhida_source=entity): 用于测试节点优化对智能体性能的提升。
  * [GAIA](https://zhida.zhihu.com/search?content_id=250332597&content_type=Article&match_order=1&q=GAIA&zhida_source=entity): 用于测试 GPTSwarm 的通用性和性能。



实验的场景设置如下：

  * 对抗性场景: 在 MMLU 数据集上，构建包含真实智能体和对抗性智能体的群体，并通过边优化过滤掉对抗性智能体，提高群体的性能。
  * 协作场景: 在 Mini Crosswords 数据集上，构建包含多个 TOT、COT 和 Reflexion 智能体的群体，并通过边优化发现更有效的协作模式和通信路径，提高群体的性能。
  * 节点优化: 在 HumanEval 数据集上，优化 ReAct 智能体中生成 Python 代码的节点的提示，提高智能体的性能。
  * 通用性测试: 在 GAIA 数据集上，构建包含多个 TOT 智能体的群体，并通过自洽性策略进行决策，测试 GPTSwarm 的通用性和性能。



## 总结与展望

这篇论文提出了 GPTSwarm，一个基于计算图的模块化框架，用于构建和优化基于 LLM 的智能体系统。GPTSwarm 的核心贡献与价值可以概括为以下几个方面：

  * **统一框架**  
将语言智能体建模为计算图，提供了一个统一的框架，方便用户构建和复用各种智能体组件。支持多种节点操作和智能体类型，可以应用于各种场景。
  * 自动化优化  
通过节点优化和边优化，自动改进智能体系统的提示和协作模式，提高其效率和性能。使用强化学习算法实现自动化优化，无需人工干预。
  * **可扩展性**  
支持构建和优化大型智能体群体。通过模块化设计，方便扩展和定制。
  * **性能提升**  
在多个数据集上，GPTSwarm 能够显著提高智能体系统的性能，优于现有的基线方法。
  * **开源框架**



GPTSwarm 提供了开源代码，方便其他研究人员和开发者使用和扩展。

## 相关链接

Mingchen Zhuge, Wenyi Wang, Louis Kirsch, Francesco Faccio et al. GPTSwarm: Language Agents as Optimizable Graphs. arXiv:2402.16823

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【流程编排洞察】01-AFLOW：自动生成智能体工作流](https://zhuanlan.zhihu.com/p/5605781955)  
> 下一篇：[【流程编排洞察】03-通过程序合成进行自然语言命令](https://zhuanlan.zhihu.com/p/7098467577)
