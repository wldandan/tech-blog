# 【AI工程论文解读】08-ChatGPT与人机协作的软件架构设计

原文链接：https://zhuanlan.zhihu.com/p/636810569

---

​

目录

[软件密集型系统](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=%E8%BD%AF%E4%BB%B6%E5%AF%86%E9%9B%86%E5%9E%8B%E7%B3%BB%E7%BB%9F&zhida_source=entity)（Software-intensive Systems）的架构设计是一个复杂的过程，需要统一利益相关者的观点、设计师的智慧、基于工具的自动化、基于模式的重用等等，以绘制一张指导软件实现和评估的蓝图。尽管它有很多好处，但面向架构的软件工程（[ACSE](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=ACSE&zhida_source=entity)，architecture-centric software engineering）也面临着许多挑战。ACSE 的挑战可能源于缺乏标准化的流程、社会技术限制和人类专业知识的匮乏等，这些都可能阻碍现有和新兴类别软件（例如 IoT、区块链、量子系统）的开发。基于大型语言模型训练的软件开发机器人（[DevBots](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=DevBots&zhida_source=entity)）可以帮助将架构师的知识与人工智能决策支持相结合，实现人机协作的ACSE ，从而实现快速架构。实现这种协作的一种新兴解决方案是 [ChatGPT](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=ChatGPT&zhida_source=entity)，这是一种颠覆性的技术，不是专门为软件工程引入的，但它能够根据自然语言处理来表达和完善架构工件。

本文通过作者们的一个案例研究得出一个初步结果，即ChatGPT 可以模仿架构师的角色来支持并主导ACSE，但它需要人类监督和决策支持来进行协作架构。

## 概述

软件密集型系统的架构使架构师能够指定结构组成、表达行为约束，并合理化设计决策，通过架构组件隐藏实现的复杂性，绘制软件实现蓝图。面向架构的软件工程（ACSE）旨在利用架构知识（例如策略和模式）、架构语言、工具和架构师的决策（人工智能）等来创建一个模型，该模型驱动软件系统的实现、验证和维护阶段。近年来，ACSE已被应用于研究在工程复杂的和新兴的软件类别(如区块链、量子系统等)中，架构所起的作用，并已被证明在工业环境中系统化软件开发非常有用。尽管ACSE潜力巨大，但其任涉及多种挑战，包括但不限于将利益相关者的观点映射到架构要求、管理架构漂移、侵蚀和技术债务，或缺乏自动化和架构师在开发复杂和新兴软件类别方面的专业知识。在这种情况下，软件工程师可能会进入一个被称为“孤独的架构师”的阶段，他需要基于流程和工具提供非侵入式支持以应对ACSE的挑战，通过重用知识并利用决策支持来解决问题。

**背景和动机：** 软件应用和服务的架构过程（即“架构过程”）统一了许多支持增量、流程中心化和系统化方法来应用ACSE于软件开发项目中。经验主义在推导或者利用架构过程方面起着关键作用，这些过程可以支持诸如分析、整合和评估等软件架构的活动。

为了丰富架构过程并赋予架构师更多的权力，研发重点是将模式和样式(知识)、推荐系统(智能)和分布式架构(协作)纳入ACSE过程中。人工智能（AI）在软件工程（SE）中的作用是一个活跃的研究领域，旨在将AI解决方案与SE实践相结合，为软件开发的过程和工具注入智能。从ACSE的角度来看，AI的研究通常旨在开发决策支持系统或开发机器人，以协助架构师对设计决策、模式和风格的选择、或预测架构故障和退化点提供建议。目前尚无研究提出创新性的解决方案，可以通过AI丰富架构过程，实现人机协作的架构设计。[人机协作架构设计](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=%E4%BA%BA%E6%9C%BA%E5%8D%8F%E4%BD%9C%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1&zhida_source=entity)可以将架构师的知识和机器人的智能代理能力协作起来，使机器人可以在人类对话和监督的基础上引导架构过程。这种协作可以使架构师将架构设计任务委托给机器人，在自然语言对话中监督机器人实现自动化，并使架构师从ACSE中繁琐的任务释放出来。

**研究目标：** ChatGPT已经成为一种颠覆性技术，代表了一种前所未有的机器人，能够在保留上下文的对话中与人类互动，产生清晰的响应来回答复杂的查询。然而，ChatGPT并不是专门为解决软件工程挑战的而开发，但它能够生成包括架构需求、[UML](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=UML&zhida_source=entity)脚本、源代码库和测试用例在内的多样化文本规范。最近发表的研究已经开始探讨ChatGPT在工程教育、软件测试和源代码生成方面的作用。考虑到ACSE可以由架构师的对话和反馈驱动，并从智能和自动化架构中受益，目前没有研究探讨ChatGPT在架构过程中作为DevBot可以发挥什么样的作用。为此，作者的研究重点是初步调查ChatGPT是否能够处理由架构师与之交谈的[架构故事](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=%E6%9E%B6%E6%9E%84%E6%95%85%E4%BA%8B&zhida_source=entity)（场景），并在人机协作架构中进行分析、整合和评估软件架构。

**贡献：** 作者们采用以过程为中心的方法和基于场景的方法，使用ChatGPT对微服务驱动的软件进行架构分析、整合和评估。初步结果表明，ChatGPT以下能力，包括但不限于处理架构故事（由架构师与之交谈），阐明达架构需求、指定模型、推荐和应用架构策略和模式，以及开发用于架构评估的场景。本研究的主要贡献：

  * 调查人机协作架构的潜力，整合ChatGPT的输出和架构师的决策，通过一个初步案例研究实现自动化ACSE。 
  * 识别ChatGPT辅助的ACSE的潜力和危险，以确定协作架构的伦理、治理和社会技术约束问题。 
  * 建立ChatGPT能力和架构师在协作架构中的生产力的基础，为未来的实证依据奠定基础（正在进行和未来的工作）。



本研究的结果可以帮助学术研究人员制定关于ChatGPT在ACSE中发挥作用的新假设，并调查新兴和未来软件的人机协作架构的可能性。实践者可以遵循所提供的指南，将他们在ACSE中的繁琐任务委托给ChatGPT进行实验。

## 研究背景和方法

接下来，将对一些核心概念进行背景说明（如图1），并讨论研究方法（如图2）。

### 人机协作架构

根据ISO/IEC/IEEE 42010:2011标准，**软件架构** 旨在通过架构组件和连接符来抽象源代码实现中的复杂性，这些组件和连接符代表了要开发的软件应用程序、服务和系统的蓝图。在学术界和工业界，以架构为中心的方法已经被证明很有效，因为它可以通过提供架构知识（如模式、风格、语言和框架）来有效地设计和开发软件。为了让软件设计师和架构师能够系统地、渐进地设计软件架构，需要有一个架构过程——即**软件架构过程** 。该过程可以包含许多细粒度的架构活动，从而支持ACSE中**架构关注点分离** （Separation of Architectural Concerns）。如图1所示的架构过程源自五个工业项目，包括三个架构活动，即架构分析、架构整合和架构评估。例如，该过程中的架构评估活动侧重于使用场景来评估设计的架构。在架构过程中，架构师可以提取并记录所需功能和其质量的要求，称为**架构重要需求** （Architecturally Significant Requirements , ASRs）。ASRs需要通过一个可以使用统一建模语言（Unified Modeling Language, UML）或架构描述语言（Architectural DescriptionLanguages , ADLs）等架构语言进行可视化或文本指定的架构模型映射到源代码实现。评估ASRs的架构模型需要使用一种架构评估方法进行评估，例如软件架构分析方法（SoftwareArchitecture Analysis Method , [SAAM](https://zhida.zhihu.com/search?content_id=229644923&content_type=Article&match_order=1&q=SAAM&zhida_source=entity)）或架构权衡分析方法（Architecture Tradeoff Analysis Method , ATAM）。

图1 LLMs、DevBots、流程、架构

软件开发机器人（DevBots）是指由人工智能驱动的会话代理或推荐系统，通过提供一定程度的自动化或在软件工程中引入智能来协助软件工程师。从软件架构的角度来看，人工智能（AI）一般和DevBots结合点仅限于机器人回答关于架构侵蚀和维护的问题或提供建议。目前没有研究或任何解决方案展示将DevBots纳入架构过程，以实现人机协作架构的软件架构设计。人机协作可以丰富架构过程，超越了问答和建议，并在ACSE中融合架构师的智慧（人类理性）和机器人的智能（自动化架构过程）。人机协作架构可以赋予缺乏经验或专业知识的初学者设计或者架构的能力，让初学者用自然语言指定需求，而通过DevBots将需求转换为ASRs、架构模型和评估场景。如图1所示，基于大语言模型（LLM）的ChatGPT可以与架构师对话，在人类监督下引导创建架构组件。

### 研究方法

现在介绍研究的整体方法，包含三个主要阶段，如图2所示。

图2 研究方法概述

**第一阶段：开发架构故事** ，软件架构故事是指通过用自然语言表达核心功能、期望质量（即ASRs）和约束，并使用文本描述预期解决方案。故事是基于分析软件领域来开发的，该领域代表系统的运行环境或通过软件解决方案实现的一组场景。架构师可以分析领域并确定场景来编写架构故事，将其作为架构过程的基础。架构故事通过提示作为人机协作架构的预处理输入到ChatGPT中。

**第二阶段：实现协作架构** 采用三个活动，详细如下。

  * 架构分析由输入到ChatGPT的架构故事驱动来表达ASRs，通过（i）自动生成和推荐的需求（由ChatGPT），或（ii）手动指定需求（由架构师），或（iii）ChatGPT和架构师之间的持续对话来细化（添加/删除/更新）需求。
  * 架构整合即将ASRs整合起来以创建一个架构模型或描述作为参考点，即可视化的软件结构和运行场景。由于UML在可用文档、易用性、图表多样性、工具支持、作为表示软件系统的语言被广泛采用等多个因素，作者们选择了UML来进行架构整合。在整合过程中，作者们还结合了知识重用和最佳实践，以策略和模式的形式来优化架构。
  * 架构评估基于架构故事中的场景，根据ASRs对整合架构进行评估。架构评估是逐步进行的，根据ASRs中的用例或场景对架构或其部分进行全面或部分验证。在评估过程中，作者们使用了软件架构分析方法（Architecture Analysis Method, SAAM）来监督ChatGPT评估架构。



**第三阶段：进行实证验证** 是对前两个阶段的补充，作为本研究的扩展，主要对协作架构进行实证验证，同时也概述了未来的工作。现有的研究范围旨在探索和介绍ChatGPT在人机协作软件架构中的作用（在第3章中）。后续工作将探讨基于实证基础指南的ChatGPT驱动协作架构所涉及的多种社会技术问题（在第5章中）。

## 协作架构案例研究

本节详细介绍了协作架构的过程，并通过案例研究进行场景化举例说明，详见图3。

 _架构故事片段_

 _“……作为迈向‘绿色校园’的一步，最小化地车辆产生的碳足迹、拥堵和噪音，大学管理部门决定推出一项自行车服务，校园访客可以按小时或按天付费使用自行车，以增强校园内外的出行便利。潜在的用户可以注册并查看他们附近（比如说在500米内）可用的自行车，并在付款后预定特定时间。为了方便这项服务，管理部门需要一款名为“CampusBike”的应用程序，该程序可在Web和移动设备上使用……”场景示例：查看可用自行车（使用位置邻近），预定特定时间的自行车（付费预定）。_

图3 人机协作架构过程概述

### 构建架构故事

架构故事是指对预期解决方案通过文本来描述待开发的软件，即通过用自然语言表达核心功能和其约束。根据图2中的方法，故事是基于分析软件领域来开发的，该领域代表系统的运行环境或通过软件解决方案实现的一组场景。架构师可以分析领域并确定场景来编写架构故事，并输入到ChatGPT中，为过程中的架构分析活动奠定基础。

### 架构分析

一旦架构故事输入到ChatGPT中，在架构分析过程中，重点是根据CampusBike软件的所需功能（如查看可用自行车）、期望质量（如响应时间¡N）以及约束（如符合相关数据安全要求）来指定ASRs。如果架构师查询时，ChatGPT需要具备能够概述ASRs或任何必要的约束的能力。然而，根据案例研究，ChatGPT所表达的ASRs和约束需要由架构师进行细化（增加、删除和修改需求）。例如，“预定自行车”由ChatGPT表达为：“……系统必须允许用户查看附近可用的自行车，并立即安全地启用自行车预定”。架构师进行了进一步的细化：

 _架构师细化后的功能：_

_查看自行车：通过位置邻近度_

 _质量：即时—在90秒内，安全—加密预定令牌_

_约束：对注册数据应用数据最小化（GDPR约束）_

在叙述架构故事之后，图4展示了架构师的查询和ChatGPT的响应（人机协作），以指定功能、质量和约束，统称为ASRs。通过两者之间的对话迭代细化ASRs，以产生最终列表详见参考资料2。

图4 制定和细化需求

### 架构整合

ASRs被整合成一个架构模型，可以用架构（建模）语言（如UML或其他架构语言）来表达。作者们使用了UML类图和组件图来创建架构模型，具体来说，使用组件图来表示整体架构，使用类图来表示架构设计的细节。在整合过程中，作者们对UML类图进行了细化，通过将单例模式应用于“UserLogin”类，来限制跨设备的单个登录会话。同时，作者们在“ViewBikes”上应用了缓存策略，在“User Location”上应用了数据最小化约束。图5。

图5 建模和细化架构设计

图5展示了架构师对ChatGPT的指令，要求它创建UML类图脚本。通过两者之间的额外对话，实现了单例模式、缓存策略和数据最小化约束在类图中的应用，详见参考资料2。

### 架构评估

一旦整合完成（如图4），就需要评估架构以确定它是否满足ASRs和约束（如图5）。作者们使用了SAAM方法（详见参考资料3）来评估整合后的架构，如图6所示。例如，架构师指定应用SAAM来评估“查看自行车”组件。ChatGPT展示了单独评估“查看自行车”组件的场景以及它与其他组件交互的场景。基于单个场景和交互场景之间的交互，生成了一份评估报告，显示了CampusBike架构的功能、质量和约束的评估结果。

图6 评估架构

## 相关工作

在本章将会讨论最相关的现有研究，概述了AI在SE和ACSE中的应用，以及ChatGPT在软件开发中的作用。

### SE和ACSE中的人工智能

关于将AI与SE相结合的研究可以分为两个不同的维度，即AI for SE（软件工程中的人工智能）和SE for AI（人工智能软件工程），详见参考资料4和5。从AI for SE的角度来看，Xie （参考资料4）认为，SE研究需要超越传统的应用AI进行基于工具的自动化和模式选择，而应该探索将智能注入软件工程过程和解决方案的方法。具体而言，SE解决方案需要在过程、模式、工具等方面维持所谓的“智能平衡”，即统一和平衡机器智能和人类智慧，以应对新兴的软件类别，如区块链和量子应用。Barenkamp等人（参考资料5）结合了系统综述和对软件开发人员的访谈结果，研究了AI在SE过程中的作用。他们研究的结果指出了SE需要在三个方面加强智能：（1）自动化繁琐和复杂的软件工程活动，如代码调试；（2）大数据分析以发现模式；（3）评估神经网络和软件定义网络中的数据。考虑到AI在软件架构中的应用背景，Herold（参考资料6）等调查了现有研究，并提出了一个概念框架，用于应用机器学习来减轻架构退化。 

### ChatGPT辅助软件工程

从软件工程的角度来看，ChatGPT是一个前所未有的聊天机器人，能够对复杂查询产生清晰明确的回答。然而，在软件开发过程中，它的潜力和危险仍然是未被探索的领域（参考资料7和8）。最近，一些提议和实验结果表明，ChatGPT的研究重点在于支持工程教育、软件编程、和测试等方面。Avila-Chauvet等人（参考资料9）详细描述了程序员如何通过与ChatGPT对话的方式，使用HTML、CSS和JavaScript在人机协助下实现开发在线任务。他们强调，尽管ChatGPT需要人类监督和干预，但它能够编写出良好的编程，并减少开发人员在编程过程中的时间和精力。类似地，在一篇文章（参考资料8）中，作者提倡采用渐进式过程（人类与ChatGPT对话），来实现遗传编程——JavaScript代码解决旅行商问题（Travelling salesman problem, TSP）。除了开发源代码，一些研究还专注于使用ChatGPT进行测试和调试（参考资料10和11）。Sobania等人（参考资料11）评估了ChatGPT在自动修复bug方面的性能。与现有的自动化bug修复技术相比，ChatGPT提供了与软件测试人员对话的方式，逐步识别和修复bug。**总结：** 根据对现有文献的综述，目前尚不存在任何研究开发去探索ChatGPT与软件工程师通过对话（基于LLM的AI）引导和支撑ACSE，其所扮演的角色。本研究补充了最近关于使用ChatGPT进行软件测试自动化和bug修复的研究，但重点放在了面向架构的软件系统开发上。在AI for SE的更广泛背景下，本研究主张人机协作的架构设计，可以将架构师的知识、监督与机器人的能力相结合，丰富ACSE过程，为软件密集型系统和服务进行架构设计。

## 讨论和有效性威胁

本章节讨论了协作架构的社会技术问题和潜在的有效性威胁。

### ChatGPT在ACSE中的社会技术问题

除了强调ChatGPT的潜力，作者们还强调了协作架构设计过程中一些需要在社会技术方面进行讨论的问题。社会技术方面，作者们指的是对问题采用统一的视角，如什么是“社会”关注、什么是协作架构设计的“技术”局限性等。这些问题都专门的系统性的调查，在此，我们仅指出了几个突出的问题，如下所示。

**响应变化：** 在人机对话中，ChatGPT可能会为完全相同的查询产生不同的响应。例如，我们观察到一个查询，如“…什么架构风格最适合CampusBike系统”，可能会产生各种不同的响应，例如微服务、分层、客户端-服务器等架构可能最适合该系统。这种类似的推荐或脚本化工件（UML脚本、ASR规范等）的变化可能会影响架构设计过程的一致性，并最终导致架构的分析、整合和评估存在差异。减少响应变化的一个解决方案是与ChatGPT进行迭代对话，以优化其输出，并由架构师进行监督，以确保所产生的架构工件是一致和连贯的。

**伦理和知识产权：** ChatGPT所表述的文本规范、特定于架构的脚本和源代码等可能会引发伦理问题，或在某些情况下侵犯版权或知识产权。例如，ChatGPT为CampusBike系统中感知用户位置的组件（GETLOCATION）生成的软件可能会导致用户位置隐私泄露和不符合监管指南（GDPR、CCPA等），这些问题必须谨慎处理。在这种情况下，架构师的作用至关重要，以确保生成的架构不违反伦理或知识产权（如有）。

**输出误差：** 这类对话式机器人的输出偏差可以归因于多个可能的因素，包括但不限于输入、训练数据或算法偏差。从架构的角度来看，关于特定架构的建模符号、策略、模式或风格等的推荐偏差，可能基于其广泛采用或训练数据中的偏差，而不是在特定环境中的最佳使用。此外，架构推荐（特定风格）、设计决策（模式选择）或验证场景（评估方法）可能会受到这种偏差的影响，在ACSE中产生次优的工件。

### 有效性威胁

有效性威胁代表了研究中的局限性、约束或潜在缺陷，可能会影响结果的普遍性、可复制性和有效性。未来的工作可以集中在最小化这些威胁，以确保方法论的严谨性和结果的普遍性。 

**内部有效性** 检查设计、实施和分析等研究过程中是否存在系统性错误（偏差）。为了设计和实施这项研究，并考虑到内部有效性，作者们遵循了一种系统方法，并利用了一种著名的架构设计过程（参考资料12）和架构评估方法（参考资料3）。基于案例研究的方法结合渐进式架构（图3）帮助分析和完善了这项研究，但是，还需要更多的工作来了解是否可以通过采用不同的架构设计过程或采用其他评估方法来验证研究。

**外部有效性** 检查研究结果是否可以推广到其他场景。作者们只对一个中等复杂的单个案例进行了研究，这可能会影响研究的普遍性。具体来说，架构设计过程的复杂度增加的情况（跨组织开发）、要开发的软件类别（关键任务软件）和人类专业知识（新手/经验丰富的工程师）都可能影响本研究的外部有效性。未来的工作计划，在结论部分中强调，通过参与架构团队并分析他们的反馈来验证协作架构设计过程，以了解外部有效性可以减少到何种程度。

**结论有效性** 决定了研究得出的结论的是否是可靠或可信。为了最小化这种威胁，作者们采用了三步过程（图2），以支持细粒度的软件架构设计过程，并验证结果（未来工作）。此外，采用了基于案例研究的方法，以确保基于场景的演示研究结果。然而，一些结论（例如架构师的生产力、ChatGPT的功效）只能通过的多个案例研究、不同的团队和协作架构设计的真实场景进行更多实验来验证。

## 结论和未来研究

ChatGPT已经成为一种颠覆性技术，一种前所未有的对话式机器人，它模仿人类对话并生成精心构思的文本工件（推荐、脚本、源代码等），常被称为“寻求问题的解决方案”。在众多的应用案例中，从内容创造到数字助手，再到担任虚拟教师等，ChatGPT作为DevBot的角色以及其架构软件密集型系统的能力仍然未被探索。本研究调查了ChatGPT协助和赋能架构师角色的架构设计过程，并与人类协作实现ACSE可能存在的潜力和风险。研究主张，在人工智能用于软件工程的背景下，应用人工智能进行基于工具的自动化的传统领域应该关注更广泛的视角，如像人机协作架构设计等领域，通过将智能注入现有流程来丰富它们。本案例研究反映了如何使用ChatGPT架构软件？在协作架构设计中需要考虑哪些因素？在在将ChatGPT整合到SE或ACSE流程中时，必须考虑响应和工件的差异、伦理影响的类型、人类决策支持/监督的水平，以及法律和社会技术问题。本研究需要经验性验证，基于证据和实验，以客观评估如下因素，诸如提高工程师生产力、优化软件工程过程、协助新晋开发人员和设计师有效地使用ChatGPT开发复杂和新兴类别的软件。 

**未来研究的需求：** 计划将本研究扩展为一系列研究，探索人类反馈和验证（即架构师的角度）并将ChatGPT集成到开发量子计算系统软件服务的过程中。作者们目前正在与多个软件开发团队（不同的人口属性，如地理分布、专业类型、经验水平和软件系统类型）进行实验，使用ChatGPT架构软件系统并记录架构师的响应。具体而言，通过涉及ChatGPT协助架构设计的案例研究，作者们可以通过访谈或文档获取架构师的反馈，以实证研究ChatGPT在ACSE中的有效性、严谨性、接受度、对人类生产力的影响以及潜在风险等方面。

## 参考资料

  1. Aakash Ahmad, Muhammad Waseem, Peng Liang, Mahdi Fehmideh, Mst Shamima Aktar and Tommi Mikkonen (2023) Towards Human-Bot Collaborative Software Architecting with ChatGPT. In Proceedings of the 27th International Conference on Evaluation and Assessment in Software Engineering (EASE).. ACM, pages 279-285.
  2. A. Ahmad, M. Waseem, P. Liang, M. Fehmideh, M. S. Aktar, and T. Mikkonen, “Replication package for the paper: Towards human-bot collaborative software architecting with chatgpt.” [https://github.com/shamimaaktar1/ChatGPT4SA](https://link.zhihu.com/?target=https%3A//github.com/shamimaaktar1/ChatGPT4SA), 2023.
  3. L. Dobrica and E. Niemela, “A survey on software architecture analysis methods,” IEEE Transactions on Software Engineering, vol. 28, no. 7, pp. 638–653, 2002.
  4. T. Xie, “Intelligent software engineering: Synergy between ai and software engineering,” in Proceedings of the 11th Innovations in Software Engineering Conference (ISEC). ACM, 2018, pp. 1–1.
  5. M. Barenkamp, J. Rebstadt, and O. Thomas, “Applications of ai in classical software engineering,” AI Perspectives, vol. 2, no. 1, p. 1, 2020.
  6. S. Herold, C. Knieke, M. Schindler, and A. Rausch, “Towards improving software architecture degradation mitigation by machine learning,” in Proceedings of the 12th International Conference on Adaptive and Self-Adaptive Systems and Applications (ADAPTIVE). IARIA, 2020, pp. 36–39.
  7. A. Borji, “A categorical archive of chatgpt failures,” arXiv preprint arXiv:2302.03494, 2023.
  8. F. Doglio, “The rise of chatgpt and the fall of the software developer - is this the beginning of the end?” Dec 2022. [Online]. Available: [https://tinyurl.com/3mxrfmjh](https://link.zhihu.com/?target=https%3A//tinyurl.com/3mxrfmjh)
  9. L. Avila-Chauvet, D. Mej´ıa, and C. O. Acosta Quiroz, “Chatgpt as a support tool for online behavioral task programming,” SSRN preprint SSRN:4329020, 2023.
  10. S. Jalil, S. Rafi, T. D. LaToza, K. Moran, and W. Lam, “Chatgpt and software testing education: Promises & perils,” arXiv preprint arXiv:2302.03287, 2023.
  11. D. Sobania, M. Briesch, C. Hanna, and J. Petke, “An analysis of the automatic bug fixing performance of chatgpt,” arXiv preprint arXiv:2301.08653, 2023.
  12. C. Hofmeister, P. Kruchten, R. L. Nord, H. Obbink, A. Ran, and P. America, “A general model of software architecture design derived from five industrial approaches,” Journal of Systems and Software, vol. 80, no. 1, pp. 106–126, 2007.


