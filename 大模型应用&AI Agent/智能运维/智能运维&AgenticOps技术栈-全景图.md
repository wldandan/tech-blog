# 智能运维&AgenticOps技术栈-全景图

原文链接：https://zhuanlan.zhihu.com/p/1995167173094695116

---

​

目录

## 概述

随着人工智能技术的迅猛发展，特别是[大语言模型](https://zhida.zhihu.com/search?content_id=269064224&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）和AI Agent技术的突破性进展，运维领域正在经历一场深刻的变革。运维体系的演进，是一部从“人力密集型”的被动响应走向智能化“自主管理”的进化史。它先后经历了传统人工运维、自动化工具、基于规则和机器学习的[AIOps](https://zhida.zhihu.com/search?content_id=269064224&content_type=Article&match_order=1&q=AIOps&zhida_source=entity)等阶段，最终在大模型与AI Agent技术的驱动下，迈入了自主决策的智能化新纪元。

## 智能运维技术栈全景图

[LLM智能体](https://zhida.zhihu.com/search?content_id=269064224&content_type=Article&match_order=1&q=LLM%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)的兴起正推动智能运维从“辅助分析”向主动的、以目标为导向的“自主智能”深刻演进，实现真正的“自愈合”系统。传统AIOps依赖机器学习进行异常检测与关联分析，而LLM智能体则实现了“感知-思考-行动”闭环的全面自动化，显著提升运维效率与系统可靠性。

  * **在异常感知阶段** ，LLM智能体利用动态基线与LLM持续分析多源海量数据，突破静态阈值告警的限制，更早识别潜在故障信号。
  * **在[根因分析](https://zhida.zhihu.com/search?content_id=269064224&content_type=Article&match_order=1&q=%E6%A0%B9%E5%9B%A0%E5%88%86%E6%9E%90&zhida_source=entity)阶段**，LLM智能体能够跨系统融合多源信息，通过[知识图谱](https://zhida.zhihu.com/search?content_id=269064224&content_type=Article&match_order=1&q=%E7%9F%A5%E8%AF%86%E5%9B%BE%E8%B0%B1&zhida_source=entity)构建服务拓扑关系，自动识别数据间的关联逻辑，将根因分析从“相关性”提升到“因果性”层面，将原本耗时数小时甚至数天的调查缩短至分钟级。
  * **在辅助修复阶段** ，其自动化能力已发展出五个层级：从辅助提问、缓解方案生成、命令推荐，直至自动执行修复操作（如重启服务、回滚部署等），推动运维工作从重复任务中解放。
  * **在故障预防方面** ，LLM智能体通过持续学习系统运行规律，结合历史数据、业务活动等因素，能够实现故障的预测性预警，将运维模式从“事后修复”推向“主动防御”，显著提升系统稳定性。



如下所示，我们根据洞察分析，将智能运维技术栈全景定义为如下几层：

本系列将会对智能运维技术栈中的智能体系统和关键能力进行分析，帮助大家由浅入深的学习智能运维的核心技术和前沿进展。

## 1 智能体系统

  1. [【运维智能体】01-微软AIOpsLab，集成运维Agents设计、开发和评估的整体框架](https://zhuanlan.zhihu.com/p/1976373761038124600)
  2. [【运维智能体】02-Ask Red Hat，提供技术问题解答和故障排除指导的AI助手](https://zhuanlan.zhihu.com/p/1976619770745991853)
  3. [【运维智能体】03-Gemini Cloud Assist，谷歌云的云原生应用全生命周期助手](https://zhuanlan.zhihu.com/p/1977059222085730460)
  4. [【运维智能体】04-IBM Instana，通过Agentic AI构建全栈观测能力](https://zhuanlan.zhihu.com/p/1977456614299693732)
  5. [【运维智能体】05-Davis® AI，因果、预测和生成式人工智能技术的结合的智能运维技术](https://zhuanlan.zhihu.com/p/1977762826962625486)
  6. [【运维智能体】06-Truefoundry，MLOps+LLMOps+InfraDevOps企业级可治理的平台](https://zhuanlan.zhihu.com/p/1979247270974223914)
  7. [【运维智能体】07-DeepFlow智能体，依托大模型自主实现业务连续性保障，提升运维效率](https://zhuanlan.zhihu.com/p/1980318597487301073)
  8. [【运维智能体】08-Microsoft Argos，基于LLM构建可解释的异常检测规则](https://zhuanlan.zhihu.com/p/1981325904736192461)
  9. [【运维智能体】09-OpenDerisk，AI驱动的工业级SRE框架](https://zhuanlan.zhihu.com/p/1982122571202859533)
  10. [【运维智能体】10-STRATUS：面向现代云平台自主可靠性工程的多智能体系统](https://zhuanlan.zhihu.com/p/1987601362868011813)
  11. [【运维智能体】11-AWS DevOps Agent：自主云运维的未来](https://zhuanlan.zhihu.com/p/1988179556159485443)
  12. [【运维智能体】12-RCAgent：基于工LLM 自主智能体的云平台根因分析](https://zhuanlan.zhihu.com/p/1989415079091909901)
  13. [【运维智能体】13-AgentFM：一个基于角色实现分布式数据库故障管理的多智能体框架](https://zhuanlan.zhihu.com/p/1999846839147598355)



## 2 智能运维能力

### 2.1 智能运维综述

  1. [【智能运维】01-LLM时代下的智能运维](https://zhuanlan.zhihu.com/p/1997334170842711253)



### 2.2 日志解析

### 2.3 异常检测

  1. [【异常检测】01-MonitorAssistant：通过大语言模型简化云服务监控](https://zhuanlan.zhihu.com/p/1983902538748143188)



### 2.4 故障诊断

  1. [【故障诊断】01-评估与总结：利用大语言模型提升对中断的理解](https://zhuanlan.zhihu.com/p/1984664974266737696)
  2. [【故障诊断】02-基于LLM的故障定位框架分析](https://zhuanlan.zhihu.com/p/2003143062445105956)
  3. [【故障诊断】03-利用多智能体、大/小语言模型实现故障的自动化定位](https://zhuanlan.zhihu.com/p/2004925791880896524)



### 2.5 根因分析

  1. [【根因分析】01-LM-PACE：基于LLM的置信度估计，助力云故障高效根因定位](https://zhuanlan.zhihu.com/p/1991546618449773981)


