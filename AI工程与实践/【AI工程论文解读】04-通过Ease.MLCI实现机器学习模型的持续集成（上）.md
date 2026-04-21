# 【AI工程论文解读】04-通过Ease.ML/CI实现机器学习模型的持续集成（上）

原文链接：https://zhuanlan.zhihu.com/p/581822019

---

​

目录

>  _[持续集成](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity) 是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，[自动化测试](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95&zhida_source=entity))来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。_

在上一篇《[【AI工程论文解读】03-DevOps for AI-人工智能应用开发面临的挑战](https://zhuanlan.zhihu.com/p/565078936)》文中，我们介绍了DevOps和ML工作流结合的解决方案和实践。本篇论文《Continuous Integration Of Machine Learning Models With Ease.ML/CI: Towards A Rigorous Yet Practical Treatment》，将分享一个用于机器学习的[持续集成系统](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%B3%BB%E7%BB%9F&zhida_source=entity)ease.ml/ci。

在现代[软件工程](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%B7%A5%E7%A8%8B&zhida_source=entity)实践中，持续集成是系统地管理软件系统开发生命周期不可或缺的部分。机器学习模型的开发生命周期同样包括设计、实现、调优、测试和部署。然而，大多数现有的持续集成引擎并不能友好的支持机器学习模型的开发。

在本文中，作者们介绍了用于机器学习的持续集成系统ease.ml/ci，一种用于该领域的特定声明性[脚本语言](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%80&zhida_source=entity)，允许用户指定具有可靠性约束的测试条件，可以将实际[生产系统](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E7%94%9F%E4%BA%A7%E7%B3%BB%E7%BB%9F&zhida_source=entity)中普遍使用的测试条件所需的标记数量降低两个[数量级](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%95%B0%E9%87%8F%E7%BA%A7&zhida_source=entity)。

## 概述

在现代软件工程中，持续集成（CI）是系统地管理开发工作生命周期的[最佳实践](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5&zhida_source=entity)的重要组成部分。使用CI引擎时，要求开发人员每天至少一次将其代码集成（即提交）到共享存储库中。每次提交都会触发代码的自动构建，然后运行预定义的[测试套件](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%B5%8B%E8%AF%95%E5%A5%97%E4%BB%B6&zhida_source=entity)。开发人员每次提交后会接收到一个通过/失败信息，保证每次提交都满足产品部署所需的属性，或者下游软件假定的必要属性。

开发机器学习模型与开发传统软件流程大体一致，因为它也是一个涉及设计、实现、调优、测试和部署的完整生命周期。随着机器学习模型更多用于以任务型为主的应用程序中，并且与传统[软件栈](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E6%A0%88&zhida_source=entity)更加紧密地集成，因此，通过系统的、严格的工程规则来管理ML开发生命周期变得越来越重要。

图1 ease.ml/ci工作流

在本文中，作者们构建了第一个机器学习持续集成系统。该系统的工作流在很大程度上遵循传统的CI系统（图1），它允许用户定义机器学习特定的测试条件，例如新模型最多只能更改旧模型的10%预测，或者新模型的精度必须比旧模型至少高1%。在机器学习模型/程序的每次提交后，系统会自动测试是否通过这些测试条件，并向开发人员反馈通过/失败信息。与传统的CI不同，机器学习的CI本质上是概率的。因此，所有测试条件都是根据用户的 (\epsilon, \delta) 可靠性要求进行评估的，其中 1-\delta （例如 0.9999 ）是有效测试的概率， \epsilon 是[容错率](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E5%AE%B9%E9%94%99%E7%8E%87&zhida_source=entity)（ _即_(1-\delta)— _[置信区间](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E7%BD%AE%E4%BF%A1%E5%8C%BA%E9%97%B4&zhida_source=entity) 的长度_)。CI引擎的目标是返回满足(\epsilon, \delta)—可靠性要求的通过/失败信息。

**技术挑战：** 对于每个提交的模型，从测试集中提取 N 个标记的数据点，得到新模型的精度(\epsilon, \delta)的[估计值](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E4%BC%B0%E8%AE%A1%E5%80%BC&zhida_source=entity)，并测试它是否满足测试条件。这种策略的挑战是与标记复杂性相关的实用性（即N有多大）。要获得 [0,1] 范围内的[随机变量](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E9%9A%8F%E6%9C%BA%E5%8F%98%E9%87%8F&zhida_source=entity)（\epsilon=0.01,\delta = 1-0.9999）的估计值，如果我们简单地应用Hoeffding[不等式](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E4%B8%8D%E7%AD%89%E5%BC%8F&zhida_source=entity)，我们需要用户提供超过46K的标记（类似地，非自适应方式时，32个模型需要63K标记；完全自适应方式时，32个模型需要156K标记）。

作者们从系统和机器学习的角度做出了贡献。

  1. **系统贡献** ：作者们提出了一种新的[系统架构](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84&zhida_source=entity)，以支持ML系统。
  2. **机器学习贡献** ：在机器学习方面，作者们开发了简单的[优化技术](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E4%BC%98%E5%8C%96%E6%8A%80%E6%9C%AF&zhida_source=entity)，以优化可以在特定领域语言中表达的测试条件。作者们的技术涵盖了不同的[交互模式](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E4%BA%A4%E4%BA%92%E6%A8%A1%E5%BC%8F&zhida_source=entity)（完全自适应、非自适应和混合场景），以及许多工业和学术合作伙伴认为有用的流行测试条件。对于测试条件的子集，能够在系统所需的标记数量上节省多达两个数量级。



在本文的后续部分，分别介绍了ease.ml/ci的设计、基本实现、更高级的优化、实验（验证技术的正确性和有效性）和其他相关工作等。

## [系统设计](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1&zhida_source=entity)

本节介绍了ease.ml/ci的设计。首先介绍了[交互模型](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E4%BA%A4%E4%BA%92%E6%A8%A1%E5%9E%8B&zhida_source=entity)和工作流程，如图1所示。然后，介绍了支持用户交互的脚本语言。并讨论了单个元素的语法和语义，以及它们的实现和可能的扩展。最终得到了两个系统实用程序，一个“[样本量](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%A0%B7%E6%9C%AC%E9%87%8F&zhida_source=entity)估计器”和一个“新测试集警报器”，其技术细节将会在后续文中介绍。

### 交互模型

ease.ml/ci是一个用于机器学习的持续集成系统。它的工作流包含四步：（1）用户在测试配置脚本中描述与[ML](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=5&q=ML&zhida_source=entity)模型质量相关的测试条件；（2）用户提供N个测试用例，其中N由系统根据给定的配置脚本自动计算得出；（3）每当开发人员提交/检入更新的ML模型/程序时，系统会触发构建；（4）系统测试是否满足测试条件，并向开发人员返回“通过/失败”信息。如果当前测试集由于重复评估而失去其“统计能力”时，系统会决定何时向用户请求新的测试集。旧的测试集可以作为用于开发新模型的[验证集](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E9%AA%8C%E8%AF%81%E9%9B%86&zhida_source=entity)发布给开发人员。

为此，特意定义了两个团队：**集成团队** ，负责提供测试集并设置可靠性需求；**开发团队** ，负责提交新模型。在实践中，这两个团队可以是相同的，但是在本文中，将会区分这两个团队，尤其是在完全适应的情况下，会将集成团队描述为用户，将开发团队描述为开发人员。

### ease.ml/ci脚本

ease.ml/ci为用户提供了一种声明性的方法，可以根据一组测试用例指定新机器学习模型的需求。然后，ease.ml/ci将这些规范编译成一个实用的工作流，使得测试用例的评估具有严格的理论保障。本文给出了ease.ml/ci脚本语言的设计，其通过Travis CI使用可扩展.travis.yml格式进行实现。

**[逻辑数据模型](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E9%80%BB%E8%BE%91%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B&zhida_source=entity)**

ease.ml/ci脚本的核心是用户指定的持续[集成测试](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95&zhida_source=entity)条件。在当前版本中，由三个变量指定该条件 \nu ={ n,o,d }。 n,o,d\in [0,1]

（1） n ：新模型的精度。（2） o ：旧模型的精度；（3） d ：相对于旧模型，新模型变更占比（百分比）。

关于语法及其语义的详见**附录A** 。

**[自适应](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=5&q=%E8%87%AA%E9%80%82%E5%BA%94&zhida_source=entity) 与非自适应集成**

ease.ml/ci与传统持续集成系统的一个显著区别是，当新模型是否通过持续集成测试的结果发布给开发人员时，测试数据集的统计能力将下降。如果开发人员愿意，可以调整下一个模型，以增加其通过测试的概率。由于需要更大的测试集，确保完全自适应情况下的通过概率将会成本更高。ease.ml/ci允许用户通过参数（full、none、firstChange）指定测试是否采用自适应：

  * 如果参数设置为full，ease.ml/ci会立即向开发人员发布新模型是否通过测试。
  * 如果参数设置为none，ease.ml/ci会接受所有提交，但会将模型是否真正通过测试的信息发送到用户指定的第三方电子邮件地址，开发人员可能无法访问该邮件。
  * 如果参数设置为firstChange，则在测试第一次通过（或失败）之前，ease.ml/ci允许完全自适应，但在测试之后会停止，并需要新的测试集。



**示例脚本**

ease.ml/ci脚本通过在Travis CI中添加ml部分，采用可扩展的.travis.yml文件设计来实现的。例如：
    
    
    ml:
      - script : ./test_model.py
      - condition : n - o > 0.02 +/- 0.01
      - reliability: 0.9999
      - mode : fp-free
      - adaptivity : full
      - steps : 32

此脚本指定了一个连续测试过程，如上所示，该过程的有效测试概率大于 0.9999 ，且仅在新模型的精度比旧模型高两点时，才会接受新的提交。该估计是以“false-positive free”的方式进行，[误差](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%AF%AF%E5%B7%AE&zhida_source=entity)预计在一个精度点内。关于“mode”，在附录A.2中给出了fp-free和fn-free两种模式的详细定义和简单示例。脚本中的适应性采用“full”模式，即系统会立即向开发人员反馈通过/失败信息。在用户向系统提供新的测试集之前，给定的测试集可以被使用多达32次。

类似地，如果用户希望指定非自适应集成过程，她可以提供以下脚本：
    
    
    ml:
      - script : ./test_model.py
      - condition : d < 0.1 +/- 0.01
      - reliability: 0.9999
      - mode : fp-free
      - adaptivity : none -> xx@abc.com
      - steps : 32

该脚本会接受每个提交，但在每次提交后，会将测试结果发送到电子邮件地址xx@abc.com。假设开发人员没有此电子邮件帐户的[访问权限](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90&zhida_source=entity)，会造成无法优化其下一个模型。

**讨论和扩展**

当前的语法，即ease.ml/ci能够捕获用户在开发过程中发现的许多有用的用例，包括推理新模型和旧模型之间的精度差异，以及推理测试数据集中新模型和旧模型之间预测变化量。原则上，ease.ml/ci可以支持更丰富的语法，如下列出了当前语法的一些限制，未来可继续研究。

  1. **准确性之外：** 当前系统不支持机器学习的其它重要质量指标，例如F1-score和AUC score等。通过用McDiarmid 's不等式替换Bennett 's不等式，以及F1-score和AUC score的敏感性，可以扩展当前系统以适应这些分数。在这种新的背景下，通过更多的优化，如使用分层样本。
  2. **比率统计：** ease.ml/ci的当前语法故意省略了除法（“/”），这对于将来的版本能够进行质量的相对比较（例如，准确度、F1-score等）非常有用。
  3. **顺序统计：** 一些用户认为顺序统计也很有用，例如，确保新模型在历史模型中位于前5位。



另外，当前系统还受限于无法检测[概念漂移](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%A6%82%E5%BF%B5%E6%BC%82%E7%A7%BB&zhida_source=entity)（domain drift or concept ship)。理论上，这一过程类似于CI，区别于固定测试集、测试多个模型，检测概念漂移仅测试单个模型，并监测其在多个测试集上的泛化。

当前版本的ease.ml/ci不支持上述功能。然而，其中许多功能可以通过开发类似的统计技术来支持。

### 系统程序

在传统的持续集成中，系统通常假定用户拥有自己构建测试套件的知识和能力。实际上，通过观察发现，即使大型科技公司经验丰富的软件工程师也可能不知道如何针对给定的可靠性需求开发适当的测试集，尤其对于ease.ml/ci来说，要求会更高。ease.ml/ci一个突出贡献其是一个技术集合，为用户管理测试集提供了实用但严格的指导：测试集需要多大？系统何时需要生成新的测试集？系统什么时候可以发布测试集并将其“降级”为开发集？虽然其中大多数问题都可以由专家根据经验和直觉回答，但ease.ml/ci的目标是提供系统、原则性的指导。为了实现这一目标，ease.ml/ci提供了两个在Travis CI等系统中没有提供的实用程序。

**[样本容量](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%A0%B7%E6%9C%AC%E5%AE%B9%E9%87%8F&zhida_source=entity) 估计器：**该程序以ease.ml/ci脚本作为输入，输出用户需要在测试集中提供的示例数量。

**新测试集告警：** 该程序以ease.ml/ci脚本和机器学习模型的提交历史记录作为输入，并在当前测试集被多次使用而无法用于测试下一个提交的模型时，向用户生成告警（例如，通过发送电子邮件）。用户收到告警后，需要向系统提供新的测试集，也可以将旧的测试集发布给开发人员。

这两个应用程序有一个不太符合实际生产，即系统在每次提交后都会提醒用户请求新的测试集，并使用Hoeffding边界预估测试集大小，但这也会导致需要大量的标记工作。

什么是“实用”？实用性当然取决于用户。尽管如此，从与不同用户的合作经验观察到，为每32个模型评估提供30000 ~ 60000个标记对许多用户来说似乎是合理的。30000 ~ 60000是2到4名工程师在一天（8小时）内以2秒/每个标记的速度进行标记的工作量，32个模型评估意味着（平均）一个月内每天提交一次。在此假设下，用户每个月只需要花一天时间用合理数量的标记器提供测试标记。如果用户无法提供此数量的标记，则在大多数常见条件下，通过将容错率提高一个或两个百分点，可以实现“廉价模式”，即每天标记数量轻松减少10倍。 

因此，为了使ease.ml/ci成为实用的工具，这些实用程序需要以更实用的方式实现。ease.ml/ci的技术贡献后续将会介绍，它可以将系统从用户请求的样本数减少两个数量级。

## 基本实现

### 单模型样本量估计器

**单个变量的估计器**

构建ease.ml/ci时，其中一个模块样本数估计器，需要对变量( n, o, and d )估计出满足 ϵ  精度的估计值，其概率为 1-δ （有效测试概率）。构造该估计器时使用标准Hoeffding边界。

样本量估计器 n:V\times[0,1]^{3} \rightarrow \aleph 是一个函数，它将变量、其[动态范围](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E5%8A%A8%E6%80%81%E8%8C%83%E5%9B%B4&zhida_source=entity)、容错性和成功率作为输入，输出测试集中所需的样本数。

n(\upsilon,r_{\upsilon},\epsilon,\delta)=\frac{-r_{\upsilon}^{2}ln\delta}{2\epsilon^{2}}

其中 r_{v} 是变量 \upsilon 的动态范围， ϵ 为误差容限， 1-δ 是成功概率。

在此使用的用于定义测试条件的语法，其定义见附录A.1。

**单个子句的估计器**

给定子句 C (例如 n-o > 0.01 )，其左侧[表达式](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%A1%A8%E8%BE%BE%E5%BC%8F&zhida_source=entity)为 Φ ，[比较运算符](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E6%AF%94%E8%BE%83%E8%BF%90%E7%AE%97%E7%AC%A6&zhida_source=entity) cmp ( > 或 < )和右侧常数，样本容量估计器返回提供左侧表达式 (ϵ,δ) —估计所需的样本数量。这可以用一个简单的递归来完成。

  1. n(EXP=c\ast v,\epsilon,\delta)=n(\upsilon,r_{\upsilon},\epsilon/c,\delta) ，其中c是一个常数。 n(c\ast v,\epsilon,\delta)=\frac{-c^{2}r_{\upsilon}^{2}ln\delta}{2\epsilon^{2}}
  2. n(EXP1+EXP2,\epsilon,\delta)=max { n(EXP1,\epsilon_{1},\frac{\delta}{2})，n(EXP2,\epsilon_{2},\frac{\delta}{2}) }， \epsilon_{1}+\epsilon_{2}<\epsilon 。同样的等式也适用于 n(EXP1-EXP2,\epsilon,\delta) 。



**单个公式的估计器**

给定一个公式 F ，它是k个子句 C_{1}，…，C_{k} 上的集合，样本估计器需要保证它能够满足每个子句 C_{i} 。构建这样一个估计器的一种方法是：

3\. n(F=C_{1}\wedge…\wedge C_{k},\epsilon,\delta)=max_{i}n(C_{i},\epsilon,\frac{\delta}{k})

示例给定公式 F ，我们现在有一个简单的样本大小估计算法。对于

F :- n - 1.1 * o > 0.01 +/- 0.01 /\ d < 0.1 +/- 0.01

该系统解决了优化问题：

### 非自适应场景

在非自适应场景中，系统评估H模型后，不会将结果发布给开发人员，结果可以发布给用户(集成团队)。

**样本量估计**

该场景下，由于所有的H模型都是独立的，因此估计样本量很容易。对于概率 1-δ ，ease.ml/ci返回每个H模型的正确值，公式 F 所需的样本数为 n(F,\epsilon,\frac{\delta}{H}) 。这是标准的并集结果。给定用户希望评估的模型数量(在ease.ml/ci脚本中的steps字段中指定)，系统就可以返回测试集中的样本数量。

**新测试集告警**

在非自适应场景中，用户提供新测试集的告警很容易实现。系统维护一个计数器，以显示测试集已被使用的次数。当该计数器达到预定义值（即steps）时，系统向用户请求新的测试集。同时，旧的测试集可以发布给开发人员。

### 完全自适应场景

在完全自适应场景下，系统将测试结果（一个bit表示通过/失败）发布给开发人员。由于会将bit信息从测试集泄露给开发人员，因此不能像非自适应场景中那样使用联合绑定。

对于这种情况，有一个简单的策略—对于每个模型，使用不同的测试集。在这种情况下，所需的样本数为 H\cdot n(F,\epsilon,\frac{\delta}{H}) 。

**样本量估计**

对于自适应场景，ease.ml/ci使用以下方法来估计H-step过程的样本量。假设开发人员是确定性的或伪随机的，那么他对下一个模型的决策只依赖于所有先前的通过/失败信息和初始模型 H_{0} 。对于 H steps，过去的通过/失败信息只有 2^{H} 种可能配置。因此，只需要在所有2^{H}可能性上强制联合边界。因此，需要的样本数是n(F,\epsilon,\frac{\delta}{2^{H}})。

**指数项是否太不切实际？**

改进后的样本量n(F,\epsilon,\frac{\delta}{2^{H}})远小于琐碎策略所要求的样本量H\cdot n(F,\epsilon,\frac{\delta}{H})。读者可能会担心自适应场景对 H 的依赖。然而，对于不是太大的 H ，例如 H = 32 ，由于 \frac{\delta}{2^{H}} 在对数范围内，仍可得出实际的样本数。例如，考虑以下简单条件：

F :- n > 0.8 +/- 0.05

H = 32 时，

n(F,\epsilon,\frac{\delta}{2^{H}})=\frac{ln2^{H}-ln\delta}{2\epsilon^{2}}

取 δ= 0.0001,ϵ =0.05 ，则n(F,\epsilon,\frac{\delta}{2^{H}})= 6,279。假设开发人员每天检入最佳模型，则用户每个月只需要提供不到7000个测试样本，该要求还在合理范围内。然而，如果 ϵ =0.01 ，这将会暴增至156,955，这种情况下，就不再实用。

**新测试集告警**

与非自适应场景类似，请求新测试集的告警很容易实现，当系统达到预定义值（即steps）时，系统会请求新测试集。同时，旧的测试集可以发布给开发人员。

### 混合场景

通过约束发布给开发人员的信息，可以获得所需样本数量的更好边界。考虑以下场景：

  1. 如果提交失败，则向开发人员返回失败；
  2. 如果提交通过，则（1）返回通过给开发人员；（2）触发新的测试集告警，向用户请求新的测试集。



与完全自适应场景相比，在该场景中，在开发人员提交通过测试的模型之后，用户立即提供一个新的测试集。

**样本量估计**

假设H为系统支持的最大steps。因为在模型通过测试后，系统将立即请求一个新的测试集，所以它并不是真正的自适应：只要开发人员继续使用相同的测试集，就可以假设最后一个模型总是失败。假设用户是一个确定性函数，它根据过去的历史记录和过去的反馈(一个Fail流)返回一个新模型，则只有H个可能状态需要使用union bound。这提供了与非自适应场景相同的界限：n(F,\epsilon,\frac{\delta}{H})。

**新测试集告警**

与前两个场景不同的是，每当用户提供的模型通过测试或达到预定义值 H 时，系统就会向用户发出告警。

**讨论**

混合场景(将[信息泄露](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E4%BF%A1%E6%81%AF%E6%B3%84%E9%9C%B2&zhida_source=entity)给开发人员)具有与非自适应情况相同的样本量估计，可能与预期相反。考虑到测试集支持的最大值 H ，由于可能在循环测试中使用新的测试集，混合场景不能总是完成所有 H 次数。换句话说，与自适应场景相反，混合场景不是通过增加更多的样本，而是通过减少测试集可以支持的steps来考虑信息泄漏。

当测试难以通过或失败时，混合场景非常有用。例如，想象以下情况：

F :- n - o > 0.1 +/- 0.01

也就是说，系统只接受将精确度提高10个精确度点的提交。在这种情况下，开发人员可能需要进行多次开发迭代来获得一个满足条件的模型。

### 条件评估

通过一个满足样本量估计器给出的样本数的测试集，可以得到子句中使用的三个变量的估计，即 \hat{n},\hat{o},\hat{d}。简单地使用这些估计值来评估条件可能会导致误报。在ease.ml/ci中，使用它们对应的置信区间代替[点估计](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E7%82%B9%E4%BC%B0%E8%AE%A1&zhida_source=entity)，并在区间上定义一个简单的代数（例如， [a, b]+[c, d] = [a+c, b+d] ），该代数用于计算单个子句的左侧。子句的计算结果仍为{ True, False, Unknown }。然后，如果用户选择 fp-free 或 fn-free ，系统会将此三值逻辑映射为二值逻辑。

### 用例与实用性分析

ease.ml/ci实现简单，支持许多实用的场景。总结如下五个用例，并分析了用户所需的样本数。

**（F1：下限最差情况质量）**
    
    
    F1          :- n > [c] +/- [epsilon]
    adaptivity  :- none
    mode        :- fn-free

此条件用于质量控制，以避免开发人员提交的低质量或者明显错误的模型。在非自适应场景中可以看到许多这种情况的用例，其中大多数需要无false-negative。

**（F2：增量式[质量改进](https://zhida.zhihu.com/search?content_id=217427847&content_type=Article&match_order=1&q=%E8%B4%A8%E9%87%8F%E6%94%B9%E8%BF%9B&zhida_source=entity)）**
    
    
    F2          :- n - o > [c] +/- [epsilon]
    adaptivity  :- full
    mode        :- fp-free
    ([c] is small)

此条件用于确保机器学习应用程序随着时间的变化持续的改进。当机器学习应用程序面向最终用户时，质量下降是不可接受的。在这种情况下，整个过程完全自适应和无误报将很重要。

图2不同条件所需的样品数， H=32 红色字体表示“不切实际”的样本数量

**（F3：质量里程碑）**
    
    
    F3          :- n - o > [c] +/- [epsilon]
    adaptivity  :- firstChange
    mode        :- fp-free
    ([c] is large)

此条件用于确保存储库仅包含重要的质量里程碑（例如，发生10个精度点跳跃后的日志模型）。虽然条件在语法上与F2相同，但整个过程是混合自适应的，无false-positive。

**（F4：无重大变更）**
    
    
    F4          :- d < [c] +/- [epsilon]
    Adaptivity  :- full | none
    Mode        :- fn-free
    ([c] is large)

此条件用于类似于F1的安全问题。当机器学习应用程序面向最终用户或作为较大应用程序的一部分时，需要确保在后续版本不会发生显著变化。该过程需要是无false-negative。

**（F5：构成条件）**

F5 :- F4  /\ F2

最流行的测试条件之一是F4和F2两个条件的组合：集成团队希望将F4和F2一起使用，这样面向最终用户的应用程序就不会经历巨大的质量变化。

**实用性分析**

我们的实现支持的哪些条件时实用的？哪些是不切实际的？

**何时实用？**

在大多数情况下，支持的条件大多是实用的。图2中显示了H = 32时所需的样本数。我们发现，对于F1和F4，所有自适应策略在2.5个精度点内是实用的；而对于F2和F3，非自适应和混合自适应策略在2.5个精度点内是实用的，完全自适应策略只有在5个精度点内才是实用的。从这个例子中我们可以看到，即使是一个简单的实现，对严格保证机器学习的CI执行代价也并不总是昂贵的。

**何时是不切实际的？**

从图2中可以看到对ϵ的强依赖性。由于Hoeffding不等式中的 O(1/\epsilon^{2}) 项，该结果符合预期。因此，在1个精度点之上，没有一种自适应策略是实用的，这对于机器学习的许多关键任务应用程序来说是很重要的。完全自适应策略比非自适应策略需要更多的样本，因此会在更高的容错性下变得不切实际。

## 参考资料

  1. Cedric Renggli, Bojan Karlaš, Bolin Ding, Feng Liu,[Wentao Wu](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/people/wentwu/), Ce Zhang, [Continuous Integration of Machine Learning Models with ease.ml/ci: Towards a Rigorous Yet Practical Treatment](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/publication/continuous-integration-of-machine-learning-models-with-ease-ml-ci-towards-a-rigorous-yet-practical-treatment/) SysML Conference (SysML 2019)



  


> 上一篇：[【AI工程论文解读】03-DevOps for AI-人工智能应用开发面临的挑战](https://zhuanlan.zhihu.com/p/565078936)  
> 下一篇：[【AI工程论文解读】05-通过Ease.ML/CI实现机器学习模型的持续集成（下）](https://zhuanlan.zhihu.com/p/581384885)
