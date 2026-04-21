# 【大模型应用】02-企业级Agent平台SAP AI洞察

原文链接：https://zhuanlan.zhihu.com/p/1928135774634750816

---

​

目录

## 1 [SAP AI](https://zhida.zhihu.com/search?content_id=260329584&content_type=Article&match_order=1&q=SAP+AI&zhida_source=entity)概述

SAP AI是SAP基于丰富的业务数据和业务实践专为企业构建的**商业AI（Business AI）** ，通过**将AI能力直接嵌入至SAP的生态系统（采购、财务、人力资源和客户服务流程等），覆盖130+个AI赋能场景** ，全面实现SAP应用的智能化，并帮助企业实现自动化、运营方式的转型。SAP AI主要的优势如下： 

  1. SAP AI解决方案深植于其关键业务管理系统中，提供基于完整业务情境的AI智能体。 
  2. 将AI与SAP多年的行业实践、业务流程相结合，提供超过130种AI场景，可覆盖财务、供应链、采购、人力资源等多个领域。 
  3. SAP AI解决方案可为用户提供最高程度的数据隐私和安全合规管理。



## 2 [SAP Business AI](https://zhida.zhihu.com/search?content_id=260329584&content_type=Article&match_order=1&q=SAP+Business+AI&zhida_source=entity)技术解析

SAP Business AI是一套SAP人工智能功能，提供专为业务构建、跨业务流程使用的“人工智能”，赋能客户敏捷地掌控业务。

  * 赋能所有业务流程，例如云ERP、人力资本管理、智能支出管理以及业务网络和客户关系管理等。
  * SAP Business AI与SAP应用程序无缝协作，利用现有数据提供切实可行的洞察并自动执行日常任务。
  * 业务流程的核心组件。



### 2.1 [SAP Joule](https://zhida.zhihu.com/search?content_id=260329584&content_type=Article&match_order=1&q=SAP+Joule&zhida_source=entity)

Joule能够调动企业内的所有AI Agent，准确执行复杂的工作流程，在日常运营中为企业创造价值。

  * **利用全面的业务数据** ：无论企业用户从何处访问Joule，企业所有的事务、 运营数据或分析随时随地可用（有权限的情况）。
  * **将AI嵌入其解决方案** ：整合了多个可以跨职能协作的AI智能体，支持用户定制AI智能体。



Joule依赖两大关键要素：

  * **[SAP Knowledge Graph](https://zhida.zhihu.com/search?content_id=260329584&content_type=Article&match_order=1&q=SAP+Knowledge+Graph&zhida_source=entity)** ：建模业务实体（例如客户、供应商、产品和交易）之间的复杂关系。将人工智能 (AI) 置于特定的 SAP 业务语义及其相互关系之上。  
1）**提供丰富互联的数据源** ，452000个ABAP表、80000个CDS视图和730万个字段。  
2）**明确实体之间的含义和关系** ，这有助于将LLM固定在SAP的整个语义模型。  
3）**将自然语言世界与SAP的结构化、表格世界以及非SAP应用程序相结合** 。
  * **[SAP Business Data Cloud](https://zhida.zhihu.com/search?content_id=260329584&content_type=Article&match_order=1&q=SAP+Business+Data+Cloud&zhida_source=entity)** ：统一和管理所有SAP数据，并与第三方数据无缝连接。  
1）将SAP Datasphere、SAP Analytics Cloud和SAP Business Warehouse整合在一起，提供统一的体验。  
2）与Databricks建立合作伙伴关系，集成了Databricks数据智能平台的功能和数据。创建和维护复杂数据管道，为AI Agent和分析工作负载提供了统一的数据基础。



### 2.1.1 Joule的四大特点

1.**深度融入业务流程** ：Joule Agent深度融入业务流程情境，助力解决更复杂的问题。

  * Joule Agent承载了SAP 50年来积累的业务流程专业知识。
  * SAP Knowledge Graph知识图谱对SAP的业务专业知识 进行编码，使Joule Agent能够精准匹配相关业务流程，确 保其能有效执行任务。
  * 通过深度融入流程（流程Grounding）使Joule Agent能 够进一步扩大在企业内的影响力，创造更多价值。 



2\. **利用所有相关数** 据：Joule Agent可以访问所有最相关的数据，以构建全局视野，得出更准确的结论，并基于洞察采取行动。

  * SAP Business Data Cloud提供统一可信数据层，确保Joule Agent可以访问企业中的所有数据（信息优势）。
  * SAP 知识图谱揭示了数据和流程之间的关联，帮助Joule Agent根据业务情境做出决策并采取行动（决策优势）。
  * 这种深度融入使Joule Agent能够洞悉问题的全貌，做出更 精确的判断，并应对更复杂的挑战。 



**3\. 行动力-技能** ：Joule Agent整合了许多预构建的技能，能够跨SAP和第三方应用执行多步骤工作流程（行动优势）。

  * Joule 提供了超过1300种技能库，可自动执行SAP Business Suite和非SAP应用程序中的任务。
  * Joule Agent使用其业务情境来决定使用哪些Joule技能以及按照何种顺序来完成影响更大的多步骤业务目标。 



  


4\. **多智能体协作** ：Joule Agent能够相互协作，通过整合多个职能领域的专业知识，解决更复杂的问题并更高效完成任务。

  * Joule Agent通过齐心协力而非各自为政来思考和采取行动。 
  * Joule开箱即用的Agents库就像一支拥有丰富技能的虚拟员 工团队，能够自主协作，合力应对最棘手的挑战。 
  * 通过Agent之间的协作和人机协同，Joule Agent可以持续 扩展其在企业中的服务范围和影响力。



### 2.1.2 Joule的辅助功能

Joule针对不同场景提供两类功能，通用类和专业类。目前仍是针对当前流程或系统进行体验或效率优化，暂未出现跨度较大的颠覆性创新。

**通用类**

由AI助手提供通用解决方案包，供各产品快速适配落地，面向用户提供普惠性的AI辅助功能。 

  * Conversational search：会话式内容查询，用于系统帮助和咨询。 
  * Direct navigation to applications：应用导航，用于应用。
  * Insights on business objects：业务对象查询与处理。



**专业类**

聚焦业务高耗能点，围绕特定业务采用大小模型等多种技术构建，解决“老大难”问题。

  * **围绕应用规划** ：AI浓度达到一定程度后，可以通过“应用助手”的颗粒度发布，应用可以从已有的“AI特性集”中选取需要的技能进行集成。  
1）**SAP Ariba Category Management – 品类管理** 在品类管理工具中预填充大量相关内容，加快多个品类的战略计划流程，加快用户上手速度。  
2）**SAP SuccessFactors HCM – HR系统管理**  
1\. 管理者完成对员工近期表现的反馈。  
2\. 回答与员工个人资料有关的问题。  
3\. 修改个人信息。  
3）SAP Integrated Business Planning for Supply Chain – 智慧供应链管理 通过将AI能力嵌入作业系统，优化风险抵御能力，并构建全新的智慧供应链体系，以实现最佳预测、管理与交付，确保供应链的可持续性。  

  * **围绕角色规划** ：围绕单一角色规划助手，系统性提升特定角色作业体验和效率。例如，Joule在围绕调度员规划角色助手时：  
1）通过Joule触发SAP Field Service Management中的操作，如触发“最佳匹配技术人员”功能、将活动分配给技术人员等。  
2）为调度员提供唯一会话式入口获取信息和下达指令。



### 2.1.3 Joule提供开发协助

SAP Build Code提供了一个由AI驱动的云开发环境，专门为SAP Cloud Application Programming Model (CAP)、SAP Fiori、移动端和 SAPUI5开发者量身定制。

通过集成Joule， 能够理解SAP开发框架的复杂性，预测开发人员的需求，提供智能建议，并自动执行文档编写和示例数据生成等重复且繁琐的任务。它可以帮助低代码、专业代码和自动化项目的开发人员提高效率、创造力，并更熟练地加速 SAP S/4HANA 等业务应用程序或扩展的开发。

  * **应用程序创建** ：使用Joule跨Java、JavaScript和ABAP的SAP编程模型生成代码、UI、数据模型和示例数据；基于上下文、注释和项目启发式的预测代码完成。
  * **代码优化** ：使用Joule重构代码、创建单元测试以及通过自然语言查询和直观操作生成代码解释、摘要等。
  * **流程和工作流自动化** ：使用自然语言查询生成自动化工作流和业务规则。
  * **代码讲解** ：核心数据服务（CDS）视图实体、类、接口和功能模块的代码讲解。



### 2.1.4 基于AI推荐和执行创建Agent

在SAP Joule Studio中通过Agent Builder自建Agent。创建Agent时，仅步骤1的用户输出目的是必选，其他步骤都是AI（Joule）提供推荐和执行，用户只需要做决策。

### 2.2 SAP Business Data Cloud

2025年2月初，SAP推出了SAP Business Data Cloud (BDC)，这是一个在统一云环境中结合现有和新的SAP Data & Analytics组件的平台。

  * **数据的统一与整合** ：整合企业关键应用中的数据，提供数据工程和业务分析能力。
  * **AI驱动的洞察应用** ：利用数据产品和AI模型连接实时数据，提供跨业务线的先进分析和规划功能。
  * **赋能AI Agent** ：增强AI助手Joule的功能，使其能够更好地支持跨职能工作流程并提升业务决策能力。



  * **工具应用**  
1) **SAP Analytics Cloud** ：SAP分析云提供高级可视化、规划和AI驱动的洞察，帮助业务用户探索数据并执行临时分析；通过AI驱动的建议和基于聊天的探索功能。  
2) **Insight App** ：洞察应用，从数据中获得切实可行的洞察，支持各种行动驱动的场景，包括分析、预测和预报。
  * 1) **数据组件**  
1) **SAP Business Data Cloud cockpit业务数据云控制台** ：是导航SAP业务数据云生态的重要工具，使用户能够浏览和发现预先提供的洞察应用程序和数据产品。  
2) **SAP Datasphere** ：是核心组件，为基于数据产品的数据建模提供基础结构。它持预置的Insight应用和自定义数据模型场景；可通过AI进行扩展；作为管理分析角色和数据访问控制的主要工具。  
3) **SAP Databricks** ：用于通过人工智能、机器学习和Pro-Code工程扩展分析功能。灵活的数据共享机制允许进行丰富的分析。  
4) **SAP BW和SAP BW/4HANA，业务仓库** ：SAP BW 7.5私有云版本和SAP BW/4HANA私有云版本可以集成到SAP业务数据云中。
  * **基础服务**  
数据从源系统复制后，进入基础服务。数据与多个应用程序的其他相关业务数据进行协调、转换和丰富，创建“数据产品”。该数据产品存储在由 SAP HANA Cloud和数据湖文件支持的超大规模环境中。基础服务管理关键数据任务，例如集成、转换和清理。
  * **源系统**  
支持SAP和非SAP源系统。负责捆绑客户业务数据并将其用于分析用例。数据被组织成实体，例如代表特定业务场景的表和视图。
  * **开放生态系统**  
与云服务提供商集成（例如Microsoft Azure、Google Cloud 和 AWS）。



### 2.3 AI Foundation

[SAP AI Foundation](https://zhida.zhihu.com/search?content_id=260329584&content_type=Article&match_order=1&q=SAP+AI+Foundation&zhida_source=entity)是企业实现AI赋能的基础平台，一套即用型AI服务和工具，旨在帮助企业在SAP的平台上构建、部署和管理智能应用。

  * 提供了先进的AI模型、算法和开发环境。
  * 支持自动化、预测分析、自然语言处理等功能。 
  * 帮助企业实现业务流程自动化、优化决策和增强用户体验。



  * **AI服务** ：可立即使用、可重复使用的AI服务。开发人员可以使用这些AI服务将AI功能添加到他们的应用程序中。  
**文档处理** ：使用机器学习来自动化文档中的信息提取过程有助于处理大量包含标题和表格内容的业务文档。  
**个性化推荐服务**  
个性化推荐：根据访客的浏览历史和/或商品描述，提供高度个性化的推荐。  
数据属性推荐：将产品、商店和用户等实体分为多个类别，预测数据中缺失数值属性的值。  
**机器翻译** ：加快软件文本（例如用户界面）和相关文档的翻译速度。
  * **生成式AI管理** ：为了加快生成式AI开发，提供了生成式**AI Hub** ，可访问来自不同提供商的各种大型语言模型 (LLM)，提供用于提示词工程、实验和其他功能的工具。
  * **AI工作负载管理** ：基于**SAP AI Core** 和**AI Launchpad** ，提供了创建、训练和评估新的AI模型，以及发布模型进行推理所需资源。
  * **业务数据和背景** ：使用向量引擎和相似性搜索将AI与业务数据和上下文关联起来。  
向量数据存储以高维向量或嵌入的形式管理非结构化数据，为AI模型提供长期记忆。  
数据管理通过多种方式融入人工智能：数据集成、数据质量和清理、数据治理、分析、自然语言处理。
  * **基础模型** ：是一种深度学习模型，为了在财务、销售或供应链等传统业务领域获得最大价值，需要针对业务环境设计基础模型。 基于匿名数据对通用大型语言模型进行微调，并基于结构化业务数据创建专有的基础模型。



### 2.3.1 AI Launchpad和AI Core

**AI Launchpad**

AI Launchpad和AI Core原生集成。AI Launchpad是基于SAP商业技术平台（SAP BTP）的一款多租户软件即服务（SaaS）应用。客户和合作伙伴可以使用SAP AI Launchpad来管理多个AI运行时（如 SAP AI Core）中的AI用例（场景）。此外，SAP AI Launchpad还通过生成式AI Core提供生成式人工智能功能。

  * **集成AI运行实例** ：添加与一个或多个AI运行时实例的连接，并在它们之间切换以执行进一步的操作。
  * **与SAP AI Core的资源组协作** ：从您的SAP AI Core运行时访问资源组，并仅对其包含的AI资产（例如模型、执行和部署）执行操作
  * **管理生成式AI提示词的生命周期** ：访问生成式AI中心，管理提示词生命周期。生成式AI中心包括提示词试验、提示词生命周期管理和管理工具。 
  * **管理AI用例的生命周期** ：探索并管理所有AI用例的生命周期。生命周期包括训练和部署AI模型以生成在线预测的端点
  * **查看用例的统计信息** ：查看有关您的AI用例（场景）及其使用情况的统计数据。通过分析这些统计信息，您可以更好地估算AI运行时所需的计算资源。



**SAP AI Core**

大规模部署和运行AI模型，支持AI场景的全生命周期管理。 

  * **管理AI场景生命周期** ：通过标准接口来管理不同运行时上的AI场景生命周期，例如模型训练、指标跟踪、数据、模型和模型部署。
  * **多租户隔离** ：隔离不同租户的AI资产和执行。
  * **集成云基础设施** ：通过Docker注册表，将AI内容产品化，将其作为服务提供给SAP BTP市场中的消费者。



### 2.3.2 AI Foundation生态能力建设

SAP携手Google及其他头部企业共同成为全新Agent2Agent (A2A) 协议的创始贡献者。 

  * 来自不同厂商的Agent能够互操作、共享上下文并协同工作，实现跨平台协作，安全地信息交互。
  * 强化了SAP对Joule的愿景，即成为跨企业工作流的Agent Orchestrator，可互操作、主动且与业务情景深度连接。



SAP通过扩展生成式AI Hub对其它公司大模型的访问。以对Google 模型的访问为例，通过生成式AI Hub，客户可以访问Google Gemini 2.0 Flash 和Flash-lite，Gemini Pro 1.0&1.5。

  * 使客户能够灵活地选择模型。 
  * 同时保持在 SAP 安全、业务环境丰富的环境中。



## 3 SAP生态构建

与基础模型、领域模型厂商展开合作，不被厂商锁定并注重数据能力。

## 4 总结

SAP Business AI建立统一的AI解决方案架构，一套AI架构，应用于不同垂直领域产品。同时，通过将AI能力深度植入至其现有关键的业务管理系统中，并利用SAP多年的行业实践、行业数据、行业知识和业务流程，基于已有生态，帮助企业实现业务能力的提升和智能化转型，构筑其商业护城河。

## 相关链接

  1. [https://community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-1-introduction-to-sap-business-ai/ba-p/13792501](https://link.zhihu.com/?target=https%3A//community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-1-introduction-to-sap-business-ai/ba-p/13792501)
  2. [https://community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-2-components-of-sap-business-ai/ba-p/13792752](https://link.zhihu.com/?target=https%3A//community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-2-components-of-sap-business-ai/ba-p/13792752)
  3. [https://community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-3-joule-sap-s-generative-ai-copilot/ba-p/13581304](https://link.zhihu.com/?target=https%3A//community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-3-joule-sap-s-generative-ai-copilot/ba-p/13581304)
  4. [https://community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-4-ai-foundation-on-sap-btp/ba-p/13806642](https://link.zhihu.com/?target=https%3A//community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-4-ai-foundation-on-sap-btp/ba-p/13806642)
  5. [https://community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-5-introduction-to-sap-ai-core/ba-p/13860233](https://link.zhihu.com/?target=https%3A//community.sap.com/t5/technology-blog-posts-by-sap/generative-ai-with-sap-part-5-introduction-to-sap-ai-core/ba-p/13860233)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【大模型应用】01-企业级Agent平台AgentForce](https://zhuanlan.zhihu.com/p/1923462173356691937)
