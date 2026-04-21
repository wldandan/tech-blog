# 【Deep Research】03-OpenAI Deep Research技术洞察

原文链接：https://zhuanlan.zhihu.com/p/1939003557341802752

---

​

目录

## 1 概述

[OpenAI Deep Research](https://zhida.zhihu.com/search?content_id=261657388&content_type=Article&match_order=1&q=OpenAI+Deep+Research&zhida_source=entity)（DR，深度研究）智能体于2025年2月正式发布，旨在利用先进的推理能力综合海量的在线信息，为用户自动化完成多步骤研究任务。其核心是基于[o3推理模型](https://zhida.zhihu.com/search?content_id=261657388&content_type=Article&match_order=1&q=o3%E6%8E%A8%E7%90%86%E6%A8%A1%E5%9E%8B&zhida_source=entity)进行微调，并在2025年4月推出基于o4-mini的轻量化版本。OpenAI DR目前是一款商业化闭源产品，作为OpenAI的下一代智能体，能够根据用户的一个提示，主动搜索、阅读、分析和综合数百个在线资源，生成研究分析师水准、附有清晰引用的综合报告，适用于金融、科学和法律等需要高强度知识工作的领域。其在[GAIA基准](https://zhida.zhihu.com/search?content_id=261657388&content_type=Article&match_order=1&q=GAIA%E5%9F%BA%E5%87%86&zhida_source=entity)平均正确率达到72.57%，Humanity's Last Exam基准正确率为26.6%。

## 2 技术方案

从技术架构看，Deep Research基于经强化学习微调的o3模型和[ReAct模式](https://zhida.zhihu.com/search?content_id=261657388&content_type=Article&match_order=1&q=ReAct%E6%A8%A1%E5%BC%8F&zhida_source=entity)构建，能够实现精准溯源和研究。其关键技术包括：

  * 采用**前置Agent + 主Agent** （推测）协同模式，前置Agent负责与用户交互，澄清与细化研究需求；主Agent则基于完整明确的需求，执行核心研究任务。
  * 主Agent所使用的基座模型，在o3模型基础上针对网络浏览和数据分析进行了优化，并采用**强化学习** 进行训练。
  * 使用**ReAct模式** 搜索实时信息，过程中可以实时的调整下一步行动，能够灵活适应中间搜索结果的变化，提升最终报告中内容的准确性。



## 3 能力解析

Deep Research的核心能力包含如下4个维度，在此将会从这几个维度对OpenAI Deep Research的能力进行解析。

  * **研究计划制定** ：根据任务目标，​​自主规划并执行多步骤研究流程​​。从各种在线来源（包括新闻文章、学术论文、代码库等）收集信息，进行分析和综合，并迭代地构建答案，遇到新的或矛盾的信息时，动态地调整其研究策略。同时，采用ReAct模式搜索实时信息，过程中可以实时调整下一步行动。
  * **工具使用与环境交互** ：使用搜索引擎、网页抓取等工具获取信息，支持在沙盒环境中执行代码，用于执行计算、进行数据分析和绘制图表。
  * **内容评估与报告生成** ：整合多个数据源信息，在信息冲突时主动判断来源准确性，输出报告中附有明确的来源引用。
  * **多角色Agent协同** ：通过前置Agent和主Agent分工协作，分别负责用户需求澄清和深度研究主要过程，有效地提升综合效果。



## 4 总结

Deep Research基于o3推理模型，凭借其强大的多源信息综合能力，高度自动化的研究流程和多工具动态调度能力，实现了复杂研究报告的自主生成。使其在深度研究领域脱颖而出，在深度研究自动化领域展现出突出潜力。然而，其在​运行成本、信息准确性和“幻觉”、透明度和可控性方面仍需持续优化。

## 相关链接

  1. [HTTPS://openai.com/index/introducing-deep-research/](https://link.zhihu.com/?target=HTTPS%3A//openai.com/index/introducing-deep-research/)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Deep Research】02-Deep Research智能体：系统性洞察与分析](https://zhuanlan.zhihu.com/p/1938525208395879422)  
> 下一篇：[【Deep Research】04-Gemini Deep Research技术洞察](https://zhuanlan.zhihu.com/p/1940785645107812348)
