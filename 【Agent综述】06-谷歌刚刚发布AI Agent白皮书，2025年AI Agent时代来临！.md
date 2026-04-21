# 【Agent综述】06-谷歌刚刚发布AI Agent白皮书，2025年AI Agent时代来临！

原文链接：https://zhuanlan.zhihu.com/p/19466252143

---

​

目录

## 概述

人类能够很好地处理复杂且混乱的模式识别任务。然而，他们通常需要依赖如书籍、Google搜索或计算器等工具来补充知识，然后得出相应的结论。

同理，AI大模型也可以通过训练，学会使用工具来获取实时信息或者提出现实世界的行动建议。例如：大模型可以利用数据库检索工具来查询特定信息，如客户的购买历史，从而提供个性化的购物建议。亦或者，根据用户的查询，大模型可以通过执行API调用，用于发送邮件回复或者代表用户完成财务交易。

为了实现这些功能，**大模型不仅需要能够接入各种外部工具，还需要具备自主规划并执行任务的能力。这种将推理、逻辑以及利用外部信息相结合的能力，构成了AI Agent的概念** 。AI Agent是一种扩展了大模型出厂能力，拥有独立功能的程序。

## 什么是Agent

宽泛地讲，[生成式AI](https://zhida.zhihu.com/search?content_id=252915781&content_type=Article&match_order=1&q=%E7%94%9F%E6%88%90%E5%BC%8FAI&zhida_source=entity) Agent可以被定义为一个应用程序，其通过观察周围世界并使用可用的工具，采取行动来实现目标。

  * AI Agent是自主的，只要提供目标或者实现意图，他们就可以独立行动，无需人类干预。
  * 即使是模糊的人类指令，AI Agent也可以推理下一步应该做什么以实现其最终目标。



在AI领域，AI Agent是一个通用的概念，本章主要基于生成式AI模型实现的AI Agent进行讨论。

为了理解AI Agent的内部工作原理，将会从驱动AI Agent行为、行动和决策的基础组件进行介绍。通过这些组件的组合，可以实现不同的架构。聚焦于核心功能，在AI Agent任职架构中，有三个基本组成部分，如图1所示。

图1 通用Agent架构和组件

### 模型

在AI Agent领域，**模型是指在AI Agent工作流中充当决策者角色的语言模型（LM）** 。

智能体可以使用一个或者多个任意规模（小型/大型）的语言模型，这些模型能够遵从基于指令的推理逻辑，如ReAct、Chain-of-Thought或[Tree-of-Thoughts](https://zhida.zhihu.com/search?content_id=252915781&content_type=Article&match_order=1&q=Tree-of-Thoughts&zhida_source=entity)。

模型可以是通用的、多模态的，或者根据特定Agents框架的需求进行微调。

为了获得最佳的应用效果，应选择和目标应用程序最契合的模型，且最好该模型已针对将在认知架构中使用的工具相关的数据特征进行过训练。例如，可以通过展示Agents能力的示例，包括Agents在各种情景中使用特定工具或者推理步骤的示例，对Agents进行进一步的优化。

### 工具

尽管基础模型在文本和图像生成方面表现出色，但无法与外部世界进行交互，极大地限制了他们的能力。工具解决了这个问题。工具使得Agent能够与外部数据和服务进行互动，大大扩展了他们的行动范围。

工具的形式多样，且复杂度不一，常见的有Web API（如GET、POST、PATCH和DELETE）方法。例如，借助工具可以更新数据库中的客户信息，或者结合获取天气数据的工具，Agents可以为用户提供旅行建议。

通过使用工具，Agents可以访问和处理真实世界的信息，从而使其能够支持更专业系统，如检索增强生成（RAG），显著扩展了Agents的能力。

### 编排

编排层描述了一个循环过程，定义了Agents如何接收信息、进行内部推理，并利用推理信息来指导其下一步行动或决策。一般来说，这个循环将持续进行，直到Agents达到其目标或触发终止条件。

编排层的复杂度会随着Agents执行的任务不同而有很大差异。例如，一些编排可能是简单的计算和决策规则，而其他编排可能包含链式逻辑、额外的机器学习算法，或者其他概率推理技术。

### Agents VS 模型

为Agents和模型之间的区别，请参见下表：

模型| Agents  
---|---  
知识受限于其训练数据。| 通过工具与外部系统连接，能够在模型之前，扩展知识。  
基于用户查询的一次性推理/预测。 除非明确为模型单独实现，否则没有会话历史或持续上下文的管理。| 自动管理的会话历史。基于用户查询自主决策进行多轮推理/预测。  
无工具| 具备支持工具的能力  
无本地逻辑层实现，需要借助提示词或者使用推理框架（如CoT、ReAct等）形成复杂提示，引导模型进行预测| 原生认知架构，内置如CoT、ReAct等推理框架或其他如LangChain等Agents框架。  
  
### 认知架构：Agents如何工作

Agents可以利用认知架构通过处理信息、做出决策，并根据上一轮的输出调整下一个行动，并如此循环迭代，实现最终目标。

在Agents认知架构中，编排是核心，其负责维护记忆、状态、推理和规划。它利用快速发展的提示工程及相关框架来引导推理和规划，使得Agents能够有效地和环境互动并完成任务。下面介绍几种流行的推理技术：

  * ReAct，为语言模型提供了一种思维过程策略，用于根据用户查询进行推理并采取行动。ReAct已被证明优于几种SOTA基准，提高了LLM的人机交互性和可信度。
  * Chain-of-Thought（CoT），通过中间步骤实现推理能力。CoT有各种子技术，包括自我一致性、主动提示和多模态CoT，适用于不同的场景。
  * Tree-of-thoughts（ToT），非常适合探索或战略前瞻任务。它泛化了链式思维提示，并允许模型探索各种思维链，作为解决LLM的通用问题的中间步骤。



Agents可以利用上述其中一种推理技术，或者其他技术，给特定用户的请求选择下一个最佳行动。例如，如下示例使用ReAct。

图2 在编排中使用ReAct推理的Agent示例

如图2所示，模型、工具和Agents配置共同工作，根据用户的输入返回了一个有根据、简洁的相应。虽然模型可以根据其先前知识猜测一个答案（幻觉），但它却使用了一个工具（航班）来搜索实时外部信息，从而能够基于真实数据做出更明智的决策，并将这些信息总结回用户。

总体来讲，Agents的响应质量与模型的推理、执行任务的能力直接相关，包括选择正确的工具、定义工具的好坏。

## 工具：通往现实世界的关键

虽然语言模型擅长处理信息，但它们缺乏直接感知和影响现实世界的能力。这限制了它们在需要与外部系统或数据进行交互时的实用性。在某种程度上，语言模型的表现仅取决于他们训练的数据所覆盖的信息。

那么，如何赋予模型与外部系统实时、能够感知上下文的交互能力呢？[Extension](https://zhida.zhihu.com/search?content_id=252915781&content_type=Article&match_order=1&q=Extension&zhida_source=entity)s、Functions、[Data Stores](https://zhida.zhihu.com/search?content_id=252915781&content_type=Article&match_order=1&q=Data+Stores&zhida_source=entity)和Plugins为模型提供了这一关键能力。

### Extension

Extension是一种以标准化方式连接AI Agent和API的组件，该方式使得智能体能够无缝执行各种API，而无需考虑其底层实现方式。假设你开发了一个智能体帮助用户预订航班。你打算使用谷歌航班API来获取航班信息，但不确定如何让你的智能体调用这个API。

图3 Agent如何和外部API交互

一种方法是通过实现自定义代码，将接收到的用户查询进行解析以提取相关信息，然后发起API调用。例如，用户输入 “I want to book a flight from Austin to Zurich”； 那么，代码需要从中提取“Austin”和“Zurich”作为相关信息，然后才能进行API调用。但如果用户输入“I want to book a flight to Zurich”，代码就无法获得出发城市信息，进而无法成功调用API。因此，这种方法维护性和扩展性都很差。

一种更为灵活的方法是使用Extension，Extension通过以下方式连接Agent和API：

  1. 通过示例代码指导Agent如何使用API接口。
  2. 指导Agent了解成功调用API接口所需的参数。



图4 通过Extension连接外部API

Extension可以独立于Agent进行构建，但应作为Agent配置的一部分来提供。Agent在运行时，使用模型和示例来决定哪个Extension（如果有的话）适合解决用户的查询。这凸显了Extension的一个关键优势，即它们内置了示例类型，使得Agent能够根据任务动态选择最适合的Extension。

图5 Agent、Extension和API之间的一对多关系

**Extension示例**

以Google的Code Interpreter extension作为例子，从自然语言描述生成和运行Python代码。
    
    
    import vertexai
    import pprint
    
    PROJECT_ID = "YOUR_PROJECT_ID"
    REGION = "us-central1"
    
    vertexai.init(project=PROJECT_ID, location=REGION)
    
    from vertexai.preview.extensions import Extension
    
    extension_code_interpreter = Extension.from_hub("code_interpreter")
    CODE_QUERY = """Write a python method to invert a binary tree in O(n) time."""
    
    response = extension_code_interpreter.execute(
        operation_id = "generate_and_execute",
        operation_params = {"query": CODE_QUERY}
        )
    
    print("Generated Code:")
    pprint.pprint({response['generated_code']})
    
    # The above snippet will generate the following code.
    ```
    Generated Code:
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right

输出如下： 
    
    
    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right
    
    def invert_binary_tree(root):
        """Inverts a binary tree."""
        if not root:
            return None
        # Swap the left and right children recursively
        root.left, root.right = invert_binary_tree(root.right), invert_binary_tree(root.left)
        return root
    
    # Example usage:
    # Construct a sample binary tree
    root = TreeNode(4)
    root.left = TreeNode(2)
    root.right = TreeNode(7)
    root.left.left = TreeNode(1)
    root.left.right = TreeNode(3)
    root.right.left = TreeNode(6)
    root.right.right = TreeNode(9)
    
    # Invert the binary tree
    inverted_root = invert_binary_tree(root)

### Functions

在软件工程领域，函数被定义为完成特定任务的代码模块，并可以根据需要重复使用。当软件开发人员编写程序时，他们通常会创建许多函数来执行各种任务，并定义何时调用function_a、何时调用function_b的逻辑，以及预期的输入和输出。

在AI Agent领域，函数的工作方式与之类似，区别点在于这里使用模型取代了软件开发人员的角色。模型可以访问一组已知函数，并根据函数的定义来决定何时调用哪个函数以及需要传递的参数。

函数与Extension在几个方面存在差异，最显著的差异如下：

  1. 模型会输出一个函数及其参数，但不实时调用API。
  2. 函数在客户端执行，而Extension在Agent端执行。



图6 函数如何和外部API交互

大多数开发人员倾向于使用功能函数，原因如下：

  * 应用程序堆栈的另一层需要进行API调用，这超出了Agent体系架构的范围（例如中间件系统、前端框架等）
  * 存在的安全性或认证限制，可能会导致Agent无法直接调用API（例如API未暴露在互联网上，或者Agent基础设施无法访问）
  * 存在的时间或操作顺序的限制，可能会使得智能体无法实时调用API（例如批处理操作、需要人工审核等）
  * 需要对Agent无法执行的API响应应用进行额外的数据转换逻辑。例如，一个API没有提供用于限制返回结果数量的过滤机制，在客户端，使用函数为开发人员提供这些转换的额外机会。
  * 开发人员希望在智能体开发过程中进行迭代，而无需为API端部署额外基础设施（即函数调用可以充当API的“存根”）。



图7 客户端、Agent端对Extension和函数的调用

**Function示例**

定义Function。
    
    
    def display_cities(cities: list[str], preferences: Optional[str] = None):
        """Provides a list of cities based on the user's search query and preferences.
    
        Args:
            preferences (str): The user's preferences for the search, like skiing, beach, restaurants, bbq, etc.
            cities (list[str]): The list of cities being recommended to the user.
    
        Returns:
            list[str]: The list of cities being recommended to the user.
        """
        return cities

接下来，初始化模型和工具，然后将用户的查询和工具传递给模型。执行下面的代码将产生如代码段底部所示的输出。
    
    
    from vertexai.generative_models import GenerativeModel, Tool, FunctionDeclaration
    
    model = GenerativeModel("gemini-1.5-flash-001")
    display_cities_function = FunctionDeclaration.from_func(display_cities)
    tool = Tool(function_declarations=[display_cities_function])
    
    message = "I’d like to take a ski trip with my family but I’m not sure where to go. "
    res = model.generate_content(message, tools=[tool])
    
    print(f"Function Name: {res.candidates[0].content.parts[0].function_call.name}")
    print(f"Function Args: {res.candidates[0].content.parts[0].function_call.args}")
    
    > Function Name: display_cities
    > Function Args: {'preferences': 'skiing', 'cities': ['Aspen', 'Vail', 'Park City']}

### Data stores

大语言模型的仅保存了初始的训练数据，这在现实世界知识不断演进的场景下，带来了数据无法实时更新的挑战。Data stores的引入解决该限制，其提供了信息的动态更新能力。

图8 Agent如何与结构化和非结构化数据交互

Data stores允许开发人员以原始格式向Agent提供额外的数据，从而无需再进行数据转换、模型重新训练或微调。Data stores将传入的文档转换为一组向量数据库embedding，Agent可以使用这些embedding来提取所需的信息，以补充其下一个动作或对用户的响应，使得模型的返回更相关，更具时效性。

图9 Data stores将Agent连接到各种类型的新实时数据

**实现和应用**

在生成式AI Agent场景，Data stores通常被实现为一种向量数据库，其以向量embedding的形式存储数据，这是一种高维向量或数学表示。近年来，在语言模型中最典型的Data stores使用案例就是RAG。RAG应用程序旨在通过让模型访问各种格式的数据来扩展模型知识的广度和深度，例如：

  * 网站内容
  * 结构化数据，如PDF、Word文档、CSV、电子表格等
  * 非结构化数据，如HTML、PDF、TXT等



图10 Agent和Data-stores之间的一对多关系，Data-stores可以代表各种类型的预索引数据

每个用户请求和Agent响应循环的基本过程如图11所示。

  1. 用户查询发送到embedding模型，生成查询的embedding表示。
  2. 使用相似度匹配算法，将查询embedding和向量数据库的内容进行匹配。
  3. 以文本格式将相似的最高的内容发送回Agent。
  4. Agent接收检索到的内容，然后制定响应和动作。
  5. 将最终相应发送给用户。



图11 基于RAG的应用程序中用户请求和Agent响应的生命周期

如图12所示，展示了一个基于RAG，通过ReAct推理/规划的Agent示例。

图12 基于RAG的应用程序，采用ReAct进行规划

### 工具总结

Extension、Function和Data Storage，每种工具都有其自己的用途，可以由Agent开发人员自行决定是单独使用还是结合使用。

| Extension| Function| Data Storage  
---|---|---|---  
执行| Agent端执行。| 客户端执行。| Agent端执行。  
使用案例| • 开发人员希望Agent控制API的调用。• 使用原生预构建的扩展时有用（例如：Vertex Search、Code Interpreter等）。• 多跳规划和API调用（即下一个动作取决于前一个动作/API调用的输出）。| • 安全性或认证限制，导致Agent无法直接调用API。• 时间或操作顺序的限制，导致智能体无法实时调用API（例如批处理操作、需要人工审核等）。• API未暴露在公网，只能内部使用。| 开发人员希望使用以下数据类型实现RAG。• Website Content from pre-indexed domains and URLs。• 结构化数据，如PDF、 Word文档、CSV、电子表格等。• 关系型/非关系型数据库。• 非结构化数据，如HTML、PDF、TXT等。  
  
## 总结

本文讨论了生成式AI Agent的基础构建模块、它们的组成以及在认知架构形式中有效实施它们的方式。本文的一些关键要点包括：

  1. Agent利用工具来扩展语言模型的的能力，如访问实时信息、提出现实世界行动建议、规划和自主执行复杂任务等。Agent可以利用一个或多个语言模型来决定何时以及如何通过各种状态转换，并使用外部工具完成任意数量的复杂任务，这些任务对于模型自身来说可能很难或不可能完成。
  2. Agent的核心是编排层，这是一种认知架构，它构建了推理、规划、决策并指导其行动。各种推理技术，如ReAct、Chain-of-Thought和Tree-of-Thoughts，为编排层提供了一个框架，用于接收信息、进行内部推理，并生成决策或响应。
  3. 工具，作为Agent通往外部世界的关键，使得Agent能够与外部系统进行交互，并让模型获取其训练数据之外的知识，工具包含如Extension、Function和Data Stores。Extension为Agent与外部API之间提供了一个桥梁，使得Agent能够实现API的调用和实时信息的检索。Function使得Agent能够生成在客户端执行的函数参数。Data Stores为Agent提供了访问结构化或非结构化数据的能力，使数据驱动的应用程序成为可能。



冰冻三尺，非一日之寒，构建复杂的Agent架构需要持续的迭代。本文对Agent进行了初步的探索，Agent的未来将会是激动人心的。随着工具变得更加复杂，推理能力得到增强，Agent将被赋予解决越来越复杂问题的能力。此外，“Agent Chain”也将是一个战略性方向。通过结合各种在特定领域或任务中表现出色的专业Agent，可以创建一个“混合Agent专家”的方法，能够在各行业和问题领域提供出色的结果。

## 相关链接

  1. 原文地址：[https://www.kaggle.com/whitepaper-agents](https://link.zhihu.com/?target=https%3A//www.kaggle.com/whitepaper-agents)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Agent综述】05-2023年人工智能体(AI Agent)开发与应用全面调研：概念、原理、开发、应用、挑战、展望](https://zhuanlan.zhihu.com/p/676288250)  
> 下一篇：[【Agent综述】07-AI Agent的基础设施](https://zhuanlan.zhihu.com/p/1908124676749791413)
