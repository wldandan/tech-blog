# 【大模型应用】03-企业级Agent平台Oracle Agent洞察

原文链接：https://zhuanlan.zhihu.com/p/1934329882638284078

---

​

目录

## 1 概述

[OCI](https://zhida.zhihu.com/search?content_id=261077900&content_type=Article&match_order=1&q=OCI&zhida_source=entity)（Oracle Cloud Infrastructure）生成式AI Agent平台是一个完全托管的企业级解决方案，旨在帮助企业轻松构建、部署和管理AI智能体。该平台可帮助企业创建智能体，提供个性化、情境感知和高度吸引人的客户体验。同时，其支持无缝集成到现有工作流程，提供全面的生命周期管理功能，并具备原生可扩展性，可适应不断变化的业务需求。其主要特色包含两点：

  * **与RAG能力集成，增强与企业知识库对话的能力** ：增强用户从海量知识源检索和生成信息的能力，支持智能体从非结构化数据源动态提取相关数据，从而提高响应准确性和上下文深度，实现访问实时、高质量的信息，同时保持对数据相关性和可信度的控制。 
  * **提供[NL2SQL](https://zhida.zhihu.com/search?content_id=261077900&content_type=Article&match_order=1&q=NL2SQL&zhida_source=entity)能力**：用户能够通过自然语言即可实现SQL查询功能，直接访问和分析结构化数据，并更快、更轻松地获得有价值的见解。



## 2 Oracle AI解决方案全景

Oracle的AI策略围绕着企业使用AI的实际场景而制定，其中主要有三种不同的模式：基础设施、模型和服务（OCI Generative AI服务）以及应用。其中企业应用内，Oracle将生成式AI嵌入到业务用户每天使用的应用和工作流中。 

  * **① AI基础设施**  
OCI提供的裸金属和虚拟机GPU集群，以及超高带宽网络和高性能存储，可以满足推断、微调或者训练模型等需求。其具有多达131072个GPU，8倍更高的可扩展性，可利用高性能挂载目标(HPMT)，实现高达500Gb/秒的持续吞吐量。 
  * **② 基础模型**  
采用大语言模型[Cohere](https://zhida.zhihu.com/search?content_id=261077900&content_type=Article&match_order=1&q=Cohere&zhida_source=entity)，Oracle也是Cohere的投资方。
  * **③ 行业模型**  
为医疗卫生和公共安全部署新模型（猜测在这两个行业拥有大量、高质量的公开数据，如Oracle Cerner管理着数十亿的电子健康记录）。 
  * **④ 内嵌AI的业务应用**  
跟SAP类似，Oracle计划将生成式AI嵌入到Fusion、[NetSuite](https://zhida.zhihu.com/search?content_id=261077900&content_type=Article&match_order=1&q=NetSuite&zhida_source=entity)等垂直SaaS产品组合中，为企业提供即开即用的AI能力。 
  * **⑤ OCI Generative AI服务**  
支持OCI客户通过简单的API，为其应用和工作流添加生成式AI功能，并享有公有云的灵活性与可扩展性。Oracle通过基于云技术的部署选项（例如OCI Dedicated Region）在企业的数据中心提供完整的AI堆栈，让企业能够将生成式AI功能与其本地数据和应用无缝结合。 
  * **⑥向量数据库**  
内置AI Vector Search，支持RAG方案，实现语义级别的数据向量化与相似性查询。Oracle还在其数据库和自动化数据库上增加了基于LLM的自然语言界面。 
  * **⑦数据安全**  
OCI提供的生成式AI服务可帮助客户更高效地掌控数据。在推理和微调过程中，Oracle不检查且不与Cohere分享企业的数据，客户训练的模型归客户私有。此外，Oracle还提供了数据溯源和数据沿袭的工具。



## 3 OCI生成式AI Agent平台技术解析

OCI生成式AI Agent服务，基于可扩展和易迭代的理念，采用模块化解耦系统及其各个组件，其架构图如下所示。

  * **Agent Core** ：LLM Agent的“大脑”负责生成概念、制定计划、选择要学习的课程、解析响应，并最终生成最终响应返回给用户。
  * **记忆** ：是一个存储聊天记录、想法和计划以及行动结果的系统。
  * **工具** ：增强智能体能力的额外功能，使其能够执行标准文本生成之外的任务，例如调用函数获取实时信息、运行Python代码或访问特定数据库以获取最新或专业信息。
  * **知识库** ：LLM会从中获取更多、更优质的数据，根据用例进行定制，从而增强响应能力。 



### 3.1 Agent Core

Agent Core作为“大脑”，具有高度可定制性，并可以适应各种类型的智能体。智能体核心的内部工作原理如下图所示。

  * **①编排循环** 是一个连续的循环，其中Agent观察其环境，并根据其观察和内部逻辑决定操作、执行操作，然后根据操作的结果和新的观察更新其理解。
  * **②动作路由器** 根据上一步的动作计划决定流程路径并将请求路由到相应的组件。 
  * **③动作响应解析器** 解析来自动作的响应并将信息输入Agent核心，为下一次迭代做准备。 
  * **④Agent消息** 生成器创建对用户的响应并完成当前对话回合。 



### 3.2 RAG检索工具

Agent core通常规划零个或多个检索步骤、可选的工具流程以及最终的生成步骤。RAG检索工具负责从各种数据源检索相关信息，并返回结果。RAG检索是提升回答准确率的最关键部分。借鉴了传统搜索引擎的以下关键技术，并将其应用于RAG检索工具：

  * **查询重写** ：接收用户的查询，并将其转换为适合搜索的格式。例如，重新格式化并优化[OpenSearch](https://zhida.zhihu.com/search?content_id=261077900&content_type=Article&match_order=1&q=OpenSearch&zhida_source=entity)搜索的用户输入查询。
  * **查询执行** ：RAG检索工具会将准备好的查询发送到一个或多个知识库，以检索前N个相关的文档，寻找所需信息。
  * **文档重新排序** ：返回调查结果后，完善搜索结果，根据一些补充标准进行排序。
  * **知识库** ：客户可以将自己的OCI与OpenSearch实例一起使用，该实例中的所有文档都已编入索引并准备就绪。可以将数据挂载到对象存储桶中，该存储桶会自动提取到服务托管的知识库中。 



### 3.3 多轮对话

在智能体中， 通过RAG Agent来处理多轮对话，提升准确度。每轮对话的输入和输出都存储在记忆中。在查询重写步骤中，Agent将当前用户查询和之前的对话作为输入，并生成与整个对话相关的搜索查询。

示例：第2回合和第3回合的重写包含了来自前面步骤（请求和响应）的重要信息。 

  1. 第1轮：“我的计算虚拟机无法访问。请帮忙。” 被重写为“计算虚拟机无法访问。” 
  2. 第2轮：“好的，我不熟悉步骤 1，您能帮忙吗？”被重写为“检查基础设施健康指标Oracle Cloud。” 
  3. 第3轮：“哇，这里有这么多指标。哪些指标能帮我解决问题？” 被重写为“无法访问虚拟机的重要指标。”



每一次对话都建立在上一次对话的基础之上。下图展示了本例第2轮在RAG智能体内部的工作流程：

  1. 原始用户查询被写入系统生成的搜索查询，该查询被发送到知识库以检索相关文档。查询重写步骤将先前对话的认知融入到搜索词中。 
  2. 对检索到的文档进行重新排序，同时输入这些文档的嵌入、用户查询的嵌入和重写的查询。
  3. 重新排序后，排名前三的文档和所有上下文信息被输入到响应生成LLM中。 



### 3.4 数据处理和保护

Agent服务结合OCI和OpenSearch，可以索引和存储非结构化数据，从而提供更快速、更准确的检索能力。借助OCI和OpenSearch的精细访问控制，客户可以在索引和查询时从管理角度对其数据进行强有力的控制。

客户可以自主选择数据处理方式。可以选择将RAG Agent链接到OCI中的现有非结构化数据（OpenSearch Cluster或对象存储）。他们的数据将在独立的工作负载中处理，并且该服务仅根据实时对话期间的查询和上下文检索数据。

### 3.5 模型准确率

为推动企业采用，通过多种技术（包括RAG）来提高准确率。GenAI RAG Agent实现了基于数据的生成，以帮助减少幻觉。同时，添加了输出护栏，以帮助进行以下验证： 

  * 获取用户查询、检索到的支持内容、生成的响应作为输入。
  * 产生分数来表明所生成的响应在多个维度上的质量，例如有用性、依据性和准确性。 



RAG智能体输出的验证流程如下所示：

GenAI智能体框架还可以使用工具，为行为纠偏提供透明的方法，从而实现更加流畅的交互。如下图示例，展示了一个具有SQL功能的智能体，包括SQL生成、修正、SQL执行和响应生成。

### 3.6 可扩展性

第一个可扩展点是**工具** 。可以利用更多类型的工具，来实现各种业务目标。

第二个可扩展点是**知识库** 。通过增加更多、更优质的知识库提升RAG能力，涵盖非结构化、半结构化和结构化数据，乃至多模态数据。可以使用各种索引与检索技术，如混合搜索（结合关键词和语义搜索的优点），以及多种嵌入模型。

### 3.7 Oracle AI Agent Studio

Oracle AI Agent Studio是Fusion云套件之一，用于创建、扩展、部署和管理企业AI智能体的综合平台，为客户和合作伙伴提供了易于使用的工具，用于创建自定义AI智能体，以满足复杂的业务需求，并帮助将生产力提升到新的水平。

Oracle AI Agent Studio包括： 

  * **Agent模板库** ：用户可以利用预建的模板配置自然语言提示，自行创建AI智能体。用户可以利用现成的模板库，支持各种业务场景，例如报价、采购处理或排班计划等。
  * **Agent团队协商** ：用户可以通过预先配置的模板来配置多个AI智能体与人员，以更好地处理复杂的任务。为了掌握更多的控制权，用户可以在多个步骤流程中加入检查点与平台。
  * **Agent增强性** ：用户能够通过新增文件、工具、修改或提示API来和扩展50多个预先包装的Oracle融合应用程序AI智能体，满足特定的行业和业务需求。
  * **LLM选择** ：用户可以使用多种大型语言模型（LLM）来满足特定的业务需求。用户可以选择专门针对Oracle Fusion应用优化的LLM（例如Llama和Cohere），或插入其他外部特定行业的LLM来满足特定的场景需求。
  * **Fusion集成** ：用户能够直接访问Oracle Fusion应用程序API、知识库和预先定义的工具，快速构建符合企业需求的Agent，而不需要进行复杂的自定义。这种深度集成自动保留了企业特定的业务逻辑，并评估由AI驱动的工作流程中。
  * **第三方系统集成** ：用户能够使用安全的API，将Oracle Fusion Applications的AI智能体与第三方智能体连接，支持即时后续步骤以及长期运行的流程。
  * **信任和安全框架** ：用户能够在安全框架下建置和部署智能体，要求AI Agent Studio中的智能体始终应用新的Oracle Fusion应用程序安全安装、策略与访问控制。
  * **验证和测试工具** ：用户能够验证与监控AI驱动工作流程中的结果，以保持信任和准确性。例如，内置的验证与测试工具有助于支持AI输出的可靠性、可重复性、可解释性、安全性与结果。



**构建AI智能体的步骤**

创建AI，共分为7步，如下所示：

### 3.8 多Agents构思

未来多智能体将围绕多种协同工作的智能体类型展开。Conversational Agent、Supervisory Agents、Functional Agent和Utility agents相互协作，共同达成预期成果。在一个典型工作流程中，这些智能体会进行交互、运用工具、查找必要支持数据、做出决策，并联合完成手头任务。

  * **Conversational Agent** ：与外部世界交互，人和软件程序等。
  * **Supervisory Agents** ：管理智能体，多个Agents的管理、协调者，并推动实现目标所需的规划和推理。
  * **Functional Agent** ：功能性Agent，也称为user-proxy agent，关联组织中的个人或者角色，是领域专家，可以使用不同的工具，能够完成特定任务，例如Hiring manager agent、Customer support agent。
  * **Utility agents** ：也称为task-based agents，关联具体的函数或者工具，被其它Agent调用完成特定子任务。



## 4 总结

Oracle基于AI基础设施、模型、数据、应用等构建了统一的AI解决方案架构，并在此基础上打造了其生成式AI服务和生成式AI智能体范式，为其构筑自己的商业护城河。其生成式AI智能体为客户带来了如下能力： - RAG范式结合了对话式AI的能力，实现“与数据对话”的能力，利用最新、丰富的上下文数据。 - GenAI智能体通过配置和连接服务与知识库，简化了AI应用的上线流程。 - GenAI智能体实现了多种技术，如查询重写、语义重排序和输出护栏，支持高保真的多轮对话，提升了上下文准确性。

## 5 企业级Agent平台发展趋势

通过分析Salesforce的Agentforce、SAP的SAP Business AI和Oracle的OCI生成式AI智能体平台等企业级Agent平台，企业级Agent平台的能力主要包含如下几方面：

  * **统一Agent平台** ：建立公司统一的Agent平台，各产品基于统一平台创建、使用Agent。
  * **跨职能协作** ：调用不同职能的Agent，实现复杂的端到端业务流程。以嵌入式形式提供AI能力，针对流程或系统进行体验或效率优化。
  * **推理技术增强** ：Salesforce的Agentforce采用了Atlas推理引擎，运用RAG检索增强技术，显著减少模型幻觉，将准确率从测试版的40%-70%提升至90%-95%。
  * **多模态与多数据源集成** ：AI Agent越来越多地集成多模态数据（如文本、图像、语音）和多数据源（如内部数据库、外部API），以提供更丰富的上下文和更准确的决策支持，例如SAP Business Data Cloud进行了数据的统一与整合，Agentforce的数据都来源于Data Cloud等。
  * **多Agent协作** ：未来AI Agent将更多地以多Agent系统的形式出现，不同Agent之间可以协作完成复杂的任务。例如在一个项目中，一个Agent负责数据收集，一个Agent负责分析，一个Agent负责报告生成。
  * **支持用户定制Agent** ：支持快捷地创建并调试Agent。
  * **生态扩展能力** ：支持接入不同的模型，支持与不同云服务商数据对接。
  * **数据隐私与安全** ：随着Agent对数据的依赖增加，数据隐私和安全成为关键。企业需要确保Agent在处理敏感数据时符合相关法规和标准，同时保护数据免受攻击和泄露。



## 6 企业级Agent平台架构思考

构建企业级Agent平台架构需要从业务价值、功能性、扩展性、安全性和用户体验等几个维度进行全面考虑。此外，平台架构应考虑多Agent协作的可扩展性，从单Agent架构向多Agent动态协同演进，以满足企业复杂任务和跨部门需求。我们认为企业级Agent平台的架构如下所示，包含基础设施层、模型层、数据云层、Agent服务层、工具/平台和业务应用层。

  * **应用层** ：企业级各种应用，可以集成使用Agent，调用不同职能的Agent，实现复杂的端到端业务流程。
  * **Agents** ：各类开箱即用的预置Agents及用户扩展的Agents。
  * **Data Cloud** ：多模态与多数据源集成，实现数据的统一与整合。
  * **模型** ：支持不同类型的模型，可扩展支持第三方模型。
  * **平台/工具** ：扩展能力，便于构建Agent，提供数据安全、提示词优化、低码开发等能力。
  * **基础设施** ：包含部署Agent、自定义模型的训练等。



## 相关链接

  1. [https://blogs.oracle.com/cloud-infrastructure/post/behind-the-scenes-with-generative-ai-agents](https://link.zhihu.com/?target=https%3A//blogs.oracle.com/cloud-infrastructure/post/behind-the-scenes-with-generative-ai-agents)
  2. [https://blogs.oracle.com/ai-and-datascience/post/ga-of-oci-gen-ai-agent-platform](https://link.zhihu.com/?target=https%3A//blogs.oracle.com/ai-and-datascience/post/ga-of-oci-gen-ai-agent-platform)
  3. [https://www.oracle.com/cn/a/ocom/docs/applications/the-rise-of-ai-agents-unleashing-productivity-and-innovation.pdf](https://link.zhihu.com/?target=https%3A//www.oracle.com/cn/a/ocom/docs/applications/the-rise-of-ai-agents-unleashing-productivity-and-innovation.pdf)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【大模型应用】02-企业级Agent平台SAP AI洞察](https://zhuanlan.zhihu.com/p/1928135774634750816)
