# 【Agent综述】01-AI Agent，通往AGI必由之路

原文链接：https://zhuanlan.zhihu.com/p/666913254

---

​

目录

> 人类一直希望人工智能能成为人类的重要助手，协助人类解决各种复杂问题，完成各种各样繁琐的任务。AGI（Artificial General Intelligence，通用人工智能）将会帮助人们实现该愿景。AGI是指一种能够像人类一样思考、学习和执行多种任务的人工智能系统。多年来，人们在人工智能领域的不断研究探索，尤其是随着[大语言模型](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)的爆火，使得人们越发清晰的认识到AI Agent（智能体）和人工智能的发展是密不可分的。AI Agent被视为是通往AGI的主要探索路线。

## 概述

11月6日，[OpenAI](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)召开了首次开发者大会。在大会上，OpenAI认可了AI Agent的发展方向，并提供了一系列产品功能用于支持AI Agent的发展，包括：

  * [GPT builder](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=GPT+builder&zhida_source=entity)：对话形式构建agent的UI，无需任何代码； 
  * [Assistants API](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=Assistants+API&zhida_source=entity)：一系列高级API帮助开发者快速搭建应用； 
  * [GPTs平台](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=GPTs%E5%B9%B3%E5%8F%B0&zhida_source=entity)：Agents分发平台。开发者可以上传自己的Agent（OpenAI叫客制化GPT），并获得利润分成



智能体究竟是什么？它经历了哪些阶段的发展？典型的技术架构是什么？有哪些应用场景？本篇文章将为你全面解析！

## Agent起源与演进

随着ChatGPT等大语言模型（LLM）获得了巨大成功，在其爆火的推动下，AI Agent（智能体）相继成为各大科技巨头布局的新风口。比如微软推出了[AutoGen](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=AutoGen&zhida_source=entity)、谷歌Deepmind推出了Robotic Agent、亚马逊推出了Bedrock Agents、阿里云ModelScopeGPT、斯坦福与谷歌联合搭建的名为[《Smallville》](https://zhida.zhihu.com/search?content_id=236332636&content_type=Article&match_order=1&q=%E3%80%8ASmallville%E3%80%8B&zhida_source=entity)的虚拟小镇等等，同时，OpenAI也已然奔赴至Agent。Agent被认为是大语言模型（LLM）的下半场，或许它将是决定大语言模型革命的关键因素。

Agent是一个历史悠久的概念，最早起源于哲学，其根源可以追溯到Aristotle（亚里士多德）和Hume（休谟）等思想家[1] [2]。其描述了一种拥有欲望、信念、意图以及采取行动能力的实体。后来，随着计算机技术的兴起与快速发展，再加上人们一直以来对人工智能的期望，研究人员将Agent概念引入至人工智能领域中。在人工智能领域，AI Agent（智能体）这一术语被赋予了一层新的含义：**具有自主性、反应性、主动性和社交能力特征的智能实体** 。

  * **自主性：** 智能体能够在没有人类或其他实体的直接干预下运行，并对其行为和内部状态具有一定的控制能力。也即智能体不仅应该具备按照明确的人类的指令完成任务的能力，还应该具备独立启动和执行行动的能力。
  * **反应性：** 智能体能够对环境中的即时变化和刺激做出快速响应的能力。也即智能体能够感知其周围环境的变化，并迅速采取适当的行动。
  * **主动性：** 智能体不仅仅是对环境做出反应，而且也需要具备主动采取行动来展示出以目标为导向的能力。该属性强调智能体能够进行推理、制定计划并采取主动措施来实现特定目标或适应环境变化。
  * **社交能力：** 智能体与其他智能体（包括人类）通过某种通信语言进行交互的能力。



图1 Agent演进示意图

Agent的演变经历了几个阶段，从20世纪80年代开始，Agent在不断演进中，从**无“脑”** 演变为**有“脑”** ，从初始的采用逻辑规则和符号来封装知识和促进推理过程，演变为采用**LLM作为大脑** ，并通过更多的组件赋予其更多的能力，如通过多模态感知和工具等策略扩展其感知环境，通过思维链实现类似推理规划能力等。

## Agent是什么

### Agent概念

**Agent（智能体）是一种能够感知环境、进行决策和执行动作的智能实体。** 不同于传统的人工智能，**Agent具备通过独立思考、调用工具去逐步完成给定目标的能力。** 在有LLM作为其大脑之后，Agent更是具备了对通用问题的自动化处理能力。

Agent与大模型的区别在于，大模型与人类之间的交互是基于Prompt实现的，需要有输入才会产生输出。当输入的Prompt不清晰时，会明显影响大模型回答的效果，大多数需要多轮输入才能得出一个效果较好答案，甚至对于部分问题，大模型甚至无法处理，如问大模型今天天气怎么样。

而对于Agent，仅需要给出一个目标，它就能根据目标进行独立、自主的思考，它会根据给定任务详细拆解出每一步的计划步骤，依靠来自外界的反馈和自主思考，自主创建Prompt，来实现目标，比如问今天天气怎么样，它分解为多个步骤，通过确定你所在地点，然后调用天气查询API等步骤，为你获得你所需要的信息。

Agent目前可分为自主智能体（Autonomous Agent）和生成智能体（Generative Agent）。

  * **自主智能体** ：如Auto-GPT，主要是为人类服务，自动执行任务并实现预期结果。
  * **生成智能体** ：如斯坦福和谷歌的西部世界小镇，它们在同一环境中生活，拥有自己的记忆和目标，不仅与人类交往，还会与其他机器人互动。



### Agent架构

通常，**LLM Agent基础架构包含LLM、规划、感知、记忆与行动五个组件** 。采用直观的方式表现，即为**LLM Agent = LLM+Planning+Preception+Memory+Action** ，其中LLM 扮演了Agent 的“大脑”，在这个系统中提供推理、规划等能力，行动中包含了工具使用。

由感知（Preception）处理来自外部环境的输入信息，作为规划、决策的信息来源，由Agent的大脑（LLM）对用户指令进行任务分解和规划（Planning），通过记忆（Memory）提升精准记忆和复杂推理能力，调用外部工具（Tools）协同执行各个子任务（Action），最后通过LLM汇总生成任务执行结果。

6月份，OpenAI Lilian Weng发表了一篇文章[3]。在该文章中，推出了基于LLM的Agent系统，其以LLM充当其大脑，并以其他关键组件为辅的架构：**规划、记忆、工具使用** ，如图2所示。

**规划**

  * 子目标和分解：Agent能够将大型/复杂任务分解为更小的、可管理的子目标，以便高效地处理复杂任务。
  * 反思和改良：Agent可以对过去的行为进行自我批判和反思，从错误中吸取经验，并为接下来的行动进行分析、总结，从而提高最终结果的质量。



**记忆**

  * 短期记忆：即Prompt内的信息，所有上下文学习都可以视为利用模型的短期记忆进行学习。
  * 长期记忆：使得Agent能够长期保存和回忆（无限）信息的能力，通常使用外部向量存储和快速检索实现。



**工具使用**

  * Agent可以学习如何调用外部 API，以获取模型权重中缺少的额外信息，这些信息通常在预训练后很难更改，包括当前信息、代码执行能力、专有信息源的访问等。



图2 基于LLM的Agent系统概览

9月份，人民大学高瓴推出《A Survey on Large Language Model based Autonomous Agents》[4]综述论文。在该论文中，Agent被定义为由四个部分组成：**分析模块（Profile）、记忆模块（Memory）、规划模块（Planning）** 和**行动模块（Action）** 。其中，动作模块包含了工具部分，并新增分析模块，如图3所示。

  * **分析模块** ：通常通过扮演特定的角色来执行任务，例如程序员、教师等。该模块旨在确定Agent的角色，包含基本信息和行为倾向等“心里信息”，然后将信息写入至配置文件，并被输入至提示中以影响LLM行为。
  * **记忆模块：** 记忆模块在Agent架构设计中起着非常重要的作用。它存储从环境中感知到的信息，并利用记录下来的记忆来促进未来的行动。记忆模块可以帮助Agent积累经验、自我进化，并以更一致、合理和有效的方式行动。
  * **规划模块：** 人们在面对复杂任务时，往往会将其分解为较简单的子任务并逐个解决。规划模块旨在赋予Agent解决复杂问题时的思考和规划能力，致使其行为更加合理、有力和可靠。
  * **行动模块：** 行动模块负责将Agent的决策转化为具体的结果。其会受到其它模块的影响，并与环境直接交互，决定着Agent完成任务的结果。



图3 基于LLM的Agent架构设计的统一框架

9月份，复旦大学NLP团队推出了《The Rise and Potential of Large Language Model Based Agents: A Survey》[2]综述论文。在该论文中，Agent被定义为由三个部分组成：**控制器（Brain）、感知（Perception）和行动端（Action）** ，如图4所示。

  * **控制器** ：由LLM构成，是Agent的核心，承担着存储知识、记忆、信息处理和决策等能力。它可以展示出推理和规划的过程，并能够很好地应对未知任务，展现出Agent的智能性。其包含如下方面的能力：自然语言交互、知识、记忆、推理 & 规划和迁移性 & 泛化性。
  * **感知** ：感知并处理来自外部环境的多模态信息，如纯文本拓展到包括文本、视觉和听觉等多模态领域。其包含如下方面的能力：文本输入、视觉输入、听觉输入和其他输入。
  * **行动** ：赋予行动能力和使用工具的能力，使其能够更好地适应环境变化，通过反馈与环境交互，甚至影响和塑造环境。



图4：基于LLM的Agent的概念框架

如上图所示例子来说明工作流程：当一个人询问是否会下雨时，感知模块将指令转换为LLM能理解的语言。然后大脑模块根据当前的天气和互联网上的天气报告进行推理。最后，行动模块回应并将雨伞递给人类。通过反复进行上述过程，代理可以持续获得反馈并与环境互动。

### Agent组成

一个LLM Agent的能力大体上达成共识，包含**大脑、感知、规划、记忆和行动** ，行动中包含工具使用。当然，LLM Agent的能力也尚在不断演进。通常，**LLM Agent基础架构包含LLM、规划、感知、记忆与行动五个组件部分** ，其中LLM 扮演了Agent 的“大脑”，在这个系统中提供推理、规划等能力。

LLM Agent决策流程如下：

图5 LLM Agent决策示意图

**LLM+规划，Agent的大脑**

LM作为Agent的核心，其本身是具备逻辑推理能力的。从LLM本身来讲，在简单事务的推理上，其已经达到很好的能力；但是面对复杂问题时，LLM会显得不够适用。其原因在于Prompt不合适，难以激发出LLM推理潜力。而Agent，能够根据给定的目标，调用LLM通过思维链的能力将复杂问题分解并进行规划，创建适合的Prompt，从而解决复杂问题。

推理以证据、知识和逻辑为基础，它使得在分析解决问题时能够做出合理的决策。而规划则赋予一种结构化的思考过程，即组织思维、设定目标，并形成应对策略。基于目标的推理和规划能力是Agent基本能力，其有助于Agent将复杂问题分解规划为更小、更简单的子任务，并逐个解决。在Agent中，其推理和规划的能力则由LLM来实现。

同时，推理和规划会赋予Agent学习的能力，有助于智能体学习积累知识和经验。并且，Agent可以对过去的行为进行自我批判和反思，从错误中吸取经验，并为接下来的行动进行分析、总结，确保其与环境更好地保持一致，从而适应环境、更有效地执行任务并成功达成目标。反思框架使Agent能够修正以往的决策、纠正之前的失误，从而不断优化其性能。

图6 Agent的反思框架

**感知环境，空间扩展至多模态**

Agent通过感知模块感知外部环境，确定环境的相关状态和变化，并将空间扩展至多模态。然后将感知的信息作为后续规划决策的信息来源。Agent支持通过各种设备获取不同的环境信息，其感知空间包含文本、视觉、听觉等多模态领域。

**记忆，扩展大模型的对上下文、历史信息、领域知识的处理能力**

记忆模块在Agent架构设计中起着非常重要的作用。对Agent系统而言，记忆可以定义为用于获取、存储、保留以及随后检索信息的过程。它可以利用记录下来的记忆来促进未来的行动。记忆模块可以帮助Agent积累经验、自我进化，并以更一致、合理和有效的方式行动。

人脑中有多种记忆类型，如感觉记忆、短期记忆和长期记忆。而对于Agent系统而言，用户在与其交互过程中产生的内容都可以认为是Agent的记忆，和人类记忆的模式能够产生对应关系。感觉记忆就是作为学习嵌入表示的原始输入，包括文本、图像或其他模态；短期记忆就是上下文，受到有限的上下文窗口长度的限制；长期记忆则可以认为是Agent在工作时需要查询的外部向量数据库，可通过快速检索进行访问。

图7 Agent中的长短期记忆，Lilian Weng. LLM Powered Autonomous Agents

**向量数据库** 通过将数据转化为向量存储，解决LLM海量知识的存储、检索、匹配问题。向量是AI理解世界的通用数据形式，大模型需要大量的数据进行训练，以获取丰富的语义和上下文信息，导致了数据量的指数级增长。而向量数据库利用Embedding技术是将高维特征数据，如文本、图片、音视频等非结构化数据映射到低维度空间，转化为向量来表示，并存储在向量数据库中。然后通过向量的相似度计算，从而实现快速、高效的数据存储和检索过程，赋予了Agent“长期记忆”的能力。同时，也大幅降低了存储和计算成本。

**行动，执行“大脑”分解的子任务**

行动模块将Agent的决策转化为具体的行动，其赋予了Agent行动能力和使用工具的能力，使其能够更好地适应环境变化，通过反馈与环境交互，甚至影响和塑造环境。

人类最显著的特征之一是能够使用工具。人类通过创造、修改和利用外部对象来完成超出身体和认知极限的任务。同样，给 LLM 配备外部工具也可以显著扩展大模型的功能，使其能够处理更加复杂的任务。Agent与大模型的最大区别在于能够使用外部工具协助其完成原本无法完成的工作。Agent具备了自主调用工具的能力，其会在获取到每一步子任务的工作后，都会判断是否需要通过调用外部工具来完成该子任务，并在完成后获取该外部工具返回的信息提供给LLM，进行下一步子任务的工作。

## Agent及其应用进展

Agent成为创业风口，头部互联网、科研机构积极布局Agent框架。2023年3 月，开发人员Significant Ggravitas在GitHub上发布了开源项目AutoGPT，它以GPT-4为核心，允许根据目标自主行动，完全无需用户提示每个操作。Agent由此迅速被带火并出圈。也因此，Agent迅速风靡AI界。Agent有望多个领域实现落地应用，有的已经出现好用的产品。

**在初创公司中** ，Fixie AI研发的对话式AI开发框架，2月当前已获得12 million融资，澜码科技研发的企业级agent应用Ask xbot，在8月份融资数千万人民币，AutoGPT在10月获得12 million USD的融资等等。

**头部互联网公司** 也积极布局，微软发布了AutoGen、谷歌Deepmind推出了Robotic Agent、亚马逊发布了Bedrock agent、Meta将带来具有不同个性和能力的AI Agent来提供帮助或娱乐、阿里发布了ModelScope-Agent等等。

**Inflection AI推出情感陪伴的个人AI助手Pi** ，主打一个高情商、会说话、能提供情绪价值。Inflection AI成立于2022年，目前最新估值40亿美元，使其在人工智能领域成为仅次于OpenAI。Pi的界面简单，能够在完成基础回答基础上，提供情绪价值。其底层模型基于Inflection-1(Transformer架构，使用数千个H100 GPU训练)，强调记忆能力，能够主动跟进过去的谈话内容。作为一款辅助Agent，Pi更多的提供一款感情陪伴。

图8 Pi在情感方面的建议，来源于36氪

**在游戏中，网易《逆水寒》手游** ，内置百位AI NPC，其将智能NPC升级至AI NPC，为玩家带来了AI游戏的体验，在游戏中可以体验到AI画师、AI作诗、AI对话剧情，处处皆有AI，让游戏不再呆板。利用Agent的人格属性，使NPC可以进行更智能化、个性化的对话。同时，利用Agent的记忆属性，使NPC可以记住之前对话，并根据对话内容改变之后与玩家的交流方式。AI Agent NPC的引入，极大提升了游戏的娱乐性，并引发了游戏社区的讨论热潮。

2023年4月，斯坦福大学的研究者们发表了名为《Generative Agents: Interactive Simulacra of Human Behavior》的论文，展示了一个由生成代理（Generative Agents）组成的虚拟西部小镇。该小镇由斯坦福与谷歌联合搭建并命名为《Smallville》，在小镇中，首次创造了多个智能体生活的虚拟环境，其内生活着25个可以模拟人类行为的生成式AI Agent。他们会有工作、会八卦、能组织社交等等。Agent具有类似人的特质、独立决策和长期记忆等功能。在小镇设计中，Agent包含了记忆、反思和规划三大要素。

图9 生成式Agents架构

在上述架构中，Agent感知其环境，并将所有感知保存在称为记忆流的Agent经验的综合记录中。基于他们的感知，该架构检索相关的记忆，并使用这些检索到的行动来确定另一个行动。这些检索到的记忆也被用来形成长期的计划和创建更高级别的反思，这些都被输入到记忆流中以供未来使用。

## AI Agent挑战与展望

本文概述了Agent，并总结了Agent包含LLM、规划、感知、记忆和行动组件的概念。在研究过程中，我们也发现了AI Agent框架当前面临的一些挑战：

  * 大模型可靠性不足，能力边界不确定，AI Agent对于复杂任务处理不够理想。如处理长期规划和任务拆解时表现欠佳、开销极高等。
  * AI Agent交互的灵活稳定性。在目前的框架中，系统经常会交互信息设计导致系统不稳定，例如身份信息错乱，AI Agent间能力差异导致对话失败等问题。
  * 多模态支持，包括AI Agent间交互方式以及环境设置等。目前学术界对大都是对还是通过将其他模态信息转换成文字信息然后交给LLM处理，在针对一些模态的转换，转换成文字，可能会存在时延、信息损失等问题（机器人）。
  * 如何让开发者、甚至普通用户，能够方便快捷的创建适用于自己的AI Agent，打造个人专属助手。



AI Agent是释放LLM潜力的关键，也是通往AGI关键路线。虽然目前AI Agent尚处于起步阶段，但随着AI Agent不断发展，投入AI Agent究的科研人员，创业者越来越多，AI Agent也必将迎来高速发展，未来是AI Agent的世界。我们认为，AI Agent框架后续的发展趋势，将会关注由任务驱动，AI Agent定制的容易度，多AI Agent管理、交互，人机交互，结构化通信等领域。

## Agent盘点

### [MetaGPT](https://zhuanlan.zhihu.com/p/668550781)

MetaGPT是一个开源多智能体框架，其实现将高级任务分解为由不同角色（产品经理、架构师、项目经理、工程师、QA工程师）处理的详细可操作组件的能力，从而促进特定角色的专业知识和协调。

  * Meta-GPT分析参见文章：[【AI Agent洞察】02-MetaGPT：面向编程的多智能体框架](https://zhuanlan.zhihu.com/p/668550781)
  * GitHub地址：[https://github.com/geekan/MetaGPT](https://link.zhihu.com/?target=https%3A//github.com/geekan/MetaGPT)



### [Auto-GPT](https://zhuanlan.zhihu.com/p/668234147)

Auto-GPT是Github上的一个免费开源项目，结合了GPT-4和GPT-3.5技术。其是一个具有长时记忆、能自主迭代、自我提示且联网查询的新的GPT 框架。

  * Auto-GPT分析参见文章：[【AI Agent洞察】03-AutoGPT：以ChatGPT为核心的自治AI智能体](https://zhuanlan.zhihu.com/p/668234147)
  * GitHub地址：[https://github.com/Significant-Gravitas](https://link.zhihu.com/?target=https%3A//github.com/Significant-Gravitas)



### [BabyAGI](https://zhuanlan.zhihu.com/p/669412115)

BabyAGI只是一个精简架构，能够根据预定义的目标自动生成任务、确定任务的优先级和执行任务。其任务规划完全依赖ChatGPT，复杂任务规划存在缺陷，且工具的集成使用能力弱。

  * BabyAGI分析参见文章：[【AI Agent洞察】04-BabyAGI：实现任务自驱动的智能体](https://zhuanlan.zhihu.com/p/669412115)
  * GitHub地址：[https://github.com/yoheinakajima/babyagi](https://link.zhihu.com/?target=https%3A//github.com/yoheinakajima/babyagi)



### [Generative Agents：Smallville](https://zhuanlan.zhihu.com/p/670665088)

Smallville描述了一个虚拟小镇，小镇中包含25个虚拟人物，皆以Agent为基础构造。Generative Agents背后是一个新的智能体架构，在该架构中，Agents系统共分为感知、记忆和行动三大模块。

  * Smallville分析参见文章：[【AI Agent洞察】05-Smallville：基于生成式智能体构建的虚拟社会](https://zhuanlan.zhihu.com/p/670665088)



### [HuggingGPT](https://zhuanlan.zhihu.com/p/673691859)

HuggingGPT是一个使用ChatGPT作为任务规划器的框架，其通过LLM协调专家模型共同完成复杂的AI任务，主要特点在于大模型和经典模型的协同。HuggingGPT包含四个步骤：任务规划、模型选择、任务执行和响应生成。

  * Auto-GPT分析参见文章：[【AI Agent洞察】06-HuggingGPT：面向多模态、多领域提供专业化解决复杂AI任务的智能体](https://zhuanlan.zhihu.com/p/673691859)
  * GitHub地址：[https://github.com/microsoft/JARVIS](https://link.zhihu.com/?target=https%3A//github.com/microsoft/JARVIS)



### [Camel](https://zhuanlan.zhihu.com/p/675696232)

Camel是一个生成式AI工具，使用户能够通过角色扮演，使多个智能体能够进行对话并合作解决分配的任务。

  * Auto-GPT分析参见文章：[【AI Agent洞察】07-CAMEL：开创了沟通式智能体，多个Agent角色通过自主沟通完成任务](https://zhuanlan.zhihu.com/p/675696232)
  * GitHub地址：[https://github.com/camel-ai/camel](https://link.zhihu.com/?target=https%3A//github.com/camel-ai/camel)



### [XAgent](https://zhuanlan.zhihu.com/p/681136067)

XAgent是一个开源、基于大型语言模型（LLM）的通用自主智能体，可以自动解决各种复杂任务。XAgent采用双环机制，外循环用于高层任务管理，起到规划（Planning）的作用，内循环用于底层任务执行，起到执行（Action）的作用。

  * XAgent分析参见文章：[【AI Agent洞察】10-XAgent：采用双循环运转机制，自主解决复杂任务的通用智能体](https://zhuanlan.zhihu.com/p/681136067)
  * GitHub地址：[https://github.com/OpenBMB/XAgent](https://link.zhihu.com/?target=https%3A//github.com/OpenBMB/XAgent)



### [Agents](https://zhuanlan.zhihu.com/p/682883749)

Agents是一个通用的Agent框架，其具有易定制、易调整等特点。相较于其他AI Agent而言，其在通用性基础上，通过SOP机制建立了可控智能体的新范式，使其具备对智能体行为的细粒度控制。

  * Agents分析参见文章：[【AI Agent洞察】11-Agents：首个采用SOP机制建立可控机制的智能体](https://zhuanlan.zhihu.com/p/682883749)
  * GitHub地址：[https://github.com/aiwaves-cn/agents](https://link.zhihu.com/?target=https%3A//github.com/aiwaves-cn/agents)



### AutoGen

Autogen是微软推出的一个Multi-Agent框架，允许用户创建和管理多个智能体，以协同完成复杂的任务。Autogen具有可定制和可对话的能力，同时支持人类输入和工具扩展其能力。

  * Autogen分析参见文章：[【AI Agent洞察】12-AutoGen：通过多Agent对话来构建LLM应用的智能体](https://zhuanlan.zhihu.com/p/685103609)
  * GitHub地址：[https://github.com/microsoft/autogen](https://link.zhihu.com/?target=https%3A//github.com/microsoft/autogen)



### [LangChain](https://zhuanlan.zhihu.com/p/665717963)

LangChain是一个帮助开发者基于大语言模型开发AI应用程序的框架。

  * [【入门LangChain系列】01-LangChain介绍](https://zhuanlan.zhihu.com/p/665717963)
  * [【入门LangChain系列】02-Model输入输出](https://zhuanlan.zhihu.com/p/669795717)
  * [【入门LangChain系列】03-Chain](https://zhuanlan.zhihu.com/p/676911584)
  * [【入门LangChain系列】04-Agent](https://zhuanlan.zhihu.com/p/679896083)
  * [【入门LangChain系列】05-索引和记忆](https://zhuanlan.zhihu.com/p/681771807)
  * [小白学LangChain：LangChain表达式语言（LCEL）](https://zhuanlan.zhihu.com/p/658836911)



更多关于AI Agent的分析，后续将会持续更新中~~

## 相关链接

  1. Lilian Weng. LLM Powered Autonomous Agents
  2. Wang, L., C. Ma, X. Feng, et al. A survey on large language model based autonomous agents. CoRR, abs/2308.11432, 2023.
  3. Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, et al. The rise and potential of large language model based agents: A survey. arXiv preprint arXiv:2309.07864, 2023.
  4. 东方证券AI Agent基于大模型的自主智能体在探索AGI的道路上前进



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【Agent综述】02-多Agent：完成复杂任务的利器，迈向AGI时代](https://zhuanlan.zhihu.com/p/32708623482)

## 参考

  1. ^Schlosser, M. Agency. In E. N. Zalta, ed., The Stanford Encyclopedia of Philosophy. Metaphysics Research Lab, Stanford University, Winter 2019 edn., 2019. <https://plato.stanford.edu/entries/agency/>
  2. ^abZhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, et al. The rise and potential of large language model based agents: A survey. arXiv preprint arXiv:2309.07864, 2023
  3. ^Lilian Weng. LLM Powered Autonomous Agents
  4. ^Wang, L., C. Ma, X. Feng, et al. A survey on large language model based autonomous agents. CoRR, abs/2308.11432, 2023.


