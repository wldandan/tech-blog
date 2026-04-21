# 【运维智能体】11-AWS DevOps Agent：自主云运维的未来

原文链接：https://zhuanlan.zhihu.com/p/1988179556159485443

---

​

目录

> 想象一位AI驱动的全天候队友，一旦监控警报响起，它便即刻启动工作，深入日志与代码，在人工介入之前，就已着手解决问题，这便是智能运维的愿景。2025年12月2日，AWS发布了一款名为AWS DevOps Agent的智能体，能够实现事件的及时解决和主动防御，持续提高系统的可靠性和性能。

## 背景：向自主化运维演进

传统云运维高度依赖人工：通过部署监控工具、设置告警规则，运维人员在收到告警后，需手动关联日志、指标与变更记录以定位故障。这种方式效率低、响应被动，且易导致“告警疲劳”——关键信号被海量通知淹没。虽然，当前[AIOps](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=AIOps&zhida_source=entity)利用机器学习技术实现了异常检测与告警聚合，提升了运维分析的水平，但是，其输出通常仍依赖人工执行修复操作。

如今，运维正迈向智能体驱动的新阶段：智能体不仅能发现问题，还可主动触发修复流程，实现从“分析”到“执行”的闭环。云运维正在从 “仅监控、只告警”迈向“**感知-分析-修复** ”的全流程自治时代。

行业预测，到2026年，超过60%的大型企业将部署由智能体驱动的智能运维系统。借助生成式AI与图分析技术，智能体能快速梳理日志、复盘故障，发现人眼难以捕捉的规律，推动运维从“静态告警”走向“持续学习与主动优化”。

## 什么是AWS DevOps Agent？

AWS DevOps Agent是一款运维智能体，可解决并主动预防事件，持续提高系统的可靠性和性能。它能够像经验丰富DevOps工程师一样，通过学习用户的资源拓扑结构与工具链，并关联可观测性工具、运维手册、代码仓库及CI/CD流水线的数据等，开展故障事件调查、找出根本原因并识别运维优化方向。DevOps Agent利用对运营和工作负载的深入了解，能够缩短解决问题的平均时间（[MTTR](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=MTTR&zhida_source=entity)），同时推动卓越运营。

该智能体与AWS生态系统深度兼容。它可无缝集成Amazon CloudWatch（指标、告警、日志）、AWS X-Ray（分布式追踪）、[AWS CloudTrail](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=AWS+CloudTrail&zhida_source=entity)（操作审计），以及 [Datadog](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=Datadog&zhida_source=entity)、Dynatrace、New Relic、[Splunk](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=Splunk&zhida_source=entity)等第三方可观测性平台。同时，智能体还能对接代码版本控制系统与构建流水线（如 GitHub、[GitLab](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=GitLab&zhida_source=entity)），从而掌握代码变更与部署历史。这意味着AWS DevOps Agent能够构建全局视角，全面覆盖应用代码、基础设施配置、运行时遥测数据及近期变更记录。

## AWS DevOps Agent架构

AWS DevOps Agent是一项全托管服务，采用双控制台架构，分别面向管理员与运维团队：

  * **管理员控制台（通过 AWS 管理控制台）**  
管理员可部署智能体，并定义智能体空间，一个智能体空间通常对应一个团队或一个工作负载。在空间中配置可访问的AWS账户、区域、外部工具，以及智能体使用的IAM角色与权限。
  * **运维团队控制台（独立Web应用或Slack/Teams集成）**  
面向SRE、运维工程师等一线人员，提供故障调查、分析拓扑结构图，以及审核、优化智能体给出的建议等功能。同时具备对话界面、故障历史查询及集成工具管理，可作为运维报告中心使用。



**数据与集成能力**

AWS DevOps Agent支持多源数据对接：

  * **AWS服务** ：支持读取CloudWatch（告警、日志、指标）、X-Ray追踪数据，采集CloudTrail事件、AWS Health事件等。
  * **第三方监控** ：内置了连接器，可对接主流监控系统（Datadog、Dynatrace、Splunk、New Relic）与代码管理/CI平台（GitHub、GitLab、[Jenkins](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=Jenkins&zhida_source=entity)等）。
  * **开发与协作工具** ：支持集成工单系统与即时通讯工具，如[ServiceNow](https://zhida.zhihu.com/search?content_id=268243129&content_type=Article&match_order=1&q=ServiceNow&zhida_source=entity)、Jira、PagerDuty、Slack、Microsoft Teams等平台。



当故障调查启动时，智能体会从CloudWatch或第三方工具中获取相关日志与指标，核查近期代码提交记录与流水线运行状态，并分析相关故障工单。所有传输与静态数据均经加密，服务端数据采用AES-256静态加密。

**安全与IAM**

  * **权限管控** ：每个智能体空间仅可访问指定的AWS账户。智能体通过IAM角色（跨账户角色或服务关联角色）获取权限，仅具备必要服务的读取权限；若启用自动操作，则按需授予部分写入权限。
  * **数据安全** ：不将用户数据用于模型训练。
  * **操作审计** ：每个决策和操作都会被记录，AWS CloudTrail会捕获智能体的API调用。



## AWS DevOps Agent的核心能力

AWS DevOps Agent整合了一整套运维能力，核心功能包括五大类：自主故障检测、根本原因分析（RCA）、修复方案建议、主动可靠性优化建议和提供统一运维全景视图。

### 自主事故检测

能够全天候监控状态，持续扫描系统异常。它可对接多种告警源（如CloudWatch、SNS、ServiceNow工单等），一旦发现异常，便自动启动故障调查流程。在实际操作中，用户可自定义触发智能体的告警或工单类型，智能体则保持持续监听状态。此外，用户还可通过交互式聊天界面按需触发智能体，或将其集成至CI/CD流水线，实现部署失败时自动调用智能体的功能。

### 根因分析

AWS DevOps Agent与可观测性工具、代码库和CI/CD流水线集成，以关联和分析遥测、代码和部署数据，同时共享其探索的假设、观察结果和根本原因调查发现。在故障触发后，智能体通过全维度采集指标、日志、追踪信息、配置文件及代码变更记录等数据，定位故障根源。它能够跨层级关联分析数据，避免了人工在多个仪表盘间切换的低效流程。

通过系统性调查，智能体能识别系统变更、输入异常、资源限制、组件故障及依赖项问题所引发的根本原因。其具体工作包括核查系统变更（如最新代码推送）、检测异常现象（如延迟或错误突增）、检查资源约束（CPU、内存、数据库限流等），最终锁定导致故障的具体组件或变更操作，并输出分析假设与观测结果，形成完整的根因分析报告。

在试点应用中，澳大利亚联邦银行曾借助该智能体在15分钟内排查出一项复杂故障，而同等任务通常需要资深工程师数小时完成。

### 自动修复建议

在定位故障根源后，智能体会立即提供可落地的修复方案。该方案涵盖具体的修复操作、验证步骤及必要的回滚流程。如：

  * 若故障由代码变更引起（如破坏SNS消息过滤器），则建议回滚变更。
  * 若因Lambda函数触发限流，则建议提高并发上限或配置预置并发。



总而言之，修复建议覆盖多个维度：回滚建议、自动扩缩容调整、资源配置修改（实际规格、数据库IOPS等）以及可观测性改进。每条建议均附有上下文与数据支撑，形成标准化方案交付用户。

此外，智能体支持将分析结果集成至用户工作流：可通过Slack、Microsoft Teams发送通知，在Jira或ServiceNow创建工单，并完整记录操作过程。

### 主动优化洞察

AWS DevOps Agent能通过持续学习历史故障数据，提供切实可行的主动预防建议。同时，它能够根据用户反馈不断优化建议，实现自我升级。如：

  * 针对重复告警，智能体会建议合并规则或调整阈值。
  * 发现负载不均衡时，提议设置自动扩缩容策略。
  * 检测到配置缺失（如高突发服务未配置HPA），会及时提示补充。
  * 关联故障与成本，如因配置不足导致的持续错误，它会指出由此产生的时间与资源浪费，并推荐优化方案。



### 统一的运维全景视图

在底层技术架构上，AWS DevOps Agent会实时扫描资源（计算、数据库、网络等）、配置和跨账户架构，构建并持续更新应用及其依赖关系的拓扑图，最终形成统一的故障分析视图。在控制台中，用户可直观查看故障影响的所有组件及其依赖关系。基于对拓扑的深度理解，智能体能够实现跨层级问题关联分析，如将网络 ACL配置错误关联到应用层异常。

此外，该视图可对接多云及本地工具，提供全系统视角的故障分析，打破数据孤岛。

## AWS DevOps Agent如何工作

AWS DevOps Agent故障修复流程包含以下六个步骤：

  1. **异常检测**  
由告警（如CloudWatch、Datadog）或工单（如ServiceNow）触发，智能体自动启动调查。
  2. **收集上下文**  
智能体收集相关日志（CloudWatch日志、应用日志等）、指标（CPU、延迟、错误率）、追踪数据（来自X-Ray或追踪工具）及部署历史，分析资源关联与变更记录，构建应用拓扑。
  3. **根因分析**  
通过机器学习模型和启发式规则来关联数据，提出探索假设并根据数据进行验证，最终定位根本原因（如代码缺陷、资源瓶颈等），并生成调查结果摘要。
  4. **生成修复方案**  
针对根因提供可操作方案，如回滚步骤、配置变更或资源调整等，每条建议附带验证方法，并可将方案推送至Slack或ServiceNow供团队执行。当前，方案执行需人工审批或触发流水线，智能体主要提供建议，也可通过其他AWS工具实现自动化部分修复工作，最终控制权由工程师自行决策。
  5. **长期改进建议**  
基于本次事件数据，提出改进措施，如完善告警规则、增强日志记录等，推动运维从“被动响应”转向“主动优化”。
  6. **人工审核与迭代**  
当前智能体以辅助角色输出分析与建议，由工程师审核执行并反馈效果，智能体据此持续优化未来的建议。



## 支持的集成

AWS DevOps Agent的设计理念是融入用户现有的工具链，其原生支持的集成工具包括：

  * **可观测性工具** ：Amazon CloudWatch（日志、指标、告警）、AWS X-Ray（追踪）；同时，也支持Datadog、Dynatrace、New Relic和Splunk等第三方应用性能监控（APM）与日志分析工具。这些系统中的任何告警或日志都可以提供给智能体，并且它能直接拉取遥测数据。
  * **CI/CD和代码仓库** ：对接代码版本控制与流水线系统，如GitHub、GitLab、AWS CodePipeline、Jenkins等。这使得它能检查最近的提交差异、查看部署日志，并建立故障事件与具体版本的关联关系。
  * **聊天运维与协作工具** ：支持向Slack、Amazon Chime、Microsoft Teams等渠道推送更新通知。用户也可通过聊天界面向智能体发起查询，获取故障分析解读。
  * **工单与事故管理** ：支持与ServiceNow、Jira或Zendesk集成，通过创建或更新故障工单触发调查流程，同时智能体可将分析结果写入工单系统。



同时，AWS DevOps Agent还支持通过MCP服务器实现自定义集成。

## 总结

当前，AIOps通过整合大数据、机器学习和自动化技术，对IT运维数据进行实时分析与处理，实现了异常检测与告警聚合等能力，但其输出通常仍依赖人工执行修复操作，无法实现故障的自主化修复。随着Agent自主感知、决策等技术的引入，AgentOps正成为AIOps演进的重要方向 —— 它通过构建具备环境感知、智能决策与自主执行能力的智能体（Agent），推动运维系统从“辅助分析”走向“主动治理”，实现故障的“感知-分析-修复”的全流程自治。

## 相关链接

  1. [https://aws.amazon.com/cn/devops-agent/features/](https://link.zhihu.com/?target=https%3A//aws.amazon.com/cn/devops-agent/features/)
  2. [https://www.aboutamazon.com/news/aws/amazon-ai-frontier-agents-autonomous-kiro](https://link.zhihu.com/?target=https%3A//www.aboutamazon.com/news/aws/amazon-ai-frontier-agents-autonomous-kiro)
  3. [https://dev.to/aws-builders/aws-devops-agent-the-future-of-autonomous-cloud-operations-3360](https://link.zhihu.com/?target=https%3A//dev.to/aws-builders/aws-devops-agent-the-future-of-autonomous-cloud-operations-3360)
  4. [https://dev.to/aws-builders/aws-devops-agent-explained-architecture-setup-and-real-root-cause-demo-cloudwatch-eks-ng7](https://link.zhihu.com/?target=https%3A//dev.to/aws-builders/aws-devops-agent-explained-architecture-setup-and-real-root-cause-demo-cloudwatch-eks-ng7)
  5. [https://www.youtube.com/watch?v=xaJIeU-xSy4](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DxaJIeU-xSy4)



> 技术全景图：[智能运维&AgenticOps技术栈-全景图](https://zhuanlan.zhihu.com/p/1995167173094695116)  
> 上一篇：[【运维智能体】10-STRATUS：面向现代云平台自主可靠性工程的多智能体系统](https://zhuanlan.zhihu.com/p/1987601362868011813)  
> 下一篇：[【运维智能体】12-RCAgent：基于工LLM 自主智能体的云平台根因分析](https://zhuanlan.zhihu.com/p/1989415079091909901)
