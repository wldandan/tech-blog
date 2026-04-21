# 【模型】01-xLAM：一系列增强AI Agent能力的大动作模型

原文链接：https://zhuanlan.zhihu.com/p/1903090007566192858

---

​

目录

## 1 概述

2025年将是Agent元年！当前，众多科技大厂如Google、AWS等纷纷积极布局Agent应用生态，推出Vertex AI Agent Builder、Amazon BedRock Aget等AI Agent产品服务/框架。

通常，Agent在完成复杂任务时，会通过一系列功能调用完成用户命令。这些调用通常会涉及大量潜在的函数/API，用于帮助满足用户的要求。每个函数/API 都有不同的描述、参数和返回值。当前，工具调用都是通过[LLM](https://zhida.zhihu.com/search?content_id=257373363&content_type=Article&match_order=1&q=LLM&zhida_source=entity)实现的，LLM会在提示符中显示适用的函数，然后LLM会根据上下文和具体目标选择合适的函数，选择相应的参数，并从所选函数中获取输出。然而，通用的LLM并不是专门为函数调用环境定制的，工具调用存在不准确的问题。为了解决这个问题，xLAM（x Large Action Model）由此而生。

**xLAM是有Salesforce开发的大型动作模型，基于广泛的函数调用环境和数据集进行训练，专为函数调用而设计，其作用在各种环境中做出决策和执行动作** 。xLAM和大型语言模型的区别在于，LAM是经过微调（使用函数调用环境和数据集）的LLM，针对函数调用进行了优化。理论上，该模型应该能够适应未见过的函数调用环境。

## 2 数据处理流水线

xLAM训练过程中，汇集了包含ToolBench、[Webshop](https://zhida.zhihu.com/search?content_id=257373363&content_type=Article&match_order=1&q=Webshop&zhida_source=entity)、ToolApaca、HotpotQA、AlfWorld、[APIBank](https://zhida.zhihu.com/search?content_id=257373363&content_type=Article&match_order=1&q=APIBank&zhida_source=entity)、Mind2Web、AgentBoard和AgentBench等著名的函数调用数据集。除了以上现有的函数调用数据集之外，还利用API-GEN、[SpecTools](https://zhida.zhihu.com/search?content_id=257373363&content_type=Article&match_order=1&q=SpecTools&zhida_source=entity)和内部大型行动模型模拟（[LAM Simulator](https://zhida.zhihu.com/search?content_id=257373363&content_type=Article&match_order=1&q=LAM+Simulator&zhida_source=entity)）环境等合成数据，构建丰富的数据池。

图1 xLAM的数据处理、训练和评估概览

xLAM模型训练所采用的数据处理流程，包括数据统一、数据增强、质量验证、通用指令数据合成以及偏好数据生成等核心环节。

### 2.1 数据统一

现有的智能体数据集来源多样，数据格式各不相同，导致引入大量噪声，同时也使得数据增强和质量验证变的更加复杂。因此，为解决上述问题，对现有数据集进行**标准化处理，统一数据格式** ，保障了数据的一致性，简化了模型训练，并增强了模型在各种基准测试中的泛化能力。

以函数调用风格设计统一的数据格式，如图2所示，该格式包含多个结构化模块：

  * **任务指令（Task Instruction）** ：描述智能体需要完成的任务。
  * **可用工具集（Available Tools）** ：定义了智能体的在执行动作时可使用的工具。
  * **格式规范（Format Instruction）** ：明确了生成响应时应遵循的输出格式规范。
  * **少样本示例（Few-shot Examples）** ：少量示例数据，帮助智能体更好的理解任务。
  * **查询（Query）** ：具体的任务或者问题信息。
  * **步骤（Steps）** ：智能体规划的任务处理过程中的具体步骤。



在每一步中，智能体的输出、环境反馈/执行结果以及用户的后续输入都以字典的形式组织起来。 

图2 统一的函数调用数据格式

统一的数据格式具备良好的兼容性，能够适应不同的环境和任务，并将使得数据处理流程能够适配多样化数据集和具有良好的扩展性。而且，模块化设计允许对数据进行细粒度的数据增强和质量验证，提升智能体数据质量。

### 2.2 数据增强

数据增强聚焦于提升数据的多样性，通过对现有数据集实施多种变换生成新的合成数据样本。数据增强技术分为**提示格式增强** 与**指令遵循增强两类** 。

**提示格式增强** ：该技术基于结构化的统一数据格式生成多样化的提示模板，具体包含两种实现方式：

  * **打乱顺序** （Order Shuffling）：随机打乱工具列表（包含名称、描述与参数信息等）、输入内容的各模块（务指令、工具集等）的顺序，将工具内部信息呈现数据随机处理，避免模型对工具顺序产生过拟合。
  * **连接标记** （Concatenation Tokens）：每个训练数据点由输入-输出序列对构成。采用特殊标记连接不同模块形成单一序列，将结构化统一格式转换为训练提示。



**指令遵循增强** ：该技术旨在通过指令多样性提升模型的指令遵循能力，具体包含两种实现方式：

  * **任务指令重构** ：利用大语言模型对任务指令进行多样化重述，以适应用户的多样化输入风格，并验证重构后的指令生成正确的函数调用的准确性。
  * **格式规范跟随** ：在统一格式中，输出格式为包含“思考”和“工具调用”（thought和tool_calls）的JSON字符串。为了避免模型对JSON格式的过度拟合，并使模型在不同格式指令下遵循不同的输出格式，提供了15中不同的输出格式（如JSON、XML、YAML、纯文本等）、对应格式指令和格式转换器。  




### 2.3 数据质量验证

采用基于规则、大语言模型作为评判（LLM-as-a-judge）的方式，对统一化后的数据集进行详细的错误分析，评估误差来源。识别出以下主要错误类型：

  * **未定义函数调用** ：数据中存在未被定义或者无法识别的函数调用。
  * **参数类型错误** ：函数调用中的参数类型和实际类型不符，比如将应该是列表类型的参数生成为字符串类型。
  * **参数幻觉** ：数据中出现不存在的参数值，源于LLM生成数据时产生的幻觉。包含两种类型幻觉：（1）工具/参数名称未在工具列表中定义；（2）参数值与查询/历史值不一致。
  * **低质量推理和规划** ：数据中的部分推理和规划步骤质量不高，可能存在逻辑不清晰或者不合理的状况。



### 2.4 数据合成

大多数公开数据集存在显著的局限，例如数据的静态性强，由性能较弱的模型生成，覆盖范围有限且缺乏执行验证；数据集专注于单一函数调用，而实际中需要支持并行调用等。

为解决上述问题，使用数据合成框架APIGen基于一组可执行的API生成可验证的数据集。其核心思想是通过多阶段验证流程（格式验证、执行验证、语义验证）确保生成数据的准确性和质量。

使用来自ToolBench的21个类别共3673个API，生成总计60000条高质量数据样本，这些数据样本为大模型训练提供了丰富和可靠的数据源。

### 2.5 数据混合

在监督微调（Supervised Fine-Tuning, SFT）阶段的数据集结合了来自三个主要来源的训练样本：清洗增强的智能体数据集、合成函数调用数据集、通用指令调优数据集。这些数据源被用于训练通用的xLAM模型。

  * **通用指令能力增强** ：为了增强xLAM的通用指令能力，整合了来自DialogStudio和Data Provenance的多样化指令微调数据集。在整合过程中，通过基于规则去除低质量数据（如重复词汇和低效对话轮次），过滤不适当或者非商业许可的数据，对数据进行相似度去重并按领域或者类别进行分类，并进行对话级和响应级的质量评估。
  * **函数调用优化** ：针对xLAM-7b-fc-r与xLAM-1b-fc-r模型，采用了有针对性的训练方法，其中50%训练数据来自高质量合成函数调用数据集，剩余50%的训练数据来自训练集中其他任务。
  * **直接偏好优化(DPO)数据处理** ：通过弱模型生成响应并进行评分，然后对生成的响应进行人工验证和抽样。通过模型与提示词迭代优化，将选定的响应分类为被拒的样本，构建高质量偏好对。



## 3 模型训练

研究人员还推出了针对特定用例量身定制的多种智能体模型。旗舰xLAM系列建立在Mixtral Instruct模型的基础上，旨在为各种智能体任务（从复杂的多轮对话到功能调用应用）提供均衡的性能。除通用xLAM模型外，研究团队还开发了两个专门用于函数调用任务的模型，即xLAM-7B-fc-r和 xLAM-1B-fc-r，它们分别基于DeepSeek-Coder-7B-instruct-v1.5和DeepSeekCoder-1.3B-instruct 构建。

图3 xLAM系列模型概览

## 4 实验

### 4.1 实验基准

  * **Webshop** ：模拟在线购物任务的环境，用于评估智能体在模拟在线购物中的表现，提供协助的能力。
  * **ToolQuery** ：用于评估智能体在跨多个领域使用工具检索信息的能力。该测试涵盖多种场景，如天气查询和电影推荐等。
  * **ToolBench** ：基于RapidAPI的实时评估平台，用于测试智能体在多轮推理和交互功能中的表现。评估涵盖领域内（in-domain）与跨领域（out-of-domain）场景，具体包括三种测试维度：基于熟悉工具执行未见过的指令、在已知工具类别中使用未见过的新工具，以及处理完全未知工具类别的新工具。
  * **Berkeley Function-Calling Leaderboard** ：重点考察模型在函数调用方面的能力，包括函数的正确识别、参数传递的准确性以及函数执行结果的有效性等方面。主要挑战模型在复杂任务中处理多次函数调用的能力。评估指标包括抽象语法树（Abstract Syntax Tree, AST）准确率和可执行准确率（executable accuracy）。



### 4.2 实验结果

在Webshop环境中，xLAM-7b-r在成功率最好，达到0.414。超越其他模型通用模型GPT-4-0125-preview和专门智能体模型AgentOhana-8x7b等，展示了xLAM模型在网络交互环境中有效执行任务的能力。

在更复杂的ToolQuery环境中，xLAM-8x7b-r和xLAM-8x22b-r也展示了高性能，在成功率上排名第二，超过了Mixtral-8x7b-inst和Mixtral-8x22b-inst等基线模型，同时展示了数据统一和合成数据流水线的有效性。

在Webshop和ToolQuery上的测试结果

在ToolQuery-Unified中，当系统提示以统一格式呈现给模型时，xLAM模型的性能比GPT模型更稳定，GPT-4o的性能下降42%，而xLAM 8x22b模型保持了相当的性能。

ToolQuery-Unified的测试结果

在ToolBench上，xLAM模型超越了TooLlama-V2和GPT-3.5-Turbo-0125等模型。此外，在涉及未见指令和未见工具的场景中，xLAM模型的性能优于 AgentOhana-8x7b，同时在未见工具设置中实现了与GPT-4-0125-preview相当的性能。展示了其在多轮推理和复杂工具使用方面的强大能力。

在ToolBench上三种不同场景的通过率

在Berkeley Function-Calling Benchmark上，xLAM模型系列在函数调用任务中的出色表现，在前20名中占据了4个位置。其中，xLAM-8x22b-r获得了最高的准确率。紧随其后，xLAM-8x7b-r排名第6，表现超过了包括 GPT-4o-mini和Claude-3在内的模型。

Berkeley Function-Calling Benchmark排行榜上的性能比较

## 5 总结

xLAM系列是一系列用于自主AI Agent的大型行动模型，涵盖了不同的参数规模，包含从1B到8x22B不等。xLAM系列模型使用可扩展且灵活的数据流水线（包含统一格式、数据增强和数据合成等）进行训练，在各种基准测试中始终表现优异。通过训练这些模型的经验，突显出严格数据处理的重要性以及数据合成在开发有能力的AI Agent方面的潜力。

## 相关链接

  1. [https://arxiv.org/pdf/2409.03215](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2409.03215)
  2. [https://www.salesforce.com/blog/large-action-model-ai-agent/](https://link.zhihu.com/?target=https%3A//www.salesforce.com/blog/large-action-model-ai-agent/)
  3. [https://pub.towardsai.net/inside-xlam-salesforces-models-specialized-for-agentic-tasks-fe5d82f5cbb9](https://link.zhihu.com/?target=https%3A//pub.towardsai.net/inside-xlam-salesforces-models-specialized-for-agentic-tasks-fe5d82f5cbb9)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【模型洞察】02-LLMOps：构筑企业级大语言模型即服务的推手](https://zhuanlan.zhihu.com/p/1905351954235891921)
