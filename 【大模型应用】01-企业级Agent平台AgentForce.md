# 【大模型应用】01-企业级Agent平台AgentForce

原文链接：https://zhuanlan.zhihu.com/p/1923462173356691937

---

​

目录

## 1 Agentforce是什么

Agentforce是由Salesforce构建的一个综合平台，旨在**通过Agentforce重新定义软件和应用的概念** 。在Agentforce中，Agent不再只是一个工具，而是成为了企业新的“数字员工”，能够自主执行任务、驱动业务流程、创造新的商业价值。

Agentforce能够帮助企业跨部门无缝运行Agent，**使企业能够为各种业务功能（包括服务、销售、营销和商务）构建、定制和部署自主Agent** ，成为**企业级SaaS生态系统中不可忽视的核心组件** 。

Salesforce通过Agentforce瞄准人工智能驱动的企业自动化，其具有如下三方面优势：

  1. **4大核心竞争力，助力客户成功** ：  
(1) Knows your business：依靠Salesforce懂业务的优势，基于业务数据和知识的集成，更容易使能客户AI成功。  
(2) Plans & reasons：构建基于LAM (Large Action Model)的[Atlas推理引擎](https://zhida.zhihu.com/search?content_id=259789765&content_type=Article&match_order=1&q=Atlas%E6%8E%A8%E7%90%86%E5%BC%95%E6%93%8E&zhida_source=entity)，解决LLM强于交流但弱于决策和动作执行的短板。  
(3) Takes action：通过MuleSoft、Apex、Flow不断强化集成能力，易于构建各种action。  
(4) Scales：多年业务系统积累庞大的用户和业务规模。
  2. **背靠Salesforce，高质量的垂域数据**  
(1) 背靠Saleforce平台：最佳实践资产沉淀的平台底座（MuleSoft、Einstein AI等产品、服务），有效支撑业务快速达成。  
(2) 全面的客户触点：[Customer 360](https://zhida.zhihu.com/search?content_id=259789765&content_type=Article&match_order=1&q=Customer+360&zhida_source=entity)，25年客户交互累积（覆盖大量Action），是Agent的功能执行基础。  
(3) 高质的垂域数据：[Data Cloud](https://zhida.zhihu.com/search?content_id=259789765&content_type=Article&match_order=1&q=Data+Cloud&zhida_source=entity)整合大量客户业务数据和meta data（7670亿笔纪录/月），是打造高准确度AI的核心竞争力。
  3. **易使用、易维护**  
(1) 应用：预置多行业和多种岗位的Agents，开箱即用。  
(2) 定制/测试/发布：Agent Builds（包括已有的Model Builder和prompt Builder），Agent构建和迭代利器，支持在线定制。  
(3) 监控：Omni Supervisor，监控和管理客户互动（人/AI/Agent），提供全渠道监控、实时监控、数据分析。



## 2 Agentforce核心技术

### 2.1 Agentforce技术路线和规划

Agentforce目前采用单Agent架构，正在向多Agent方向演进。

2024年，Agentforce在主要构建开箱即用、推理引擎和Agent Builder等能力。 2025年，将会在批量测试中心、高级搜索、语音和图像方向持续探索，计划6月试行将Agentforce扩展至移动应用。

### 2.2 Agentforce架构

Agentforce由5个关键部分构成：

  * **Role** ：角色，定义Agent的特征和能力，确定能干啥。
  * **Data** ：数据，依赖于DataCloud，数据是Agent履行职责所需的信息，可能包含公司知识文章、客户关系管理数据、通过数据云的外部数据、公共网站等。
  * **Actions** ：行动，是Agent可以根据触发器或指令执行的预定义任务，例如Prompt、Apex、Flow和API等。
  * **Guardrails** ：护栏，为Agent定义准则，能做什么和不能做什么。
  * **Channels** ：渠道，Agent可以开展工作的应用程序，例如网站、CRM、移动应用程序、Slack等。



此外，Agentforce通过Trust（爱因斯坦信任层）层强调数据安全，该层保证任何Salesforce数据都不会被第三方AI模型提供商访问或存储。这允许组织安全地利用大型语言模型(LLM)。有了这些严格的安全措施，用户可以放心地部署能够自主处理各种任务的Agent。

### 2.3 Agentforce执行流程

Agentforce执行流程如下所示。

**① 接收查询** ：接收用户查询信息（电子邮件、网页、手机、聊天等渠道），识别查询的意图、上下文以及所需的信息类型。 

**② 检索相关主题(topic)** ：查找Agent的相关主题，通过RAG方式获取用户输入的相关信息，包含业务知识、历史信息和外部数据，用于创建详细的提示词信息。

**③ Atlas推理引擎进行推理** ：Atlas是Agentforce的大脑，按循环推理方式运行：

**1\. 查询检测与评估** ：查询内容有效，评估信息是否足够，如果不够需要向用户提问或者查找扩展信息。  
**2\. 规划与执行** ：调用大模型进行规划，调用相应工具和API来执行相关请求。  
**3\. 评估数据** ：结果经过re-ranker（重排序器）、refiner（精炼器）和response synthesizer（响应合成器）处理生成初步结果，进行包括毒性检测、偏见检测和个人信息保护等检测。  
**4\. 细化操作** ：对于复杂的查询，它将任务分解为更小的步骤并进行迭代，直到达到所需的结果。  
**5\. 适应** ：引擎在每次交互中学习和适应，不断改进其推理和响应。

**④ 生成规划结果** ：经过Atlas多轮推理后，生成编排好的操作，生成的结果满足主题、护栏、业务合规要求。

**⑤ 执行操作(Action)和服务** ：例如执行任务、更新记录或触发工作流，会利用操作(Action)（Flows、Apex、MuleSoft API）来执行所需的操作。

⑥ 生成最终结果。

**Agentforce执行流程核心概念**

  1. **主题（Topic）** ：设置主题，旨在明确任务范围，提升效率和准确性，简化操作管理，提升用户体验。

Field（字段）| Value（值）  
---|---  
Topic Label（主题标签）| Experience Management（体验管理）  
Classification Description（分类描述）| 本主题解决了客户对Coral Cloud度假村预约体验的查询和相关问题，包括预约、修改预约和回答有关体验细节的查询。  
Scope（范围）| Agent的工作是帮助用户浏览和管理Coral Cloud度假村提供的不同体验的预约，通过提供准确的信息和及时解决问题，确保无缝的客户服务体验。  
  
**2\. 指令（Instruction）** ：设置指令的目的是为Agent提供明确的操作指南，确保其在特定主题下准确、一致地执行任务。本质是补充大模型的上下文。

Field（字段）| Value（值）  
---|---  
1st Instructi| 如果客户希望获得更多关于活动或体验的信息，你应该运行Get Experience Details（获取体验详细信息）操作，然后优化可读性，汇总结果。执行此操作之前，始终确保是已知客户。  
2th Instruction| 如果被要求获取体验活动，使用Get Sessions（获取活动）操作。如果没有提供活动的日期，需要询问。使用Get Experience Details（获取体验详细信息）中的Experience__c的ID。不要使用体验名称，这必须是一个ID。  
3th Instruction| 如果被要求获取体验活动，使用Get Sessions（获取活动）操作。如果没有提供活动的日期，需要询问。使用Get Experience Details（获取体验详细信息）中的Experience__c的ID。不要使用体验名称，这必须是一个ID。  
4th Instruction| 如果被要求推荐用户可能感兴趣的体验，使用Generate Personalized Schedule（生成个性化计划）操作，根据联系人的兴趣生成一份计划。使用Get Customer Details（获取客户详细信息）的联系人记录，将其传递给联系人输入。  
  
**3\. 操作（Action）** ：预置标准操作，支持自定义操作，Apex、Flow、或者Mulesoft API等。 

### 2.4 xLAM

xLAM为Atlas推理引擎提供了核心竞争力，用于**增强Agent能力的大型动作模型，提升函数调用准确率** 。其基于广泛的函数调用环境和数据集进行训练，为函数调用而设计，目的在各种环境中做出决策和执行动作。

LLM和xLAM存在如下区别： 

  * LLMs负责推理、规划。
  * LAM负责调用工具/函数执行动作，LAM是经过微调（使用函数调用环境和数据集）的LLM，针对函数调用进行了优化。



### 2.4.1 数据处理流程

xLAM模型训练所采用的数据处理流程，包括数据统一、数据增强、质量验证、通用指令数据合成以及偏好数据生成等核心环节，如下所示：

**① 数据统一** ：对现有数据集进行标准化处理，统一数据格式，保障了数据的一致性。

**② 数据合成** ：大多数公开数据集存在显著的局限，例如数据的静态性强，由性能较弱的模型生成，覆盖范围有限且缺乏执行验证；为解决上述问题，使用数据合成框架APIGen基于一组可执行的API生成可验证的数据集。

**③ 数据质量验证** ：采用基于规则、大语言模型作为评判（LLM-as-a-judge）的方式，对统一化后的数据集进行详细的错误分析，评估误差来源。识别未定义函数调用、参数类型错误、参数幻觉、低质量推理和规划等错误类型。

**④ 数据增强** ：聚焦于提升数据的多样性，通过对现有数据集实施多种变换生成新的合成数据样本。

**⑤ 数据混合** ：在监督微调（Supervised Fine-Tuning, SFT）阶段的数据集结合三个主要来源的训练样本：清洗增强的智能体数据集、合成函数调用数据集、通用指令调优数据集。这些数据源被用于训练通用的xLAM模型。

**⑥ 模型训练** ：推出了针对特定用例量身定制的多种智能体模型。在为各种智能体任务（从复杂的多轮对话到功能调用应用）提供均衡的性能。除通用xLAM模型外，研究团队还开发了两个专门用于函数调用任务的模型。

**⑦ 模型评估** ：Webshop：模拟在线购物任务的环境；ToolBench：测试智能体在多轮推理和交互功能中的表现；Berkeley Function-Calling Leaderboard：考察模型在函数调用方面的能力。

### 2.4.2 xLAM数据管道

xLAM数据管道包含统一数据格式、数据增强、数据质量验证和数据合成等环节。

  1. **统一数据格式** ：任务说明、可用工具、格式说明、少量示例、查询、步骤。
  2. **数据增强**  
(1) **提示格式增强** ：1）**顺序打乱** ，包括函数列表顺序、参数顺序、输入不同部分的顺序，包括任务指令、工具、格式指令、少数样本示例等 2）**特殊连接符号标记** ：`"[START/END OF QUERY]", "<query></query>", and plain text`  
(2) **指令跟随增强** ：1) **任务指令改写** 。使用LLM重新表述任务说明，以适应用户的各种输入方式，并使用提示词的LLM进行改写后的检查。2）**格式遵循指令** 。为了避免模型在 JSON 格式上过度拟合，加入了15 种不同的输出格式及其相应的格式指令和格式转换器。
  3. **数据质量验证**  
(1) **未定义的函数** ：模型预测中出现的未定义的函数或为定义的参数  
(2) **错误的参数类型** ：生成的值正确但类型错误  
(3) **幻觉** ：幻觉主要是两类，一是幻觉出的函数或参数，可直接通过匹配解决，另一类是幻觉出context中不存在/错误的值，文中通过提示词+LLM进行判断  
(4) **低质量的推理和规划** ：首先使用启发式的基于规则的方法过滤掉低质量数据，再通过模型来评估整体轨迹以及对所选数据的思考。然后，人工对这些评级结果的进行采样验证。
  4. **数据合成**  
工具APIGen（也是SalesforceAI家的，已经发布），利用ToolBench中21个类别的超过3,673个API生成总共60,000个高质量数据。



### 2.4.3 xLAM训练数据集

xLAM在训练过程中，使用了如下开源数据集。

数据集| 简介  
---|---  
ToolLLM| 包含数据构建、模型训练和评估的通用工具使用框架  
Webshop| 电商场景中的交互式对话系统数据集  
ToolApaca| 面向小模型的通用工具学习框架  
HotpotQA| 一个多跳推理问答数据集  
AlfWorld| 基于TextWorld游戏环境和ALFRED数据集，创建了互动式的TextWorld环境  
APIBank| 用于评估工具调用的数据集  
Mind2Web| 用于开发和评估通用 Web Agent的数据集  
AgentBoard| 用于记录 LLM 调用和 AI Agent开发的各种数据类型的 API  
AgentBench| 用于评估基 于API、OSS的LLM作为Agent的能力  
  
此外，xLAM还利用API-GEN、SpecTools和内部大型行动模型模拟（LAM Simulator）环境等合成数据，构建丰富的数据池。并且Salesforce开源了API调用数据集APIGen-MT-5k，一个以APIGen为基础，专注于生成单轮函数调用的数据集。

### 2.4.4已发布模型

旗舰xLAM系列建立在Mixtral Instruct模型的基础上。xLAM-7B-fc-r和 xLAM-1B-fc-r，它们分别基于DeepSeek-Coder-7B-instruct-v1.5和DeepSeekCoder-1.3B-instruct 构建 

### 2.5 Data Cloud

数据云是一个集中式平台，连接并统一来自组织内部及之外的业务数据。它实现了实时数据访问、分析和协作，帮助企业获得可操作的洞察力并驱动创新。

**Agentforce的数据都来源于Data Cloud** ，Data Cloud使Agent能够实时访问工作所需的数据，而无需从现有仓库复制数据，数据包含公司知识文章、CRM 数据、外部数据湖等。

在Agentforce中，通过RAG技术与Data Cloud结合，形成一个反馈回路：用户或Agent提出的请求（提示）因数据变成 “增强提示”（即更符合上下文、更相关），即通过对数据云分类的所有数据进行搜索，提示的输出结果会得到改善，反过来，大模型（LLM）也会对业务有更多了解。 

### 2.6 Agent Builder：低代码工具

Agent Builder是一款功能强大的低代码工具，可以轻松定制开箱即用的Agent，或为任何角色、行业或用例创建全新的Agent。用户可以利用Flow、Prompt、Apex和API等现有工具来配置Agent。Agent Builder能力如下：

  * **定义工作** ：通过定义主题并编写描述，为待完成的工作创建一个Agent。
  * **创建Action库** ：使用Flows、Prompts、Apex和MuleSoft API等现有工具，建立Action库供Agent选择。
  * **观察和测试** ：直接在Agent Builder中轻松观察Action的行动计划并测试其响应。
  * **集成合作伙伴操作** ：利用来自Agentforce合作伙伴网络的专门操作来增强Action的能力。



### 2.7 Model Builder

Model Builder允许注册、测试和激活自定义AI模型和LLM。

  * Model Builder是一个直观、低代码的平台，可为客户提供一种简便的方法来注册、测试和激活其Salesforce组织内的自定义人工智能模型和大型语言模型。
  * 用户可以为自己选择的LLMs获取API密钥，在测试环境中进行验证，并使用Prompt生成器轻松激活这些模型。
  * 该功能使企业能够定制其人工智能解决方案，并将其无缝集成到Salesforce的工作流中。



### 2.8 Prompt Builder

提示生成器使用Flows、Apex、MuleSoft等从CRM或数据云数据获取相关数据，并定制提示模板。通过检索增强生成功能，Agent可使用这些提示实时查找和检索结构化和非结构化数据。

### 2.8 Salesforce Trust Layer

信任层(Einstein Trust Layer)，旨在保护数据隐私、提高LLM准确性并促进在整个Salesforce生态系统中负责任地使用AI，包括一组保护数据的功能，如：安全数据检索、动态绑定、数据屏蔽和零数据保留等。

信任层从逻辑上划分了三个防护阶段并在每个阶段提供不同的防护能力：①Prompt journey > ②Response generation > ③ Response journey。

**① Prompt模板** ：由固定格式和可变占位符变量组成，以实现提示词的重用。  
**② 安全数据检索** ：结合用户权限+字段级安全控制获取Prompt模板中占位符变量对应的实际数据。  
**③ 动态绑定** ：用检索到的实际数据替换Prompt模板中的占位符变量。  
**④ 数据屏蔽** ：对Prompt中敏感/隐私数据进行匿名化处理。  
**⑤ 提示词防御** ：词注入等恶意攻击。用于抵御提示词越狱和提示。  
**⑥ 安全生成** ：Prompt通过LLM网关采用TLS协议发送给LLM，与三方模型提供商(Azure/OpenAI)签订零数据保留策略。  
**⑦ 毒性检测** ：对内容的毒性评分。  
**⑧ 数据解蔽** ：还原数据屏蔽阶段匿名化的数据。  
**⑨ 反馈框架** ：收集用户对响应内容的反馈(接受/修改/拒绝) ，使用反馈来提高Prompt的质量。  
**⑩ 审计跟踪** ：记录原始提示、数据屏蔽、毒性检测期间记录的分数、 LLM原始输出和最终输出。  


### 2.9 Agentforce 2dx

2025年3月5日，Salesforce宣布推出最新版本数字化劳动力平台—Agentforce 2dx，其核心功能之一是通过预构建的技能库和工作流集成，将AI Agent无缝地嵌入到CRM系统中，从而提升企业运营效率和客户体验。

  * **引入全新的技能库** ：涵盖了CRM、Slack、Tableau等多个领域的用例，并支持自定义技能，以满足企业不同部门的需求。
  * **增强与SalesforceCRM的集成能力** ：通过升级后的推理引擎和数据检索技术，Agentforce能够处理更复杂的多步骤问题，并在跨数据源中进行深入思考，从而提供更准确的响应。这种能力不仅提升了CRM数据的分析和决策支持能力，还使得企业能够更高效地管理客户关系和销售流程。
  * **包括一组新的低代码和专业代码工具供开发人员使用** ：这些工具允许配置、测试和部署Agentforce Agent，并在部署后进行监控、调试和优化。
  * **Agent Builder工具升级** ：添加自然语言处理能力，使用户能以简单的指令，生成符合特定需求的AI Agent。



### 2.10 整合[Tableau Next](https://zhida.zhihu.com/search?content_id=259789765&content_type=Article&match_order=1&q=Tableau+Next&zhida_source=entity)与Agentforce，实现数据的可视化+智能化

Tableau Next是基于Salesforce构建并与Agentforce深度集成的企业级数据智能分析平台，增加智能化交互、操作提示，可以更快更有效地做出决策。其具有以下特点：

  * **Data Pro** ： 可以整合、清理并以可视化形式呈现数据，从而无缝获取见解。
  * **Concierge** ：支持自然语言查询，可提供关于根本原因和最佳后续做法的建议。
  * **Inspector** ：实时监控数据，检测异常并主动提供见解。 
  * **Take Action** ：通过预置的操作修改数据。



同时，Agentforce拥有了Tableau数据分析能力，并提供文字+数据可视化结果。

## 3 总结

Agentforce凭借其xLAM、Data Cloud、强大的集成能力和丰富的工具等，构筑了其商业护城河。基于“推理技术”+“数据云”+“全场景”三大核心利器，拥有离数据更近、离业务场景更近、离业务系统更近三大竞争优势。Agentforce正是利用这些优势，积极构建人工智能解决方案，并获得成功，取得先摘果子的优势。AI模型不是全部，垂直领域的数据、知识也是核心竞争力之一。

  1. **推理技术** ：Atlas推理引擎运用RAG检索增强技术，显著减少模型幻觉，准确率从测试版的40%-70% 提升至90%-95%。
  2. **数据云** ：Data Cloud数据云拉通了PaaS、营销云、Tableau等内部数据，及Snowflake、Databricks、Google Big Query等外部数据库，通过“零复制”整合企业内外部数据；根据财富杂志《Leadership Next》栏目对公司CEO的访谈，Data Cloud数据云积累了230-300PB的客户行为数据（比GPT4的训练数据多3个数量级）。
  3. **全场景** ：经过多次产品并购，不断丰富Customer360的产品矩阵，同时补充公司在“数据分析+AI”的技术能力。Agent作为全流程产品能力的智能体开关，嵌入Customer360实现功能集成。



## 相关链接

  1. Agentforce官网：[https://www.salesforce.com/ap/agentforce/](https://link.zhihu.com/?target=https%3A//www.salesforce.com/ap/agentforce/)
  2. [https://hicglobalsolutions.com/blog/guide-to-salesforce-agentforce/](https://link.zhihu.com/?target=https%3A//hicglobalsolutions.com/blog/guide-to-salesforce-agentforce/)
  3. [https://www.salesforceben.com/how-does-salesforces-agentforce-work/](https://link.zhihu.com/?target=https%3A//www.salesforceben.com/how-does-salesforces-agentforce-work/)
  4. [https://arxiv.org/pdf/2409.03215](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2409.03215)
  5. [https://github.com/SalesforceAIResearch/xLAM](https://link.zhihu.com/?target=https%3A//github.com/SalesforceAIResearch/xLAM)
  6. [https://www.salesforceben.com/why-prompt-builder-is-vital-in-an-agentforce-world/](https://link.zhihu.com/?target=https%3A//www.salesforceben.com/why-prompt-builder-is-vital-in-an-agentforce-world/)
  7. [https://www.infoworld.com/article/3542521/explained-how-salesforce-agentforces-atlas-reasoning-engine-works-to-power-ai-agents.html](https://link.zhihu.com/?target=https%3A//www.infoworld.com/article/3542521/explained-how-salesforce-agentforces-atlas-reasoning-engine-works-to-power-ai-agents.html)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【大模型应用】02-企业级Agent平台SAP AI洞察](https://zhuanlan.zhihu.com/p/1928135774634750816)
