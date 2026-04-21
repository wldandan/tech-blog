# 【GraphRAG】02-微软&蚂蚁GraphRAG方案解析

原文链接：https://zhuanlan.zhihu.com/p/1854007770

---

​

目录

## 背景

微软GraphRAG和蚂蚁GraphRAG产生背景类似，都是为了**克服传统RAG的不足** ，比如由于缺乏特定领域知识、实时更新信息，导致模型存在一定局限性。这些不足容易引发**幻觉现象** ，即模型生成不准确甚至是虚构的信息。以及在实际场景中面临的一些问题，比如忽视上下文关系、存在冗余信息、缺乏全局信息等。

如上图所示，与传统的 RAG 不同，GraphRAG 从预先构建的**[图数据库](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=%E5%9B%BE%E6%95%B0%E6%8D%AE%E5%BA%93&zhida_source=entity) 中检索**与给定查询相关且包含关系知识的图元素，如上图所示。这些元素可能包括节点、三元组、路径或子图。

GraphRAG 考虑了文本之间的联系，能够更准确、更全面地检索关系信息。

此外，图数据，如知识图，对文本数据进行了抽象和总结，大幅缩短了输入文本的长度，减少了冗长的问题。通过检索子图或图社区，能够获取全面的信息，借助捕获图结构中更广泛的上下文和相互联系，有效解决 QFS 挑战。

如上图所示为GraphRAG整体架构，GraphRAG 是借助外部结构化[知识图谱](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=%E7%9F%A5%E8%AF%86%E5%9B%BE%E8%B0%B1&zhida_source=entity)来增进语言模型的上下文理解，并生成更具洞见响应的框架。GraphRAG 的目标在于从数据库中检索出最为相关的知识，进而提升下游任务的答案质量。

蚂蚁GraphRAG更关注实际业务场景下的**图算法优化** ，旨在解决图数据在实际应用中的一系列痛点，微软涵盖了更广泛的研究领域和理论基础。

除了微软和蚂蚁，其他公司的关于GraphRAG项目也逐渐问世。比如**NebulaGraph** 为GraphRAG项提供了**NebulaGraph数据库** ，是一款开源的、分布式的、易扩展的原生图数据库。

NebulaGraph将大型语言模型融入NebulaGraph数据库，比如集成了[LlamaIndex](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=LlamaIndex&zhida_source=entity)和LangChain等大语言模型框架，提供更智能、更精准的搜索结果。此外NebulaGraph 还提供了一系列的生态工具，如 NebulaGraph Studio，这是一款可以通过 Web 访问的开源图数据库可视化工具，提供 schema 模式管理、数据导入、编写 nGQL 查询等一站式服务。

再如**Neo4j** 为GraphRAG项目提供了基于LLM提取**知识图谱的生成器：llm-graph-builder** ，如下图所示。

再如**开源项目Camel** 宣布在其框架中引入Graph RAG Agent。目前并开源了**知识图谱抽取的 Prompt** ，组合使用UnstructuredIO和neo4j实现节点和关系抽取。

## 模型架构

无论是微软、蚂蚁的GraphRAG，总体思路相同，整个Pipeline也可划分为索引(Indexing)与查询(Query)两个阶段；进一步细分，可分为基于图的索引（G-Indexing）、图引导检索（G-Retrieval）和图增强生成（G-Generation）；其次两者都用到了大语言模型LLM，[图神经网络](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=%E5%9B%BE%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C&zhida_source=entity)GNN。  
不同点在于技术细节，包括所用的大模型，图数据库等等。

### 微软的Graph RAG

**微软的Graph RAG核心链路** ：利用大型语言模型 (LLMs) 从您的来源中提取知识图谱；将此图谱聚类成不同粒度级别的相关实体社区；遍历所有社区以创建“社区答案”，并进行缩减以创建最终答案。如下图所示。  


  
微软有特有的社区算法如**[Leiden算法](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=Leiden%E7%AE%97%E6%B3%95&zhida_source=entity)** ，此算法用来生成图谱中实体的社区层次结构。能够递归地对图谱进行社区聚类，直到达到某个社区规模的阈值。通过这种方式，GraphRAG能够帮助用户在不同的粒度级别上导航和总结图谱。微软还有特有的图谱嵌入技术，将实体和关系映射到低维向量空间中。这种做法有助于保留图谱中的结构信息和语义信息，便于后续进行相似性度量、聚类等分析任务。此外，微软提供了**[Azure平台](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=Azure%E5%B9%B3%E5%8F%B0&zhida_source=entity)** ，里面提供了很多的大模型比如LangChain、LlamaIndex、RAGFlow、Ollama等等。微软的GraphRAG与OpenAI耦合度较高。  
微软整体架构如下所示：  


### 蚂蚁的Graph RAG

**蚂蚁的Graph RAG核心链路** ：索引（三元组抽取）：通过LLM服务实现文档的三元组提取，写入图数据库；检索（子图召回）：通过LLM服务实现查询的关键词提取和泛化（大小写、别称、同义词等），并基于关键词实现子图遍历（DFS/BFS），搜索N跳以内的局部子图；生成（子图上下文）：将局部子图数据格式化为文本，作为上下文和问题一起提交给大模型处理。  
蚂蚁的整体架构设计如下：  


  
值得一提的是，蚂蚁开发了首个对外开源的Graph RAG框架，蚂蚁全自主的开源产品：**DB-GPT[50] +[OpenSPG](https://zhida.zhihu.com/search?content_id=249393086&content_type=Article&match_order=1&q=OpenSPG&zhida_source=entity)[42] + TuGraph[46]**，如下图所示。  


  
DB-GPT是一个开源的AI原生数据应用开发框架，目的是构建大模型领域的基础设施，通过开发多模型管理(SMMF)、Text2SQL效果优化、RAG框架以及优化、Multi-Agents框架协作、AWEL(智能体工作流编排)等多种技术能力，让围绕数据库构建大模型应用更简单，更方便。  


OpenSPG是蚂蚁集团结合多年金融领域多元场景知识图谱构建与应用业务经验的总结，并与OpenKG联合推出的基于SPG(Semantic-enhanced Programmable Graph)框架研发的知识图谱引擎。  


  
TuGraph是蚂蚁集团与清华大学联合研发的大规模图处理系统，构建了包含图数据库、图计算引擎、图机器学习、图研发平台的完善图技术体系。支持海量多源的关联数据的实时处理，显著提升数据分析效率，支撑了蚂蚁支付、安全、社交、公益、数据治理等300多个场景应用，多次打破图数据库性能基准测试LDBC-SNB世界纪录，并跻身IDC中国图数据库市场领导者象限。  


## 重点应用

  * 蚂蚁的GraphRAG广泛应用于金融、医疗、法律等领域。
  * 微软的GraphRAG可以用于构建搜索引擎、聊天机器人、内容生成应用等。它通过构建基于图谱的文本索引，将整个文本集合分解成更小、更易于管理的社区模块，从而扩展了模型的理解和生成能力。



## 对比总结

基于上述三张，我汇总了一份表格，方便查看不同点，这里主要突出特有特点，他们在AI框架、知识图谱、图存储系统中存在交集，部分框架或者系统可以通用。

## 相关链接

  1. [https://www.cnblogs.com/fanzhidongyzby/p/18252630/graphrag](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/fanzhidongyzby/p/18252630/graphrag)
  2. [https://qianfanmarket.baidu.com/article/detail/1117485](https://link.zhihu.com/?target=https%3A//qianfanmarket.baidu.com/article/detail/1117485)
  3. [https://blog.csdn.net/m0_59163425/article/details/140388353](https://link.zhihu.com/?target=https%3A//blog.csdn.net/m0_59163425/article/details/140388353)
  4. [http://docs.dbgpt.cn/docs/cookbook/rag/graph_rag_app_develop/](https://link.zhihu.com/?target=http%3A//docs.dbgpt.cn/docs/cookbook/rag/graph_rag_app_develop/)
  5. [https://qianfanmarket.baidu.com/article/detail/1117026](https://link.zhihu.com/?target=https%3A//qianfanmarket.baidu.com/article/detail/1117026)
  6. [https://zhuanlan.zhihu.com/p/709528088](https://zhuanlan.zhihu.com/p/709528088)
  7. [https://unite.ai/zh-CN/Graph-rag-%E7%9A%84%E5%8A%9B%E9%87%8F-%E6%99%BA%E8%83%BD%E6%90%9C%E7%B4%A2%E7%9A%84%E6%9C%AA%E6%9D%A5/](https://link.zhihu.com/?target=https%3A//unite.ai/zh-CN/Graph-rag-%25E7%259A%2584%25E5%258A%259B%25E9%2587%258F-%25E6%2599%25BA%25E8%2583%25BD%25E6%2590%259C%25E7%25B4%25A2%25E7%259A%2584%25E6%259C%25AA%25E6%259D%25A5/)
  8. [https://www.nebula-graph.com.cn/posts/your-GraphRAG](https://link.zhihu.com/?target=https%3A//www.nebula-graph.com.cn/posts/your-GraphRAG)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【GraphRAG技术洞察】01-GraphRAG：从局部到全局，一种面向查询摘要的GraphRAG方法](https://zhuanlan.zhihu.com/p/1431617467)  
> 下一篇：[【GraphRAG技术洞察】03-GraphRAG Indexing介绍及源码解读](https://zhuanlan.zhihu.com/p/3115108247)
