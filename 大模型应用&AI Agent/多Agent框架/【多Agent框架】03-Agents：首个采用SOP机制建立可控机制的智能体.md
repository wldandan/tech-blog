# 【多Agent框架】03-Agents：首个采用SOP机制建立可控机制的智能体

原文链接：https://zhuanlan.zhihu.com/p/682883749

---

​

目录

## Agents简介

随着大语言模型的爆火，使得人们看见了一个通往AGI的方向。基于[LLM](https://zhida.zhihu.com/search?content_id=239882096&content_type=Article&match_order=1&q=LLM&zhida_source=entity)所具备的意图识别解释、生成计划并自主行动的能力，人们相继构建出自主的智能体。智能体以LLM作为大脑，通过与环境、人类和其他智能体的交互，可以自主的解决各种复杂的任务。

Agents是一款开源的通用多智能体，包含规划、记忆、工具、多智能体通信和符号计划（SOPs，也称为标准作业程序）等功能。Agents具有如下功能：

  * **长短期记忆** ：Agents集成了记忆组件，并使用向量数据库和语义搜索使智能体能够存储和检索长期记忆，并定期使用便签存储器更新短期工作记忆。用户只需在配置文件中填写字段，就可以选择为智能体配备长期记忆、短期记忆或两者兼备。
  * **工具使用和网络导航** ：Agents支持一些常用的外部API，并提供一个抽象类，使开发人员能够轻松集成其他工具。同时，还支持通过将Web搜索和网络导航定义为专门的API，使智能体能够浏览互联网并收集信息。
  * **多智能体通信** ：Agents支持单智能体和多智能体，其多智能体通信采用“动态调度”的能力。动态调度不是根据硬编码规则安排智能体的行动顺序，而是提供了定义控制器智能体的选项，控制器智能体充当“协调者”，根据其角色和当前情况决定哪个智能体执行下一个动作。
  * **人机交互** ：Agents无缝支持单智能体和多智能体场景下的人机交互，使一个或多个人能够与智能体进行沟通和交互。
  * **可控性** ：Agents通过符号计划（也称为标准作业程序SOPs）提供了一种建立可控智能体的新范式。Agents中的SOP是一组详细记录、循序渐进的说明，概述了特定任务或过程应该如何由一个智能体或多个智能体执行。符号计划的引入提供了对智能体行为的细粒度控制，使智能体的行为更加稳定/可预测，并同时便于调整/优化Agent。



图1 Agents的工作流程图

## Agents设计

Agents主要包含Agent和环境两个类，此外还包含一个用于符号计划的类，称为SOP（标准作业程序），使智能体具有更好的可控性。上述的主要类通过配置文件初始化，配置文件可以用纯文本填写。Agents初始化和运行（多）智能体系统的典型脚本示例如代码1所示。配置文件不仅定义了这些核心对象，还将复杂的提示分解为模块化的提示组件。提示的分解显著降低了用户构建（多）智能体系统所需的专业知识和工作量。

代码1：使用Agents初始化和运行（多）智能体系统的示例代码
    
    
    def main ()
        # agents is a dict of one or multiple agents .
        agents = Agent . from_config ("./ config . json ")
        sop = SOP . from_config ("./ config . json ")
        environment = Environment . from_config ("./ config . json ")
        run ( agents , sop , environment )

### Agent

Agent类有观察环境（agent._observe(environment)）、根据当前状态行动（agent._act()）和更新记忆（agent._update_memory()）等方法。上述方法都封装在agent.step()方法中，方便开发人员轻松地自定义智能体的新功能。同时，Agents还有一个“_is_user”属性，如果将其设置为“True”，Agents将选择向人类用户提供观察和记忆信息，并等待人类用户输入操作。上述设计允许在单个智能体和多个智能体系统中灵活进行人机交互，允许人类用户扮演一个或多个角色。

### SOP

SOP类包含智能体状态的图形表示。每个状态指定了SOP描述的任务中所有智能体的某个子任务或子目标。状态被抽象为State类，State对象包含智能体在状态中利用LLM和各种工具或API所使用的模块化提示。

SOP对象还包括一个基于LLM的控制函数，用于决定不同状态之间的转换和下一个要执行动作的智能体。状态转移函数命名为sop._transit()，智能体路由函数命名为sop._route()。这两个函数都封装在sop.next()函数中，该函数在主循环中使用。

图2 客户服务Agent--------------------------------图3 销售Agent

  


### 环境

环境类（Environment）抽象了Agents所处的环境。环境包含两个主要函数：environment._observed()和environment.update()。environment._observed()定义了环境对Agent行动的影响（即在观察时应向Agent传递哪些信息），而environment.update()定义了Agent行动对环境的影响。

图4: 多智能体系统

基于Agents的（多）智能体系统的执行逻辑非常直观。如代码2所示，在每次迭代中，SOP首先根据Agent和环境决定状态转换，并选择下一个要执行动作的Agent。然后Agent根据其状态和环境采取行动，环境根据新的行动更新自身。最后，如果工作流需要根据中间执行结果动态调整计划，可以解析操作的输出，定义一个新的状态并将其添加到当前SOP中。

代码2：AGENTS中（多）智能体系统的运行循环示例代码
    
    
    def run ( agents , sop , environment ) :
        while not sop . finished :
            agent , state = sop . step ( agents , environment )
            action = agent . step ( state , environment )
            environment . update ( agent , action )
            # optional , in case of dynamic planning
            # new_states = get_new_states ( action )
            # sop . add_states ( new_states )

### 核心功能的实现

**长短期记忆**

在Agents中，长期记忆是动作历史，通过句子转换器进行嵌入，存储在[VectorDB](https://zhida.zhihu.com/search?content_id=239882096&content_type=Article&match_order=1&q=VectorDB&zhida_source=entity)中，并通过语义搜索进行查询。短期记忆，即工作记忆，以自然语言形式存在，并通过LLM更新。

**工具使用和网页导航**

Agents通过ToolComponents支持工具使用和网页导航。对于每个外部工具或API，开发者可以将API调用包装在ToolComponent.func()方法中。对于那些API调用与上下文相关的复杂工具，Agents集成了[OpenAI GPT API](https://zhida.zhihu.com/search?content_id=239882096&content_type=Article&match_order=1&q=OpenAI+GPT+API&zhida_source=entity)的“函数调用”功能，让LLMs决定如何使用这些工具。通过将网页搜索实现为一种专门的工具，实现了网页导航功能。

**多智能体通信**

Agents包含一个控制器函数，该函数通过考虑之前的行动、环境和当前状态的目标，使用LLM动态决定哪个智能体将执行下一个行动，使得多智能体通信更加灵活。

**人机交互**

通过允许人类用户在配置文件中将某个智能体的“is_user”字段更改为“True”，支持多智能体系统中的人机交互。由此，用户可以自己扮演智能体的角色，输入自己的行动并与环境中的其他智能体进行交互。

## Agents总结

Agents是一个通用的Agent框架，其具有易定制、易调整等特点。相较于其他AI Agent而言，其在通用性基础上，通过[SOP机制](https://zhida.zhihu.com/search?content_id=239882096&content_type=Article&match_order=1&q=SOP%E6%9C%BA%E5%88%B6&zhida_source=entity)建立了可控智能体的新范式。使其具备对智能体行为的细粒度控制，保障智能体的行为更加稳定/可预测，并同时便于调整/优化智能体。虽然Agents的概念相对齐全，但是整体实现简单，还是属于学术界仿真的阶段。

## 参考资料

  1. W. Zhou, Y. E. Jiang, L. Li, J. Wu, T. Wang, S. Qiu, J. Zhang, J. Chen, R. Wu, S. Wang, S. Zhu, J. Chen, W. Zhang, N. Zhang, H. Chen, P. Cui, and M. Sachan. Agents: An open-source framework for autonomous language agents, 2023b.
  2. [https://github.com/aiwaves-cn/agents](https://link.zhihu.com/?target=https%3A//github.com/aiwaves-cn/agents)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】02-CAMEL：开创了沟通式智能体，多个Agent角色通过自主沟通完成任务](https://zhuanlan.zhihu.com/p/675696232)  
> 下一篇：[【多Agent洞察】04-AutoGen：通过多Agent对话来构建LLM应用的智能体](https://zhuanlan.zhihu.com/p/685103609)
