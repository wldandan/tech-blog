# 【AI工程】01-开篇词：AI工程（AI Engineering）初探

原文链接：https://zhuanlan.zhihu.com/p/464692752

---

> 2021年10月，在[Gartner](https://zhida.zhihu.com/search?content_id=191395858&content_type=Article&match_order=1&q=Gartner&zhida_source=entity)公司发布的《[2022年十二大重要战略技术趋势](https://link.zhihu.com/?target=https%3A//www.gartner.com/en/information-technology/insights/top-technology-trends)》中，将[AI工程](https://zhida.zhihu.com/search?content_id=191395858&content_type=Article&match_order=1&q=AI%E5%B7%A5%E7%A8%8B&zhida_source=entity)（AI Engineering）列为未来三到五年 **企业数字业务创新的加速器。**

一、最近在探索[MindSpore](https://zhida.zhihu.com/search?content_id=191395858&content_type=Article&match_order=1&q=MindSpore&zhida_source=entity)，也对AI工程相关技术做了些分析，关于“这个趋势”，个人认为有**两个驱动因素** ：  
  
**1) 传统的软件工程无法直接适用于AI软件的开发**  
由于AI软件（[软件2.0](https://link.zhihu.com/?target=https%3A//www.oreilly.com/radar/the-road-to-software-2-0/)）同 传统软件（软件1.0） 在实现上存在较大差异(实现方式、开发流程等)，因此传统的软件工程方法并不完全适用AI软件开发。

> **传统软件：** 也称软件1.0，在开发态，其**决策逻辑由程序员编写** 。在运行态，基于**决策逻辑 + 外部输入，获得结果(完全符合预期** //不考虑Bug的情况**)** 。   
> **AI软件：** 也称[软件2.0](https://link.zhihu.com/?target=https%3A//www.oreilly.com/radar/the-road-to-software-2-0/)，指基于神经网络构建的软件。其**决策逻辑由样本数据与预期结果训练而来** 。当软件运行时，**基于(训练得到的)决策逻辑+ 外部输入，获得结果(不能100%符合预期、存在漂移)。**

  
**2) AI工程化是AI大规模发展的必经之路**  
任何一个行业、企业，只要有场景，有积累的数据，有算力，都可以落地AI应用，**但落地效率、周期会远超预期。**

> Gartner的研究表明，只有**53%的项目能够从AI原型转化为生产** 。而AI 要成为企业的生产力，就必须以工程化的技术来解决模型开发、训练、预测等全链路生命周期的问题。

AI工程的出现，**正好能弥补这个短板** 。进一步，随着AI大规模的“平民化”，应用场景的丰富会不断催熟AI工程；而AI工程则会对场景落地提供关键支撑，相辅相成。  
  
因此，基于如上两点，就不难理解为什么Gartner会将 AI工程定义为未来战略技术趋势之一。

二、作为软件工程师，对于这个趋势，我准备做什么？

相比于软件工程、[DevOps](https://zhida.zhihu.com/search?content_id=191395858&content_type=Article&match_order=1&q=DevOps&zhida_source=entity)、持续交付等理念、实践及方法论的成熟，AI工程作为一个新生技术，有很多未知仍需要探索。因此，我计划通过一系列文章，探讨AI工程的概念、知识体系、相关方法，并基于MindSpore实战。初步构思包括如下三大部分（具体细节见文末）：  
  
**一、AI工程基础概念：** 梳理AI工程的知识体系、全景图及发展现状。  
**二、AI工程方法探索：** 基于AI软件开发的生命周期，探索相关的工程方法、技术和工具。  
**三、实战案例：** 通过端到端的案例，探讨如何将AI工程技术应用于实战。  
  
AI工程是一门新兴学科，不少方法、实践与工具还在不断探索的过程中。如果你有兴趣，欢迎与我们一起加入探索的行列，贡献案例或提出宝贵建议。

* * *

本文作为开篇，**将介绍什么是AI工程、以及 为什么它会被列为未来重要的技术趋势** 。

**1\. 什么是AI工程**

从Gartner发布的《[2022年十二大重要战略技术趋势](https://link.zhihu.com/?target=https%3A//www.gartner.com/en/information-technology/insights/top-technology-trends)》中看到，AI工程（AI Engineering）是使用数据处理、预训练模型、机器学习流水线([MLOps](https://zhida.zhihu.com/search?content_id=191395858&content_type=Article&match_order=1&q=MLOps&zhida_source=entity)) 等开发AI软件的技术统称，帮助企业更高效的利用AI创造价值。

> “AI engineering is the discipline of operationalizing updates to AI models, using integrated data and model and development pipelines to deliver consistent business value from AI. It combines automated update pipelines with strong AI governance. AI engineering automates updates to data, models and applications to streamline AI delivery. Combined with strong AI governance, AI engineering will operationalize the delivery of AI to ensure its ongoing business value.“

如果从软件工程演进的视角看如上的描述，就会发现其本质是在构建AI模型/软件的全生命周期中，系统化、规范化地使用方法、工具和实践，确保高质量、高效率地交付业务价值。

因此，通俗的讲，**AI工程是一系列方法、工具和实践的集合，确保AI模型/软件的高效交付，具备可信、健壮性及可解释性，并持续地为用户创造价值。**

### 2\. AI工程诞生的推动因素

**2\. 1 传统软件工程无法有效应对AI领域的挑战**

谈到AI工程，自然会联想到业界通常提到的软件工程，那为什么传统软件工程解决不了AI领域的问题？这得先从**传统软件与** AI软件**的差异说起:**

**传统软件：** 也称软件1.0，在开发态，其**决策逻辑由程序员编写** 。在运行态，基于**决策逻辑 + 外部输入，获得结果(完全达到预期)** 。 

**AI软件：** 也称[软件2.0](https://link.zhihu.com/?target=https%3A//www.oreilly.com/radar/the-road-to-software-2-0/)，指基于神经网络构建的软件。其**决策逻辑由样本数据与预期结果训练而来** 。当软件运行时，**基于(训练得到的)决策逻辑+ 外部输入，获得结果(无法完全达到预期)。**

**二者在驱动方式、实现方式、演进方式 以及 开发流程上存在显著差异：**

**1)驱动方式不同：**  
  
传统软件是**规则驱动** 的，即程序员将问题的解决方案转化为一条条明确的逻辑规则，再分解到各个模块中转化为代码实现，模块间具有清晰边界，而且对于同一个输入，每个模块具有确定性的输出，也就保证了系统整体具有确定性的输出。 

AI软件是**数据驱动** 的，是基于训练集数据通过机器的自我学习得到的模型，而不是根据人类知识编写出来的。训练得到的模型是对问题最优解决方案的一种逼近，至于逼近程度有多好，则取决于训练所使用的数据质量和算法有效性。从这个角度讲，对于机器学习系统，Datasets are the new code，可见数据的重要作用，不亚于传统软件中的代码。  
另一方面，模型输出的结果具有不确定性，只能在统计意义上告诉用户哪种输出结果的概率是最大的，这本质上也是由于其数据驱动的方式而决定的（训练集不能代表所有真实情况）。

《SE4AI：A System Engineeering Problem》by Foutse Khomh

**2) 实现方式不同**

传统软件的实现方式采取**“分而治之+组合”的策略。** 先根据需求与架构方法，设计成不同模块及代码实现，再遵循从代码行->函数->模块->子系统->系统逐步实现并自底向上不断组合的实现机制。

而AI软件的实现方式采取了**“逐步逼近+搜索”的策略。** 在设计算法和网络结构后，只是得到了一个模型的骨架，而其中各层神经网络的权重是不确定的，需要在整个可能的权重分布空间中进行搜索，才能得到最优权重，而搜索是通过对模型进行训练/调优，将损失函数最小化。

**3) 演进方式不同**

对于传统软件，**触发其变更的条件在于需求发生变化或者检测出系统bug** ，此时需要通过修复bug、添加新功能或重构等方式保持系统的生命力，从而不断演进。

而对于AI软件，触发其变更的条件在于数据发生了变化或者检测出模型对于数据的表现不及预期，此时需要通过模型重训的方式，增强模型的泛化能力、健壮性和稳定性。

**4) 开发流程不同**

传统软件开发的流程一般会经历 **需求分析、系统设计、代码实现、验证、发布以及运维的过程** ，如下：

传统软件开发流程

> **需求分析：** 对软件系统要满足的业务需求进行分析和适当的拆解，划分成一个个小的可实例化描述的子需求；  
> **系统设计：** 建立业务领域模型，并对系统进行功能模块划分、定义每个模块的接口和输入、输出和各种用例；  
> **代码编写：** 通过编码实现每个模块的功能，并通过单元测试和模块级测试；  
> **测试验证：** 按照模块->子系统->系统的顺序逐步集成，每一步都需要通过集成测试来保证正确性；  
> **发布部署：** 正式部署到生产环境，被用户所使用；  
> **系统监控：** 实时监控系统在生产环境的功能正确性、性能和可靠性等指标，当发现系统有bug或有新的需求产生时，进行新一轮的上述过程。

而AI软件开发过程一般会经历 **问题抽象、数据准备、算法设计、模型训练、模型评估与调优、部署的过程** ：

AI软件开发流程

> **问题抽象：** 将要解决的问题转化成为一个机器学习领域的特定问题，如回归、分类、聚类等；  
> **数据准备：** 包括数据获取（下载数据集或自行收集数据）、数据清洗、数据增强、数据分析，以及对数据进行特征提取等工作；  
> **算法设计：** 选择已有的的标准算法或者设计新的算法，编写代码实现算法；  
> **模型训练：** 训练基线模型，在验证集上初步验证模型效果，并通过调整超参数提高模型精度和性能；  
> **模型调优：** 在测试集上对模型效果进行评估，通过准确率、精确率、召回率等评估模型效果；  
> **部署应用：** 将模型部署到生产环境提供推理服务，并持续监控模型在线运行过程中的健壮性、稳定性和对生产环境数据的适应程度，当发现对于生产环境数据不能很好地适应的话，需要采集生产环境数据并进行新一轮的数据处理、算法调整、模型训练和评估过程。

正是由于如上几点的差异，如果将传统软件工程方法直接应用于AI软件上，会出现诸多问题，如：

  * 在需求维度看，问题无法转化为明确的需求规格进行描述，难以通过确定性的规则提出解决方案；
  * 在实现维度看，无法分拆成一个个小的单元各个击破，因为神经网络模型往往是作为一个整体提供服务；另外，AI系统中新增了数据和模型这两个重要的软件要素，而不仅仅是代码。
  * 从测试维度看，无法将传统的单元测试、集成测试等方法简单应用于AI系统，测试覆盖率等指标也不能简单地用于评估AI系统测试的完备度
  * 从维护维度看，数据分析、模型训练和调优、模型监控等方面会面临新的挑战。



因此，AI软件的出现意味着过去熟知的软件开发方式正在悄然发生根本性变化，时代需要应用新的工程方法来更好的开发与维护AI软件。

**2\. 2 AI大规模落地的背后需要强大的工程能力**

人工智能发展史上经历过三次兴衰，而最近的一次人工智能浪潮的标志性事件，无疑是2016年AlphaGo大败人类世界围棋冠军，它标志着人工智能在沉寂三十多年后再次爆发。

从这一事件后，随着AI技术的蓬勃发展，AI已不再是少数科学家实验室中的“神话”，它已逐步走进千行百业，成为智能时代的重要基础设施。

这一波浪潮汹涌的背后，当然与人工智能发展的三要素（数据、算法和算力）的逐步成熟密不可分，但是，这是否意味着只要具备了**足够的数据、精巧的算法、充沛的算力** ，就一定能够轻松训练出像AlphaGo一样厉害的AI？ 

任何一个行业，任何一个企业，只要有场景，有积累的数据，有算力，现在都可以落地AI应用。“但落地速度会远远超过预期”。在Gartner看来，目前只有53%的项目能够从AI原型转化为生产。而AI 要成为企业的生产力，就必须以工程化的技术来解决模型开发、训练、管理、预测等全链路生命周期的问题。

AI工程的出现，正好能弥补这个短板，确保AI模型/AI软件的高效交付，并具备可信、健壮性及可解释性。

进一步，随着AI大规模的“平民化”，场景将会不断催熟AI工程；而AI工程也会持续对场景落地提供重要支撑，相辅相成。

以2020年OpenAI发布的人工智能大模型GPT-3为例，它使得自然语言处理领域的许多问题得到了令人惊讶的进展，但其本身的复杂度非常高，光是模型参数就达到1750亿个，训练数据量达到45TB，训练出的模型大小有700GB。

如此复杂的工作，自然不是有了数据、算法及算力就能达成的。从GPT-3所发表的论文来看，有31位作者参与编写，而各位作者分成若干个小组，数据处理、模型设计、代码编写、调试参数等角色各司其职、相互配合，每位作者都是各自领域的专家，既具备在AI领域独到的洞见和创新能力，也具备将AI实现、落地的工程化能力。

**可见，AI技术的成功，既要具备数据、算法、算力三要素，又要依托于强大的AI工程能 力，才能让复杂的人工智能技术加速商业创新。**

* * *

### 3\. “AI工程与实践”的大纲

AI工程方法与技术的发展，离不开AI框架与工具的支持。

笔者在探索MindSpore的过程中，对国内外前沿的AI工程技术做了系统化梳理。因此，将会通过系列文章，把AI工程的知识与实际案例相结合，分享这些技术背后的原理、方法和实践，并基于MindSpore实战。

该系列的文章初步构思包括如下三部分：

**一、基础概念**

从宏观视角梳理AI工程的知识体系，展现AI工程方法与技术的全景图与发展现状。通过这一部分，建立AI工程的总体认识，并了解其中的基本概念。

**二、全生命周期软件工程方法**

循着AI软件开发全生命周期中的各个步骤，通过例子讲解每一步中可以使用哪些软件工程方法、技术和工具，能够解决哪些实际问题，通过这一部分，你不仅可以了解到这些技术的应用，还能了解这些技术背后的原理。

其中的技术主题包括但不限于：

  * [迭代0](https://zhuanlan.zhihu.com/p/500536548)：机器学习项目开始前要做哪些准备工作？
  * 数据分析与处理：怎样识别数据中的缺陷？怎样处理数据才高效？
  * 动静态图统一：动态图的易用性 与 静态图的高效 是否 可以兼得？
  * 函数式编程：在机器学习中如何使用？
  * [设计模式](https://zhuanlan.zhihu.com/p/502911554)：机器学习中有哪些设计模式？
  * [自动并行](https://zhuanlan.zhihu.com/p/528964900)：加速模型训练的利器
  * 可视化调试调优：让模型调试调优明明白白
  * 故障模式：机器学习中有哪些故障模式？
  * 模型评估：怎样评估模型的健壮性、可靠性、安全性等？
  * 模型的全场景部署：对于端、边、云，如何高效的部署？
  * MLOps和[[CD4ML](https://zhida.zhihu.com/search?content_id=191395858&content_type=Article&match_order=1&q=CD4ML&zhida_source=entity)](https://zhuanlan.zhihu.com/p/552173360)：机器学习中如何玩转DevOps和持续交付？
  * AI工程师的能力模型：怎样才能逐步成长为AI工程的高手？
  * ……



**三、实战案例**

结合当前人工智能技术最流行领域，如图像处理、自然语言处理、文本分类等，通过完整案例和基于MindSpore的源码实现，讲解如何将AI工程技术应用于实战。

最后，由于AI工程是一门新兴学科，不少方法、实践与工具还在不断探索的过程中。如果你有兴趣，欢迎与我们一起加入探索的行列，贡献案例或提出宝贵建议。

  * MindSpore官网：[https://www.mindspore.cn](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/)
  * MindSpore论坛：[https://bbs.huaweicloud.com/forum/forum-1076-1.html](https://link.zhihu.com/?target=https%3A//bbs.huaweicloud.com/forum/forum-1076-1.html)
  * 代码仓地址：
    * Gitee：[https://gitee.com/mindspore/mindspore](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/mindspore)
    * GitHub：[https://github.com/mindspore-ai/mindspore](https://link.zhihu.com/?target=https%3A//github.com/mindspore-ai/mindspore)



* * *

> 下一篇：[02：AI工程（AI Engineering）面面观](https://zhuanlan.zhihu.com/p/484046440)

**参考：**  
  
[AI工程化，让人工智能回归现实-技术圈](https://link.zhihu.com/?target=https%3A//jishuin.proginn.com/p/763bfbd606e1)  
[Gartner发布2021重要战略趋势|Gartner中国](https://link.zhihu.com/?target=https%3A//www.gartner.com/cn/newsroom/press-releases/2021-top-strategic-technologies-cn)  
[https://medium.com/machine-learning-in-practice/how-machine-learning-differs-from-traditional-software-80d0a235ff3b](https://link.zhihu.com/?target=https%3A//medium.com/machine-learning-in-practice/how-machine-learning-differs-from-traditional-software-80d0a235ff3b)  
[https://karpathy.medium.com/software-2-0-a64152b37c35](https://link.zhihu.com/?target=https%3A//karpathy.medium.com/software-2-0-a64152b37c35)  
[Gartner Predicts Organizations Will Adopt a Robust AI Engineering Strategy in 2021 | Stefanini](https://link.zhihu.com/?target=https%3A//stefanini.com/en/trends/news/gartner-predicts-more-organizations-adopts-ai-engineering-strate%23whats-AI-Engineering)

展开阅读全文​
