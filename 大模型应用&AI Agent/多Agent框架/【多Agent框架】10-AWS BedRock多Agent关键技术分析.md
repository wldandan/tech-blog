# 【多Agent框架】10-AWS BedRock多Agent关键技术分析

原文链接：https://zhuanlan.zhihu.com/p/1891162001331450878

---

​

目录

## 1 AWS多Agent介绍

AWS在2024年12月发表的论文《Towards Effective GenAI Multi-Agent Collaboration: Design and Evaluation for Enterprise Applications》中，提出了多层级结构智能体协作框架（MAC），定义了两种类型的Agent（Supervisor/Specialist），使用Supervisor对任务进行拆解，并交由Specialist智能体进行处理。框架提供了智能路由机制以及payload引用功能，优化了多Agent通信效率以及编排效率。论文最后提出多智能体框架测试基准，并验证了不同场景下，基于MAC（多Agent协同）相比开源多Agent框架和单Agent的成功率提升。

## 2 AWS多Agent框架关键技术

论文中给出了框架中对多Agent分类和定义，提出了Supervisor Agent（①）和Specialist Agent（②）的概念。其中Supervisor Agent主要用于流程组织、任务分解以及分配任务/请求到Specialist Agent，Specialist Agent用于完成特定的任务（如订酒店、订机票），类似于前文提到的流程型Agent的单条分支的功能。

框架中提出了三种关键技术：

  1. **统一多Agent通信协议，实现并行通信（③）** ：Agent间通信组件，是多智能体协作的关键组成部分，框架定义了标准的通信协议，在Supervisor和Specialist之间发送和接收消息，同时支持并行通信，提升通信效率。
  2. **通过Payload引用（④）降低Agent间交换知识的成本** ：在代理之间高效地共享大型内容块（如代码片段或详细的旅行行程），显著降低通信开销。代理可以使用唯一标识符引用先前共享的负载，而不是重复传输大数据块。
  3. **通过[动态路由](https://zhida.zhihu.com/search?content_id=255964076&content_type=Article&match_order=1&q=%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1&zhida_source=entity)（⑤）完成agent间高效协作**: 通过一个快速分类器判别专家Agent是否直接bypass请求，还是针对请求对任务进行编排，然后再协同其它Specialist Agent共同完成任务。



### 2.1统一多Agent通信协议及并行通信

**统一通信协议**

AWS的多Agent框架借助了大模型的function call能力，将Supervisor和Specialist间通信作为工具（send_message）使用，从而完成智能体间的消息交换，send_message()函数有**接受者** 和**内容** 两个入参。Specialist的返回格式如下。  
  
…  
  
这样的好处在于统一了用户和智能体/智能体和智能体间的交互接口/协议，同时框架直接利用模型对于底层工具调用相同的机制，无需额外开发通信模块。

**并行通信**

如下图，Supervisor Agent通过分析用户输入，将任务拆解为订酒店、订餐厅以及本地信息查询三个子任务，同时并行的将三个任务分给不同的智能体执行。并行通信是框架在工程上的实现，主要是提升了通信的效率。

### 2.2 通过payload引用降低Agent间交换知识的成本

如上图，多Agent协同过程中，如果包含代码生成的场景，大量代码的交换会降低通信的效率。优化智能体间的通信，尤其是减少智能体间的时延，对于多智能体间的信息交换是至关重要的。当前主流框架均使用结构化的通信交流，并设计通信模式以提高执行效果。如**AutoGPT** 运行智能体间进行多轮对话的，以进行信息共享和结果优化；**LangGraph** 中的智能体通过有向无环图的任务顺序进行通信，确保信息传递的准确性和及时性。这些框架的通信机制大都只用于优化处理结果，但优化流程往往**加重了通信效率** 。

AWS的多智能体框架提出了payload引用的概念，将需要共享的内容自动编码，仅传递编码的id，在实际的使用时，解析编码id获取内容，完成内容的传递。详细处理流程如上图：

  1. ①检测与标记：当智能体生成包含结构化内容（如代码片段）的消息时，系统会自动检测这些内容，并将其标记为 Payload，同时为其分配一个唯一标识符fm3ej。
  2. ②引用标记：当智能体发送的内容中需要引用某个Payload时，系统会自动监测引用标签，并使用一个简化的引用标签<share_payload id=“fm3ej”>将其替换。
  3. ③替换标记：当智能体接收的内容中包含对fm3ej payload的引用时，系统会将该引用标签解析为其引用的真正内容，并传递给智能体。



因为论文中并没有给出实际的框架实现，我们推测AWS多智能体间是通过网络跨进程通信的，框架需要将引用的payload外置，使用时候再做解析，把直接通信的成本降低。但同时需要保证模型不针对编码的标签进行生成，这需要模型侧做一些配合。

### 2.3 通过动态路由完成agent间高效协作

高效的编排结构能有效提升智能体间的协作能力，更好的完成共同目标。智能体编排结构的设计不仅需要考虑如何更有效的调度智能体，完成对用户请求的有效处理，还需要考虑提升性能，减少用户对处理时间的感知。当前主流框架的编排结构主要分为中心化（centralized）和去中心化结构（decentralized）。

  * **中心化结构的框架（ChatDev）** ：使用中心智能体对任务进行拆解与分析，并将拆解后的任务分配给专家智能体进行处理。各专家智能体处理完后，交由中心智能体汇总处理结果，总结并反馈给用户。
  * * **去中心化结构的框架（MAD，Generative Agents）** ：各智能体可以独立做出决策，而不依赖于中心节点的指令，最后通过共识或投票机制完成对用户请求的处理。



AWS的多Agent框架从实际的场景出发，采用了中心化的协作方式，使用Supervisor Agent来进行任务编排和分配。Supervisor在处理请求时，需要先通过一个高性能的分类器，完成意图识别。如果认为不需要任务规划分解时，如下图所示，当识别意图为酒店查询时，直接bypass请求给酒店预定Agent处理。分类识别失败，走原有的任务规划分解的逻辑。这样做的好处是提升了部分简单、明确任务场景的执行效率。

### 2.4 效果评测

论文中给出了多Agent框架测评的工具设计，以及基于三个场景（1）抵押贷款融资、（2）软件开发、（3）旅行计划的评测。对比了AWS多Agent框架和单Agent以及其它多Agent框架在GSR（成功率）等指标下的对比。  
测试主要基于[Claud](https://zhida.zhihu.com/search?content_id=255964076&content_type=Article&match_order=1&q=Claud&zhida_source=entity)不同规格的模型，三个场景下，多Agent的总体成功率都接近90%，和单Agent相比，提升超过37%。多Agent带来的损失主要在通信时延会有所增加。

和其它多Agent框架对比，AWS的多Agent框架也实现了超过30%的成功率提升。

## 3 如何使用[Bedrock](https://zhida.zhihu.com/search?content_id=255964076&content_type=Article&match_order=1&q=Bedrock&zhida_source=entity)构建多Agent

当前，AWS的用户已经可以在Bedrock上创建并使用多Agent，也可以通过AWS的SDK完成多Agent的创建和执行。整个的流程比较简单，如对于下图中的社交媒体活动场景，Campaign Manager是Supervisor Agent，它可以把任务分发给活动策略规划的Agent和活动参与度预测Agent来完成一次媒体活动的规划。

在Bedrock上通过下面的流程，就可以快速通过零码方式完成多Agent的开发和运行，Bedrock同时提供了基础的多Agent调测能力。和论文中提到的智能路由不同，当前用户需要手动制定协作模式是routing（bypass）或者supervisor（由supervisor做任务规划）。

## 4 总结

2024年中虽然涌现了一些多Agent的开源框架，如Langraph、AgentScope等，也有Coze的多Agent编排，和Azure的AutoGen。随着本次的AWS大会上推出的Bedrock多Agent，25年会有更多的平台支持多Agent特性，围绕多Agent的商业落地案例也会逐步增多，很有可能成为商用落地的元年。

## 相关链接

  1. 《Towards Effective GenAI Multi-Agent Collaboration: Design and Evaluation for Enterprise Applications》
  2. AWS re:Invent 2024 top announcements  
Design and Evaluation for Enterprise Applications》
  3. AWS re:Invent 2024 top announcements
  4. Introducing multi-agent collaboration capability for Amazon Bedrock



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】09-具身LLM智能体学习有组织的团队合作](https://zhuanlan.zhihu.com/p/16131160730)  
> 下一篇：[【多Agent洞察】11-AWS Multi-agent Orchestrator技术分析](https://zhuanlan.zhihu.com/p/1893252158327063392)
