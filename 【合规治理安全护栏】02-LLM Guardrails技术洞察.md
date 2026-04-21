# 【合规治理/安全护栏】02-LLM Guardrails技术洞察

原文链接：https://zhuanlan.zhihu.com/p/1949495420163199375

---

​

目录

## 1 概述

随着[大语言模型](https://zhida.zhihu.com/search?content_id=262938597&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）技术的快速演进，其优秀的能力使得大模型应用，尤其是LLM智能体更是井喷而出。越来越多的企业开始依赖LLM应用来提升生产力、自动化流程和优化用户体验。无论是用于客户服务、内容生成，还是数据分析，LLM应用都展现出了巨大的潜力，成为推动数字化转型的关键力量。然而，LLM的强大能力也带来了许多不可忽视的风险。

当前，幻觉、安全和隐私是影响LLM应用商用落地的最大风险点。幻觉是指大模型生成虚假、逻辑错误或无事实依据的信息，根因是大模型基于概率预测而非真实理解生成内容。这种缺陷是企业落地大模型应用的最大担忧，尤其是金融、医疗等严谨性领域，例如错误药方剂量或虚构市场数据可能导致致命后果或重大经济损失。LLM的安全、隐私泄露等风险，包括训练数据泄露、用户提示词暴露等安全隐患，导致不满足法律法规要求。

图1 2024年Gartner调研报告显示，尽管生成式AI具有巨大潜力，但幻觉、误导性输出、安全和隐私泄漏是企业落地大模型应用TOP3风险

那么，如何确保在广泛应用LLM的同时，能够有效管理其输出，避免生成有害、误导性或不准确的信息？如何防止模型滥用、保护用户隐私，并确保模型输出符合道德、法律和行业标准？

为了解决这些挑战，[LLM Guardrails](https://zhida.zhihu.com/search?content_id=262938597&content_type=Article&match_order=1&q=LLM+Guardrails&zhida_source=entity)至关重要。**LLM Guardrails，作为用户与LLM之间的关键安全层，为解决大模型应用输出的不可控风险（如幻觉、有害内容、隐私泄露等）​，​提供多种防护机制（输入过滤、生成校验、输出审核等）和结构化约束（如格式校验、合规检查）​，实现大模型应用满足可靠(与事实相符)、安全、合规要求** 。

图2 LLM Guardrails常见分类

## 2 AWS Bedrock Guardrails

[Amazon Bedrock Guardrails](https://zhida.zhihu.com/search?content_id=262938597&content_type=Article&match_order=1&q=Amazon+Bedrock+Guardrails&zhida_source=entity)是AWS提供的一组可配置的安全策略，允许用户根据特定的使用场景和责任AI政策，定制化地保护其生成式AI应用程序。这些策略可以应用于Amazon Bedrock支持的多种基础模型，包括微调模型和自托管模型，确保在生成内容时符合安全和隐私标准。根据相关信息，其最多可阻隔85%的不良和有害内容，从用于检索增强生成（RAG）和汇总使用案例的模型中过滤75%以上的幻觉响应。Amazon Bedrock Guardrails主要包含如下能力：

  * **内容过滤** ：根据阈值(NONE/Low/Medium/Hight)过滤包含有害或不需要内容的输入提示或模型响应，支持维度：仇恨/侮辱/性/暴力/不当行为/提示词攻击。 
  * **主题过滤** ：过滤包含预定义主题(如经济/政治)的输入提示或模型响应。 
  * **单词过滤** ：过滤包含预定义单词、短语和亵渎性语言的的输入提示或模型响应。内置亵渎过滤器开关，支持自定义过滤单词列表。 
  * **敏感信息过滤** ：过滤包含预定义敏感性信息(如个人身份信息PII)的输入提示或模型响应。支持PII和正则表达式两种类型。 
  * **上下文语境检查** ：根据配置的可靠性和相关性阈值过滤模型响应中包含不基于企业数据或与用户查询无关的虚构内容。 
  * **自动推理检查** ：利用自动推理技术帮助防止幻觉造成的事实错误。



对于在AWS Bedrock上托管的模型，Amazon Bedrock Guardrails会采取主动的防护机制，自动对模型的输入输出进行防护；同时，对于未在AWS Bedrock托管的模型，其支持通过显式调用ApplyGuardrail API为应用提供防护机制。其整体的应用流程如下图所示：

图3 AWS Bedrock Guardrails应用流程

如图3所示，全面的大模型应用防护机制由Prompting protection + LLM Guardrails(Input guardrails + Output guardrails) + LLM built-in guardrails（模型内置护栏）组成。AWS Bedrock Guardrails自动对AWS Bedrock托管的大模型提供输入输出防护机制（即Input guardrails + Output guardrails ）。

### 2.1 关键能力——自动推理检查

自动推理检查(Automated Reasoning Checks)基于自动推理技术，使用数学可验证的确定性逻辑来验证、纠正和解释大模型输出结果，确保大模型输出结果与事实的一致性。自动推理检测是业界唯一一个采用自动推理技术的生成式AI防护措施。AWS的自动推理技术的应用场景包含：

  * **CodeGuru Reviewer** ：帮助开发人员识别软件BUG。
  * **Inspector Classic** ：自动分析EC2实例是否存在潜在配置错误。
  * **S3** ：自动分析ACL访问策略是否存在潜在的配置错误。
  * **VPC Reachability Analyzer** ：帮助解和可视化AWS网络。



**自动推理检查方案**

自动推理检查方案如图所示：

  1. 开发者将源文档(如承保规则指南)与意图示例上传到自动推理检查服务以创建自动推理策略。
  2. 自动推理检查服务识别用户文档中的重要概念，生成变量信息(变量名/数据类型/描述)。
  3. 自动推理检查服务以决策树形式构建操作变量的形式化逻辑规则，为方便不具备形式逻辑专业知识的用户优化，规则会转换为自然语言描述，供开发者调整和修改。
  4. 自动推理检查服务保存由变量与规则等信息组成的自动推理策略。
  5. 大模型应用调用LLM服务处理用户Query。
  6. 大模型应用调用ApplyGuardrail API接口核查大模型输出结果，请求包含原始用户Query以及LLM服务的输出信息。
  7. 自动推理检查服务将Query和LLM输出信息转换为逻辑表示，根据自动推理库中的策略完成自动推理，返回检查结果（使用的规则、变量和变量值、以及使断言有效的建议）给大模型应用。



**自动推理检查使用流程**

（1）创建策略，自动推理检查服务根据用户上传的文档自动生成变量和规则信息，变量和规则信息的准确性影响后续自动推理检查结果，需要开发者确认，支持手工编辑修改。

（2）调试策略，目的是通过测试用例验证策略中变量和规则的准确性，如果验证结果不符预期，可通过手工编辑规则和变量来优化。自动推理检查根据自动推理策略验证测试用例中的问题和答案，能够识别任何与事实不符或不准确的内容，并为验证结果提供解释：

  * 答案无效：显示导致该结果的规则原因，以及提取的变量和建议，建议会显示一组使结论变为有效的变量赋值。
  * 答案有效：建议会显示一系列结果成立所必需的赋值，这些是答案中未说明的假设。



（3）启用策略，创建一个新的防护机制并为其命名。选择“启用自动推理策略”，并选择要使用的策略及策略版本，然后完成防护机制配置。

（4）运行时推理，系统根据策略进行推理生成防护措施。

## 3 [NVIDIA NeMo Guardrails](https://zhida.zhihu.com/search?content_id=262938597&content_type=Article&match_order=1&q=NVIDIA+NeMo+Guardrails&zhida_source=entity)

NVIDIA NeMo Guardrails是英伟达开源的Guardrails框架，旨在为大型语言模型（LLM）驱动的对话系统添加可编程的安全护栏。NVIDIA NeMo Guardrails支持开发者使用Colang语言定义用户与大模型交互时应遵循的护栏，为应用提供幻觉、安全隐私防护。其核心价值是定义了一套用户与大模型交互流程、可扩展的护栏防护机制，预置了多种护栏实现（依赖于模型或者三方护栏API ）。

NVIDIA NeMo Guardrails支持作为独立的Python库集成到Python程序中，其提供了LangChain扩展包，可被LangChain应用集成。同时，其可以作为一个独立的REST服务使用。

通过可编程防护栏实现防护的流程如图所示。 

  * **Input rails** ：输入护栏，对于非预期输入可以进行拒绝、输入改写（例如敏感信息匿名化）。
  * Dialog rails：对话护栏，控制整个对话流程，用户和机器人之间严格按照预定义的流程进行对话（严格意义上讲，相比输入/输出等护栏，对话护栏不能算是一种护栏）。
  * **Retrieval rails** ：检索护栏，应用与RAG场景，检查粒度为检索结果的每个chunk，检测粒度是检索结果的一个chunk，对于非预期chunk可以进行拒绝、改写（例如敏感信息匿名化） 。
  * **Execution rails** ：执行护栏，用于执行Action（严格意义上讲，执行护栏不能算是一种护栏）。
  * **Output rails** ：输出护栏，对于非预期输出可以进行拒绝输出、改写（如删除敏感信息）。



### 3.1 Colang

Colang是一种基于事件的建模语言，专为人类和机器人间高度灵活对话而设计，语法为自然语言和Python语法的混合体，由Python运行时解释。其架构如下图所示：

  * Colang Interpreter根据Colang脚本中定义的规则检测和生成事件序列、处理事件。借助大型语言模型和检索增强生成服务，基于Colang能够实现用于处理用户和系统实时交互的应用程序。
  * Sensor Servers负责从用户输入数据中提取相关事件并将其转发到Interaction Manager。
  * Action Servers负责处理来自②Interaction Manager的事件以生成输出数据。事件通常具有较小的有效负载，而数据流可以携带更多数据。



Colang核心语法包括：blocks、statements、expressions、keywords和variables。blocks有三种类型：用户消息define user …、助手消息define bot …、工作流define flow …。Colang脚本是一个.co文件，由一个或多个Flow定义组成，Colang入口脚本固定为[http://main.co](https://link.zhihu.com/?target=http%3A//main.co)。

### 3.2 NeMo Guardrails处理用户请求流程

NeMo Guardrails处理用户请求流程如下所示：

  1. **生成用户意图** ：接收到用户输入的话术后，使用LLM生成意图，例如把用户话术Hi或者Hello转换成意图express greet。调用LLM时使用的Prompt可配置，其中包含一组将用户话术转化为意图的示例（通过对.co脚本中定义的所有用户消息示例进行向量搜索，选择最相关的五个作为示例）。
  2. **确定下一步行为** ：
     1. 如果.co中存在与用户意图匹配的Flow，则使用该Flow(Flow决定机器人应以什么消息进行响应，或执行某个Action)；
     2. 如果不存在匹配的Flow，则通过LLM生成一个Flow（在.co脚本中使用向量搜索，将与用户意图最相关的前5个Flow放到提示词中，由LLM决定使用哪个Flow；如果搜索不到相关Flow，则使用系统预置Flow：ask general question）。
  3. **执行Flow** ：执行Colang实现的Flow代码。如果涉及执行Action，如execute self_check_input，则走④；如果涉及机器人回复信息，如bot refuse to respond about slander，则走⑤。
  4. **执行Action** ：Aciton是用Python实现的一段业务处理逻辑，例如使用LangChain来调用一个chain或者工具。
  5. **生成机器人消息** ：
     1. 如果配置了本地知识库，则搜索最相关的文本块，并将检索结果放到提示词中。
     2. 如果.co中存在指定的机器人输出意图，如define bot refuse to respond about slander，则从输出意图的输出话术列表中随机找一个返回。
     3. 如果.co中不存在指定的机器人输出意图，则调用LLM生成输出话术（在.co脚本中使用向量搜索，将前5个与输出意图最相关的意图放到提示词中，由LLM决定用哪个；如果搜索不到相关输出意图，则使用系统预置输出意图：bot response for general question）。



NeMo Guardrails处理用户输入过程经历三个阶段5种护栏。

  * **阶段一：输入验证阶段**  
① 输入护栏(Input Rails)：用户输入首先由输入护栏处理，由输入护栏决定是否允许输入、是否改写输入。
  * **阶段二：对话阶段**  
② 对话护栏：整个对话阶段就是对话护栏，对话护栏中会调用③检索护栏和④执行护栏。（所以严格上说对话护栏不是护栏）  
③ 检索护栏：RAG场景，检索护栏检查每个检索结果chunk，由检索护栏决定是否允许使用该chunk作为模型输入，是否改写chunk（例如屏蔽敏感数据）。  
④ 执行护栏：执行Action，Action可以包含任何业务处理，例如调用工具。（所以严格上说执行护栏不是护栏）
  * **阶段三：输出验证阶段**  
⑤ 输出护栏(Output Rails)：大模型生成回复消息后由输出边栏处理，由输出护栏决定是否允许输出，是否改写输出。



## 4 Salesforce

Salesforce依托可信任的AI云架构为企业提供安全、合规且高效的生成式AI体验。该架构通过多层防护机制，确保客户数据的隐私和安全，同时提升AI应用的可靠性和透明度，其核心组件是Einstein Trust Layer。其架构如下图所示：

  * **Hyperforce** ：基础设施层，通过三A部署（Authentication, Authorization, Audit）和零信任架构实现安全性。
  * **Data Cloud** ：数据云层，Einstein平台数据底座，为Copilot访问CRM数据和外部数据提供服务。
  * **Open Models** ：统一的大模型托管平台，支持Salesforce自研模型、三方模型、用户自有模型。
  * **Trust Layer** ：信任层，保护数据隐私、提高LLM准确性并促进在整个Salesforce生态系统中负责任地使用AI。
  * **Builders** ：构建层，提供构建提示词、Copilot、Action等所需的工具/服务。



### 4.1 Einstein Trust Layer

Einstein Trust Layer是Salesforce核心，旨在保护数据隐私和安全，促进Salesforce生态系统能负责任地使用AI。其提供一组安全隐私保护功能，如安全数据检索、动态绑定、数据屏蔽和零数据保留等，但不支持幻觉防护。Einstein Trust Layer从逻辑上划分了三个防护阶段，并在每个阶段提供不同的防护能力：①Prompt journey > ②Response generation > ③ Response journey。

① **Prompt模板** ：由固定格式和可变占位符变量组成，以实现提示词的重用。为提供便利性和一致性， Salesforce预置各种业务用例的Prompt模板，例如销售电子邮件或客户服务回复。通过{!XXX}的方式定义占位符变量，运行态用实际的CRM数据替换。

② **安全数据检索** ：结合用户权限+字段级安全控制获取Prompt模板中占位符变量对应的实际数据，以便为Prompt提供输入数据。

③ **动态绑定** ：用检索到的实际数据替换Prompt模板中的占位符变量。如下所示，通过语义检索从其他数据源（如 Knowledge文章和客户历史记录）获取相关信息替换Prompt模板中的占位符变量。

④ **数据屏蔽** ：对Prompt中敏感/隐私数据进行匿名化处理。，如个人身份信息 (PII) 和支付卡行业 (PCI) 数据等，避免敏感数据暴露给LLM。敏感信息条目包括公司名称/信用卡/邮箱/IBAN码/姓名/护照/电话号码/美国驾照/个人纳税识别号码/社会保障号。支持选择需要屏蔽的条目。

⑤ **提示词防御** ：在Prompt中额外增加的系统指令，指导LLM在特定情况下如何行事，降低输出意外或有害内容的可能性，从而抵御提示词注入等攻击。

⑥ **安全生成** ：提示词通过Secure LLM Gateway采用TLS协议发送给三方LLM服务，并与三方LLM服务商签署零数据留存(Zero Data Retention)协议，三方LLM服务商不能将数据用于训练或者产品改进，Prompt和LLM生成的数据都不会被外部存储。

⑦ **毒性检测** ：采用毒性检测模型对LLM的回复信息进行评分，将LLM回复和评分信息返回给应用，支撑应用保护用户不会看到有毒/仇恨/暴力/性/粗野/亵渎性等危害性内容。毒性评分作为审计内容被存储数据云中。Salesforce表示毒性检测的准确性与语言相关，不保证100%的识别准确率。

⑧ **数据解蔽** ：将④数据屏蔽阶段屏蔽掉的数据条目还原为原始数据。此外，回复底部增加引用源链接（指向文章来源的链接），增加可信度。

⑨ **反馈框架** ：收集用户对响应内容的反馈(接受/修改/拒绝) ，使用反馈来提高Prompt的质量。

⑩ **审计跟踪** ：记录用户与系统交互过程中产生的数据，如Prompt、原始回复、有毒语言分数、反馈等，都会被审计跟踪模块收集和记录。提供预构建的报告和仪表板以供分析。

## 5 蚂蚁金融[蚁天鉴](https://zhida.zhihu.com/search?content_id=262938597&content_type=Article&match_order=1&q=%E8%9A%81%E5%A4%A9%E9%89%B4&zhida_source=entity)

基于金融场景中的大量实践，蚂蚁金融形成了“大模型 + 知识 + 服务 + 可信围栏”的架构，对内服务于支小宝和支小助等金融智能体，对外也向20+外部结构和企业使用。在该架构下，依托可信围栏构建了蚂蚁金融大模型的LLM Guardrails——蚁天鉴。蚁天鉴是由蚂蚁联合清华大学联合打造的大模型安全一体化解决方案，提供双重防御护栏，内置防护关注训练阶段的数据清洗和风险抑制；外置护栏融合智能风控技术，拦截输入和输出风险内容，保障应用安全。

蚁天鉴2.0包含大模型安全检测平台“蚁鉴”和大模型风险防御平台“天鉴”两大产品：

  * **蚁鉴** ：用生成式能力检测生成式系统，覆盖了内容安全、数据安全、科技伦理全风险类型，适用文本、表格、图像、音频、视频等全数据模态。 
  * **天鉴** ：动态监测用户与模型的交互，防止诱导攻击，对生成的回答内容进行风险过滤，保障大模型上线后从用户输入到生成输出的整体安全防御。



其解决方案架构如下所示：

  * **大模型X-ray** ：针对大模型内在神经元进行X光扫描，支持研究人员了解大模型内部发生什么、定位可能引发风险的神经元、并进行编辑修正。
  * **AI鉴真** ：支持多模态内容真实性及深度伪造检测，可快速精准鉴别图像、视频、音频、文本内容真伪，图像识别准确率99.9%。
  * **可解释性检测工具** ：通过逻辑推理、因果推断、可视化等技术，从完整性、准确性、稳定性等7个维度及20余项评估指标，提供可解释性量化分析能力。
  * **高效评测** ：拥有超300万高质量测评题库，支持最高50万/日的饱和式攻击和逐级诱导深度攻击，1工作日内完成测评，全流程自动化率99%。
  * **数据集** ：依托生成模型自建百万量级音视图多模态合成数据集，有效应对AI换脸、声音模拟、证件伪造等各类深度伪造风险场景。



### 5.1 大小模型协同

  * 蚁天鉴采用大小模型协同防御的方案，输入先通过性价比和可信度高的专精小模型检查，偏灰的潜在风险再由大模型检查。
  * 其风险知识库每日动态更新，数据来源于官网/官媒/内外部文档/审核案例库，覆盖内容安全、数据安全、伦理安全3大类风险类型的40+细分类目标签；采用AutoPrompt技术让模型生成最理解自己的提示词，提升模型性能4~10%；真实交互数据沉淀为审核案例库丰富大模型知识。
  * 策略控制台提供大小模型协同策略配置、效果分析、实时监控管理能力。



### 5.2 基于大模型面向交互构建多层防御护栏

  * 通过多层防御护栏，在提问环节对潜在风险做预防，对生成的回答内容进行风险过滤，保障其应用安全，并能面向用户和场景进行动态策略调优。
  * 结合精准匹配(通过关键词进行扩充和改写，包括英文代替、特殊字符代替等) + 模糊匹配对内容实现内容检查和拦截，支持PII识别过滤敏感信息。
  * 通过意图理解、话题理解和领域理解，针对不同话题和领域采用不同的领域知识库和Prompt引导大模型生成回答。
  * 通过明水印、暗水印技术对大模型生成的图像和短视频内容进行版权保护。



## 6 总结

LLM Guardrails是大模型应用安全生态的核心要素，其重要性随大模型在各行各业的深度应用而日益凸显。作为关键的运行时防护层，Guardrails确保了LLM行为符合预设的道德、法律与业务合规要求。其实现方式已从初期的简单规则过滤，演进为覆盖AI全流程（从输入验证到输出审核）的复杂多阶段AI驱动系统。

展望未来，LLM Guardrails技术将持续向更智能、自适应的方向发展。多模态Guardrails的出现将使其能够处理更广泛的内容类型，策略即代码与自动化合规将简化大规模部署与管理。持续评估与红队自动化将成为常态，确保Guardrails能动态响应新型攻击。最终，LLM Guardrails将超越单纯的防御工具角色，成为实现负责任、可信赖AI部署的基石，助力AI技术安全、高效地释放其社会潜能。

## 相关链接

  1. [https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html](https://link.zhihu.com/?target=https%3A//docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html)
  2. [https://aws.amazon.com/cn/bedrock/guardrails/](https://link.zhihu.com/?target=https%3A//aws.amazon.com/cn/bedrock/guardrails/)
  3. [https://aws.amazon.com/cn/blogs/machine-learning/build-safe-and-responsible-generative-ai-applications-with-guardrails/](https://link.zhihu.com/?target=https%3A//aws.amazon.com/cn/blogs/machine-learning/build-safe-and-responsible-generative-ai-applications-with-guardrails/)
  4. [https://aws.amazon.com/cn/blogs/aws/guardrails-for-amazon-bedrock-can-now-detect-hallucinations-and-safeguard-apps-built-using-custom-or-third-party-fms/](https://link.zhihu.com/?target=https%3A//aws.amazon.com/cn/blogs/aws/guardrails-for-amazon-bedrock-can-now-detect-hallucinations-and-safeguard-apps-built-using-custom-or-third-party-fms/)
  5. [https://github.com/NVIDIA/NeMo-Guardrails/blob/develop/docs/user-guides/guardrails-process.md](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/NeMo-Guardrails/blob/develop/docs/user-guides/guardrails-process.md)
  6. [https://docs.nvidia.com/nemo/guardrails/latest/](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/nemo/guardrails/latest/)
  7. [https://www.salesforce.com/artificial-intelligence/trusted-ai/](https://link.zhihu.com/?target=https%3A//www.salesforce.com/artificial-intelligence/trusted-ai/)
  8. [https://trailhead.salesforce.com/zh-CN/content/learn/modules/the-einstein-trust-layer/follow-the-response-journey](https://link.zhihu.com/?target=https%3A//trailhead.salesforce.com/zh-CN/content/learn/modules/the-einstein-trust-layer/follow-the-response-journey)
  9. [https://marmatodigital.com/salesforce-einstein-trust-layer/](https://link.zhihu.com/?target=https%3A//marmatodigital.com/salesforce-einstein-trust-layer/)
  10. [https://www.constellationr.com/blog-news/insights/salesforce-launches-ai-cloud-aims-be-abstraction-layer-between-corporate-data](https://link.zhihu.com/?target=https%3A//www.constellationr.com/blog-news/insights/salesforce-launches-ai-cloud-aims-be-abstraction-layer-between-corporate-data)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【合规治理/安全护栏】01-Progent：首个面向AI Agent的可编程权限控制机制](https://zhuanlan.zhihu.com/p/1913314905567786294)
