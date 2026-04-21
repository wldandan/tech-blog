# 【根因分析】02-COCA：融合代码知识的分布式系统生成式根因分析

原文链接：https://zhuanlan.zhihu.com/p/2017271940709101923

---

​

目录

## 一、背景和挑战

[分布式系统](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F&zhida_source=entity)故障频发，用户常在GitHub、JIRA等平台提交问题报告，但这些报告复杂度高，导致人工RCA耗时费力、易出错。因此，自动化[根因分析](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=%E6%A0%B9%E5%9B%A0%E5%88%86%E6%9E%90&zhida_source=entity)（RCA）对保障系统可靠性至关重要，但主流方法依赖完备的运行时监控数据，而这类数据在问题平台难以获取。虽然近期研究利用[大语言模型](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLMs）分析报告，但仍受限于用户信息的不完整与模糊等。当前，RCA存在局限如下：

  * **传统监控驱动的RCA** ：依赖日志、指标、链路等全量运行时数据，需要**正常/故障状态的对比数据** ，而问题平台通常不提供这类数据，无法落地。
  * **现有LLM-based RCA** ：仅基于问题报告和历史案例做分析，受限于用户提交的**信息不完整、模糊，缺少故障背后的执行逻辑与系统代码上下文** ，难以精准定位根因。
  * **传统故障定位技术** ：仅能定位可复现的代码Bug，无法处理分布式系统中**网络、竞态、配置等非代码类故障** ，且依赖Bug复现与运行时覆盖率，在issue场景下不适用。



因此，研究者们在论文中提出了**[COCA](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=COCA&zhida_source=entity)** ，一种面向问题报告、基于代码知识增强的RCA方法。

  1. 首先，COCA会基于问题报告中的数据，智能提取相关代码片段并重构执行路径，为后续的根因分析提供完整的执行上下文。
  2. 随后，其会构建融合历史问题报告与提取的代码知识的提示词，驱动LLM生成详细的根因总结，并定位故障责任组件。



## 二、COCA核心框架与实现

COCA以问题报告 + 项目代码为输入，通过四个核心阶段完成端到端的根因分析，针对性解决上述三大挑战：

  1. **日志源码检索阶段** ：COCA 根据问题报告中的日志消息，定位到对应的日志语句行号，并映射出系统中的相关代码点位。
  2. **[执行路径重构](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=%E6%89%A7%E8%A1%8C%E8%B7%AF%E5%BE%84%E9%87%8D%E6%9E%84&zhida_source=entity) 阶段**：基于识别出的代码点位，结合 RPC 调用图和方法间分析，重构故障发生前的执行路径。
  3. **[故障相关代码画像](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=%E6%95%85%E9%9A%9C%E7%9B%B8%E5%85%B3%E4%BB%A3%E7%A0%81%E7%94%BB%E5%83%8F&zhida_source=entity) 阶段**：COCA 对带有执行顺序的方法级代码片段进行分析和索引，生成包含方法签名和文档的索引信息。大语言模型根据初步问题理解与索引信息，检索完整的方法代码。
  4. **[根因推理](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=%E6%A0%B9%E5%9B%A0%E6%8E%A8%E7%90%86&zhida_source=entity) 阶段**：结合相似的历史问题报告，COCA 利用所有已获取的知识进行问题诊断，输出全面的根因总结和故障组件定位结果。



### 2.1 日志源检索（Logging Source Retrieval）

解决日志与代码的匹配难题，分为两步：

  * **日志语句还原** ：基于静态分析的回溯方法，解析数据依赖与控制流，还原源码中由动态变量拼接的日志语句，生成各分支的原始日志模板。
  * **日志模板匹配** ：针对日志消息可能匹配多个模板的问题，COCA采用前缀树进行全局非剪枝递归匹配，并优先选择静态部分最多的模板，以确保准确匹配到对应的日志记录语句，以锁定故障相关的代码点位。



### 2.2 执行路径重构（Execution Path Reconstruction）

解决分布式执行路径还原难题：

  * 基于锁定的代码点位，构建过程间控制流图（ICFG），通过方法间/方法内分析还原本地调用的执行链路。
  * 提出**[RPCBridge](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=RPCBridge&zhida_source=entity)** 核心创新：解析gRPC/Thrift等RPC框架的IDL文件，识别客户端RPC调用，匹配服务端实现类，桥接跨节点的RPC调用边，补全分布式场景下断裂的调用栈，还原跨服务的完整执行路径。



### 2.3 故障相关代码画像（Failure-related Code Profiling）

解决代码上下文过载问题，分为两步：

  * **代码片段索引** ：对执行路径中的所有方法，基于标准方法文档和Soot生成方法签名，构建轻量索引。若方法缺少文档，则通过LLM生成标准化文档。
  * **代码片段检索** ：将问题报告、析后的执行路径与代码索引输入LLM，由模型识别与故障相关的代码片段，返回需详细审查的方法签名。据此提取对应方法代码，供后续阶段使用。



### 2.4 根因推断（Root Cause Inference）

完成最终的RCA输出：

  * **上下文学习** ：采用上下文学习（ICL）策略，通过BM25算法召回与当前issue最相似的5个历史故障案例，为模型提供诊断经验。
  * **诊断结果推理** ：整合issue报告、重构的执行路径、检索到的核心代码片段，完成根因推理。COCA聚焦两大核心任务：**根因总结** （为故障缓解与修复提供简洁清晰的指导）、**根因定位** （按可能性排序输出故障责任组件，明确实际根因的分析范围）。



## 三、实验验证与核心结果

### 3.1 实验设置

  * **数据集** ：覆盖5个主流开源分布式系统（MapReduce、HDFS、HBase、Cassandra、ZooKeeper），共106个经过标注的真实生产issue，代码规模从184K到1.1M行不等，覆盖数据丢失、系统卡死、状态不一致、结果错误、资源泄漏等7类核心故障。
  * **基线方法** ：业界SOTA的LLM-based RCA方案RCACopilot、ReAct，以及纯基础LLM，所有方法统一使用相同骨干模型保证公平对比。
  * **评估指标** ： 
    * 根因总结：BLEU-4、ROUGE-1、METEOR、语义相似度、人工评估的有用性。
    * 根因定位：Exact Match（精准匹配）、Top-3/Top-5召回率。



### 3.2 核心实验结果

**整体性能领先SOTA** （以[GPT-4o](https://zhida.zhihu.com/search?content_id=271608282&content_type=Article&match_order=1&q=GPT-4o&zhida_source=entity)为骨干模型）

  * 根因总结：相比最优基线RCACopilot，BLEU-4平均提升22.0%，ROUGE-1提升6.8%，METEOR提升10.6%，语义相似度与人工有用性评分均全面领先。
  * 根因定位：相比最优基线RCACopilot，Exact Match平均提升28.3%，Top-3提升20.7%，Top-5提升4.3%；相比纯基础模型，Exact Match提升高达56.3%。



**总结** ：通过将代码知识融入根因分析流程，COCA在根因总结与根因定位两个维度均展现出了卓越的性能，显著优于所有基线方法，尤其是基础模型。

**强泛化性**

在GPT-3.5、LLaMa-3.1-405b、Gemini-1.5-Pro、Claude-3.5-Sonnet等主流LLM上，COCA均能带来显著且稳定的性能提升：根因定位Exact Match平均提升43.3%，根因总结BLEU-4平均提升33.2%，证明框架与骨干模型解耦，可适配各类大语言模型。

**消融实验验证**

四个核心模块缺一不可，移除任一模块都会导致性能显著下降；2阶（COCA的默认设置）的执行路径深度达到最优效果，1阶会遗漏关键信息，3阶会引入过多无关代码；RPCBridge覆盖了重构路径中约15.7%的RPC调用，是分布式场景路径还原的关键。

## 四、总结

COCA是首次将代码知识融入分布式系统问题报告的自动化根因分析框架。实验结果表明，COCA的性能显著优于所有基线方法，且能泛化到不同的LLM中。其核心贡献如下：

  * 首次将代码知识系统性地融入分布式系统issue报告的自动RCA框架，填补了现有方案缺少系统内部执行上下文的空白。
  * 设计了日志还原匹配、RPCBridge、代码画像检索等一系列创新组件，系统性解决了代码知识融入RCA的核心挑战。
  * 通过真实案例与对照实验证明，COCA可将分布式系统资深维护人员需要数小时完成的故障诊断工作，压缩至平均19.4秒完成，大幅降低了分布式系统的运维诊断成本，具备直接落地到云平台、大规模分布式系统生产运维场景的实用价值。



## 相关链接

  1. [https://arxiv.org/pdf/2503.23051](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2503.23051)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【根因分析】01-LM-PACE：基于LLM的置信度估计，助力云故障高效根因定位](https://zhuanlan.zhihu.com/p/1991546618449773981)
