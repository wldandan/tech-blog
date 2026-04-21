# 【AI工程】06-CD4ML-机器学习的持续交付（上）

原文链接：https://zhuanlan.zhihu.com/p/552173360

---

​

目录

> [持续交付](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98&zhida_source=entity)（CD ,Continuous Delivery），是一种软件工程方法，让[软件产品](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E4%BA%A7%E5%93%81&zhida_source=entity)的产出过程在一个短周期内完成，以保证软件可以稳定、持续的保持在随时可以发布的状况。它的目标在于让软件的构建、测试与发布变得更快、更频繁。这种方式可以减少软件开发的成本与时间，减少风险。

## 概述

在上一篇《[聊聊机器学习系统中的技术债](https://zhuanlan.zhihu.com/p/532922813)》文中，我们介绍了在构建可靠的机器学习系统时存在的[技术债](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=2&q=%E6%8A%80%E6%9C%AF%E5%80%BA&zhida_source=entity)。最近阅读了《Continuous Delivery for Machine Learning》文章，本篇文章主要介绍了**如何实现机器学习的持续交付** 。

近年来，随着AI相关技术快速演进，机器学习应用也越来越流行。然而，和传统的软件（如Web服务或移动应用程序）相比，机器学习应用的开发、部署和持续改进过程会更加复杂。其复杂性通常体现在三个**可变** 的维度：**代码本身、模型和数据** 。由于其可变性，导致了它们的行为通常很复杂且难以预测，而且更难测试、更难解释、更难改进。机器学习的持续交付（CD4ML）采取将传统软件的持续交付原则和实践引入到机器学习应用中的方式，建立的适用于机器学习应用的基本原则。

## CD4ML概念

在Sculley等谷歌工程师于2015年发表的《[Hidden Technical Debt in Machine Learning Systems](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=Hidden+Technical+Debt+in+Machine+Learning+Systems&zhida_source=entity)》论文中，他们强调在实际的机器学习系统中，只有小部分代码由ML代码组成，其周围所需要的基础设施庞大而复杂。他们还讨论了此类系统中可能积累的技术债源头，如[数据依赖性](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E4%BE%9D%E8%B5%96%E6%80%A7&zhida_source=entity)、模型复杂性、可复现性、监控、测试和应对外部世界的变化等。

Jez Humble和David Farley在其开创性著作中《持续交付》中指出：

> “ _持续交付是一种能够以可持续的方式安全、快速地将所有的变更（包括新功能、配置更改、错误修复和实验）交付至生产或用户的能力。_ ”

除了代码之外，[ML模型](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=ML%E6%A8%A1%E5%9E%8B&zhida_source=entity)和训练数据的变更是另一种需要管理并融入软件交付过程的更改，如下图1所示。

 _

图1：在机器学习应用中3个可变的维度和引起变化的部分原因：数据、模型和代码_

考虑到上述原因，可以扩展持续交付的定义，纳入现实世界机器学习系统中存在的[新元素](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%96%B0%E5%85%83%E7%B4%A0&zhida_source=entity)和挑战，定义机器学习的持续交付（CD4ML）。

>  _**机器学习的持续交付(CD4ML)** 是一种软件工程方法，通过该方法，让各跨职能团队以小而安全的增量迭代方式完成基于代码、数据和模型的机器学习[应用程序开发](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%8F%91&zhida_source=entity)，这些应用程序可以在一个短周期内完成，并保持在随时可以再迭代和可靠发布的状态。_

该定义包含以下基本原则：

  * **软件工程方法** ：促使团队高效地产出高质量的软件。
  * **跨职能团队** ：跨数据工程、数据科学、机器学习工程、开发、运营和其他知识领域，且具有不同技能和工作流程的专家共同协作，充分发挥团队中每个成员的技能和优势。
  * **基于代码、数据和机器学习模型开发应用** ：ML[软件开发过程](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E8%BF%87%E7%A8%8B&zhida_source=entity)中的所有组件都需要不同的工具和工作流，需要进行相应的[版本控制](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6&zhida_source=entity)和管理。
  * **小而安全的[增量迭代](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=2&q=%E5%A2%9E%E9%87%8F%E8%BF%AD%E4%BB%A3&zhida_source=entity)**：一款软件产品的发布通常会经过多次小增量的迭代发布，这也促成每次迭代发布产生的变化结果具有可见性和可控制性，从而增加流程的安全性。
  * **可复现且可靠的软件发布** ：虽然模型输出结果具有不确定性且难以复现，但将ML软件发布到生产环境中的过程是可靠且可复现的，同时，应尽可能利用自动化。
  * **随时发布软件** ：保持可以随时将ML软件交付生产至关重要。软件持续的保持在随时可以发布的状态，让[业务决策](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E4%B8%9A%E5%8A%A1%E5%86%B3%E7%AD%96&zhida_source=entity)何时发布产品，避免让技术成为其瓶颈。
  * **短周期** ：短周期意味着开发周期以天甚至小时为单位，而不是周、月甚至年。可靠的自动化流程是实现这一目标的关键。通过生产环境中获得的结果来调整ML模型，可以形成一个良好[反馈循环](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%8F%8D%E9%A6%88%E5%BE%AA%E7%8E%AF&zhida_source=entity)。



在本文中，将介绍实现CD4ML时发现的重要技术组件，并根据示例ML应用程序来解释并演示如何将不同的工具结合用于实现完整的[端到端](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%AB%AF%E5%88%B0%E7%AB%AF&zhida_source=entity)流程。同时，作者介绍了所选工具的替代工具，以及他们基于在行业的实践在该领域最新研究成果。

## CD4ML示例：用于销售预测的机器学习应用程序

如何将持续交付运用到构建机器学习系统中，文章的作者们发布了一个用于展示ML的[案例研究](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%A1%88%E4%BE%8B%E7%A0%94%E7%A9%B6&zhida_source=entity)，该案例用于预测在其平台上销售的汽车价格。该示例基于公开的问题和[数据集](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)构建，用于说明CD4ML实现过程。

示例基于监督学习算法和流行的Scikit-learn Python库，使用标记的输入数据训练预测模型，将该[模型集成](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E9%9B%86%E6%88%90&zhida_source=entity)到一个简单的 Web 应用程序中，然后将其部署到云中的生产环境中。如下图2展示了此流程。

 _

图2：训练ML模型、将其与web应用程序集成并部署到生产中的流程_

部署完成后，用户可以在Web应用中选择产品和预测日期，该模型将输出其对该产品当天销售量的预测。

 _

图3：Web应用界面图_

### 挑战

端到端实施的过程中，存在两个挑战。

第一个挑战是**[组织结构](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%BB%84%E7%BB%87%E7%BB%93%E6%9E%84&zhida_source=entity)** ：不同的团队可能参与流程中的不同部分，并且存在[交接成本](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E4%BA%A4%E6%8E%A5%E6%88%90%E6%9C%AC&zhida_source=entity)。可能会导致产生延迟交付和大量的团队交流成本，阻碍将ML应用程序部署到生产中的端到端过程自动化的能力。

 _

图4：大型组织中的常见职能孤岛障碍_

第二个挑战是**技术层面** ：如何使流程具有可[复用性](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%A4%8D%E7%94%A8%E6%80%A7&zhida_source=entity)和可审计性。由于各个团队使用不同的工具并遵循不同的工作流程，因此很难实现端到端的自动化。除代码之外，还有更多的组件需要管理，并且对组件的版本控制也很难。

本文主要聚焦于探讨技术层面的挑战和解决方案。为此，将会深入研究每个[技术组件](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=2&q=%E6%8A%80%E6%9C%AF%E7%BB%84%E4%BB%B6&zhida_source=entity)，并逐步改进和扩展端到端流程，使其更加完善。

## CD4ML的技术组件

### 可发现和可访问的数据

对于机器学习而言，[数据源](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%BA%90&zhida_source=entity)是核心部分。如何引入[外部数据源](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%A4%96%E9%83%A8%E6%95%B0%E6%8D%AE%E6%BA%90&zhida_source=entity)也很重要。作者发现了一些收集和获取可用数据的常见模式，例如[数据湖](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%B9%96&zhida_source=entity)（Data Lake）架构、传统的[数据仓库](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E4%BB%93%E5%BA%93&zhida_source=entity)、实时数据流集合、或者作者尝试的去中心化的[数据网格](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%BD%91%E6%A0%BC&zhida_source=entity)（Data Mesh）架构。

> ** _数据流水线（_ Data Pipelines _）_**  
>  _流水线在计算机领域是一个常见的术语，尤其是在 ML 应用程序中。“数据流水线”定义为将输入数据通过一系列的转换，然后将生成的数据作为输出的过程。输入和输出数据都可以获取并存储在不同的地方，例如数据库、流、文件等。数据的转换过程通常在代码中实现，一些ETL工具也支持以图形化方式处理。_

对于CD4ML，作者们将数据流水线作为一个组件进行管理，同时对其进行版本控制、测试并部署到目标执行环境。

在示例中，作者们进行[数据分析](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%88%86%E6%9E%90&zhida_source=entity)之后，决定将多个文件合并成为单个CSV文件，并清理了无关数据和可能在引入有害噪声的数据（例如负销售）。同时，将输出的数据文件存储在[云存储系统](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E4%BA%91%E5%AD%98%E5%82%A8%E7%B3%BB%E7%BB%9F&zhida_source=entity)中。作者们使用此文件作为输入训练数据的快照，基于文件夹结构和文件命名约定，并设计出一种简单的方法来对数据集进行版本控制。

在现实世界中，可能需要搭建更复杂的Data Pipelines（数据流水线），便于[数据科学家](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%A7%91%E5%AD%A6%E5%AE%B6&zhida_source=entity)轻松的从多个数据源获取数据并使用。

### 可复现的模型训练

>  _**机器学习流水线（Machine Learning Pipelines）**_  
>  _“机器学习流水线”，也称为“模型训练流水线”，是以数据和代码为输入，生成经过训练的 ML 模型作为输出的过程。这个过程通常涉及数据清洗和预处理、[特征工程](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E5%B7%A5%E7%A8%8B&zhida_source=entity)（Feature Engineering）、选择模型和[算法](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=2&q=%E7%AE%97%E6%B3%95&zhida_source=entity)、优化和[评估模型](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%AF%84%E4%BC%B0%E6%A8%A1%E5%9E%8B&zhida_source=entity)。_

数据准备好之后，即可进入模型构建中的迭代数据科学的工作流。通常将数据分为训练集和[验证集](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E9%AA%8C%E8%AF%81%E9%9B%86&zhida_source=entity)，并尝试组合使用不同的算法、调整其参数和超参数。从而产生一个可以被验证集评估的模型，以评估模型预测的质量。机器学习(ML)流水线就是一步步迭代训练模型的过程。

在[图5](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/cd4ml.html%23ml-pipeline-1.png)中展示了如何为销售预测问题构建ML管道，其中包含不同的代码、数据和模型组件。输入数据、中间训练和验证集数据以及输出模型。此外，流水线的各个阶段通常处于不断变化中，这使得数据科学家很难在本地环境之外进行复现。

图5：销售预测问题的机器学习流水线以及使用DVC实现自动化的3个步骤

为了在代码中正规化管理模型训练过程，使用DVC(Data Science Version Control)的开源工具。它提供了与Git类似的语义，也解决了一些ML特定的问题。

  * 多个后端插件，用于在[源代码](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%BA%90%E4%BB%A3%E7%A0%81&zhida_source=entity)管理存储库之外的外部存储器上获取和存储大型文件。
  * 可以[跟踪文件](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%B7%9F%E8%B8%AA%E6%96%87%E4%BB%B6&zhida_source=entity)的版本，允许我们在数据更改时重新训练模型。
  * 跟踪用于执行ML流水线的[依赖关系图](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB%E5%9B%BE&zhida_source=entity)和命令，允许在其他环境中复现该过程。
  * 它可以与Git分支集成，允许多个实验共存。



也可以使用Pachydrm和MLflow开源工具来解决上述问题。

### 模型服务

确定合适的模型之后，需要决定如何在生产环境中使用。如下是实现上述目标的几种模式：

  * **模型内嵌** ：一种简单的方法，将模型视为构建和打包应用程序的依赖。即可以将应用程序构件和版本视为应用程序代码和所选模型的组合。
  * **模型部署为单独的服务** ：在该方法中，模型封装在一个服务中，该服务可以独立于应用程序进行部署，因此，可以实现独立发布模型更新。但由于每次预测都需要进行[远程调用](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%BF%9C%E7%A8%8B%E8%B0%83%E7%94%A8&zhida_source=entity)，可能在推理时会引入延迟。
  * **模型作为数据发布** ：在该方法中，模型也被独立处理和发布，但应用程序将在运行时将其作为数据读取。在流式/实时场景中已经可以看到这种用法，应用程序可以订阅新模型版本发布事件，在读取新的模型到内存中之前，会继续使用旧版本模型进行预测。蓝绿部署（Blue Green Deployment）或[金丝雀](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E9%87%91%E4%B8%9D%E9%9B%80&zhida_source=entity)发布（Canary Releases）等软件发布模式也适用于此场景。



作者们采用了更简单的模型内嵌的方法，其模型导出为[序列化对象](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%BA%8F%E5%88%97%E5%8C%96%E5%AF%B9%E8%B1%A1&zhida_source=entity)（pickle文件）并由 DVC 推送到存储。除了pickle之外，MLeap和H2O等也可以实现模型内嵌模式。除此之外，MLflow Models提供一种标准方法来打包不同类型的模型，如“[模型即服务](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E5%8D%B3%E6%9C%8D%E5%8A%A1&zhida_source=entity)”和“模型内嵌”，供不同的下游工具使用。这是当前的开发领域，各种工具和供应商正在努力完成的任务。

无论决定使用哪种模式，模型与其消费者之间总是存在隐含的契约。该模型通常会期望输入数据符合指定要求，如果数据科学家改变了该契约从而需要新的输入或者新的特征，则可能会出现[应用集成](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%BA%94%E7%94%A8%E9%9B%86%E6%88%90&zhida_source=entity)问题，并导致应用程序损坏。

### [集成测试](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95&zhida_source=entity)

可以在 ML 工作流程中引入不同类型的测试。尽管存在不确定性及难以自动化的问题，但还是有很多可用的[自动化测试](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95&zhida_source=entity)，有助于提升ML 系统的价值及整体质量。

  * **验证数据** ：我们可以添加测试，根据预期模式来验证输入数据，或验证我们对其有效值的假设。例如，它们在预期范围内，或不为空。对于工程特征值，我们可以编写单元测试来检验其是否计算正确。例如，对数字[特征缩放](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E7%BC%A9%E6%94%BE&zhida_source=entity)或归一化，独热编码向量（one-hot encoded vectors）包含全零和单个1，或者使用合适的值替换缺失值。
  * **验证[组件集成](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%BB%84%E4%BB%B6%E9%9B%86%E6%88%90&zhida_source=entity)**：我们可以使用类似的方法来测试不同服务之间的集成，使用[契约测试](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%A5%91%E7%BA%A6%E6%B5%8B%E8%AF%95&zhida_source=entity)（Contract Tests）来验证预期的模型接口与使用中的应用程序的兼容性。另一种相关测试类型，用于确保即使将模型以不同的格式发布到生产环境中，也仍然可以产生相同的结果。这可以通过在相同的验证数据集上运行原始模型和生产模型，并对比他们的结果来实现。
  * **验证模型质量** ：虽然ML模型的性能具有不确定性，但数据科学家通常会收集和监控一系列指标来评估模型的性能，例如错误率、准确性、AUC、ROC、[混淆矩阵](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%B7%B7%E6%B7%86%E7%9F%A9%E9%98%B5&zhida_source=entity)、精度、召回率等。它们在参数和超参数调优过程中也很有用。作为简单的质量门，可以在流水线中引入[阈值测试](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E9%98%88%E5%80%BC%E6%B5%8B%E8%AF%95&zhida_source=entity)或棘轮，以确保新模型性能基线不会降级。
  * **验证模型偏差和公平性** ：相对于在整体测试和验证数据集上获得良好的表现，检查模型在特定[数据切片](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%88%87%E7%89%87&zhida_source=entity)上的基准表现也很重要。例如，在训练数据中可能存在固有的偏差，特征的给定值（例如种族、性别或地区）与真实世界中的实际分布相比，有更多的数据点，因此对不同数据切片的表现进行检查尤其重要。像Facets工具可以帮助你对数据进行[可视化](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%8F%AF%E8%A7%86%E5%8C%96&zhida_source=entity)的切片，发现数据集中不同[特征值](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=2&q=%E7%89%B9%E5%BE%81%E5%80%BC&zhida_source=entity)的分布。



有了更多类型的测试，将使你重新思考测试[金字塔](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E9%87%91%E5%AD%97%E5%A1%94&zhida_source=entity)的构成，如图6所示：你可以考虑为每种类型的组件（代码、模型和数据）分别设计单独的金字塔，也可以考虑如何组合它们。总的来说，ML系统的测试和质量更为复杂，可以作为另一主题深入探讨。

 _

图6：CD4ML中如何组合不同测试金字塔的示例_

### 实验跟踪

为了建立起实验[管理流程](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%AE%A1%E7%90%86%E6%B5%81%E7%A8%8B&zhida_source=entity)，实验信息的捕捉和记录尤其重要。开发者们将会用它决定该将哪个模型部署到生产环境中。由于[数据科学](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=7&q=%E6%95%B0%E6%8D%AE%E7%A7%91%E5%AD%A6&zhida_source=entity)工作以研究为中心，因此通常会同时有多个实验共存的情况，其中很多实验可能永远无法投入生产。

研究阶段的实验方法不同于传统的软件开发过程，其实验过程中很多实验代码都将被丢弃，只有少数代码被认为值得投入生产。因此，需要定义一种方法来跟踪它们。

在示例中，作者们采用DVC建议的方法，使用不同的Git分支来跟踪源代码中的不同实验。DVC可以从不同分支或标签的实验中获取和显示指标，从而可以轻松地在它们之间切换。对于ML实验，预计大多数分支永远不会被集成，并且实验之间的代码变化通常很小。从自动化[持续集成](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity)角度看，确实希望为每个实验训练多个模型，并收集指标以告知哪个模型可以被用于下一阶段部署。

### 模型部署

在作者给出的简单示例中，只尝试了构建一个模型，将该模型嵌入至应用程序中并部署。但在现实世界中，部署可能会面临更复杂的场景：

  * **多个模型** ：有时可能有多个模型执行相同的任务。例如，针对每一个产品都训练一个模型。在这种情况下，将模型部署为单独的服务，使用应用程序通过独立的 API 调用并获取预测结果可能会更合适。然后，你可以决定哪些模型在已发布的接口中被使用。
  * **影子模型** ：在考虑替换生产环境中的模型时，会用到该模式。你可以将新模型与当前模型并列部署，将新模型作为影子模型。发送相同的生产数据给两个模型，同时收集影子模型的表现数据，然后再以此决定是否将旧模型升级为新模型。
  * **竞争模型** ：该模型适用于当你尝试在生产环境中使用某个模型的多个版本（如 A/B 测试），试图找出哪个版本更好这一稍微复杂的场景。这里的难点来自于基础设施和路由规则，即如何确保将流量重新定向到正确的模型，同时，可能需要花费一些时间，确保收集足够的数据用于做出有[统计学](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%BB%9F%E8%AE%A1%E5%AD%A6&zhida_source=entity)意义的决策。评估多个竞争模型的另一种流行方法是多臂老虎机问题（Multi-Armed Bandits），其也需要定义一种方法来预测和监控各个使用模型的结果。将竞争模型应用于ML是一个活跃的研究领域，已经有一些工具和服务出现，例如Seldon core和Azure Personalizer。
  * **在线学习模型** ：与我们前面讨论的模型不同，在线学习模型是经过离线训练产生，用于在线预测，在线学习模型使用的算法和技术可以随着新数据的到来而不断提高其性能。他们在生产中不断学习。但这带来了额外的复杂性，因为如果没有向模型提供相同的数据，作为一个静态组件的版本化的模型将产生不同的结果。你不仅需要对训练数据进行版本控制，还需要对会影响模型性能的生产数据进行版本控制。



为了支持更复杂的部署场景，使用弹性基础设施将使你收益匪浅。除了可以在生产中运行多个模型之外，弹性基础设施还支持按需扩缩容，从而增加[系统的可靠性](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%8F%AF%E9%9D%A0%E6%80%A7&zhida_source=entity)和可扩展性。

### 持续交付编排

>  _**部署流水线（Deployment Pipeline）**_  
>  _“部署流程线”定义为软件从版本控制到投入生产的自动化流程，包含在各个阶段，提交、测试和部署到不同环境。_  
>  _在CD4ML中，可以将自动化和手动ML治理阶段建模到我们的部署流水线中，检测模型偏差、合理性，或者引入可解释性，帮助[决策模型](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E5%86%B3%E7%AD%96%E6%A8%A1%E5%9E%8B&zhida_source=entity)是否可以投入至生产环境。_

在所有主要构建模块就绪后，就需要使用持续交付的编排工具将所有模块串在一起。在这个领域中，有很多工具可供选择，其中大部分都提供了配置和执行部署流水线的方法，用以构建并发布软件到生产环境。而在CD4ML中，存在一些其他编排需求：**配置搭建基础设施和执行ML流水线用于训练并捕获不同模型实验的指标；数据流水线的构建、测试和部署；通过不同类型的测试、验证，确定要采用的模型；配置搭建基础设施和将模型部署至生产环境** 。

在 CD4ML 中，我们有额外的编排要求：基础设施的配置和机器学习管道的执行，以训练和捕获来自多个模型实验的指标；我们的数据管道的构建、测试和部署过程；不同类型的测试和验证来决定推广哪些模型；提供基础设施并将我们的模型部署到生产中。

作者们选择GoCD作为他们的持续交付工具，因为它是以流水线概念为首要设计目标构建的。不仅如此，GoCD还允许组合编排不同的流水线、提供多种[触发器](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%A7%A6%E5%8F%91%E5%99%A8&zhida_source=entity)和在流水线各阶段定义手动或自动升级方法，来实现配置复杂的工作流和依赖项。 

最后，持续交付编排的另一个方面是定义回滚流程，以防部署的模型在生产中表现不佳或不正确。这也为整个交付过程增加了另一个安全保障。

### 模型监控和观测

模型上线后，就需要了解它在实际生产环境中的表现并闭环整个数据反馈循环。在这里，可以复用现有应用程序和服务的基础设施，提供的所有监控和观测所需的基础设施。

运行中的系统通常会使用日志聚合和指标收集工具来捕获数据，例如业务KPI、软件可靠性、性能指标、故障调试信息和异常情况下触发报警的其它指标。我们还可以使用这些工具来捕获数据，用于了解模型的表现，例如：

  * **模型输入** ：通过给模型提供的数据，了解全部模型训练存在的偏差。**模型输出：** 模型从输入中得到预测和建议，以了解模型在真实数据上的表现。
  * **模型可解释性输出** ：例如模型的各项系数、ELI5 或 LIME 输出等指标，允许进一步研究以了解模型如何进行预测，以识别在训练期间未发现的潜在过度拟合或偏差。
  * **模型输出和决策** ：基于给定的生产环境的输入数据，我们的模型做出了哪些预测，以及根据这些预测做了哪些决策。有时应用程序可能会选择忽略模型并根据预定义的规则做出决定（或避免未来出现的偏差）。
  * **用户行为和奖励** ：基于进一步的用户行为，可以捕获奖励指标以了解模型是否达到预期的效果。例如，如果我们显示产品推荐，那么可以跟踪用户何时决定购买推荐产品作为奖励。
  * **模型公平性** ：针对可能存在偏颇的已知特征（例如种族、性别、年龄、收入群体等）分析输入数据和输出预测。 



当在生产中部署了多个模型时，收集、监控和观察数据变得尤为重要。例如，可能有一个影子模型需要评估，或者可能正在进行拆分测试，或者使用多个模型进行多臂老虎机（multi-arm bandit）实验。

如果你在[边缘设备](https://zhida.zhihu.com/search?content_id=210838087&content_type=Article&match_order=1&q=%E8%BE%B9%E7%BC%98%E8%AE%BE%E5%A4%87&zhida_source=entity)（例如终端设备）训练或运行联合模型，或者正在部署在线学习模型，这些模型会不断学习生产环境中的新数据，并随时间推移而发生改变。

通过捕获这些数据，您可以闭环整个数据反馈循环。它可以通过收集更多真实数据（例如在定价引擎或推荐系统）实现，或通过在循环中添加人员来分析从生产中捕获的新数据，创建用于制作新模型或改进模型的新训练数据集而实现。关闭环数据反馈循环是CD4ML一大优势，因为它使得人们能够根据实际生产数据中获取的经验来调整模型，从而实现持续改进的目的。

我们将在下一篇文章继续为大家分享CD4ML，敬请关注。我们之前已经发布了一系列的AI工程相关文章，欢迎大家查看，详见专栏[AI工程与实践](https://www.zhihu.com/column/c_1488835248573706240)。

## 参考资料

  1. Danilo Sato, Arif Wider, Christoph Windheuser, [Continuous Delivery for Machine Learning](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/cd4ml.html)



  


> 上一篇：[05-基于MindSpore的Resnet-50模型分布式训练实践](https://zhuanlan.zhihu.com/p/531465000)  
> 下一篇：[07-CD4ML-机器学习的持续交付（下）](https://zhuanlan.zhihu.com/p/554459107)
