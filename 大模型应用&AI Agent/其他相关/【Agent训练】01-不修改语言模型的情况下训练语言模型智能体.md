# 【Agent训练】01-不修改语言模型的情况下训练语言模型智能体

原文链接：https://zhuanlan.zhihu.com/p/698325773

---

​

目录

研究人员和实践者最近将强大的[大型语言模型](https://zhida.zhihu.com/search?content_id=243313629&content_type=Article&match_order=1&q=%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）重新定义为[智能体](https://zhida.zhihu.com/search?content_id=243313629&content_type=Article&match_order=1&q=%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)（Agent），使它们能够通过使用专门的函数来自动完成复杂的任务。为了促进LLM Agent的开发，本篇论文《Offline Training of Language Model Agents with Functions as Learnable Weights》提出了一种**不修改LLM权重来训练LLM** **Agent** 的新范式，其灵感来自人类如何通过锻造工具来适应现实世界的任务。

作者引入了AgentOptimizer，利用LLM来更新Agent的函数，并设计具有回滚和[提前停止](https://zhida.zhihu.com/search?content_id=243313629&content_type=Article&match_order=1&q=%E6%8F%90%E5%89%8D%E5%81%9C%E6%AD%A2&zhida_source=entity)两种策略的Agent训练算法，以简化训练过程，并对三项任务（数学推理、表格处理和一般现实世界任务）进行了实验，证明Agent训练新范式显著提高了LLM Agents的性能。

## 背景介绍

将大型语言模型（LLM）重新构建为Agent，LLM可以利用现有函数来完成复杂的任务。

为了使LLM Agent发挥有用功效，用户需要先手动创建对特定下游任务有帮助的函数。这个过程可能需要多次迭代，因此非常耗时。由于LLM是黑匣子，研究人员发现LLM意外地无法利用某些类型的函数。作为回应，研究人员试图通过使用真实函数调用对LLM进行微调，来提高底层LLM使用现有函数的能力，但这个微调过程需要大量的计算资源。更糟糕的是，它限制了可以使用哪些LLM，因为许多LLM模型是专有的。

## 技术方案

受到人造工具成为人类用户的延伸这一事实的启发，以及人类如何锻造工具以最好地适应现实世界的任务，而不是改变人类的生物结构以适应一组静态的工具，本文提出了一种新的Agent训练范式，该范式“锻造”了**LLM** **Agent使用的函数** ，以最好地适应训练任务。

它同时解决了上述两个挑战，因为它不需要对底层的LLM进行微调，并且可以从一个空的函数集开始。

具体来说，在传统模型训练和Agent训练之间做了一个类比，如图1所示。

  1. **Agent训练过程不是更新模型参数，而是更新LLM Agent的函数，将它们视为Agent的“可训练参数”** ；
  2. **Agent过程使用Agent的执行历史和在训练任务上的表现作为更新Agent函数的基础，而不是在训练集上计算损失。**



作者引入了 AgentOptimizer，它利用 LLM 根据执行历史记录和当前epoch中的Agent生成的/真实的答案来更新Agent的函数。特别地，指示AgentOptimizer通过执行预定义的函数操作动作（添加、修改和删除）之一来逐步更新当前函数集，而不是在每个优化步骤重新生成整个函数集。

使用AgentOptimizer，**LLM** **Agent训练的总体工作流程** 如下：

给定一个训练集和空函数集，在每个epoch，首先根据训练集评估Agent系统，并收集执行历史以及Agent生成的/真实的答案，然后将此信息提供给AgentOptimizer以执行优化步骤来更新当前函数集。

为了避免函数更新导致的潜在性能下降，作者引入了两个简单的策略：回滚和提前停止。前者是在训练集上的性能下降时撤销当前的函数更新并回退到之前的状态，而后者是在一定数量的连续优化步骤没有在训练集上引入任何性能提升时提前终止优化过程。

## 实验结果

为了评估所提出的Agent训练的有效性，本文对三个不同的任务进行了实验：数学推理、表格处理和一般现实世界任务，使用所提出的Agent训练方法来训练两个典型的LLM Agent系统：[GPT-4](https://zhida.zhihu.com/search?content_id=243313629&content_type=Article&match_order=1&q=GPT-4&zhida_source=entity) \+ Agent和ReAct Agent，对于这两种系统均使用Python初始化作为可以执行LLM建议的Python代码的初始函数。

  1. 数学推理  
如表1所示，**在七种数据类型中Agent训练在大多数情况下都能在测试集上获得更好的性能** 。此外，几乎所有样例的训练表现都有所改善或保持不变。  




  
从结果来看，对于计数和概率问题，当训练GPT-4+Agent时训练性能保持不变，而测试性能**从72.5%提高到76.3%** 。这表明在特定情况下，即使生成的函数不会导致训练集上的性能改进，它们也会对未见过的测试数据有所帮助。

  1. 表格处理和一般现实世界任务



对表格处理任务[TabMWP](https://zhida.zhihu.com/search?content_id=243313629&content_type=Article&match_order=1&q=TabMWP&zhida_source=entity)和一般现实世界任务GAIA进行评估，如表2所示，Agent训练提升了两个Agent系统的性能。

由于这两个数据集比MATH更真实和复杂，从结果来看，**Agent训练可以生成通用和可用的函数，从而提高Agent的现实任务解决能力** ，这表明Agent训练在一定程度上具有实用性。

## 总结

本文提出了一种新的方法来训练专门的LLM Agent，其核心思想是在LLM Agent训练和传统模型训练之间进行类比，传统模型中的可学习参数对应于LLM Agent的操作函数，而模型的损失函数对应于Agent的历史性能指标。

利用LLM的优化能力，通过AgentOptimizer更新Agent函数来增强Agent，在训练两个典型的Agent系统的多个不同任务上评估了所提出的方法，并证明了Agent训练表现出明显的性能提升。

## 参考资料

  1. Zhang S, Zhang J, Liu J, et al. Training Language Model Agents without Modifying Language Models[J]. arXiv preprint arXiv:2402.11359, 2024.



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【AgentOS】01-AIOS：LLM智能体操作系统](https://zhuanlan.zhihu.com/p/691420682)
