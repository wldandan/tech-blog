# 【单Agent框架】07-具有超长语境要点记忆功能的人类启发式阅读智能体

原文链接：https://zhuanlan.zhihu.com/p/699566756

---

​

目录

本篇论文介绍了一种称为 **ReadAgent** 的新方法，这是一种受人类启发的大语言模型 （[LLM](https://zhida.zhihu.com/search?content_id=243589687&content_type=Article&match_order=1&q=LLM&zhida_source=entity)） 代理系统，旨在通过将有效上下文长度增加到 20 倍来**处理非常长的上下文** 。受人类交互式阅读长文档的启发，ReadAgent **使用提示系统** 来决定在情节记忆中存储哪些内容，将这些情节压缩成简短情节记忆，称为“要点记忆”，并在需要时查找原始文本中的段落。主要研究结果表明，ReadAgent在QuALITY、[NarrativeQA](https://zhida.zhihu.com/search?content_id=243589687&content_type=Article&match_order=1&q=NarrativeQA&zhida_source=entity)、QMSum三个长文档阅读理解任务上的表现优于基线，同时将有效上下文窗口扩展了3至20倍。

## 背景介绍

当前LLM具有较好的语言理解能力，但在阅读的上下文长度上存在局限性，而人类可以阅读、理解和推理非常长的上下文，这主要是由于阅读方法上的差异。LLM倾向于以相对被动的方式逐字逐句地消费确切的内容，而人类则专注于更模糊的要点信息，并参与交互式阅读过程，在需要时在原文中查找相关细节。

## 技术方案

本篇论文提出了一种LLM代理系统ReadAgent，可以像人类的方法一样处理长上下文。

  1. **情节分页** ：提示LLM**在连续文本中暂停阅读的位置** ，也就是如何分成多页。
  2. **要点记忆** ：提示LLM**将每个页面压缩成较短的要点** ，并将要点与相应的上下文关联。
  3. **交互式查找** ：LLM根据给定的任务和要点列表，**找到相关的页面，将要点与这些页面相关联** ，并解决给定任务。



### 情节分页

每一步向LLM提供一些信息，包括从上一个暂停点开始，以及在达到最大字数限制时结束。提示LLM选择段落之间哪个点为自然暂停点，然后将上一个暂停点和当前暂停点之间的内容视为一个片段，成为页面。

### 要点记忆

为每个页面提供一个Prompt，让LLM把内容压缩成要点或摘要。Prompt中使用了单词“shorten”来保留叙事的流程，使其串联起来更自然，因为实验发现“summarize”可能会生成一个重组的总结。

### 交互式查找

给定一个关于长文档的任务，ReadAgent除了使用要点记忆之外，还能查找到原文中的相关细节。由于要点记忆与页码相关，只需提示LLM想找哪一页，并在给定任务的情况下再次读取。论文中提出了**两种查找策略** ：**一次并行查找所有页（ReadAgent-P）** 和**每次顺序查找一页（ReadAgent-S）** 。

ReadAgent-P中，提供了一个最大页数，同时也提示LLM使用尽可能少的页面，避免不必要的计算开销。选定的原始页面替换了要点在记忆中的位置，保留了整个叙事流程。

ReadAgent-S中，模型会先查看之前扩展的页面，然后再决定扩展哪个页面，会让模型获得比并行查找更多的信息，也会带来更多的计算开销。

## 实验结果

论文使用**[PaLM 2-L](https://zhida.zhihu.com/search?content_id=243589687&content_type=Article&match_order=1&q=PaLM+2-L&zhida_source=entity) 指令调优**来进行实验和评价，PaLM 2-L的上下文长度为8K tokens。同时，**自定义了两种评分方法** ，统称为**[LLM Raters](https://zhida.zhihu.com/search?content_id=243589687&content_type=Article&match_order=1&q=LLM+Raters&zhida_source=entity)** 。其中，**LR-1** 是一个严格的评估分数，计算所有示例中完全匹配的百分比，**LR-2** 是一个宽松的评估分数，计算完全匹配和部分匹配的百分比。论文将常规的在长文本中查找相关页面作为基线，再与ReadAgent进行比较。

首先来看QuALITY中的表现，图2显示了原文和要点的字数统计，压缩率为84.24%。表1中ReadAgent查找1-5页时，以66.26%的压缩率给出了最好结果，因此上下文中可以容纳3倍的tokens。

其次是NarrativeQA，Gutenberg Validation的压缩率为96.8%，Gutenberg Test的压缩率为91.98%，在F-measure和LLM Raters上的表现也优于原始RAG。最后是QMSum，测试性能随着压缩率的降低和提高，因此查找更多页面比查找较少页面的表现更好；ReadAgent-S的性能大大优于ReadAgent-P，但代价是检索阶段请求数增加了6倍。

## 总结

论文提出了一个简单的交互式提示系统**ReadAgent** ，用于**减少LLM的上下文长度和上下文使用限制** 。在**精度、ROUGE分数** 等性能指标上，**ReadAgent的表现优于原始RAG** 。结果表明，LLM可以**生成长上下文的压缩文本表示** ，并使用这些文本进行**交互式推理** ，**查找出相关内容** 来解决已知任务。但是，当前方案不可能提供无限的上下文长度，也不能在要点记忆本身非常长的情况下保证好的性能。

## 参考资料

  1. Lee K H, Chen X, Furuta H, et al. A Human-Inspired Reading Agent with Gist Memory of Very Long Contexts[J]. arXiv preprint arXiv:2402.09727, 2024.



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【单Agent洞察】06-KG-Agent：一种基于知识图谱实现复杂推理的高效自治智能体框架](https://zhuanlan.zhihu.com/p/692978901)
