# 【GraphRAG】04-GraphRAG Query介绍及源码解读

原文链接：https://zhuanlan.zhihu.com/p/3990498959

---

​

目录

**GraphRAG Query介绍及源码解读**

## 介绍

在上一篇文章《GraphRAG Indexing介绍及源码解读》中已经介绍了[知识图谱](https://zhida.zhihu.com/search?content_id=249820611&content_type=Article&match_order=1&q=%E7%9F%A5%E8%AF%86%E5%9B%BE%E8%B0%B1&zhida_source=entity)的构建过程，当构建完知识图谱后，就可以基于知识图谱来提问了。GraphRAG提供了两种查询模式：

  * 本地搜索（[LocalSearch](https://zhida.zhihu.com/search?content_id=249820611&content_type=Article&match_order=1&q=LocalSearch&zhida_source=entity)）  
通过结合 AI 提取的知识图谱中的相关数据和原始文档的文本块来生成答案。这种方法适用于需要理解文档中提到的特定实体的问题（例如，洋甘菊的治愈属性是什么？）。
  * 全局搜索（GlobalSearch）



通过以 [map-reduce](https://zhida.zhihu.com/search?content_id=249820611&content_type=Article&match_order=1&q=map-reduce&zhida_source=entity) 方式搜索所有 AI 生成的社区报告来生成答案。这是一种资源密集型方法，但通常对于需要整体理解数据集的问题能给出很好的响应（例如，这个笔记本中提到的草药最重要的价值是什么？）。

## 工作流

### 本地搜索

在Local模式查询中，主要结合知识图谱中的结构化信息和原始文档中的非结构化数据，以此来构建增强生成的上下文。然后，借助于语言模型（[LLM](https://zhida.zhihu.com/search?content_id=249820611&content_type=Article&match_order=1&q=LLM&zhida_source=entity)）来获取响应。大致流程如下：

  1. 根据query和对话历史，借助实体的description_embedding向量进行相似度检索，从知识图谱中识别出最相关的实体。
  2. 找到这些实体更多的信息，包含：


  * 关联的原始文本块。提取其文本内容。
  * 关联的社区。提取其社区报告。
  * 关联的实体。提取其实体描述信息。
  * 关联的关系。提取其关系描述信息。
  * 关联的协变量。默认不生成。


  1. 对这些提取的信息进行排序与筛选，最终形成参考的上下文。
  2. 借助LLM和prompt，生成最终响应。



### 全局搜索  


  
Global Search 使用特定层次的 [Community Report](https://zhida.zhihu.com/search?content_id=249820611&content_type=Article&match_order=1&q=Community+Report&zhida_source=entity) 的集合。由于单个上下文可能无法容纳这些 Community Reports，需要进行 MapReduce 操作：

  1. 把 Community Reports 切分为多个部分，然后每个部分基于用户 query 使用 LLM 并发进行推理，每个部分总结几个主要的总结点。在推理总结时，会让 LLM 生成权重，方便最后进行 reduce 操作。
  2. 把每个部分推理的结果进行合并，将所有总结点进行 reduce 操作——再次使用 LLM 进行摘要总结。



## Query源码解读

执行query的语句如下：

# Global searchpython -m graphrag.query \\--root ./ragtest \\--method global \"What are the top themes in this story?"# Local searchpython -m graphrag.query \\--root ./ragtest \\--method local \"Who is Scrooge，and what are his main relationships?"  
---  
  
函数调用关系如下图所示：

graphrag/query/__main__.py中的主函数会依据参数不同，分别路由至cli::run_local_search()

以及cli::run_global_search()。

### 本地搜索

cli::run_local_search()函数主要调用了factories.py::get_local_search_engine()，返回一个LocalSearch类。

LocalSearch类的关键参数：

  * llm: 用于生成响应的 OpenAI 模型对象
  * context_builder: 用于从知识模型对象集合准备上下文数据的 上下文构建器对象
  * system_prompt: 用于生成搜索响应的提示模板。默认模板可在 system_prompt中找到
  * response_type: 描述所需响应类型和格式的自由文本(例如, 多个段落, 多页报告)
  * llm_params: 要传递给 LLM 调用的其他参数(如温度、最大标记数)的字典
  * context_builder_params: 在构建搜索提示的上下文时传递给 context_builder对象的其他参数字典
  * callbacks: 可选的回调函数, 可用于为 LLM 的完成流事件提供自定义事件处理程序



LocalSearch类中的asearch()相对比较简单，会直接根据上下文给出回复。

构建适合单个上下文窗口的本地搜索上下文，并为用户查询生成答案。

System Prompt：[https://github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/local_search/system_prompt.py](https://link.zhihu.com/?target=https%3A//github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/local_search/system_prompt.py)

\---角色---您是一个很有帮助的助手，回答有关提供的表中数据的问题。---目标---生成响应用户问题的目标长度和格式的响应，总结适合响应长度和格式的输入数据表中的所有信息，并结合任何相关的常识。如果你不知道答案，就直说吧。不要胡编乱造。数据支持的点应列出其数据参考，如下所示："这是多个数据引用支持的示例语句[数据：<数据集名称>（记录id）;<数据集名称>（记录id）]。"不要在单个引用中列出超过5个记录ID。相反，列出最相关的前5个记录id，并添加“+more”以表示还有更多。  
---  
  
这种模式更接近于常规的RAG语义检索策略，所消耗的Token也比较少。

为本地搜索Prompt构建上下文数据的算法：

[https://github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/local_search/mixed_context.py](https://link.zhihu.com/?target=https%3A//github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/local_search/mixed_context.py)

看代码基本都是先识别实体，实体作为知识图中的接入点，支持提取更多相关细节，如连接的实体、关系、实体协变量和社区报告，此外，它还从与识别的实体相关联的原始输入文档中提取相关文本块。然后，对这些候选数据源进行优先级排序和筛选，以适合预定义大小的单个上下文窗口，该上下文窗口用于生成对用户查询的响应。

### 全局搜索

cli::run_global_search()函数主要调用了factories.py::get_global_search_engine()，返回一个GlobalSearch类。

GlobalSearch类的关键参数：

  * llm：用于响应生成的 OpenAI 模型对象
  * context_builder：用于从社区报告中准备上下文数据的 上下文构建器对象
  * map_system_prompt：map 阶段使用的提示模板。默认模板可以在 map_system_prompt中找到
  * reduce_system_prompt：reduce 阶段使用的提示模板。默认模板可以在 reduce_system_prompt中找到
  * response_type：描述所需响应类型和格式的自由文本（例如，多段落，多页报告）
  * allow_general_knowledge：将其设置为 True 将在 reduce_system_prompt 中添加额外的指令，提示 LLM 在数据集以外的相关真实世界知识。请注意，这可能会增加凭空虚构的可能性，但对于某些场景可能很有用。默认值为 False
  * general_knowledge_inclusion_prompt：如果启用了 allow_general_knowledge，则添加到 reduce_system_prompt 中的指令。默认指令可以在 general_knowledge_instruction中找到
  * max_data_tokens：上下文数据的令牌预算
  * map_llm_params：一个字典，包含将在 map 阶段传递给 LLM 调用的其他参数（例如，temperature，max_tokens）
  * reduce_llm_params：一个字典，包含将在 reduce 阶段传递给 LLM 调用的其他参数（例如，temperature，max_tokens）
  * context_builder_params：一个字典，用于在 map 阶段构建上下文窗口时传递给 context_builder对象的其他参数。
  * concurrent_coroutines：控制 map 阶段的并行程度。
  * callbacks：可选的回调函数，可用于为 LLM 的完成流事件提供定制的事件处理程序。



GlobalSearch类的核心方法是structured_search/global_search/search.py::GlobalSearch.asearch()：

具体使用了map-reduce方法，首先使用大模型并行地为每个社区的summary生成答案，然后再汇总所有答案生成最终结果。

**map** **prompt：**[ https://github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/global_search/map_system_prompt.py](https://link.zhihu.com/?target=https%3A//github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/global_search/map_system_prompt.py)

\---角色---在回答有关提供的表中数据的问题时，您是一个非常有帮助的助手。---目标---您应该使用以下数据表中提供的数据作为生成响应的主要上下文。如果您不知道答案，或者输入数据表没有包含足够的信息来提供答案，请直接说出来。不要胡编乱造。答复中的每个关键点应包含以下要素：-描述：对关键点的全面描述。-重要性得分：介于0-100之间的整数得分，表示该点在回答用户的问题时有多重要。“我不知道”类型的回答的得分为0分。响应应为JSON格式，如下所示：{{"积分":[{{"description": "第1点的描述[数据：报告（报告ID）]","得分"：得分_值}}，{{"description": "第2点的说明[数据：报告（报告ID）]","得分"：得分_值}}]}}回答应保留“将”、“可能”或“将”等情态动词的原义和用法。有数据支持的要点应列出相关报告作为参考，如下所示："这是数据引用支持的示例语句【数据：报告（报告ids）】"**不要在单个引用中列出超过5个记录ID**。相反，列出最相关的前5个记录id，并添加“+more”以表示还有更多。  
---  
  
**reduce prompt：**[ https://github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/global_search/reduce_system_prompt.py](https://link.zhihu.com/?target=https%3A//github.com/microsoft/graphrag/blob/main/graphrag/query/structured_search/global_search/reduce_system_prompt.py)

\---角色---通过综合来自多个分析师的观点，您是回答有关数据集的问题的有用助手。---目标---生成目标长度和格式的响应，响应用户的问题，汇总来自多个分析人员的所有报告，这些分析人员专注于数据集的不同部分。请注意，下面提供的分析员报告按重要性降序排列。如果您不知道答案，或者提供的报告没有包含足够的信息来提供答案，请直说。不要胡编乱造。最终答复应删除分析师报告中的所有不相关信息，并将清理后的信息合并为一个全面的答复，该答复应提供对所有关键点和含义的解释，以适合答复的长度和格式。根据回复的长度和格式，添加相应的章节和注释。在markdown中设置响应样式。回答应保留“将”、“可能”或“将”等情态动词的原义和用法。回复还应保留分析员报告中以前包含的所有数据参考，但不提及多位分析员在分析过程中的作用。**不要在单个引用中列出超过5个记录ID**。相反，列出最相关的前5个记录id，并添加“+more”以表示还有更多。  
---  
  
也正是因为这种map-reduce的机制，导致global search对token的消耗量极大。

## 相关链接

  1. [https://microsoft.github.io/graphrag/posts/query/overview/](https://link.zhihu.com/?target=https%3A//microsoft.github.io/graphrag/posts/query/overview/)
  2. [https://www.cnblogs.com/fanzhidongyzby/p/18294348/ms-graphrag](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/fanzhidongyzby/p/18294348/ms-graphrag)
  3. [https://mp.weixin.qq.com/s/8YEeVsARGol3NJC6X-1Bsw](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/8YEeVsARGol3NJC6X-1Bsw)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【GraphRAG技术洞察】03-GraphRAG Indexing介绍及源码解读](https://zhuanlan.zhihu.com/p/3115108247)  
> 下一篇：[【GraphRAG技术洞察】05-小白上手GraphRAG](https://zhuanlan.zhihu.com/p/5115690518)
