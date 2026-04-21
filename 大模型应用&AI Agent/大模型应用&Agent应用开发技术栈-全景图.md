# 大模型应用&Agent应用开发技术栈-全景图

原文链接：https://zhuanlan.zhihu.com/p/694428893

---

​

目录

> 人类一直希望人工智能能成为人类的重要助手，协助人类解决各种复杂问题，完成各种各样繁琐的任务。AGI（Artificial General Intelligence，通用人工智能）将会帮助人们实现该愿景。AGI是指一种能够像人类一样思考、学习和执行多种任务的人工智能系统。

2025年将是Agent元年！当前，众多科技大厂如Google、AWS、微软、字节等纷纷积极布局Agent应用生态，推出Vertex AI Agent Builder、Amazon BedRock Agen、Coze、LangChain、[AutoGen](https://zhida.zhihu.com/search?content_id=242447727&content_type=Article&match_order=1&q=AutoGen&zhida_source=entity)等AI Agent产品服务/框架。并且，越来越多的企业开始依赖Agent来提升生产力、自动化流程和优化用户体验。无论是用于客户服务、内容生成，还是数据分析，Agent都展现出了巨大的潜力，成为推动智能化转型的关键力量。

## **“大模型应用”被定义为基于LLM的自主软件实体，能够感知环境、做出决策、采取行动（工具调用），并以类似人类的方式与用户或其他系统交互** 。

AI Agent是大模型应用典型、主要的应用范式。Agent技术正经历从​​单一功能体​​向​​协同智能系统​​的演进，其发展脉络清晰体现了人工智能从“工具化”走向“自主化”的范式转变。  
  
​早期的**单Agent，聚焦于在特定领域（如智能客服、数据分析）中高效完成目标明确的任务** 。然而，单Agent的能力存在天然边界。现实中复杂的问题往往需要多领域知识和分工协作，因此推动了多Agent系统（MAS）的兴起。**多Agent基于角色分工将多个专用Agent组成团队，再通过Agent之间的通信、博弈与协作机制，实现复杂任务的动态分解、执行和资源协调** 。到Agentic AI阶段，系统视角取代个体功能视角。**Agentic AI被描述为一种范式、架构或系统，由多个Agent构成，在复杂动态环境中能够主动感知、理解、规划、行动并持续学习和适应的AI系统** 。其具有自主性、目标驱动、环境交互、学习与适应性等关键特征。从单Agent到多Agent，再至Agentic AI，Agent技术从“单体智能”到“群体智能”，最终走向“生态智能”，推动行业从自动化走向类人自主化的长期自治。  
  
随着大模型的加入，大模型应用开发区别于传统应用的开发，因此，需要一个新的大模型应用开发技术栈。那么，大模型应用开发技术栈有什么独到之处呢？与基本的LLM聊天机器人相比，大模型应用开发是一个更具挑战性的工程问题，因为它们需要状态管理（保留消息/事件历史、存储长期记忆、在Agent循环中执行多个LLM调用）和工具执行（安全执行LLM输出的动作并返回结果）。因此，大模型应用开发技术栈和标准的LLM堆栈、乃至传统应用开发技术栈是不同的。大模型应用开发技术栈全景应该是什么样呢？如下所示，我们根据洞察分析，将**大模型应用开发技术栈** 全景定义为如下几层：

**一、大模型应用层**

主要是典型的大模型应用，如Agentforce、[Deep Research](https://zhida.zhihu.com/search?content_id=242447727&content_type=Article&match_order=1&q=Deep+Research&zhida_source=entity)等等，如下列举出部分常见场景下的典型Agent应用。

  * **企业 &办公**  
[钉钉AI助理](https://link.zhihu.com/?target=https%3A//extension.dingtalk.com/)、[Laiye](https://link.zhihu.com/?target=https%3A//laiye.com/)、[Lindy](https://link.zhihu.com/?target=https%3A//www.lindy.ai/)、[OpenAI Deep Research](https://link.zhihu.com/?target=https%3A//chatgpt.com/)、[Gemini Deep Research](https://link.zhihu.com/?target=https%3A//gemini.google.com/app)
  * **营销**  
[Agentforce](https://link.zhihu.com/?target=https%3A//www.salesforce.com/ap/)、[Marketingforce](https://link.zhihu.com/?target=https%3A//www.marketingforce.com/)、[regie](https://link.zhihu.com/?target=https%3A//www.regie.ai/)、[clay](https://link.zhihu.com/?target=https%3A//www.clay.com/)、[CueGrowth](https://link.zhihu.com/?target=https%3A//cuegrowth.ai/)
  * **金融 &财税**  
[蚂蚁数字科技](https://link.zhihu.com/?target=https%3A//antdigital.com/)、[百融云创](https://link.zhihu.com/?target=https%3A//www.brgroup.com/)、[众安信科](https://link.zhihu.com/?target=https%3A//www.zhongan.tech/)、[Auquan](https://link.zhihu.com/?target=https%3A//www.auquan.com/)、[埃森哲](https://link.zhihu.com/?target=https%3A//www.accenture.com/us-en/services/data-ai)、[Zest](https://link.zhihu.com/?target=https%3A//www.zest.ai/)、[Temenos](https://link.zhihu.com/?target=https%3A//www.temenos.com/innovation/ai/)
  * **客服和支持**  
[Cognigy](https://link.zhihu.com/?target=https%3A//www.cognigy.com/)、[Ada](https://link.zhihu.com/?target=https%3A//www.ada.cx/)、[Amelia](https://link.zhihu.com/?target=https%3A//www.soundhound.com/)、[Decagon](https://link.zhihu.com/?target=https%3A//decagon.ai/)、[Sierra](https://link.zhihu.com/?target=https%3A//sierra.ai/)
  * **编码**  
[Cursor](https://link.zhihu.com/?target=https%3A//cursor.com/)、[Windsurf](https://link.zhihu.com/?target=https%3A//windsurf.com/)、[Replit](https://link.zhihu.com/?target=https%3A//replit.com/)、[GitHub Copilot](https://link.zhihu.com/?target=https%3A//github.com/features/copilot)
  * **数据分析**  
[数势科技](https://link.zhihu.com/?target=https%3A//www.digitforce.com/)、[DataCanvas 九章云极](https://link.zhihu.com/?target=https%3A//www.datacanvas.com/)、[DataRobot](https://link.zhihu.com/?target=https%3A//www.datarobot.com/)
  * **个人助手**  
[Pi](https://link.zhihu.com/?target=https%3A//pi.ai/)、[AWS Alexa](https://link.zhihu.com/?target=https%3A//developer.amazon.com/en-US/alexa)、[华为小艺](https://link.zhihu.com/?target=https%3A//consumer.huawei.com/cn/mobileservices/celia/)、[Apple Siri](https://link.zhihu.com/?target=https%3A//www.apple.com/sg/siri/)、[Notion](https://link.zhihu.com/?target=https%3A//www.notion.com/)
  * **内容创作 &知识管理**  
[Jasper](https://link.zhihu.com/?target=https%3A//www.jasper.ai/)、[Grammarly](https://link.zhihu.com/?target=https%3A//www.grammarly.com/)、[Hebbia](https://link.zhihu.com/?target=https%3A//www.hebbia.com/)、[Hyperwrite](https://link.zhihu.com/?target=https%3A//www.hyperwriteai.com/)
  * **法律/合同**  
[Harvey](https://link.zhihu.com/?target=https%3A//www.harvey.ai/)、[Litera](https://link.zhihu.com/?target=https%3A//www.litera.com/)、[Robin](https://link.zhihu.com/?target=https%3A//robinai.com/)、[Legartis](https://link.zhihu.com/?target=https%3A//www.legartis.ai/)



**二、平台层**

**主要是业界流行的技术平台，如LangChain/LangGraph、Dify、Coze、AutoGen等。**

这一层中，又分成几个关键的组成部分：开发套件、应用框架（流程编排、观测、测试评估、提示词优化、规划、模型路由、工具、记忆、合规）、数据管理与检索。

**三、模型层**

**四、基础设施层**

大模型应用开发技术栈

本系列将会对大模型应用开发技术栈中的框架及关键技术进行分析，帮助大家由浅入深的学习大模型应用开发的核心技术和前沿进展。

## 一、大模型应用

  1. [【大模型应用】01-企业级Agent平台AgentForce](https://zhuanlan.zhihu.com/p/1923462173356691937)
  2. [【大模型应用】02-企业级Agent平台SAP AI洞察](https://zhuanlan.zhihu.com/p/1928135774634750816)
  3. [【大模型应用】03-企业级Agent平台Oracle Agent洞察](https://zhuanlan.zhihu.com/p/1934329882638284078)



### Deep Research

  1. [【Deep Research】01-Deep Research的全面综述：系统、方法与应用](https://zhuanlan.zhihu.com/p/1936367642064693029)
  2. [【Deep Research】02-Deep Research智能体：系统性洞察与分析](https://zhuanlan.zhihu.com/p/1938525208395879422)
  3. [【Deep Research】03-OpenAI Deep Research技术洞察](https://zhuanlan.zhihu.com/p/1939003557341802752)
  4. [【Deep Research】04-Gemini Deep Research技术洞察](https://zhuanlan.zhihu.com/p/1940785645107812348)
  5. [【Deep Research】05-Genspark Deep Research技术洞察](https://zhuanlan.zhihu.com/p/1942259499411935994)



### Agent入门到精通

  1. [【Agent入门到精通】01 - AI Agent的本质](https://zhuanlan.zhihu.com/p/1998733226689202164)
  2. [【Agent入门到精通】02-Agent架构设计：ReAct、ReWOO与思维链](https://zhuanlan.zhihu.com/p/1998817328603878485)
  3. [【Agent入门到精通】03-工具调用（Function Calling）：Agent的手和脚](https://zhuanlan.zhihu.com/p/1999201160813384313)
  4. [【Agent入门到精通】04-MCP协议深度解析：连接AI与数据源的标准化桥梁](https://zhuanlan.zhihu.com/p/1999203686707130939)
  5. [【Agent入门到精通】05-Workflow架构：可视化Agent编排平台](https://zhuanlan.zhihu.com/p/1999206497670947974)
  6. [【Agent入门到精通】06-Skills系统：Agent的模块化能力](https://zhuanlan.zhihu.com/p/1999207466928477997)
  7. [【Agent入门到精通】07-记忆系统：让Agent拥有上下文感知能力](https://zhuanlan.zhihu.com/p/2000654068553651951)
  8. [【Agent入门到精通】08-规划与推理：Agent如何分解复杂任务](https://zhuanlan.zhihu.com/p/2000654382052689377)
  9. [【Agent入门到精通】09-多模态Agent：视觉、语音与文本的融合](https://zhuanlan.zhihu.com/p/2000655295198823249)
  10. [【Agent入门到精通】10-上下文工程：Agent系统的”神经系统”](https://zhuanlan.zhihu.com/p/2003783359764112722)
  11. [【Agent入门到精通】11-Agent评估与优化：现代框架与实践指南](https://zhuanlan.zhihu.com/p/2003203050169463280)
  12. [【Agent入门到精通】12-Multi-Agent系统：协作、竞争与涌现](https://zhuanlan.zhihu.com/p/2003212333846118909)
  13. [【Agent入门到精通】13-生产级Agent架构：可靠性、安全性与可观测性](https://zhuanlan.zhihu.com/p/2003761515438811109)
  14. [【Agent入门到精通】14-AI Agent的未来：通往AGI之路](https://zhuanlan.zhihu.com/p/2007391883593283178)



### AI Agent工程化实战

  1. [【AI Agent工程化实战】01 - AI Agent工程化元年：从95%失败率看生产级落地的真正挑战](https://zhuanlan.zhihu.com/p/2010437035492664715)
  2. [【AI Agent工程化实战】02-Skill-insight：Agent skills从”感觉”到”量化”的进化之路](https://zhuanlan.zhihu.com/p/2014044608666019231)



## 二、应用开发平台

### **2.1 开发框架**

  1. [【AI应用工程】01-大模型应用开发之工程实践](https://zhuanlan.zhihu.com/p/685855883)
  2. [【AI应用开发框架】01-使用SGLang，实现基于大语言模型的高效编程](https://zhuanlan.zhihu.com/p/687578009)
  3. [【入门LangChain系列】01-LangChain介绍](https://zhuanlan.zhihu.com/p/665717963)
  4. [【入门LangChain系列】02-Model输入输出](https://zhuanlan.zhihu.com/p/669795717)
  5. [【入门LangChain系列】03-Chain](https://zhuanlan.zhihu.com/p/676911584)
  6. [【入门LangChain系列】04-Agent](https://zhuanlan.zhihu.com/p/679896083)
  7. [【入门LangChain系列】05-索引和记忆](https://zhuanlan.zhihu.com/p/681771807)
  8. [【入门LangChain系列】06-小白学LangChain：LangChain表达式语言（LCEL）](https://zhuanlan.zhihu.com/p/658836911)



### **2.2 Agent综述**

  1. [【Agent综述】01-AI Agent，通往AGI必由之路](https://zhuanlan.zhihu.com/p/666913254)
  2. [【Agent综述】02-多Agent：完成复杂任务的利器，迈向AGI时代的神器](https://zhuanlan.zhihu.com/p/32708623482)
  3. [【Agent综述】03-基于大型语言模型的多Agent：进展与挑战综述](https://zhuanlan.zhihu.com/p/693901264)
  4. [【Agent综述】04-新兴人工智能Agent架构全景：用于推理、规划和工具调用的综述](https://zhuanlan.zhihu.com/p/697643750)
  5. [【Agent综述】05-2023年人工智能体(AI Agent)开发与应用全面调研：概念、原理、开发、应用、挑战、展望](https://zhuanlan.zhihu.com/p/676288250)
  6. [【Agent综述】06-谷歌刚刚发布AI Agent白皮书，2025年AI Agent时代来临！](https://zhuanlan.zhihu.com/p/19466252143)
  7. [【Agent综述】07-AI Agent的基础设施](https://zhuanlan.zhihu.com/p/1908124676749791413)
  8. [【Agent综述】08-AI Agent圣经：未来发展趋势报告](https://zhuanlan.zhihu.com/p/1992993195878027887)



### **2.3 单Agent** 框架

  1. [【单Agent框架】01-AutoGPT：以ChatGPT为核心的自治AI智能体](https://zhuanlan.zhihu.com/p/668234147)
  2. [【单Agent框架】02-BabyAGI：首个可动态调整优先级的任务自驱动智能体](https://zhuanlan.zhihu.com/p/669412115)
  3. [【单Agent框架】03-Smallville：基于生成式智能体构建的虚拟社会](https://zhuanlan.zhihu.com/p/670665088)
  4. [【单Agent框架】04-HuggingGPT：面向多模态、多领域提供专业化解决复杂AI任务的智能体](https://zhuanlan.zhihu.com/p/673691859)
  5. [【单Agent框架】05-XAgent：采用双循环运转机制，自主解决复杂任务的通用智能体](https://zhuanlan.zhihu.com/p/681136067)
  6. [【单Agent框架】06-KG-Agent：一种基于知识图谱实现复杂推理的高效自治智能体框架](https://zhuanlan.zhihu.com/p/692978901)
  7. [【单Agent框架】07-具有超长语境要点记忆功能的人类启发式阅读智能体](https://zhuanlan.zhihu.com/p/699566756)



### **2.4 多Agent框架**

  1. [【多Agent框架】01-MetaGPT：面向编程的多智能体框架](https://zhuanlan.zhihu.com/p/668550781)
  2. [【多Agent框架】02-CAMEL：开创了沟通式智能体，多个Agent角色通过自主沟通完成任务](https://zhuanlan.zhihu.com/p/675696232)
  3. [【多Agent框架】03-Agents：首个采用SOP机制建立可控机制的智能体](https://zhuanlan.zhihu.com/p/682883749)
  4. [【多Agent框架】04-AutoGen：通过多Agent对话来构建LLM应用的智能体](https://zhuanlan.zhihu.com/p/685103609)
  5. [【多Agent框架】05-AgentScope：灵活而强大的多Agent平台](https://zhuanlan.zhihu.com/p/695452823)
  6. [【多Agent框架】06-基于LLM的多智能体协同决策多样化的选举方法](https://zhuanlan.zhihu.com/p/11075409117)
  7. [【多Agent框架】07-通过学习感知策略梯度算法实现多智能体合作](https://zhuanlan.zhihu.com/p/11985058911)
  8. [【多Agent框架】08-多智能体强化学习中的合作与公平性](https://zhuanlan.zhihu.com/p/14536702106)
  9. [【多Agent框架】09-具身LLM智能体学习有组织的团队合作](https://zhuanlan.zhihu.com/p/16131160730)
  10. [【多Agent框架】10-AWS BedRock多Agent关键技术分析](https://zhuanlan.zhihu.com/p/1891162001331450878)
  11. [【多Agent框架】11-AWS Multi-agent Orchestrator技术分析](https://zhuanlan.zhihu.com/p/1893252158327063392)
  12. [【多Agent框架】12-PydanticAI关键技术分析](https://zhuanlan.zhihu.com/p/1899832254022263668)
  13. [【多Agent框架】13-多智能体设计：通过优化提示词与拓扑结构优化智能体系统](https://zhuanlan.zhihu.com/p/1910732602626797678)



### 2.5 Agent设计模式

  1. [【Agent设计模式】00-AI智能体工作流](https://zhuanlan.zhihu.com/p/690202723)
  2. [【Agent设计模式】01-智能时代已至！Agent设计模式综述](https://zhuanlan.zhihu.com/p/711206099)
  3. [【Agent设计模式】02-AI应用开发范式：Agent设计模式综述](https://zhuanlan.zhihu.com/p/717380268)
  4. [【Agent设计模式】03-画像类模式](https://zhuanlan.zhihu.com/p/719977778)
  5. [【Agent设计模式】04-模型调用类模式](https://zhuanlan.zhihu.com/p/720938743)
  6. [【Agent设计模式】05-工具与行动类模式](https://zhuanlan.zhihu.com/p/722327514)
  7. [【Agent设计模式】06-编排类模式](https://zhuanlan.zhihu.com/p/888623055)



### 2.6 Agent Skill

  1. [【Skills】01-AI Agent架构革命：为什么Skills模式正在取代Workflow？](https://zhuanlan.zhihu.com/p/1997784551670448322)
  2. [【Skills】02-集成Skills的单智能体，能否终结多智能体？](https://zhuanlan.zhihu.com/p/1999195993162396617)
  3. [【Skills】03-Skill-Insight：给 Agent 装上"后视镜"，让Skills开始进化](https://zhuanlan.zhihu.com/p/2004914907972401040)
  4. [【Skills】04-Skill-insight：Agent skills从”感觉”到”量化”的进化之路](https://zhuanlan.zhihu.com/p/2014044608666019231)
  5. [【Skills】05- Agent Skill分析，如何实现技能的编排和评估](https://zhuanlan.zhihu.com/p/2014393687812110022)
  6. [【Skills】06- 如何实现Skills的自进化](https://zhuanlan.zhihu.com/p/2015436383439847936)
  7. [【Skills】07-告别Skill’盲盒’！Skill-insight三招让Agent精准可迭代](https://zhuanlan.zhihu.com/p/2019459824727897901)
  8. [【Skills】08-Skill的终局：不仅是执行，更是能自进化](https://zhuanlan.zhihu.com/p/2019809405609230673)
  9. [【Skills】09-SkillProbe：用“魔法”打败“魔法”——用Skill审计Skills](https://zhuanlan.zhihu.com/p/2020165820400054775)
  10. [【Skills】10-SkillRouter：破解大规模Skills选择难题的新范式](https://zhuanlan.zhihu.com/p/2020593144639562073)
  11. [【Skills】11-Trace2Skill：让智能体自己总结经验，自动生成可复用技能](https://zhuanlan.zhihu.com/p/2020916672396051215)
  12. [【Skills】12-D2Skill：双粒度动态技能库，驱动策略 - 技能协同进化的新范式](https://zhuanlan.zhihu.com/p/2022678212954657726)



## 三、框架

### 3.1 流程编排

  1. [【流程编排】01-AFLOW：自动生成智能体工作流](https://zhuanlan.zhihu.com/p/5605781955)
  2. [【流程编排】02-GPTSwarm：作为可优化图谱的语言智能体](https://zhuanlan.zhihu.com/p/6550590254)
  3. [【流程编排】03-通过程序合成进行自然语言命令](https://zhuanlan.zhihu.com/p/7098467577)
  4. [【流程编排】04-Gorilla：连接大模型和海量API](https://zhuanlan.zhihu.com/p/23382140815)



### 3.2 观测/调测

  1. [【观测/调测】01-通过eBPF实现AI应用的可观测性](https://zhuanlan.zhihu.com/p/1897702390850885133)



### 3.3 测试/评估

  1. [【测试/评估】01-Agent评估探讨](https://zhuanlan.zhihu.com/p/718248880)
  2. [【测试/评估】02-Agent-as-a-Judge: 用智能体评估智能体](https://zhuanlan.zhihu.com/p/13238264056)
  3. [【测试/评估】03-Thinking-LLM-as-a-Judge：基于规划和推理学习的评估方法](https://zhuanlan.zhihu.com/p/1908812520539534270)
  4. [【测试/评估】04-LLM-based Agents评估综述](https://zhuanlan.zhihu.com/p/1911378309859739418)
  5. [【测试/评估】05-多轮对话LLM智能体评估综述](https://zhuanlan.zhihu.com/p/1915489942370514690)
  6. [【测试/评估】06-IntellAgent：用于评估对话式AI系统的多智能体框架](https://zhuanlan.zhihu.com/p/1916575353981363579)
  7. [【测试/评估】07-LLM智能体幻觉分类和缓解](https://zhuanlan.zhihu.com/p/1961075945097102156)



### 3.4 提示词生成与优化

  1. [【提示词生成与优化】02-TextGrad：通过文本实现自动“微分”](https://zhuanlan.zhihu.com/p/9192012739)
  2. [【提示词生成与优化】03-ell：将提示视为函数的轻量级提示工程库](https://zhuanlan.zhihu.com/p/9941505513)
  3. [【提示词生成与优化】04-GRAD-SUM：利用梯度摘要优化提示工程](https://zhuanlan.zhihu.com/p/10673233589)
  4. [【提示词生成与优化】05-Prompt自优化框架洞察分析](https://zhuanlan.zhihu.com/p/29133814672)
  5. [【提示词生成与优化】06-CoD：少写快想，一种高准确率、低延迟、低成本的新提示技术](https://zhuanlan.zhihu.com/p/1898757867579900226)
  6. [【DSPy】01-Prompt工程或成为过去时，DSPy将会是拯救Prompt思维的利器](https://zhuanlan.zhihu.com/p/705107414)
  7. [【DSPy】02-DSPy：将声明式语言模型调用编译为自我改进流水线](https://zhuanlan.zhihu.com/p/705291734)
  8. [【DSPy】03-多标签分类的上下文学习](https://zhuanlan.zhihu.com/p/706118311)
  9. [【DSPy】04-RAG坦途已现！DSPy，将会革命性改变RAG系统的构建方式](https://zhuanlan.zhihu.com/p/706135629)
  10. [【DSPy】05-DSPy的“前世今生”，从DSPy的核心论文解析其技术演进之路](https://zhuanlan.zhihu.com/p/707184607)
  11. [【DSPy】06-Prompt或许的新未来， DSPy使用从0到1快速上手](https://zhuanlan.zhihu.com/p/707925423)
  12. [【DSPy】07-丝分缕解！带你了解DSPy核心模块的源码实现之Signature类](https://zhuanlan.zhihu.com/p/708332838)
  13. [【DSPy】08-丝分缕解！带你了解DSPy核心模块的源码实现之Module类](https://zhuanlan.zhihu.com/p/709324353)
  14. [【DSPy】09-丝分缕解！带你了解DSPy核心模块的源码实现之Optimizer类](https://zhuanlan.zhihu.com/p/709694815)
  15. [【DSPy】10-DSPy Visualizer：可视化Prompt优化过程](https://zhuanlan.zhihu.com/p/714277212)
  16. [【DSPy】11-DSPy和Langchain的无缝集成](https://zhuanlan.zhihu.com/p/714868095)
  17. [【DSPy】12-DSPy Optimizer进阶分析之BootstrapFewShot优化原理](https://zhuanlan.zhihu.com/p/716163290)
  18. [【PromptWizard】01-PromptWizard，Prompt工程上又一颗璀璨明星](https://zhuanlan.zhihu.com/p/25290259323)
  19. [【PromptWizard】02-Prompt又一选择，从0-1快速上手PromptWizard](https://zhuanlan.zhihu.com/p/26154065107)
  20. [【PromptWizard】03-PromptWizard源码解析](https://zhuanlan.zhihu.com/p/27109942600)
  21. [【PromptWizard】04-PromptWizard和DSPy上手实操案例&实验数据对比](https://zhuanlan.zhihu.com/p/27861853778)



### 3.5 规划

  1. [【规划】02-AutoGPT+P：利用大型语言模型进行基于情境的任务规划](https://zhuanlan.zhihu.com/p/694730389)
  2. [【规划】03-LLM整合多个函数调用的新思路！LLMCompiler：用于并行函数调用的LLM编译器](https://zhuanlan.zhihu.com/p/713091653)
  3. [【规划】04-Armap：通过自动化奖励建模与规划扩展智能体](https://zhuanlan.zhihu.com/p/1918318256994911365)



### 3.6 模型路由

  1. [【模型路由】01-OrchestraLLM：高效协调对话状态跟踪语言模型](https://zhuanlan.zhihu.com/p/700883845)
  2. [【模型路由】02-RouteLLM：利用偏好数据学习路由LLMS](https://zhuanlan.zhihu.com/p/21408688093)



### 3.7 工具使用

  1. [【工具使用】01-ToolLLM：促进大型语言模型掌握 16000 多个真实世界应用程序接口](https://zhuanlan.zhihu.com/p/32362542619)



### 3.8 记忆

  1. [【记忆】01-增强大语言模型 Agents 的工作记忆能力](https://zhuanlan.zhihu.com/p/696105075)
  2. [【记忆】02-基于大模型的Agents之间的记忆共享](https://zhuanlan.zhihu.com/p/703158721)
  3. [【记忆】03-在记忆中思考：回忆和后期思考使LLM具有长期记忆](https://zhuanlan.zhihu.com/p/17147052697)
  4. [【记忆】04-Letta，为AI智能体解锁记忆潜能的开发框架](https://zhuanlan.zhihu.com/p/29629889237)
  5. [【记忆】05-SRMT：面向多智能体终身路径规划的共享内存技术](https://zhuanlan.zhihu.com/p/1920569731095696375)



### 3.9 协议

  1. [【协议】01-MCP：AI时代的HTTP，大模型应用连接数据源的新桥梁](https://zhuanlan.zhihu.com/p/30949247510)
  2. [【协议】02-Google开源的A2A (Agent2Agent)协议，对多Agent会带来什么影响](https://zhuanlan.zhihu.com/p/1895434611774949228)
  3. [【协议】03-Token从15万降低至2千，使用MCP执行代码](https://zhuanlan.zhihu.com/p/1971238278771500576)



### 3.10 合规治理/安全护栏

  1. [【合规治理/安全护栏】01-Progent：首个面向AI Agent的可编程权限控制机制](https://zhuanlan.zhihu.com/p/1913314905567786294)
  2. [【合规治理/安全护栏】02-LLM Guardrails技术洞察](https://zhuanlan.zhihu.com/p/1949495420163199375)



### 3.11 上下文工程

  1. [【上下文工程】01-为AI Agent高效构建上下文](https://zhuanlan.zhihu.com/p/1992637122281228256)



## 四、数据管理与检索

### **4.1 RAG**

是一种结合信息检索和生成技术的方法，旨在通过检索外部知识库中的相关信息来增强大语言模型（LLM）的生成能力，提高生成内容的准确性和丰富性。

  1. [【RAG】01-浅谈RAG的十大挑战](https://zhuanlan.zhihu.com/p/696904158)
  2. [【RAG】02-RAP：针对多模态LLM Agent的上下文记忆检索增强规划](https://zhuanlan.zhihu.com/p/700207538)
  3. [【RAG】03-Agent检索增强生成：突破传统RAG局限，构建更加智能、贴近事实的LLM应用！](https://zhuanlan.zhihu.com/p/678681627)



**4.1.1 GraphRAG**

  1. [【GraphRAG】01-GraphRAG：从局部到全局，一种面向查询摘要的GraphRAG方法](https://zhuanlan.zhihu.com/p/1431617467)
  2. [【GraphRAG】02-微软&蚂蚁GraphRAG方案解析](https://zhuanlan.zhihu.com/p/1854007770)
  3. [【GraphRAG】03-GraphRAG Indexing介绍及源码解读](https://zhuanlan.zhihu.com/p/3115108247)
  4. [【GraphRAG】04-GraphRAG Query介绍及源码解读](https://zhuanlan.zhihu.com/p/3990498959)
  5. [【GraphRAG】05-小白上手GraphRAG](https://zhuanlan.zhihu.com/p/5115690518)



## 五 Agent工程

  1. [【Agent工程】01-Agent工程技术洞察、挑战以及解决方案](https://zhuanlan.zhihu.com/p/1968688671378093192)
  2. [【Agent工程】02-Agentic AI工程，生产级AI Agent的蓝图](https://zhuanlan.zhihu.com/p/1962188213138493758)
  3. [【Agent工程】03-AgentOps 综述：分类、挑战与未来方向](https://zhuanlan.zhihu.com/p/1966171570302083788)
  4. [【Agent工程】04-AgentOps：实现LLM智能体的可观测性](https://zhuanlan.zhihu.com/p/1972254110007943664)
  5. [【Agent工程】05-智能体工程已来：所有AI团队必须掌握的新范式](https://zhuanlan.zhihu.com/p/1996587006495580843)



## 六、模型层

  1. [【模型】01-xLAM：一系列增强AI Agent能力的大动作模型](https://zhuanlan.zhihu.com/p/1903090007566192858)
  2. [【模型】02-LLMOps：构筑企业级大语言模型即服务](https://zhuanlan.zhihu.com/p/1905351954235891921)



## 七、Agent实践

  1. [【Agent实践】01-手把手教你构建通用智能体](https://zhuanlan.zhihu.com/p/1900646554236347169)
  2. [【Agent实践】02-OpenAI构建通用AI Agent的实用指南](https://zhuanlan.zhihu.com/p/1906393292926592369)
  3. [【Agent实践】03-构建Agent的12原则](https://zhuanlan.zhihu.com/p/1951370323745284347)



## 其他相关

  1. [【AgentOS】01-AIOS：LLM智能体操作系统](https://zhuanlan.zhihu.com/p/691420682)
  2. [【Agent训练】01-不修改语言模型的情况下训练语言模型智能体](https://zhuanlan.zhihu.com/p/698325773)
  3. [【Agent失败模式】01-多智能体失败模式分析](https://zhuanlan.zhihu.com/p/1896206649586340104)



**端云协同**

  1. [【端云协同】01-MobA：实现移动端任务高效自动化的两级智能体系统](https://zhuanlan.zhihu.com/p/12421816522)
  2. [【端云协同】02-CAMPHOR：端侧设备上进行多输入规划和高阶推理的多Agents协作](https://zhuanlan.zhihu.com/p/25154092517)


