# 【协议】03-Token从15万降低至2千，使用MCP执行代码

原文链接：https://zhuanlan.zhihu.com/p/1971238278771500576

---

​

目录

自2024年11月[MCP协议](https://zhida.zhihu.com/search?content_id=266202047&content_type=Article&match_order=1&q=MCP%E5%8D%8F%E8%AE%AE&zhida_source=entity)正式发布以来，其发展势头迅猛，普及速度远超预期。在极短的时间内，社区已贡献构建了数千个MCP服务器，展现出强大的生态活力。同时，所有主流编程语言均已提供相应的SDK支持，极大地降低了开发者的使用门槛。凭借其广泛的适配性与社区的积极响应，该协议已迅速崛起，成为连接[智能体](https://zhida.zhihu.com/search?content_id=266202047&content_type=Article&match_order=1&q=%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)与工具/资源之间通信的事实上的行业标准。

如今开发者常构建可访问数十个MCP服务器、数百乃至数千种工具的智能体。但随着连接工具数量增长，AI Agent需要预先加载每个工具定义，通过[上下文窗口](https://zhida.zhihu.com/search?content_id=266202047&content_type=Article&match_order=1&q=%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AA%97%E5%8F%A3&zhida_source=entity)来回传递中间结果，并且即使在不积极使用时也要将所有内容保存在内存中。这极大的消耗了token，复杂工作流中需要超过15万个token。

[Anthropic](https://zhida.zhihu.com/search?content_id=266202047&content_type=Article&match_order=1&q=Anthropic&zhida_source=entity)刚刚发布了一份新指南，改变了AI Agent使用工具的方式，减少了98%的token使用量，并显著加快了工作流程。

## 1 工具过度消耗Token，降低了智能体的效率

构建AI Agent时，它们需要工具才能执行任何有用的操作。当通过MCP服务器将智能体连接到数十个或数百个工具甚至更大规模时，有两种常见模式会导致智能体成本和延迟增加。

  1. 工具定义使上下文窗口过载
  2. 中间结果消耗额外token



### 工具定义使上下文窗口过载

多数MCP客户端会预加载所有工具定义到上下文中，通过直接工具调用语法向模型暴露。这些工具定义可能如下所示：
    
    
    salesforce.updateRecord
        Description: Updates a record in Salesforce
        Parameters:
                   objectType (required, string): Type of Salesforce object (Lead, Contact,      Account, etc.)
                   recordId (required, string): The ID of the record to update
                   data (required, object): Fields to update with their new values
         Returns: Updated record object with confirmation

工具描述会占用更多上下文窗口空间，增加响应时间与成本。当智能体连接数千个工具时，其在处理请求前需先处理数十万个tokens。

### 中间结果消耗额外token

大多数MCP客户端允许模型直接调用MCP工具。例如，要求智能体：“从Google Drive下载我的会议记录并附加到Salesforce潜在客户中。”

模型会进行如下调用：
    
    
    TOOL CALL: gdrive.getDocument(documentId: "abc123")
            → returns "Discussed Q4 goals...\n[full transcript text]"
               (loaded into model context)
    
    TOOL CALL: salesforce.updateRecord(
                objectType: "SalesMeeting",
                recordId: "00Q5f000001abcXYZ",
                data: { "Notes": "Discussed Q4 goals...\n[full transcript text written out]" }
            )
            (model needs to write entire transcript into context again)

每个中间结果都必须经过模型处理。在此示例中，完整通话记录会流经两次。对于2小时的销售会议，这可能意味着额外处理50,000个token。更大的文档甚至可能超出上下文窗口限制，导致工作流中断。

对于大型文档或复杂数据结构，模型在工具调用之间复制数据时更容易出错。

MCP客户端将工具定义加载到模型的上下文窗口中，并协调一个消息循环，使每个工具调用和结果在操作间通过模型传递

### 造成的后果

上述问题将严重影响应用的性能，并引发一系列问题：

  * **成本急剧上升** ：token用量与API费用直接挂钩，需要花费更多的钱。
  * **系统运行迟缓** ：模型需要处理更多token数据，token数量与响应延迟成正比，用户不得不等待更长时间。
  * **快速触达限制** ：上下文窗口存在容量上限。当工具定义和中间结果就消耗15万令牌时，实际任务的操作空间将所剩无几。
  * **扩展举步维艰** ：想要增加工具？每个新工具都会加剧问题。开发者将被迫在功能完整性与运行效率之间做出取舍。



## 2 通过MCP执行代码，解决上述问题

随着[代码执行环境](https://zhida.zhihu.com/search?content_id=266202047&content_type=Article&match_order=1&q=%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%E7%8E%AF%E5%A2%83&zhida_source=entity)在智能体中日益普及，解决方案是**将MCP服务器呈现为代码API而非直接工具调用** 。智能体可通过编写代码与MCP服务器交互。此方法可应对两大挑战：**智能体仅需加载必要工具，并在执行环境中处理数据后再将结果传回模型** 。

使用MCP执行代码——智能体只导入它需要的内容，在环境内部处理数据，并返回简洁的结果。

和当前方法之间的区别如下：

  * 当前方法：智能体使用工具调用API → 模型加载所有工具定义 → 模型直接调用工具 → 结果通过上下文返回
  * 代码执行方法：智能体编写代码 → 代码仅导入所需工具 → 代码执行并处理数据 → 仅将最终结果返回给模型



实现方式有多种。其一是从连接的MCP服务器生成所有可用工具的文件树。以下是基于TypeScript的实现示例：
    
    
    servers
    ├── google-drive
    │   ├── getDocument.ts
    │   ├── ... (other tools)
    │   └── index.ts
    ├── salesforce
    │   ├── updateRecord.ts
    │   ├── ... (other tools)
    │   └── index.ts
    └── ... (other servers)

然后，每个工具对应一个文件，例如：
    
    
    // ./servers/google-drive/getDocument.ts
    import { callMCPTool } from "../../../client.js";
    
    interface GetDocumentInput {
      documentId: string;
    }
    
    interface GetDocumentResponse {
      content: string;
    }
    
    /* Read a document from Google Drive */
    export async function getDocument(input: GetDocumentInput): Promise<GetDocumentResponse> {
      return callMCPTool<GetDocumentResponse>('google_drive__get_document', input);
    }

从Google Drive至Salesforce示例则转化为代码：
    
    
    // Read transcript from Google Docs and add to Salesforce prospect
    import * as gdrive from './servers/google-drive';
    import * as salesforce from './servers/salesforce';
    
    const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content;
    await salesforce.updateRecord({
      objectType: 'SalesMeeting',
      recordId: '00Q5f000001abcXYZ',
      data: { Notes: transcript }
    });

智能体通过探索文件系统来发现工具：列出`./servers/`目录查找可用服务器（如google-drive和salesforce），然后读取所需的具体工具文件（如`getDocument.ts`和`updateRecord.ts`）以了解每个工具的接口。

**中间数据永远不会接触到模型的上下文，所有操作都在代码执行环境中进行** 。这使得智能体仅需加载当前任务所需的定义，将令牌使用量从 150,000 减少到 2,000，时间和成本节省达98.7%。

节省token并非唯一的好处，由于执行的是代码而不是链式调用工具，因此运行速度也更快。相较于现在直接调用工具的方法，该方法需要一个安全的代码执行环境，会增加其复杂性。但对于需要扩展的生产级人工智能应用而言，其收益远远大于前期投入成本。通过该方法，可以获得更便宜的运营成本、更快的响应速度，并且能够将智能体连接到数百种工具而不会受到上下文限制。

## 3 MCP代码执行的优势

通过按需加载工具、在数据到达模型前进行过滤，以及单步执行复杂逻辑，MCP代码执行使智能体能够更高效地利用上下文。该方法在安全性和状态管理方面也有益处。

  1. **Token使用效率大幅提升** ：将工作流程从150,000个token减少到2,000个token，成本将下降98%以上。更低的token使用量意味着可以构建更复杂的智能体，而无需担心超出上下文限制。可以让智能体访问更多工具、处理更长的对话并处理更大的数据集。
  2. **渐进式披露** ：模型非常擅长导航文件系统。将工具以代码形式呈现在文件系统中，允许模型按需读取工具定义，而非一次性预加载所有内容。  
此外，可在服务器中添加`[search_tools](https://zhida.zhihu.com/search?content_id=266202047&content_type=Article&match_order=1&q=search_tools&zhida_source=entity)`工具来查找相关定义。例如，当使用前述假设的Salesforce服务器时，智能体搜索“salesforce”并仅加载当前任务所需的工具。在`search_tools`工具中包含详细级别参数，允许智能体选择所需的详细程度（如仅名称、名称和描述，或包含模式的完整定义），也有助于智能体节省上下文并高效查找工具。 
  3. **高效数据处理** ：处理大型数据集时，智能体可在返回结果前通过代码对结果进行过滤和转换。  
以获取 10,000 行电子表格为例，智能体仅看到五行而非十万行。类似模式适用于聚合、跨多个数据源的联接或提取特定字段——所有这些都无需膨胀上下文窗口。 
  4. **更强大且高效的控制流** ：循环、条件判断和错误处理可通过熟悉代码模式完成，而非链式调用单个工具。 
  5. **隐私保护优势** ：敏感数据可以在工作流程中流动，而无需进入模型的上下文。  
只有显式记录或返回的值才会对模型可见，其他所有信息都保留在执行环境中。如此可以处理机密信息，并基于此做出决策，然后仅返回最终结果。 
  6. **状态持久化** ：智能体可以将中间结果保存到文件中，从而能够恢复工作并跟踪进度。  
这使得跨多个会话的长时间运行任务成为可能。智能体无需每次都保持上下文关联或从头开始。它可以保存进度、保存状态，并在下次运行时从上次中断的地方继续执行。 
  7. **可复用的技能** ：随着持续开发与迭代，智能体还可以将它们自己的代码持久化为可重用函数，并慢慢将多个函数构建为一套“技能”。这时，其与Anthropic的技能（Skills）功能紧密相关，技能是一个包含可复用指令、脚本和资源的文件夹，旨在提升模型在特定任务中的表现。通过在其中添加SKILL.md文件，可以构建结构化的技能文档，便于模型参考与调用。



## 4 总结

MCP为智能体连接多种工具和系统提供了基础协议。然而，一旦连接服务器过多，工具定义和结果会消耗过量token，降低智能体效率。使用MCP执行代码，可以有效的提升token使用效率，随着后续的不断完善和迭代，逐渐演化为一项技能，为后续其他智能体的复用提供支撑。

## 相关链接

  1. [https://www.anthropic.com/engineering/code-execution-with-mcp](https://link.zhihu.com/?target=https%3A//www.anthropic.com/engineering/code-execution-with-mcp)
  2. [https://medium.com/ai-software-engineer/anthropic-just-solved-ai-agent-bloat-150k-tokens-down-to-2k-code-execution-with-mcp-8266b8e80301](https://link.zhihu.com/?target=https%3A//medium.com/ai-software-engineer/anthropic-just-solved-ai-agent-bloat-150k-tokens-down-to-2k-code-execution-with-mcp-8266b8e80301)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【协议】02-Google开源的A2A (Agent2Agent)协议，对多Agent会带来什么影响](https://zhuanlan.zhihu.com/p/1895434611774949228)
