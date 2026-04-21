# 【Deep Research】08-DeerFlow技术洞察

原文链接：https://zhuanlan.zhihu.com/p/1946890604698146510

---

​

目录

## 1 概述

[Deerflow](https://zhida.zhihu.com/search?content_id=262635751&content_type=Article&match_order=1&q=Deerflow&zhida_source=entity)是字节跳动于2025年5月开源的深度研究项目，迄今为止Github Star数已达15.6k+，实现模块化的多智能体系统架构，专为自动化研究和代码分析而设计。该系统基于[LangGraph](https://zhida.zhihu.com/search?content_id=262635751&content_type=Article&match_order=1&q=LangGraph&zhida_source=entity)构建，实现了灵活的基于状态的工作流，集成多类LLM模型（如推理型和基础型），支持Tavily、[DuckDuckGo](https://zhida.zhihu.com/search?content_id=262635751&content_type=Article&match_order=1&q=DuckDuckGo&zhida_source=entity)、Brave Search和Arxiv的搜索工具、爬虫和MCP服务，支持接入RAGFlow和[VikingDB](https://zhida.zhihu.com/search?content_id=262635751&content_type=Article&match_order=1&q=VikingDB&zhida_source=entity)的私域知识库，实现人机协作和多模态内容生成。

## 2 技术方案

从技术架构看，Deerflow基于LangGraph构建了包含协调器、规划器、研究员、编码员和报告员的多Agent系统，以及丰富的工具实现研究质量突破。关键技术包括：

  * **多智能体架构** ：基于LangGraph构建的模块化的多智能体架构，实现了灵活的基于状态的工作流。
  * **工具和MCP集成** ：支持Tavily、DuckDuckGo、Brave Search和Arxiv的搜索工具、爬虫和MCP服务，支持接入RAGFlow和VikingDB的私域知识库。
  * **人机协作** ：支持Human in the Loop，系统在执行前向用户展示生成的研究计划，用户可确认接受和编辑计划。支持报告后期编辑，允许AI优化。



## 3 能力解析

  * **研究计划制定** ：规划器Agent负责任务分解与策略规划，分析研究目标生成研究计划，研究员和编码员按计划执行，根据上下文信息的充分性决定进一步研究迭代或调用报告员Agent生成最终报告。若规划器生成的研究计划解析失败则与用户交互重新制定计划。
  * **工具使用与环境交互** ：集成多种LLM模型，根据需求配置推理型或基础型模型，支持Tavily、DuckDuckGo、Brave Search和Arxiv的搜索工具、爬虫和MCP服务，支持接入RAGFlow和VikingDB的私域知识库。
  * **内容评估与报告生成** ：通过报告员Agent汇总团队研究的发现、处理组织信息、生成全面的结构化图文研究报告，支持从报告到PodCast和PPT的多模态内容生成。
  * **多角色Agent协同** ：引入协调器、规划器、研究员、编码员和报告员形成多Agent系统，协调器作为用户和系统之间的主要接口，在适当时候将任务委派给规划器，规划器负责研究计划制定和是否调用报告员生成最终报告，研究员可使用搜索工具、爬虫和MCP服务进行信息收集，编码员使用Python REPL工具执行代码分析、算法验证等技术任务，研究员和编码员按研究计划执行任务。



## 4 总结

DeerFlow是字节跳动推出的开源深度研究平台，通过模块化多智能体协作自动化复杂研究流程，集成多类LLM模型、搜索工具、MCP服务以及私域知识库，允许人工审核和编辑研究计划，允许报告后期编辑，支持多模态内容生成。

## 相关链接

  1. [https://github.com/bytedance/deer-flow](https://link.zhihu.com/?target=https%3A//github.com/bytedance/deer-flow)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Deep Research】07-LangChain Open Deep Research技术洞察](https://zhuanlan.zhihu.com/p/1945049592199615459)  
> 下一篇：[【Deep Research】09-WebAgent技术洞察](https://zhuanlan.zhihu.com/p/1947233864620709465)
