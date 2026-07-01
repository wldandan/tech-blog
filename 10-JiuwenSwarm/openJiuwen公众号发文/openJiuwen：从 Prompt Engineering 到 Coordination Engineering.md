# openJiuwen：从 Prompt Engineering 到 Coordination Engineering
openJiuwen117次阅读创建于2026-04-28更新于2026-06-08
从Prompt Engineering提示词工程、Context Engineering上下文工程，到如今爆火的Harness Engineering，围绕大模型的落地方法论持续升级，核心都是为了打磨单智能体的能力。
然而复杂任务日趋多元化，越来越多场景需要多智能体组队协作，分工完成信息调研、逻辑分析、任务执行、结果核验等环节。单一个体的能力优化已然存在明显上限，一套面向 AI 团队协同的全新研究方向，正在成为行业亟待解答的新课题：
**如何让多个智能体像一支精锐团队一样，自主分工、高效沟通、无缝协作？并且让这种协作能力可沉淀、可复用、可进化？**
近期我们发现，**华为支持的openJiuwen社区**给出了一套方案——
围绕**Coordination Engineering**这一下一跳工程范式，他们发布了一套完整的多智能体协同技术体系：**Agent Team**实现团队自主协作，业界首发**Team Skills**沉淀协作经验，**Team Skills Hub**打通共享生态，**Team Skills自演进**驱动团队持续进化。
Team Skills Hub平台：https://teamskills.openjiuwen.com/
从"单兵作战"到"精锐团队"，再到"团队能力可复制、可进化"，这套"全家桶"正在重新定义多智能体协作的工程边界。
## 关于JiuwenClaw
JiuwenClaw，是由华为2012实验室、华为云AgentArts与社区开发者联合，在openJiuwen开源社区共建的"小龙虾"AI Agent，主打"懂你所想，自主演进"。
项目地址：
https://atomgit.com/openJiuwen/jiuwenclaw
https://github.com/openJiuwen-ai/jiuwenclaw
同时，openJiuwen开源社区协同华为云AgentArts，打造了企业级办公智能体OfficeClaw，聚焦内容生成、文件处理、知识搜索等各类办公场景，助力企业办公效率跃升。
## 为什么需要Coordination Engineering？
当前行业对AI Agent的优化集中在Harness Engineering：提示词工程、工具编排、护栏机制、任务循环、工作空间管理——这些是让单个Agent从"能用"到"好用"的基础功。
但现实世界的复杂任务，往往不是一个Agent能搞定的。
全屋装修要硬装设计师、软装设计师、艺术顾问协同；200页技术报告需要10个方向的专家并行调研；医疗会诊要多科室专家联合研判。
这些场景需要的不是"一个更强的Agent"，而是"一支配合默契的团队"。
这就是Coordination Engineering要解决的命题：**团队编排、任务调度、通信协议、隔离机制、故障恢复、可观测性**——一整套让多个Agent从"各干各的"到"协同作战"的系统工程。
Coordination Engineering不是对Harness Engineering的否定，而是它的自然延伸。每个Teammate内部仍然是一个完整的Harness SDK Agent，拥有全部的Harness Engineering能力；而Coordination Engineering在此之上增加了协调层，让多个优秀的个体，组成一个高效的团队。
## Agent Team Engine：让团队"自主高效协作"
JiuwenClaw Agent Team是Coordination Engineering的核心载体。
其设计理念很直接：**模拟真实团队的协作方式**——一个Leader Agent负责需求分析、团队组建和任务规划；多个Teammate Agent各自领取任务、独立执行、汇报结果，通过共享工作区协同产出。
图片加载失败
### 分级自主协同：Leader智能编排，Teammate自主执行
传统的Multi-Agent方案往往需要手动编排——预先定义谁先谁后、谁跟谁通信。JiuwenClaw Agent Team则把这件事交给了Leader Agent自己。
Leader 负责全局统筹，用户只需提出需求，它便能动态组建团队、规划任务并梳理依赖关系，全程跟进进度、灵活调整协作安排。
Teammate 则自主独立作业，主动匹配认领任务、独立推进执行，遇阻及时求助，完工自动同步状态。
通过Team Workspace团队级共享工作区，各Agent之间无需手动互传文件，即可直接读写，天然共享。
整套体系以任务 + 消息双驱动运转，形成**层级化自主协同**模式，复刻人类团队高效分工、灵活协作的工作逻辑。
当然，如果想要预设团队角色和执行流程，可以使用Team Skills。
### 全生命周期管控：关键决策不放手，团队运转不停滞
团队协作不是无限权限，关键决策、敏感操作均需要Leader参与审批，避免恶意操作。
多成员协作最怕隐性停滞：Teammate异常僵死、任务长时间无人认领、已认领任务迟迟不完成、消息漏接等问题，会导致协作卡住。Agent Team通过内外部双**事件驱动机制**来规避风险，空闲Teammate主动认领待领任务、Leader识别超时任务并重新规划或换人、消息接收方优先处理未读。**任何单点问题都能在可控时间内被发现并处理**，不会阻塞整个团队协作。
### Agent Team实战
如让Agent Team组建装修团队，完成毛坯房—>硬装—>软装的全流程
视频加载失败
Agent Team会组建包含硬装设计师、软装设计师、艺术家的装修团队。各Agent分工明确、紧密协作，硬装软装有了初稿后，艺术家还会主动提供指导建议，过程中使用Seedream图像编辑Skill，轻松完成从硬装布局到软装搭配、艺术装饰的全流程设计。

## 从Agent Team到Team Skills：让协作能力"会沉淀、可复制"
Agent Team解决了**当下怎么协作**的问题。
但会话结束后，这些经验全部消失。下次遇到同类任务，Leader仍然要从零开始规划：需要几个角色、如何分工、谁先谁后、什么条件算完成。
**如何让团队协作不再从零开始，让优秀的协作模式可沉淀、可复用、可进化？**
正是**Team Skills**解决的问题。
openJiuwen社区重磅发布——**JiuwenClaw Team Skills**，这是**业界首个面向多Agent协作的标准化能力包规范**。
它将多智能体团队的协作流程、任务范式、沟通策略、执行规范沉淀为团队协作SOP——把Agent Team一次成功的协作全链路，包括需求拆解、团队组建、任务分配、通信机制、冲突处理、交付规范等，封装为标准化的团队技能，让"一支优秀团队"变成"一套可复制的团队能力"。
简单来说：
**Agent Team让团队"自主高效协作"，Team Skills让团队协作能力"会沉淀、可复制"。**
图片加载失败
### Team Skills长什么样
一个Team Skill，就是一个文件夹目录结构：
<team-skill-name>/
├── SKILL.md              ← 这个团队叫什么、干什么、成员有谁
├── roles/                ← 每个成员角色各自负责什么
│   ├── <role-a>.md
│   └── <role-b>.md
├── workflow.md           ← 大家怎么配合、执行顺序是什么
├── bind.md               ← 遇到问题怎么处理、边界在哪里
├── dependencies.yaml     ← 依赖哪些外部工具
└── examples/ | templates/ | assets/ ...  ← 自由扩展
结构非常简单。一个SKILL.md加上roles/里几个角色定义，就能组起一支可用的团队。简单任务两三个文件就够，复杂任务再按需逐步补充，几乎没有门槛。
### Team Skills怎么创建
JiuwenClaw不但提出了Team Skills标准，同时发布了配套创建技能"**团队技能自动生成专家（teamskill-creator）**"。
在Team Skills Hub平台上，下载该技能并安装到JiuwenClaw上，只需**输入一句自然语言描述**，即可自动生成完整的Team Skill。同时支持将现有单Agent Skill转化为Team Skill，或者修改已有的Team Skill，如增减角色、调整执行流程。
创建过程中， 还能自动检索当前已有的skill和工具，装配给对应成员；如果本地没有对应skill，还能通过**find机制**从技能市场（如SkillNet、ClawHub）上检索合适的Skill。
"团队技能自动生成专家（teamskill-creator）"在任意支持加载Skill的框架上（如OpenClaw，hermes-agent，JiuwenClaw等）均可使用。
以创建"多学科自动分诊的医疗专家团队"技能为例：
在Team Skills Hub上下载团队技能自动生成专家（teamskill-creator）
在JiuwenClaw上安装该技能
输入"帮我创建一个医疗专家会诊的团队技能，要求科目齐全，能根据用户的病情描述按需加载"
视频加载失败
最终生成了具备23位AI医学专科专家的团队技能，该技能可根据病情动态创建多个不同专科专家团进行会诊。
### Team Skills实战验证
**基于JiuwenClaw的多学科医疗专家团队联合会诊：**
用户输入"我最近浑身有点酸痛，能不能用团队技能帮我诊断一下"。
![视频3-Team Skills专家团会诊](/api/blogs/files/zh/机器之心_|_openJiuwen：从_Prompt_Engineering_到_Coordination_Engineering/视频3-Team Skills专家团会诊.mp4)
系统不会先让Leader手动决定"该找哪些专家"，而是先由分诊角色读取用户的症状信息，判断可能涉及哪些科室方向，再按需动态创建对应的专科专家成员，并为每位专家分配明确的分析重点。
分诊完成后，对应的专科专家被即时创建，同时展开并行分析；随后由主任医生统一汇总各方意见，输出一份完整的会诊报告。
整个过程不仅能看到建议方案，还能实时看到专家是如何被选中和创建的、哪些环节已完成、哪些角色正在并行工作、下一步由谁接力——整条协作过程可见、可追踪、可复盘。
**跨框架兼容**
Team Skills扩展的是Agent Skills开放标准，不依赖特定平台框架。通过使用teamskill-creator创建的Team Skill，已在Claude Code上验证可完全遵从。凡是支持Agent Skills标准的平台，都可以直接复用Team Skills的能力，在Claude Code、Cursor等支持多智能体协同的平台上零适配运行。
## Team Skills Hub：共建团队技能生态
技能沉淀下来了，怎么共享？
openJiuwen提供了**Team Skills Hub**平台，支持上传、检索、下载、维护团队技能。
图片加载失败
当前已内置了一批开箱即用的Team Skills，包含**开发编程、办公与生产力、内容创作、多模态与媒体、数据与科研、合规与法律、生活与健康、金融与理财**八大类别。
地址：https://teamskills.openjiuwen.com/
大家可以体验使用"团队技能自动生成专家（teamskill-creator）"生成Team Skills，并上传至Team Skills Hub平台共享——用你的协作经验，帮助更多人高效协作。
## Team Skills自演进：让团队越用越强
协作经验沉淀下来了，团队技能可以复用了——但这还不够。
真正的团队不会止步于"复制过去"——它需要在每一次实战中自我迭代，让团队整体与每位成员都越用越强。
JiuwenClaw现已为Team Skills带来**自演进能力**：
### 双层自演进：团队与成员协同进化
Team Skills自演进不是只在一个维度上优化，而是同时在**团队技能层**和**成员技能层**两个层面展开：
**团队技能层：** 系统根据任务执行轨迹自主演进Team Skills——增加成员角色、补充约束规则、优化协作流程等，让Leader Agent的任务规划与团队管控能力持续升级；
**成员技能层：** 每位成员的Skill同样自主进化——工具报错、接口超时等实战经验被自动沉淀，再次遇到同类问题时直接解决，不再重复踩坑。
团队在进步，个人也在成长——双层协同进化，让整个系统越跑越强。
### 演进补丁架构：经验独立存储，原始Skill不动
自演进带来的一个潜在风险是：改多了，原始技能被改得面目全非怎么办？
JiuwenClaw的做法是——**演进内容以独立的经验条目附加到Skills上，而非直接修改原始文件。**
每条经验携带触发来源、上下文、时间戳与质量评分，可单独审查与淘汰。Skills本身升级后，已积累的经验无缝沿用，不存在冲突。
这就像人类团队的"经验手册"与"操作规程"分开管理：规程是基础，经验是补充，互不干扰，各自迭代。
### 量化评估与生命周期管理
并非所有经验都值得保留，JiuwenClaw对每条演进经验进行**有效性、使用率、新鲜度**评分，更新优先级。用户可随时审阅，确保演进过程始终透明可控，效果不劣化。
## Team Skills 从创建到自演进全流程实战
以旅行规划团队为例，演示团队技能从创建、执行、到自演进的全流程：
![视频4-Team Skills创建到自演进实战](/api/blogs/files/zh/机器之心_|_openJiuwen：从_Prompt_Engineering_到_Coordination_Engineering/视频4-Team Skills创建到自演进实战.mp4)
自动生成的旅行规划团队技能包含交通专家、住宿专家、景区专家、以及方案汇总、预算审核五个专家，在执行过程中通过自演进增加了朋友圈文案创作专家角色。六位团队成员会根据用户旅行目的地、日期、预算等信息自主协作输出完整的旅行规划方案与朋友圈文案。
图片加载失败
**第一步，创建团队技能**
用户描述需求后，系统基于teamskill-creator自动生成完整的旅行规划Team Skill，同时支持搜索和下载社区已有的相关技能来增强团队能力。生成后即可查看成员列表与Mermaid协作流程图，团队结构一目了然。
**第二步，执行中自动沉淀经验**
户输入实际出行需求后，Agent Team使用该团队技能，创建专属旅行规划团队执行任务。
执行过程中，遇到冲突各成员可自主协商解决，如交通专家先规划了晚上到达的航班，景区专家规划了下午的行程，两者冲突，主动协商后将航班更新为上午。
执行完成后会系统可自动识别出费用审核与朋友圈文案存在职责耦合，随即生成演进经验并提交用户审批：建议拆分为两个专家并行完成。
**第三步，基于经验重构技能**
用户确认演进经验后，系统一键重构——自动新增专家角色、更新协作流程。与此同时，每位成员在任务执行中积累的实战经验也同步沉淀，团队层与成员层协同进化。
团队结构在自动调整，协作流程在持续优化，成员经验在逐步积累——用得越多，团队越强。

## 完整闭环：从协同到进化
回顾整条技术链路，openJiuwen围绕Coordination Engineering构建了一个完整闭环：
**Agent Team Engine**→ 让多智能体自主分工、高效协同，完成从"单兵作战"到"精锐团队"的关键跨越；
**Team Skills** → 多智能体的“开发平台”，将协作流程、经验标准化封装，让"一支优秀团队"变成"一套可复制的团队能力"；
**Team Skills Hub** → 打通共享生态，让协作经验在社区中流通、复用；
**Team Skills自演进** → 在每一次实战中自动迭代，让团队整体与每位成员越用越强。
从Agent Team到Team Skills，再到Team Skills Hub与Team Skills自演进，JiuwenClaw持续打通"单智能体好用—多智能体协同—团队能力沉淀—团队能力进化"的完整闭环，让Agent团队协作从"一次性组队"走向"团队化作战"。
每个Teammate内部仍然是一个完整的Harness SDK Agent，拥有全部的Harness Engineering能力；Coordination Engineering在此之上增加了协调层与进化层，进一步增强了Evolution Engineering能力——**你用得越多，团队越强。**
## 立即体验
JiuwenClaw最新版本已开源Agent Team与Team Skills全套能力，可以在Web页面或飞书频道切换集群模式体验。
以飞书频道为例，用户可使用/mode team切换到集群模式，输入请求即可使用多智能体协作。若想切换其他模式，可使用/mode agent.plan切换规划模式，/mode agent.fast切换性能模式。
当前，Team Skills Hub已经沉淀了开发编程、办公生产力提效、内容创作等多个场景的团队高效协作技能，大家可以即刻上手体验！
同时也可以使用"团队技能自动生成专家（teamskill-creator）"创建Team Skills，并上传到Team Skills Hub，共享你的协作经验，共同构建Team Skills生态，让Agent协作更智能、更高效！
