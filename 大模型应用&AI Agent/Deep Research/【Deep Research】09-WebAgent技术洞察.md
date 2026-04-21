# 【Deep Research】09-WebAgent技术洞察

原文链接：https://zhuanlan.zhihu.com/p/1947233864620709465

---

​

目录

## 1 概述

WebAgent是阿里巴巴集团[通义实验室](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=%E9%80%9A%E4%B9%89%E5%AE%9E%E9%AA%8C%E5%AE%A4&zhida_source=entity)于2025年1月开源的多个自主搜索智能体，包括WebWalker（2025年1月）、WebDancer（2025年5月）、WebSailor（2025年7月）和WebShaper（2025年7月），迄今为止Github Star数已达5k+。其能够主动整合不同文献或信息源中的观点，通过多步推理生成全面且精准的分析报告。

  * WebAgent系列在技术架构方面，从早期多Agent协同架构演进为训练出具备Agentic能力的[ReAct智能体](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=ReAct%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)，实现自主推理与执行。。
  * 在QA数据集构建方面，经历了从[CRAWLQA](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=CRAWLQA&zhida_source=entity)（模拟人类浏览行为）和[E2HQA](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=E2HQA&zhida_source=entity)（简单QA迭代复杂化）的数据合成到可扩展图的数据合成再到形式化驱动的数据合成，打造高不确定性的复杂任务数据集，提升Agent复杂推理能力；
  * 在训练范式方面，涵盖[SFT+RL](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=SFT%2BRL&zhida_source=entity)的微调方式和RFT冷启动+RL的Post Training方式，最终逐步提升模型在动态环境中的适应性、泛化能力和复杂推理能。
  * 在开源的内容方面，主要集中在前向推理与评估层面，数据集构建的完整实现细节与训练流程尚未开源。



## 2 技术方案

从技术架构看，WebAgent涵盖从数据到训练的全流程创新。关键技术包括：

  * **技术架构** ：WebWalker输入给定的网址和问题，Agent对网站进行充分的探索和挖掘并回答问题，采用Explorer+Critic的多智能体架构。WebDancer、WebSailor和WebShaper则实现自主推理与执行，采用高不确定性任务数据集+训练优化的方式直接得到具备Agentic能力的ReAct智能体，构建“思考-行动-观察”的循环机制。
  * **QA数据集构建与轨迹采样** ：爬取网页构建浏览数据集，提升Agent复杂推理能力。WebDancer采用CRAWLQA和E2HQA方式生成复杂QA对，WebSailor通过可扩展的图合成数据得到[SailorFog-QA](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=SailorFog-QA&zhida_source=entity)数据集，WebShaper通过形式化驱动数据合成框架（种子问题构建+扩展器ReAct智能体）生成QA对。经过LLM/LRM/RFT等方式重构和筛选高质量训练轨迹样本。
  * **模型训练创新** ：WebDancer和WebShaper均采用SFT+RL的双阶段训练方式，在RL阶段WebDancer使用[DAPO算法](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=DAPO%E7%AE%97%E6%B3%95&zhida_source=entity)，WebShaper使用[GRPO算法](https://zhida.zhihu.com/search?content_id=262678739&content_type=Article&match_order=1&q=GRPO%E7%AE%97%E6%B3%95&zhida_source=entity)，以优化决策和泛化能力。WebSailor采用RFT冷启动+DUPO的RL的Post Training方式，提升模型在动态环境中的适应性、泛化能力和复杂推理能。



WebDancer

WebWalker

## 3 能力解析

  * **研究计划制定** ：根据任务目标自主决策，判断下一步该访问哪个网页、执行搜索或调用工具，并根据返回结果决定是否调整路径，交替进行思考-行动-观察，形成一个自主推理与执行的闭环。
  * **工具使用与环境交互** ：WebWalker只集成了visit_page工具，WebDancer和WebSailor集成了search和visit两种基本信息检索工具。适应开放动态环境，支持自主修正。
  * **内容评估与报告生成** ： WebAgent侧重自主完成复杂信息检索，但其具备的多步推理能力和迭代优化策略使其能够动态调整策略并整合不同观点生成最终研究报告。
  * **多角色Agent协同** ：WebWalker采用Explorer+Critic的多Agent架构。WebDancer、WebSailor和WebShaper则是训练出具备agentic能力的ReAct智能体，实现自主推理与执行。



## 4 总结

WebAgent旨在构建基于Web浏览器的原生Agentic模型，通过模拟人类在网络中的感知、决策和行动循环，支持智能体自主访问、筛选和整合信息，具备端到端信息检索与多步推理能力，最终生成全面且精准的分析报告。

## 相关链接

  1. [https://github.com/Alibaba-NLP/WebAgent](https://link.zhihu.com/?target=https%3A//github.com/Alibaba-NLP/WebAgent)
  2. [https://developer.aliyun.com/article/1667448](https://link.zhihu.com/?target=https%3A//developer.aliyun.com/article/1667448)
  3. [https://developer.aliyun.com/article/1672146](https://link.zhihu.com/?target=https%3A//developer.aliyun.com/article/1672146)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Deep Research】08-DeerFlow技术洞察](https://zhuanlan.zhihu.com/p/1946890604698146510)
