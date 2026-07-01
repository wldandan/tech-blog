# 利用 JiuwenSwarm AgentSwarm 打造自动化研发团队
万少 openJiuwen 2026年5月20日 18:09 江苏 听全文

本文介绍如何通过 **JiuwenSwarm AgentSwarm** 构建自动化研发团队，实现字幕软件开发、AtomGit Issue/PR 智能处理与飞书文档同步。
## JiuwenSwarm 平台概述
**openJiuwen** 是一个开源的 AI Agent 平台，其核心目标是为开发者提供灵活、强大且易于使用的 AI Agent 开发与运行环境。
借助这一平台，开发者能够迅速搭建起可处理各种复杂度的 AI Agent，实现多个 Agent 之间的协作互动，从而高效地打造出生产级别的可靠 AI Agent。同时，该平台也帮助企业和个人用户快速构建自己的 AI Agent 系统或平台，促进商用级 Agentic AI 技术的广泛落地应用。
### 系统架构
openJiuwen 的整体架构由三大核心模块构成：**openJiuwen Core**、**openJiuwen Studio** 以及 **openJiuwen Ops**。

在开发阶段，openJiuwen 提供了 Agent 编排与构建能力，让开发者能够快速创建 Agent 并进行高效开发。
### 预置智能体类型
目前，openJiuwen 提供了两种预置智能体类型，为用户提供丰富的功能和灵活的开发选项，以适应不同场景下的智能化需求：
• **ReActAgent**：这是一种基于 ReAct（推理 + 行动）规划范式的 Agent。它通过「思考 → 行动 → 观察」的循环迭代过程来完成任务。ReActAgent 拥有强大的多轮推理和自我纠错能力，能够进行动态决策并灵活适应环境变化，特别适合那些需要复杂推理和策略调整的任务场景。• **Workflow Agent**：这是一种面向多步骤任务的流程自动化 Agent，严格按照用户预设的任务流程来高效执行复杂任务。它强调通过预定义流程实现任务的规范化与高效执行，适用于任务结构清晰、能够分解为多个明确步骤的场景。
## JiuwenSwarm 概述
它是一款基于 openJiuwen Python 框架开发的 AI Agent，天然拥有良好的生态兼容性和扩展能力。在模型接入方面，JiuwenSwarm 支持华为云 MaaS 等主流模型平台，用户只需简单配置即可调用华为云的模型服务。此外，JiuwenSwarm 已实现与小艺开放平台的无缝对接，华为手机用户可以通过小艺直接唤醒并使用 JiuwenSwarm，享受随叫随到的智能服务。

## 什么是 AgentSwarm
从 Prompt Engineering（提示工程）、Context Engineering（语境工程）演进至当下备受瞩目的 Harness Engineering（驾驭工程），AI 领域的工程范式始终处于高速迭代之中。
目前，针对单一智能体的“驾驭与治理”已逐渐成为行业标配，但如何让多个智能体仿若一支精锐团队，实现自主分工、高效沟通与无缝协作，依然是横亘在行业面前的核心挑战。
恰逢这一趋势风口，获华为支持的 openJiuwen 社区正式发布了最新的 JiuwenSwarm 版本。该版本重磅新增了对 AgentSwarm 多智能体协同能力的支持，并前瞻性地提出 Harness Engineering 的下一个演进方向是“Coordination Engineering（协同工程）”，成功将多智
能体自主协同从抽象的概念转化为可直接落地的实战场景。

## 飞书群中添加机器人
**参考文档**：飞书频道配置指南
在飞书群聊中，添加创建好的飞书机器人后，用户可以直接在群聊中对话。飞书机器人负责接收用户的指令，并代理用户与 **JiuwenSwarm** 进行交互。

## 开启 AgentSwarm
在飞书对话中，输入以下命令即可主动开启 AgentSwarm 模式：
/mode team
然后可以直接输入需求来安排任务。

## 工作流程
整体工作流程如下图所示：

AgentSwarm 的标准协作流程包含以下步骤：
1. **Leader** 接收到任务后，开始拆分任务
2. **Leader** 布置任务
3. **Teammate** 认领任务
4. **Teammate** 执行任务
5. **Leader** 和 **Teammate** 之间相互通信
6. **Teammate** 汇报任务进度
7. **Leader** 总结工作
以下是各阶段的操作截图：
**步骤 1-2：任务拆分与布置**

**步骤 3-4：任务认领与执行**





**步骤 5-6：通信与汇报**


**步骤 7：总结**

### 结束
当所有任务完成后，Leader 会输出最终的总结报告。

## 自动处理 PR
**工具地址**：[https://atomgit.com/GitCode/ag-cli]
可以在 **JiuwenSwarm** 中配置 **AtomGit** 的 CLI 工具，设置监听后自动处理审核和合并 PR。
⚠️ **注意**：监听需要提供公网域名，用来接收 AtomGit 的 Webhooks 通知。
**AtomGit** 提供了 ag-cli 工具，可以在 **JiuwenSwarm** 中接入。接入后，**AgentSwarm** 即可通过它来管理代码仓库，并自动实现 PR 的管理。

可以参考文档在系统中安装 ag-cli，也可以直接将文档提供给 **JiuwenSwarm** 进行安装。

安装到最后，需要手动进行授权。


### 模拟生成 PR
PR 的生成主要演示了 **JiuwenSwarm** 有能力接管 **AtomGit**。


## 自动生成飞书日报
**参考文档**：[飞书 CLI 官方文档]

飞书推出了 lark-cli 终端工具，可以在 **JiuwenSwarm** 中接入和使用。接入后，**JiuwenSwarm** 便具备了操作飞书文档的能力。
当然，**JiuwenSwarm** 本身已接入飞书机器人，也可以直接给该机器人开通相关权限来操作飞书文档。
### JiuwenSwarm 接入飞书 lark-cli

### 授权给 lark-cli


## 总结
通过 **JiuwenSwarm AgentSwarm** 的 Leader-Teammate 协作框架，我们可以轻松打造一支 **7×24 小时在线的自动化研发团队**。从需求拆解、任务分配到执行汇报，Leader 与 Teammate 之间的高效协同，让复杂项目的推进变得井然有序。
本文以三大实战场景为例——**字幕软件开发**、**AtomGit Issue/PR 智能化处理**、**飞书文档同步与日报生成**——展示了 AgentSwarm 在真实研发流程中的落地价值。借助飞书机器人作为交互入口，结合 ag-cli 和 lark-cli 等工具链，AI Agent 不仅能写代码，还能管理代码仓库、操作协作平台，真正融入团队的工作流。
未来，随着 AgentSwarm 在更多业务场景中的深入应用，人机协作的研发模式将成为提升团队效率的关键杠杆。
