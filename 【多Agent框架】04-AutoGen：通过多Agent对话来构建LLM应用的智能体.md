# 【多Agent框架】04-AutoGen：通过多Agent对话来构建LLM应用的智能体

原文链接：https://zhuanlan.zhihu.com/p/685103609

---

​

目录

## [AutoGen](https://zhida.zhihu.com/search?content_id=240375554&content_type=Article&match_order=1&q=AutoGen&zhida_source=entity)简介

随着大语言模型的爆火，使得人们看见了一个通往AGI的方向。基于[LLM](https://zhida.zhihu.com/search?content_id=240375554&content_type=Article&match_order=1&q=LLM&zhida_source=entity)所具备的意图识别解释、生成计划并自主行动的能力，人们相继构建出自主的智能体。智能体以LLM作为大脑，通过与环境、人类和其他智能体的交互，可以自主的解决各种复杂的任务。

Autogen是微软推出的一个Multi-Agent框架，允许用户创建和管理多个智能体，以协同完成复杂的任务。Autogen具有**可定制** 和**可对话** 的能力，同时支持**人类输入** 和**工具** 扩展其能力。基于Autogen，可以简化、优化和自动化大型语言模型的工作流程。其具有如下特点：

  * Autogen最大的特点是其**可定制、可交互、可人工干预** 。每个Agent都可以具有不同的能力，通过多个Agent之间的对话可以解决它们的局限性。
  * Autogen具有**高可扩展性** ，可以根据不同的场景，定义并实现所需的Agent类型（通过扩展子类实现）。
  * Autogen将复杂的Multi-Agent简化为定义一组**具有特定能力和角色** 的Agent。用户可以定义Agent之间的互动行为，包含定义Agent角色，及其协同模式。例如，如果我们要构建一个工程团队，我们可能会有一个是工程师，另一个是项目经理，再有一个是质量保证等等。然后你需要定义agent之间的互动行为，即当一个Agent从另一个Agent收到消息时该如何回复。所以你不仅仅是在定义Agent和角色，你还在定义它们如何协同工作。
  * Autogen并不完全依赖于[OpenAI](https://zhida.zhihu.com/search?content_id=240375554&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)的API（当然它自身提供了对OpenAI的API支持），只要提供了API，用户可以用Autogen**对接任何LLM** 。



图1：AutoGen通过多Agent对话实现多样化的LLM应用。（左）AutoGen具有对话能力、可定制性，并且可以基于LLMs、工具、人类甚至其组合。 （中间顶部）Agent可以对话解决任务。 （中间底部）该框架支持灵活的对话模式。（右）它们可以形成一个聊天，包括人类的参与。

## Autogen实现

AutoGen的核心设计原则是使用多Agent对话来简化和整合多Agent工作流程。其旨在最大限度地提高实现Agent的可重用性。AutoGen主要包含两个关键概念：**可对话Agent** 和**对话编程** 。

### 可对话Agent

在AutoGen中，可对话智能体是一个具有特定角色的实体，可以传递消息与其他可对话智能体之间发送和接收信息，例如启动或继续对话。它根据发送和接收的消息维护其内部上下文，并可以配置具有一组能力，例如由LLMs、工具或人类输入等启用。

**由LLMs、人类和工具提供支持的智能体能力**

AutoGen支持根据需求灵活的赋予智能体各种能力，其能力可以是以下能力的单个或者多个之间的组合。

  * **LLMs：** 基于LLMs的智能体，可以利用LLMs的许多优秀能力，如角色扮演、基于对话历史的简化性推理、提供反馈、根据反馈进行适应等。通过提示技术，这些能力支持以不同的方式组合起来，以增加智能体的技能和自主性。AutoGen还加强了LLM的推理能力，如结果缓存、错误处理、消息模板等。
  * **人类：** AutoGen允许人类参与到智能对话中，具体取决于智能对的配置，可以在对话的某些回合征求人类输入。
  * **工具：** AutoGen支持基于代码或函数来扩展其工具。



**Agent定制和合作**

AutoGen允许通过重用或扩展内置智能体轻松创建具有专业能力和角色的智能体。图2的黄色阴影区域提供了AutoGen中内置智能体的草图。ConversableAgent类是最高级的智能体抽象，可以默认使用LLMs、人类和工具。AssistantAgent和UserProxyAgent是两个预配置的ConversableAgent子类，分别表示常见的使用模式，即作为AI助手（由LLMs支持）和作为人类代理以征求人类输入或执行代码/函数调用（由人类和/或工具支持）。

图2：使用AutoGen实现多智能体对话的示意图。顶部子图示例AutoGen提供的内置智能体，这些智能体具有统一的对话接口并可进行自定义。中间子图显示了使用AutoGen开发具有自定义回复函数的双智能体系统的示例。底部子图说明了在程序执行期间从双智能体系统中产生的自动化智能体对话。

### 对话编程

AutoGen采用了对话编程，这是一种考虑两个概念的范式：一是计算，即在多智能体对话中计算其响应的行为。二是控制流程，即这些计算发生的顺序（或条件）。AutoGen具有以下设计模式，以促进对话编程：

**自动智能体对话的统一接口和自动回复机制**

AutoGen中的智能体具有统一的对话接口，用于执行相应的以对话为中心的计算，包括用于发送/接收消息的发送/接收函数和用于基于接收到的消息采取行动和生成响应的生成回复函数。AutoGen还引入并默认采用智能体自动回复机制来实现以对话驱动的控制：一旦智能体接收到来自另一个智能体的消息，它会自动调用生成回复并将回复发送回发送者，除非满足终止条件。AutoGen提供基于LLM推理、代码或函数执行或人类输入的内置回复函数。还可以注册自定义回复函数来自定义智能体的行为模式，例如在回复发送者智能体之前与另一个智能体进行对话。在这种机制下，一旦注册了回复函数并初始化了对话，对话流程就会自然引发，因此智能体对话会在没有任何额外控制平面的情况下自然进行，即不需要一个专门控制对话流程的模块。

**结合编程和自然语言的控制**

AutoGen允许在各种控制流管理模式中使用编程和自然语言：

  * **通过LLM的自然语言控制** ：在AutoGen中，**可以通过使用自然语言提示基于LLM的智能体来控制对话流程。** 例如，AutoGen中内置的AssistantAgent的默认系统消息使用自然语言指示智能体修复错误并重新生成代码，如果先前的结果表明存在错误的话。
  * **编程语言控制：** 在AutoGen中，**可以使用Python代码来指定终止条件、人类输入模式和工具执行逻辑** ，例如自动回复的最大次数。还可以注册编程的自动回复函数来使用Python代码控制对话流程，如图2中标识为“Conversation-Driven Control Flow”的代码块所示。
  * **自然语言和编程语言之间的控制转换：** AutoGen还支持自然语言和编程语言之间的灵活控制转换。可以通过调用包含某些控制逻辑的LLM推理来从代码转换为自然语言控制；或通过LLM提议的函数调用从自然语言转换为代码控制。



在对话编程范式中，可以实现各种模式的多智能体对话。除了具有预定义流程的静态对话外，AutoGen还支持具有多个智能体的动态对话流程。AutoGen提供了两种实现这一目标的常用方法：

  1. **自定义生成回复函数：** 在自定义生成回复函数中，一个智能体可以保持当前对话，并根据当前消息和上下文与其他智能体进行对话。
  2. **函数调用：** LLM根据对话状态决定是否调用特定的函数。通过向调用的函数发送消息给其他智能体，LLM可以驱动动态的多智能体对话。此外，AutoGen还通过内置的GroupChatManager支持更复杂的动态群组对话，它可以动态选择下一个发言者，并将其响应广播给其他智能体。



## Autogen多Agent核心能力

构建复杂的多智能体对话系统涉及如下两点：

  * 定义一组具有专门功能和角色的可对话智能体。
  * 定义智能体之间的交互行为，即一个智能体在接收来自另一个智能体的消息时应如何响应。



### 协作模式

Autogen所适用的场景示例：

Autogen支持以下多种协作模式：

  * **主从模式（群聊）** ：需要一个核心的Agent负责整个流程，其它Agent负责接受信息并将结果返回给主Agent，如：狼人杀场景，法官掌控整个流程；
  * **合作模式** ：Agent之间关系相对平等，一般是线性关系，如：软件开发场景下的架构师、产品经理、开发人员和测试人员是平等协作关系；
  * **竞争模式** ：Agent之间的关系是竞争或对抗的关系，如：狼人杀场景，预言家和狼人是竞争关系，辩论场景的双方辩手。



### 通信模型

AutoGen支持同步和异步的通信，其通信的消息类型是字典或字符串，消息可以包含内容（必填）、被调用函数、消息作用（function或assistant）、上下文（用于OpenAI API调用）等。

消息的传播方式，在一对一的场景下需要指定对应的recipient/sender；在群聊场景下，需要实现消息的广播。

### 人机协同

Autogen支持了多个Agent间交互、协同的能力，同时支持人工干预其中。当前典型的交互方式是通过user proxy agent里设置human_input_mode参数，从而在assistant agent每次返回结果时给予反馈。

## AutoGen总结

Autogen通过Agents之间的**对话和交互** ，简化了**复杂工作流** 的LLM**工作流程的编排** ，实现**自动化** 。同时，其可扩展性强，提供了缓存、错误处理、上下文编程等高级机制，借助可自定义和可对话的Agent，支持复杂工作流。同样，Autogen还具有一些待改进点，如Agent工作流固定化，Agents的自由度和交互的有效性待增强，框架没有plan、memory、prompt模块。

## 参考资料

  1. Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. Autogen: Enabling next-gen llm applications via multi- agent conversation framework, August 01, 2023 2023. 28 pages.
  2. [https://github.com/microsoft/autogen](https://link.zhihu.com/?target=https%3A//github.com/microsoft/autogen)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】03-Agents：首个采用SOP机制建立可控机制的智能体](https://zhuanlan.zhihu.com/p/682883749)  
> 下一篇：[【多Agent洞察】05-AgentScope：灵活而强大的多Agent平台](https://zhuanlan.zhihu.com/p/695452823)
