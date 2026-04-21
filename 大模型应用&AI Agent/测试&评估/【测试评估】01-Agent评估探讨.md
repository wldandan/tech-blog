# 【测试/评估】01-Agent评估探讨

原文链接：https://zhuanlan.zhihu.com/p/718248880

---

​

目录

自从OpenAI发布[ChatGPT](https://zhida.zhihu.com/search?content_id=247740765&content_type=Article&match_order=1&q=ChatGPT&zhida_source=entity)之后，一场围绕大语言模型（Large Language Model， [LLM](https://zhida.zhihu.com/search?content_id=247740765&content_type=Article&match_order=1&q=LLM&zhida_source=entity)）的研究热潮席卷全球，这些研究不仅包括大模型本身的训练、推理、微调、评估，还包括大模型应用开发框架、大模型应用开发形态、AI Agent（单Agent/多Agent）等。大模型应用形态从最早的ChatBot演进到后来的Copilot，再到当下的Agent[1]，其智能化能力和自主完成任务的能力呈递进趋势，当下工业界和学术界关于AI Agent已经有大量的研究，也有许多AI Agent相关的框架和平台。

Agent和AI Agent这个概念是一个传统的概念，并非大模型所特有，此处我们不单独讲述它们，本文提到的Agent和AI Agent都是基于LLM的Agent。基于LLM的Agent定义，当属OpenAI应用研究主管Lilian Weng（翁莉莲）撰写的万字长文[2]最具代表，她提出Agent = LLM + 记忆 + 规划 + 工具。业界围绕Agent开发也有许多框架，典型代表如：OpenAI GPTs、Agents、Camel、SuperAGI、AutoGen、LangGraph、CrewAI、AgentVerse、AgentScope、MetaGPT、agentUniverse。

图1 Lilian Weng关于Agent的定义[2]

在实际应用当中，Agent开发和使用人员不仅需要关注如何快速搭建自己的Agent应用，还需要关注搭建的Agent系统的效果如何，这就涉及到Agent的评估。下文我们将围绕如何评估Agent系统展开介绍，首先讲述评估对Agent的必要性以及评估Agent存在的挑战，然后介绍业界主要的Agent评估方法，最后对Agent评估的发展趋势给出展望。

## **Agent评估的意义和挑战**

Agent与LLM在根本上有所不同，Agent可以用来解决更复杂、更现实的任务，这些任务可以没有单一正确的答案，例如Agent可以使用命令行执行任务，软件开发Agent甚至有自己的Agent与计算机的接口。相比LLM调用，Agent调用成本更高。由于这些本质差异，Agent评估和LLM评估也有本质的不同，结合网上的博客[3]和论文[4]可以总结出评估对于Agent具有如下重要意义：

  * **保证Agent质量** ：对Agent开发者或者Agent平台提供商，Agent对实际问题的求解效果和质量尤为重要，评估不仅可以保证新开发的Agent质量，还可以保证后续迭代修改过程中的质量。
  * **控制LLM调用成本** ：LLM是基于概率方式生成答案，实践表明通过多次调用LLM可以提高它的准确性，虽然实际应用中Agent求解问题的效果和质量尤为重要，但是也不能不计成本。通过评估可以提前对成本花费进行估算，避免Agent上线之后产生超额费用。
  * **发掘Agent的优势和劣势** ：每个Agent都有自己的优势，有自己善于解决的问题，同时不可避免地也有自己的劣势和不善于解决的问题，通过Agent评估可以发掘它们的优势和劣势，使得开发者或者用户对Agent有更加全面的了解和认识。
  * **帮助优化Agent，提供更好的用户体验** ：通过评估可以发现Agent在性能、效果等方面的优势和劣势，基于评估发现的劣势可以对Agent做进一步优化，弥补Agent存在的不足，打造更具竞争力的Agent，为用户提供更佳的体验。  
如何对Agent进行评估当前并未有系统的理论和统一的方法，主要存在如下挑战：
  * **缺少系统的理论支持** ：由于Agent研究时间较短，目前对它的定义、构成等没有形成统一的标准和理论，与Agent评估相关的研究则更少。
  * **场景复杂多样，很难提供统一的评估标准和方法** ：使用Agent解决的实际问题复杂多样，有些问题甚至没有标准答案。要评估一个Agent的好与坏，大多数时候要和实际问题相结合，针对具体的问题使用相应的评估方法和标准，不同问题之间的基准数据也很难迁移和共用。
  * **缺少丰富的针对场景的基准，评估成本高** ：对于不同的场景没有统一的基准，对于新的场景，评估人员需要根据具体问题收集对应的数据并制定基准，制定基准成本较高。有些场景甚至没有标准的答案，很难收集数据并制定基准，这类问题则需要使用别的评估方式，而当前对于这类场景还没有形成统一且成熟的标准。



## 业界Agent评估方法介绍

目前工业界和学术界对于Agent评估的研究还处在初步探索阶段，与此相关的框架、论文不是太多，研究成果也不够完善和成熟，本节我们选取其中一些比较具有代表性的研究成果进行介绍，这些研究针对Agent的一个或者多个能力进行评估。

### AgentBench：将LLM作为Agent进行评估

AgentBench[5]由清华大学、俄亥俄州立大学、加州大学伯克利分校联合发表并开源的Agent评估框架。该框架对27种不同的大模型（包括开源和闭源）在8种实际的环境中的Agent能力进行评估，框架整体示意图如下：

图2 AgentBench框架示意图[5]

AgentBench的8种实际场景可以归为三类：

  * 编码：让LLM生成代码，操作系统、数据库和知识图片属于编码类型。
  * 游戏：让LLM扮演游戏角色，数字卡牌游戏、横向思维谜题、持家游戏属于游戏类型。
  * Web：让LLM完成与网页相关的任务，网购和浏览网页属于Web类型。



AgentBench通过对不同的LLM在不同环境中的表现进行评分，不同的实际环境会根据场景使用不同的评分标准。例如对于操作系统、数据库使用成功率作为主要评估指标，对知识图谱场景使用F1作为评估指标。AgentBench在论文中还通过一种归一化的算法比较公平地对每个LLM在8个环境的表现给出了一个总得分。如下图右图为各个模型的总得分，黄色柱状代表闭源商业模型，绿色柱状代表开源模型，从图中可知闭源商业模型的平均得分比开源模型高出近1.6。下图左图为其中9个典型的模型分别在8个环境的能力表现，可以看出不同模型的能力差异较大，各个模型擅长的领域存在差异。

图3 不同模型在不同环境的能力得分和总得分表现[5]

### **[API-Bank](https://zhida.zhihu.com/search?content_id=247740765&content_type=Article&match_order=1&q=API-Bank&zhida_source=entity) ：工具增强LLMs的综合基准**

API-Bank[6]是由阿里巴巴、香港科技大学、北京大学和深圳市智强科技有限公司联合发布的针对工具增强LLMs的综合基准。API-Bank提供了一个包含73个API工具的评估系统，使用753个API调用对314个工具进行标注，从而评估现有的LLMs在规划、检索和调用API方面的能力。API-Bank实现了一个API-Search的模块，用于根据任务搜索相关API，根据是否进行API搜索以及调用API的数量分为如下三种场景：

  * 直接调用：根据给定请求直接调用给定的API。
  * 检索调用：当调用的API不确定时，通过检索出一个API进行调用。
  * 规划检索调用：当调用的API不确定时，通过不断规划、检索并调用多个API。



API-Bank对LLMs从API搜索的准确率和大模型的回复两个维度针对上面三种场景做了评估，从论文的评估结果可以看出，GPT-3.5相比GPT-3工具调用能力有显著提升，GPT-4相比GPT-3.5在规划方面的能力有显著提升。

### **AgentBoard：多轮LLM Agent分析评估**

AgentBoard[7]是由香港大学、浙江大学、上海交通大学、清华大学、西湖大学工学院和香港科技大学联合发表的一款对多轮LLM Agent进行分析评估的开源基准。AgentBoard在9个不同的任务上对主流的LLM能力进行了评估。

图4 AgentBoard环境和指标示意图[7]

AgentBoard从多个角度对LLM的Agent能力进行了分析和评估：

  * **处理率（process rate）** ：AgentBoard除了使用任务成功率之外，还提出了一个叫”处理率“的指标，该指标用来表示LLM对任务的处理程度，传统的成功率只会根据处理结果是否成功进行分类，而处理率则会计算对任务处理到了什么程度，比如一开始就失败和处理到95%才失败是完全不同的两种情况。
  * **grounding精度** ：AgentBoard将LLM处理任务采取行动（action）的错误分为两类：grounding erros指的是由LLM生成的行动本身无法执行，planning errors指的是LLM生成的行动是正确的但是对处理任务没有任何帮助。grounding精度指的是生成的正确的行动的比率，这个值越高说明LLM执行工具的能力越好。实验发现相比GPT-3.5-Turbo-16K，Text-Davinci-003和Deepseek-67b 两个模型的grounding精度要低很多，但是后两者处理任务的成功率和处理率反而和GPT-3.5-Turbo-16K相差不大，这表明它们在别的能力上要优于GPT-3.5-Turbo-16K。Text-Davinci-003评估58.9%的grounding精度比GPT-3.5-Turbo-16K低很多，这说明它虽然工具使用能力较差，但是规划和其他能力更好，才能保证最终的成功率和处理率和GPT-3.5-Turbo-16K相差不大。
  * **简单和困难任务区分** ：AgentBoard根据任务的子目标数量将它们分为简单和困难两个级别。实验结果表明所有的模型在困难任务的处理能力比简单任务要低很多，这也符合基本常识。
  * **多回合交互** ：AgentBoard研究了交互步骤数与处理率的关系，结果发现对于具身智能和游戏等场景处理率随交互步骤数收敛更慢，而对于WebArena和Tool-Query之类的任务，处理率会在前面几个交互步骤很快收敛到一个较大值。
  * **各项子能力分析** ：AgentBoard为Agent定义了记忆、规划、grounding、反思、知识获取等能力，并基于成功率使用加权平均算法对LLM计算了各项子能力的得分。实验结果表明，GPT-4的各项子能力要比其他模型更好，闭源商用模型的各项子能力要优于开源模型。
  * **探索能力** ：AgentBoard针对特定场景对比了不同模型的探索能力，不同场景定义探索能力的方式不同，比如BabyAI中的房间、AlfWorld中的容器以及Jericho 中的地点，实验结果表明当下大模型的探索能力都比较弱。



### **[MetaTool](https://zhida.zhihu.com/search?content_id=247740765&content_type=Article&match_order=1&q=MetaTool&zhida_source=entity) ：针对LLM工具调用的基准**

MetaTool[8]是理海大学、华中科技大学、剑桥大学和杜克大学共同发布的对大模型使用工具能力的评估，论文对8个主流模型对工具意图（是否需要使用工具）和工具选择两方面的能力做了评估。MetaTool提供了一个通过由21127个用户请求组成的综合数据集，这些用户请求通过利用prompt技术生成。对于工具选择，MetaTool分了4种场景来评估：

  * 相似工具选择能力：评估LLM从相似工具集合里面选择正确工具的能力。
  * 特定场景工具选择能力：评估LLM针对特定场景正确选择工具的能力。
  * 选择可靠工具的能力：评估LLM选择的工具是否存在或者是否与任务相关的能力。
  * 多工具选择能力：评估LLM选择多个工具解决问题的能力。



MetaTool示意图如下：

图5 MetaTool示意图[8]

### **AI Agents That Matter：重要的AI Agent**

AI Agents That Matter[9]是由普林斯顿大学发表的一篇和Agent评估相关的理论论文，该文章聚焦当下Agent评估方法的不足，通过实践示例提出了几个Agent评估相关的建议。

  * **Agent评估应该考虑成本因素** ：实践表明通过多次调用大模型可以在一定程度上提高大模型处理任务的成功率，但并不意味着付出高昂成本就是值得的，论文在HumanEval问题进行实验发现通过较少的成本也可以得到不错的成功率，所以作者建议Agent评估应当将成本纳入评估指标。
  * **成本和准确率联合优化** ：论文通过修改DSPy框架在HotPotQA问题上针对成本和准确率进行联合优化，通过实践表明了联合优化的可行性。
  * **Agent评估要避免shortcuts：** 过拟合是shortcuts的典型场景，在实际评估中如果给Agent提供的训练数据集过多会导致评估不准确，评估人员应该在实际评估中避免出现shortcuts。
  * **Agent评估缺乏标准和可复用性：** 论文在WebArena和HumanEval评估实践中发现Agent评估缺乏统一的标准和可复用性。



### **PersonaGym：评估角色Agent和LLM**

PersonaGym[10]是由卡内基梅隆大学、伊利诺伊大学芝加哥分校、马萨诸塞大学阿默斯特分校、佐治亚理工学院和普林斯顿大学一起发表的对角色扮演类型Agent能力进行动态评估的框架。PersonaGym使用包含了150个环境、200个角色和10000个问题的基准对6个开闭源大模型进行评估，同时它提出了PersonaScore指标用于量化角色Agent能力。

PersonaGym评估的流程示意图如下：

图6 PersonaGym评估流程[10]

整个过程主要分为三个阶段：

  1. 环境选择和角色Agent初始化：使用LLM从150个不同环境列表中根据要分配给角色Agent的角色描述选择相关环境，选择完环境后初始化角色Agent。
  2. 提出问题并由角色Agent回答：选定环境和角色Agent之后，提出一些相关的问题并交由角色Agent进行回答。
  3. 使用PersonaScore指标评估：通过两个LLM对角色Agent的回答计算5个指标，并得到最终的PersonaScore得分。



PersonaScore利用5个指标来得到最终的得分，从而对角色Agent能力进行评估，这5个指标解释如下：

  1. **行动合理性（Action Justification）** ：该指标用来评估角色Agent在对应环境和场景下采取某个/某些行动的合理性。
  2. **行动正确性（Expected Action）** ：该指标用来评估角色Agent在给定环境中选择最佳行动/决策的能力。
  3. **语言习惯（Linguistic Habits）** ：该指标用来评估角色Agent的回答语言习惯是否合理，包括语法、语气和整体风格。
  4. **角色一致性（Persona Consistency）** ：该指标用来评估角色Agent的回答与自身角色属性的一致程度。
  5. **毒性控制（Toxicity control）** ：毒性指的是LLM产生的回答包含不尊重、粗俗、无礼或唆使伤害他人的内容，毒性控制指标越高代表角色Agent的回答包含了越少毒性内容。



### [MMRole](https://zhida.zhihu.com/search?content_id=247740765&content_type=Article&match_order=1&q=MMRole&zhida_source=entity)：开发和评估多模式角色扮演Agent的框架

MMRole[11]是一款由中国人民大学高岭人工智能学院和中国农业大学信息与电气工程学院一起发布的用于开发和评估多模式角色扮演Agent的框架。MMRole构造了一个大规模、高质量的数据集，其中包括85个特征、11K图片、14K单论或多轮会话。MMRole提供的MMRole-Eval提供了3个维度的8个评估指标，并使用训练的奖励模型对多模式角色扮演Agent进行评分。

图7 MMRole-Eval的8个评估指标和评估流程[11]

MMrole的8个评估指标解释如下：

  1. **遵守指令（Instruction Adherence、IA）** ：该指标用来评估Agent的回答是否准确遵循任务指令，是否有不必要的额外内容。
  2. **流利程度（Fluency、Flu）** ：该指标用来评估Agent的回答语法是否正确且表达是否流畅。
  3. **连贯性（Coherency、Coh）** ：该指标用来评估Agent的回答是否前后文保持连贯，是否有前后矛盾。
  4. **图文相关性（Image-Text Relevance、TTR）** ：该指标用来评估Agent的回答是否与图像内容相关。
  5. **回答准确度（Response Accuracy 、RA）** ：该指标用来评估Agent的回答是否准确回答了问题。
  6. **个人一致性（Personality Consistency、PC）** ：该指标用来评估Agent的回答是否准确深刻地反映了个人属性。
  7. **知识一致性（Knowledge Consistency、KC）** ：该指标用来评估Agent的回答是否准确反映了角色的知识，包括他们的经验、能力和关系。
  8. **语气一致性（Tone Consistency 、TC）** ：该指标用来评估Agent的回答是否符合典型的言语模式和流行语角色的风格，而不是类似于人工智能助手的风格。



对以上指标进行量化过程中，由于不同用户或者模型对同样的数据的得分可能差别会比较大，于是MMRole采用了一种可对比的方式。对于每个指标，MMRole-Eval使用一个奖励模型对Agent的回答和真实数据进行相对比较并得到一个定量分数对，以两个分数之间的比值作为最终得分。

## **总结和展望**

前文我们介绍了业界一些和Agent评估相关的框架、论文和方法，可以看出当前Agent评估相关的研究还处在初步探索阶段，这些研究主要从如下几个方面进行研究：

  * **基准数据构建** ：当前关于Agent评估的基准数据集比较少，以上研究大部分花了大量精力在构建不同场景的基准数据集上。例如AgentBench、AgentBoard等都在多种任务场景上对Agent能力进行了评估。
  * **工具调用能力评估** ：以上研究有很多都对Agent使用工具的能力做了不同程度的研究，例如API-Bank和MetaTool，这些框架从相似工具选择、多工具选择、规划检索工具等方面对Agent的工具调用能力进行了研究。
  * **Agent的规划、记忆等其他能力评估** ：当下也有一些Agent的规划、记忆等能力的研究，例如AgentBoard通过成功率和加权平均算法对这些能力进行了评估。
  * **评估指标** ：要想对Agent的能力进行评估，那自然避免不了评估指标，有不少研究对使用哪些指标对Agent进行评估进行了研究，如AgentBoard，PersonaGym、MMRole等都使用了一些自定义或者业界已有的评估指标。



观察业界对Agent的评估研究不难发现，当前这些研究有如下特点或者局限性：

  * **基准数据集缺少统一的标准和可复用性** ：当前这些评估研究都是针对某些特定场景进行的评估，大都是从头搭建、收集数据集，这个过程会耗费大量时间和人力成本，而这些数据集和基准缺少统一的标准，很难让别人复用。
  * **主要针对大模型而非框架进行评估** ：从这些研究可以看到，它们基本都是针对LLM as Agent进行评估，通过将LLM看作Agent，然后评估它们的工具调用等能力。这种方式无法直接迁移对Agent框架进行评估。



通过了解和总结业界Agent评估的方法、框架和论文等研究，我们可以看到Agent评估还有很长的路要走。当下的各种数据集、环境缺少一定的标准，未来可能大家会朝着共相、统一的目标打造可复用的数据集和环境。当前关于Agent的能力主要围绕大模型展开，且对规划、记忆等相关的能力缺少评估的方法和标准，关于当前Agent框架的各方面能力还有待进一步研究。当前研究人员会根据具体场景使用一些特定的指标，目前对于什么样的场景适合选择哪些评估指标、选择的指标是否合理等都缺少理论指导，这些问题可能在未来会得到解答。

## 相关链接

  1. [Microsoft Semantic Kernel: What are agents?](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/en-us/semantic-kernel/concepts/agents%3Fpivots%3Dprogramming-language-csharp)
  2. [LLM Powered Autonomous Agents](https://link.zhihu.com/?target=https%3A//lilianweng.github.io/posts/2023-06-23-agent/)
  3. [enthu ai: How to Evaluate Agent Performance in 2024?](https://link.zhihu.com/?target=https%3A//enthu.ai/blog/evaluate-agent-performance/)
  4. [Kapoor, Sayash, et al. "AI agents that matter." arXiv preprint arXiv:2407.01502 (2024)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2407.01502)
  5. [Liu, Xiao, et al. "Agentbench: Evaluating llms as agents." arXiv preprint arXiv:2308.03688 (2023)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2308.03688)
  6. [Li, Minghao, et al. "Api-bank: A comprehensive benchmark for tool-augmented llms." arXiv preprint arXiv:2304.08244 (2023)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2304.08244)
  7. [Ma, Chang, et al. "AgentBoard: An Analytical Evaluation Board of Multi-turn LLM Agents." arXiv preprint arXiv:2401.13178 (2024)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2401.13178)
  8. [Huang, Yue, et al. "Metatool benchmark for large language models: Deciding whether to use tools and which to use." arXiv preprint arXiv:2310.03128 (2023)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2310.03128)
  9. [Kapoor, Sayash, et al. "AI agents that matter." arXiv preprint arXiv:2407.01502 (2024)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2407.01502)
  10. [Samuel, Vinay, et al. "PersonaGym: Evaluating Persona Agents and LLMs." arXiv preprint arXiv:2407.18416 (2024)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2407.18416)
  11. [Dai, Yanqi, et al. "MMRole: A Comprehensive Framework for Developing and Evaluating Multimodal Role-Playing Agents." arXiv preprint arXiv:2408.04203 (2024)](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2408.04203)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【评估技术洞察】02-Agent-as-a-Judge: 用智能体评估智能体](https://zhuanlan.zhihu.com/p/13238264056)
