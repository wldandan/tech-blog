# 一文彻底理清Serverless的价值、特征和挑战

原文链接：https://zhuanlan.zhihu.com/p/107063667

---

## 摘要：

Serverless是使能开发者**聚焦业务实现** 、**无需关注运维与基础设施** 、**并能按需使用和付费的技术。** 其核心价值为“零运维成本” + “零资源浪费”；在带来先进开发理念与角色职责转变的同时，也面临着诸多的挑战......  


****

* * *

2019年3月，[伯克利](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E4%BC%AF%E5%85%8B%E5%88%A9&zhida_source=entity)的一篇论文《A Berkeley View on Serveless Computing》对Serveless做了全面的分析，并预测**它将成为未来云计算的主宰** 。

>  _“We predict that serverless computing will grow to dominate the future of cloud computing” – Berkeley CS Dept_

2019年12月，AWS REInvent上发布的数个Serverless更新，反响热烈。同时，在[云原生](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E4%BA%91%E5%8E%9F%E7%94%9F&zhida_source=entity)领域颇具影响力的 KubeCon&CloudNativeCon 会议中，关于 Serverless 的话题，2018 年有 20 个， 2019 年增长至 35 个，业界对Serverless的重视与期待可见一斑。

Serverless无疑已经成为学术界和工业界关注的焦点。

### **一、什么是Serverless**

回顾互联网快速发展的数年，催生了诸多技术创新，如大数据、微服务、DevOps等。这些技术并没有一个官方的定义，而是依赖于社区的蓬勃发展，不断优化其描述。

类似的，Serverless也没有一个官方的定义，但已有不少总结帮助我们理解到底什么是Serverless，如下是个人认为社区中颇具代表的总结：

### **1.使能开发者，关注[业务逻辑](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91&zhida_source=entity)本身而非应用的部署、维护和管理**

> Serverless technology is an infrastructure abstraction which enables developers to **focus on writing software and managing application performance, rather than deploying, maintaining and scaling the underlying server[infrastructure](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=2&q=infrastructure&zhida_source=entity)**。  
> —《Overview of Serverless technology, Enterprise Serverless User Cases》

### **2.无需管理服务器即可运行应用，并能按需的执行、扩缩容和付费**

> Serverless computing refers to the concept of building and running applications that do not require server management. It describes **a fine-grained deployment meodel where applications, bundled as one or more functions, are uploaded to a platform and then executed, scaled and billed in response to the exact demand needed at the moment**.  
> — 《What is Serverless Computing, CNCF Serverless WhitePaper v1.0》

### **3.是FaaS + BaaS的共同体**

> Serverless encompasses **two different areas: FaaS and BaaS.**

FaaS(Function As A Service) - 由开发人员实现，与[编程语言](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80&zhida_source=entity)相关，运行在容器或Micro-VM中。 

BaaS(Backend As A Service) - 由云厂商提供，支撑应用的完整业务逻辑，如[数据存储](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8&zhida_source=entity)、消息队列、用户鉴权等。典型的BaaS包括：

  * [数据库系统](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B3%BB%E7%BB%9F&zhida_source=entity) (RDS/DynamoDB…)
  * [存储系统](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%AD%98%E5%82%A8%E7%B3%BB%E7%BB%9F&zhida_source=entity) (S3…)
  * [鉴权系统](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E9%89%B4%E6%9D%83%E7%B3%BB%E7%BB%9F&zhida_source=entity) (Auth0…)
  * [集成系统](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E9%9B%86%E6%88%90%E7%B3%BB%E7%BB%9F&zhida_source=entity) (API-GW, EventsBridge…)



基于如上的定义，我们对Serverless已有了大致的理解，核心关键字：

  * **无需关注服务器**
  * **使能业务快速开发**
  * **由FaaS+BaaS生态构成**



因此，我将Serverless的定义，总结为：使能开发者**聚焦业务实现、无需关注运维与基础设施、并能按需使用和付费的技术。**

### **二、Serverless的核心**

在伯克利发表的论文《A Berkeley View on Serverless Computing》中，对云计算提出了一大假设：

**Serverless的出现，极大简化了[云计算](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=3&q=%E4%BA%91%E8%AE%A1%E7%AE%97&zhida_source=entity)的编程模型，将是软件生产力的一次大变革**。举例来说，（Serverful）向Serverless的演进，就如同若干年前，汇编语言向[高级语言](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E9%AB%98%E7%BA%A7%E8%AF%AD%E8%A8%80&zhida_source=entity)的演进。

>  _回想汇编语言的时代，开发者不仅需要考虑指令集，还需要熟悉[寄存器](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%AF%84%E5%AD%98%E5%99%A8&zhida_source=entity)的读取、信号的中断等。而高级语言的诞生，解放了开发者，不再需要考虑这些底层的细节，更关注业务逻辑。 就好比没有人会用汇编来实现Web应用，而是使用诸如PHP/JSP/NodeJS等技术。_

回顾过去几年，软件开发领域存在的几**次创新与变革** ，都极大的提高了开发者的工作效率：

> IaaS的出现：解决了基础架构的成本和[可扩展性](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%8F%AF%E6%89%A9%E5%B1%95%E6%80%A7&zhida_source=entity)问题  
> 容器(Docker)的出现：定义了标准的打包、部署和发布的机制  
> Kuberneters的出现：定义了容器的编排机制  
> Serverless的出现：使能开发者聚焦业务实现、无需关注运维与基础设施、并能按需使用和付费。

这些变革有一个共同点：**那就是赋予了工程师更多的权利，帮助其更快的交付应用。**

让我们聚焦Severless。为什么Serverless当前备受关注，是什么特征，使得它能够“使能**开发者聚焦业务实现** ”**呢** ？

个人理解其主要有是下两大核心特征：

**1\. 零运维成本**  
  
**开发人员/运维人员不再需要承担构建分布式系统的(绝大部分)运维成本** 。

为了构建[高并发](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E9%AB%98%E5%B9%B6%E5%8F%91&zhida_source=entity)、高可用的[分布式系统](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=2&q=%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F&zhida_source=entity)，我们通常需要**考虑基础设施和应用层面的因素，包括但不限于** ：

  * **基础设施层面：**  
a)补丁与升级  
b)[扩容机制](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6&zhida_source=entity)  
c)安全  
......  

  * **应用层面：**  
a)[单点故障](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%8D%95%E7%82%B9%E6%95%85%E9%9A%9C&zhida_source=entity)  
b)异地容灾  
c)负载均衡  
d)自动扩缩容  
e)监控告警  
f)调用链跟踪  
......



Serverless不仅帮助开发人员降低**基础设施的维护和[管理成本](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E7%AE%A1%E7%90%86%E6%88%90%E6%9C%AC&zhida_source=entity)**（服务器创建、补丁、升级），也**极大的简化了应用层面的运维工作**(负载均衡、[自动扩缩容](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=2&q=%E8%87%AA%E5%8A%A8%E6%89%A9%E7%BC%A9%E5%AE%B9&zhida_source=entity)等)，使得开发人员可以把精力聚焦在真正的业务实现上，灵活应对VUCA时代**需求的不确定性** 以及**[交付周期](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E4%BA%A4%E4%BB%98%E5%91%A8%E6%9C%9F&zhida_source=entity) 慢/成本高**等挑战。

**2\. 零资源浪费**  
  
**按照代码实际运行所占用的资源计费，而非分配或者租用的资源** 。

以计算资源为例，对于[公有云](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%85%AC%E6%9C%89%E4%BA%91&zhida_source=entity)的IaaS资源而言，计费方式是基于租用时长(如AWS的EC2，实例启动后，不运行任何业务，也会以分钟为单位进行计费)。

而Serverless的收费是按照**(代码)运行时消耗的资源** 收费。如Lambda是按照**请求次数** 及**代码实际运行** 占用的资源(**GB-秒为单位**)进行计费。  
  
**如无请求， 则无需支付任何费用** 。

更多关于Lamda的计费原则，请参考

[](https://link.zhihu.com/?target=https%3A//aws.amazon.com/cn/lambda/pricing/)

  


### **三、什么场景适合使用Serverless**

Serverless的应用场景，通常是“**[事件驱动](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8&zhida_source=entity) 、短生命周期的无状态**”应用。

主要包括如下几类：

### **1.事件驱动的后端服务：**

包括Web、移动App、IOT应用的后端服务等，如：

  * Web、移动App的后端实现(通过用户的交互触发事件)
  * IOT应用的后端实现(通过设备状态的变化触发或者上报事件)
  * WebHook的后台处理（通过Hook方式触发的事件）
  * ChatBot等[聊天机器人](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E8%81%8A%E5%A4%A9%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity)
  * Alexa等智能音箱的应用



### **2.[数据处理](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)类应用：**

包括对数据的渲染、叠加或者处理等应用，如：

  * 报表等数据处理（对原始数据进行封装）
  * ETL或者[数据同步](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5&zhida_source=entity)（对数据的抽取、转化和加载）
  * 视频或者[图像处理](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86&zhida_source=entity) (生成不同分辨率的图像或者视频)



### **3.适配类应用**

  * 协议类型的转换(如IOT中不同设备管理，需要协议适配)
  * 与第三方平台的接入（如通过第三方提供的接口，接入或者协作的业务）



>  _可以看出，移动应用/互联网应用作为目前应用形态的主流，在Serverless应用占比中是最高的。个人估计，随着IOT、AI等应用的普适与快速发展，这类应用在采用Serverless的统计中占比会增加。_

### **四、Serverless带来的改变**

### **1.开发理念的转变**

Serverless的出现，使得开发者更注重逻辑的快速实现，而无需关注运行、维护的成本，也就是**从以资源为中心向以应用为中心** 观念的转变。

### **2.角色职责的转变**

Serverless的出现，使得架构师、开发人员和运维人员的职责都发生了变化：

  * [架构师](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=2&q=%E6%9E%B6%E6%9E%84%E5%B8%88&zhida_source=entity)：从单体应用的**只关注[架构设计](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1&zhida_source=entity)**，到微服务时代的**“You build it, you own it”**(从设计到运行到运维)。Serverless的出现，使得架构师能够更加聚焦业务带来的技术挑战。
  * 开发人员：[微服务](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=3&q=%E5%BE%AE%E6%9C%8D%E5%8A%A1&zhida_source=entity)时代，开发者需学习不同框架、语言；Serveless时代， 关注输入/输出，和函数代码的处理逻辑
  * 运维人员：关注基础设施、运行时、监控告警、可靠性等问题，大部分可以交给Serverless平台处理了。



### **五、Serverless面临的挑战**

Serverless为开发人员带来了诸多便捷，但它依然处理发展的初期，相比与完善的Serverful的生态，还存在很多不足。主要包括：

### **1.计算领域**

  * [冷启动](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%86%B7%E5%90%AF%E5%8A%A8&zhida_source=entity)
  * [异构计算](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%BC%82%E6%9E%84%E8%AE%A1%E7%AE%97&zhida_source=entity)
  * 调度优化



### **2.存储模型**

  * 如何实现高性能、成本可控且可透明伸缩的存储机制



### **3.通信模型**

Serverless中的Function是不能直接寻址的，所以无法通过现有的数据一致性[算法](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E7%AE%97%E6%B3%95&zhida_source=entity)/机制实现数据的共享或者传递。Serverless必然会推动分布式数据协同机制的发展。

### **4.安全相关**

  * 物理资源共享引起的攻击
  * [细粒度](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E7%BB%86%E7%B2%92%E5%BA%A6&zhida_source=entity)的权限控制
  * 数据[安全风险](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E5%AE%89%E5%85%A8%E9%A3%8E%E9%99%A9&zhida_source=entity)



### **5.生态**

  * 工具成熟度
  * [遗留系统](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E9%81%97%E7%95%99%E7%B3%BB%E7%BB%9F&zhida_source=entity)的迁移
  * Serverless的设计模式
  * 合适的资源抽象
  * [端边云](https://zhida.zhihu.com/search?content_id=111922418&content_type=Article&match_order=1&q=%E7%AB%AF%E8%BE%B9%E4%BA%91&zhida_source=entity)的协同



### **参考**

  * 《A Berkeley View on Serveless Computing》
  * 《CNCF Serverless Whitepaper》
  * 《Enterprise Serverless user cases》
  * [Serverless architecture guide](https://link.zhihu.com/?target=https%3A//www.simform.com/serverless-architecture-guide/)


