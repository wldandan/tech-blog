# 【多Agent框架】12-PydanticAI关键技术分析

原文链接：https://zhuanlan.zhihu.com/p/1899832254022263668

---

​

目录

## 1 概述

[PydanticAI](https://zhida.zhihu.com/search?content_id=257050892&content_type=Article&match_order=1&q=PydanticAI&zhida_source=entity)是一款由Pydantic团队开发的Python Agent框架。其旨在通过类型安全、[结构化验证](https://zhida.zhihu.com/search?content_id=257050892&content_type=Article&match_order=1&q=%E7%BB%93%E6%9E%84%E5%8C%96%E9%AA%8C%E8%AF%81&zhida_source=entity)和依赖注入等功能，帮助开发者简化生产级大模型应用的开发。PydanticAI的亮点在于无缝结合了Pydantic，Pydantic是一个强大的Python库，用于轻松验证和解析数据。它能够确保数据的准确性并符合预期结构，尤其适用于处理诸如JSON文件、用户输入或API响应等外部数据。

## 2 PydanticAI架构

PydanticAI通过集成Pydantic和[logfire](https://zhida.zhihu.com/search?content_id=257050892&content_type=Article&match_order=1&q=logfire&zhida_source=entity)，形成核心竞争力： 

1、Pydantic：提供运行时全流程数据校验能力，并降低生产环境下因数据类型异常导致的难以定位的问题的概率。

2、logfire（可选扩展）：提供可观测能力，包括状态监控、报告生成、性能监控、错误分析等能力，比如可以自动记录Pydantic报告的校验失败原因。 

3、PydanticAI，包含两个大模块：

  * pydantic-ai-slim：以Agent作为大模型交互入口，提供工具执行、Prompt框架、输出校验、上下文管理等能力。
  * pydantic-graph：一个通用的[有限状态机](https://zhida.zhihu.com/search?content_id=257050892&content_type=Article&match_order=1&q=%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA&zhida_source=entity)的图执行器，基于类型注解，支持调度器外置。



## 3 PydanticAI关键功能

### 3.1 类型安全

基于Pydantic，PydanticAI提供对LLM输出的强类型校验功能，能够确保模型返回的数据结构与预期完全一致。 

  * 通过严格的类型检查，在代码编写阶段提前发现潜在的类型不匹配等问题。 
  * 对输入和输出的数据进行严格的类型验证，确保程序运行中数据的一致性和完整性。 
  * 支持自动将外部数据（如 JSON、字典）转换为定义好的数据模型，并在此过程中进行验证。



### 3.2 集成Pydantic Logfire

集成Logfire工具，为大模型应用提供实时调试、性能监控与问题定位能力，助力开发者快速排查问题并优化系统性能。

### 3.3 结构化响应

借助Pydantic对静态和流式响应进行验证，确保PydanticAI每次运行得到的响应都具有一致的结构。 - 预定义数据模型：定义模型输出的数据结构。 - 自动验证与类型转换：自动验证模型的输出是否符合预定义的数据模型。 - 结构化错误处理：如果模型的输出不符合预定义的数据模型，PydanticAI会捕获异常并提供清晰的错误信息，便于调试和处理。

### 3.4 **流式响应**

PydanticAI支持流式响应的生成与验证。这种方式特别适用于如实时聊天、流式文本生成或逐块处理数据，能够利用这一功能一边接收数据，一边校验其合法性，从而提升整体性能。

### 3.1 **依赖注入系统**

提供可选的依赖注入系统，为Agent的系统提示、工具和结果验证器提供数据和服务。该能力对于测试和以评估为驱动的迭代开发很有帮助。例如把它想象成一个“智能管家”，帮你管理各种数据和服务，让你的Agent在不同的场景下都能高效工作。

### 3.1 **[Graph支持](https://zhida.zhihu.com/search?content_id=257050892&content_type=Article&match_order=1&q=Graph%E6%94%AF%E6%8C%81&zhida_source=entity)**

PydanticAI通过使用类型提示定义图，助力开发者能够以模块化和结构化的方式定义、验证和执行复杂的数据处理流程或AI工作流。Graph由几个关键部分组成： - 图形运行上下文：类似于PydanticAI的RunContext。它保存了图形的状态和依赖关系，并在节点运行时传递给节点。 - 结束：表示图形运行结束。 - 节点：图中的执行节点，一般由如下部分组成。 - 包含调用节点时所需/可选参数的字段。 - 在run方法中执行节点的业务逻辑。 - 返回run方法的注解，这些注解由pydantic-graph读取，用于确定节点的出边。 - 图形：执行图本身，由一组节点类（即BaseNode子类）组成。

## 4 PydanticAI使用入门

  1. 使用pip安装PydanticAI。  
pip install pydantic-ai 
  2. (可选)如果需要安装Logfire，使用如下命令安装。  
pip install 'pydantic-ai[logfire]
  3. 为LLM提供商设置API令牌。Pydantic可直接与OpenAI和VertexAI等协同工作。 
  4. 创建PydanticAI智能体。这段代码使用PydanticAI实现了一个银行支持智能体。


    
    
    from dataclasses import dataclass
    
    from pydantic import BaseModel, Field
    
    from pydantic_ai import Agent, RunContext
    
    
    class DatabaseConn:
        """This is a fake database for example purposes.
    
        In reality, you'd be connecting to an external database
        (e.g. PostgreSQL) to get information about customers.
        """
    
        @classmethod
        async def customer_name(cls, *, id: int) -> str | None:
            if id == 123:
                return 'John'
    
        @classmethod
        async def customer_balance(cls, *, id: int, include_pending: bool) -> float:
            if id == 123 and include_pending:
                return 123.45
            else:
                raise ValueError('Customer not found')
    
    
    @dataclass
    class SupportDependencies:
        customer_id: int
        db: DatabaseConn
    
    
    class SupportResult(BaseModel):
        support_advice: str = Field(description='Advice returned to the customer')
        block_card: bool = Field(description='Whether to block their card or not')
        risk: int = Field(description='Risk level of query', ge=0, le=10)
    
    
    support_agent = Agent(
        'openai:gpt-4o',
        deps_type=SupportDependencies,
        result_type=SupportResult,
        system_prompt=(
            'You are a support agent in our bank, give the '
            'customer support and judge the risk level of their query. '
            "Reply using the customer's name."
        ),
    )
    
    
    @support_agent.system_prompt
    async def add_customer_name(ctx: RunContext[SupportDependencies]) -> str:
        customer_name = await ctx.deps.db.customer_name(id=ctx.deps.customer_id)
        return f"The customer's name is {customer_name!r}"
    
    
    @support_agent.tool
    async def customer_balance(
        ctx: RunContext[SupportDependencies], include_pending: bool
    ) -> str:
        """Returns the customer's current account balance."""
        balance = await ctx.deps.db.customer_balance(
            id=ctx.deps.customer_id,
            include_pending=include_pending,
        )
        return f'${balance:.2f}'
    
    
    if __name__ == '__main__':
        deps = SupportDependencies(customer_id=123, db=DatabaseConn())
        result = support_agent.run_sync('What is my balance?', deps=deps)
        print(result.data)
        """
        support_advice='Hello John, your current account balance, including pending transactions, is $123.45.' block_card=False risk=1
        """
    
        result = support_agent.run_sync('I just lost my card!', deps=deps)
        print(result.data)
        """
        support_advice="I'm sorry to hear that, John. We are temporarily blocking your card to prevent unauthorized transactions." block_card=True risk=8
        """

## 5 总结

PydanticAI是一个将大语言模型与数据验证、结构化处理相结合的工具。其基于Pydantic框架构建，旨在提升大模型在复杂任务中的可靠性和准确性。通过Pydantic的强大数据验证功能，PydanticAI能够严格定义输入输出的数据格式，将大语言模型的自由生成能力与结构化数据处理结合起来，确保模型生成的结果符合预期格式并避免出错。同时，它还能对多步骤任务中的中间结果进行校验，减少错误传播的可能性，使其特别适用于需要高可靠性和精确性的应用场景，如自动化任务、工具调用或系统集成等。PydanticAI的出现为开发者提供了一种更稳健的方式来构建基于大模型的应用程序。

## 6 相关链接

  1. [https://ai.pydantic.dev/](https://link.zhihu.com/?target=https%3A//ai.pydantic.dev/)
  2. [https://github.com/pydantic/pydantic-ai](https://link.zhihu.com/?target=https%3A//github.com/pydantic/pydantic-ai)
  3. [https://medium.com/data-science-in-your-pocket/pydanticai-pydantic-ai-agent-framework-for-llms-6c60c86e0e48](https://link.zhihu.com/?target=https%3A//medium.com/data-science-in-your-pocket/pydanticai-pydantic-ai-agent-framework-for-llms-6c60c86e0e48)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】11-AWS Multi-agent Orchestrator技术分析](https://zhuanlan.zhihu.com/p/1893252158327063392)  
> 下一篇：[【多Agent洞察】13-多智能体设计：通过优化提示词与拓扑结构优化智能体系统](https://zhuanlan.zhihu.com/p/1910732602626797678)
