# 【运维智能体】07-DeepFlow智能体，依托大模型自主实现业务连续性保障，提升运维效率

原文链接：https://zhuanlan.zhihu.com/p/1980318597487301073

---

​

目录

## 1 概述

[DeepFlow智能体](https://zhida.zhihu.com/search?content_id=267256951&content_type=Article&match_order=1&q=DeepFlow%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)是云杉网络于2025年2月发布的智能运维产品，该智能体能够自主感知环境、推理决策并执行任务，通过使用DeepFlow提供的各类可观测性工具，自主完成保障业务连续性的工作。

在金融、电信、电力、智能制造等要求高可靠的行业，DeepFlow智能体已展现出卓越价值——从加速核心系统上线，到避免关键业务雪崩，再到突发情况应对，提供预防为主、快速止血和及时决策的全方位保障。

## 2 技术方案

DeepFlow智能体的架构涵盖交互层、感知层、推理层和执行层，通过[零侵扰采集技术](https://zhida.zhihu.com/search?content_id=267256951&content_type=Article&match_order=1&q=%E9%9B%B6%E4%BE%B5%E6%89%B0%E9%87%87%E9%9B%86%E6%8A%80%E6%9C%AF&zhida_source=entity)、思维链状态机技术和[自适应感知技术](https://zhida.zhihu.com/search?content_id=267256951&content_type=Article&match_order=1&q=%E8%87%AA%E9%80%82%E5%BA%94%E6%84%9F%E7%9F%A5%E6%8A%80%E6%9C%AF&zhida_source=entity)解决了可观测性、幻觉问题和成本效益的技术挑战。

  * **交互层** ：实现用户与DeepFlow智能体的交互。交互层包含如下组件，每个组件均可为客户提供量身定制。
  * **感知层** ：为智能体提供对外部环境感知能力。DeepFlow智能体的感知层，通过按需和实时分析环境中的可观测性数据，实现对业务运行状态的全面感知。感知层性能主要受制于数据分析能力，可以通过引入感知加速进行水平扩展。
  * **推理层** ：是DeepFlow智能体的大脑，包含一系列规划和记忆系统。推理层性能主要受制于模型性能，DeepFlow智能体可通过增加AI算力提供推理层性能扩展。
  * **执行层** ：DeepFlow智能体通过执行层为业务提供执行建议或方案。执行层若对接控制层，可实现业务稳定性的全自动保障。



## 3 关键能力解析

  * **分钟级诊断** ：基于多维数据的实时关联分析，结合[故障模式库](https://zhida.zhihu.com/search?content_id=267256951&content_type=Article&match_order=1&q=%E6%95%85%E9%9A%9C%E6%A8%A1%E5%BC%8F%E5%BA%93&zhida_source=entity)与知识图谱，实现快速故障定位与推理。 
    * 多维数据实时关联分析：依托历史经验匹配故障模式，快速定位故障传播路径与影响范围。
    * 故障模式库和知识图谱：建立故障症状、原因、解决措施间的关联关系，通过推理模型实现路径快速检索与智能推理。
  * **不间断巡检** ：7x24小时持续监控业务健康状态，实时预警业务风险，快速关联分析业务告警。 
    * 对业务健康度进行7x24检查：实时发现核心指标异常，同步分析各组件运行状态。
    * 对业务风险即时预警：基于时序数据建模，开展预测性分析，识别潜在趋势风险。
    * 对业务告警进行快速关联分析：结合故障传播分析与资源依赖分析，精确定位问题根源。
  * **一句话问数** ：整合多源实时数据，结合自然语言理解与意图识别。 
    * 多源数据实时整合：通过智能化数据特征提取，自动生成复杂查询逻辑。
    * 自然语言理解意图识别：结合业务与用户上下文精准理解意图，有效抑制通用大模型幻觉。



**核心技术**

面对可观测性问题、幻觉问题和成本问题，DeepFlow智能体通过以下几个原创技术解决：

  * **零侵扰采集** ：通过融合cBPF、eBPF、[Wasm](https://zhida.zhihu.com/search?content_id=267256951&content_type=Article&match_order=1&q=Wasm&zhida_source=entity)等技术，实现对大规模分布式业务和基础设施的零侵扰数据采集，解决DeepFlow智能体运行环境的完全可观测性问题。
  * **思维状态机** ：通过思维链（Chain of Thought）指引，解决由大模型推理带来的幻觉问题。
  * **自适应感知** ：自适应感知技术实现了推理前感知和推理中感知的混合感知技术。



## 4 总结

DeepFlow智能体具备自主感知、推理与执行能力，集成多项核心技术，可以有效应对可观测性、大模型幻觉及运维成本等挑战。其多层架构支持分钟级故障诊断、持续业务巡检和自然语言交互等功能，已在金融、电力等高要求行业实现以预防为主、快速响应的业务保障。未来，智能运维将向更自主、自适应方向发展，推动运维模式从“被动响应”转向“主动预测与预防”，持续提升系统稳定性和运营效率。

## 相关链接

  1. [https://www.yunshan.net/news/detail/42180](https://link.zhihu.com/?target=https%3A//www.yunshan.net/news/detail/42180)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【运维智能体】06-Truefoundry，MLOps+LLMOps+InfraDevOps企业级可治理的平台](https://zhuanlan.zhihu.com/p/1979247270974223914)  
> 下一篇：[【运维智能体】08-Microsoft Argos，基于LLM构建可解释的异常检测规则](https://zhuanlan.zhihu.com/p/1981325904736192461)
