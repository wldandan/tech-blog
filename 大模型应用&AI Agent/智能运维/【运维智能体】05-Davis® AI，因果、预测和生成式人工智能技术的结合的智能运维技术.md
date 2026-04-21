# 【运维智能体】05-Davis® AI，因果、预测和生成式人工智能技术的结合的智能运维技术

原文链接：https://zhuanlan.zhihu.com/p/1977762826962625486

---

​

目录

## 1 概述

Davis® AI是Dynatrace于2019年11月推出的业界首款闭源超模态AI，它将预测性AI、因果关系AI和生成式AI相结合，通过对数据的转换和扩充，为IT运维、安全等领域提供精准预测、问题根源分析、自动化建议等智能服务。

通过部署Dynatrace的统一可观测性与安全平台，TD Bank在数字化转型中成功突破了监控工具互不兼容、运维效率低、交易故障难以及时定位等瓶颈：运维效率提升75%，监控成本降低45%，交易故障定位能力得到显著优化。

## 2 技术方案

Davis® AI架构分数据支撑层，AI引擎层和应用能力层共三层。

**上层**

应用能力层直接面向用户需求，输出可直接使用的智能运维成果，覆盖问题与风险识别、[AIOps](https://zhida.zhihu.com/search?content_id=266967190&content_type=Article&match_order=1&q=AIOps&zhida_source=entity) 安全运维、根因定位、生产力建议、自动化执行及入门指导等典型场景。

  * 该层支持用户通过自然语言输入，借助Auto-Prompt机制自动生成操作提示，并直接实现DQL数据查询、图表生成、工作流构建等功能，有效降低技术门槛。



**中层**

AI引擎层作为系统核心，承担数据分析、推理与价值转化的关键任务，依托多模态AI协同实现智能驱动。其核心AI技术包括：

  * **预测性AI** （Predictive AI）：实现多维基线、应用流量与服务负载的持续预测与异常预警；
  * **因果AI** （Causal AI）：结合拓扑上下文，分析可观测性与安全数据，自动完成异常分组、根因定位与业务影响排序；
  * **生成式AI** （Generative AI）：通过Davis CoPilot，基于前两者输出生成查询语句、仪表板和工作流，并提供自然语言交互能力。



辅助支撑技术包括机器学习、Smartscape拓扑技术、语义词典与上下文编辑功能，增强AI对IT环境的理解；同时内置PII与GDPR合规机制，保障数据处理的安全合规。

**下层**

数据支撑层为上层AI分析提供全量、高质量数据基础，具备以下能力：

  * **数据源覆盖** ：整合拓扑（Topology）、追踪（Traces）、指标（Metrics）、日志（Logs）、行为（Behavior）、代码（Code）、元数据（Metadata）、网络（Network）等全链路IT数据。
  * **数据存储与处理** ：基于[Grail™数据湖仓](https://zhida.zhihu.com/search?content_id=266967190&content_type=Article&match_order=1&q=Grail%E2%84%A2%E6%95%B0%E6%8D%AE%E6%B9%96%E4%BB%93&zhida_source=entity)实现统一存储，结合大语言模型，支持无需索引的即时查询与智能分析，无需复杂配置即可高效处理海量数据。



## 3 关键能力解析

Davis具备异常检测与自动根因分析能力，并集成智能问答功能，帮助用户快速识别、定位并理解系统问题.

**智能运维**

Davis的智能运维包含异常检测和自动根因分析能力：

  * 异常检测：支持基于静态阈值、自适应阈值或更复杂的AI模型方法，实时检测系统异常行为。
  * 根因分析：能够自动识别影响客户的问题，并借助拓扑、事务及代码级信息，准确定位问题根源。Davis通过以下流程实现自动根因分析： 事件摄取 → 事件归一化 → 拓扑创建 → 聚合 → 去重 → 根本原因分析。分析完成后，Davis会以可视化图谱形式呈现根因分析结果，提升调优过程的可解释性，避免传统“黑盒”操作。



**智能问答**

基于Davis CoPilot实现，该组件依托大型语言模型（LLM）构建，能够根据用户输入和上下文生成概率性响应，通过预测最可能的文本序列进行回答。

Davis CoPilot采用检索增强生成（RAG）方法，为底层LLM补充相关上下文信息，将自然语言转化为DQL查询，具备NL2DQL（自然语言转DQL）与DQL2NL（DQL转自然语言）的双向能力，帮助用户无需直接编写查询语句即可访问底层数据。

## 4 总结

基于Dynatrace的Davis® AI的智能运维实践显示，未来智能运维将深度融合多模态AI，向高度自动化和人性化交互方向发展。其整合了预测性AI、因果AI与生成式AI，构建了从数据、分析到应用的三层架构，实现了精准预警、根因定位和自动化建议，并在TD Bank的实践中有效提升了运维效率、降低了成本。未来，智能运维将更注重自动化闭环、跨域安全协同及基于业务影响的决策，通过清晰的根因分析和人性化交互降低技术门槛，逐步向自主预防、自我优化的“无人值守”运维目标演进。

## 相关链接

  1. [https://docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai](https://link.zhihu.com/?target=https%3A//docs.dynatrace.com/docs/discover-dynatrace/platform/davis-ai)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【运维智能体】04-IBM Instana，通过Agentic AI构建全栈观测能力](https://zhuanlan.zhihu.com/p/1977456614299693732)  
> 下一篇：[【运维智能体】06-Truefoundry，MLOps+LLMOps+InfraDevOps企业级可治理的平台](https://zhuanlan.zhihu.com/p/1979247270974223914)
