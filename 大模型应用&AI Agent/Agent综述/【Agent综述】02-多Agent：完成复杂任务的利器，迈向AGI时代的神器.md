# 【Agent综述】02-多Agent：完成复杂任务的利器，迈向AGI时代的神器

原文链接：https://zhuanlan.zhihu.com/p/32708623482

---

​

目录

近年，随着大语言模型（[LLM](https://zhida.zhihu.com/search?content_id=255564045&content_type=Article&match_order=1&q=LLM&zhida_source=entity)）的爆火，其具备的强大的自然语言处理能力，成为AI领域的一大热点。它们不仅能生成和理解文本，还能进行复杂的分析和推理。同时，基于LLM的Agent（[智能体](https://zhida.zhihu.com/search?content_id=255564045&content_type=Article&match_order=1&q=%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)）也走入了人们的视野。智能体借助LLM的思考和推理能力，能够针对任务生成执行计划，并调用工具逐步完成任务目标。Agent被认为是大语言模型（LLM）的下半场，或许它将是决定大语言模型革命的关键因素。

## 什么是Agent

### 起源和演进

Agent的概念最早起源于哲学，其根源可以追溯到Aristotle（亚里士多德）和Hume（休谟）等思想家1。它描述了一种拥有欲望、信念、意图以及采取行动能力的实体。后来，人们将其引入至AI领域中，Agent被赋予了一层新的含义：**具有自主性、反应性、主动性和社会性特征的智能实体** 。

  * **自主性：** 智能体在没有人类或其他实体的直接干预下运行，并**对其行动和内部状态具有某种程度的控制** 。
  * **反应性：** 智能体**感知其环境** (可能是物理世界、通过图形用户界面的用户、一组其他智能体、互联网，或者可能是所有这些的结合体)，并及时对发生的变化**做出响应** 。
  * **主动性：** 智能体不仅仅是对环境做出反应的，它们还能够通过采取**主动行动** 来展示以目标为导向的行为。
  * **社会性：** 智能体的社会属性，它是指智能体通过某种通信语言与其他智能体（可能还包括人类）**互动和社交的能力** 。



Agent的演变经历了几个阶段，从20世纪80年代开始，Agent在不断演进中，从**无“脑”** 演变为**有“脑”。** 从初始的采用逻辑规则和符号来封装知识和促进推理过程，演变为采用**LLM作为大脑** ，并通过更多的组件赋予其更多的能力，如通过工具扩展其能力，通过思维链实现类似推理规划能力等。

图1 Agent演进示意图

### Agent的概念

Agent是一种能够感知环境、进行决策和执行动作的智能实体。其目前可分为[自主智能体](https://zhida.zhihu.com/search?content_id=255564045&content_type=Article&match_order=1&q=%E8%87%AA%E4%B8%BB%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)（Autonomous Agent）和[生成智能体](https://zhida.zhihu.com/search?content_id=255564045&content_type=Article&match_order=1&q=%E7%94%9F%E6%88%90%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)（Generative Agent）。

  * **自主智能体** ：用于解决问题，如Auto-GPT，通过调用工具，分解复杂问题，编排思维模式等方法，更好的解决用户提出的问题。
  * **生成智能体** ：用于模拟研究，如斯坦福和谷歌的西部世界小镇，用agent模拟人的行为选择，探究人群体在特定环境下的行为表现，应用于疫情传播，舆情传播等研究。



2023年6月份，OpenAI Lilian Weng发表了一篇文章。在该文章中，推出了基于LLM的Agent系统，其以**LLM充当其大脑** ，并以其他关键组件为辅的架构：**规划、记忆、工具** ，如图2所示。

图2 基于LLM的Agent系统概览

**LLM：** 扮演了Agent 的“大脑”， 是Agents的核心，承担着存储知识、记忆、信息处理和决策等能力。

  * **规划：** 提供规划能力，帮助Agents将复杂问题分解规划为更小、更简单的子任务，并逐个解决。同时，Agents可以对过去的行为进行自我批判和反思，从错误中吸取经验，使Agents能够修正以往的决策、纠正之前的失误，从而不断优化其性能。
  * **记忆：** 对Agents系统而言，记忆可以定义为用于获取、存储、保留以及随后检索信息的过程。它可以利用记录下来的记忆来促进未来的行动。记忆模块可以帮助Agents积累经验、自我进化，并以更一致、合理和有效的方式行动。
  * **工具：** Agents可以学习如何调用外部API，以获取模型权重中缺少的额外信息，这些信息通常在预训练后很难更改，包括当前信息、代码执行能力、专有信息源的访问等。



后续在2023年9月，人大和复旦相继发表论文，在他们的论文中，相继为Agent系统架构提出了**Profile（配置）** 和**Preception（感知环境）** 概念，促使Agent架构持续演进。

  * **Profile（配置）：** 通常通过扮演特定的角色来执行任务，例如程序员、教师等。该模块旨在确定单Agents的角色，包含基本信息和行为倾向等“心理”信息；多Agent之间的社会信息。
  * **Preception（感知）** ：感知并处理来自外部环境的多模态信息，如纯文本拓展到包括文本、视觉和听觉等多模态领域。



当前，**基于LLM的Agent能力大体达成共识，包含大脑、规划、感知、行动** 等。不过，其还在不断的演进，尚未收敛。后续，Agent系统框架演进的趋势，将会由任务驱动，越来越注重Agent定制的容易度，多Agent管理、交互等。

## 为什么需要多Agent

### 多Agent的概念

单Agent指一个独立的Agent，在执行任务时，通常由其自己进行。而**多Agent系统是指由多个相互作用的，基于LLM构建的Agent所组成的系统** 。它们通过**共享信息** 、**通信** 和**协调行动** 来共同**解决问题或完成任务。**

在20世纪80-90年代，已经有研究者开始关注Agent之间的交互、协作和协调等问题。后来，随着科学技术的不断更新迭代，研究者们在各个阶段，不断的对多Agent进行探索。在2024年，Taicheng Guo等在《Large Language Model based Multi-Agents: A Survey of Progress and Challenges》论文中提出了目前较为全面和准确的多Agent定义：**多Agent系统是由环境接口、画像、通信、能力获取为核心要素的系统。**

图3多Agent演进示意图

### 为什么需要多Agent

 _What magical trick makes us intelligent? The trick is that there is no trick. The power of intelligence stems from our vast diversity, not from any single, perfect principle._

_是什么神奇的技巧赋予了我们智能？答案是并没有什么诀窍。智能的力量源自广泛的多样性，而非任何单一、完美的原则。_

_——Marvin Minsky，《Society of Mind》, 1986_

虽然单Agent在多个场景下表现出色，但在一些复杂环境下，其适用性仍然存在着局限性。

  1. 其一，Agent的认知依赖于模型，使得其难以在瞬息万变的复杂环境中，及时全面把握多元信息。从而产生知识盲区和认知偏差，导致其可能在新的环境下无法做出正确决策。
  2. 其二，对于一些复杂的问题，可能涉及多种模型协同配合，而单Agent难以整合不同的AI范式，解决复杂问题的能力有限。
  3. 其三，单Agent系统的可扩展性较差，难以适应不断变化的复杂业务需求。



多Agent系统，由多个独立的Agent构成，每个Agent都拥有其独特角色定义、领域知识和资源，可以通过灵活协作，共同完成复杂的决策任务。通过角色定义，赋予每个Agent不同的职责，使其可以根据专长承担不同的子任务，高效的完成复杂任务的处理，从而让系统拥有了强大的处理多样化的复杂任务的能力。同时，多Agent天然具有可扩展性，通过引入新的Agent即可实现无缝扩展。整体上来讲，多样化的Agent将会带来“3F”优势：

  1. **Focus** ：让模型每次只专注一件事情，效果更好。  
其根本原因是各个LLM对理解冗长、复杂输入的能力参差不齐。
  2. **Fragment** ：允许将复杂任务分解为多个简单的子任务。  
分而治之，降低复杂问题的解决难度。
  3. **Fusion** ：允许多个模型被混用，分别用于处理各模型擅长的事情



1+1+… > n，专业的Agent做专业的事

### 多Agent应用场景

当前多Agent系统主要应用于两大类场景：问题解决、模拟世界

一、问题解决（Problem Solving）：**利用多个Agent的专业能力，通过协作解决实际的复杂问题。**

  * **软件开发（Software Development）** ：模拟软件开发过程中的不同角色，如产品经理、工程师等，以协作完成软件开发任务。
  * **具身智能体（Embodied Agents）** ：用于模拟具有不同能力的Agent，Agent通过协作完成复杂的现实世界规划和操作任务，例如仓库管理。
  * **科学实验（Science Experiments）** ：构建一个多Agent的科学团队，不同的Agent扮演不同的专家角色，合作进行科学实验。人类在这个过程中可以起到协助作用，处理Agent的信息并及时给予反馈。
  * **科学辩论（Science Debate）** ：在科学辩论场景中，Agent通过辩论来增强集体推理能力，通过多轮辩论达成共识答案。



二、世界模拟（World Simulation）：**模拟特定的交互场景，用于研究现实世界互动的复杂性与多样性。**

  * **社会模拟（Societal Simulation）** ：模拟社会行为和动态，探索潜在的社会动态和传播，测试社会科学理论。
  * **游戏（Gaming）** ：模拟游戏环境，Agent在游戏中扮演各种角色，用于测试游戏理论假设。
  * **心理学（Psychology）** ：模拟具有不同特征和思维过程的人类，分析他们的多样化行为。
  * **经济（Economy）** ：模拟经济和金融交易环境，探索宏观经济活动、信息市场、金融交易等。
  * **推荐系统（Recommender Systems）** ：在推荐系统中模拟用户和物品的交互，优化它们以反映和调整现实世界的交互差异。
  * **政策制定（Policy Making）** ：模拟政策制定过程，通过模拟虚拟政府或政策对不同社区的影响来提供决策洞察。
  * **疾病传播模拟（Disease Propagation Simulation）** ：用于模拟疾病传播，分析Agent对疾病爆发的响应行为。



## 多Agent核心要素

多Agent系统是指由多个相互作用的，基于LLM构建的Agent所组成的系统。我们认为，一个多Agent系统主要包含五大核心要素：**环境** 、**画像** 、**通信** 、**协作** 和**演进** 。

图4 多Agent架构图示意图

  1. **环境** ：多Agent系统所处的物理或虚拟空间，定义了所有Agent交互和执行任务的上下文，包括环境中的规则、资源及其他可能影响Agent行为的因素。
  2. **画像** ：对各个Agent特征、能力和行为模式的描述，可以人工设定画像、使用预置画像或者基于任务需求自生成画像，并能根据模型的变化进行自优化。
  3. **通信** ：各个Agent之间信息交换的方法，通过定义统一的通信接口、原语与协议，规定了Agent之间如何发送和接收信息，以及通信的格式、语义和顺序。
  4. **协作** ：各个Agent之间的交互关系与拓扑结构，可以使用预设的协作模式，也可以根据任务需求实时创建Agent或调整Agent数量，自主协作以适应不同任务需求。
  5. **演进：** 多Agent系统通过联合学习不断提升整个系统水平的过程，各Agent利用外部知识，与环境交互并相互反馈，积累和共享经验，以适应复杂环境和实现持续优化。



### 环境

**环境是指多Agent系统所处的物理或虚拟空间，定义了所有Agent交互和执行任务的上下文。** 典型的环境分为两种：虚拟环境、物理环境。

  * **虚拟环境** ：由人类构建的模拟或虚拟环境。如：MetaGPT的软件开发（代码解释器作为仿真环境）、 Alympics游戏（游戏空间作为仿真环境）。



图5 虚拟环境

  * **物理接口** ：真实世界的环境。Agent直接与物理实体交互并遵守现实世界的物理规律。如：扫地机器人、三明治制作等。



图6 物理环境

### 画像

**画像是对各个Agent特征、能力和行为模式的描述** ，可以人工设定画像、使用预置画像或者基于任务需求自生成画像，并能根据模型的变化进行自优化。例如：

  * 游戏模拟场景：利用画像区分不同Agent扮演的游戏角色。
  * 软件开发场景：利用画像将Agent分为开发/测试等工程师。
  * 辩论场景：利用画像决定Agent是支持者、反对者还是裁判。



Agent画像的定义方式一般分为三种：**预定义** 、**模型生成** 和**数据驱动。**

**预定义**

设计系统时由开发者预先定义不同Agent画像。

如图7所示，例如软件开发场景中，会预先对“工程师”的名称、角色、目标、限制等进行设定。

图7 画像示意

**模型生成**

在系统运行期，通过模型（如LLM）动态生成agent的画像。

如图8所示，ChatDev系统用于模拟一个基于多agent的软件开发公司，它将软件开发过程分为设计、开发、测试等不同阶段的子任务，在执行每个任务时，使用动态生成的Agent去解决问题。

  1. 使用链式的处理流程按照设计、开发、测试等流程处理任务。
  2. 解决每个子任务的时候，将任务相关的信息发送给LLM，由LLM生成完成该任务所需的多个Agent，之后由Agent共同完成该任务。



图8 模型生成画像

**数据驱动生成**

基于已有的数据生成agent的画像。

如图9所示，Agent4Rec是一个电影推荐算法优化系统，它通过真实数据集中“电影偏好”、“评分倾向”两类数据，生成了1000个不同的agent，用于模拟1000个不同的用户，然后让这些Agent模拟用户给电影打分和评论，以便对电影推荐算法进行评估和优化。

图9 数据生成画像 Source：《On Generative Agents in Recommendation》

当前，画像已广泛被用于构建多Agent系统。如AutoGen AgentBuilder可根据任务需求动态的基于LLM生成Agent画像，亦或是基于Prompt自优化技术动态优化多Agent画像。

如下图，即为AutoGen AgentBuilder可根据任务需求动态的基于LLM生成Agent画像。

图10 AutoGen AgentBuilder基于LLM生成Agent画像

  * **Agent信息生成** ：AgentBuilder利用LLM和任务场景描述，依次生成需要的Agent的name、sys_msg、description信息。
  * **Agent构建** ：AgentBuilder将生成的Agent和对应的信息组装为AutoGen的Agent类。
  * **配置保存和加载** ：AgentBuilder支持将生成的Agent配置信息保存为文件，并支持从文件中再次加载。
  * **大模型配置** ：AgentBuilder和生成的Agent支持配置不同的LLM，但是生成的Agent之间的LLM配置都相同。



**基于多Agent Prompt自优化画像** ，通常指根据场景描述自动扩展多Agent的Prompt，同时可根据场景对多Agent的Prompt进行端到端调优。业界多Agent Prompt自优化技术的发展现状：

  * AgentScope支持多Agent提示词扩展。
  * DSPy与TextGrad支持端到端优化。



### 通信

**通信** 是各个Agent之间信息交换的方法，通过定义统一的通信接口、原语与协议，规定了Agent之间如何发送和接收信息，以及通信的格式、语义和顺序。

**通信内容是指Agent之间发送的消息中所包含的实际信息。**

  * 目前多以文本形式进行通信，未来可能会涉及多模态通信。  
**通信协议是用于定义自主Agent之间如何交换信息、协作和协调行为的规范和规则集合** ，以确保各个智能体能够有效地沟通和合作，通常包括：
  * **消息格式和语法** ：定义消息的结构、内容和表示方式。
  * **通信模式** ：描述智能体之间通信的方式和顺序，例如请求-响应模式、发布-订阅模式、共享消息池等。
  * **协议规则** ：规定智能体在不同情景下如何生成、发送、接收和处理消息，以确保通信过程的有序和一致性。
  * **错误处理和恢复机制** ：定义在通信过程中出现错误或异常时，智能体应采取的措施和恢复步骤。



当前，如MetaGPT、SuperAGI框架中已经在使用结构化的Agent通信协议构建多Agent的通信能力。同时，如论文“**Internet of Agents（IoA）** ”的研究中，通过定义一套统一的通信接口与协议使得异构Agent之间能够通信。从而当多Agent可以无缝地协作以实现更高的智能和能力。

在MetaGPT框架中，**通信结构** 采用了共享消息池，使得Agent能够从公共信息中选择性获得信息，提升通信效率**。** 但MetaGPT整体通信顺序需预先定义。在**通信协议层** ，其使用了结构化的通信协议用于规定消息发送方、接收方、内容等。

图11 MetaGPT框架通信架构图

在论文《Internet of Agents: Weaving a Web of Heterogeneous Agents for Collaborative Intelligence》中，作者们通过定义一套统一的通信接口与协议使得异构Agent之间能够通信。

图12 IoA框架架构图

IoA在整体架构上**采用客户端-服务端架构模式** ：所有的Agent都作为客户端，服务端作为Agent之间通信的中心，Agent之间的所有通信由服务端转发，客户端和服务端通过REST和websocket两种接口通信。

在通信协议方面，其采用了**统一的通信接口与协议，** 使得基于不同框架开发的Agent（异构Agent）之间也可以进行通信，只需各个Agent实现对标准接口与协议的适配即可。

IoA为不同的操作定义了不同的通信结构：

  1. **header头** ：所有消息都带有header，里面包含了发送者、状态和会话标识。
  2. **讨论消息** ：Agent之间进行会话讨论的消息包含了消息内容、消息类型、下一个发言Agent等，客户端与服务端采用websocket方式发送该消息。
  3. **组建团队消息** ：组建团队的消息里面包含了团队目标、成员等信息，客户端调用服务端的/teamup接口发送消息。
  4. **分配任务消息** ：Agent之间分配任务的消息包含了任务标识、任务描述等信息，客户端与服务端采用websocket方式发送该消息。



图13 IoA的通信结构示意图

### 协作

**协作** 指的是各个Agent之间的交互关系与拓扑结构，可以使用预设的协作模式，也可以根据任务需求实时创建Agent或调整Agent数量，自主协作以适应不同任务需求。

多Agent协作模式的实现分为两种方式：

  1. **静态编排** ：根据任务场景编写固定拓扑结构的协作模式，一般需要开发者自己编写代码或者借助编排框架实现。
  2. **动态编排** ：根据任务场景由大模型动态组建多Agent团队，在处理任务过程中可以根据当前状态动态调整Agent协作模式，进行自主协作。



**静态编排**

在协作的静态编排中，包含**图编排、角色编排和任务编排** 三种主流方式。如LangGraph采用了图编排、AutoGen采用角色编排、CrewAI采用任务编排方式。

在LangGraph中，其以带循环的图编排方式支持多Agent复杂协作模式，可以通过SDK制定图中的节点、边和共享状态，编排多个Agent、工具完成任务。

图14 LangGraph图编排示意图

在AutoGen中，其以角色为中心，通过定义角色职责和对话方式，编排协作方式。

图15 AutoGen角色编排示意图

在左图中，AutoGen采用了**合作（中心化）模式** ，writer负责通过写代码解决用户问题，safeguard负责代码安全扫描，commander负责执行代码。在右图中，，AutoGen采用了**竞争（去中心化）模式** ，三个ConversableAgent，board负责移动棋子，玩家A和B进行对弈。

**动态编排**

动态编排的方式丰富多样，可以利用预定义Agent动态方式进行编排，实在Agent运行过程中动态选择下一个Agent实现动态协作编排，还可以基于元模板生成技术动态构建可写作的多Agent系统，实现动态编排。

在AutoAgents中，其**利用预定义Agent动态生成Agent并分解任务** 。AutoAgents在执行过程中，分为两个阶段。

  1. **起草阶段（Draft Stage）** ：使用三个预定义Agent合作生成完成任务需要的Agent列表和任务分解。
  2. **执行阶段（Execution Stage）** ：动态生成的Agent解决各自的任务，预定义Agent（Action Observer）负责管理团队Agent执行任务的顺序。



图16 AutoAgents架构示意图

在整体上，AutoAgents的Agent分为两大类：

  1. **预定义Agent**


  * **规划者（Planner）** ：根据用户求解的任务动态生成多个不同的Agent角色组成团队，并将原始任务拆分给团队中各个Agent。
  * **Agent观察者（Agent Observer）** ：评判规划者动态生成的Agent团队是否满足任务需求，如果不满足给出建议。
  * **计划观察者（Plan Observer）** ：评判规划者拆分给各个Agent的任务是否合理，如果不合理给出建议。
  * **行动观察者（Action Observer）** ：对团队各个Agent的执行结果进行观察并决定下一步要执行的任务。



  


  1. **动态生成的Agent** ：由规划者借助大模型能力生成，默认采用[ReAct模式](https://zhida.zhihu.com/search?content_id=255564045&content_type=Article&match_order=1&q=ReAct%E6%A8%A1%E5%BC%8F&zhida_source=entity)执行任务，Agent之间不直接交互，由行动观察者通过prompt上下文实现信息传递。



在AgentVerse中，其通过**在运行过程中通过评估反馈动态选择下一个发言的Agent，实现动态协作。** AgentVerse选择过程中，分为两个阶段：

  1. **专家召集阶段** ：可动态生成solver和critic的两种Agent类型的画像。
  2. **协同决策阶段** ：可以利用LLM能力从solver和critic Agent里面决策下一个发言的Agent。



图17 AgentVerse架构示意图

在论文《Adaptive In-conversation Team Building for Language Model Agents》中，作者们提出了根据场景描述自动生成多个Agent的配置项，为特定任务动态构建多Agent团队的方案。其主要步骤包括：

  1. 根据任务描述生成Agent名称。
  2. 根据任务和Agent名称生成对应的角色技能。
  3. 将生成的多个Agent组织为一个多Agent团队求解任务。



技术实现机制主要基于元模板生成技术。

图18 根据任务自适应生成Agent流程示意图

### 演进

**演进** 是多Agent系统通过协作式探索和联合学习不断提升整个系统水平的过程，各Agent协同利用外部知识，通过与环境交互以及相互反馈来积累和共享经验，以适应复杂环境和实现持续优化。

多Agent演进的实现分为两种方式：

  1. **反馈** ：Agent通过接收特定行为的执行结果从而了解其行为的潜在影响并适应复杂和动态的问题。反馈根据来源可分为：


  * **来自环境的反馈** ：来自真实或者虚拟仿真环境，例如软件开发场景可以以代码解释器的结果作为反馈。
  * **来自其他Agent的反馈** ：来自其他Agent的处理结果。例如科学辩论场景Agent之间会进行交互。
  * **来自人类的反馈** ：人类的直接干预和输入。


  1. **自我调整** ：Agent可以通过以下方式进行自我调整：


  * **记忆** ：Agent使用记忆保存历史信息（经验），并利用记忆来调整自身行为。
  * **自演化** ：Agent可以通过自我修改来动态的迭代，比如改变初始目标和规划策略，或根据反馈或日志进行自我训练。



对于多Agent而言，其演进依赖于反馈和经验，而持续的反馈和经验的基础技术，则是记忆共享。多个Agent通过实时记忆过滤、存储和检索框架，共享彼此的交互记忆，以增强上下文学习过程。

在论文《Memory Sharing for Large Language Model based Agents》中，作者们提出了一种记忆共享框架，支持记忆共享、生成和检索。该方案针对**技术挑战，** Agent上下文的缺失或冗余都会导致任务处理能力下降，需要为各个Agent提供记忆上下文，包括Agent独享记忆、Agent间共享记忆。采用了**关键技术，** 实时记忆过滤、存储和检索机制，以及通过多Agent交互学习实现记忆的动态增长和多样性增强。

图18 记忆共享框架

在论文《MemGPT: Towards LLMs as Operating Systems》中，其提出的MemGPT采用虚拟上下文管理扩展上下文长度，用于多Agent场景进行超长上下文扩展。该方案针对**技术挑战，** 受限于固定长度的上下文窗口，限制了Agents处理长对话或长文档的能力；即使是长上下文模型也难以有效利用额外的上下文信息。其解决该挑战的**关键技术** ，即在Agent记忆层面提供虚拟上下文管理；同时允许智能管理不同存储层级，从而有效地扩展上下文。可供多Agent系统参考。

图19 记忆共享框架

微软团队提出了一种名为STE（Simulated Trial and Error，模拟试错）方法，通过反复探索学习API工具的的使用实现多Agent的自演进。STE分为两大阶段：

  1. **探索** ：该阶段主要利用LLM能力针对已有的API工具进行探索调用。
  2. STE会经过多次回合进行探索，每一回合首先利用LLM根据API描述生成问题（query），然后利用ReAct模式使用API求解问题。
  3. 每一回合内使用短期记忆记录ReAct求解过程。
  4. 每一回合结束将对应的问题和处理结果作为长期记忆保存。
  5. **开发** ：将探索阶段求解问题及对应答案作为后续问题求解等场景的经验。对应上图将探索经验作为工具使用示例，通过工具微调或者情境学习方式利用工具求解问题。



图19 STE整理流程示意图

在STE实现中，其还利用了长短期记忆存储经验。**短期记忆** 用于记录在某一个求解问题的回合中ReAct模式的探索轨迹。**长期记忆** 用于在多轮求解问题回合中存储过去的试错经验，具体指的是历史搜索过程中的请求及对应的处理成功或者失败的记录。

图20 STE利用长短期记忆存储经验

在AppAgent中，其通过模拟操作智能手机应用，利用LLM多模态能力生成手机应用的使用说明文档，将该文档作为经验完成用户的任务。AppAgent分为两个阶段：

  1. **探索阶段（exploration phrase）** ：通过LLM自主探索或观察人类行为，操作手机完成特定任务从而学习对应应用的使用，最终生成该应用的使用说明文档。
  2. **部署阶段（deployment phrase）** ：将探索阶段生成的应用使用说明文档作为经验，操作手机应用自主完成用户任务。



图21 AppAgent流程示意图

## 多Agent案例解析

随着Agent的爆火并出圈，Agent迅速风靡AI界。当前，已有大批多Agent框架被提出来，多Agent也有望多个领域实现落地应用。下面就当前具有代表性的部分多Agent框架做一个对比和解析。

### Camel

Camel，一个社会型智能体研究框架，由多个Agent角色通过自主沟通完成具体任务。

Camel于2023.4.16日开源，其开创了沟通式智能体的有意义的探索，首次提出了角色扮演的概念，用agent来替代人工输入，减少了人在过程中的参与。

**技术方案**

  * 提出 “角色扮演”（Role-Playing），使多个智能体能够进行对话并合作解决分配的任务。
  * 用户通过提示来引导聊天智能体完成任务，同时与人类意图保持一致。



图22 Camel角色扮演框架示意图

如上图，使用Camel开发一个带有情感分析能力的工具，该工具可以对社交媒体平台上针对特定股票的正面和负面评论进行分析。

**能力解析**

  1. 环境：无.
  2. 配置：预定义方式，通过代码定义角色名称、角色特征。
  3. 通信：使用文本形式作为通信内容，以结构化通信协议在Agent之间交换信息。
  4. 协作：预设的Agent通信的协作模式。
  5. 演进


  * 反馈：增加中间评价Agent，根据用户Agent和助理Agent产生的各种观点给出反馈来进行下一步决策，也可以接受来自用户的反馈。
  * 自我调整策略：无。



**总结**

  * 提出时间最早的多Agent框架之一。
  * 每次对话只支持两个Agent间进行，无法支持更复杂的交互形式。
  * 缺少自我调整策略，缺少记忆能力。



### MetaGPT

MetaGPT，聚焦软件开发场景的多Agent框架。

MetaGPT于2023.7.2日开源，MetaGPT通过角色定义、任务分解、流程标准化和共享消息池等技术管理多Agent。**以软件开发场景为主** ，在HumanEval数据集Pass@1得分85.9，高于GPT-4的67.0。最近在规划辩论、狼人杀、我的世界、斯坦福镇等场景。

**技术方案**

图23 MetaGPT框架示意图

通信协议（左）和具有可执行反馈的迭代编程（右）的示例。左图：Agent使用共享消息池发布结构化消息，还可以根据Agent信息订阅相关消息。右图：生成初始代码后，工程师Agent运行并检查错误。如果发生错误，工程师Agent会检索存储在记忆中的历史信息，并将它们与PRD、系统设计和代码文件进行比较。

**能力解析**

  1. **环境** ：通过**沙盒** 实现agent和环境的交互，使用代码解释器模拟执行代码。
  2. **画像** ：使用**预定义** 方式，内置了产品经理、架构师、项目经理、开发工程师和QA工程师五个角色，并为每个角色预定义了不同的配置信息。
  3. **通信** ：以文本作为通信内容，采用结构化通信协议，通过共享消息池交换信息。  




图24 MetaGPT通信架构示意图

  1. **协作：** 采用预设的方式由开发者根据场景定义协作模式。
  2. **演进：**


  * **反馈：A** gent之间会交换信息，工程师等角色也会从环境（代码解释器）获取信息。
  * **自我调整：** Agent使用记忆保存历史信息，软件开发工程师以代码解释器的执行结果进行自我进化。

**

图25 MetaGPT演进架构示意图**

如上图所示，MetaGPT里面的开发工程师首先通过LLM生成对应任务的代码，然后通过代码解释器执行代码，代码执行结果反馈给开发工程师，工程师以此作为输入进行反思和决策下一步动作，如果代码执行通过则认为任务处理完成，如果代码执行未通过或者结果不符合预期，则再次优化代码。

**总结**

  * 当前在软件开发场景表现较好（面向规则、流程固定）。
  * 但对于更复杂的场景（如规则和流程不固定）无法动态增加Agent、也无法动态与下游交互。
  * 还未看到竞争等通信范式的支持，根据其规划的场景预计正在支持中。



### LangGraph

LangGraph，基于LangChain构建多Agent应用。

LangGraph于2023.8.10日开源，是基于LangChain之上构建的一个扩展库，将LangChain的应用扩展到多Agent领域，支撑构建有状态和多角色的Agents应用，并实现了灵活构建多Agent的能力。

**技术方案**

通过构建工作流图支持多种Agent协作模式。

图26 监督者模式示意图

**监督者模式** ：引入监督者实现任务分发

  * 通信范式：合作
  * 通信结构：中心化



图27 任务分发模式示意图

**任务分发模式** ：引入路由节点，将任务分发给不同的Agent/函数节点。

  * 通信范式：合作
  * 通信结构：去中心化



图28 多级团队模式示意图

**多层级团队模式** ：根据任务逻辑分层，在每层引入监督者，实现多层级任务分发

  * 通信范式：合作
  * 通信结构：团队间分层、团队内中心化



**能力解析**

  1. **环境** ：不涉及与环境交互接口，Agents间基于特定通信范式完成任务。
  2. **画像** ：采用预定义方式，通过配置文件定义角色名称、角色特征、能力。
  3. **通信** ：Agent之间通过开发者自己定义的共享状态类（State）实现通信。
  4. **协作** ：开发者采用预设方式通过图编排方式实现自己需要的协作模式。
  5. **演进** ：


  * **反馈** ：Agent的反馈可以来源于：Agent间交互、人类反馈，需用户自行实现。
  * **自我调整** ：



支持记忆能力，Agent可以存储、访问自己的状态信息。

暂不支持自我进化等能力。

**总结**

  * **优点** ：
  * 精细控制：提供了对Agent行为、状态的精细控制，允许开发者定义Agent之间的交互和协作逻辑，能够精确地管理Agent的行为，以适应不同的应用场景。
  * 图状结构直观：使用图状结构来表示多代理系统，使得系统结构更加直观，便于理解和调试。
  * **不足** ：



复杂度高：图形化表示和精细控制同时增加了系统复杂性。因此掌握LangGraph需要一定的学习成本。

### AutoGen

AutoGen，可定制的智能体，支持灵活多样的会话场景。

Camel于2023.10.7日开源，**以对话形式解决问题** ，既支持定制agent，也支持灵活多样的对话模式，该框架简化、优化和自动化了大型语言模型的工作流程。

**技术方案**

图29 AutoGen架构示意图

Agent有两大特点：

  * 可通信：各agent是可以互相通信的，也就是说每个agent均可以收发信息，以发起或延续对话。
  * 可定制：agent可以被定制，以集成LLMs或人工输入或是工具。



**能力解析**

  1. **环境** ：通过**沙盒** 实现agent和环境的交互，agent可以调用外部工具，也可以使用代码解释器。
  2. **画像** ：使用**预定义** 方式。



内置**AssistantAgent** 、**UserProxyAgent** 和**GroupChatManager** 三种agent

  * AssistantAgent：可对话agent，不与人类交互，只与LLM或外部工具交互。
  * UserProxyAgent：可对话agent，可与人类进行交互。
  * GroupChatManager：可对话agent，负责群组里面的agent的管理。


  1. **通信** ：使用文本形式的通信内容和结构化通信协议实现Agent通信。
  2. **协作** ：既支持开发者预设协作，也支持利用AgentBuilder动态构建。  




  
**合作/中心化**  
writer负责通过写代码解决用户问题。  
safeguard负责代码安全扫描。  
commander负责执行代码。  


  
**竞争/去中心化**  
三个ConversableAgent。  
board负责移动棋子。  
玩家A和B进行对弈。

  1. **演进** ：


  * **反馈** ：agent之间会交换信息，可以调用外部工具从环境获得反馈，也可以人机交互。
  * **自我调整** ：agent可以通过别的agent的处理结果进行自我演化，比如通过编写代码求解用户问题场景，writer可以基于commander的执行结果进行迭代。



**总结**

  * 有微软背书，是目前生态最强的多Agent框架之一。
  * 支持定制Agent，但现有的三种Agent不足以支撑更复杂场景的开发，需要不断增加新的agent类型来实现扩展。
  * 人机交互的方式只支持预置的三种（Always、Never、Terminate），不支持根据任务运行情况而进行动态交互。



### Agents

Agents，支持人机交互、Agent与Agent交互，及多种通信范式。

Camel于2023.10.8日开源，专注multi-agent的框架，通过SOP定义工作流程，支持长短期记忆、工具使用、访问网络、多agent通信和人机交互等核心功能。

**技术方案**

图30 AGENTS框架示意图

  * 可以允许使用 LLM 决断状态转移过程与下一步调用的Agent。
  * 多Agent通信：根据agent的角色和当前历史记录决定下一个Agent。
  * 工具使用：能够使用外部工具。



**能力解析**

  1. **环境** ：支持沙盒类型接口，工具使用、访问网络。
  2. **画像** ：采用**预定义** 方式，通过配置文件定义角色名称、角色特征、能力。
  3. **通信** ：采用文本形式的通信内容和结构化通信协议。
  4. **协作** ：框架采用预设协作模式。
  5. **演进**


  * **反馈** ：**支持人机交互** ，用户可以自己扮演agent角色，输入自己的操作，并与环境中的其他语言agent进行交互。
  * **自我调整** ：根据长期记忆及其它Agent返回的信息更新短期记忆。



**总结**

需要人工配置各个角色的状态转移规则，角色数量多，状态转移情况多，配置复杂度将提升。

### CrewAI

CrewAI，构建面向生产的AI Agent编排框架，支持与LangChain的集成。

CrewAI于2023.10.27日开源，专为构建和编排AI Agent而设计的框架，提供了一组通用的工具和库（包括LangChain的所有工具），可用于处理多agent系统的常见任务，如agent通信、协同。

**技术方案**

图31 CrewAI框架示意图

  * 支持LangChain的所有工具。
  * Agent可以自主委派任务并相互查询。
  * 使用可定制的工具定义任务并支持动态分配给agent。
  * 支持多种LLM模型，包括开源和本地模型。



图32 示例：使用CrewAI编排两个agent实现创意生成

**能力解析**

  1. **环境** ：支持沙盒类型接口，工具使用、访问网络等。
  2. **画像** ：采用**预定义** 方式，开发者通过代码定义agent的角色、目标和工具等。
  3. **通信** ：采用文本形式的通信内容，通过结构化的通信协议调度Agent执行任务。
  4. **协作** ：采用预设协作模式。
  5. **演进**


  * **反馈** ：Agent可以委托其他Agent执行任务，或调用外部工具从环境获取反馈。
  * **自我调整** ：通过多种记忆来提高整体效率和效果。



短期记忆：自己最近的交互和结果。

长期记忆：历史执行记录。

实体记忆：和任务有关的实体信息（人物、地点、概念）。

情境记忆：交互上下文。

**总结**

当前的处理流程只支持顺序和分层，后续会逐步增加协商和共识方式。

## 多Agent总结

本文概述了多Agents，并总结了多Agent构成的核心要素。当前，主流多Agent框架对于虚拟环境、预定义画像、预设协作模式、多种反馈方式等能力已基本支持。而对于动态生成Agent 、动态编排协作模式、记忆共享、自演化等能力还在探索期。

  1. **环境** ：大多已支持与虚拟环境的交互，与物理世界的交互仍在探索期。
  2. **画像** ：以预定义画像为主流方式，主要是由开发者通过配置进行预定义，通过模型生成画像、数据驱动生成画像仍处在探索期，近来也涌现出动态优化画像的技术。
  3. **通信** ：


  * **通信内容** ：主要是文本类型，鲜见对其它模态内容的支持。
  * **通信协议** ：已有结构化的通信协议出现，但还未形成广泛采用的标准。


  1. **协作** ：大部分框架以静态编排协作模式为主，动态编排协作模式的方式仍在探索期。
  2. **演进** ：大都支持来自环境、其他Agent、人等多种反馈方式，支持长短期记忆，已有针对特定场景的单Agent的自我学习和演进技术的研究，但对于多Agent的联合学习和演进的研究仍较少。



## 多Agent挑战和思路

在研究过程中，我们也发现了多Agent框架当前面临的一些挑战：

  * 对于环境，建立与物理环境的连接，遵从物理世界的规律和规则。
  * 对于画像，根据环境的模型的变化自优化Agent画像的能力不足。
  * 对于通信，如何实现通信的可靠性、降低运行的时延，如何并行处理多模态内容的传输，语义传输效率低等问题，有待解决。
  * 对于协作，如何实现高效的编排，建立协商与容错机制，消减幻觉放大问题，人需进一步探索。
  * 对于演进，如何利用多Agent各自的反馈实现多Agent的联合学习，使得集体智慧最佳化；如何自主的积累经验，并实现自主的外部知识的筛选和获取，也是有待进一步的探索。



为解决上述挑战，将会持续在具身智能、Prompt自优化、通信协议、高效的协作/协商算法、辩论+裁决机制以及协作规则和奖惩机制、多Agent记忆和经验共享等领域持续的探索。

## 相关链接

  1. Schlosser, M. Agency. In E. N. Zalta, ed., [The Stanford Encyclopedia of Philosophy.](https://link.zhihu.com/?target=https%3A//plato.stanford.edu/entries/agency/) Metaphysics Research Lab, Stanford University, Winter 2019 edn., 2019.
  2. Lilian Weng. LLM Powered Autonomous Agents
  3. Wang, L., C. Ma, X. Feng, et al. A survey on large language model based autonomous agents. CoRR, abs/2308.11432, 2023.
  4. Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, et al. The rise and potential of large language model based agents: A survey. arXiv preprint arXiv:2309.07864, 2023.
  5. Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, Xiangliang Zhang. Large Language Model based Multi-Agents: A Survey of Progress and Challenges. arXiv preprint arXiv:2402.01680.
  6. Chen Qian, Wei Liu, Hongzhang Liu, Nuo Chen, Yufan Dang, Jiahao Li, Cheng Yang, Weize Chen, Yusheng Su, Xin Cong, et al. ChatDev: Communicative Agents for Software Development. arXiv preprint arXiv:2307.07924.
  7. An Zhang, Yuxin Chen, Leheng Sheng, Xiang Wang, Tat-Seng Chua. On Generative Agents in Recommendation. arXiv preprint arXiv:2310.10108.
  8. Linxin Song, Jiale Liu, Jieyu Zhang, Shaokun Zhang, Ao Luo, Shijian Wang, Qingyun Wu, Chi Wang, et al. Adaptive In-conversation Team Building for Language Model Agents. arXiv preprint arXiv:2405.19425.
  9. Hang Gao, Yongfeng Zhang. Memory Sharing for Large Language Model based Agents. arXiv preprint arXiv:2404.09982.
  10. Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, et al. MemGPT: Towards LLMs as Operating Systems. arXiv preprint arXiv:2310.08560.
  11. Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. Camel: Communicative agents for" mind" exploration of large scale language model society. arXiv preprint arXiv:2303.17760.
  12. [https://github.com/camel-ai/camel](https://link.zhihu.com/?target=https%3A//github.com/camel-ai/camel)
  13. Sirui Hong, Xiawu Zheng, Jonathan Chen, Yuheng Cheng, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, et al. MetaGPT: Meta programming for multi-agent collaborative framework. arXiv preprint arXiv:2308.00352.
  14. [https://github.com/geekan/MetaGPT](https://link.zhihu.com/?target=https%3A//github.com/geekan/MetaGPT)
  15. [https://www.langchain.com/langgraph](https://link.zhihu.com/?target=https%3A//www.langchain.com/langgraph)
  16. Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang, and Chi Wang. Autogen: Enabling next-gen llm applications via multi- agent conversation framework, August 01, 2023 2023. 28 pages.
  17. [https://github.com/microsoft/autogen](https://link.zhihu.com/?target=https%3A//github.com/microsoft/autogen)
  18. W. Zhou, Y. E. Jiang, L. Li, J. Wu, T. Wang, S. Qiu, J. Zhang, J. Chen, R. Wu, S. Wang, S. Zhu, J. Chen, W. Zhang, N. Zhang, H. Chen, P. Cui, and M. Sachan. Agents: An open-source framework for autonomous language agents, 2023b.
  19. [https://github.com/aiwaves-cn/agents](https://link.zhihu.com/?target=https%3A//github.com/aiwaves-cn/agents)
  20. [https://www.crewai.com/](https://link.zhihu.com/?target=https%3A//www.crewai.com/)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Agent综述】01-AI Agent，通往AGI必由之路](https://zhuanlan.zhihu.com/p/666913254)  
> 下一篇：[【Agent综述】03-基于大型语言模型的多Agent：进展与挑战综述](https://zhuanlan.zhihu.com/p/693901264)
