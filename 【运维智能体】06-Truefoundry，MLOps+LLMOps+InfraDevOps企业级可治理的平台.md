# 【运维智能体】06-Truefoundry，MLOps+LLMOps+InfraDevOps企业级可治理的平台

原文链接：https://zhuanlan.zhihu.com/p/1979247270974223914

---

​

目录

## 1 概述

[TrueFoundry](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=TrueFoundry&zhida_source=entity)是一款企业级一体化AI工程平台（AI Engineering + [AI Gateway](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=AI+Gateway&zhida_source=entity)），支持用户在自有云或[Kubernetes](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=Kubernetes&zhida_source=entity)环境进行部署。它集成了从模型/应用开发、部署、观测到治理与成本管控的全生命周期管理，并提供一个与[OpenAI](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity)兼容的AI网关，无缝连接上千种模型与MCP工具生态。

## 2 技术方案

AI Engineering平台与AI Gateway共同构成了企业级AI应用的核心支撑体系，前者实现了从模型开发到生产运维的端到端自动化生命周期管理，而后者则作为统一入口，提供了对多模型资源的智能调度、安全管控与可观性。

**AI Engineering（AI应用与模型生命周期管理平台）**

  * 帮助企业实现从模型开发到生产落地的端到端自动化，覆盖模型与服务的构建、部署、版本管理及回滚全过程。
  * 支持工作流、批处理、RAG、Agent等多种推理任务的统一编排与管理。
  * 提供动态GPU资源调度与弹性伸缩能力，提升资源利用率。
  * 内置实验追踪、性能基准测试与端到端监控，保障模型表现与业务效果。



**AI Gateway（多种资源的统一入口）**

AI Gateway是应用与多个LLM服务商/MCP Servers之间的统一入口，提供统一的模型API。

  * 其具备智能路由与自动回退机制，可根据策略选择最优或备用模型。
  * 支持访问控制、配额、审计与速率限制、成本统计与可观测性。
  * 支持多团队协作、混合云部署等复杂场景。



## 3 关键能力解析

**智能调优**

  * **推理加速预设** ：原生集成[NVIDIA NIM](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=NVIDIA+NIM&zhida_source=entity)与TRT-LLM，提供面向吞吐量或时延的预设配置选项，并支持多精度GPU选项，显著降低人工调优成本。
  * **端到端追踪** ：基于内置[OpenTelemetry](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=OpenTelemetry&zhida_source=entity) tracing实现全链路追踪，可对延迟、成本、Token使用量及API成功率等关键指标进行采集与分析，并无缝对接现有可观测体系。
  * **模型路由策略** ：支持基于用户预设规则（如低延迟优先、低成本优先、地理位置等）的智能路由机制，Gateway可自动选择模型或备用模型，提升服务稳健性与资源利用效率。路由规则通过UI界面或者YAML声明式配置进行管理，包含顶层model_configs（如限额、失败容忍）与rules（路由策略）部分。 
    * **流量分配路由** ：在规则中定义每个目标模型的权重（如90%流量到模型A，10%流量到模型B），系统按权重分配在“健康”模型中。
    * **延迟路由** ：系统计算各模型近期的平均单Token延迟，将流量尽可能导向延迟最低（或“接近最快”的模型”）的端点。



**智能运维**

主要是支持多区域、多集群的统一治理与观测，需要用户预配置规则。

**智能部署**

TrueFoundry支持一键部署RAG组件，以及声明式部署在线推理API、批处理、消息队列异步推理等多形态应用。

  * **声明式部署** ：基于Kubernetes与GitOps理念，采用YAML文件声明服务期望状态，涵盖镜像、CPU/GPU/内存资源、模型路径、API端点、环境变量等配置。用户通过CLI或Git提交配置后，平台的控制层会读取该文件，并自动在底层的Kubernetes集群中完成资源调度与应用部署。
  * **RAG部署** ：2024年推出[Cognita框架](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=Cognita%E6%A1%86%E6%9E%B6&zhida_source=entity)，支持模块化构建RAG应用。系统内置数据加载器、解析器、嵌入模型、重排器、存储库等核心组件的基类模板，用户可通过SDK接口注册自定义模块。部署完成后，用户可通过API调用或专用UI界面进行检索查询，兼顾开发灵活性与非技术用户的易用性。



## 4 Llo11yPop实践

NV团队基于TrueFoundry平台成功构建并部署了一套新型多智能体对话系统与领域专用机器学习模型。该系统利用从GPU集群采集的各类遥测数据（包括利用率、功耗、内存使用情况及错误信息等），实现了对GPU资源利用率的自动化评级，并为工作负载优化提供建议。

系统架构与流程如下：

  * 采用主管智能体进行任务规划与调度，统筹多个领域的分析智能体（MoA），分别负责GPU参数、Slurm数据和系统日志等遥测数据的分析。
  * 分析过程通过查询智能体调用Elasticsearch工具获取数据，支撑各MoA完成分析任务。
  * 系统运行遵循[OODA循环](https://zhida.zhihu.com/search?content_id=267125745&content_type=Article&match_order=1&q=OODA%E5%BE%AA%E7%8E%AF&zhida_source=entity)机制（观察、定向、决策、行动），由监督智能体自主执行，实现闭环优化。在引入OODA循环之前，所有流程均基于提示词工程实现。



在项目初期，团队人员参与验证集群优化建议的有效性，并在任务不适用时提供反馈，从而构建了一个基于人工反馈的强化学习（RLHF）循环，使系统能够不断迭代和改进。

TrueFoundry平台为系统提供了跨云及本地资源的统一管理能力，显著提升了部署和验证效率。

## 5 总结

TrueFoundry作为企业级AI工程平台，通过AI Engineering平台和AI Gateway两大核心组件，为企业提供从模型开发、部署到运维治理的全生命周期管理，并支持在自有云或Kubernetes环境中灵活部署。在智能运维方面，平台已具备多区域、多集群的统一治理与观测能力，允许用户通过预配置规则实现自动化运维。展望未来，智能运维将朝着更自主、更智能的方向发展，通过深度集成AI与自动化技术，实现故障预测、根因分析、动态资源优化以及自适应策略调整，从而进一步提升系统稳定性、资源利用效率与运维自动化水平，为企业大规模、跨环境的AI应用提供坚实支撑。

## 相关链接

  1. [https://www.truefoundry.com/](https://link.zhihu.com/?target=https%3A//www.truefoundry.com/)
  2. [https://docs.truefoundry.com/docs/create-and-setup-your-account](https://link.zhihu.com/?target=https%3A//docs.truefoundry.com/docs/create-and-setup-your-account)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【运维智能体】05-Davis® AI，因果、预测和生成式人工智能技术的结合的智能运维技术](https://zhuanlan.zhihu.com/p/1977762826962625486)  
> 下一篇：[【运维智能体】07-DeepFlow智能体，依托大模型自主实现业务连续性保障，提升运维效率](https://zhuanlan.zhihu.com/p/1980318597487301073)
