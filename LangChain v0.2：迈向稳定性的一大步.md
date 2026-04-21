# LangChain v0.2：迈向稳定性的一大步

原文链接：https://zhuanlan.zhihu.com/p/698934783

---

​

目录

[LangChain](https://zhida.zhihu.com/search?content_id=243449034&content_type=Article&match_order=1&q=LangChain&zhida_source=entity)社区于5月20日发布了LangChain v0.2版本。在该版本中，主要提升了LangChain的稳定性和安全性。v0.2版本的更新主要体现在如下几个方面：

  * LangChain和LangChain-Community完全分离
  * 新的（带版本号的）文档
  * 更加成熟和可控的智能体框架
  * 改进了LLM接口标准化，尤其是[工具调用](https://zhida.zhihu.com/search?content_id=243449034&content_type=Article&match_order=1&q=%E5%B7%A5%E5%85%B7%E8%B0%83%E7%94%A8&zhida_source=entity)
  * 更好的流式支持
  * 30+个新的合作伙伴包



## **追求稳定性：LangChain架构的演进**

LangChain v0.2中最显著的变化之一是将**langchain软件包与[langchain-community](https://zhida.zhihu.com/search?content_id=243449034&content_type=Article&match_order=1&q=langchain-community&zhida_source=entity)解耦**。因此，langchain-community现在依赖于langchain-core和langchain。这是在langchain v0.1.0中开始的工作的延续，旨在创建一个更稳定和更独立的软件包。  
由于langchain-community包含大量第三方集成，因此其有大量（可选的）依赖项，大量文件，并且由于集成的性质，该软件包偶尔会受到CVE的影响。因此，将langchain对langchain-community的依赖性移除，使langchain更轻量级、更专注和更安全。

## **提升易用性：文档版本化**

根据社区的反馈，对LangChain文档进行了改进。  
首先，文档会进行版本化。保留现有文档作为v0.1版本，并开始构建一个独立的v0.2版本。目前，文档默认为v0.1版本，在v0.2版本正式发布后，将开始默认使用v0.2版本文档。版本化的管理可以更好地反映软件包的状态。  
其次，文档架构将会更加扁平和简单。主要分为四个部分：**教程、操作指南、概念指南和API参考** 。方便用户更容易查找文档获得信息。  
同时，将会增加“LangChain的演进”文档介绍，更好地突出LangChain的变化。

## **功能增强：[LangGraph](https://zhida.zhihu.com/search?content_id=243449034&content_type=Article&match_order=1&q=LangGraph&zhida_source=entity)智能体**

自LangChain推出之初，难以定制预构建的链和智能体的内部结构成为大家反馈最多的问题。之前通过引入了[LCEL](https://zhida.zhihu.com/search?content_id=243449034&content_type=Article&match_order=1&q=LCEL&zhida_source=entity)解决了链的问题。本次将会解决智能体的问题。  
在此之前，LangChain中的智能体一直以AgentExecutor为基础，它是一个具有硬编码逻辑的单一类，用于说明如何运行智能体。虽然已经为这个类添加了很多的参数，以支持越来越复杂的智能体，但它仍然不是真正易用的。  
因此，引入了LangGraph，它是LangChain的扩展，专门用于创建智能体。可以将其视为“智能体的LCEL”。它在LCEL的基础上添加了两个重要组件：轻松定义循环的能力（对于智能体很重要，但对于链不需要）和内置记忆的功能。在langchain v0.2中，仍将保留旧的AgentExecutor，但LangGraph会成为构建智能体的推荐方式。如何从AgentExecutor迁移到LangGraph，请参见[迁移文档](https://link.zhihu.com/?target=https%3A//python.langchain.com/v0.2/docs/how_to/migrate_agent/%3Fref%3Dblog.langchain.dev)。

## **改进了对流处理的支持和标准化了工具调用等**

相较于langchain v0.1.0，当前在以下几个方面做了大量改进：

  * **标准化的Chat模型接口** ：添加了标准化的工具调用支持，并增加了一个用于结构化输出的标准化接口，以实现简化不同LLMs之间的无缝切换。
  * **异步支持** ：改进许多核心抽象的异步支持。
  * **[流处理支持](https://zhida.zhihu.com/search?content_id=243449034&content_type=Article&match_order=1&q=%E6%B5%81%E5%A4%84%E7%90%86%E6%94%AF%E6%8C%81&zhida_source=entity)** ：流处理对于 LLM 应用程序至关重，因此添加了事件流式处理API来改进了流处理的支持。
  * **合作伙伴包** ：在Python中，为包括MongoDB、Mistral和Together AI 等20多个供应商添加了专用软件包；在JavaScript中，为Google VertexAI、Weaviate和Cloudflare等17个供应商添加了专用软件包。



## 如何升级

考虑向后兼容性，为方便从v0.1到v0.2的升级，增加了迁移CLI，并会提供版本间的变更文档说明版本之间的变化点。
