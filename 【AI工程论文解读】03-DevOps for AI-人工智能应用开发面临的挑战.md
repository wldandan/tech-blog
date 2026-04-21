# 【AI工程论文解读】03-DevOps for AI-人工智能应用开发面临的挑战

原文链接：https://zhuanlan.zhihu.com/p/565078936

---

​

目录

>  _DevOps（研发运营一体化）：是 Development 和 Operations 的组合词，它是一组过程、方法与系统的统称，用于促进开发（应用程序/[软件工程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%B7%A5%E7%A8%8B&zhida_source=entity)）、技术运营和质量保障（QA）部门之间的沟通、协作与整合。研发运营一体化是将应用的需求、开发、测试、部署和运营统一起来，基于整个组织的协作和应用架构的优化，实现[敏捷开发](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%8F%E6%8D%B7%E5%BC%80%E5%8F%91&zhida_source=entity)、持续交付和应用运营的[无缝集成](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%97%A0%E7%BC%9D%E9%9B%86%E6%88%90&zhida_source=entity)，在保证稳定的同时，快速交付高质量的软件及服务，灵活应对快速变化的业务需求和市场环境。_

在上一篇《**[【AI工程】06-CD4ML-机器学习的持续交付（上）](https://zhuanlan.zhihu.com/p/552173360)** 》文中，我们介绍了如何实现机器学习的持续交付。最近阅读了《DevOps for AI – Challenges in Development of AI-enabled Applications》，本篇文章主要介绍了将DevOps和ML工作流结合的解决方案和实践。

近年来，随着AI相关技术快速演进，AI应用也广泛的出现在人们的日常生活中。然而，由于机器学习系统本身的复杂性—ML核心是通过大量训练迭代来找到最佳的预测模型，导致开发包含机器学习组件的软件系统时，开发过程会变得异常复杂。现代[软件开发过程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E8%BF%87%E7%A8%8B&zhida_source=entity)中，如DevOps，已经被广泛采用，用于应对频繁的开发迭代和软件变更的持续交付。尽管最新的[软件开发技术](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E6%8A%80%E6%9C%AF&zhida_source=entity)可以解决构建基于ML的软件系统所面临的一些问题，但目前还没有一个关于如何将它们与ML[工作流](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=2&q=%E5%B7%A5%E4%BD%9C%E6%B5%81&zhida_source=entity)结合起来的既定流程。在《DevOps for AI – Challenges in Development of AI-enabled Applications》文章中，作者结合工业案例指出了包括ML组件在内的复杂软件系统开发中会面临的挑战，然后讨论了由DevOps和ML工作流流程结合驱动解决挑战的可能解决方案。

## 概述

ML是近十年来发展迅速、最有前途的技术，几乎应用于所有商业领域和研究领域。在许多领域（如[医疗诊断](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%8C%BB%E7%96%97%E8%AF%8A%E6%96%AD&zhida_source=entity)），与传统应用程序相比，ML在提供结果方面表现出优势，并在某些活动中表现出优于人类智慧。这些趋势引发了对数据、AI科学家以及软件开发人员的巨大需求。

早期研究表明，构建算法(即开发ML模型)只是成功开发并运行基于AI的软件系统所需全部工作的一小部分。ML软件实现与传统软件的区别在于，它们的逻辑不是[显式编程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%98%BE%E5%BC%8F%E7%BC%96%E7%A8%8B&zhida_source=entity)的，而是通过从数据中学习自动创建的。因此，基于ML的软件系统的开发过程涉及到不同的活动，包括[数据收集](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86&zhida_source=entity)、数据准备、定义ML模型(如[深度网络模型](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%B7%B1%E5%BA%A6%E7%BD%91%E7%BB%9C%E6%A8%A1%E5%9E%8B&zhida_source=entity))、进行训练的过程以[量化模型](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E9%87%8F%E5%8C%96%E6%A8%A1%E5%9E%8B&zhida_source=entity)参数并获得预期的结果。这个过程被称为ML工作流。ML工作流需要一套复杂的工具和活动的支持，而获得一个高效的流程本身就是一个挑战。

然而，ML工作流并不涵盖整个软件开发过程。它并没有解决如何高效地进行软件开发的问题，这就引申出一个问题，即ML工作流和软件开发过程应该如何关联起来，或者作为一个更普遍的问题：包含[AI组件](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=AI%E7%BB%84%E4%BB%B6&zhida_source=entity)的软件系统的开发过程是什么？

在本文中，作者探讨了包含ML组件在内的软件开发过程的新要求。总结了ML组件需求带来的一些新的挑战，并提出了可用于开发、运行和演进基于ML的[软件系统](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=7&q=%E8%BD%AF%E4%BB%B6%E7%B3%BB%E7%BB%9F&zhida_source=entity)的软件开发模型。

现代软件开发过程应用了[敏捷原则](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%8F%E6%8D%B7%E5%8E%9F%E5%88%99&zhida_source=entity)，最常用的敏捷过程之一是DevOps，它将开发和运营集成到一个通用过程中，使开发人员能够快速反馈应用程序性能及其使用情况。具体来说，本文讨论了ML工作流与DevOps的集成。

## ML工作流

机器学习工作流描述了开发基于ML的软件系统时通常执行的[开发阶段](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E9%98%B6%E6%AE%B5&zhida_source=entity)和活动。如图1所示，ML工作流包含的阶段：模型需求、数据收集、数据清洗、数据标签、[特征工程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E5%B7%A5%E7%A8%8B&zhida_source=entity)、模型训练、模型评估、模型部署和模型监测。

图1：ML开发工作流阶段

对上述阶段进一步分组：（1）**[数据管理](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86&zhida_source=entity)** 、（2）**ML建模和** （3）**模型运营** 。数据管理过程包括数据的收集及数据准备。数据管理过程可以与ML建模分开执行—数据通常存储在[企业数据仓库](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E4%BC%81%E4%B8%9A%E6%95%B0%E6%8D%AE%E4%BB%93%E5%BA%93&zhida_source=entity)或开放式存储，提供给不同的ML应用程序使用。

ML建模过程相当于一个简单组件的开发，即创建模型、调整数据、训练模型，最后[评估模型](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%AF%84%E4%BC%B0%E6%A8%A1%E5%9E%8B&zhida_source=entity)。此过程是高度迭代的，并由结果驱动，如模型精度、准确率或ML模型训练期间使用的其它质量属性。模型的部署和监测属于模型运营过程—模型在软件系统中的集成及其性能。监测ML组件被视为ML工作流的一个组成部分，其有助于向ML建模提供反馈，可能会导致使用新数据集重新训练模型，或者使用新的特征和新的模型架构重新设计模型。

在实际环境中，由于各种原因，使得开发ML系统具有挑战性。例如用于构建模型的数据存在大量的数据质量问题，并且在训练模型之前需要花费大量的精力来准备数据，因为通常这些数据不能轻易地作为模型的输入。此外，在ML建模阶段，手动和临时程序的使用也会使模型试验结果难以重现，降低了试验的效率。

尽管模型的开发存在挑战，但其仍然是构成整个ML系统的一小部分。一旦数据科学家选择了最终的模型，就需要将其部署并集成到最终用户应用程序中。对于复杂的系统，模型的集成和部署需要考虑与软件系统相关的大量需求。基于ML的应用程序还需要持续监测，以便检测预测结果和数据中的错误（例如，训练服务偏差），并重新训练模型。

## [DevOps流程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=DevOps%E6%B5%81%E7%A8%8B&zhida_source=entity)

DevOps是一种[软件开发方法](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E6%96%B9%E6%B3%95&zhida_source=entity)，强调软件开发和运营之间的协作，以便运营软件系统并加快软件变更的交付。在[持续部署](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%83%A8%E7%BD%B2&zhida_source=entity)（CD）的背景下，其有助于创建一个可重复、可靠的流程，以便在生产环境中频繁发布变更软件版本。基于[敏捷软件开发](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%8F%E6%8D%B7%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91&zhida_source=entity)方法，DevOps的一个重要原则是构建、测试、部署和运营流程的自动化（如图2所示），同时确保所有的软件artifacts都有[版本控制](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6&zhida_source=entity)。因此，DevOps在部署管道中的实践涉及[软件部署](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E9%83%A8%E7%BD%B2&zhida_source=entity)过程的自动化，包括环境的自动配置，旨在最大限度地减少从软件开发团队到运营团队的移交债务。在软件开发实践中，部署管道是整个软件过程的技术表现，包括从版本控制到最终用户看到软件变更的所有阶段。部署管道中的自动化通常由基础架构团队完成，其将更具优势。

图2. DevOps工作流程

虽然DevOps方法也有一些非技术的实践，用于软件开发人员之间的有效协作，但大部分的重点都集中在技术实践上。在基于在线的应用程序中，部署机制被纳入到[持续集成](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity)（CI）系统中，并与定义的一些触发器相结合，用于促使[验收测试](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E9%AA%8C%E6%94%B6%E6%B5%8B%E8%AF%95&zhida_source=entity)或生产环境变更时的自动部署。作为部署过程的一部分，[配置管理](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86&zhida_source=entity)工具用于有新的变更时，根据选定的部署策略（如蓝绿部署或者滚动升级），自动调配、配置和升级现有环境。在其他领域，如信息[物理系统](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E7%89%A9%E7%90%86%E7%B3%BB%E7%BB%9F&zhida_source=entity)，或嵌入式系统，由于运营环境不同于[开发环境](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)，DevOps过程会包括额外的活动。可能为了提高开发周期的效率，会建立很多的[模拟实验](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%A8%A1%E6%8B%9F%E5%AE%9E%E9%AA%8C&zhida_source=entity)。

## 集成ML工作流和软件系统生命周期的挑战

ML工作流（如图1）定义了构建[ML模型](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=4&q=ML%E6%A8%A1%E5%9E%8B&zhida_source=entity)的端到端过程，但它忽略了软件开发过程。ML模型始终是应用程序或软件系统的一部分。软件系统的开发有另一种逻辑和过程（如图2左侧所示），运营和开发过程是相互反馈循环紧密相连（如图2右侧所示），但并没有很好的定义如何与ML工作流连接。这就导致执行ML工作流和DevOps时会面临很多挑战。本节介绍了推动ML工作流和DevOps集成的许多挑战。

  1. **多环境** ：假设ML训练有一个用于构建ML模型的[数据存储](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8&zhida_source=entity)。在ML工作流中，大多数与[数据处理](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)相关的活动都可以和其他活动解耦。ML模型训练对算力要求很高，大型模型通常需要[分布式](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F&zhida_source=entity)方式增加高性能的算力资源来训练，以加快训练速度。因此，开发环境需要交互式[软件开发工具](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7&zhida_source=entity)来调度本地或者分布计算单元的训练，通常是CPU和GPU的组合。另一方面，由于有限的计算和存储资源可能会上云，因此运行环境和传统的软件可能类似(例如基于web的应用程序)，也可能完全不同(例如[自动驾驶系统](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E9%A9%BE%E9%A9%B6%E7%B3%BB%E7%BB%9F&zhida_source=entity))。  
与传统软件开发相比，ML软件开发的主要挑战是如何[处理数据](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE&zhida_source=entity)和ML模型，而不仅仅是代码。用于跟踪和管理代码的工具，如Git，不足以确保有效跟踪和管理ML组件（代码、数据、模型）的版本依赖关系。类似地，持续集成（CI）工具不仅是测试代码，还用于测试和验证用于训练模型的数据。虽然一般使用CI工具来协调ML模型开发，但在[分布式训练](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F%E8%AE%AD%E7%BB%83&zhida_source=entity)的背景下，他们不足以确保在专用硬件（如GPU）上有效训练模型。除了不同的工具，ML软件开发过程还需要拥有不同的专业知识的开发者——[数据科学家](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=2&q=%E6%95%B0%E6%8D%AE%E7%A7%91%E5%AD%A6%E5%AE%B6&zhida_source=entity)、AI专家和软件工程师。以上这些在环境、工具、流程和专业知识方面差异，对ML软件开发提出了新的挑战：（1）如何确保成功管理多个开发环境？（2）如何确保组件之间无缝的相互通信和交互？（3）如何确保多学科专业知识，并实现领域专家之间的高效协作？
  2. **互补需求** ：现代软件开发流程将传统的[需求管理](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E9%9C%80%E6%B1%82%E7%AE%A1%E7%90%86&zhida_source=entity)与利益相关者的结果/数据反馈结合起来，最近还使用人工智能来增强系统运营的反馈。与用户验收相关的软件系统特性通常通过A/B测试进行评估。在许多领域，与质量属性相关的非功能性需求是开发过程的主要关注点。例如，与可靠性（safety、security、可靠性、健壮性等）、资源（实时性、性能、能耗、计算和存储约束）、用户界面（可用性）相关的属性。与ML工作流的要求不同：[数据质量](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=2&q=%E6%95%B0%E6%8D%AE%E8%B4%A8%E9%87%8F&zhida_source=entity)、数据管理的有效性（数据准备、标签）、训练过程的有效性（CPU训练时间）以及训练过程中所需的资源。关于ML模型的质量，有一组指标可以显示预测的质量：准确性、精确度和召回等。在特征工程中，[特征选择](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E9%80%89%E6%8B%A9&zhida_source=entity)及其处理(归一化、转换等)的质量、有效性与预测质量有关。  
在集成过程中，主要的挑战在于理解软件系统需求和ML需求之间的关系。例如，系统特征与[数据集](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=2&q=%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)特征之间的关系。显然，这些并不相同，但它们可能是相关的——当将特性指定为系统需求时，哪些特性应该包括在ML模型中？另一个例子：为了实现[系统的可靠性](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%8F%AF%E9%9D%A0%E6%80%A7&zhida_source=entity)，我们是否需要确保ML模型的精度达到一定的水平？性能要求也存在类似的问题：ML[模型性能](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E6%80%A7%E8%83%BD&zhida_source=entity)对系统性能的影响是什么——例如实时性要求？上述例子可以总结为一个共同的挑战：（1）如何将ML需求与软件系统需求联系起来？（2）如何从系统需求中得出ML需求？
  3. **系统验证** VS. **ML评估** ：ML评估的目标是提高预测质量——首先找到指定代价函数的最小值，然后以准确性等ML指标评估结果。这些指标的度量在开始时是未知的，需要通过多次迭代、调整ML模型和输入数据集来寻求[最优值](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%9C%80%E4%BC%98%E5%80%BC&zhida_source=entity)。该过程没有预定的目标，也没有过程的预判（比如时间和资源）。软件系统有规范去度量其正确性。正确性可以是[二进制](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E4%BA%8C%E8%BF%9B%E5%88%B6&zhida_source=entity)类型(正确/失败)，也可以是和非功能属性(如可靠性)相关的一种方式。在验证过程中，由于系统不应该被更改/调整，因此验证过程通常不是一个[迭代过程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%BF%AD%E4%BB%A3%E8%BF%87%E7%A8%8B&zhida_source=entity)。这种验证通常通过测试或者分析（静态/动态、正式/经验）。  
这里又有一个问题，即这些过程是如何关联的？我们能使用ML评估指标来分析系统的正确性吗？如果系统中集成了不同的AI模型，那么这个问题就尤为重要。或者反过来，我们能从系统验证中得出ML模型不够准确或精确的结论吗？那么，软件系统验证与ML评估如何相关？
  4. **系统演进** VS. **ML模型演进** ：软件的持续变化促成了像DevOps等开发流程的出现，这些流程明确地支持持续集成、持续部署、自动验证和生产中特定类型的测试，如A/B测试。软件评估包括新功能和改进功能的变更以及内部基础结构的变更。这些更改会影响ML模块的更改请求，包含功能和非功能属性。ML工作流侧重于另一种类型的演进：（1）由于数据集变化引起的重新训练需求而演进，来自新环境的新数据可能需要通过再训练来重新[建模](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=8&q=%E5%BB%BA%E6%A8%A1&zhida_source=entity)。（2）模型本身的变化/优化产生的演进，但使用相同的数据集训练。虽然连续的软件演进会以一种可控的方式映射到ML模型的演进(尽管这需要建立程序)，但模型的数据输入的变化，可能会以不受控和意外的方式发生，这可能会直接影响ML模型的准确性和性能。上下文的变更可能需要使用新的数据集重新训练ML模型。然而，重新训练的模型可能会在新的上下文中正常工作，但它的性能在旧的环境中依然会下降。因此，衍生出新的挑战：（1）软件的演进如何影响ML模型的演进？（2）如何控制环境变化对ML模型性能和模型演进的需求？（3）如何控制ML模型的演进以实现对软件系统演进同样的控制水准？
  5. **运营和反馈循环** ：ML工作流和DevOps本身被设计为独立的迭代过程，但它们也相互依赖。数据管理包括数据收集，收集的数据用于在开发的初始阶段为训练提供数据。然而，在许多情况下，新数据是在系统运营期间收集的，尤其是新数据覆盖现有环境数据时，ML模型必须重新训练。ML模型必须部署在开发环境中，才能在开发过程中使用，并需要与可执行系统一起部署。在运营中，系统必须提供有关系统性能和用户对开发环境的接受程度信息，以便为系统的进一步演进提供反馈。系统还需要监控已部署的ML模型，以便检查其在当前环境中的性能。要获得成功和高效的端到端流程，必须不断交换不同类型的信息。这里面临的挑战：（1）流程之间应交换哪些信息？（2）信息交换的触发机制是什么，信息交换的频率是什么？



## 集成ML工作流和DevOps流程

最终的目的，通过 ML工作流和DevOps流程的集成能够实现在生产中快速、迭代和持续地开发、部署和运营基于AI的软件系统。[集成流程](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E9%9B%86%E6%88%90%E6%B5%81%E7%A8%8B&zhida_source=entity)期望在基于AI的系统开发中实现所有流程步骤内和跨流程步骤的自动化，包括构建、测试、部署和基础设施管理。所有对构建基于AI的系统重要的工件（代码、数据和模型）都是可复现的，能够适应并以小增量可靠地发布。集成过程特别适用于基于新的流数据或ML模型参数优化（如更改超参数）重新训练部署ML模型的场景。整个基于AI的系统生命周期将ML工作流和DevOps集成到四个不同的流程中，即数据管理(DM)、ML建模（Mod）、软件开发（Dev）和系统运营（Ops），如图3所示。

图3. ML工作流和DevOps流程集成

### **数据管理(DM)**

[数据管理阶段](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86%E9%98%B6%E6%AE%B5&zhida_source=entity)通过建立活动和系统，来收集数据、选择数据、增加标签和策划将用于训练ML模型的特征。标准化活动和系统有助于以[最优化](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%9C%80%E4%BC%98%E5%8C%96&zhida_source=entity)的格式提取和存储数据，便于在ML建模阶段使用，同时加强数据的适当安全[访问控制](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E8%AE%BF%E9%97%AE%E6%8E%A7%E5%88%B6&zhida_source=entity)。生成的数据集可以作为代码和原始数据的连接器，以便从中选择和提取数据样本。在附录案例A中，所有销售代表与潜在客户之间的邮件通信数据都存储在数据库中，从数据库中选择少量的数据样本进行标注。然而，提取涉及一系列不同的编码，以获取电子邮件文本、提取实体和构建实体周边关系。在附录案例B中，通过从包含[日志文件](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6&zhida_source=entity)和日志分析（由外部工具执行）的大型数据收集存储中提取数据，实现数据采集服务来构建数据集。

### **ML建模（Mod）**

一旦在ML建模阶段提供了新的数据集进行训练，就需要提供训练环境以方便模型训练和评估。ML模型实验可以在ML平台上进行，该平台由内部开发或作为开源工具获取，提供分析数据集和训练ML模型的能力。在每个ML模型实验运行的阶段，数据集、训练模型(包括模型配置)和评估结果都被存储起来，以便与历史基线（例如当前正在生产的模型）进行比较。为了方便在ML模型实验成功完成后跟踪工件依赖关系，开发人员需要显式地将工件及其依赖关系(数据集、模型和代码)指定并注册到依赖关系跟踪系统中，该系统将实验运行历史数据存储在数据库中。对于每个工件，开发人员指定其[元数据](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%85%83%E6%95%B0%E6%8D%AE&zhida_source=entity)（名称、版本号、注册日期）和依赖关系信息，这些信息包含对工件依赖的其他元素的引用。例如，**数据集依赖关系** 信息可以是（1）[数据源](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%BA%90&zhida_source=entity)，例如日志ID，（2）用于从原始数据中提取数据集的代码的Git哈希，（3）版本化的数据标签集和（4）提取脚本的入口点等。对于**存储工件及其依赖关系** 信息来说，重要的是它应该具有不变性，即能够每次重新创建/再现完全相同的工件。版本化的工件和它们的依赖信息可以被其他系统从外部访问，例如，通过简单的API调用构建系统。在模型实验阶段结束时，选择最终的训练模型，并验证该模型在与其他ML组件隔离的情况下工作正常。附录中案例A使用外部开发的AI平台来构建和部署名为Databricks的模型。使用该工具，案例A将执行ML模型实验步骤的整个流程存储为单个存档文件，并将结果模型存储在一个版本控制的存储库中。在附录案例B中，在案例B中，ML建模阶段涉及[数据挖掘](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%8C%96%E6%8E%98&zhida_source=entity)、模型训练和评估等人工步骤。在成功训练一个模型之后，模型连同元数据信息(名称、版本、分类器、特性)也存储在模型[注册表](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%B3%A8%E5%86%8C%E8%A1%A8&zhida_source=entity)中。

### **软件开发（Dev）**

当部署经过训练的模型时，首先要验证它与系统的其他部分(即其他组件)是否正确运行。构建系统在大型测试集上执行系统范围的集成和验证，以给出系统所有部分在不同组件之间如何执行的整体视图。此外，由于构建系统可以访问工件依赖信息和功能，例如使用代码(数据集生成代码、模型训练代码、评估度量生成代码)对工件进行版本控制和存储，因此构建系统可以以这样的方式实现，即它具有定期编排和运行整个模型训练作业的功能。在后者的情况下，重新训练模型将是自动化的，构建过程将被指定为失败、中止或成功。使用最佳模型成功构建—新模型性能优于生产模型—已在模型注册表中注册。如果构建失败或中止，开发人员可以选择使用优化的参数集重新构建模型。CI系统执行的历史构建还可以用来识别和分析由于模型更新而在其他组件中出现的bug。最后，只有在成功构建后，包括通过定期集成和系统测试，才会将经过验证的模型的最新注册版本发布/推广到后续的验收环境或生产环境从而保证质量。定期测试包括测试不同输入的AI组件，以确保适当的响应。在附录中案例A，可以通过重新运行包含Databricks工具生成的整个模型管道步骤的单个存档文件来重新生成训练过的模型。附录中案例B还计划纳入CI系统，以帮助处理由于时间压力而积累的技术债务，这些债务要求团队快速向利益相关者交付系统。例如，正在实施单元测试，以帮助捕获数据摄取服务引起的问题。

### **系统运营（Ops）**

一旦ML系统组件在生产环境中运行(包括训练过的模型)，系统将被持续监视，以检测性能下降等问题。部署到生产环境可以是手动、半自动化或自动化的过程，部署一般在开发流程中的预生产环境中成功执行测试之后进行。Ops阶段还通过提供预测来确保AI组件的实时测试，同时遵守严格的延迟要求。在此步骤中，开发人员还根据[实时数据](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%AE%9E%E6%97%B6%E6%95%B0%E6%8D%AE&zhida_source=entity)收集模型的执行情况信息。这些信息可以用来触发模型的再训练。

## 结论

在本文中，作者演示了ML工作流和DevOps流程的集成，以帮助系统地构建基于人工智能的软件系统。在开发基于ML的软件系统时，该方法还有助于解决一些已确定的挑战。特别是ML模型实验和部署期间的手动步骤，包括解决ML工件的版本控制和依赖管理问题。本文也希望通过进一步的讨论，以解决基于人工智能系统发展中的许多挑战。

## 参考案例

案例A：外出（OOO）回复检测。OOO回复[检测系统](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E6%A3%80%E6%B5%8B%E7%B3%BB%E7%BB%9F&zhida_source=entity)是基于web的销售合作平台中的一个人工智能组件，用于优化销售代表和潜在客户之间的沟通。AI组件从OOO电子邮件回复中提取信息，如联系人和日期，并自动提示销售代表采取相关操作，如添加联系信息，在返回日期暂停和恢复销售行动的自动顺序。

案例B：退回电信硬件中的故障[分类系统](https://zhida.zhihu.com/search?content_id=213705393&content_type=Article&match_order=1&q=%E5%88%86%E7%B1%BB%E7%B3%BB%E7%BB%9F&zhida_source=entity)。退回的电信硬件故障分类系统。人工智能组件根据无线网络的日志数据对返回硬件中的问题(故障)进行分类。筛选操作员作为最终用户，将返回的硬件与AI组件连接起来，并获得分数，以判断它是软件相关故障、硬件相关故障还是无故障。如果硬件良好（没有故障），则将其送回客户；但如果硬件损坏，则将其送去维修。

## 参考资料

  1. Lucy Ellen Lwakatare, Ivica Crnkovic, Jan Bosch, DevOps for AI – Challenges in Development of AI-enabled Applications
  2. 中国信通院《中国DevOps现状调查报告（2022年）》



  


> 上一篇：[【AI工程论文解读】02-聊聊机器学习系统中的技术债](https://zhuanlan.zhihu.com/p/532922813)  
> 下一篇：[【AI工程论文解读】04-通过Ease.ML/CI实现机器学习模型的持续集成（上）](https://zhuanlan.zhihu.com/p/581822019)
