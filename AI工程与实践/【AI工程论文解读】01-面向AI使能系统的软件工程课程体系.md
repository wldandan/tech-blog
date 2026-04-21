# 【AI工程论文解读】01-面向AI使能系统的软件工程课程体系

原文链接：https://zhuanlan.zhihu.com/p/520689011

---

​

目录

> 人工智能自1956年诞生以来，相关理论和技术持续演进。直到近十年，得益于深度学习等[算法](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E7%AE%97%E6%B3%95&zhida_source=entity)的突破、算力的不断提升以及海量数据的持续积累，AI才得以真正大范围地从[实验室研究](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%AE%9E%E9%AA%8C%E5%AE%A4%E7%A0%94%E7%A9%B6&zhida_source=entity)走向产业实践。未来AI除了重视技术创新以外，还更加关注工程实践和安全可信，这也构成了新的"[三维](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E4%B8%89%E7%BB%B4&zhida_source=entity)"发展坐标。  
>  \--[中国信通院](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E4%B8%AD%E5%9B%BD%E4%BF%A1%E9%80%9A%E9%99%A2&zhida_source=entity)《人工智能白皮书（2022年）》

## 概述

人工智能（AI），包括ML和数据分析的子领域，是许多开发者和师生们感兴趣的热门话题。然而，尽管人工智能课程比比皆是，如高校课程以及许多在线MOOC教程，但我们会发现，在构建涉及AI的完整系统时，AI教育通常侧重于算法和技术，通常局限于优化模型精度。很少有课程关注AI与[软件工程](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%B7%A5%E7%A8%8B&zhida_source=entity)方面的结合。因此，逐渐兴起一门新兴学科-[AI工程](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=AI%E5%B7%A5%E7%A8%8B&zhida_source=entity)，通俗的讲，AI工程是一系列方法、工具和实践的集合，确保AI模型/软件的高效交付，具备可信、健壮性及可解释性，并持续地为用户创造价值。

最近阅读了《Teaching Software Engineering for AI-Enabled Systems》这篇论文，面对AI课程中很少有解决工程问题的课程现状，[卡内基梅隆大学](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%8D%A1%E5%86%85%E5%9F%BA%E6%A2%85%E9%9A%86%E5%A4%A7%E5%AD%A6&zhida_source=entity)的Christian Kästner和Eunsuk Kang设计了一门新课程，向有ML背景的学生教授软件工程技术，在AI与软件工程结合的方向进行了探索，并分享了教授该课程的经验和相关教材，教材请参见[https://github.com/ckaestne/seai/](https://link.zhihu.com/?target=https%3A//github.com/ckaestne/seai/)。

本次主要从论文中的[课程设计](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E8%AF%BE%E7%A8%8B%E8%AE%BE%E8%AE%A1&zhida_source=entity)、经验总结几方面介绍下本次的教学实践。

## 课程设计

在设计课程时，考虑到[数据科学家](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%A7%91%E5%AD%A6%E5%AE%B6&zhida_source=entity)主要关注如何使用尖端技术构建模型和算法，软件工程师专注于如何通过业务代码构建AI系统。因此，课程重点围绕**如何使用软件工程技术来构建更好的系统** 。从软件工程视角来看如何构建支持人工智能的系统，软件工程师通过哪些工作可以将机器学习的想法转化为可扩展和可靠的产品。**AI工程重点关注设计、实施、运营和质量保证问题。**

虽然AI组件（特别是[ML模型](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=ML%E6%A8%A1%E5%9E%8B&zhida_source=entity)）有明显的特征，但它们也与软件工程中的核心主题紧密相关，例如：

  * 在实践中，软件工程师已经例行公事地处理未明确和不可靠的组件。**把不可靠或不受信任的组件构建为可靠和安全的[系统技术](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E6%8A%80%E6%9C%AF&zhida_source=entity)**，这些在构建支持AI使能系统时变得至关重要。
  * 在建立AI使能系统时，**环境** 也发挥着至关重要的作用。软件工程师区分机器和世界，识别环境假设，并在环境背景下评估质量，这些概念对于构建支持AI使能系统也很重要。
  * 在软件工程中，模块化和[结构化](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E7%BB%93%E6%9E%84%E5%8C%96&zhida_source=entity)的失败是众所周知的，例如，在功能交互方面，需要仔细的设计和[系统级](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E7%BA%A7&zhida_source=entity)测试。同样，**强大的架构和质量保证制度** （超出模型精度）在支持AI使能系统中发挥着重要作用。
  * 软件工程师开发了许多技术来**监控系统、评估生产中的系统和自动化决策，包括A/B测试和持续部署** ，这些技术在构建支持AI使能系统时再次变为必不可少的部分。
  * 许多模型会变得庞大，学习、使用和版本[管理成本](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E7%AE%A1%E7%90%86%E6%88%90%E6%9C%AC&zhida_source=entity)高昂。但是，我们收集了大量分布式、[可扩展系统](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%8F%AF%E6%89%A9%E5%B1%95%E7%B3%BB%E7%BB%9F&zhida_source=entity)的构建和运营，跟踪修订等相关**知识** ，管理大规模配置。这些都有助于设计人员学习AI使能系统。



### 课程范围和知识点

设计课程时，确定了如下假设和范围：

  1. 结合软件工程的术语、技术和体系（例如测试覆盖率、架构视图、[故障树](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%95%85%E9%9A%9C%E6%A0%91&zhida_source=entity)），将软件工程的概念转换到AI使能系统。
  2. 关注AI工程实现，不关注具体的AI技术。AI组件很大程度上被认为是一个黑盒，因此较少讨论其内部机制（例如，决策树、神经网络、[符号人工智能](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E7%AC%A6%E5%8F%B7%E4%BA%BA%E5%B7%A5%E6%99%BA%E8%83%BD&zhida_source=entity)）；除了一些必备的基础知识之外，较少关注[数据分析](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=2&q=%E6%95%B0%E6%8D%AE%E5%88%86%E6%9E%90&zhida_source=entity)过程中各个步骤，因此，课程与其他AI技术课程不冲突。
  3. 围绕具体场景决策和分析课程的实践设计，及其实施管道（例如，监控、[自动部署](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2&zhida_source=entity)、容器）的实战经验。



  


课程中确定了软件工程生命周期各阶段的主题，从广义上讲，课程涵盖：

  * **需求** ：了解系统目标，缺乏的AI组件规范；识别和界定质量关键点（超出模型精度），设定Safety、Security、Fairness的期望；[威胁分析](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%A8%81%E8%83%81%E5%88%86%E6%9E%90&zhida_source=entity)和故障处理；规划如何处理错误。
  * **架构** ：考虑质量属性之间的平衡（例如，学习时间、推理延迟、模型大小、可更新性、可解释性），规划AI组件的部署位置和方式，数据来源，模型编排、[面向服务的体系结构](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E9%9D%A2%E5%90%91%E6%9C%8D%E5%8A%A1%E7%9A%84%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84&zhida_source=entity)。
  * **实施和维护** ：设计可扩展的分布式数据和计算系统，用于实验、A/B测试、“Canary”版本发布和持续交付的基础实施，出处和[配置管理](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86&zhida_source=entity)，系统监控。
  * **质量保证** ：离线和生产中测量模型质量，保证数据质量，测试整个ML管道，Safety、Security、Fairness Analysis。
  * **流程** ：迭代和规划，跨学科团队合作，[技术负债](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%8A%80%E6%9C%AF%E8%B4%9F%E5%80%BA&zhida_source=entity)，制订道德决策。



### 课程任务

课程设计目标是培养面向实战的AI工程经验。 课程基于电影[流媒体](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%B5%81%E5%AA%92%E4%BD%93&zhida_source=entity)think Netflix构建场景，通过模拟用户观看，并对数据进行分析，得出用户喜欢的电影类型。共创建了五个小组作业：

  1. **建模基础和离线评估** ：从系统（Kafka流和API）收集数据，构建和评估模型（通常是[协同过滤](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%8D%8F%E5%90%8C%E8%BF%87%E6%BB%A4&zhida_source=entity)），以熟悉基础架构，练习基本ML技能，并加入所有团队成员。
  2. **[权衡分析](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%9D%83%E8%A1%A1%E5%88%86%E6%9E%90&zhida_source=entity)** ：通过尝试和比较不同的[建模技术](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%BB%BA%E6%A8%A1%E6%8A%80%E6%9C%AF&zhida_source=entity)，并总结经验进行权衡，专注于测量各种质量（超出模型精度）。
  3. **基础架构部署和测试** ：将解决方案从Jupyter Notebook迁移到强大且可扩展的学习和推理基础架构，构建[数据质量](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=2&q=%E6%95%B0%E6%8D%AE%E8%B4%A8%E9%87%8F&zhida_source=entity)检查并测试整个基础架构，评估生产中的模型质量，并使用[Docker容器](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=Docker%E5%AE%B9%E5%99%A8&zhida_source=entity)将模型部署为REST API。
  4. **模型更新** ：完全自动化模型更新（持续部署，包括自动的Canary发布），在不停机的情况下部署更新，并使用自研的基础架构在生产中进行A/B测试实验。
  5. **[反馈回路](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%8F%8D%E9%A6%88%E5%9B%9E%E8%B7%AF&zhida_source=entity)** ：分析系统中潜在的反馈回路和攻击场景，设计预防措施，并持续监控系统性能。



## 经验

**重点和[先决条件](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%85%88%E5%86%B3%E6%9D%A1%E4%BB%B6&zhida_source=entity)：**更好的解决方案是为来自任何背景的学生教授单独的部分——对于ML学生，我们将从零开始引入软件工程概念（例如，[持续集成](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity)、版本控制、软件架构），而不是对先前的经验做出假设；对于软件工程学生，我们的课程可以从实用的ML管道和模型[质量度量](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E8%B4%A8%E9%87%8F%E5%BA%A6%E9%87%8F&zhida_source=entity)开始。对于软件工程硕士课程，理想的情况是，内容分成更小的部分，将其作为模块集成到我们现有的软件工程课程中，涉及需求、软件架构和质量保证。

**[仿真工程](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E4%BB%BF%E7%9C%9F%E5%B7%A5%E7%A8%8B&zhida_source=entity) ：**构建和运行[仿真器](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E4%BB%BF%E7%9C%9F%E5%99%A8&zhida_source=entity)需要大量的工程工作，由于[时间限制](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%97%B6%E9%97%B4%E9%99%90%E5%88%B6&zhida_source=entity)，我们无法在第一个产品中实现许多功能。例如，我们的[模拟器](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E6%A8%A1%E6%8B%9F%E5%99%A8&zhida_source=entity)中的观看行为并不是特别真实（例如，我们没有喜欢多部电影的高级用户，我们没有停止、重新开始或倒带电影的模型，我们没有明确地对[人口统计数据](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E4%BA%BA%E5%8F%A3%E7%BB%9F%E8%AE%A1%E6%95%B0%E6%8D%AE&zhida_source=entity)或位置进行建模用户）。我们还发现，16万用户平均每秒产生 70个事件和 1个推荐请求的使用规模太低，无法让学生认真考虑运营成本和性能，这有时会导致相当肤浅的[工程权衡](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%B7%A5%E7%A8%8B%E6%9D%83%E8%A1%A1&zhida_source=entity)和明显的答案。推荐电影的 ML 任务在计算上也没有足够的挑战性，无法在不同的学习技术之间提供有趣的权衡讨论。在未来的产品中，可能值得将模拟器扩展到更多用户，并探索涉及图片、音频或视频的其它学习任务。

**实际基础** ：除了家庭作业中使用的电影推荐场景外，我们几乎在每一堂课中都使用不同的场景来讨论不同问题的广度以及做出系统特定设计和权衡决策的重要性。同时，可能值得更深入地探索一些场景，甚至具体的实现。

**工具** ：由于人工智能是一个活跃的、快速发展的领域，我们努力为公平性和可解释性等新兴主题寻找标准技术或成熟的工具。虽然有一些工具（例如，谷歌的What-If工具），但大多数工具仍然是实验性的，难以用于教学。通过软件工程，可以形成一套成熟的工具和标准来满足AI系统开发人员和教育工作者不断增长的需求。

## 总结

具有AI组件的系统在构建和维护方面具有挑战性。面向对人工智能感兴趣的学生教授软件工程，以培养更广泛的思维，而不是狭隘地关注静态数据集和模型质量，这是一次很有意义的课程探索。

同时，Gartner的研究表明，只有**53%的项目能够从AI原型转化为生产** 。而AI 要成为企业的生产力，就必须以[工程化](https://zhida.zhihu.com/search?content_id=203840796&content_type=Article&match_order=1&q=%E5%B7%A5%E7%A8%8B%E5%8C%96&zhida_source=entity)的技术来解决模型开发、训练、预测等全链路生命周期的问题。AI工程的出现，**正好能弥补这个短板** 。进一步，随着AI大规模的“平民化”，相信在不远的将来，应用场景的丰富会不断催熟AI工程，而AI工程则会对场景落地提供关键支撑，相辅相成。

  


## 参考文献

[1]Christian Kästner, Eunsuk Kang, 2020. Teaching Software Engineering for AI-Enabled Systems. ICSE-SEET 2020.

[2]中国信通院《人工智能白皮书（2022年）》

  


> 下一篇：[【AI工程论文解读】02-聊聊机器学习系统中的技术债](https://zhuanlan.zhihu.com/p/532922813)
