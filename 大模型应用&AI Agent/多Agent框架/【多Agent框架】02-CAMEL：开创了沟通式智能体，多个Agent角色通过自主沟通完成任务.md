# 【多Agent框架】02-CAMEL：开创了沟通式智能体，多个Agent角色通过自主沟通完成任务

原文链接：https://zhuanlan.zhihu.com/p/675696232

---

​

目录

## [CAMEL](https://zhida.zhihu.com/search?content_id=238284776&content_type=Article&match_order=1&q=CAMEL&zhida_source=entity)简介

近年，随着[大语言模型](https://zhida.zhihu.com/search?content_id=238284776&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)的快速发展，它们的能力不断的扩展延伸，不仅能生成和理解文本，还能进行复杂的分析和推理，在解决复杂任务方面更是取得了显著进展。然而，他们在解决复杂任务时，很大程度上还是依赖于人类的输入—Prompt，这也给人类带来巨大的挑战和成本。

CAMEL，是Communicative Agents for “Mind” Exploration of Large Scale Language Model Society的缩写，大意为大规模语言模型社会的“心智”探索通讯智能体。

**CAMEL开创了沟通式智能体的探索** ，提出了一个名为**“角色扮演”（Role-Playing）的新型社会型多智能体框架** 。其**通过多个智能体之间进行对话和合作来完成分配的任务** 。智能体会被赋予不同的角色，并拥有相应角色的专业知识背景，从而利用其知识来找到满足共同任务的解决方案。该框架使用启发式提示（Inception Prompt）来引导聊天智能体完成任务，并与人类意图保持一致。

图1 CAMEL整体方案

在上述方案中，我们假设设置AI用户智能体和[AI助手智能体](https://zhida.zhihu.com/search?content_id=238284776&content_type=Article&match_order=1&q=AI%E5%8A%A9%E6%89%8B%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)两种：

  * AI用户智能体：向AI助手提供指令，目标是完成任务，AI助手和AI用户将通过多轮对话合作完成指定任务，直到AI用户确定任务已完成。
  * AI助手智能体：遵循AI用户的指令，并以解决任务的方法进行回应，AI用户负责向 AI助手发出指令，并将对话引导向任务完成。



CAMEL贡献点如下：

  1. 引入了一种新的沟通式智能体（通过角色扮演实现），让智能体在最少的人工干预下通过自主合作，共同完成任务；
  2. 提供了一种为研究多智能体合作行为和能力的可扩展方法，阐述了实现自主合作的挑战，并提供了应对这些挑战的策略。
  3. 开源了CAMEL智能体，其中包含各种智能体的实现、数据生成流程、数据分析工具和收集的数据集，以支持沟通时智能体等领域的研究及更多其他相关领域的研究。



## CAMEL实现原理

### 角色扮演框架

CAMEL专注于以任务为导向的角色扮演，包含一个AI助理（AI Assistant）和一个AI用户（AI User）。在多智能体系统接收到人类用户的初步想法和角色分配后，任务指定智能体将提供详细的描述，使想法更加具体化。然后，AI助理和AI用户将通过多轮对话合作完成指定的任务，直到AI用户确定任务完成为止。一方面，AI用户负责向AI助理提供指令，并引导对话朝着任务完成的方向进行；另一方面，AI助理则需要遵循AI用户的指示，做出回答并提供具体的解决方案。完整的角色扮演框架如图2所示。

图2 CAMEL角色扮演框架

**人类输入和任务说明**

角色扮演会话将从人类的想法和选择的角色开始。如图2中的所示，人类想开发一个用于股市的交易机器人，然后选择了一个Python程序员和一个股票交易员两个角色。在确定了想法和角色之后，任务指定智能体将根据输入的想法，为AI助理角色提出一个具体的任务，以帮助AI用户角色完成任务。引入任务指定智能体主要原因为对话智能体通常需要一个具体的任务提示来实现任务，而对于非领域专家来说，这可能具有挑战性或费时。

**AI助理—用户角色分配**

在任务制定之后，将会把AI助理角色和AI用户角色分别分配给用户代理和助理代理，以完成指定的任务。在实践中，系统消息会传递给每个智能体，声明他们的角色。将AI助理的提示/消息称为p_{A}，AI用户的提示/消息为 p_{u} 。在对话开始之前，系统消息会传递给智能体。我们用F_{1}和F_{2}分别表示两个大规模自回归语言模型。当系统消息分别传递给这些模型时，得到 A\leftarrow F_{1}^{p_{A}}  和 U\leftarrow F_{2}^{p_{u}}  ，分别称为AI助理代理和AI用户代理。在图2中，AI助理和AI用户在角色扮演会话开始时，分别被分配为Python程序员和股票交易员的角色。AI用户充当任务规划者，参与交互式规划，确定AI助理执行的步骤是否可行。与此同时，AI助理充当任务执行者，提供解决方案，执行计划的步骤，并向AI用户提供响应。

**任务为导向的对话**

在角色分配完成后，AI助理和AI用户将以指令的方式合作完成任务。形式上，用 I_{t} 表示在时间 t 内获得的AI用户指令，用 S_{t} 表示AI助理的解决方案。截止时间t获得的对话消息集合表示如下所示：

M_{t}=\left\\{ \left( I_{0},S_{0} \right),...,\left( I_{t},S_{t} \right) \right\\}=\left\\{ \left( I_{i},S_{i} \right) \right\\}|_{i=0}^{t} (1)

在下一个时间 t+1 时，AI用户获取历史对话消息集合 M_{t} 并提供一个新的指示 I_{t+1} 。生成的指示消息I_{t+1}随后与消息集合M_{t}一起传递给AI助理 A 。AI助理将以S_{t+1}作为解决方案进行响应。随后，消息集合也会更新成M_{t+1}：

I_{t+1}=u\left( M_{t} \right) (2) 

S_{t+1}=A\left( M_{t},I_{t+1} \right) (3)

M_{t+1}\leftarrow M_{t}\cup\left( I_{t+1},S_{t+1} \right) (4)

**评审者参与**

为了增强角色扮演框架的可控性，引入了一个[评审者智能体](https://zhida.zhihu.com/search?content_id=238284776&content_type=Article&match_order=1&q=%E8%AF%84%E5%AE%A1%E8%80%85%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)，能够从角色扮演智能体中选择建议或提供反馈。使得任务解决过程中能够进行类似搜索树的决策。在实践中，批评者可以是一个AI智能体或一个人类。

### 启示式提示

对于角色扮演的沟通式智能体，提示工程至关重要。在CAMEL中，提示工程仅在角色扮演的开始阶段，用于任务规范和角色分配。一旦进入对话阶段，AI助理和AI用户将自动重复为对方生成提示，直到任务完成后终止。也因此，作者们将其技术称为启示式提示（Inception Prompting）。Inception提示包括三个提示：任务指定器提示 p_{t} ，AI助理系统提示 p_{A} 和AI用户系统提示p_{u}。以AI Society场景的Inception提示为例，如图3所示。任务指定器提示包含有关AI助理和AI用户在角色扮演会话中的角色的信息。因此，任务指定器代理可以将初步的任务/想法作为输入，并生成具体的任务。AI助理系统提示p_{A}和AI用户系统提示示p_{u}。大多是对称的，包括分配的任务和角色的信息、通信协议、终止条件以及避免不必要行为的约束或要求。

图3：AI Society角色扮演的Inception提示

**提示工程**

AI 助理系统提示示例如下：

  * 记住你是<ASSISTANT_ROLE> ，我是<USER_ROLE>。将选定的角色分配给AI助理智能体，并提供有关AI用户角色的信息。
  * 不要交换角色！不要指导我！这样可以防止代理之间交换角色。
  * 如果由于身体、道德、法律原因或能力问题，你无法执行我的指示，你必须诚实地拒绝并解释原因。从而禁止智能体生成有害、虚假、非法和误导性信息。
  * 除非我说任务已完成，你应该始终以“解决方案：…”开头。
  * 始终以“下一个请求”结束你的解决方案。这确保AI助理通过请求新的指示来继续对话。



对于AI用户系统提示，除了相反的角色分配，用户系统提示与助理提示在以下方面存在差异：

  * 你必须通过以下两种方式之一来指导我完成任务：1. 使用必要的输入进行指导：...；2. 不使用任何输入进行指导：...。这遵循指令的典型数据结构，允许生成的指令—解决方案对易于微调LLMs。
  * 请一直给我提供指示和必要的输入，直到你认为任务已完成。当任务完成时，你只能回复一个单词“完成”。



## CAMEL总结

CAMEL开创了沟通式智能体的探索，通过角色扮演的方式相互交流，使得智能体能够在最小化人工干预的情况下自主合作完成任务。同时，经过详尽的评估，往往可以得到更好的解决方案。然而，CAMEL仍面临“角色翻转、无意义的重复对话消息”这些比较困难的挑战，仍需后续不断的探索。

## CAMEL体验

通过conda and pip安装CAMEL。

  1. 创建一个conda虚拟环境；


    
    
    conda create --name camel python=3.10

2\. 激活camel conda环境；
    
    
    conda create --name camel python=3.10

3\. 克隆GitHub仓库；
    
    
    git clone -b v0.1.0 https://github.com/camel-ai/camel.git

4\. 进入项目目录；
    
    
    cd camel

5\. 从源代码安装CAMEL；
    
    
    pip install -e .

6\. 或者如果你想使用 "huggingface agent"，通过如下方式安装。
    
    
    pip install -e .[huggingface-agent] # (Optional)

## 参考资料

  1. Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. Camel: Communicative agents for" mind" exploration of large scale language model society. arXiv preprint arXiv:2303.17760, 2023. 3, 5, 13
  2. [https://github.com/camel-ai/camel](https://link.zhihu.com/?target=https%3A//github.com/camel-ai/camel)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】01-MetaGPT：面向编程的多智能体框架](https://zhuanlan.zhihu.com/p/668550781)  
> 下一篇：[【多Agent洞察】03-Agents：首个采用SOP机制建立可控机制的智能体](https://zhuanlan.zhihu.com/p/682883749)
