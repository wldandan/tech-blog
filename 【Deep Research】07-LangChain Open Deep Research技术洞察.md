# 【Deep Research】07-LangChain Open Deep Research技术洞察

原文链接：https://zhuanlan.zhihu.com/p/1945049592199615459

---

​

目录

## 1 概述

[LangChain](https://zhida.zhihu.com/search?content_id=262435768&content_type=Article&match_order=1&q=LangChain&zhida_source=entity) Open Deep Research是LangChain官方于2024年12月完全开源的深度研究智能体，迄今为止Github Star数已达5.9k+，可进行全面的深入研究。该系统基于[LangGraph](https://zhida.zhihu.com/search?content_id=262435768&content_type=Article&match_order=1&q=LangGraph&zhida_source=entity)构建，提供了广泛的配置选项来自定义研究过程，用户可选择搜索工具（如Tavily、OpenAI原生Web搜索、[Anthropic原生Web搜索](https://zhida.zhihu.com/search?content_id=262435768&content_type=Article&match_order=1&q=Anthropic%E5%8E%9F%E7%94%9FWeb%E6%90%9C%E7%B4%A2&zhida_source=entity)等），设置反思研究追问的次数、最大迭代次数等，也支持自定义模型、搜索工具和[MCP服务](https://zhida.zhihu.com/search?content_id=262435768&content_type=Article&match_order=1&q=MCP%E6%9C%8D%E5%8A%A1&zhida_source=entity)，从而提供高质量的研究成果。

## 2 技术方案

从技术架构看，其基于LangGraph实现"范围界定-研究-报告撰写"的三阶段研究流程，允许用户自由替换模型与搜索工具。关键技术包括：

  * **多智能体研究架构** ：采用多智能体架构，主管智能体对子智能体进行分工协作与动态调整从而实现高效、全面的研究。
  * **上下文工程** ：压缩用户对话生成研究简报，子智能体清洗研究结果，过滤无关信息、提取关键结论并标注来源。
  * 基于LangGraph实现深度研究流程，支持用户自定义模型、搜索工具和MCP服务，设置反思研究追问的次数、最大迭代次数等。



## 3 能力解析

  * **研究计划制定** ：主管智能体根据简报内容将任务拆分为多个子任务，每个子智能体负责一个子任务，主管智能体灵活选择子智能体是否进行并行处理，子智能体返回结果后主管智能体判断是否满足需求，如不满足则根据差距继续追加子智能体，循环迭代直至研究结果完全覆盖简报范围。
  * **工具使用与环境交互** ：可选择搜索工具（如Tavily、OpenAI原生Web搜索、Anthropic原生Web搜索等）和MCP服务，也支持用户自定义。
  * **内容评估与报告生成** ：均采用prompt方式，子智能体清洗研究结果返回给主管智能体，主管智能体评估结果是否覆盖简报范围，另外单独的LLM进行总结生成报告。
  * **多角色Agent协同** ：采用“主管智能体（supervisor）+ 子智能体（sub-agents）”的架构，主管智能体负责统筹协调各个子智能体并判断返回结果有效性，各个子智能体在隔离的上下文中自行调用搜索工具与MCP服务，完成搜索后简化结果并返回。



## 4 总结

LangChain Open Deep Research是一个完全开源的深度研究平台，通过多阶段流程高效生成高质量研究报告，支持用户自定义模型、搜索工具和MCP服务。LangChain将在令牌效率优化、实时质量评估、长期记忆与缓存机制方面持续优化。

## 相关链接

  1. [https://github.com/langchain-ai/open_deep_research](https://link.zhihu.com/?target=https%3A//github.com/langchain-ai/open_deep_research)
  2. [https://blog.langchain.com/open-deep-research/](https://link.zhihu.com/?target=https%3A//blog.langchain.com/open-deep-research/)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Deep Research】06-Kimi-Researcher技术洞察](https://zhuanlan.zhihu.com/p/1943344392275461055)  
> 下一篇：[【Deep Research】08-DeerFlow技术洞察](https://zhuanlan.zhihu.com/p/1946890604698146510)
