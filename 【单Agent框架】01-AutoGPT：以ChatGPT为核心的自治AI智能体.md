# 【单Agent框架】01-AutoGPT：以ChatGPT为核心的自治AI智能体

原文链接：https://zhuanlan.zhihu.com/p/668234147

---

​

目录

## 简介

[AutoGPT](https://zhida.zhihu.com/search?content_id=236626490&content_type=Article&match_order=1&q=AutoGPT&zhida_source=entity)于2023年3月30日由游戏公司Significant Gravitas Ltd.的创始人Toran Bruce Richards发布。而在发布后不久就成为GitHub上的顶级趋势开源库，此后在Twitter上一再流行。同年4月，Auto GPT成为国内外的热门话题，那AutoGPT到底是什么呢？

其实，AutoGPT是一个AI agent（智能体），也是开源的应用程序，结合了GPT-4和[GPT-3.5](https://zhida.zhihu.com/search?content_id=236626490&content_type=Article&match_order=1&q=GPT-3.5&zhida_source=entity)技术，给定自然语言的目标，它将尝试通过将其分解成子任务，并在自动循环中使用互联网和其他工具来实现这一目标，它由GPT-4驱动，自主地开发和管理业务。说简单点，你给AutoGPT一个题目，它会自己思考，给出实现的步骤以及实现细节。

与ChatGPT不同的是，用户不需要不断对AI提问以获得对应回答，在AutoGPT中只需为其提供一个AI名称、描述和五个目标，然后AutoGPT就可以自己完成项目。

例如：
    
    
    名称：Chef-GPT
    
    角色：一个旨在从网上找到普通食谱并将其转化为米其林星级食谱的人工智能。
    
    目标1：找到一个简单的在线食谱
    
    目标2：将这个简单的食谱转化为米其林星级版本

一旦AutoGPT收到了描述和目标，它将开始自行进行操作，直到项目达到令人满意的水平。

## 工作原理

如下图所示，我们以一个例子展示讲解AutoGPT架构和原理。

图片来自网络

  1. 用户（人类）定义 AI 代理的名称，并指定最多 5 个目标。
  2. 根据用户的设置，生成初始提示（prompt）并将其发送到 [ChatGPT API](https://zhida.zhihu.com/search?content_id=236626490&content_type=Article&match_order=1&q=ChatGPT+API&zhida_source=entity)。
  3. ChatGPT 返回一个 json 字符串（理想情况下），其中包括它的想法、推理、计划和批评。json 还包括下一个要执行的命令及其参数。
  4. 该命令是从 ChatGPT 的响应中提取和解析的。如果发出关闭/task_complete命令，则系统关闭。否则，适当的命令执行器使用给定的参数执行命令。
  5. 执行的命令返回一个字符串值。例如，谷歌搜索命令会返回搜索结果，browse_website命令会返回抓取网站内容的摘要，write_to_file会返回写入文件的状态等。
  6. ChatGPT 输出 (4) 和命令返回字符串 (5) 组合在一起，添加到内存中
  7. (6) 中的上下文被添加到短期记忆中，仅存储为文本。
  8. (6) 中的上下文也被添加到长期记忆中。AutoGPT支持 [Pinecone](https://zhida.zhihu.com/search?content_id=236626490&content_type=Article&match_order=1&q=Pinecone&zhida_source=entity)、本地数据存储等。
  9. 给定来自短期记忆 (7) 的最新上下文，查询来自 (8) 的长期记忆以获得前 K 个最相关的记忆片段。top-K 最相关的记忆被添加到提示中，如上图中{relevant memory}，回忆添加在"这让你想起过去的事件"下。
  10. 构建了一个新的提示，使用与初始提示 (2) 相同的指令、来自 (9) 的相关记忆，以及末尾的指令GENERATE NEXT COMMAND JSON。这个新提示用于调用 ChatGPT，重复步骤（3）到（10）直到任务完成，即 ChatGPT 发出task_complete/关闭命令。 



## 优势与局限性

### 优势

如上文所述，你给AutoGPT一个目标，它会在网络上搜索最佳信息，然后自主执行任务，并继续不断改进自己，而AutoGPT在每个提示后都会要求你的许可，以确保项目朝着正确的方向发展。

可以看出AutoGPT的反馈循环：计划 -> 批评 -> 行动 -> 阅读反馈 -> 计划

### 依托于强大的GPT

AutoGPT是基于强大的GPT-4和GPT-3.5语言模型构建的，它们充当机器人的大脑，帮助它思考和推理。

### 自主运行与迭代

AutoGPT可以自主地处理任务，在无人值守的情况下执行基于互联网的操作，如Web搜索、Web表单和API交互等，这样减少人力需求，从而降低运营成本，且自主迭代来尝试实现给定的目标，它还可以从过去的经验中学习，以改善其结果。

### 内存管理

AutoGPT可以通过与向量数据库集成，来保留上下文并做出更加明智的决策，就像是给机器人配备长时记忆，记住过去的经历，而实际上AutoGPT通过写入和读取数据库、文件，来管理短期和长期内存。

### 多功能性

总体来说，AutoGPT可以**读写文件** 、**浏览网页** 、审**查自己提示的结果** ，以及将其与所说的提示历史记录相结合。此外其还具有**互联网访问** 、**长期和短期内存管理** 、**用于文本生成的GPT-4实例以及使用GPT-3.5进行文件存储和生成摘要** 等优势功能。

### 局限性

### 开销极高

AutoGPT是基于GPT-3.5和GPT-4而建立起来的。而GPT-4的单个token价格为GPT-3.5的15倍。

假设每次任务需要50个step（较好状况下），每个step会花费6K tokens的GPT-4 使用量，Prompt（提示词）和Completion（回答）的平均每一千tokens花费是0.05美元（因为实际使用中回答使用的token远远多于提示词），汇率为1美元 : 6.8人民币，那么花费就是50*6*0.05*6.8=102人民币。

这仅仅只是理想状况下，而且假设了使用时Auto-GPT没有出现其他的问题（后续会提到），单次任务的成本就为100余元。这个成本显然是不可以被大规模应用的。

### 死循环现象

常见死循环现象在执行任务的时候，AutoGPT会将任务细化并分解。但是一旦遇到了一些GPT-4都无法处理的问题时，就会陷入自我循环，每一个step执行完后的动作都为"do_nothing"，而且下一个动作仍为这个。

但是每次都会将相同的Prompt交给GPT-4处理从而造成了极其大量的资源浪费现象。而且从目前来看并没有什么很好的解决方案。

### 执行/响应速度过慢

从网上一些博主的实践经验可知，AutoGPT执行速度/响应速度过慢。GPT-4的生成token的速度就比GPT-3.5慢许多，再加上脚本执行其它指令所消耗的时间就更长了。

以上都是一些明显的局限性，但AutoGPT仍是在当前条件比较有限的前提下对AI agent做出的一个有益尝试。

通过主任务生成子任务（也有人叫子智能体）的方法而让AI通过LLM脱离人类监督自行完成任务可能是未来的发展方向之一。

在可预见的未来，AI agent会进一步发展，为复杂问题的解决给出一种新式的答案。

## 应用前景

可以给自己下指令的 AI 揭示了非常多的应用可能性，根据国内外开发者的经验，支持包含做市场调研、生成博客大纲、开发应用程序、搭建网站、私人财务顾问等等多种场景。

比如，代码生成、自然语言描述和文档都是用户现在可以使用 AutoGPT 实现自动化的过程。因此，用户可以将编写代码或文档的任务委托给模型，从而节省他们的精力和时间。

以下具体用例是利用 AutoGPT 来提高他们的生产力、执行市场研究，甚至创建网站。

### 自动执行待办事项

用户可以使用 AutoGPT 创建待办事项列表，不仅可以概述任务，还可以自动执行这些任务。

例如，可以指示 AutoGPT 发现错误，将 GitHub 更新提交到存储库，并在 Slack 上或通过电子邮件报告问题。

### 自动市场调查

用户可以使用 AutoGPT 进行市场调查，以深入了解潜在的雇主，毕竟找工作也是市场调研。

例如，通过给定 AI 的目标，可以为某种"理想工作"选择一家理想的公司，然后确定公司的主要竞争对手，发现他们对雇用开发人员的偏好，并在求职或谈判中利用这些信息为求职者谋取优势。

### 个人自定义网站

用户可以使用 AutoGPT 将简历转换成视觉上吸引人的登陆页面，展示自己的技能、经验和成就。通过这个自定义网站可以提高知名度并在竞争中脱颖而出，还可以要求 AI 工具为社交网络创建具有专业外观的头像。

### 数据分析决策

用户给可以使用 AutoGPT 协助完成数据分析任务，扫描和解释数据以提出新的行动计划。

例如：

  1. 性能分析： AutoGPT 可以分析性能指标和日志，识别瓶颈或需要优化的区域。
  2. 功能优先级排序：根据用户反馈、软件使用数据或市场趋势，AutoGPT 可以帮助软件架构师确定在开发路线图中应优先考虑哪些功能或增强功能。 
  3. 预测性维护： AutoGPT 可以分析系统日志和操作数据，以预测潜在的故障点或需要维护的系统区域。通过主动解决这些问题，软件架构师可以减少停机时间并提高整体系统可靠性。
  4. 代码审查协助：通过分析提交历史和代码更改，AutoGPT 可以帮助识别潜在问题，例如代码重复或不一致的编码实践。
  5. 资源分配： AutoGPT 可以分析项目数据、时间表和团队绩效指标，以提供有关更有效地分配资源的建议。这可以帮助软件架构师做出数据驱动的人员配置、预算和项目管理决策。 



## 最新进展

在论文《Auto-GPT for Online Decision Making: Benchmarks and Additional Opinions》中，作者也探讨了AutoGPT智能体的能力和局限性，并在在线决策任务上对AutoGPT进行了深入的研究。在此过程中，作者还提出了一种新方法，即通过外部专家模型提供额外意见的方式来增强LLMs。

实验结果突出了AutoGPT风格agent通过直截了当的提示设计，成功地适应了复杂的在线决策任务，超过了专门为这些任务设计的基于模仿学习(IL)方法的基线模型。在为AutoGPT提供动力的基础LLM中，GPT-4显示出卓越的性能。

此外，还引入了一项创新方案，纳入外部专家模型的额外意见，进一步增强AutoGPT风格agent的决策能力，特别是有利于GPT-4。

而附加意见算法为AutoGPT风格的agent提供了一种轻量级的监督训练方法，在不需要对LLM进行广泛微调的情况下，可以提高性能。实验证明了这种方法的有效性和适应性，特别是对于具有易于收集的行动策略训练数据的任务。

## 体验AutoGPT

### 准备工作

一个海外云服务器或者本地也行，本地网络需要能访问到[OpenAI](https://zhida.zhihu.com/search?content_id=236626490&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)，直接按照以下步骤安装就行，这里以云服务器Ubuntu为例

注意要使用AutoPGT，将使用你的OpenAI账户中的积分。然而，你可以使用免费版本中包含的高达18美元

### 安装python3
    
    
    apt-get install python3

### 配置获取

  1. 获取ChatGPT账号的key，地址：[https://platform.openai.com/account/api-keys](https://link.zhihu.com/?target=https%3A//platform.openai.com/account/api-keys)
  2. （可选）获取PINECONE API key，地址[https://www.pinecone.io](https://link.zhihu.com/?target=https%3A//www.pinecone.io/)



### 安装

下载代码
    
    
    git clone https://github.com/Torantulino/Auto-GPT.git
    
    # 进入目录
    
    cd Auto-GPT
    
    # 安装依赖
    
    pip install -r requirements.txt

### 修改配置

修改.env.template文件为.env文件
    
    
    mv .env.template .env

编辑.env文件，把OpenAI的key写入
    
    
    vim .env

修改内容如下：

（图片来自网络）

### 运行
    
    
    python3 -m autogpt

就能在命令行运行AutoGPT，效果如下图：

（图片来自网络）

汉化版本已经有大神开源，流程与上面类似，请参考汉化版：[https://github.com/RealHossie/Auto-GPT-Chinese](https://link.zhihu.com/?target=https%3A//github.com/RealHossie/Auto-GPT-Chinese)

## 参考材料

  * GitHub地址：[https://github.com/Significant-Gravitas/AutoGPT](https://link.zhihu.com/?target=https%3A//github.com/Significant-Gravitas/AutoGPT)
  * 《From GPT to AutoGPT: a Brief Attention in NLP Processing using DL》：[https://www.researchgate.net/publication/370107141_From_GPT_to_AutoGPT_a_Brief_Attention_in_NLP_Processing_using_DL](https://link.zhihu.com/?target=https%3A//www.researchgate.net/publication/370107141_From_GPT_to_AutoGPT_a_Brief_Attention_in_NLP_Processing_using_DL)
  * 维基百科：[https://en.wikipedia.org/wiki/Auto-GPT](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Auto-GPT)
  * 中文官网：[https://autogpt.cn/](https://link.zhihu.com/?target=https%3A//autogpt.cn/)
  * 工作流程：[https://blog.csdn.net/bruce__ray/article/details/89667166](https://link.zhihu.com/?target=https%3A//blog.csdn.net/bruce__ray/article/details/89667166)
  * 安装过程：[https://cloud.tencent.com/developer/article/2275756](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/article/2275756)
  * 应用案例：[https://baijiahao.baidu.com/s?id=1766850686131011690&wfr=spider&for=pc](https://link.zhihu.com/?target=https%3A//baijiahao.baidu.com/s%3Fid%3D1766850686131011690%26wfr%3Dspider%26for%3Dpc)
  * 《Auto-GPT for Online Decision Making: Benchmarks and Additional Opinions》：[https://arxiv.org/pdf/2306.02224.pdf](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2306.02224.pdf)
  * 论文解读：[https://zhuanlan.zhihu.com/p/635368762](https://zhuanlan.zhihu.com/p/635368762)



本文作者：SebastianHan

更多文章

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【单Agent洞察】02-BabyAGI：首个可动态调整优先级的任务自驱动智能体](https://zhuanlan.zhihu.com/p/669412115)
