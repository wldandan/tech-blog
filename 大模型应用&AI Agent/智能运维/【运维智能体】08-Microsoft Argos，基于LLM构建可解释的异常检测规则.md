# 【运维智能体】08-Microsoft Argos，基于LLM构建可解释的异常检测规则

原文链接：https://zhuanlan.zhihu.com/p/1981325904736192461

---

​

目录

## 1 概述

云服务的可靠性与可用性对业务至关重要，而时序指标异常检测是保障云服务健康的核心手段。但现有系统普遍存在无法同时满足可解释性、可复现性、自主性三大生产级需求的问题。为此，华盛顿大学与[微软研究院](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=%E5%BE%AE%E8%BD%AF%E7%A0%94%E7%A9%B6%E9%99%A2&zhida_source=entity)联合推出研究性的项目[Argos系统](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=Argos%E7%B3%BB%E7%BB%9F&zhida_source=entity)。

Argos一个基于[大语言模型](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)的智能时间序列异常检测系统，其核心创新在于通过[LLM](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=LLM&zhida_source=entity)自主生成可解释和可重复的[异常检测规则](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=%E5%BC%82%E5%B8%B8%E6%A3%80%E6%B5%8B%E8%A7%84%E5%88%99&zhida_source=entity)。​该系统的主要功能包括：自主规则生成、迭代反馈优化、模型融合以及高效推理。通过这些功能，Argos能够显著提升异常检测的准确性和效率，同时保持检测结果的可解释性，这对于需要审计和合规的企业环境尤为重要。

## 2 技术方案

Argos采用 “数据预处理→规则训练→部署” 三阶段流程，核心是[多智能体协作](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E5%8D%8F%E4%BD%9C&zhida_source=entity)的规则优化pipeline与模型融合策略。

**数据预处理阶段**

对输入时序数据执行三步处理：

  1. **缩放与索引** ：将数值缩放到指定有效数字，移除原始时间戳并按顺序重索引；
  2. **分块** ：按数据集校准的块大小分割（KPI=2500，Yahoo=500，内部 = 1000），平衡上下文与LLM窗口限制；
  3. **[Tokenizer适配](https://zhida.zhihu.com/search?content_id=267349632&content_type=Article&match_order=1&q=Tokenizer%E9%80%82%E9%85%8D&zhida_source=entity)** ：对GPT类模型，在数值digits间加空格，避免Token分割偏差。



**规则训练阶段：多智能体迭代优化**

训练引擎包含三个核心智能体，通过反馈循环提升规则质量：

  * **检测智能体（Detection Agent）** ：基于模板生成Python规则函数，要求先注释 “正常模式” 与 “异常规则”，再写代码。
  * **修复智能体（Repair Agent）** ：执行规则代码，基于错误日志（如语法错误、运行时错误）修正规则，直至无语法错误。
  * **审查智能体（Review Agent）** ：在验证集上评估规则精度，若精度低于上一迭代，提供 “精度对比 + 代码差异 + 错误样本”，指导规则优化。



此外，训练阶段还包含：

  * **Top-k 规则选择** ：每次迭代生成n个规则，选k个精度最高的进入下一迭代（模拟beam search），加速收敛。
  * **规则分类训练** ：分别针对现有检测器的 假阴性（FN）和假阳性（FP）样本训练规则，避免过拟合（训练时混入真实负/正样本）。



**部署阶段：模型融合保证精度**

通过聚合器（Aggregator）融合“现有基线检测器”与“LLM生成的FN/FP规则”，按“基线标注正常但假阴性规则标注异常则修正为异常、基线标注异常但假阳性规则标注正常则修正为正常” 的逻辑修正结果，完成云基础设施时序数据的精准异常检测。

## 3 关键能力解析

将大语言模型生成的规则与传统的基于规则的方法相结合，可以在保持可重复性的同时，弥补自主性与可解释性之间的差距。

  * **在训练阶段使用LLM识别异常规则** ：现有基于LLM方法在检测阶段，存在随机性且缺乏可重复性，而此方法生成的python脚本具有可解释性和可重复性。
  * **迭代检查以提高准确性** ：检测规则转化成可执行代码，加以语法检测。
  * **提高效率** ：Top K或其他策略如时间。
  * **将LLM与传统方法聚合** ：LLM本身具有随机性，难以保证生成的规则优于现有的生产级检测系统。



## 4 总结

Argos实现了“可解释性、可复现性、自主性”三大生产核心属性的统一 —— 通过LLM生成带自然语言注释的可执行Python规则保障可解释性，以确定性代码执行确保可复现性，依托“检测 - 修复 - 审查”多智能体pipeline实现规则自动生成与优化，无需人工干预。其基于大语言模型的自主规则生成能力代表了技术发展的前沿方向，为企业的运维监控提供了全新的思路。

## 相关链接

  1. [https://arxiv.org/pdf/2501.14170](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2501.14170)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【运维智能体】07-DeepFlow智能体，依托大模型自主实现业务连续性保障，提升运维效率](https://zhuanlan.zhihu.com/p/1980318597487301073)  
> 下一篇：[【运维智能体】09-OpenDerisk，AI驱动的工业级SRE框架](https://zhuanlan.zhihu.com/p/1982122571202859533)
