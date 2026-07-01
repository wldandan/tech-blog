# 从Skill Insight到Agent Insight：一次以Harness为中心的演进

> 一句话：当AI可靠性的重心从"模型/能力"转移到"harness（智能体运行时骨架）"，观测与评测就必须从围绕单个skill升级到围绕整个agent。本文重点讲清：转向Agent Insight后我们已经做到了什么，以及接下来往哪走。

![alt text](skill-to-agent.png)

过去两年，AI工程的重心经历了一次安静但根本的转移。2023–2024 年，行业把精力几乎都放在"把模型/能力调得更好"上；到了 2025 年，大家发现一个尴尬的事实：**调得再好的模型，搬到生产环境照样会失败**。失败不在模型本身，而在包裹模型的那层运行时基础设施。Skill Insight是我们最初围绕单个能力（skill）建立的观测、分析、评测与优化体系，它解决的是"零件好不好"。但当可靠性的瓶颈转移到能力之外，只盯零件就不够了。Agent Insight正是为此而生。本文面向决策层，先讲清为什么这次转变是必然的，再重点讲转向Agent Insight后我们的当前能力与未来演进。

![alt text](skill-insight-to-agent-insight.png)

# 一、从Skill Insight转向Agent Insight：从"看零件"升级到"看整车"

![alt text](whytoagent.png)

**当AI的可靠性瓶颈从"模型/能力"转移到Harness（运行时骨架）之后，真正变化的不是"要不要做Harness"，而是它对insight能力提出了全新的诉求。** Harness的行为是动态、多步、跨会话的，一个任务能否跑通往往取决于上下文怎么管、何时算完成这些运行时逻辑。这意味着：观测要从单个skill上移到完整轨迹，但更核心的诉求，落在测评与优化这两项能力上。

**第一，对测评能力的新诉求：评测单位必须从"单个skill好不好"升级到"整个Agent轨迹好不好"。** 业界已明确提出"骨架感知的评测（scaffold-aware evaluation）"，主张把围绕模型的整个控制栈纳入被评测的系统；同时，失败必须能按结构化维度归因，而不只是打一个分，NeurIPS 2025的MAST就把多智能体失败归为系统设计、协作错位、验证缺失三大类。这些都是单skill评测看不见、也评不出的。

**第二，对优化能力的新诉求：优化必须从"人工调单个skill"升级到"评测信号驱动的闭环"。** 只评测不反哺，等于把诊断信息浪费掉；Harness时代要的是让评测结果直接驱动skill乃至骨架的自动改写与再验证。

代价是真实的：Gartner预测到 2027 年底超过 40% 的Aagentic AI项目会被取消、CMU的TheAgentCompany基准里最强模型也只自主完成 30.3% 的真实任务、MIT调研显示约 95% 的生成式AI试点没有可衡量回报。而这些失败，传统只盯单skill、停在token/延迟层的观测既看不见、更无法反哺。一句话：Skill Insight满足不了Harness时代对"测评 — 优化"的诉求，这正是必须转向Agent Insight的根本原因。

# 二、 转向Agent Insight：我们的当前能力与未来演进

## 2.1 我们当前已经实现的能力

Skill Insight 与 Agent Insight 不是替代，而是包含与升级——后者把前者作为子能力纳入，再补上只有在系统/Harness 视角下才看得见的部分。

| 维度 | Skill Insight（以skill为中心） | Agent Insight（以Agent为中心，当前已实现） |
|---|---|---|
| 观测 | 单个skill的输入、输出、表现 | 端到端的智能体行为：skill调用链、工具调用、规划决策的全链路 |
| 分析 | 单能力质量分析 | 跨步骤轨迹分析、协作关系分析 |
| 评测 | 单能力评测 | 任务成功率、轨迹正确性、工具调用正确性等系统级评测 |
| 故障诊断 | —— | 系统级根因定位（规格 / 协作 / 验证三类） |
| 优化 | 单能力优化 | 以skill评测为信号，驱动skill 的"生成—运行—诊断—优化"闭环 |

这套能力直接对应第一节的问题：端到端观测补上了"看不见Harness 层"的盲区；系统级评测把单点指标升级到了轨迹级；故障诊断把"出错了"变成了"为什么出错、错在哪一类"；而skill评测驱动的优化闭环，则把诊断信息变成了改进动作，而不是浪费掉。

## 2.2 我们的未来演进方向

![alt text](4pillars.png)

未来仍以评测为底座往前走——评测决定了能不能发现差距、能不能驱动优化。下面顺着以Agent框架为中心的四个维度（观测 / 评测 / 诊断 / 优化）逐一说明：中心是被服务的 Agent框架（Runtime / Harness），四个维度从外向内为它供给能力。

### 观测（Observation） 

看得见Agent的完整行为，是整套体系的"眼睛"，为后面的评测与诊断提供原始数据。我们已经从只盯单个skill的输入输出，做到端到端轨迹观测，记录skill调用链、工具调用、规划决策的全链路，让一个任务从接收到完成的每一步都可回溯。下一步是**从离线观测走向真实流量上的在线/生产监控**，并从线上轨迹自动沉淀评测数据集，让生产里跑过的真实任务反哺评测。

### 评测（Evaluation）

这是整套体系的核心，也是最值得强调的维度——它既是诊断与优化的依据，也是生产门槛的标准制定者。评测单位已经从"单个skill好不好"升级到"整个 Agent轨迹好不好"，覆盖任务成功率、轨迹正确性、工具调用正确性等系统级指标。

再往前，粒度要细到轨迹与多轮会话级，因为最强模型在单轮任务约 58% 成功率，一旦进入多轮对话就掉到约 35%，评测必须覆盖完整轨迹而非单步快照，并用经过验证的LLM-as-a-Judge流水线实现规模化打分。在此之上，最有价值的扩展是**横向对比评测**：用同一套任务集跑在不同模型（GPT、Claude、Gemini、开源模型）、不同平台/Harness之上，对比成功率、轨迹质量与成本，做成"选型基准 + 回归基线"，既回答"换哪个模型/平台更好"，也守住每次迭代后质量不回退。而要让跨厂商、跨平台的结果可比，前提是数据模型对齐OpenTelemetry GenAI语义约定，使AI可观测可度量、可比较、可互通。

### 诊断（Diagnosis）

诊断要回答的不是"错了"，而是"为什么错、错在哪一类"，把评测出的失败转成可定位、可归因的结论。当前按结构化维度归因，把失败归为规格/协作/验证三类（参考NeurIPS 2025的MAST失败分类法），而不是只打一个分。

未来则在生产里对**质量漂移（而不只是延迟）告警，并补上根因定位、护栏（guardrail）与合规审计**，从"能观测"走到"**可运作、可管控**"；保密与数据保护意识在当前模型里近乎为零，这类风险也需纳入评测与监控。

### 优化 · 生产化（Optimization）

这是整条链路的终点，也是最值得押注的方向：把诊断信息变成改进动作，而不是浪费掉。当前以skill评测为信号，跑通"生成—运行—诊断—优化"闭环，让诊断结果直接驱动改写与再验证。

未来从skill级扩展到**Harness级自动优化**，最终走向以评测为闸门的在线持续自进化，以DSPy生态的GEPA为代表的反思式优化，能从执行轨迹中诊断"为什么失败"，再自动进化出更优的prompt，在多个基准上以最多 35 倍更少的试验次数超越强化学习；研究也已开始把Harness本身当作搜索空间，让智能体自动改写自己的骨架。一句话：把失败轨迹喂给优化器，自动改写、自动验证，而非依赖人工调参。

四个维度串起来是一条链路：**观测看清楚，评测定标准，诊断找原因，优化自动改**。它们持续向中心的Agent框架供给能力。最终只有跨过评测设定的生产就绪门槛，任务成功率达标、轨迹正确、无安全/合规违规、成本与延迟在预算内（成本攀升、价值不清、风险管控不足正是Gartner列出的Agentic项目被取消主因），才允许上线。

# 这场演进的本质

把观测单位从零件换到整车，听起来像一次技术升级。更准确地说，是一次认知切换。

过去我们问的是"这个skill好不好"。现在要问的是"**这个Agent能不能在真实世界里干活**"。前者是能力评估问题，后者是**可靠性工程问题**。区别在于：能力评估可以靠benchmark解决，可靠性工程必须靠生产环境里的观测、评测、诊断、优化的闭环才能托住。

Agent普遍"能演示、难运作"，行业数据已经把这个结论砸实了。转向Agent Insight后，我们当前已经实现了端到端观测、系统级评测、结构化故障诊断，以及以skill评测为信号的优化闭环。

未来三条主线是向OpenTelemetry标准对齐、评测走向轨迹级与自动评判、以及把诊断推进到对skill与Harness的自进化优化。把这三条做扎实，Agent Insight就能成为我们智能体可靠性的核心基础设施。

# Agent Insight已经在公司内部正式开源啦！

嗨，各位伙伴！

很高兴告诉大家，Agent Insight已经正式开源啦！🎉🎉🎉

从现在起，项目的全部代码、文档都对内开放，每一位同事都可以自由访问、使用，也欢迎你参与进来，一起把它变得更好。

🔗 仓库地址（点击阅读原文可访问）：https://atomgit.com/openeuler/agent-insight
📖 文档/使用指南：https://atomgit.com/openeuler/agent-insight/blob/master/docs/user-guide/home.md

无论你擅长什么，这里都有你的舞台：

🐛 在实际使用中遇到问题？来提Issue，或直接反馈你的想法；

✨ 有新功能点子、想优化体验？欢迎提交设计或需求；

🔧 想动手写代码？认领任务、提交Merge Request，你的代码会被看见；

📝 擅长文档或测试？完善说明、补充用例同样价值巨大。

开源不只是开放代码，更是把大家的能力和智慧连接起来。期待你的一条关注、一条反馈、一次贡献，让Agent Insight成为更强大的Agent观测平台🚀

一起来玩，一起来造！

# 延伸阅读

- arXiv:2603.25723, *Natural-Language Agent Harnesses*（把 harness 显式化为可评测、可优化的对象）
- Xu et al., *TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks*（CMU，生产级任务基准）
- Cemri et al., *Why Do Multi-Agent LLM Systems Fail?*（NeurIPS 2025，失败分类法 MAST）

# 参考资料

[1] NJ Raman. "The Agent Harness: Why the Infrastructure Around Your LLM Is More Important Than the LLM Itself". Medium, 2026. https://medium.com/@nraman.n6/the-agent-harness-why-the-infrastructure-around-your-llm-is-more-important-than-the-llm-itself-3a6e5cbb2e97 （访问于 2026-05-29）

[2] N. D. Q. Bui. "Building AI Coding Agents for the Terminal: Scaffolding, Harness, Context Engineering, and Lessons Learned". arXiv:2603.05344. https://arxiv.org/html/2603.05344v1 （访问于 2026-05-29）

[3] The Menon Lab. "Meta-Harness: The Agent That Rewrites Its Own Scaffolding". 2026. https://themenonlab.blog/blog/meta-harness-self-evolving-agent-scaffolding （访问于 2026-05-29）

[4] Gartner. "Gartner Predicts Over 40% of Agentic AI Projects Will Be Canceled by End of 2027". 2025-06-25. https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027 （访问于 2026-05-29）

[5] Xu, F. et al. "TheAgentCompany: Benchmarking LLM Agents on Consequential Real World Tasks". arXiv:2412.14161. https://arxiv.org/abs/2412.14161 （访问于 2026-05-29）

[6] Cemri, M. et al. "Why Do Multi-Agent LLM Systems Fail?". NeurIPS 2025 / arXiv:2503.13657. https://arxiv.org/abs/2503.13657 （访问于 2026-05-29）

[7] Atlan. "AI Agent Observability: A Complete Guide for 2026 & Beyond". https://atlan.com/know/ai-agent-observability/ （访问于 2026-05-29）

[8] "Natural-Language Agent Harnesses". arXiv:2603.25723. https://arxiv.org/html/2603.25723v1 （访问于 2026-05-29）

[9] OpenTelemetry. "Semantic conventions for generative AI systems". https://opentelemetry.io/docs/specs/semconv/gen-ai/ （访问于 2026-05-29）

[10] OpenTelemetry. "Semantic Conventions for GenAI agent and framework spans". https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/ （访问于 2026-05-29）

[11] Datadog. "Datadog LLM Observability natively supports OpenTelemetry GenAI Semantic Conventions". https://www.datadoghq.com/blog/llm-otel-semantic-convention/ （访问于 2026-05-29）

[12] Maxim AI. "Top 5 AI Agent Evaluation Platforms". https://www.getmaxim.ai/articles/top-5-ai-agent-evaluation-platforms-in-2025/ （访问于 2026-05-29）

[13] Agrawal, L. et al. "GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning". arXiv:2507.19457. https://dspy.ai/api/optimizers/GEPA/overview/ （访问于 2026-05-29）

[14] GEPA (open-source repository). https://github.com/gepa-ai/gepa （访问于 2026-05-29）

[15] Huang, K.-H., Prabhakar, A. et al. (Salesforce AI Research). "CRMArena-Pro: Holistic Assessment of LLM Agents Across Diverse Business Scenarios and Interactions". arXiv:2505.18878. https://arxiv.org/abs/2505.18878 （访问于 2026-05-29）

[16] Confident AI. "Top 7 LLM Observability Tools" (生产轨迹评测、质量漂移告警与反馈闭环). https://www.confident-ai.com/knowledge-base/compare/top-7-llm-observability-tools （访问于 2026-05-29）
