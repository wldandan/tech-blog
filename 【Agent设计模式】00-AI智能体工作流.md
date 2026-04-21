# 【Agent设计模式】00-AI智能体工作流

原文链接：https://zhuanlan.zhihu.com/p/690202723

---

​

目录

在大语言模型发布后，AI智能体在去年一度成为热点话题。那么，AI智能体的发展前景如何？人工智能下一个突破的方向是什么？

近日，**斯坦福大学教授、人工智能著名学者吴恩达** 在文章中指出AI智能体的工作流程将在今年推动人工智能取得巨大进步，甚至可能超过下一代基础模型。他呼吁AI领域的工作人员关注该趋势，并认为这是一个重要的趋势。AI Agent可能会代表人工智能的未来发展方向。

后续部分，基于吴恩达的系列文章进行了整理，内容如下。

## 为什么需要[AI智能体工作流](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=AI%E6%99%BA%E8%83%BD%E4%BD%93%E5%B7%A5%E4%BD%9C%E6%B5%81&zhida_source=entity)

目前，主要在零样本（zero-shot）模式下使用[LLMs](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=LLMs&zhida_source=entity)。即大模型根据提示生成最终输出，并不会做任何的修改。这就好比要求一个人从头到尾直接写一篇文章，不允许修改，并期待高质量的结果。尽管困难重重，LLMs还是出色地完成了这项任务！

然而，通过Agent工作流，我们可以要求LLMs对文档进行多次迭代。例如，它可以采取如下一系列步骤：

  * 规划大纲。
  * 决定是否需要进行网络搜索以收集更多信息。
  * 写初稿。
  * 通读初稿，找出不合理的论据或无关信息。
  * 根据发现的不足之处修改初稿。
  * …



这种迭代过程对于大多数人类作者来说是写出好文章的关键。对于AI来说，这种迭代工作流产生的结果比一次性写作要好得多。

同时，吴恩达根据数据发现，[GPT-3.5](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=GPT-3.5&zhida_source=entity)（零样本）的正确率为48.1%，[GPT-4](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=GPT-4&zhida_source=entity)（零样本）的正确率更高，为67.0%，然而其表现相差并不是很大。但是，通过引入迭代智能体工作流，GPT-3.5的正确率高达95.1%。

由上可见，相对于传统的LLMs使用方式，智能体工作流是通过多次迭代，逐步构建出更高质量的输出。甚至，基于GPT-4构建一个AI智能体的工作流，可以达到GPT-5的水平。那么，如何构建AI智能体工作流呢？

## AI智能体四种设计模式

对于AI智能体工作流，吴恩达分享了4种用于构建智能体的设计模式，该模式已经在其团队的许多应用中进行了使用。

  * **[反思](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=%E5%8F%8D%E6%80%9D&zhida_source=entity) （Reflection）：**LLM检查自己的工作，找出改进方法。
  * **[工具使用](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8&zhida_source=entity) （Tool use）：**为LLM提供工具，例如网络搜索、代码执行或任何其他功能，以帮助它收集信息、采取行动或处理数据。
  * **规划（Planning）：** LLM提出并执行多步骤计划来实现目标（如，为一篇文章写大纲，然后进行在线研究，然后写草稿等）。
  * **[多智能体协作](https://zhida.zhihu.com/search?content_id=241508777&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E5%8D%8F%E4%BD%9C&zhida_source=entity) （Multi-agent collaboration）：**多个AI智能体一起工作，分解任务，讨论和辩论各种想法，从而提供比单个智能体更好的解决方案。



### 反思（Reflection）

在吴恩达后续的文章中，首先讨论了反思（Reflection）。他认为，反思是一种相对容易实现的设计模式，它能带来令人惊讶的性能提升。

我们可能有过这样的经历：提示ChatGPT/Claude/Gemini，收到了不满意的输出，然后提供批评性反馈以帮助LLM改进其响应，接着获得了更好的响应。如果将提供批判性反馈的步骤自动化，让模型自动批评自己的输出并改进响应，会怎么样呢？这就是 "反思 "的关键。

以要求LLM编写代码为例。我们可以提示它直接生成所需的代码，以执行某个任务X。然后，我们可以提示LLM对自己的输出进行反思，例如：

 _以下是为任务X编写的代码：[之前生成的代码]_

_仔细检查代码的正确性、风格和效率，并就如何改进提出建设性建议。_

有时，这会导致LLM发现问题并提出建设性的建议。接下来，我们可以向LLM提供上下文提示，包括（i）先前生成的代码和（ii）建设性反馈，以及(iii)要求它使用反馈来重写代码。这样可以得到更好的回复。重复批评/重写的过程可能会带来进一步的改进。通过这种自我反思的过程，LLM可以发现自己的不足，并改进自己在各种任务（包括生成代码、编写文本和回答问题）中的输出。

除了自我反思之外，我们还可以为LLM提供一些工具，帮助它对输出结果进行评估；例如，通过一些单元测试来运行它的代码，检查它是否能在测试用例中生成正确的结果，或者通过搜索网络来仔细检查文本输出。然后，它可以对发现的任何错误进行反思，并提出改进想法。

此外，我们还可以使用多智能体框架来实现反思。创建两个不同的智能体很容易，一个负责生成好的输出结果，另一个负责对第一个智能体的输出结果提出建设性的批评意见。通过两个智能体之间的讨论，改进响应。

反思是一种相对基础的智能体工作流，但在一些案例中，它能大大改善应用程序的结果。如果想了解更多关于反思的内容，推荐阅读以下论文：

  * “Self-Refine: Iterative Refinement with Self-Feedback,” Madaan et al., 2023
  * “Reflexion: Language Agents with Verbal Reinforcement Learning,” Shinn et al., 2023
  * “CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing,” Gou et al., 2024



### 工具使用（Tool use）

工具使用是AI智能体工作流的一种关键设计模式，在工具使用中，LLM被赋予可以请求调用功能来收集信息、采取行动或处理数据。目前基于LLM的系统已经可以进行网络搜索或者执行代码实现工具的使用。但是工具的使用远远超出了这些例子。

如果你对一个基于LLM的在线聊天系统进行提示，“根据评论者的评价，最好的咖啡机是什么?”，该系统可能会决定进行网络搜索并下载一个或多个网页以获取上下文。早期时候，LLM的开发人员意识到仅依靠预训练过的transformer来生成输出token是有限的，而向LLM提供网络搜索工具可以让它做得更多。有了这样的工具，LLM要么被微调，要么被提示（可能只有很少的提示）生成一个特殊的字符串，比如{tool: web-search, query: "coffee maker reviews"}，以请求调用搜索引擎。（字符串的确切格式取决于实现。）然后，一个后处理步骤会查找这样的字符串，在找到一个字符串时调用带有相关参数的web搜索函数，并将结果作为额外的输入上下文传递回LLM以进行进一步处理。

同样，如果提问“如果我以7%的复利投资100美元，12年后最终会拿到多少钱?”，而不是试图直接使用transformer网络生成答案（这不太可能产生正确的答案），LLM可能使用代码执行工具运行Python命令来计算100 *(1+0.07)**12以获得正确答案。可能会生成这样的字符串:{tool: python-interpreter, code: "100 *(1+0.07)**12"}。

但是，智能体工作流中的工具使用不止于此。开发人员正在使用函数来搜索不同的资源（网络、维基百科、arXiv等），与生产力工具对接（发送电子邮件、读/写日历条目等），生成或解释图像等等。我们可以使用给出许多函数详细描述的上下文来提示LLM，这些描述可能包括函数功能的描述，以及函数期望的参数等详细信息。

此外，在正在构建的系统中，LLM可以使用数百种工具。由于工具太多，可能无法将工具全部放到LLM上下文中，因此，在这种情况下，可以使用启发式方法来选择在当前处理步骤中包含在LLM上下文中的最相关子集。这种技术在下文引用的Gorilla论文中有所描述，它类似于如果有太多文本要作为上下文纳入LLM，检索增强生成（RAG）系统会提供启发式方法来选择要纳入的文本子集。

在LLM发展早期，LMMs（Large Multimodal Models）模型如LLaVa、GPT-4V和Gemini普及之前，LLMs无法直接处理图像，因此计算机视觉领域大量使用了工具。当时，基于LLM的系统处理图像的唯一方法是调用一个函数，如执行对象识别或者其他功能。此后，工具使用的实践迅速发展，GPT-4 的函数调用功能是向通用工具使用迈出的重要一步。从那时起，越来越多的LLMs被开发出来，都会具备工具使用能力。

如果对工具使用感兴趣，推荐阅读以下论文：

  * “Gorilla: Large Language Model Connected with Massive APIs,” Patil et al. (2023)
  * “MM-REACT: Prompting ChatGPT for Multimodal Reasoning and Action,” Yang et al. (2023)
  * “Efficient Tool Use with Chain-of-Abstraction Reasoning,” Gao et al. (2024)



### 规划（Planning）

规划 (Planning) 是一种关键的AI设计模式，在该模式中，使用大型语言模型 (LLM) 自主决定执行哪些步骤顺序来完成更大的任务。例如，如果我们要求智能体对给定主题进行在线研究，我们可能会使用LLM将目标分解为更小的子任务，例如研究特定的子主题，综合发现和编写报告。

许多任务无法通过单个步骤或使用单个工具的调用来完成，但智能体可以决定采取哪些步骤。例如，为了简化HuggingGPT论文中的一个示例（引用部分后附），如果你想让一个智能体参考一张男孩的照片，并以相同的姿势画一张女孩的照片，那么这个任务可能被分解成两个不同的步骤：（1）检测男孩照片中的姿势，（2）呈现一张和被检测姿势一样的女孩照片。可以通过输出像“{tool: pose-detection, input: image.jpg, output: temp1} {tool: pose-to-image, input: temp1, output: final.jpg}”这样的字符串来对LLM进行微调或提示（使用少量提示）以指定计划。

这个结构化的输出指定了要采取的两个步骤，然后触发软件调用姿势检测工具，然后用姿势到图像 (pose-to-image) 工具来完成任务。（本例仅作说明用途，HuggingGPT使用了不同的格式。）

诚然，许多智能体工作流不需要规划。例如，你可以让智能体对其输出进行固定次数的反思和改进。在这种情况下，智能体采取的步骤顺序是固定且确定的。但是对于无法提前将任务分解为一组步骤的复杂任务，规划将允许智能体动态地决定采取哪些步骤。

一方面，规划是一种非常强大的能力；另一方面，它也可能导致难以预测的结果。根据我的经验，虽然我可以让反思和使用工具的智能体设计模式可靠地工作并提高应用程序的性能，但规划是一种不太成熟的技术，我发现我很难提前预测它会做出什么。但这个领域仍在迅速发展，我相信规划能力将迅速提高。

如果对LLM规划感兴趣，推荐阅读以下论文：

  * “Chain-of-Thought Prompting Elicits Reasoning in Large Language Models,” Wei et al. (2022)
  * “HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in Hugging Face,” Shen et al. (2023)
  * “Understanding the planning of LLM agents: A survey,” by Huang et al. (2024)



### 多智能体协作（Multi-Agent Collaboration）

对于编程这样的复杂任务，多智能体会将任务分解为子任务，由不同的角色——如软件工程师、产品经理、设计师、QA（质量保证）工程师等来执行，并让不同的智能体完成不同的子任务。

可以通过提示一个LLM（或者提示多个LLM）执行不同的任务来构建不同的智能体。例如，要构建一个软件工程师智能体，我们可能会提示LLM：“你是编写清晰、高效代码的专家。编写代码来执行任务......”

使用多个智能体来进行编程的几个理由如下：

  1. 许多团队使用多智能体获得了很好的结果。此外，消融研究（例如，下面引用的AutoGen论文）表明，多智能体比单智能体具有更好的性能。
  2. 尽管现在有些LLM可以接受很长的输入上下文（例如，Gemini 1.5 Pro可以接受100万个token），但它们真正理解长而复杂的输入的能力确参差不齐。在一个智能体工作流中，LLM被提示一次只专注于一件事就可以提供更好的性能。通过告诉它什么时候应该扮演软件工程师，还可以指定在该角色的子任务中哪些是重要的。例如，上面的提示强调清晰、高效的代码，而不是可扩展和高安全的代码。通过将整体任务分解为子任务，可以更好地优化子任务。
  3. 可能最重要的是，作为开发人员，多智能体设计模式为我们提供了一个将复杂任务分解为子任务的框架。当编写在单个CPU上运行的代码时，我们经常将程序分解为不同的进程或线程。这是一个有用的抽象概念，它能让我们将一个任务（如Web浏览器）分解成更容易编程的子任务。我发现通过多智能体角色进行思考也是一种有用的抽象。



在许多公司，管理者通常会决定雇佣什么角色（职位），然后决定如何将复杂的项目——比如编写一大型软件或准备一份研究报告——分解为更小的任务，并分配给擅长不同领域的员工。使用多个智能体亦是同理。每个智能体会实现自己的工作流程，拥有自己的记忆，并且可以向其他智能体寻求帮助。智能体还可以参与规划和工具使用。这将导致LLM调用和智能体之间消息传递的不和谐，从而形成非常复杂的工作流。

AutoGen, Crew AI和LangGraph等新兴框架为构建多智能体解决方案提供了丰富的方法。

与“规划”设计模式一样，多智能体协作的输出质量很难预测，尤其是当允许智能体自由交互并为其提供多种工具时。而更成熟的“反思”模式和“工具使用”模式会更可靠。

如果对多智能体感兴趣，推荐阅读以下内容：

  * “Communicative Agents for Software Development,” Qian et al. (2023) (the ChatDev paper)
  * “AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation,” Wu et al. (2023) 
  * “MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework,” Hong et al. (2023)



后续，将会继续分享其他智能体设计模式。

## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【Agent设计模式】01-智能时代已至！Agent设计模式综述](https://zhuanlan.zhihu.com/p/711206099)
