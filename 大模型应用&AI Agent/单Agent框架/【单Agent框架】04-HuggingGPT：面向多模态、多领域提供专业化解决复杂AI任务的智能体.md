# 【单Agent框架】04-HuggingGPT：面向多模态、多领域提供专业化解决复杂AI任务的智能体

原文链接：https://zhuanlan.zhihu.com/p/673691859

---

​

目录

[HuggingGPT](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=HuggingGPT&zhida_source=entity)是浙江大学、微软亚洲研究院合作开发的开源项目，以chatGPT和[Hugging Face](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=Hugging+Face&zhida_source=entity)为基础构建的一个框架，融合[LLM](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=LLM&zhida_source=entity)能力以及AI领域模型的能力，来解决不同领域和模态的AI任务。项目地址是：[https://github.com/microsoft/JARVIS](https://link.zhihu.com/?target=https%3A//github.com/microsoft/JARVIS) 。项目名称借鉴了漫威《钢铁侠》中人工智能管家的名称，也传递了项目想要实现通用人工智能的愿景。

## 背景&原理

### LLM迈向AGI的挑战

大型语言模型（如[ChatGPT](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=ChatGPT&zhida_source=entity)）在各种自然语言处理（NLP）任务中的出色表现，吸引了学术界和工业界的极大关注。基于对大量文本语料库的大规模预训练和来自人类反馈的强化学习（[RLHF](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=RLHF&zhida_source=entity)），LLM可以在语言理解、生成、交互和推理方面产生卓越的能力。LLM强大的能力也推动了许多新兴的研究课题（如上下文学习、指令学习和[思维链提示](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=%E6%80%9D%E7%BB%B4%E9%93%BE%E6%8F%90%E7%A4%BA&zhida_source=entity)）进一步研究 LLM 的巨大潜力，并为我们构建人工通用智能（AGI）系统带来了无限的可能性。  
目前的LLM技术仍然不完善，在构建 AGI 系统的过程中面临着一系列挑战：

  * 受限于文本生成的输入和输出形式，当前的 LLM 缺乏处理视觉和语音等复杂信息的能力。
  * 在现实世界的场景中，一些复杂的任务通常由多个子任务组成，因此需要多个模型的调度和协作。
  * 对于一些具有挑战性的任务，LLM 仍然比一些专家（例如，微调模型）弱。 如何解决这些问题，可能是 LLM 迈向 AGI 系统的第一步，也是关键一步。



### 与领域模型融合

根据上述描述，为了处理复杂的人工智能任务，LLM应该能够与外部模型协调，以利用这些模型的能力来解决不同领域和多模态的AI任务。因此，关键是如何选择合适的中间件来建立 LLM 和 AI 模型之间的连接。  
利用语言能力，每个人工智能模型都可以通过总结其模型功能来表示为一种语言形式。因此，自然语言可以被设置为 LLM 连接人工智能模型的通用接口。换句话说，通过将这些模型描述结合到提示中，LLM可以被认为是管理人工智能模型（如计划、调度和合作）的大脑。因此，这种策略使 LLM 能够调用外部模型来解决人工智能任务。  
但如果我们想将多个人工智能模型集成到 LLM 中，会出现另一个挑战将：解决大量人工智能任务需要收集大量高质量的模型描述，这需要大量的即时工程。不过，一些ML社区（如Hugging Face等）通常提供各种各样的适用模型，这些模型具有定义明确的模型描述，用于解决特定的人工智能任务，如语言、视觉和语音。  
由此，基于以上思路，把LLM（ChatGPT）和ML社区（HuggingFace）连接起来构建huggingGPT系统，它可以处理来自不同模式的输入并解决许多复杂的人工智能任务。更具体地说，对于 Hugging Face中的每个 AI 模型，我们使用库中对应的模型描述，并将其融合到提示中，以建立与 ChatGPT 的连接。之后，在我们的系统中，LLM（即ChatGPT）将充当大脑来确定用户问题的答案。

## 实例讲解

HuggingGPT是一个以大型语言模型（ChatGPT）充当控制器，以专家模型（HuggingFace）为执行者的协作系统。HuggingGPT以语言为接口，通过ChatGPT进行任务规划、模型选择，使用专家模型处理对应领域的问题，生成最终结果，来系统解决人工智能任务。

解决不同领域和模态的AI任务是迈向通用人工智能的关键一步。虽然现在有大量的AI模型可以用于解决不同的领域和模态的问题，但是它们不能解决复杂的AI问题。由于大模型(LLM)在语言理解、生成、交互和推理上展现出很强的能力，因此构建HuggingGPT框架，连接不同的AI模型来解决AI任务。ChatGPT充当一个控制器的作用来管理现有的AI模型以解决复杂的AI任务，以语言为一个通用的接口来启动AI处理这些任务。  
下面我们以一个真实的用户问题实例，来进行讲解：

我们看一下用户问题：你能描述一下这张照片是什么？以及图片中的有多少个实体？问题和这张照片作为输入，传递到chatGPT，通过语义理解和分析，主要是和图像相关，规划了图像相关的几个任务。 然后，在HuggingFace中选择对应任务的的模型，像图像识别、实体检测等。接着，根据任务，执行相应的模型，并将结果返回给ChatGPT。最后，ChatGPT利用返回各个模型的返回的结果，生成答案，包括两部分：图片描述为“一群长颈鹿和斑马在田野里吃草”，实体检测结果是“有5个，分别是XX及得分”，还把对应的动物用红框框了出来；并且还会说明是怎么得到这个结果，分别使用了 image classification, object detection和image caption三个任务，以及对应的任务所选择的模型。  
根据任务讲解的实际步骤，整个HuggingGPT的过程可以分为四个阶段：

  * 任务规划：使用ChatGPT分析用户的请求，了解用户的意图。通过提示将它们拆解为可能的可解决任务。
  * 模型选择：为了解决计划任务，ChatGPT选择托管在基于模型描述的Hugging Face。
  * 任务执行：调用并执行每个选择的模型，并将结果返回给ChatGPT。
  * 响应生成：最后，利用ChatGPT集成来自所有模型的预测并为用户生成响应。



## HuggingGPT详细说明

HuggingGPT 可以看作是一个协作系统，以ChatGPT为控制器，以专家模型为执行者的一个合作系统，整体的工作流为：任务规划、模型选择、任务执行和回答生成。过程描述如下：

  1. ChatGPT首先解析用户请求，将其分解为多个任务，并根据其知识规划任务顺序和依赖关系。
  2. ChatGPT根据 Hugging Face中的模型描述，将解析后的任务分配给对应的专家模型。
  3. 专家模型在推理端点上执行分配的任务，并将执行信息和推理结果返回给ChatGPT。
  4. 最后，ChatGPT总结执行过程日志和推理结果，并将信息整合返回给用户。



### 如何规划任务

在第一个阶段，大型语言模型接受用户的请求，将其分解为一系列结构化任务。  
复杂的请求通常涉及多个任务，大型语言模型需要确定这些任务的依赖关系和执行顺序。为了提示大型语言模型进行有效的任务规划，HuggingGPT在其提示设计中采用了基于规范的指令和基于演示的解析。  
基于规范的指令任务规范为任务提供了统一的模板，并允许大型语言模型通过槽位归档进行任务解析。HuggingGPT 为任务解析设计了四个接口，分别是任务类型、任务ID、任务依赖项和任务参数： 

  * task id：任务 ID 为任务规划提供了一个唯一的标识符，用于引用相关任务及其生成的资源。
  * task type：任务类型涵盖语言、视觉、视频、音频等不同任务。HuggingGPT当前支持的任务列表如下表所示。
  * task dependencies：任务相关性定义了执行所需的先决条件任务。只有在完成所有先决条件相关任务后，才会启动该任务。
  * task arguments：任务参数包含执行任务所需的参数列表。它包含三个子字段，根据任务类型填充文本、图像和音频资源。它们是从用户的请求或相关任务的生成资源中解析的。不同任务类型的相应参数类型如下表所示。



在模板中规定了task的类型范围，HuggingGPT融合了HuggingFce中成百上千的模型和GPT，可以解决24种任务，包括文本分类、对象检测、语义分割、图像生成、问答、文本语音转换和文本视频转换。支持的任务列类型表如下： 

### 如何选择模型

解析任务列表后，HuggingGPT 接下来需要匹配任务和模型，即为任务列表中的每个任务选择合适的模型。  
为此，我们首先从 [HuggingFace Hub](https://zhida.zhihu.com/search?content_id=237838890&content_type=Article&match_order=1&q=HuggingFace+Hub&zhida_source=entity) 获得专家模型的描述，然后通过上下文中的任务模型分配机制为任务动态选择模型。这种做法允许增量模型访问（简单地提供专家模型的描述），并且更加开放和灵活。  
HuggingFace Hub 上托管的专家模型附有全面的模型描述，这些描述通常由开发人员提供。这些描述包含有关模型的功能、体系结构、支持的语言和域、许可等方面的信息。这些信息有效地支持 HuggingGPT 根据用户请求和模型描述的相关性为任务选择正确模型的决定。task描述如下：

  
在上下文任务模型分配中，我们将任务和模型的分配视为单选问题，其中潜在模型作为给定上下文中的选项呈现。通过在提示中包括用户查询和解析的任务，HuggingGPT 可以为手头的任务选择最合适的模型。然而，由于最大上下文长度的限制，不可能总是在提示中包含所有相关的模型信息。为了解决这个问题，我们根据模型的任务类型（task type）筛选模型，只保留与当前任务类型匹配的模型，并且对剩下的模型根据下载量排序，然后选择top-K个模型作为HuggingGPT的候选模型。  


### 如何执行任务

任务被分配给特定的模型后，接下来就是执行任务，即执行模型推理。  
为了加速和计算稳定性，HuggingGPT 在混合推理端点上运行这些模型。通过将任务参数作为输入，模型计算推理结果，然后将其发送回大型语言模型。为了进一步提高推理效率，可以对不具有资源依赖性的模型进行并行化。这意味着可以同时启动满足先决条件依赖关系的多个任务。

  * 混合端点：理想的场景是我们只在HuggingFace上使用推理端点。为了保持系统的稳定和高效，HuggingGPT在本地提取并运行一些常见或耗时的模型。局部推理端点很快，但覆盖的模型较少，而HuggingFace的推理端点则相反。因此，本地端点比HuggingFace的推理端点具有更高的优先级。只有在匹配的模型未在本地部署的情况下，HuggingGPT才会在HuggingFace端点上运行该模型。
  * 资源依赖性：尽管HuggingGPT能够通过任务规划来开发任务顺序，但在任务执行阶段有效管理任务之间的资源依赖性仍然具有挑战性。因为HuggingGPT无法在任务规划阶段为任务指定未来生成的资源。为了解决这个问题，我们使用一个唯一的符号<resource>来管理资源依赖关系。具体来说，HuggingGPT将先决任务生成的资源标识为<resource-task_id>，其中task_id是先决任务的任务id。在任务规划阶段，如果存在依赖于task_id任务生成的资源的任务，则HuggingGPT将此符号设置为任务参数中相应的资源子字段。然后在任务执行阶段，HuggingGPT将此符号动态替换为先决任务生成的资源。此策略使HuggingGPT能够在任务执行期间有效地处理资源依赖关系。



### 如何生成回答

在所有任务执行完成后，HuggingGPT 进入响应生成阶段。  
在这个阶段，HuggingGPT将前三个阶段（任务规划、模型选择和任务执行）的所有信息集成在一起，包括计划任务的列表、为任务选择的模型以及模型的推理结果。当然，最重要的是推理结果，这是 HuggingGPT做出最终决策的支持。这些推理结果以结构化格式出现，例如对象检测模型中具有检测概率的边界框、问答模型中的答案分布等。HuggingGPT允许LLM接收这些推理结果作为输入，并将具有置信度的响应汇总，返回给用户。  


## 限制与不足

尽管HuggingGPT 提出了人工智能解决方案的新范式，但仍然存在一些局限性或改进空间：

  * 任务规划严重依赖LLM的能力：大模型并不能100%确保生成的计划始终是可行且最优的，因此，探索优化LLM的方法至关重要，进而提升任务规划能力。
  * 效率：效率的瓶颈主要在使用大型语言模型的推理上。对于每一轮用户请求，HuggingGPT 都需要在任务规划、模型选择和响应生成阶段至少与大型语言模型进行一次交互，大大增加了响应延迟，并导致用户体验下降。
  * 最大上下文长度的限制：受 LLM 可以接受的最大令牌数量的限制，HuggingGPT 面临着最大上下文长度的限制。我们使用了会话窗口，在任务规划阶段只跟踪了会话上下文来缓解它。
  * 系统稳定性：一是在大型语言模型的推理过程中通常是不可控的。大型语言模型在推理时偶尔会不符合指令，并且输出格式可能会超出预期，从而导致程序工作流中出现异常。二是HuggingFace的推理端点上托管的专家模型的状态不可控。HuggingFace上的专家模型可能会受到网络延迟或服务状态的影响，导致任务执行阶段出现错误。



## 总结

HuggingGPT以语言为接口，将LLM与人工智能模型连接起来，来系统解决人工智能任务。  
LLM可以被视为管理人工智能模型的控制器，并且可以利用HuggingFace等机器学习社区的模型来解决用户的不同请求。通过利用LLM在理解和推理方面的优势来剖析用户的意图，并将任务分解为多个子任务。然后，基于专家模型描述，HuggingGPT为每个任务分配最合适的模型，并集成不同模型的结果。通过利用来自机器学习社区的众多人工智能模型的能力，HuggingGPT在解决具有挑战性的人工智能任务方面展现了巨大的潜力。

## 使用说明

HuggingGPT已经在GitHub上公开，并且持续在演进，地址如下：[https://github.com/microsoft/JARVIS](https://link.zhihu.com/?target=https%3A//github.com/microsoft/JARVIS)。  


从服务器下载和运行 HuggingGPT ：
    
    
    # setup env
    cd server
    conda install pytorch torchvision torchaudio pytorch-cuda=11.6 -c pytorch 
    pip install git+https://github.com/huggingface/diffusers.git
    pip install git+https://github.com/huggingface/transformers
    pip install -r requirements.txt
    # download models
    cd models
    sh download.sh
    # run server
    cd ..
    python bot_server.py
    python model_server.py

从 web 运行 HuggingGPT 的方法如下：
    
    
    cd web
    npm install
    npm run dev

## 参考资料

论文下载地址：[https://arxiv.org/pdf/2303.17580.pdf](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2303.17580.pdf)  
开源代码：[https://github.com/microsoft/JARVIS](https://link.zhihu.com/?target=https%3A//github.com/microsoft/JARVIS)

> 综 述：[【AI Agent洞察】01-Agents，通往AGI必由之路](https://zhuanlan.zhihu.com/p/666913254)  
> 上一篇：[【AI Agent洞察】05-Smallville：基于生成式智能体构建的虚拟社会](https://zhuanlan.zhihu.com/p/670665088)  
> 下一篇：[【AI Agent洞察】07-CAMEL：开创了沟通式智能体，多个Agent角色通过自主沟通完成任务](https://zhuanlan.zhihu.com/p/675696232)
