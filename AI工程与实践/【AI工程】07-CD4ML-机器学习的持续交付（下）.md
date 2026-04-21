# 【AI工程】07-CD4ML-机器学习的持续交付（下）

原文链接：https://zhuanlan.zhihu.com/p/554459107

---

​

目录

在上一篇《[【AI工程】06-CD4ML-机器学习的持续交付（上）](https://zhuanlan.zhihu.com/p/552173360)》文中，我们介绍了CD4ML的概念和技术组件，本篇主要为大家介绍CD4ML端到端的流程、以及对CD4ML未来的探索和展望。

## 端到端的CD4ML流程

通过逐步解决每个技术挑战，并使用一系列的工具和技术，我们设法搭建了的端到端的CD4ML流程，如图7所示，该流程管理基于三个维度：**代码** 、**模型** 和**数据** 。

 _

图7：机器学习的[持续交付](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=2&q=%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98&zhida_source=entity)端到端流程_

在此基础上，我们需要一种简单的方法来管理、发现、访问和版本化我们的数据。然后，将模型的构建和训练工作自动化，使其可被重用。通过该流程，可以实现同时实验并训练多个模型，并跟踪记录相关数据。确定合适模型之后，就可以考虑如何交付至生产中并提供相应服务。由于模型的不断更新，在部署至生产环境之前，对其进行测试，保障结果符合用户预期。模型投入生产后，需要通过基础设施提供的监控和观测服务，收集并分析新数据，然后用于丰富训练数据集。以此完成整个持续改进的闭环。

持续交付编排工具用于协调端到端CD4ML流程，按需提供基础设施，管理模型和应用程序如何部署到生产环境。

## 路在何方

在本小节中，将重点介绍研讨会材料中未涉及的一些改进领域，以及待进一步探索的开放领域。

### [数据版本控制](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6&zhida_source=entity)

说及CD4ML时，经常会遇到的问题是“当数据发生变化时，如何触发流水线？”

在作者的示例中，他们采取了如下方式。如图5中所示，机器学习流水线从download_data.py脚本开始，其负责从共享区下载训练数据集。如果更改了共享区中数据集的内容，这并不会立即触发流水线，原因是程序代码未更改，DVC也无法检测到它。为了将数据版本化，我们必须创建一个新文件或更改文件名，这反过来又需要我们更新download_data.py脚本中的数据路径，并在此提交代码。

一种更好的改进方法是允许DVC[跟踪文件](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E8%B7%9F%E8%B8%AA%E6%96%87%E4%BB%B6&zhida_source=entity)内容，替代手动写的代码。因此，作者们也将其机器学习流水线稍作修改，如图8所示。

图8：更新第一步以允许DVC跟踪数据版本并简化ML管道

这会创建一个[元数据文件](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E5%85%83%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6&zhida_source=entity)，用于跟踪和校验提交到Git的文件内容。当文件内容发生变化时， DVC会更新该元数据文件，该变更将会触发流水线。

虽然这允许在数据更改时重新训练模型，但并不能解决数据版本控制的全部问题。一方面是数据历史：理想情况下，需要保留所有的数据变更记录，但实际上，往往是不可行的。另一方面是数据来源：了解[处理数据](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE&zhida_source=entity)时哪一步导致了数据变更，以及变更数据如何在不同的[数据集](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=4&q=%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)之间传播。还有一个问题，随着时间的推移，[数据结构](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84&zhida_source=entity)和类型可能发生变化，这些变化是否前后兼容。

在[流媒体](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%B5%81%E5%AA%92%E4%BD%93&zhida_source=entity)领域，数据版本控制变得更加复杂，因此期望在该领域中能有更多的实践、工具和技术。

### 数据流水线

作者们更倾向于在代码中定义数据流水线的开源工具，因为这更容易进行版本控制、测试和部署。例如，如果使用Spark，则可以使用Scala编写数据流水线，同时使用ScalaTest或spark-testing-base对其进行测试，然后可以将job打包为JAR组件，并通过GoCD的部署流水线进行部署。

由于数据流水线通常为批处理作业或作为长时间运行的[流应用程序](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%B5%81%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F&zhida_source=entity)，因此作者们未将它们包含在图8中的端到端CD4ML流程图中。由于流水线的输出可能会发生变化，但该输出并不是模型或者应用程序所期望的，这也会带来一个潜在集成问题。因此，将集成和数据契约测试作为部署流水线的一部分，可以捕捉这部分的错误。

与数据流水线相关的另一种测试类型是[数据质量检查](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E8%B4%A8%E9%87%8F%E6%A3%80%E6%9F%A5&zhida_source=entity)，该主题是一个广泛讨论的话题，可以在另一篇的文章中讨论。

### [平台思维](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E5%B9%B3%E5%8F%B0%E6%80%9D%E7%BB%B4&zhida_source=entity)

实现CD4ML时，需要使用多种工具和技术。如果有多个团队都在尝试做一个系统，那么他们可能会做大量重复的工作，比如查找可用的工具。这里，[平台化思维](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E5%B9%B3%E5%8F%B0%E5%8C%96%E6%80%9D%E7%BB%B4&zhida_source=entity)可以帮助我们。通过平台化思维，集中在平台上构建与领域无关的工具，用它解决工具潜在的复杂性，帮助其他团队更快更容易的尝试和使用。

### 无偏差的演进智能系统

ML系统部署到生产环境后，它将开始预测并处理看不见的数据，它甚至可能替代之前使用的基于规则的系统。有一点不能忽视，即系统中使用的训练数据和模型验证是基于之前系统的历史数据，而这些历史数据可能会被之前系统的固有偏差所影响。此外，[ML系统](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=2&q=ML%E7%B3%BB%E7%BB%9F&zhida_source=entity)对用户产生的影响，也会影响未来的训练数据。

通过如下两个例子来理解影响。

**第一个示例** ，假设有一个应用程序可以预测需求，用于决定要订购和提供给客户的产品数量。如果预测需求低于实际需求，将没有足够的商品出售，因此该商品的销量会减少，如果仅使用这些新交易的数据作为改进模型的数据集，那么随着时间的推移，会导致需求预测数量不断减少。

**第二个示例** ，假设正在构建一个异常[检测模型](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%A3%80%E6%B5%8B%E6%A8%A1%E5%9E%8B&zhida_source=entity)，用于确定客户的信用卡交易是否具有欺诈行为。如果应用程序采用了模型的决策阻止了交易，那么随着时间的推移，将只会有模型允许的交易（“真实标签”）和越来越少的欺诈性交易数据用来训练。由于训练数据会偏向“好的”交易，模型的性能也会下降。

解决上述问题并不容易。第一个示例中，零售商还考虑了缺货的情况，并订购了比预期更多的商品以弥补潜在的短缺。对于欺诈检测场景，有时可以使用[概率分布](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%A6%82%E7%8E%87%E5%88%86%E5%B8%83&zhida_source=entity)来忽略或覆盖模型的分类。同样重要的是，需要认识到很多数据集是时间的，即它们的分布随时间而变化。许多验证方法使用了随机[数据分割](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%88%86%E5%89%B2&zhida_source=entity)，基于他们符合[i.i.d.](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Independent_and_identically_distributed_random_variables)(独立均匀分布)的假设，然而一旦将时间因素考虑在内，这个假设将不再成立。

因此，重要的是，不仅要捕获模型的输入/输出，还要看最后应用是否直接采用模型的输出。可以通过对数据进行注释，在后续的训练中规避这种偏差。面临这些问题时，管理训练数据并拥有一个允许人员管理训练数据的系统是另一个[关键组件](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=%E5%85%B3%E9%94%AE%E7%BB%84%E4%BB%B6&zhida_source=entity)。

随着时间的推移，智能系统的演进和ML模型的改进将被视为“元学习（[meta-learning](https://zhida.zhihu.com/search?content_id=211345865&content_type=Article&match_order=1&q=meta-learning&zhida_source=entity)）”问题。许多该领域的最新研究都集中在这类问题上。例如，强化学习技术的应用（如多臂老虎机），或生产环境中的在线学习。

## 结论

随着机器学习技术不断发展演进，我们对管理和交付此类应用程序到生产环境的知识也在不断积累。通过引入和扩展持续交付的规范和实践，以安全可靠的方法去更好地管理更新发布引入的风险。

使用示例销售预测应用程序，在本文中展示了一系列CD4ML的技术组件，并讨论了一些实现它们的方法。随着ML技术的持续发展，会不断有新工具产生和消失，但持续交付的核心规范将会持续存在价值，在你开发自己的ML应用时，也将会给你提供重要的参考与指导。

## 参考资料

  1. Danilo Sato, Arif Wider, Christoph Windheuser, [Continuous Delivery for Machine Learning](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/cd4ml.html)



  


> 上一篇：[06-CD4ML-机器学习的持续交付（上）](https://zhuanlan.zhihu.com/p/552173360)  
> 下一篇：[08-MLOps工具-在Charmed Kubeflow上运行MindSpore](https://zhuanlan.zhihu.com/p/568398333)
