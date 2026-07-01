# 让 Agent 优化自己：从读懂 Harness 到自进化实践

> 上一篇 Skill 会自己长大了。但 Skill 只是 Agent 能力的一半。另一半叫 Harness——这篇文章把它拆开，然后用 Auto Harness 让 Agent 学会自己改它。

*JiuwenSwarm 实践系列 · #03 · ~4000 words*

---

## 这篇做什么

上一篇 Skill 自演进——技能文档在用的时候自己攒经验。但 Skill 只是 Agent 露出水面的那一角。

Agent 跑起来的完整公式：**Agent = Model + Harness。** Model 管"聪明"——推理、理解、生成。Harness 管"能干"——system prompt、工具定义、Rail、Skill 文档、子 Agent 编排、长期记忆。没 Harness，模型只是个会说话的概率模型。套上 Harness，它才是能行动的 Agent。

![Agent = Model + Harness](images/03_agent_formula.png)

问题在哪？Model 的后训练（RLHF → DPO → GRPO）已经是成熟工程了。**Harness 的后训练，至今几乎全靠人。** 调 prompt、改 tool desc、写 skill 文档、配 rail 钩子。每次优化都是人在试、在猜、在调。

Auto Harness 就是 openJiuwen 的工程答案：模型权重一条不动，让 Harness 在实战里自己学、自己改。这篇先拆 Harness 的结构，再带你跑一次 Auto Harness。

---

## Harness 拆开来看

openJiuwen 把 Harness 分了两层。这不是学术概念——是 Auto Harness 能做双层优化的工程基础。

**Meta Harness（基座）。** 通用编排引擎。控制流、路由、权限、上下文管理、生命周期、错误恢复。所有 Agent 共享。它是无状态的：不包含任何领域特化内容。场景化能力全通过 Expert Harness 扩展注入。

基座层不管"擅长什么领域"。它只管一件事：Agent 怎么跑起来。

**Expert Harness（扩展）。** 领域特化的配置包，按需加载。里面有什么？跟场景相关的提示词、技能、工具配置、MCP 插件、Rail 扩展、上下文资产、完成验证条件，还有 Agent 的角色画像。

一个真实目录：

```
coding_expert/
  harness_config.yaml
  config.json
  prompt_sections/
  tools/
  skills/
  rails/
  soul.md
  identity.md
  stop_eval_conditions/
```

一个 Agent 实例就是 Meta Harness + 0~N 个 Expert Harness。同一个基座，挂不同扩展组合，变成不同领域的专家——编程的、设计的、营销的、法律审查的。Expert Harness 支持热加载，多个可叠加。

两层安全边界也不一样。Meta Harness 改一次影响所有 Agent，必须走源码 → PR → 人工审查 → 合入主干。Expert Harness 只影响特定场景，生成 Package → 热加载 → 即插即用。迭代快得多。

![双层 Harness 架构：Meta（基座）+ Expert（扩展）](images/03_harness_two_layer.png)

---

## Auto Harness 怎么跑：评测驱动的闭环

核心流程一句话：**评测 → 规划 → 实施 → 再评测。循环到达到预期为止。** 不是学术复现，是真实可跑的工程交付。

![Auto Harness 核心闭环：评测 → 规划 → 实施 → 再评测](images/03_auto_harness_loop.png)

**评测集生成。** 收自然语言优化目标，LLM 分析任务维度，自动生成覆盖不同场景的评测数据。自己已经有评测集也可以直接加载。

**评测与评估。** 用当前 Harness 配置建 Agent，在评测集上跑。收集成功率、耗时、质量评分。达标了就停。

**设计优化计划。** 没达标，从结果里找根因。检索经验库里之前类似问题的优化策略和效果。约束本轮可改范围，输出带依赖关系的优化动作清单——有些必须先做，比如先修 Rail 再修依赖这个 Rail 的 Skill。

**实施。** 每个动作在独立 git worktree 里隔离执行。依赖关系按拓扑排序分波并行——没依赖的同时跑，有依赖的等前置完了再跑。全跑完，两阶段 fix loop：CI 修语法和格式，评审验逻辑正确性。

**经验沉淀。** 优化生效后自动写经验库：问题类型、归因、修复策略、效果度量。越优化越准。

全流程有兜底：预算超了自动停、限制可改路径、不可变文件要额外审批、失败自动回滚。

---

## 动手：Expert Harness 优化

Expert Harness 优化是最常用的。不碰基座代码，迭代快、风险小。

在 TUI 里：

```
/auto-harness run --pipeline optimize_expert_harness \
  提升一下你自己的办公能力，要求能够：
  1. 擅长生成图文并茂、排版美观、有逻辑性的 PPT
  2. 熟练进行 Word 操作
  3. 熟练处理财务相关的 Excel 表格
  4. 对所有生成文件的文件名做敏感信息检查，这是硬性约束，写入文件前做强制检查
```

编排器自动跑完整条：

- **评测**：当前 Expert Harness 在办公场景下跑。PPT 排版质量低、Excel 公式错、敏感信息检查完全缺失。分数量化了"现在有多差"。
- **规划**：输出优化动作清单——新增 PPT/Excel/Word 工具和技能、新增敏感信息检查 Rail。约束类（文件安全校验）先跑，增强类（办公技能）并行。
- **实施**：每个动作独立 worktree。约束 Rail 先写完，办公工具和 Skill 并行生成。fix loop：CI 验证 Rail 语法 + 评审确认工具声明和 Skill 行为一致。
- **再评测**：重新跑——PPT 排版上来了、Excel 公式准了、敏感信息检查生效了。达标，停。

产物是一个新版 Expert Harness 扩展目录。Web 端直接激活——挂上或卸下，你自己选。

---

## 更底层：Meta Harness 优化

流程跟 Expert 差不多，但多了三件事。

**定时任务（跑之前）。** 按周期自动触发。每 48 小时一次，不用人守着。

**业界调研（跑之前）。** 竞品差距分析。自动搜业界较好实现，输出 gap 列表按 impact × feasibility 排。比如调研 Claude Code 的上下文压缩，发现自己的触发阈值偏高、归档粒度偏粗——差距转成优化动作。

**提 PR（跑完之后）。** 基座改动影响所有场景，代码变更提 PR 给人工审核，不 auto-merge。

一个真实例子：

```
/auto-harness run --pipeline optimize_meta_harness \
  调研当前和 Claude Code 在上下文压缩特性上的差异和不足，吸收提升自己能力
```

编排器自动调研 → gap 列表 → 评测当前 Meta Harness → 代码级优化动作（新增压缩 Rail + 改触发逻辑，两者有依赖）→ worktree 隔离实施 + CI 门控 → 再评测 → 提 PR。

---

## 这篇完了

Agent = Model + Harness。Model 的后训练很成熟。Harness 的后训练至今靠人。Auto Harness 把"人调配置"变成"Agent 跑评测、找差距、自动改配置、再验证"的闭环。

Expert Harness 优化你最常用——几句话描述需求，编排器自动跑完，产出一个即插即用的领域能力包。Meta Harness 优化加了定时、竞品调研、PR 审核——Agent 能自己吸收业界最佳实践来改基座。

下一篇最后一篇：每个 Agent 都有自己的 Harness 了，怎么让一群 Agent 组队协同？那是 AgentTeam。

---

*下一篇见。*

### 自评

| 维度 | 分 | 说明 |
|------|-----|------|
| 直接性 | 9 | 开篇直接给公式，不铺垫；每个环节只用一句话描述 |
| 节奏 | 9 | 双层 Harness 解释用长短交替："基座层不管领域。它只管一件事" |
| 信任度 | 9 | 砍掉"值得注意的是""值得强调的是"等引导语；"很成熟"直接用，不加"已经" |
| 真实性 | 9 | "跟 Expert 差不多，但多了三件事""你自己选"——去掉解释性包装 |
| 精炼度 | 8 | 八类组件排比仍略长，但这是技术文档的必要精确性 |
| **总分** | **44** | |
