# SkillFab：Agent 技能的生产线，不是 GitHub 的复刻

> 论文：SkillFab: An Agent-Native Skill Production Platform — System Design Technical Report
> 作者：Anjie Xu, Yifeng Cai, Yi Li, Zixing Wang, Zhiyu Zhang, Jingfan Chen, Ruohan Xu, Leye Wang（北大、华为、西工大、中关村学院）
> 发布时间：2026 年 7 月
> 部署：skillfab.ai | 评测：skilltester.ai

---

## 一、Agent 学会了新技能，然后呢？

现代 Agent 可以自己探索、反思、调用工具，经常在一次任务里摸索出一套很有效的操作流程。但任务结束之后，这套流程就丢了。下一个 Agent 遇到同样的需求，还是从零开始。没有可复用的产物，没有版本历史，没有人审过。

SkillFab 做的事：**把 Agent 技能的发现、实现、审查、发布串成一条完整的生产线。** 它像一个专门给 Agent 技能用的 GitHub，但起点不一样。GitHub 的 issue 通常挂在已有仓库下面，SkillFab 的 issue 可以在任何代码存在之前就提出："我需要一个能检测操作系统的技能，但还没有人写。"

---

## 二、核心理念：需求先行 + 复用先行

SkillFab 的工作流由两个互补原则驱动。

**需求先行（demand-first）。** 一个缺失的能力可以在任何仓库或实现存在之前就被记录为 issue。传统流程是先有仓库、才有 issue。SkillFab 反过来，issue 可能只是一个能力缺口描述："Agent 在执行平台相关命令时不知道该用 apt 还是 yum。"

**复用先行（reuse-first）。** Agent 接到任务后先查技能注册表。如果已经有合适的技能，直接用，不开启新开发。只有查不到或复用失败时，才进入需求先行流程：记录 issue、分配仓库、开发提交。

两个原则互补：复用避免重复造轮子，需求先行确保"想造但还没造的轮子"不被遗忘。

### 核心对象

SkillFab 借鉴了 GitHub 的协作词汇，但把工作单元从"代码"换成了"技能"：

- **Issue**：能力缺口。可以没有关联仓库，没有已有技能。
- **Repo**：实现者的 Git 工作区。issue 被认领后分配。
- **Submission**：一次实现尝试，记录分支、提交和审查状态。一次 issue 可以有多次 submission——上一次失败了，下一次继续。
- **Skill**：审查通过并发布的结果，以 SKILL.md 为锚点、带版本号。
- **Commit Snapshot**：推送后由原生 Rust 模块摄入的实际文件快照。审查时看的是这个快照，不是聊天记录。

---

## 三、架构：三层分离

SkillFab 把系统拆成三个平面：

**控制平面。** Hono Node 应用，通过三种界面暴露同一套对象：Web 页面给人看、REST API 给脚本调、MCP 端点给 Agent 调。三种界面操作的是同一份状态，Agent 和人类看到的是同一个 issue、同一份审查记录。

**证据平面。** 独立的 Git 服务器处理推送，原生 Rust 模块做 commit range 摄入，产出的文件快照成为审查依据。每一次发布决定都可以追溯到一个具体的 commit 和文件版本。Commit 本身不算充分证据——只有在被链接到一个 submission 和摄入快照之后，才成为可引用的审查依据。

**共享状态。** SQLite 存储工作流状态和元数据，裸 Git 仓库存储提交历史，workflow-events 供 Agent 中断恢复。

三层之间最重要的边界是**工作流意图和软件证据分离**。MCP 调用创建 issue、提交审查、发布技能；Git 推送提供实际的文件内容。两个通路解耦：MCP 调用失败不应该污染仓库证据，Git 推送失败也不应该让工作流状态静默推进。推送被接受后，原生摄入路径读取 commit range、记录文件快照，工作流层只有在摄入可见后才推进状态。这个边界给了审查一个具体的不变量：每一次发布决定都可以追溯到特定 commit 的特定文件。

---

## 四、Agent 怎么用：MCP 作为一等公民

SkillFab 的 MCP 端点是给 Agent 用的原生执行界面。Agent 通过结构化工具调用完成整个技能生产流程，不需要知道底层有多少数据库表。

一个典型的 Agent 操作序列：`create_issue` 记录能力缺口 → `list_available_issues` 发现待实现需求 → `create_repo` 创建仓库和 submission → Git push 提交 SKILL.md 和辅助文件 → `verify_push` 确认摄入可见 → `request_review` 请求审查 → 维护者审查 → `certify_skill` 发布到注册表。

Agent 不直接操作数据库。它问的是"有没有可做的工作""这个推送被摄入了吗""审查状态是什么"。Agent 接口和人类协作模型保持一致，不因底层实现变化而变脆。

**中断恢复是 MCP 设计的核心考量。** `workflow-state` 返回当前阶段、下一步动作、最新 submission、审查状态、发布状态、重复标记。`workflow-events` 返回最近的 issue 事件历史。Agent 中断后不需要从原始日志重建状态，读这两条就够了。这意味着一个长周期的技能开发过程可以被多个 Agent 接力完成，每个接手时都能精确知道自己从哪里继续。

---

## 五、Submission 有自己的状态机

SkillFab 区分了 Issue 状态和 Submission 状态。Issue 状态记录平台层面的需求，包括还没有任何实现的缺口。Submission 状态记录一次实现尝试的生命周期：**Open → Submitted → Needs Work（循环）→ Approved → Published**，或者 Rejected / Abandoned 终止。

**一次 issue 可以活过多次失败的 submission。** 实现尝试被拒绝或放弃了，issue 还在，下一个 Agent 开新的 submission 继续做。一个已发布的技能可以关闭当前 issue，但未来出现复用失败、环境变化或工具行为变更时，可以开新的 issue。

失败路径同样由状态机管理。推送缺失或摄入不可见，`verify_push` 阻止 submission 进入审查。维护者要求修改，submission 回到 Needs Work，开发者重新推送、重新验证、重新请求审查。每一次状态转换都有命名状态、有记录事件、有下一步动作。这个设计让中断、修改和替换作为平台状态可见，Agent 不需要从聊天历史里重建上下文。

---

## 六、三个案例

论文用三个案例覆盖了平台的主路径。

**OS-detect 技能。** 走完一条完整的端到端路径：创建 issue → 创建仓库 → Git 推送 SKILL.md 和辅助脚本 → 原生摄入 2 个文件 → 审查通过 → 认证发布 → 注册表读取。整个过程通过 MCP 工具和 Git 推送完成，不走数据库直写。

**Docker 操作知识变技能。** 把 Docker 的沙箱管理、多服务编排、热部署、Dockerfile 模板等操作经验打包成结构化技能包，走正常 issue → submission → review → publish 流程。这个案例验证了已有的工程实践可以转化为可复用的 Agent 知识，不需要引入新平台对象。

**外部优化进入治理。** 用 SkillOpt 优化了一个已有技能（alfworld-agent v0.1.0 → v0.2.0），优化后以普通 submission 提交到 SkillFab。外部优化器不需要成为平台内置组件。维护者审查 diff 和优化证据后批准，新版本发布。这个案例验证了外部优化的技能改进可以无缝进入 SkillFab 的审查和版本管理体系。

---

## 七、安全和运维

几个安全设计值得留意。审查 Agent 不需要执行未信任的提交包，审查者只看 SKILL.md、diff、文件快照和审查历史。长生命周期 API token 由用户持有且可撤销，Git 写入使用短生命周期、仓库范围的推送 URL。原生 range 摄入和工作流事件让重复或失败的 Agent 动作可见，不藏在外部 transcript 里。

**SkillFab 和 SkillTester 是分开的两个项目。** SkillFab 管技能的生产和治理，SkillTester 管技能的效用和安全性评测。未来认证流程可以把 SkillTester 的评测结果作为外部证据引入，不把评测逻辑塞进生产平台。

---

## 八、总结

SkillFab 不是方法创新，是基础设施。它解决了一个 Agent 技能生态里很实际的痛点：**怎么让一个 Agent 学会的东西，变成所有 Agent 都能用的东西。**

当前的 Agent 技能大多是个人项目。一个人写一个 SKILL.md，手动传到 GitHub 或某个市场。没有审查，没有版本管理，没有能力缺口的公开记录。技能库在涨，但生产过程是手工的、孤立的、不可追溯的。

SkillFab 给的答案：把软件工程的协作基础设施——issue 追踪、Git 证据、审查门禁、注册表发布——适配到 Agent 技能的粒度上。一个技能就是一个可以提 issue、可以推代码、可以被审查、可以发布版本、可以被搜索复用的协作产物。

> 论文：https://arxiv.org/abs/2607.03780v1
> 部署：https://skillfab.ai | 评测：https://skilltester.ai
