# 【记忆】04-Letta，为AI智能体解锁记忆潜能的开发框架

原文链接：https://zhuanlan.zhihu.com/p/29629889237

---

​

目录

## 1 概述

在不断发展的AI开发领域，创建具有持久记忆和高级推理能力的Agent变得越来越重要。Letta（前身为[MemGPT](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=MemGPT&zhida_source=entity)）是一个由初创公司推出的开创性开源框架，用于创建具有记忆功能的[大型语言模型](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）应用，它能为AI Agent增加长期记忆和高级推理能力，使其能够在长时间对话中保持连贯性和一致性。

Letta的实现思路是采用操作系统内存管理的思想解决Agent与LLMs会话中的长期记忆问题。通过引入[虚拟上下文管理](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=%E8%99%9A%E6%8B%9F%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AE%A1%E7%90%86&zhida_source=entity)技术，解决了上下文长度的限制问题；通过[状态管理](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86&zhida_source=entity)，帮助用户创建有状态（即有记忆）的LLM Agent，从而让Agent保持相对较长的记忆。

有状态的Agent，在LLM Agent与外界交互时，它会积累状态，包括学习到的行为、环境事实和过往交互的记忆，可以有效地管理这些不断增长的知识，在整合新体验的同时保持始终如一的行为。

## 2 Letta核心能力

  * **有状态的LLMs** ：Letta为大型语言模型（LLMs）添加了长期记忆和推理功能，使其能够在不同会话中保留信息，从而提高上下文理解能力。
  * **内存架构** ：在MemGPT的支持下，Letta的记忆系统允许LLMs存储和检索过去的交互或外部数据，根据以前的上下文优化响应（类似于知识图谱或持久记忆数据库）。
  * **自定义模型** ：支持与任何LLM集成（如[GPT-3](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=GPT-3&zhida_source=entity)、GPT-4等），使其能够适应各种用例和模型偏好，而无需与特定框架绑定。
  * **Agent开发环境（ADE）** ：基于图形用户界面的工具，用于创建、编辑和管理LLM Agent。开发人员可在一个集中式平台上设计工作流程、定义内存行为和调试Agent任务。
  * **[Letta Server](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=Letta+Server&zhida_source=entity)** ：通过RESTful API管理应用程序状态、内存和API交互，为AI应用程序（如网络应用程序、移动应用程序）提供动力。它可处理会话连续性和后台进程。



## 3 Letta框架关键技术

Letta前身为MemGPT，MemGPT现已成为Letta的一部分，是支持Letta内存管理功能的关键技术。它解决了对话式人工智能的一个关键局限，即无法在长时间对话中保持对话的连贯性。本部分将会对MemGPT的基础架构和关键技术进行介绍。

### 3.1 [MemGPT框架架构](https://zhida.zhihu.com/search?content_id=254948502&content_type=Article&match_order=1&q=MemGPT%E6%A1%86%E6%9E%B6%E6%9E%B6%E6%9E%84&zhida_source=entity)

MemGPT的**多级内存结构** 包含**主要上下文** （类似于主存/物理内存/RAM）和**外部上下文** （类似于磁盘内存/磁盘存储）两级。主要上下文由LLM组成提示令牌。外部上下文是指保存在LLM固定上下文窗口之外的任何信息。

在MemGPT中，一个固定上下文的LLM处理器由**分层存储系统 + 用于管理记忆的functions执行，有限上下文窗口 + 无限外部记忆** 构成。

  * LLM的主要上下文由系统指令、工作上下文和 FIFO队列组成。
  * 函数调用：MemGPT使用functions在主要上下文和外部上下文（存档和检索存储数据库）之间传递数据。
  * 函数链：LLM可以通过在其输出中生成一个特殊的关键字参数（request heartbeat=true）来请求立即进行后续的LLM推理，以将函数调用串联在一起；这种函数串联允许MemGPT进行多步检索，以回答用户查询。
  * 输出：大模型的输出LLM Completion Tokens，表现为一次函数调用（包括调用工具、搜索历史记录、回信息给用户等）。



  * **System Instruction** ：该部分内容相对固定，主要包含Agent基本设定，memory编辑功能特性说明，以及可以调用的tool工具等。
  * **Working Context** ：也称为Core Memory或In-Context memory，用于保存Agent的记忆块。
  * **FIFO Queue** ：主要是包含最近的对话message记录。
  * **Archival Storage** ：Archival Storage可视作为Core memory的补充，提供了“无限空间“。
  * **Recall Storage** ：将历史过往的message对话记录全部留存保存下来，以便Agent随时进行检索调用。
  * **Function Executor** ：即通过Function Executor来调用memory编辑管理工具。
  * **Queue Manager** ：内置的message queue管理工具。



### 3.2 主要上下文&外部上下文

MemGPT的**多级内存结构** 包含**主要上下文** （类似于主存/物理内存/RAM）和**外部上下文** （类似于磁盘内存/磁盘存储）两级。主要上下文是LLM在推理期间使用的固定长度上下文窗口，而外部上下文包含可以通过显式函数调用选择性复制到主上下文中的信息。

  * **主要上下文** ：LLM prompt tokens = 系统指令 + 工作上下文 + FIFO队列
    * 系统指令（只读）：告诉大模型控制流，不同层级的记忆，以及如何使用functions去调用记忆。
    * 工作上下文（读/写）：定长，用于存储重要事实、偏好、persona、用户信息。
    * FIFO队列：存储历史消息，包括用户、agent、系统提示、function call的结果；其中第一条信息是循环更新的被“踢出”队列的消息们的总结。
  * **外部上下文** ：必须显示调用function才能被LLM processor看到并处理。



Letta借鉴操作系统的多层内存结构实现虚拟内存管理，从而实现Letta多层记忆的管理。

  * **内部记忆（主要上下文，类比物理内存）** ： 提供即时访问的能力。  

    * **核心记忆core memory（有限长度）** ：核心内存单元保存在初始系统指令文件中，并且总是在上下文中可用（Agent将在任何时候看到它）。无搜索核心内存的功能，因为它总是在上下文窗口（在初始系统消息中）中可见。核心记忆主要包含系统指令，agent persona，用户信息。
      * <persona>：存储当前角色的详细信息，指导您的行为和响应，有助于Agent在互动中保持一致性和个性。
      * <human> ：存储使用者信息，允许进行更个性化的、像朋友一样的对话。
      * 核心记忆的增改均由LLM调用functions实现，增加核心记忆（core_memory_append），替换核心记忆（core_memory_replace）。
  * **外部记忆（类比磁盘）** ：持久化存储，提供分页访问的能力。  

    * **归档记忆 archival memory（无限长度，重要的消息）** ：保存在即时上下文之外，必须显示调用检索/搜索操作以查看，归档记忆的读写均由LLM调用functions实现。一个更结构化和更深层次的存储空间，用于存储你的回应、见解或任何其他不适合放入核心记忆，但足够重要的记忆，不会只留给“召回记忆”。
    * **召回记忆 recall memory（无限长度，所有对话历史消息）** ：在即时上下文中只能看到最近的消息，必须显示调用搜索操作从“recall memory”数据库中搜索完整消息历史记录。召回记忆的读由LLM调用functions实现。召回记忆的写为自动保存，由内部Queue Manager来实现。



### 3.3 队列管理器

  * **上下文限制的认知** ：在自我编辑机制中，意识到上下文（即对话内容）限制是关键。MemGPT通过向处理器发出有关令牌限制的警告来指导其记忆管理决策。这意味着模型会注意到每次对话输入和输出的长度限制，从而做出更智能的管理决策。
  * **记忆检索机制的设计** ：MemGPT的记忆检索机制设计时考虑到了这些令牌限制，通过分页（Pagination）来防止检索调用超出上下文窗口。  

    * **warning token count** ：当FIFO队列超出70%长度时，Queue Manager会将“memory pressure”系统警告放入队列，告知LLM尽快调用MemGPT functions将FIFO队列中的重要信息存入工作上下文中(working context)或存档(archival storage)。
    * **flush token count** ：当FIFO队列达到100%长度时，Queue Manager会主动将其中50%的信息踢出队列并存入recall storage，并使用原有的总结 + 新的被提出的消息 生成一个新的总结。



### 3.4 函数执行器

MemGPT允许LLM自己生成函数调用来实现主要上下文和外部上下文之间的数据移动。实现方式是在prompt中告诉LLM：1)记忆分层和每层的作用，2)每个函数的schema。

函数执行器总共有6个核心功能，其中1个是与用户交互，另外5个都是处理记忆。

  * send_message：给用户发一条显示的信息（Letta的其他任何想法和动作都对用户不可见）
  * core_memory_append
  * core_memory_replace
  * archival_memory_insert
  * archival_memory_search
  * conversation_search



### 3.5 控制流与工具链

MemGPT实现了类似操作系统的事件循环和中断处理，允许LLM在返回之前链接多个函数，从而平滑地交错LLM处理、内存管理和用户交互。

  * **事件触发LLM推理** ：在MemGPT中，事件是触发LLM（大型语言模型）推理的触发器。
  * **事件类型** ：
    * **用户消息** ：用户在聊天应用中的消息。
    * **系统消息** ：如主要上下文容量警告。
    * **用户交互** ：例如用户登录或上传文档的提醒。
    * **定时事件** ：按schedule运行的事件，使得MemGPT可以在没有用户干预的情况下自动运行。
  * **事件解析** ：MemGPT通过解析器（Parser）处理事件，将它们转换为普通文本消息。这些文本消息可以附加到主要上下文中，最终作为输入传递给LLM处理器。



MemGPT在处理复杂任务时，通过**函数链** （Function Chaining）实现多步骤操作的机制。

  * **函数链的必要性** ：许多实际任务需要连续调用多个函数。例如，浏览单个查询结果的多页内容，或从多个文档中汇总数据。函数链允许MemGPT在返回控制权给用户之前，依次执行多个函数调用。
  * **控制权特殊标志** ：在MemGPT中，函数可以带有一个特殊标志“yield”，该标志请求在函数执行完成后立即将控制权返回给LLM 处理器。
    * 如果存在这个标志，MemGPT会将函数输出添加到主要上下文中，并继续运行处理器。
    * 如果没有这个特殊标志，MemGPT会在下一个外部事件（例如用户消息或schedule中断）触发前，暂停LLM处理器，也就是系统会等待新的外部事件来触发后续操作。



总结来说，函数链使得MemGPT能够在复杂任务中执行多步骤操作，提高处理效率和灵活性，而不需要用户逐步手动触发每个步骤。

## 4 总结

Letta解决了当前LLMs最关键的局限性之一。通过内存的分层管理等技术为释放LLM潜力提供了一条思路。

  1. Letta在兼容性、开发部署便利性方面值得九问学习，比如支持云端（商业版本）和本地部署，支持多种LLM provider，支持SDK/API/ADE等多种客户端，支持用户自行接入Postgres和SQLite数据库，支持Agent持久化。
  2. Letta是一个白盒解决方案，支持透明操作，可以让用户更信任。
  3. 网络评论Letta调试较为麻烦，通常需要许多轮对话才能看到效果。当前的大模型应用面向的用户大多是网页端/移动端用户，倾向于快问快答，对于Agent长期存储记忆的需求可能不高，使用对上下文做summary的方式来处理记忆可能更轻量化。未来可以挖掘用户需要长期记忆的场景。



## 5 问题和讨论

  1. Letta的状态管理、分层记忆能力、Agent之间互相调用的能力，如何在多Agent协作中实现记忆管理？？
  2. MemGPT采用了主要上下文和外部上下文的多级内存结构。这种结构在处理复杂任务时如何平衡即时访问和持久化存储的需求？在实际应用中，如何优化这种内存结构以提高性能？
  3. MemGPT通过函数链实现多步骤操作的机制。这种机制在处理复杂任务时如何确保每个步骤的准确性和连贯性？特别是在需要连续调用多个函数的场景中，如何避免错误传播或任务中断？



## 6 相关链接

  1. [How it works：https://awslabs.github.io/multi-agent-orchestrator/agents/built-in/supervisor-agent/](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3D0kW6oClCesE)
  2. [https://github.com/letta-ai/letta](https://link.zhihu.com/?target=https%3A//github.com/letta-ai/letta)
  3. [https://arxiv.org/abs/2310.08560](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2310.08560)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【记忆技术洞察】03-在记忆中思考：回忆和后期思考使LLM具有长期记忆](https://zhuanlan.zhihu.com/p/17147052697)
