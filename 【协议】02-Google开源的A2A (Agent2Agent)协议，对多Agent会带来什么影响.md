# 【协议】02-Google开源的A2A (Agent2Agent)协议，对多Agent会带来什么影响

原文链接：https://zhuanlan.zhihu.com/p/1895434611774949228

---

​

目录

2025年4月9日，Google发布了文章《A2A: A new era of agent interoperability》，提出了一个新的开放协议**A2A（Agent-to-Agent），允许AI Agent之间在各种企业平台或应用程序上相互通信、安全地交换信息和协调行动** 。4月10日，在Google Cloud Next 2025大会上，谷歌开源了Agent2Agent Protocol。

本文介绍是什么A2A，它的定位以及和[MCP](https://zhida.zhihu.com/search?content_id=256425635&content_type=Article&match_order=1&q=MCP&zhida_source=entity)有什么关系，欢迎大家一起讨论。

## 1 什么是A2A

**[A2A协议](https://zhida.zhihu.com/search?content_id=256425635&content_type=Article&match_order=1&q=A2A%E5%8D%8F%E8%AE%AE&zhida_source=entity) 是一项开放标准，可让不同平台和框架之间的AI Agent进行通信和协作，而无需考虑其底层技术。其旨在通过实现真正的多Agent场景，最大限度地发挥Agentic AI的优势**。

A2A的核心作用是解决**Agent之间的沟通和协作** ，以及Agent与用户的交互。即Agent之间能像人与人交流一样，传递意图、协商任务、共享信息。A2A使开发人员构建的Agent，能够连接任何其它的Agent（基于A2A构建），并使用户能够灵活组合来自不同供应商的Agent。最重要的是，企业可以通过标准化方法管理其跨不同平台和云环境的Agent。

### 1.1 A2A设计原则

A2A作为开放协议，为Agent之间的协作提供了标准方式，不受底层框架或供应商限制。在与合作伙伴共同设计该协议时，遵循了五大核心原则：

  * **拥抱Agent能力** ：专注于让**Agent以自然的、非结构化的方式进行协作** ，即使它们不共享内存、工具和上下文。使得其支持构建真正的多Agent场景，而非将Agent局限为"工具"。
  * **基于现有标准构建** ：该协议**建立在现有的流行标准之上** ，包括HTTP、SSE、JSON-RPC等，便于与企业现有IT基础设施集成。
  * **安全** ：在设计上支持**企业级身份验证和授权** ，首发版本即实现与OpenAPI认证方案的全面对接。
  * **支持长时间运行任务** ：协议设计灵活，既能处理即时任务，也能支持需要数小时甚至数日（含人工介入）的深度研究，过程中可实时提供反馈、通知和状态更新。
  * **模态无关** ：Agent世界不局限于**文本** ，A2A设计支持包括**音频、视频** 在内的多种模态。



### 1.2 A2A如何工作

如上图所示，A2A促进了客户端Agent和远程Agent之间的通信。客户端Agent负责制定和传达任务，远程Agent则负责执行任务。该交互涉及以下几个关键能力：

  * **能力发现** ：Agent可通过JSON格式的"Agent卡片"（Agent Card）公布其能力，使客户端Agent能识别最佳执行者并通过A2A建立通信。
  * **任务管理** ：以任务完成为导向的通信架构，协议定义"任务"对象及其生命周期。任务可立即完成，对于长期任务，Agent间可通过持续通信保持状态同步。任务输出称为"工件（artifact）"。
  * **协作机制** ：Agent可通过互发消息传递上下文、回复、工件或用户指令。
  * **用户体验协商** ：每条消息包含"部件（parts）"——完整内容单元（如生成图像）。每个部件指定内容类型，允许Agent协商所需格式，并显式包含用户界面能力协商（如iframe、视频、网页表单等）。



有关协议工作原理的详细信息，请参阅[规范草案](https://link.zhihu.com/?target=https%3A//github.com/google/A2A)。

## 2 MCP vs A2A

2024年11月25日，[Anthropic](https://zhida.zhihu.com/search?content_id=256425635&content_type=Article&match_order=1&q=Anthropic&zhida_source=entity)发布了MCP（Model Context Protocol），其提供了一种统一的、标准化的方式，来管理和声明AI应用中的外部资源和上下文。其核心作用是统一并标准化AI应用与工具/资源交互的方式，使得AI应用可以有效的连接到工具、API和资源。

那么，MCP和A2A有什么区别？

### 2.1 MCP和A2A的区别

MCP是连接Agent与资源/工具的桥梁，而A2A则是开启Agent交互大门的钥匙。

协议| 定位不同| 解决问题不同  
---|---|---  
MCP| 关注Agent与Tools/Resources的交互，强调结构化的调用和资源获取。| 解决的是如何让[LLM](https://zhida.zhihu.com/search?content_id=256425635&content_type=Article&match_order=1&q=LLM&zhida_source=entity)/Agent标准化地使用外部工具和资源的问题。  
A2A| 关注Agent与Agent的交互，强调更自然、灵活的协作方式。| 解决的是如何让不同平台/框架的Agent能够像人类一样互相理解和协作的问题。  
  
### 2.2 互补而非替代：A2A与MCP的协同工作

A2A和MCP并非相互替代的关系，而是互补的。一个复杂的Agent应用往往同时需要这两种协议。

下图展示了一个典型的“Agentic Application”（智能体应用）架构：

  1. Agent（可能包含子Agent）基于Agent框架和LLM构建。
  2. 当Agent需要与外部Agent（如Blackbox Agent 1, Blackbox Agent 2）进行沟通协作时，使用A2A协议。
  3. 当Agent需要访问结构化的资源、工具时，使用MCP。



这个架构清晰地表明，Agent需要MCP来“使用工具、获取资源”，同时需要A2A来“与其他Agent对话、协作”。

## 3 总结

A2A和MCP是实现更复杂、更强大的多Agent应用商业落地的重要基础。MCP专注于Agent与工具/资源的连接，标准化函数调用；而A2A则专注于Agent之间的自然协作与沟通，它们通过互补的方式共同构成了未来复杂Agent应用的基础设施。

## 相关链接

  1. [https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/](https://link.zhihu.com/?target=https%3A//developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)
  2. [https://google.github.io/A2A/#/topics/a2a_and_mcp](https://link.zhihu.com/?target=https%3A//google.github.io/A2A/%23/topics/a2a_and_mcp)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【协议】01-MCP：AI时代的HTTP，大模型应用连接数据源的新桥梁](https://zhuanlan.zhihu.com/p/30949247510)  
> 下一篇：[【协议】03-Token从15万降低至2千，使用MCP执行代码](https://zhuanlan.zhihu.com/p/1971238278771500576)
