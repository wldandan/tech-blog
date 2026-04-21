# 你需要了解的32种Serverless Function Patterns

原文链接：https://zhuanlan.zhihu.com/p/129204380

---

2019年3月，[伯克利](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E4%BC%AF%E5%85%8B%E5%88%A9&zhida_source=entity)的一篇论文《A Berkeley View on Serveless Computing》对Serveless做了全面的分析，并预测它将成为未来[云计算](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E4%BA%91%E8%AE%A1%E7%AE%97&zhida_source=entity)的主宰。

>  _“We predict that serverless computing will grow to dominate the future of cloud computing” – Berkeley CS Dept_

随着各大云厂商对Serverless技术的加码，Serverless技术无疑已经成为学术界和工业界关注的焦点。

本文的内容主要参考论文《Serverless Cloud Computing (Function-as-a-Service) Patterns: A Multivocal Literature Review》，整理了FaaS的常用32种设计模式。

**一、简介**

Severless的出现极大了降低了系统的运维成本和资源浪费，实践者在使用FaaS构建系统的同时，也遇到了一些挑战，如[冷启动](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E5%86%B7%E5%90%AF%E5%8A%A8&zhida_source=entity)、函数编排、数据共享等。本文通过对这些挑战的分析，并基于2017-2019的相关学术论文及社区贡献（博客、白皮书、会议、书籍和实践者的对话），梳理出5大类，32种[设计模式](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=2&q=%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F&zhida_source=entity)，旨在帮助开发者有效解决FaaS使用中存在的一些通用挑战。

在技术的快速演进过程中，[架构模式](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F&zhida_source=entity)能够很好的指导开发者构建易于维护的系统。如[面向对象的设计模式](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F&zhida_source=entity)（GOF，1994），SOA模式（Erl，2008），微服务模式（Taibi et al.，2018）（Chris 2017) 等; 最近2年多，越来越多的从业者开始讨论Severless的模式。

本文讨论了32种Serverless的设计模式，并将其分为五类，如下图所示：

  * **编排和聚合(Orchestration & Aggregation)**
  * **[事件管理](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E4%BA%8B%E4%BB%B6%E7%AE%A1%E7%90%86&zhida_source=entity)(Event Management)**
  * **可用性(Availability)**
  * **通信(Communication)**
  * **授权( Authorization)**



**二、主要模式介绍**

因为文中模式较多，接下来主要介绍常用的模式。

**2.1 Orchestration and Aggregation**  


  * **[聚合器](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E8%81%9A%E5%90%88%E5%99%A8&zhida_source=entity)(Aggregator): **  
Problem：将几个服务的API暴露到一个EndPoint  
Solution：使用Fun与Service独立通信，并将结果聚合返回。



  * **扇入/扇出(Fan In/Fan Out)**  
Problem：Function属于长任务，超过最大[执行时间](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E6%89%A7%E8%A1%8C%E6%97%B6%E9%97%B4&zhida_source=entity)。  
Solution: 将工作拆分，[并行化](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E5%B9%B6%E8%A1%8C%E5%8C%96&zhida_source=entity)执行，再聚合，缩短计算过程。  
Issues：函数间存在强耦合，增加复杂度。



  * **Function-Chain**  
Problem：Function属于长任务，超过最大执行时间。   
Solution: 形成function链。 



本类中其他的模式包括：

  * **基于Queue Based Load Leveling**  
Problem：使用不可伸缩的后端构建可伸缩时应用出现的问题。，  
Solution：使用function+队列，对请求流量的控制。



  * **基于Function的路由**



  * **基于Function的[状态机](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E7%8A%B6%E6%80%81%E6%9C%BA&zhida_source=entity)，类似于AWS的StepFun，Azure的Durable Fun等**



  * **Data Lake作为[数据源](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%BA%90&zhida_source=entity)，使用Function进行数据的实时加工和处理。**



  


**2.2 Event-Management**

  * **基于Function的[读写分离](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB&zhida_source=entity)**



**2.3 Availability Patterns**

除了现在常说的Bulkhead和Circuit breaker，还包括数据最终一致性的处理。

Circuit breaker Pattern

数据最终一致性处理

  


**2.4 Communication Patterns**

Data Stream

  


状态外置

**2.5 Authorization Patterns**

**主要包括** The Gatekeeper 和 Valet key两种方式。

前者是每次请求都需要过Function，后者是类似JWT，持有token，一段时间内不需要再验证。

  


The Gatekeeper

Valet key 

**三、趋势和开放性讨论**

随着开发和部署Serverless的公司数量不断增加，越来越多的人加入到这个方向的讨论和研究，那什么是[最佳实践](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5&zhida_source=entity)或设计模式。到目前为止，大多数公司都在用已知的方法及来自成熟技术（如[微服务](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=2&q=%E5%BE%AE%E6%9C%8D%E5%8A%A1&zhida_source=entity)、[web服务](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=web%E6%9C%8D%E5%8A%A1&zhida_source=entity)）的模式探索Serverless领域。而实际上，在Serverless的快速发展中，存在的挑战是多样性的，包括：

  * **如何组合微服务和Function**  
微服务可以由一个或多个[无服务器函数](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E6%97%A0%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%87%BD%E6%95%B0&zhida_source=entity)组成。然而，如何将Function组合成一个完整的应用程序或一个微服务还不清楚。  

  * **缺乏稳定的生态工具**  
支持无[服务器开发](https://zhida.zhihu.com/search?content_id=116842938&content_type=Article&match_order=1&q=%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%BC%80%E5%8F%91&zhida_source=entity)的工具仍然是有限的，它的成熟决定了Serverless的生态以及长期发展。  

  * **Function的重用**  
一旦一个不断发展的系统拥有成千上万的Funcition，对于如此复杂的系统，如何建立良好的可理解性



上述各点都需要更多的经验报告和实证调查。

* * *

原文请参考

[](https://link.zhihu.com/?target=https%3A//www.researchgate.net/publication/340121613_Patterns_for_Serverless_Functions_Function-as-a-Service_A_Multivocal_Literature_Review%3FenrichId%3Drgreq-b0507e4329c6d58098c8e916385a1a55-XXX%26enrichSource%3DY292ZXJQYWdlOzM0MDEyMTYxMztBUzo4NzI1ODYxMzczMjU1NjhAMTU4NTA1MjE1NTE5Ng%253D%253D%26el%3D1_x_3%26_esc%3DpublicationCoverPdf)
