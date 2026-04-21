# MindSpore，易用性提升的思考与实践

原文链接：https://zhuanlan.zhihu.com/p/526449703

---

​

目录

> 随着科技不断创新发展，开源技术的重要价值日渐凸显，拥抱开源已成为行业趋势。当前，不仅在 IT 和互联网企业，银行电信等不少企业级客户也在不少技术领域开始引入开源技术，开源已经成为推动企业核心技术创新和[数字化转型](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%95%B0%E5%AD%97%E5%8C%96%E8%BD%AC%E5%9E%8B&zhida_source=entity)的新动力。  
> AI 框架作为发展人工智能所必需的基础设施之一，承担着 AI 技术生态中[操作系统](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&zhida_source=entity)的角色，是 AI学术创新与产业商业化的重要载体。**AI框架开源** ，能够依靠开源社区，聚合人才和智慧，助推AI框架的快速升级，对[AI框架](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=3&q=AI%E6%A1%86%E6%9E%B6&zhida_source=entity)的发展至关重要。随着AI框架的快速发展，也将极大的助力人工智能由理论走入实践， 快速进入了场景化应用得时代。  
> 而一款成功的开源软件，往往在构筑代码/核心能力时，也需要考虑软件的易用性，从而获得开发者的青睐。

## 概述

在使用开源软件的过程中，大家可能或多或少都遇到过一些问题，比如安装过程复杂、找不到案例、缺少示例、问题定位难等，我把这些问题称之为开源软件的“摩擦力”。

一个成功的开源软件，想要赢得开发者的青睐，代码/核心能力的开放实际上只是冰山一角，如何构建开发者体验，持续提升软件易用性，从而消除开源软件的“摩擦力”，也是我们需要思考的工作。

[MindSpore](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/)是[华为公司](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E5%8D%8E%E4%B8%BA%E5%85%AC%E5%8F%B8&zhida_source=entity)2020年3月**开源的全场景AI计算框架** ，源于全产业的[最佳实践](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5&zhida_source=entity)，旨在实现易开发、高效执行、全场景覆盖三大目标。其具有自动并行、动静态图结合、全场景部署协同、全栈协同加速等特点。

2020年3月正式[开源](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=13&q=%E5%BC%80%E6%BA%90&zhida_source=entity)至今，从最初的0.1版本快速迭代到1.7版本，获得国内众多高校、科研机构、企业和开发者的支持。MindSpore的下载量突破145万+；内外部核心贡献者超过1300个；在100+高校开课，覆盖学生5万+。 

对于MindSpore，我们如何进一步提升其易用性呢？

## 什么是易用性

说起易用性，咱先聊聊什么是开发者体验。开发者体验（DX，**D** eveloper e**X** perience）来源于用户体验（UX，**U** ser e**X** perience）。**用户体验** 指**人们对于使用的产品、系统或者服务的认知印象和回应用户。** 通俗来讲就是**“这个东西好不好用，用起来方不方便”** 。而**开发者体验** ，**与用户体验类似，面向的对象** 主要是**开发者** ，关注的内容变为库、API、文档、相关工具等。如下先简单介绍用户体验和开发者体验的发展历程。

  * 1995年，[美国](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%BE%8E%E5%9B%BD&zhida_source=entity)认知心理学家Donald Norman提出“**用户体验** ”概念，他将[用户体验设计](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%94%A8%E6%88%B7%E4%BD%93%E9%AA%8C%E8%AE%BE%E8%AE%A1&zhida_source=entity)定义为三层，**本能层** 、**行为层** 和**[反思层](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E5%8F%8D%E6%80%9D%E5%B1%82&zhida_source=entity)** 。首先是如何通过第一印象去吸引用户；然后在用户使用产品时，又如何通过产品的[交互体验](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E4%BA%A4%E4%BA%92%E4%BD%93%E9%AA%8C&zhida_source=entity)去吸引用户；最后上升到产品的意义，以及使用者对产品产生的情感升华。
  * 2007年，挪威[用户体验设计师](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%94%A8%E6%88%B7%E4%BD%93%E9%AA%8C%E8%AE%BE%E8%AE%A1%E5%B8%88&zhida_source=entity)Magnus Revang提出“**用户体验轮** ” 研究模型。在该模型中，通过围绕核心价值周边的因素，比如可**获取性、可靠性、易得性** 和**易用性** 等，共同去实现中间的核心-**价值** 。
  * 2010年，Google提出了**HEART模型** ，主要用来衡量Web应用的用户体验，主要衡量因素包含**愉悦感、参与度、接受度** 和**[留存率](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%95%99%E5%AD%98%E7%8E%87&zhida_source=entity)** 等。
  * 2017年，ThoughtWorks 提出开发者体验的一些关键因素，比如**文档、错误呈现、易用、交互** 和**触点** 等。



那么，开发者体验和易用性又有什么关系呢？开发者体验的覆盖范围要比易用性更大。从[开源软件](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=6&q=%E5%BC%80%E6%BA%90%E8%BD%AF%E4%BB%B6&zhida_source=entity)使用角度看，开发者体验包含三个层次：能用/可用->**易用/好用** ->享用/爱用。

第一阶段，能用/好用。关注点是软件功能本身的完备可用、质量稳定可靠。在该阶段，开发者能够借助该软件完成特定目标，但是可能会存在一些瑕疵或者阻塞点。

**第二阶段** ，易用/好用。关注点是**易学习低门槛、易使用高效率** 。在该阶段，用户能够借助软件**高效完成** 特定目标，它体现的是**易用性和好用性。**

第三阶段，享用/爱用。关注点是软件具备的独到之处、愉悦感。在该阶段，软件已经比较成熟，而且用户经过前期的使用之后，能够感受到软件的一些独到之处，用户开始满意、喜欢并且爱用。

当前主要关注**第二阶段** ，即**易用/好用** ，同时也会涉及第三阶段。

易用，通俗的讲就是好用、容易上手。那么，在一款[软件产品](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E4%BA%A7%E5%93%81&zhida_source=entity)中，我们怎么定义其易用性？在《GB/T 29836-2013 系统与软件易用性》中，将其定义为：**产品在特定使用环境下为了特定目标可以为特定用户使用的程度** 。对于其细化的[指标体系](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%8C%87%E6%A0%87%E4%BD%93%E7%B3%BB&zhida_source=entity)，又细分为**易理解性、易学性、易操作性、和吸引性** 。即易用性是指产品被用户使用时，能够被用户理解、学习、使用和吸引用户的能力。易用性是产品的基本自然属性，标志着最终产品的可用性和成熟度。

那么，针对MindSpore，我们该如何提升它的易用性？

## 提升易用性，我们准备怎么做

对于MindSpore，从用户群体来看，主要分为**学习型用户、科研型用户、生产部署型用户** 。针对这些用户群体，我们从**易学习低门槛、易开发高效率、问题快速闭环** 三方面，**提升用户易用性** 。

  * **易学习低门槛** ：提供丰富的学习资料，帮助用户快速上手和学习，包含文档教程、应用案例、以及AI工程&实践（AI开发过程中的工程实践）。Gartner发布的一篇文章中提到一种创新技术-AI工程，认为随着AI的[普惠化](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%99%AE%E6%83%A0%E5%8C%96&zhida_source=entity)，如何通过如设计模式、MLOps流水线等帮助开发者更好的去使用AI，将会变得越来越重要。[[AI工程](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=3&q=AI%E5%B7%A5%E7%A8%8B&zhida_source=entity)](https://www.zhihu.com/column/c_1488835248573706240)是一系列方法、工具和实践的集合，确保AI模型/软件的高效交付，具备可信、健壮性及可解释性，并持续地为用户创造价值。相信随着AI工程技术的成熟&普及，将会对AI的广泛应用产生巨大的推动。
  * **易开发高效率：** 基于AI软件开发流程，提供各项开发能力，兼顾极简易用与灵活高效，包含环境准备、[数据处理](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)、模型开发、调试调优和[部署推理](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E9%83%A8%E7%BD%B2%E6%8E%A8%E7%90%86&zhida_source=entity)等。整个流程就是AI开发的端到端流程，其更多体现如何帮助开发者提高开发效率。
  * **问题及时闭环：** 专门团队和专有接口人，及时处理线上线下问题，提升MindSpore开发者的满意度。



## 提升易用性，我们最近做了什么

在相继发布的MindSpore 1.6和1.7版本，我们围绕着易学习低门槛、易开发高效率和问题快速闭环等目标，在MindSpore易用性上做了大量工作。

下面我们重点讲几个关键的特性。

### 1.支持数据处理自适应调优，充分发挥数据处理性能

在模型训练时，数据处理的并行度是初始时确定的，过程中无法调整。并行度设置过小（处理能力差）或者过大（资源切换开销高）都会影响整体的数据处理效率。

为解决上述问题，MindSpore在1.7版本提供了一种数据处理自动调优的工具——[Dataset AutoTune](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/tutorials/experts/zh-CN/r1.7/debug/dataset_autotune.html)。在训练过程中，该工具可以帮助用户根据系统环境资源的情况自动调整MindSpore Data数据处理管道的[并行度](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=3&q=%E5%B9%B6%E8%A1%8C%E5%BA%A6&zhida_source=entity)和内存使用度，利用当前系统资源加快数据处理管道的处理速度。

在整个网络训练的过程中，Dataset AutoTune工具会持续检测当前训练性能的瓶颈处于数据处理侧还是模型运算侧。如果检测到瓶颈在数据处理侧，则将进一步对数据处理管道中各个操作（如GeneratorDataset、map、batch等）的参数进行调整，以加快该操作的计算速度。我们以一个例子来说明：

### 2.一站式智能[开发环境](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)MindSpore Dev Toolkit

新用户使用MindSpore时，在熟悉环境配置时，或者业务开发需要键入大量代码，亦或查找MindSpore知识，都会花费较多时间，从而无法聚焦在核心业务。

针对上述场景，MindSpore提供了一站式智能开发环境[MindSpore Dev ToolKit](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/ide-plugin)。其能够使能用户及时体验MindSpore框架，并提供运行管理、智能知识搜索、[智能代码补全](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E6%99%BA%E8%83%BD%E4%BB%A3%E7%A0%81%E8%A1%A5%E5%85%A8&zhida_source=entity)和算子互搜等能力，致力于让所有用户丝滑地摆脱[环境干扰](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%8E%AF%E5%A2%83%E5%B9%B2%E6%89%B0&zhida_source=entity)学习人工智能，让人工智能回归[算法](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%AE%97%E6%B3%95&zhida_source=entity)本身。MindSpore Dev ToolKit具备以下特点：

  * 基于Conda提供的MindSpore环境管理方式，实现一键环境管理，5分钟快速将MindSpore全场景AI框架及依赖安装在独立环境中并部署最佳实践。
  * 基于[语义搜索](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E8%AF%AD%E4%B9%89%E6%90%9C%E7%B4%A2&zhida_source=entity)等能力，在Dev Toolkit内提供全面的MindSpore知识内容检索，在IDE中快速获取详细的文档支持。
  * 基于MindSpore ModelZoo等最佳实践数据集实现智能代码补全功能，让用户在编写相关代码时获取实时提示，补全达80%的高准确性，有效的帮助用户快速获取需要的信息，提升编码效率。



### 3.提供断点续训能力，训练任务中断可原地恢复

在超大模型训练，为保证训练效率，保存间隔较长（如5-6小时/次）。一旦系统出现异常时，最坏情况丢失前N小时的训练结果，再次重头开始训练时，会造成较大的时间损失。

为解决上述问题，MindSpore提供了断点续训能力，自动保存异常点模型状态，当任务中断后，从中断的检查点处恢复并继续训练。

### 4.MindSpore Vision—易用易理解的主流工具库

在手动[处理数据](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE&zhida_source=entity)、构建网络、调试，往往会耗费大量时间。这时，需要套件提供开箱即用能力，来提升开发效率。

[MindSpore Vision](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/vision/docs/zh-CN/r0.1/index.html)是基于MindSpore的开源计算机视觉[研究工具](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E7%A0%94%E7%A9%B6%E5%B7%A5%E5%85%B7&zhida_source=entity)库，支持主流计算机视觉网络，重点提供[图像分类](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E5%9B%BE%E5%83%8F%E5%88%86%E7%B1%BB&zhida_source=entity)能力；支持50+预训练模型（ViT，EfficientNet等），常用数据集接口（[cifar10](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=cifar10&zhida_source=entity)）。可以实现开箱即用、大幅提升开发效率。我们以一个例子来说明：

从如下示例，可以看出开发效率显著提升，如官网（Lenet样例）代码整体减少90%

  * 使用MindSpore Vision前：


    
    
    # 总计200+
    //构建网络
    def construct(self, x):
            x = self.conv1(x)
            x = self.relu(x)
            x = self.max_pool2d(x)
            x = self.conv2(x)
            x = self.relu(x)
            x = self.max_pool2d(x)
            x = self.flatten(x)
            x = self.fc1(x)
            x = self.relu(x)
            x = self.fc2(x)
            x = self.relu(x)
            x = self.fc3(x)
    //数据处理
    …

  * 使用MindSpore Vision后：


    
    
    # 总计20行
    …
    #网络定义（1行）
    network = lenet(num_classes=10, pretrained=False)
    …

## 为什么成立易用性SIG

未来，MindSpore团队将会不断推出易用性新特性、持续做好开发者体验。同时，MindSpore团队**更希望广大开发者们能够加入进来** ，一起打造易学易用、灵活高效的AI框架。为此，我们成立了[易用性SIG](https://zhuanlan.zhihu.com/p/490275689)。

易用性SIG作为连接开发者的桥梁，其目标是**和开发者共同打造易学易用、灵活高效的AI框架，持续提升MindSpore易用性，助力开发者成功** 。

**易用性SIG是一个倾听开发者声音的渠道** ，在这里，你可以直接提出对MindSpore易用性的改进需求，需求将及时反馈给MindSpore社区进行评估。

**易用性SIG也是一个为开发者提供帮助的渠道** ，在你学习和使用MindSpore的过程中，对于文档信息体验、安装、API使用、语法/算子/模型支持、报错信息等方面有任何疑问，都可以在这里发问，SIG会通过社区互助的形式帮你答疑解惑。

**易用性SIG更是一个开发者共同交流和学习的平台** ，这里有丰富的技术活动，不仅有易用性相关特性讲解和演示，还有业界专家对AI工程方法和最佳实践进行分享，更有学术界大牛进行[前沿技术](https://zhida.zhihu.com/search?content_id=205121550&content_type=Article&match_order=1&q=%E5%89%8D%E6%B2%BF%E6%8A%80%E6%9C%AF&zhida_source=entity)分享，以及开发者现身说法，分享他们的故事与经验。我们还可以一起来开发知识问答机器人，让AI普惠每一位开发者，让开发者学好、用好MindSpore是我们最大的心愿！

更多详细的易用性SIG介绍请参见[易用性SIG介绍](https://zhuanlan.zhihu.com/p/490275689)。同时，欢迎各位开发者踊跃参与，不断壮大这个技术圈子。添加小助手的微信（vx: msusig），小助手拉你进群，大家一起交流更多易用性相关话题！

## 参考资料

[1][《GB/T 29836-2013 系统与软件易用性》](https://link.zhihu.com/?target=http%3A//std.samr.gov.cn/gb/search/gbDetailed%3Fid%3D71F772D8072BD3A7E05397BE0A0AB82A)

[2][中国信通院《AI 框架发展白皮书（2022年）》](https://link.zhihu.com/?target=http%3A//www.caict.ac.cn/kxyj/qwfb/bps/202202/t20220225_397170.htm)

[3] MindSpore官网：[https://www.mindspore.cn/](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/)

[4]易用性SIG介绍：[https://zhuanlan.zhihu.com/p/490275689](https://zhuanlan.zhihu.com/p/490275689)

[5]MindSpore Dev ToolKit：[https://gitee.com/mindspore/ide-plugin](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/ide-plugin)
