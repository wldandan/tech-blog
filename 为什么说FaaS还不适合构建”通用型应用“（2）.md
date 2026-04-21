# 为什么说FaaS还不适合构建”通用型应用“（2）

原文链接：https://zhuanlan.zhihu.com/p/137697322

---

在[上一篇文章](https://zhuanlan.zhihu.com/p/108609862)中，我们提到了现有的FaaS主要适合**高吞吐、快速伸缩、高可用** 的应用场景，但对于**低延时、强一致、长时间运行(long-running)等** 的应用（如实时分析与预测、机器学习、[事务处理](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86&zhida_source=entity)等），却并不太友好；也提到了无状态的FaaS目前在Serverless中占据了重要的地位，但由于缺乏有状态的处理，距离Serverless的宏伟愿景还远不够。

本文将进一步剖析， **“为什么FaaS需要有状态？”。**

**先抛出观点：现有的FaaS能力不足，无法满足应用的多样性。**

* * *

## **一、为什么说现有FaaS能力不足？**

Serverless是未来云计算的范式，它不仅是基础设施的自动化与效率提升，也代表着下一代PaaS的方向，帮助开发者快速、低成本的构建和运行应用。

> 在论文《无服务器计算：经济和架构影响》中描述的Serverless广阔前景如下：“Serverless”是指新一代的PaaS，其接收请求、处理、响应，并能够自动执行[容量规划](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%AE%B9%E9%87%8F%E8%A7%84%E5%88%92&zhida_source=entity)、调度和运维监控。基于该平台，开发人员只需要关注如何有效实现业务逻辑。

现有的Serverless计算主要以FaaS为代表，其**优势** 包括如下两点：

  * **零运维成本**  
开发人员/运维人员不再需要承担**分布式系统的运维成本，** 如[负载均衡](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1&zhida_source=entity)/单点故障/自动扩缩容/补丁升级/异地容灾等，并能提供快速的[弹性伸缩](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%BC%B9%E6%80%A7%E4%BC%B8%E7%BC%A9&zhida_source=entity)能力。  

  * **零资源浪费**  
按照**代码运行所需的资源** ，**而非租用的资源** 进行计费，即无请求等于无成本。



> 以计算资源为例，对于传统[公有云](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%85%AC%E6%9C%89%E4%BA%91&zhida_source=entity)基础设施资源而言，计费方式是基于资源的租用时长(如EC2启动后，即便不运行任何业务，也会以分钟为单位进行计费）。而基于Serverless，基础设施是按照运行(代码)时消耗的资源进行收费，如Lambda是按照请求次数及（占用资源的）GB-S为单位进行计费。无请求处理， 则无需支付任何费用。

目前的FaaS，其典型特征为**无状态、短生命周期、不可直接寻址，** 在它们提供如上核心价值的同时，也带来了一些新的挑战：

## **1\. 依赖[外部存储](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%A4%96%E9%83%A8%E5%AD%98%E5%82%A8&zhida_source=entity)**

    * [短生命周期](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=2&q=%E7%9F%AD%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F&zhida_source=entity)  
Function的生命周期有限（如AWS Lambda的生命周期为15分钟），如果业务**逻辑复杂、处理耗时较长** ，则需要将(生命周期内未处理完的)**上下文数据暂存到外部存储或从外部存储恢复** 。  

    * 无状态  
Function没有状态，则意味着业务逻辑**只能将状态存放在外部存储机制中** 。现实世界中的应用，存在大量对状态的处理，而这也是[微服务设计](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%BE%AE%E6%9C%8D%E5%8A%A1%E8%AE%BE%E8%AE%A1&zhida_source=entity)与落地中一直面临的挑战。  

    * 不可直接寻址  
Function间无法通过[点对点](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E7%82%B9%E5%AF%B9%E7%82%B9&zhida_source=entity)方式直接通信，需要借助于其他机制(如发布/订阅)通信，同时通信过程中的**[数据交换](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E4%BA%A4%E6%8D%A2&zhida_source=entity) 需要依赖于外部存储**。



一方面，FaaS对外部存储存在依赖；而另一方面，现有的存储服务却不能很好的满足FaaS，

> OBS(如AWS S3)：延迟高、[数据模型](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B&zhida_source=entity)简单(Low Cost、High Latency、Simple Data Model)  
> Aurora Serveless：成本高、仅支持关系型数据模型(Expensive、Relation Model)  
> DynamoDB：不支持分片的自动伸缩(No autoscale of shards)  
> Traditional RDS：不支持按需付费的模式(No serverless Pricing Model)

A Berkeley View on Serverless Computing

因此，**必然导致如下成本的增加：**

  * **调用次数** 。如上下文恢复、状态变化等需再次触发Function。
  * **资源使用** 。额外使用云服务的存储，增加了使用成本；而自建存储资源，则增加了运维成本。



## 2\. 一致性问题

Function自身无状态，依赖于外部存储，于是**当多个Function实例并发处理同一份数据时，必然会带来数据一致性的问题** 。更多数据一致性的内容，可参考相关文章。

[](https://zhuanlan.zhihu.com/p/41889527)

## 3\. I/O能力受限

从云供应商角度看，为了提高资源利用率，将多个Function实例运行在同一个VM/Host上，因此**Function实例之间实际上是[共享带宽](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%85%B1%E4%BA%AB%E5%B8%A6%E5%AE%BD&zhida_source=entity)**。在《Berkeley View On Serverless》的论文明确提出，**AWS Lambda的平均I/O带宽为60M/S** ，因此对于[大数据](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%A4%A7%E6%95%B0%E6%8D%AE&zhida_source=entity)量存储的应用，如机器学习中的[模型训练](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E8%AE%AD%E7%BB%83&zhida_source=entity)，必然会因为此而受到影响，**如延迟增加，计算资源使用率低。**

## 4\. 连接池管理复杂

在微服务时代，当多个服务访问同一个数据库（Service Per Table），就存在连接不够用的问题。在Serverless时代，由于Function的短生命周期，快速启动、快速释放，因此频繁的获取/释放连接，**导致对[连接池](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=2&q=%E8%BF%9E%E6%8E%A5%E6%B1%A0&zhida_source=entity)管理的问题更加突出。**

## 5\. 需更多的Trade-off

由于Function的典型特征，对于低[延时](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=2&q=%E5%BB%B6%E6%97%B6&zhida_source=entity)、强一致、长时间运行等的应用，需要更多的[Trade-off](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=2&q=Trade-off&zhida_source=entity)。在论文《[Serverless Cloud Computing (Function-as-a-Service) Patterns](https://zhuanlan.zhihu.com/p/129204380)》，作者就整理了FaaS的常用32种设计模式，从设计上应对现有FaaS的不足。

[](https://zhuanlan.zhihu.com/p/129204380)

  


譬如，对于一个长时运行的应用，为了享受到**Serverless的红利（** 零成本/零浪费**）** ，需要采用**Chain或Fan-In/Fan-Out等设计模式** ，将[应用逻辑](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%BA%94%E7%94%A8%E9%80%BB%E8%BE%91&zhida_source=entity)拆分成更多子任务，以适配Function的短生命周期。

## 二、为什么说FaaS无法满足应用的多样性？

首先，我们看看目前FaaS的User Case占比：

《A Berkeley View on Serverless Computing》

可以看出，Top3的场景主要在Web/API、ETL和第三方集成等，而这些场景无疑是过去十年移动互联网和典型IT领域的代表。而未来的科技最大程度丰富人们的生活，将聚焦在：

[](https://link.zhihu.com/?target=https%3A//kknews.cc/tech/ql3qzob.html)

其典型的应用场景包括如下，但不限于:

  * **AI相关的训练：** 在训练过程中，系统会频繁的存/取模型，进行迭代，以获得最优解。因此，需要低延时、高I/O的访问能力，如[参数服务器](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E5%8F%82%E6%95%B0%E6%9C%8D%E5%8A%A1%E5%99%A8&zhida_source=entity)。  
  

  * **实时预测/推荐/预警：**[无人机](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E6%97%A0%E4%BA%BA%E6%9C%BA&zhida_source=entity)、无人驾驶、机器人，需实时预测周边的变化并作出精准决策；海量IOT数据的实时处理；智能家居中的[人脸识别](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E4%BA%BA%E8%84%B8%E8%AF%86%E5%88%AB&zhida_source=entity)，有陌生人访问时实时预警等。  
  

  * **共享协作：** 新冠疫情的爆发，为共享协作带来了新的机会。如协同办公、共享文档、白板等。这些应用虽然在过去也有实现的方式，但有状态的FaaS，无疑能极大降低实现成本。  
  

  * **基于分布式事务的工作[流处理](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=1&q=%E6%B5%81%E5%A4%84%E7%90%86&zhida_source=entity)：**如联程的机票、酒店行程预定，如果其中某一段行程不可预定，则需要对整个行程进行回滚。目前的方案如Saga、TCC等，但实现复杂度较高。



## 总结

无状态的FaaS目前在Serverless中占据了重要的地位，但距离Serverless的宏伟愿景还远不够。相信随着Serverless的快速发展，对**[有状态](https://zhida.zhihu.com/search?content_id=118730918&content_type=Article&match_order=5&q=%E6%9C%89%E7%8A%B6%E6%80%81&zhida_source=entity) 的探索与突破**将会成为重要的课题。

本文从两个维度描述了**有状态的FaaS的重要性：**

**①现有FaaS的能力不足；②无法满足未来应用的多样性。**

  


## 参考

[Cloud Programming Simplified: A Berkeley View on Serverless Computing](https://link.zhihu.com/?target=https%3A//www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.pdf)  
[Leveraging Stateful Functions to Power the Next Generation of Event-Driven Applications](https://link.zhihu.com/?target=https%3A//www.datacouncil.ai/hubfs/Data%2520Council/slides/nyc19/3.%2520seth-wiesman-leveraging-stateful-functions.pdf)  
[https://www.datacouncil.ai/talks/leveraging-stateful-functions-to-power-the-next-generation-of-event-driven-applications](https://link.zhihu.com/?target=https%3A//www.datacouncil.ai/talks/leveraging-stateful-functions-to-power-the-next-generation-of-event-driven-applications)
