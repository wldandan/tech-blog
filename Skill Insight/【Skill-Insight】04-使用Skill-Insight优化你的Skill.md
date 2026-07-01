# 使用Skill-insight优化你的Skill

在前面的3篇，我们把生成Skill、评测和管理Skill，讲清楚了。三篇的路是一根线：

```
生成Skill → 扩展/归纳应对不同输入 → 观测与管理保证质量
```

但还缺最后一步 —— **观测到问题之后呢？**

数据显示准确率低、流程有偏离、关键步骤被跳过。看完了，然后呢？动手修。但**怎么修才不是瞎修？怎么修完能确认真的好起来了**？Skill-insight提供了**四种优化模式**（静态评估、动态修复、用户反馈、混合优化），覆盖从代码质量体检、运行时故障自愈到定制化打磨的全场景。

本文作为优化能力的入门篇，将以最直观易上手的**Feedback（用户反馈）模式**为例，带你走完一次完整的优化流程——通过几轮自然语言反馈，将一个通用的Skill逐步定制为符合团队需求的版本。掌握了这个基础流程后，你可以进一步探索Static、Dynamic、Hybrid等更多优化模式，应对更复杂的场景。

# 一、为什么需要优化Skill

## 1.1 Skill优化的价值

一个高质量的Skill应当具备三个特征：逻辑严谨（覆盖边界条件）、环境适应（兼容不同使用场景）、贴合业务（符合团队的实际规范和偏好）。然而在实际使用中，Skill从"能跑"到"好用"之间往往存在明显的差距。

以第一篇中生成的`git-commit-guide`为例——它完整覆盖了Conventional Commits的标准规范，但当不同团队拿去使用时，很快就会暴露出"水土不服"的问题：有的团队要求commit message用中文写description；有的团队有自己的模块命名（如`biz-order`、`infra-mq`），通用的Scope推荐完全不适用；有的团队有额外的规范约束（如强制要求关联Issue号）。

这些问题的本质不是Skill有缺陷，而是它还停留在"通用模板"阶段，没有被定制化为符合特定团队需求的版本。而在运维场景中，Skill还可能因为操作系统升级、依赖库变更等环境漂移而直接失效。**优化的核心价值，是让Skill在全生命周期中持续保持高质量**——发布前通过质量评估消除隐患，运行中通过故障轨迹自动修复，使用中通过用户反馈持续打磨。

## 1.2 常见痛点

- **痛点一：生成即终点，缺少迭代机制**。 无论通过什么方式生成Skill，"生成完"往往就意味着"结束了"。当用户发现Skill的某些细节不符合预期时，通常只能手动打开`SKILL.md`逐行修改——既不知道哪些地方该改，也难以保证改完不破坏其他部分。缺少一套标准化的迭代机制，优化全凭个人经验。
- **痛点二：用户反馈无法闭环**。用户在实际使用中积累的反馈（"Scope推荐不准"、"示例不够"、"缺少中文支持"）是最有价值的优化信号，但传统流程中这些反馈只能以口头沟通或工单的形式传递给Skill维护者，再由人工判断如何修改。反馈到落地的链路太长、损耗太大。
- **痛点三：修改风险不可控**。手动修改Skill时，改一处可能影响其他部分——比如修改了type的定义表，但忘了同步更新`references/commit-types.md`中的对应内容，导致Agent执行时引用的参考资料与核心指令不一致。缺少版本管理和Diff审核机制，修改的风险难以控制。

![alt text](image.png)

## 1.3 Skill-insight的优化定位

针对以上痛点，Skill-insight提供了一套**标准化的Skill优化体系**。它的核心思路是：不做全量重写，只做定点修补；不靠人工排查，靠数据驱动定位；不搞一锤子买卖，靠版本快照支撑可追溯的迭代。通过静态质量评估、动态故障修复、用户反馈迭代等多种优化模式的组合，让Skill在全生命周期中持续保持高质量。

# 二、Skill-insight优化能力总览

## 2.1 四大优化模式

Skill-insight将底层优化能力拆解为四个原子化模式，每种模式对应不同的场景和输入：

- **Static（静态优化）**：不依赖任何运行时数据，仅基于Skill自身代码与配置进行冷启动式评估与修复。系统从六个维度打分（职责明确性、结构规范性、指令适配性、内容一致性、风险可控性、脚本及文档质量），不达标的维度会自动生成诊断报告并完成修补。适用于Skill首次发布前的质量门禁。
- **Dynamic（动态优化）**：基于Skill实际运行时产生的失败轨迹（崩溃日志、Trace信息）进行针对性修补。引擎仅锁定失效代码块进行定点修复，而非全量重写，避免"幻觉"导致正常部分被误改。适用于Skill在生产环境中因依赖变更等问题运行失败后的快速修复。
- **Feedback（反馈优化）——本文入门示例选用此模式**：基于用户的明确反馈意见进行针对性迭代。你不需要等到Skill报错才能优化，只要觉得某个细节不满意，直接用自然语言告诉Agent，系统就会精准定位到对应的代码段落进行修改，同时保持其他部分不变。每轮修改生成小版本号（如v1.1），你确认后才晋升为稳定主版本。选择它作为入门演示，是因为它上手门槛最低、不依赖Trace数据，适合第一次体验优化流程。
- **Hybrid（混合优化）**：先执行静态优化，再执行动态优化，以static → dynamic的流水线覆盖"代码缺陷 + 环境适配"两类问题。适用于需要彻底优化的场景。

![alt text](四种Skill优化方式手绘图.png)

## 2.2 优化流程核心环节

无论选择哪种模式，完整的优化流程都遵循四个核心环节：

1. **环境准备**：安装Skill-insight优化器组件，确认Agent终端环境可用。
2. **执行优化**：选择合适的优化模式（或直接用自然语言描述反馈），由引擎自动完成分析与修补。
3. **Diff评审**：系统以可视化差异对比（Diff Viewer）展示优化前后的变更，用户通过Accept / Revise / Revert快捷指令完成确认，实现Human-in-the-loop的质量把控。
4. **版本管理**：每次优化的候选版本都保存在`snapshots/`目录中，配套`meta.json`记录优化来源、模式与触发原因。主版本与小版本分层管理，支持随时回滚与追溯。

# 三、准备工作

在开始优化之前，请确认以下环境已就绪：

- **Agent编码终端**：你需要一个支持Agent能力的编码终端，例如OpenCode或Claude Code。确保终端可以正常执行`npx`命令，并且网络可达Skill-insight的仓库地址。
- **Node.js运行时**：安装依赖`npx`，请确保本地已安装Node.js（建议v20及以上）和npm：

   ```bash
   node -v   # 应输出v20.x或更高
   npm -v    # 应输出10.x.x或更高
   ```

- **待优化的Skill**：本文以入门文章中生成的`git-commit-guide`Skill为例。如果你还没有，可以在Agent终端中执行`帮我做一个Git Commit规范的Skill`快速生成一个。

生成后的目录结构如下：

```
git-commit-guide/
├── SKILL.md                          # 核心Skill文件
└── references/
    ├── commit-types.md               # 提交类型详细定义
    └── commit-examples.md            # 各场景commit message示例集
```

# 四、实操步骤：以Feedback模式为例的入门演示

Skill-insight支持Static、Dynamic、Feedback、Hybrid四种优化模式。本文选择**Feedback（用户反馈）模式**作为入门演示，原因很简单：它不依赖运行失败的Trace日志，也不需要理解静态评估的六维评分体系——你只需要用自然语言告诉Agent"哪里不满意"，系统就能自动完成定点修补。这是上手门槛最低、效果最直观的优化方式，非常适合作为第一次优化体验。

下面以一个真实场景来演示：你的团队要求commit message的description部分使用中文，并且Scope必须使用项目自己的模块命名。我们通过Feedback模式，将通用的`git-commit-guide`定制为符合团队规范的版本。

> 💡 **其他模式适用场景预览：**如果你的Skill刚写好还没跑过，推荐先用**Static**做一次质量体检；如果Skill在生产环境中报错了，用**Dynamic**基于失败轨迹自动修复；如果你想一次做彻底优化，用**Hybrid**让系统按static → dynamic自动编排。这些模式的详细用法将在后续进阶篇中展开。

## Step 1：安装优化器组件

如果你在入门篇中已经安装过Skill-insight，可以跳过这一步。否则，在Agent终端中执行：

```bash
npx skills add https://gitcode.com/openeuler/witty-skill-insight.git
```

安装完成后，优化器将作为Agent可调用的工具注册到当前环境中。

## Step 2：数据准备

确认`git-commit-guide`Skill已在你的Skill目录中，并回顾一下当前`SKILL.md`的核心内容。入门篇中生成的版本包含以下5步工作流：

- Step 1：理解格式结构——`<type>(<scope>): <description>`
- Step 2：选择提交类型——feat / fix / docs 等十种类型
- Step 3：填写Scope——通用规则，无项目定制
- Step 4：编写Description——英文祈使句，50字符以内
- Step 5：添加Body和Footer——Breaking Change标记方式

当前版本的问题很明确：Description要求用英文祈使句写，但团队要求用中文；Scope的推荐值是通用的（如`auth`、`api`、`ui`），没有覆盖项目实际的模块命名。

## Step 3：开始优化——用自然语言提交反馈

在Agent终端中直接告诉它你的需求：

**第一轮反馈——中文Description支持：**

> "优化一下git-commit-guide Skill：我们团队要求description部分用中文编写，不要求英文祈使句。请修改Step 4的规则说明和示例，同时更新references/commit-examples.md中的所有示例。"

Agent 会调用 Feedback 模式的优化引擎，精准定位到`SKILL.md`中Step4的内容块和`references/commit-examples.md`文件，进行定点修补。

优化完成后，系统进入Diff Review环节。你会看到类似如下的变更：

```diff
  ## Step 4：编写 Description

- **规则：**
- - 使用英文祈使句（如 add, fix, update）
- - 首字母大写
- - 50 字符以内
- - 结尾不加句号
-
- **正确示例：**
- feat(auth): Add OAuth2 login support
- fix(api): Resolve null pointer in user query
-
- **错误示例：**
- feat(auth): added oauth2 login support  ← 时态错误
- fix(api): Fix.  ← 信息不足

+ **规则：**
+ - 使用中文描述，简明扼要
+ - 控制在 50 个字符以内（约 25 个汉字）
+ - 结尾不加句号
+ - 描述"做了什么"，而非"做了什么改动"
+
+ **正确示例：**
+ feat(auth): 新增 OAuth2 登录支持
+ fix(api): 修复用户查询空指针异常
+
+ **错误示例：**
+ feat(auth): 我加了一个oauth2的登录功能  ← 口语化，过长
+ fix(api): 修复了bug  ← 信息不足，未说明具体问题
```

此时你可以选择：

- **Accept**：满意，落盘为v1.1。
- **Revise**：继续提反馈，如"错误示例再加一个中英文混用的情况"。
- **Revert**：不满意，回滚到v1。

假设你选择**Accept**，系统将此版本保存为v1.1。

**第二轮反馈——定制化Scope：**

> "继续优化git-commit-guide：我们项目的模块包括biz-order（订单业务）、biz-pay（支付业务）、infra-mq（消息队列）、infra-cache（缓存）、common-util（公共工具）。请在Step 3中添加这些作为推荐Scope，并在references/commit-types.md的示例中使用这些Scope。"

引擎再次定位到Step 3和参考文件中的相关段落，生成v1.2：

```diff
  ## Step 3：填写Scope

  Scope 用于标识本次提交影响的模块或功能区域。

- **常见 Scope 值：**
- auth, api, ui, db, config, docs, test

+ **项目推荐 Scope 值：**
+
+ | Scope | 含义 |
+ |-------|------|
+ | biz-order | 订单业务模块 |
+ | biz-pay | 支付业务模块 |
+ | infra-mq | 消息队列基础设施 |
+ | infra-cache | 缓存基础设施 |
+ | common-util | 公共工具库 |
+
+ **填写原则：**
+ - 优先使用上述推荐值
+ - 如涉及新模块，与团队对齐后再添加
+ - 跨多个模块的提交可省略 Scope
```

选择Accept，保存为v1.2。

**第三轮反馈——强制关联Issue：**

> "再加一个规则：所有feat和fix类型的commit必须在Footer中关联Issue号，格式为`Refs: #123`。在Step 5中补充这个要求。"

引擎定位到Step 5，补充Footer规则，生成v1.3。Accept后，这个版本晋升为稳定的v2主版本。

## Step 4：对优化后的Skill进行调试与验证

三轮反馈优化完成后，让我们验证一下效果。在Agent终端中测试：

> "我刚修复了支付模块的一个金额计算精度问题，帮我写一个commit message。"

Agent使用优化后的Skill生成：

```
fix(biz-pay): 修复金额计算精度丢失问题

支付金额在经过多次折扣计算后，由于浮点数精度问题导致
最终金额与预期不符。改用BigDecimal进行定点运算。

Refs: #456
```

对比优化前（通用版本）可能生成的结果：

```
fix(api): Fix amount calculation precision issue
```

可以看到：Description已经是中文、Scope使用了项目定制的`biz-pay`、Footer自动关联了Issue号——完全符合团队规范。

## Step 5：执行优化后的Skill

确认Accept后，优化后的Skill已替换原版本。后续团队成员在Agent终端中使用时，无需做任何额外配置——他们只需像往常一样：

> "帮我写一个commit message，我在infra-mq模块新增了延迟消息队列的支持。"

Agent会自动基于定制化后的Skill生成符合团队规范的结果：

```
feat(infra-mq): 新增延迟消息队列支持

基于 RocketMQ 的延迟级别机制，实现消息延迟投递功能。
支持秒级和分钟级两种延迟粒度。

Refs: #789
```

## Step 6：进行评测与对比

优化完成后，可以通过Skill-insight的评测能力对优化效果进行量化验证。

**静态评分对比。**在优化前后分别查看六维质量评分：

| 评估维度 | 优化前（v1） | 优化后（v2） | 变化 |
|---------|-------------|-------------|------|
| 职责明确性 | 8/10 | 9/10 | +1 |
| 结构规范性 | 8/10 | 9/10 | +1 |
| 指令适配性 | 5/10 | 9/10 | **+4** |
| 内容一致性 | 7/10 | 9/10 | +2 |
| 风险可控性 | 7/10 | 8/10 | +1 |
| 脚本及文档质量 | 6/10 | 9/10 | **+3** |

提升最显著的是**指令适配性**（+4）和**脚本及文档质量**（+3）。这正是Feedback模式的价值所在——它让Skill从"符合通用规范"进化为"贴合团队实际需求"。

**版本追溯。**通过`snapshots/`目录可以完整追溯优化历史：

```
snapshots/
├── v1/                # 原始生成版本
│   └── meta.json
├── v1.1/              # 第一轮反馈：中文Description
│   └── meta.json
├── v1.2/              # 第二轮反馈：定制化Scope
│   └── meta.json
├── v1.3 → v2/         # 第三轮反馈：强制关联Issue → 晋升主版本
│   └── meta.json
```

每个版本的`meta.json`记录了完整的元数据：

```json
{
  "version": "v1.1",
  "source": "user",
  "mode": "feedback",
  "trigger": "团队要求 description 使用中文编写",
  "parent_version": "v1",
  "timestamp": "2025-07-10T14:22:00Z"
}
```

如果后续发现某轮修改引入了问题，可以随时Revert到任意历史版本。

---

# 五、案例演示：从通用到定制的完整蜕变

让我们完整回顾`git-commit-guide` Skill从生成到定制化的全过程，以及优化前后的效果对比。

## 优化前：通用版 SKILL.md（节选）

```yaml
---
name: git-commit-guide
description: >
  指导编写符合 Conventional Commits 规范的 Git commit message。
---
```

```markdown
## Step 3：填写 Scope

常见 Scope 值：auth, api, ui, db, config, docs, test

## Step 4：编写 Description

规则：
- 使用英文祈使句（如 add, fix, update）
- 首字母大写
- 50 字符以内

正确示例：
feat(auth): Add OAuth2 login support

## Step 5：添加 Body 和 Footer

Breaking Change 标记方式...（无强制关联 Issue 的要求）
```

## 优化后：定制版SKILL.md（节选）

```yaml
---
name: git-commit-guide
description: >
  指导编写符合团队定制化 Conventional Commits 规范的 Git commit message。
  支持中文 description，使用项目专属 Scope 命名。
---
```

```markdown
## Step 3：填写 Scope

项目推荐 Scope 值：

| Scope | 含义 |
|-------|------|
| biz-order | 订单业务模块 |
| biz-pay | 支付业务模块 |
| infra-mq | 消息队列基础设施 |
| infra-cache | 缓存基础设施 |
| common-util | 公共工具库 |

填写原则：优先使用推荐值，跨模块提交可省略。

## Step 4：编写 Description

规则：
- 使用中文描述，简明扼要
- 控制在 50 个字符以内（约 25 个汉字）
- 描述"做了什么"，而非"做了什么改动"

正确示例：
feat(biz-order): 新增订单批量导出功能
fix(infra-cache): 修复缓存穿透防护失效问题

错误示例：
feat(biz-order): 我加了个导出功能  ← 口语化
fix: 修复了bug  ← 信息不足，缺少 Scope

## Step 5：添加 Body 和 Footer

所有 feat 和 fix 类型必须在 Footer 中关联 Issue：
格式：Refs: #<Issue号>

完整示例：
feat(infra-mq): 新增延迟消息队列支持

基于 RocketMQ 的延迟级别机制，实现消息延迟投递功能。
支持秒级和分钟级两种延迟粒度。

Refs: #789
```

### 效果对比

| 对比项 | 优化前（通用版） | 优化后（定制版） |
|-------|----------------|----------------|
| Description 语言 | 英文祈使句 | 中文描述 |
| Scope 推荐值 | auth, api, ui 等通用值 | biz-order, infra-mq 等项目专属值 |
| Issue 关联 | 无强制要求 | feat/fix 类型强制关联 |
| 示例贴合度 | 通用示例 | 使用项目真实模块名的示例 |
| 新人上手体验 | 需要自行理解规范再映射到项目 | 直接参照示例即可编写 |
| 修改方式 | — | 三轮 Feedback 定点修补，未改动 Step 1、Step 2 |

关键亮点：三轮Feedback优化**仅修改了Step 3、Step 4、Step 5和对应的references文件**——Step 1（格式结构）和Step 2（类型速查表）作为通用知识保持不变。这正是"局部定点修补"的核心价值：只改需要改的，不碰不该碰的。

# 六、小结

本文以入门篇中生成的`git-commit-guide`Skill为例，选择上手门槛最低的**Feedback模式**，演示了一次完整的优化流程——通过三轮自然语言反馈，将一个通用的Git Commit规范Skill逐步定制为完全符合团队需求的版本。

需要强调的是，**Feedback只是Skill-insight四种优化模式中的一种**。它最适合"Skill能跑但不够贴合需求"的场景。在实际Skill运维中，你还会遇到更多情况：

- **Skill刚写好，不确定质量如何？** → 用**Static模式**做一次六维质量体检，在上线前发现潜在问题。
- **Skill在生产环境中报错了？** → 用**Dynamic模式**基于失败Trace自动定位和修复问题代码，无需人工排查日志。
- **想一次性做彻底优化？** → 用**Hybrid模式**让系统按static → dynamic自动编排，全面覆盖代码缺陷和环境适配问题。

这四种模式共享同一套核心机制——局部定点修补、Diff Review交互闭环、快照版本管理——只是输入来源和触发场景不同。掌握了本文的Feedback流程后，切换到其他模式几乎没有额外的学习成本。

后续我们还将推出Static和Dynamic模式的专题文章，届时会结合运维场景展开更深入的实操演示。

想第一时间获取后续内容？欢迎关注项目仓库并Star：

🔗 **项目地址**：<https://gitcode.com/openeuler/witty-skill-insight>

📦 **npm包**：[@witty-ai/skill-insight](https://www.npmjs.com/package/@witty-ai/skill-insight)

有问题？提Issue，社区一起搞定：[Issue入口](https://atomgit.com/openeuler/witty-skill-insight/issues)

---

# 七、常见问题

**Q：Feedback模式会不会把我原本正常的代码改坏？**

不会。Feedback模式的核心设计原则是"局部定点修补"。引擎只会锁定你反馈中提到的代码块进行修改，不会触碰其他部分。同时，每次优化都会生成候选版本供你Diff Review，只有明确Accept后才会落盘。如果不满意，随时Revert回滚。

**Q：一次反馈可以提多个修改点吗？**

可以，但建议每次反馈聚焦1-2个关联的修改点，以便Diff Review时更容易判断修改是否符合预期。如果修改点较多且彼此独立（如"改Scope"和"改Description语言"），分成多轮反馈效果更好——每轮生成独立小版本，也更方便后续回滚。

**Q：Feedback模式和Static / Dynamic模式有什么区别？该用哪个？**

三者的输入来源不同：Static基于Skill代码本身的质量评估，Dynamic基于运行失败的Trace日志，Feedback基于用户的主观反馈。简单来说——Skill没报错但你觉得不好用，用Feedback；Skill报错了，用Dynamic；Skill刚写好还没跑过，用Static做一次体检。不确定时，用Hybrid让系统自动编排。

**Q：版本号是怎么管理的？v1.1和v2的区别是什么？**

Static和Dynamic这类系统级优化会生成新的主版本（如v1 → v2）；Feedback模式下的每轮修改生成小版本（如v1.1、v1.2）。当用户Accept后，最新小版本晋升为稳定的主版本。所有版本保存在`snapshots/`目录，配套`meta.json`记录完整元数据。

**Q：已有的Skill可以直接用Skill-insight优化吗？**

可以。无论Skill是通过什么方式生成的（Skill Creator、手动编写、或从文档提取），都可以直接作为Skill-insight优化器的输入。只要Skill目录结构符合规范（包含`SKILL.md`），就能立即开始优化。

# 相关链接

1. 项目地址：<https://gitcode.com/openeuler/witty-skill-insight>
2. npm包：[@witty-ai/skill-insight](https://www.npmjs.com/package/@witty-ai/skill-insight)