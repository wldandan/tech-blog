# 【单Agent框架】03-Smallville：基于生成式智能体构建的虚拟社会

原文链接：https://zhuanlan.zhihu.com/p/670665088

---

​

目录

## AI Agent

AI Agent（人工智能代理）是一种能够感知环境、进行决策和执行动作的智能实体，也可以理解为一种用于实现预定目标的代码或机制。其工作方式类似于人类代理，能够接收输入数据（例如传感器信息、文本、图像等），通过分析和处理这些数据，理解环境和任务要求，并做出相应的决策和行动。

AI Agent 是如何构建的呢？[OpenAI](https://zhida.zhihu.com/search?content_id=237166852&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)的Lilian Weng在博客发表了一篇文章：《[[LLM](https://zhida.zhihu.com/search?content_id=237166852&content_type=Article&match_order=1&q=LLM&zhida_source=entity) Powered Autonomous Agents](https://link.zhihu.com/?target=https%3A//lilianweng.github.io/posts/2023-06-23-agent/)》，这篇文章描述了 Agent 系统的全貌：

### **基于LLM的Agent系统概览**

在该系统中，Agent 是中心，可以处理各类复杂的“代理”任务；以下4个组件为Agent服务：

  * **Tools** ：提供工具，如计算、搜索、执行代码等；
  * **Memory** ：提供历史数据存储能力，防止交流过程中遗忘之前的信息；
  * **Planning** ：提供目标分解、反思、链式思考等能力，一般基于大语言模型（Large Language Model，LLM）设计；
  * **Action** ：即Agent具体执行的动作。



目前，AI Agent已经被研究应用于多个场景，如语音助手、自动驾驶、智能机器人等。LLM作为AI Agent的“大脑”，可以帮助AI Agent理解和处理各类复杂任务。近期，基于LLM强大的涌现能力，一个有趣名词逐渐受到越来越多人的关注：“Generative Agents（生成式代理）”。

## Generative Agents

什么是Generative Agents呢？Generative Agents一种利用生成模型来模拟可信人类行为的多智能体。在[斯坦福大学](https://zhida.zhihu.com/search?content_id=237166852&content_type=Article&match_order=1&q=%E6%96%AF%E5%9D%A6%E7%A6%8F%E5%A4%A7%E5%AD%A6&zhida_source=entity)发布的一篇学术论文《[Generative Agents: Interactive Simulacra of Human Behavior](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2304.03442v1.pdf)》中，有专业且生动的介绍：论文中描述了一个虚拟小镇，小镇中包含25个虚拟人物，皆以Agent为基础构造。Generative Agents背后是一个新的智能体架构，论文中所提出的Generative Agents架构如下图所示。在该架构中，Agents系统共分为感知、记忆和行动三大模块，其具体的工作模式如下：

  1. Agents能够感知其周边环境并获取各类信息，且这些信息都可以存储在记忆流系统中；
  2. Agents在感知到信息后，会在记忆流系统中检索并获取历史记忆，基于所检索到的结果决定其下一次行为；
  3. Agents基于记忆流系统，还能基于检索到的记忆形成长期的计划和创建更高级别的反思，且反思和规划的结果又能重新存储回记忆流中，以供未来使用。



### **Generative Agents架构**

Generative Agents架构的工作模式是不是与人类行为很像？在该论文中，作者为了更形象地描述Generative Agents，创建了一个虚拟沙盘场景：一方面，研究人员构建了一个名为“SmallVille”的虚拟小镇场景，并为小镇配置了很多可以互动的组件（如居民区、大学、药店、咖啡屋、酒吧、超市、公园等），如下图所示；另一方面，研究人员基于Generative Agents架构，构建了25个虚拟人物（生成式智能体），同时为每个人物编写了人设（如身份信息、家庭信息、人物性格、社交关系等）并创建了记忆流管理系统。这些虚拟人物可以基于自己的人设和记忆在小镇中自由交互，并模拟一些复杂的人类行为。

### **[Smallville](https://zhida.zhihu.com/search?content_id=237166852&content_type=Article&match_order=1&q=Smallville&zhida_source=entity) 沙盒世界**

以下举一个论文中提到的虚拟人物John Lin的例子：

  1. **人物背景** ：John Lin是一名Willow市场药店的店主，他热爱帮助人们。他总是在寻找让顾客获得药物更加便捷的方式。John Lin与妻子Mei Lin和儿子Eddy Lin一起生活，Mei Lin是一名大学教授，Eddy Lin正在学习音乐；John Lin非常爱他的家人；John Lin认识隔壁老夫妇Sam Moore和Jennifer Moore已经有几年了；John Lin认为Sam Moore是个善良友好的人；John Lin很熟悉他的邻居Yuriko Yamamoto；John Lin知道他的邻居Tamara Taylor和Carmen Ortiz，但没有见过他们；John Lin和Tom Moreno是Willows市场和药店的同事；John Lin和Tom Moreno是朋友，喜欢一起讨论当地政治；John Lin对Moreno家庭有一定的了解——丈夫Tom Moreno和妻子Jane Moreno；
  2. **日常行为** ：John Lin早上6点左右醒来，完成他的早晨常规活动，包括刷牙、淋浴和吃早餐。他与妻子Mei和儿子Eddy进行简短的交流后，便出门开始他一天的工作。



与John Lin一样，研究者所创建的其余24个虚拟人物在Generative Agents架构赋予的能力下，均可自行完成日常活动，并可以与其他虚拟人物进行社交互动，甚至可能涌现出复杂的未经设计的行为（如Sam想竞选市长、Isabella 筹办情人节派对等）。

Generative Agents能够基于动态演变的环境和经验不断地进行记忆、检索以及反思，同时还能与其他Generative Agents完成交互，且往往会涌现出逼近现实人类的复杂行为，而不仅是只能按照预先设置完成特定任务的“机器人”。

## 不足与展望

在该论文中，研究者向我们展示了Generative Agents强大的能力，那它是否存在局限性和问题呢？结论是肯定的。论文作者发现：

  * **记忆检索问题** ：由于Generative Agents的记忆量会随着时间推移和环境感知而不断增加，在越来越大的记忆流合集中，Generative Agents有可能检索到不恰当位置上的记忆来主导其后续的行为，这可能使得其行为随着时间的推移而变得越来越不可信；
  * **错误描述问题** ：Generative Agents的感知和行为在一定程度上由环境中各组件的描述决定。当开发者对某个节点的描述不够准确而产生歧义或者错误时，由于Generative Agents并不具有现实人类的主观或者客观判断，就有可能导致其做出与预期相悖的行为；
  * **指令影响** ：受指令的影响，Generative Agents的行事风格过于贴近大语言模型本身，导致其行事风格单一，在某些场合下可能并不适合；此外，Generative Agents也很少会拒绝。



除了斯坦福大学发布的论文中所述的角色扮演游戏场景外，Generative Agents还可以应用于更多复杂场景，如内容创作（如营销、广告、剧本等）、主题辩论、教育培训等。此外，Generative Agents可能对于目前大语言模型所存在的无法实时接入互联网、上下文知识欠缺、可能存在偏见以及容易被引诱等问题有一定的改进效果。

在AI大模型飞速发展的时代，Generative Agents会是大模型应用的未来吗？我们又能够基于Generative Agents做什么呢？开放给大家探讨和交流。

## 参考

  1. Park J S, O'Brien J, Cai C J, et al. Generative agents: Interactive simulacra of human behavior[C]//Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology. 2023: 1-22.
  2. Lilian Weng, LLM Powered Autonomous Agents. (2023-06-23) [2023-11-27]. [https://lilianweng.github.io/posts/2023-06-23-agent/](https://link.zhihu.com/?target=https%3A//lilianweng.github.io/posts/2023-06-23-agent/).



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【单Agent洞察】02-BabyAGI：首个可动态调整优先级的任务自驱动智能体](https://zhuanlan.zhihu.com/p/669412115)  
> 下一篇：[【单Agent洞察】04-HuggingGPT：面向多模态、多领域提供专业化解决复杂AI任务的智能体](https://zhuanlan.zhihu.com/p/673691859)
