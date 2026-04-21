# 【RAG】01-浅谈RAG的十大挑战

原文链接：https://zhuanlan.zhihu.com/p/696904158

---

​

目录

## 背景介绍

[ChatGPT](https://zhida.zhihu.com/search?content_id=242997851&content_type=Article&match_order=1&q=ChatGPT&zhida_source=entity)的火爆问世，让人们对问答机器人重拾了信心，有了大模型的加持，ChatGPT还成功通过了谷歌的编程面试。普通大众可以借助ChatGPT完成日常问答、翻译、文本生成、文本分类等多种任务。但是如果直接拿它来回答专业领域的问题，可能就不那么尽如人意了。问题主要表现在：

  1. 依赖大模型的内在知识，已有的内容可能过时或者来自非权威的来源，还容易产生大量幻觉等。
  2. 如果尝试对大模型进行微调，其参数量大，微调成本非常高，还容易出现过拟合的现象。



而检索增强生成（Retrieval Augmented Generation，RAG）通过将检索技术和大模型的结合，能很好地解决上述问题，成为了领域知识问答最受欢迎的解决方案。

## RAG简介

RAG可以从权威的、预先确定的知识库中检索出相关的上下文片段，然后将问题和检索结果一起传给大模型，大模型依赖这些特定输入生成最终的答案。这样既保证了来源的可靠性，避免了大模型的过度发挥，也省去了大模型微调所需的成本。

图1：RAG工作流程，来源[1]

常见的RAG工作流程如上图所示，可以分成3个步骤，同时也分别与RAG名称中的3个单词对应：

  1. **检索（Retrieval）** ：根据用户问题，检索已知知识库的相关上下文信息。这里，会将用户问题通过[Embedding模型](https://zhida.zhihu.com/search?content_id=242997851&content_type=Article&match_order=1&q=Embedding%E6%A8%A1%E5%9E%8B&zhida_source=entity)转化为向量，与向量数据库的内容进行比对，通过相似性搜索，获得与向量数据库中最匹配的Top K个结果。
  2. **增强（Augmented）** ：将用户问题与检索结果一起放到Prompt模板中，为下一步大模型生成做好充分准备。
  3. **生成（Generation）** ：将检索增强后的Prompt传递给大模型，生成最终的回答。



图2：RAG架构，来源[2]

从工程角度来看，RAG又可以分为离线和在线两个阶段：

  1. 数据准备阶段（离线）：导入不同格式的领域知识文件，经过文本分割和向量化，存储到向量数据库。主要包括：数据提取、文本分割、向量化、数据入库等环节。
  2. 应用阶段（在线）：主要包括数据检索和大模型生成，也就是前面提到的检索、增强、生成的整个工作流程。



## 面临挑战

然而，RAG方案是一个领域知识问答的整体架构，真正应用时还是会面临非常多的挑战，因为领域知识问答对回答的准确性、完整性有着非常高的要求。

RAG回答效果不好时，大部分原因可能是没有在知识库中找到正确的上下文，这与数据准备和数据检索都有着紧密的联系。获得检索结果后，Prompt的质量和大模型的能力将直接影响最终的输出。下面我们来看看每个阶段的关键挑战有哪些。

### 数据准备

数据准备一般是离线的过程，它包括以下几部分：

  * 数据提取：解析不同格式的领域知识文件，如word、pdf、markdown、html等，将其处理为同一个范式，并进行必要的过滤、压缩、格式化等数据处理，甚至需要记录一些元数据信息，作为后续检索时的参考。
  * 文本分割：文本分割需要考虑Embedding模型的Tokens限制以及语义的完整性。常见的有按段落或句子切分、固定长度切分等。
  * 向量化：向量化是将切分好的文本转化为向量的过程，它需要使用到Embedding模型，可以直接选择开源的常用模型，如BGE、M3E等，也可以基于领域知识微调后使用。
  * 数据入库：数据向量化后建立索引，存入向量数据库。常见的向量数据库有Faiss、Chroma、ElasticSearch、LanceDB等。



图3：数据准备，来源[2]

这个阶段的主要挑战：

**挑战1. 文本分割不合理，导致召回的内容不准确或不完整。**

切分后的文档块将直接影响检索时与用户问题的匹配程度，块太小可能无法回答某些问题，而块太大则可能导致答案中包含不必要的信息。

目前的切分方式有很多，但是怎么样的才是适合本领域文档的，可能受文件格式、写作习惯、表达方式等因素的影响，需要结合不同的情况选择最佳的切分方案，甚至可能需要针对不同类型的内容做一些特殊处理。最重要的还需要根据其语义进行合理的切分。

### 数据检索

常见的数据检索方法包括向量检索、关键词检索等。

  * 向量检索：也称相似性检索，计算用户问题转化后的向量与向量数据库中存储向量之间的相似度。常见的相似度计算方法有余弦相似度、曼哈顿距离（L1）和欧式距离（L2）等。
  * 关键词检索：主要通过关键词构建倒排索引，在全文中找到关键词的所在位置、出现频次等。



由于检索效果不同，一般建议采用多种检索方法混合的方式，来提升召回率。多种检索方式检索出的相关知识片段可能不同，此时还需要通过[Rerank模型](https://zhida.zhihu.com/search?content_id=242997851&content_type=Article&match_order=1&q=Rerank%E6%A8%A1%E5%9E%8B&zhida_source=entity)，进行整体的排序，来帮助大模型得到更好的输出。

图4：数据检索，来源[2]

这个阶段的主要挑战：

**挑战2. 向量检索作为主流检索方式，主要基于相似度判断，但可能因为各种原因而出现差异。**

  * 语义歧义：向量表示可能无法捕捉概念之间的细微差别，导致术语混淆。
  * 大小与方向：余弦相似度是一种常见的度量，重点关注向量的方向而不是它们的大小。这可能会导致语义上遥远但方向上相似的匹配。
  * 粒度不匹配：用户问题的向量可能代表特定概念，但如果知识库仅包含更广泛的主题，则可能会检索到比预期更广泛的结果。
  * 全局相似性与局部相似性：大多数向量搜索机制都识别全局相似性，但可能未捕获的局部或上下文会更相似。
  * 稀疏检索挑战：检索机制可能很难在庞大的知识库中识别正确的段落，特别是当所需的信息稀疏地分布在多个文档中时。



**挑战3. 多个检索结果排名和优先级不合适，导致大模型的回答没有抓住重点。**

确定多个检索到的段落对于生成任务的重要性或相关性可能具有挑战性，增强过程必须适当权衡每个段落的价值。这就要求在排序阶段，可能也要具备一些领域的认知能力。

### 大模型生成

RAG中，Prompt需要包括用户问题、检索结果和任务描述，一般描述为：“你是什么领域的专家，你需要基于这些检索结果，回答这个用户问题”。

在大模型的选择上，需要综合考虑领域知识的语言、大模型对领域知识的理解等，确保可以获得较好的结果输出。

图5：大模型生成，来源[2]

这个阶段的主要挑战：

**挑战4. 如何优化Prompt，让大模型基于现有知识和提示信息，生成更优的回答。**

Prompt不仅仅是把问题和检索结果做简单的封装，针对不同的生成模型，需要有不同的表达方式和补充限定。为了避免大模型的过度发挥，可能需要给大模型提供一个固定“人设”，做好各种可能情况的限定，比如“你是这个领域的专家”、“你只能基于提供的问题和相关结果做概括和总结”等等。

**挑战5. 大模型如何保证回答的连贯性和一致性。**

由于提供的知识片段可能来自不同的文档，也可能是通过用户问题中不同的关键词找到的内容，大模型需要集成这些信息，确保生成的输出在逻辑上连贯并保持一致。

**挑战6. 大模型如何更好地理解领域知识片段。**

不同的大模型对该领域知识以及术语的理解程度不同，导致在对检索结果的理解和输出上也会存在差异。可以通过适当的微调来提升该领域的语言理解和生成能力。

### 语料准备

RAG各个环节的调优以及RAG的评估，都离不开领域知识的语料——QA对。主要包括以下几方面：

  1. 用于Embedding模型微调的QA对：包含领域知识问题、正向回答和负向回答，目的是提升用户问题与向量数据库的相似度匹配能力。
  2. 用于Rerank模型微调的QA对：包含领域知识问题和回答，目的是提升多个检索结果的排序能力。
  3. 用于大模型微调的QA对：包含领域知识问题和回答，目的是提升对Prompt内容的理解和总结能力。
  4. 用于评估RAG质量的QA对：包含领域知识问题和回答，目的是为RAG质量评估提供依据。



这个阶段的主要挑战：

**挑战7. 如何提供高质量的QA对，辅助RAG的调优。**

一般的调优都会要求尽可能多地提供QA对，但如果人为去创造，需要耗费大量的工作量，让机器去生成又觉得不一定靠谱。因此，如何提供高质量的QA对，是影响RAG最终效果的重要因素。

### 意图识别

大模型通常会编造一些内容，而且仅靠Prompt提示大模型，也不一定有效，这时候就需要在进入RAG流程前就做好意图识别，先基于场景分流，避免大模型的过度发挥。

比如判定为闲聊场景时，就不走知识召回流程，使用闲聊Prompt传递给大模型回答；判定为该领域的内容时再进入到领域知识问答流程；判定为特定问题时直接给出模板回答等。

这个阶段的主要挑战：

**挑战8. 什么情况下拒答的合适的。**

当用户提出一个无法仅凭现有文档回答的问题时，需要及时拒绝，避免给出不合适的答案而误导用户。一般情况下会设置一个相似度阈值，或者依据已有的场景语料，但也无法保证完全准确。太过拒答显得“高冷”，太过“热情”又显得不可信。

### 多模态RAG

知识库中除了基本的文字，表格、图片、样例代码、样式、参考文档链接等多模态信息也是必不可少的，因为这些被认为是对用户理解更为友好的方式，然而对机器理解却不太友好。因此，从数据准备到数据检索到大模型生成，每个步骤都需要对多模态的内容做好处理，才不会导致这部分信息的丢失，更有助于回答结果的呈现。

这个阶段的主要挑战：

**挑战9. 多模态的支持是必然趋势。**

在多模态RAG的研究中，针对不同的模态，包括图像、代码、结构化知识、音频和视频，有不同的检索和合成程序、目标任务和挑战。比如通过图像检索扩展文本生成的上下文，利用样例代码和相关文档增强代码生成等等。

### RAG评价

RAG生成的结果需要有一套科学的评价体系，才能有效说明它的好坏。

提到评价一般都会使用各种指标，例如RAGAs（Retrieval-Augmented Generation Assessment）框架，提供了一系列评估指标，包括上下文精度（Context Precision）、上下文召回（Context Recall）、忠诚度（Faithfulness）和答案相关性（Answer Relevancy）等。

这些指标共同构成了RAGAs评分，用于全面评估RAG的能力。它通过计算RAG中的question（用户问题）、contexts（RAG检索的文档块）、answer（大模型的回答）、ground_truth（问题的标准答案）的一致性程度来判断当前RAG的领域知识能力，其中question和ground_truth需要人工给出，contexts和answer来自于RAG。

这个阶段的主要挑战：

**挑战10. 如何评价RAG在该领域已达到可用的程度。**

类似RAGAs的评分方式，question和ground_truth需要单独提供，因此question的设计和ground_truth的质量就直接影响了评估结果的可靠性。

同时，由于评价过程还需要基于大模型和Embedding模型，对这里用到的模型的可靠性，以及传给模型的Prompt的准确性有了要求，否则评价出的指标也是不可信的。

## 总结和期望

RAG作为目前比较热门的方向，还在不断地优化和更新。它给领域知识问答带来了福音，但想要真正满足领域用户的问答需求，还需要经过大量实践。期待着哪一天，RAG能像各领域专家一样，不仅完全懂得领域内所有的知识库内容，还能不断学习新知识，提升“自己”，成为一个可信赖的专业领域机器人，让用户再也没有难找的知识。

## 相关链接

  1. [Retrieval-Augmented Generation (RAG): From Theory to LangChain Implementation]([https://towardsdatascience.com/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2](https://link.zhihu.com/?target=https%3A//towardsdatascience.com/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2))
  2. [一文搞懂大模型RAG应用（附实践案例）]([https://mp.weixin.qq.com/s?__biz=MzkxNjYxMjUwMA==&mid=2247484519&idx=1&sn=55a9fa5107d3b69a019db36703b86538&chksm=c14c709cf63bf98a615255cb55041338eaaa024066f721c9c81456c99eaf8f423ae27a56fa8a&token=1160261652&lang=zh_CN#rd](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzkxNjYxMjUwMA%3D%3D%26mid%3D2247484519%26idx%3D1%26sn%3D55a9fa5107d3b69a019db36703b86538%26chksm%3Dc14c709cf63bf98a615255cb55041338eaaa024066f721c9c81456c99eaf8f423ae27a56fa8a%26token%3D1160261652%26lang%3Dzh_CN%23rd))
  3. [Langchain-Chatchat]([https://github.com/chatchat-space/Langchain-Chatchat](https://link.zhihu.com/?target=https%3A//github.com/chatchat-space/Langchain-Chatchat))
  4. [Advanced RAG Techniques: an Illustrated Overview]([https://pub.towardsai.net/advanced-rag-techniques-an-illustrated-overview-04d193d8fec6](https://link.zhihu.com/?target=https%3A//pub.towardsai.net/advanced-rag-techniques-an-illustrated-overview-04d193d8fec6))
  5. [Why do RAG pipelines fail? Advanced RAG Patterns — Part1]([https://cloudatlas.me/why-do-rag-pipelines-fail-advanced-rag-patterns-part1-841faad8b3c2](https://link.zhihu.com/?target=https%3A//cloudatlas.me/why-do-rag-pipelines-fail-advanced-rag-patterns-part1-841faad8b3c2))
  6. [How to improve RAG peformance — Advanced RAG Patterns — Part2]([https://cloudatlas.me/how-to-improve-rag-peformance-advanced-rag-patterns-part2-0c84e2df66e6](https://link.zhihu.com/?target=https%3A//cloudatlas.me/how-to-improve-rag-peformance-advanced-rag-patterns-part2-0c84e2df66e6))
  7. [Seven Failure Points When Engineering a Retrieval Augmented Generation System]([https://arxiv.org/abs/2401.05856v1](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2401.05856v1))
  8. [Retrieving Multimodal Information for Augmented Generation: A Survey]([https://arxiv.org/abs/2303.10868](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2303.10868))
  9. [Ragas tutorials]([https://docs.ragas.io/en/stable/](https://link.zhihu.com/?target=https%3A//docs.ragas.io/en/stable/))



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【RAG技术洞察】02-RAP：针对多模态LLM Agent的上下文记忆检索增强规划](https://zhuanlan.zhihu.com/p/700207538)
