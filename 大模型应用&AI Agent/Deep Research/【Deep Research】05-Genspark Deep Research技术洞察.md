# 【Deep Research】05-Genspark Deep Research技术洞察

原文链接：https://zhuanlan.zhihu.com/p/1942259499411935994

---

​

目录

## 1 概述

[Genspark](https://zhida.zhihu.com/search?content_id=262112759&content_type=Article&match_order=1&q=Genspark&zhida_source=entity)于2025年2月由前百度高管景鲲、朱凯华正式推出，是一款面向办公场景的商业化通用智能体产品。其核心功能包含内容创作（PPT文档）和深度研究等，旨在通过多智能体框架提供个性化和高效的信息搜索服务。它基于创新的[MoA](https://zhida.zhihu.com/search?content_id=262112759&content_type=Article&match_order=1&q=MoA&zhida_source=entity)（Mixture-of-Agents）架构构建，能够动态协调多个专业化的人工智能模型（9个）和工具（80+内部工具），高效且准确地完成用户指定的复杂任务。在[GAIA基准测试](https://zhida.zhihu.com/search?content_id=262112759&content_type=Article&match_order=1&q=GAIA%E5%9F%BA%E5%87%86%E6%B5%8B%E8%AF%95&zhida_source=entity)中，平均正确率达到73.1%。

## 2 技术方案

从技术架构看，Genspark Deep Research基于MoA多智能体协同框架研究质量突破，关键技术包括：

  * 首先生成全局研究计划，然后进入一个迭代过程。迭代内根据计划执行搜索、阅读、反思（智能体评估本迭代输出质量，并推理/规划下一步动作），如此进行多次迭代，直到信息完备，满足要求后终止。使用多轮迭代 + 反思演绎慢思考的过程，实现思考过程的可控可定制。
  * **多LLM集体见解** ，采用动态的模型集成和智能路由策略，对同一任务调用多个LLM，获取不同模型的异构输出，补齐模型能力短板。每个模型都会搜索内容提供专门的响应和独特的视角。
  * **反思完善** ，通过工作流编排对各模型的输出进行交叉验证、反思和完善，消减模型幻觉。
  * 推理模型采用[OpenAI](https://zhida.zhihu.com/search?content_id=262112759&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)的o3-mini-high和[DeepSeek R1](https://zhida.zhihu.com/search?content_id=262112759&content_type=Article&match_order=1&q=DeepSeek+R1&zhida_source=entity)，增强的复杂分析推理能力；并通过10+专有数据集，加强系统对特定领域知识和现实世界上下文的理解能力，从而能够支持更明智的决策制定和更准确的输出结果。



## 3 能力解析

Deep Research的核心能力包含如下4个维度，在此将会从这几个维度对Genspark Deep Research的能力进行解析。

  * **研究计划制定** ：支持执行过程中自主调整后续研究计划。首先会调用initial_plan工具，用来对问题进行初步规划，将问题拆分为一系列更小、更容易处理的子任务。然后执行这些子任务，并通过对子任务的输出进行反思，动态更新后续的研究计划，持续完善结果。
  * **工具使用与环境交互** ：支持使用搜索工具等并行获取在线信息，生成研究报告。同时，系统采用智能工具路由与检索技术，能够根据任务特性自动匹配最适合的工具组合。
  * **内容评估与报告生成** ：支持跨来源信息的搜集和综合，内置反思和多智能体交叉验证机制。
  * **多角色Agent协同** ：基于MoA架构多角色智能体协同，具体实现细节未公开。



## 4 总结

Genspark Deep Research凭借其MoA架构，实现了强大的复杂任务处理能力，能够生成结构化、信息密度高的洞察报告。然而，其在​​信息隐私和安全、引用标注方式较杂乱、运行成本​等方面仍需持续投入优化。

## 相关链接

  1. [https://mainfunc.ai/blog/genspark_autopilot_agent_deep_research_v2](https://link.zhihu.com/?target=https%3A//mainfunc.ai/blog/genspark_autopilot_agent_deep_research_v2)
  2. [https://mainfunc.ai/blog/genspark_moa_powered_search](https://link.zhihu.com/?target=https%3A//mainfunc.ai/blog/genspark_moa_powered_search)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Deep Research】04-Gemini Deep Research技术洞察](https://zhuanlan.zhihu.com/p/1940785645107812348)  
> 下一篇：[【Deep Research】06-Kimi-Researcher技术洞察](https://zhuanlan.zhihu.com/p/1943344392275461055)
