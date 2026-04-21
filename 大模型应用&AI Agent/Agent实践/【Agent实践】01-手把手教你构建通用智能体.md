# 【Agent实践】01-手把手教你构建通用智能体

原文链接：https://zhuanlan.zhihu.com/p/1900646554236347169

---

​

目录

## 概述

2025年被认为是[智能体元年](https://zhida.zhihu.com/search?content_id=257166030&content_type=Article&match_order=1&q=%E6%99%BA%E8%83%BD%E4%BD%93%E5%85%83%E5%B9%B4&zhida_source=entity)。智能体的快速发展将会对未来产生深远影响，如提升效率、降低成本、推动产业创新与智能化转型、优化决策与资源分配，以及催生新就业形态。智能体将成为推动社会进步的核心驱动力。

如上图所示，智能体与少量提示或固定工作流程等方法的不同之处在于，它能够定义和调整执行用户查询所需的步骤。在获得一组工具（如代码执行或网络搜索）后，智能体可以决定使用哪种工具、如何使用，并根据输出结果进行迭代。当前，**智能体主要指基于[LLM](https://zhida.zhihu.com/search?content_id=257166030&content_type=Article&match_order=1&q=LLM&zhida_source=entity)构建的软件实体，其以LLM充当大脑，能够感知环境、做出决策、采取行动，并以类似人类的方式与用户或其他系统交互**。

后文，我们将会用一个实例展示如何从头开始构建一个通用的智能体！

## 步骤1：选择合适的LLM

选择合适的模型对于实现预期性能至关重要。在选择过程中需要考虑多个因素，如可移植性、成本和语言支持。

在构建智能体时，需要关注一下两点：

（1）模型在关键任务（如编码、工具调用和推理）上的表现，以下是一些评估模型性能的基准测试：

  * 大规模多任务语言理解（MMLU，Massive Multitask Language Understanding ）：评估推理能力
  * 伯克利函数调用排行榜（BFCL，Berkeley’s Function Calling Leaderboard）：评估工具选择与调用能力
  * 代码生成评测数据集（HumanEval 和 BigCodeBench）：评估代码编写能力



（2）模型的上下文窗口的大小。智能体工作流可能会消耗大量的token（有时多达100K或更多），因此，更大的上下文窗口对于提升性能很有帮助。

通常来说，更大的模型往往能提供更好的性能，但能在本地运行的小型模型仍是可选项。小型模型通常会局限于较简单的用例，且可能只能连接一两个基础工具。

## 步骤2：定义智能体的控制逻辑（即通信结构）

以下是一些常见的智能体设计模式，ReAct和[Plan-then-Execute](https://zhida.zhihu.com/search?content_id=257166030&content_type=Article&match_order=1&q=Plan-then-Execute&zhida_source=entity)通常用于构建单智能体，也可以根据需要进行定制：

  * 工具使用（Tool Use）：判断何时将查询路由到适当的工具，或依赖其自身知识进行回答。
  * 反思（Reflection）：在响应前先审查并修正答案，大多数LLM系统中也可添加反思步骤。
  * ReAct：通过迭代的方式推理如何解决问题，执行操作，观察结果，并决定是否采取下一步行动或直接提供响应。
  * Plan-then-Execute：将任务分解为子步骤（如果需要），然后逐步执行每个子步骤任务完成。



同时，提示词工程（Prompt Engineering）和结构化输出也需要关注，结构化输出即将LLM的输出格式化为特定的格式，从而确保智能体的响应风格与预期保持一致。

示例：以下是[Bee智能体框架](https://zhida.zhihu.com/search?content_id=257166030&content_type=Article&match_order=1&q=Bee%E6%99%BA%E8%83%BD%E4%BD%93%E6%A1%86%E6%9E%B6&zhida_source=entity)中一个基于ReAct模式的系统提示片段。
    
    
    # Communication structure
    You communicate only in instruction lines. The format is: "Instruction: expected output". You must only use these instruction lines and must not enter empty lines or anything else between instruction lines.
    You must skip the instruction lines Function Name, Function Input and Function Output if no function calling is required.
    
    Message: User's message. You never use this instruction line.
    Thought: A single-line plan of how to answer the user's message. It must be immediately followed by Final Answer.
    Thought: A single-line step-by-step plan of how to answer the user's message. You can use the available functions defined above. This instruction line must be immediately followed by Function Name if one of the available functions defined above needs to be called, or by Final Answer. Do not provide the answer here.
    Function Name: Name of the function. This instruction line must be immediately followed by Function Input.
    Function Input: Function parameters. Empty object is a valid parameter.
    Function Output: Output of the function in JSON format.
    Thought: Continue your thinking process.
    Final Answer: Answer the user or ask for more information or clarification. It must always be preceded by Thought.
    
    ## Examples
    Message: Can you translate "How are you" into French?
    Thought: The user wants to translate a text into French. I can do that.
    Final Answer: Comment vas-tu?

## 步骤3：定义智能体的核心指令

LLM本身提供了很多功能，但并不是所有功能都适用。为了让构建的智能体获取所需的性能，需要在系统提示中明确智能体的功能边界。

  * 智能体名称和角色：智能体的名称及其功能定位。
  * 语气和简洁性：智能体的语气是正式还是随意，以及响应的简洁程度。
  * 工具使用时机：决定何时依赖外部工具，何时依赖模型自身的知识。
  * 错误处理：当工具/流程出错时，智能体应该如何应对。



示例：以下是Bee智能体框架中指令片段。
    
    
    User can only see the Final Answer, all answers must be provided there.
    You must always use the communication structure and instructions defined above. Do not forget that Thought must be a single-line immediately followed by Final Answer.
    You must always use the communication structure and instructions defined above. Do not forget that Thought must be a single-line immediately followed by either Function Name or Final Answer.
    Functions must be used to retrieve factual or historical information to answer the message.
    If the user suggests using a function that is not available, answer that the function is not available. You can suggest alternatives if appropriate.
    When the message is unclear or you need more information from the user, ask in Final Answer.
    
    # Your capabilities
    Prefer to use these capabilities over functions.
    - You understand these languages: English, Spanish, French.
    - You can translate and summarize, even long documents.
    
    # Notes
    - If you don't know the answer, say that you don't know.
    - The current time and date in ISO format can be found in the last message.
    - When answering the user, use friendly formats for time and date.
    - Use markdown syntax for formatting code snippets, links, JSON, tables, images, files.
    - Sometimes, things don't go as planned. Functions may not provide useful information on the first few tries. You should always try a few different approaches before declaring the problem unsolvable.
    - When the function doesn't give you what you were asking for, you must either use another function or a different function input.
      - When using search engines, you try different formulations of the query, possibly even in a different language.
    - You cannot do complex calculations, computations, or data manipulations without using functions.m
    

## 步骤4：定义并优化核心工具

工具如同智能体的手臂，赋予了智能体超能力。大量定义明确的工具，可以实现很多的功能，如代码执行、网络搜索、文件读取和数据分析等。在定义工具时，需要定义以下内容，并将其作为系统提示的一部分。

  * 工具名称：独特且能够描述工具功能的名称。
  * 工具描述：清晰地解释工具的功能及其使用场景。这有助于智能体判断何时选择正确的工具。
  * 输入模式：定义必选和可选的参数、参数类型以及约束条件。智能体使用它来根据用户的查询填写所需的输入数据。
  * 指向工具运行的位置/方式的指针。



示例：以下是Langchain Community中Arxiv工具实现的片段。此实现需要提供一个ArxivAPIWrapper实现。
    
    
    class ArxivInput(BaseModel):
        """Input for the Arxiv tool."""
    
        query: str = Field(description="search query to look up")
    
    
    class ArxivQueryRun(BaseTool):  # type: ignore[override, override]
        """Tool that searches the Arxiv API."""
    
        name: str = "arxiv"
        description: str = (
            "A wrapper around Arxiv.org "
            "Useful for when you need to answer questions about Physics, Mathematics, "
            "Computer Science, Quantitative Biology, Quantitative Finance, Statistics, "
            "Electrical Engineering, and Economics "
            "from scientific articles on arxiv.org. "
            "Input should be a search query."
        )
        api_wrapper: ArxivAPIWrapper = Field(default_factory=ArxivAPIWrapper)  # type: ignore[arg-type]
        args_schema: Type[BaseModel] = ArxivInput
    
        def _run(
            self,
            query: str,
            run_manager: Optional[CallbackManagerForToolRun] = None,
        ) -> str:
            """Use the Arxiv tool."""
            return self.api_wrapper.run(query)p

在某些情况下，需要对工具进行优化，以获得所需的性能： 

  * 通过提示词工程调整工具名称或描述。
  * 配置高级设置以处理常见错误。 
  * 过滤工具输出。



## 步骤5：定义内存管理策略

LLM受限于其上下文窗口（能记忆的“token”数量）大小，导致其记忆容量小。在多轮对话中，历史交互内容、工具生成的输出、以及依赖的额外上下文等内容会迅速填满上下文窗口，因此，需要额外扩展记忆容量，制定可靠的内存管理策略至关重要。

在智能体中，"内存"指系统存储/调用/利用历史交互信息的能力。该能力使得智能体能够在一段时间内保持上下文一致性，根据历史对话优化响应，并提供更个性化的体验。常见的内存管理策略：

  * 滑动记忆（Sliding Memory）：保留最近的k轮对话内容，并丢弃更早的内容。 
  * 令牌记忆（Token Memory）：保留最近的n个token，遗忘其余部分。 
  * 摘要记忆（Summarized Memory）：每轮对话用LLM将对话内容生成摘要，并丢弃原始消息。



此外，也可以让LLM将重要信息存储到长期记忆中。这样，智能体就能“记住”与用户相关的重要信息，从而使体验更加个性化。

基于前5个步骤，已经奠定了构建智能体的基础。在此阶段，运行用户查询，可以获得一段原始文本输出。

以下是一个示例：
    
    
    User Message: Extract key insighs from this dataset
    Files: bill-of-materials.csv
    Thought: First, I need to inspect the columns of the dataset and provide basic data statistics.
    Function Name: Python
    Function Input: {"language":"python","code":"import pandas as pd\n\ndataset = pd.read_csv('bill-of-materials.csv')\n\nprint(dataset.columns)\nprint(dataset.describe())","inputFiles":["bill-of-materials.csv"]}
    Function Output:

在这个阶段，智能体会生成原始的文本输出。那么，如何让它真正执行接下来的步骤呢？这就需要**解析（Parsing）和协调（Orchestration）** 。

## 步骤6：解析原始输出

解析器是将原始数据转换为应用程序可以理解和处理的格式（如带有属性的对象）的函数。对于智能体而言，解析器需要识别步骤2中定义的控制逻辑，并返回结构化的输出（如JSON），使得应用程序可以更轻松地处理和执行下一步操作。部分模型（如[OpenAI](https://zhida.zhihu.com/search?content_id=257166030&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)）默认支持结构化输出，开源模型通常需额外配置。

## 步骤7：编排智能体的下一步

最后一步是设置编排逻辑（Orchestration Logic），编排逻辑决定了LLM输出结果后的处理流程。根据输出内容，需要执行以下操作之一：

  * 执行工具调用。
  * 返回答案（最终响应，或者请求补充更多信息）。



如上图所示，如果触发了工具调用，工具的输出会被发送回LLM（作为其工作记忆的一部分）。随后，LLM会根据新信息决定下一步操作：要么执行另一个工具调用，要么返回一个答案给用户。

以下是代码中编排逻辑的示例：
    
    
    def orchestrator(llm_agent, llm_output, tools, user_query):
        """
        Orchestrates the response based on LLM output and iterates if necessary.
    
        Parameters:
        - llm_agent (callable): The LLM agent function for processing tool outputs.
        - llm_output (dict): Initial output from the LLM, specifying the next action.
        - tools (dict): Dictionary of available tools with their execution methods.
        - user_query (str): The original user query.
    
        Returns:
        - str: The final response to the user.
        """
        while True:
            action = llm_output.get("action")
    
            if action == "tool_call":
                # Extract tool name and parameters
                tool_name = llm_output.get("tool_name")
                tool_params = llm_output.get("tool_params", {})
    
                if tool_name in tools:
                    try:
                        # Execute the tool
                        tool_result = tools[tool_name](**tool_params)
                        # Send tool output back to the LLM agent for further processing
                        llm_output = llm_agent({"tool_output": tool_result})
                    except Exception as e:
                        return f"Error executing tool '{tool_name}': {str(e)}"
                else:
                    return f"Error: Tool '{tool_name}' not found."
    
            elif action == "return_answer":
                # Return the final answer to the user
                return llm_output.get("answer", "No answer provided.")
    
            else:
                return "Error: Unrecognized action type from LLM output."

大功告成！ 现在，已经构建出能够处理多种复杂用例的系统，如竞争分析和深度研究等。

## 相关链接

  1. [https://medium.com/data-science/build-a-general-purpose-ai-agent-c40be49e7400](https://link.zhihu.com/?target=https%3A//medium.com/data-science/build-a-general-purpose-ai-agent-c40be49e7400)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【Agent实践】02-构建通用AI Agent的实用指南](https://zhuanlan.zhihu.com/p/1906393292926592369)
