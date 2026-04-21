# LangGraph Cloud，一站式闭环Agent开发、调试和监控的基础设施

原文链接：https://zhuanlan.zhihu.com/p/706135365

---

​

目录

[LangChain](https://zhida.zhihu.com/search?content_id=245049981&content_type=Article&match_order=1&q=LangChain&zhida_source=entity)社区于6月24日发布了新的产品更新。在该版本中，主要推出了[LangGraph Cloud](https://zhida.zhihu.com/search?content_id=245049981&content_type=Article&match_order=1&q=LangGraph+Cloud&zhida_source=entity)封闭测试版、LangSmith中的自我改进评估器等能力。更多版本更新信息详见[https://blog.langchain.dev/week-of-6-24-langchain-release-notes/](https://link.zhihu.com/?target=https%3A//blog.langchain.dev/week-of-6-24-langchain-release-notes/)。

## **LangGraph Cloud**

在LangGraph v0.1稳定版基础上，LangChain发布了LangGraph Cloud，目前处于封闭测试阶段。LangGraph Cloud为大规模部署Agent提供了专用基础设施。

LangGraph Cloud支持以可扩展、容错的方式大规模部署LangGraph Agent。只需一键部署，即可在LangSmith中获得集成的跟踪和监控体验。LangGraph Cloud提供了一个集成的开发者体验，一站式实现Agent原型设计、调试和监控的Agent工作流。此外，LangGraph Cloud还提供了LangGraph Studio，用于调试Agent故障模式和快速迭代。

在LangGraph基础上，LangGraph Cloud增加了如下能力：

  * 双重文本处理在当前运行的行线程上处理新的用户输入。它支持四种不同的策略来处理额外的上下文：拒绝、队列、中断和回滚。
  * 用于长期运行任务的异步后台作业。可以通过轮询或webhook 检查任务是否完成。
  * 按计划运行常见任务的[Cron作业](https://zhida.zhihu.com/search?content_id=245049981&content_type=Article&match_order=1&q=Cron%E4%BD%9C%E4%B8%9A&zhida_source=entity)。



## LangSmith中的自我改进LLM评估器

使用“[LLM-as-a-Judge](https://zhida.zhihu.com/search?content_id=245049981&content_type=Article&match_order=1&q=LLM-as-a-Judge&zhida_source=entity)”是对LLM应用程序的输出进行分级的常用方法。其包括将生成的输出传递给一个单独的LLM，并要求它对输出进行评估。但是，确保“LLM-as-a-Judge”性能良好需要另一轮提示工程。谁来评价评估者？

LangSmith通过允许用户对LLM评估者的反馈进行修正来解决这个问题，修正后的反馈将被存储为少样本示例，用于调整/改进“LLM-as-a-Judge”。无需手动调整提示，即可改进未来的评估，确保更准确的测试。

## LangChain

LangChain支持一行代码初始化任何模型。使用LangChain Python中的通用模型初始化器，可以使用任何常见的聊天模型，而无需记住不同的导入路径和类名。
