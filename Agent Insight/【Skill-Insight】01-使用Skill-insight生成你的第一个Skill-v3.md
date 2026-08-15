# 入门篇 | 使用Skill-insight生成你的第一个Skill

> 你的Agent拥有了"大脑"，但它还缺一双"手"——Skill就是那双手。今天，我们用一句话，让Agent学会一项新技能。

***

如果你正在做Agent落地，一定遇到过这样的尴尬：模型很聪明，但真正让它"干活"的时候，要么找不到对的Skill，要么执行过程一团黑雾，出了问题只能反复试错。

**Skill-insight**就是为了终结这种困境而生的。

这篇文章不讲架构、不谈原理，只做一件事——**手把手带你用Skill-insight生成第一个可用的Skill**，从安装到运行，从生成到评测，全程实操，跟着走就能跑通。

***

# 一、什么是Skill-insight？

一句话概括：**Skill-insight 是一个让Agent的Skill从"能用"到"好用"的全生命周期管理平台。**

它解决的是Agent生态中Skill层面最核心的三个痛点：

**痛点一：Skill越多越乱。** 当你的Skill库膨胀到 40\~50 个以上时，召回率会从 95% 骤降至 30% 以下——Agent 找不到该用的 Skill，等于白写。Skill-insight在批量生成时自动去冗余、合并相似、抽取模式，帮你把Skill库保持在"精而准"的状态。

**痛点二：执行过程是黑盒。** 传统评测只看"任务完没完成"，但即便结果正确，中间可能跳过了关键步骤。Skill-insight自动生成执行流程图，与预期流程逐步比对，哪里偏离、哪里冗余、哪里跳过，一目了然。

**痛点三：优化全靠猜。** 没有执行数据支撑，改Skill只能反复试错。Skill-insight提供多维度评测指标（准确率、召回率、时延、Token消耗、成本等），并能自动归因定位瓶颈——是Skill写得不好，还是模型能力不足，分得清清楚楚。

简单来说，Skill-insight覆盖了Skill的**生成 → 评测 → 优化**全链路，让你不再"盲人摸象"。

***

# 二、准备工作

在开始之前，请确认以下环境已就绪：

| 准备项            | 说明                                             |
| :------------- | :--------------------------------------------- |
| **Node.js**    | 版本 ≥ 20，用于安装和运行Skill-insight                  |
| **Agent 框架**   | 推荐使用OpenCode（本文以此为例），也支持Claude Code、OpenClaw |
| **网络环境**       | 安装过程需要访问npm仓库和Git仓库                        |
| **一份案例文档（可选）** | 如果你想从文档生成Skill，准备好PDF/Markdown文件即可          |

> 💡 **没有文档也没关系。** Skill-insight支持纯自然语言生成——你只需要一句话描述需求，它就能帮你从零构建Skill。

***

# 三、实操步骤

## Step 1：安装Skill-insight平台

打开终端，一行命令搞定：

```bash
npx @witty-ai/skill-insight install
```

安装完成后，打开浏览器访问`http://localhost:3000`，你会看到Skill-insight的可视化看板。默认账号是`admin`，直接登录即可。

> 📌 **如果你需要源码部署**（适用于团队内网环境）：
>
> ```bash
> # 克隆代码
> git clone https://gitcode.com/openeuler/witty-skill-insight.git
> cd witty-skill-insight
> # 安装依赖
> npm install
> 生产模式启动（自动初始化环境）
> bash scripts/restart.sh
> ```

接下来，在OpenCode中注册Skill工具包：

```bash
npx skills add https://gitcode.com/openeuler/witty-skill-insight.git
```

至此，环境准备完毕。

***

## Step 2：生成你的第一个Skill

环境就绪后，打开OpenCode终端，我们来生成第一个Skill。

这里以**Git Commit规范指南** 为例——这是每个开发团队都会用到的场景：团队成员提交代码时commit message格式五花八门，你希望Agent能自动指导大家按Conventional Commits规范来写。

在OpenCode终端中输入一句话：

```
帮我做一个Git Commit规范的Skill
```

就这么简单。Skill-insight的**General 场景引擎**会被激活，它会自动完成以下工作：理解你的需求意图，调研Conventional Commits规范的最新标准，设计Skill的文件结构和内容分层，最终生成一套完整的、开箱即用的Skill制品。

整个过程中，Agent可能会在"确认"环节向你展示Skill骨架蓝图，供你Review并调整。如果你想跳过交互确认、一步到位，可以在指令中加上相关描述：

```
帮我做一个Git Commit规范的Skill，跳过交互，直接生成
```

> 💡 **Skill-insight不止支持自然语言生成。** 如果你手头有现成的规范文档（PDF/Markdown），也可以直接说"根据xxx.pdf生成一个Skill"，Doc-Pipeline引擎会严格基于文档内容进行提取，确保生成内容的可追溯性。

***

## Step 3：调试与验证

Skill生成完毕后，你不需要手动执行任何验证命令——**Skill-insight会自动触发质量检查流程**，在生成结束的瞬间就帮你把关。

这套内置的自动化验收机制分为两级：

**L1 结构检查** —— 自动确认Skill目录下的`SKILL.md`是否存在、YAML Frontmatter 格式是否合规、必要章节是否齐全。这一层确保你的 Skill "骨架"没问题。

**L2 内容检查** —— 自动对生成的内容执行语法校验和静态扫描，拦截代码幻觉和引用断链等问题。这一层确保你的 Skill "肌肉"能正常工作。

如果检查发现问题，生成流程会自动尝试修正并重新验证。最终交付到你手中的，是已经通过两级质量关卡的Skill制品。整个过程对你来说是透明的——你只需要关注生成结果即可。

***

## Step 4：使用技能 & 评测效果

验证通过后，把生成的Skill放到OpenCode的Skill目录下，然后执行一个真实任务来检验它：

```
我刚完成了一个功能开发，帮我写一个规范的Git commit message
```

任务执行完毕后，你可以在OpenCode终端页面右上角找到**Skill-insight卡片**，点击「查看详情」，即可跳转到平台查看本次执行的完整追溯信息。

在看板上，你可以看到：

**执行流程图** —— Agent实际走了哪些步骤，和Skill预期流程是否一致，偏离和冗余的节点会被高亮标识。

**多维指标面板** —— 包括准确率、Skill召回率、执行时延、Token 消耗、调用次数等关键数据，帮你量化这个Skill的实际表现。

### 深度评测（可选）

如果你想进一步使用准确率、Skill召回率、失败归因等高级评测能力：

1. 在平台主页面点击左上角**⚙️ Eval Config**，配置评测模型（支持DeepSeek/OpenAI/Anthropic/自定义模型）。
2. 点击右上角 **数据集管理**，录入测试用例：用户问题、预期答案、预期调用的Skill。
3. 运行评测，平台会自动生成对比报告。

如果评测结果不理想，Skill-insight还提供了一键优化能力：

```
/si-optimizer <待优化的Skill路径>
```

优化后的Skill会自动加载到OpenCode的Skill目录下。重启OpenCode后再跑一次同样的任务，在平台上就能直观对比优化前后的效果差异。

***

# 四、案例演示：Git Commit规范指南Skill

下面是一个完整的案例演示，展示从"一句话输入"到"得到可用Skill"的全过程。

## 输入指令

```
帮我做一个 Git Commit规范的Skill
```

## 生成产物结构

Skill-insight会在你的Skill目录下生成如下结构的标准制品：

```
git-commit-guide/
├── SKILL.md                          # Skill说明文档（含元数据、使用指南、核心指令）
└── references/
    ├── commit-types.md               # 各提交类型的详细定义与边界判断
    └── commit-examples.md            # 各类场景的完整commit message示例集
```

可以看到，Skill-insight自动将内容拆分为三层结构：`SKILL.md`是核心入口，包含完整的格式说明和步骤化指令；`references/`目录下则是配套的详细参考资料，按职责分文件组织，既方便Agent按需检索，也让开发者能快速定位和维护。

## SKILL.md 核心内容

生成的`SKILL.md`包含规范的YAML Frontmatter元数据和分步骤的核心指令：

```yaml
---
name: git-commit-guide
description: >
  指导编写符合Conventional Commits规范的Git commit message。当用户提到Git commit、
  提交代码、commit message、提交规范，或需要帮助编写commit message 时触发。
---
```

核心指令部分被设计为清晰的5步工作流：

**Step 1：理解格式结构** —— 标准格式`<type>(<scope>): <description>`的各组成部分（type、scope、description、body、footer）及其规则说明。

**Step 2：选择提交类型** —— 提供feat/fix/docs/style/refactor/perf/test/build/ci/chore十种类型的速查表，并给出选择原则（如"优化性能用perf而非refactor"）。

**Step 3：填写Scope** —— 何时使用、何时省略、填写规则及常见值。

**Step 4：编写Description** —— 50字符以内、使用祈使句、首字母大写、结尾无句号等关键规则，配有正反对比示例。

**Step 5：添加Body和Footer** —— Breaking Change的三种标记方式（`!` 后缀、Footer声明、或两者结合），以及完整的多段示例。

## 参考文件示例

`references/commit-types.md`中对每种类型都给出了详细的适用场景、SemVer影响和**边界判断**规则。例如`fix`类型：

```markdown
## fix — Bug 修复

修复代码中的错误或缺陷。

**适用场景：**
- 修复 crash、异常
- 修复逻辑错误
- 修复数据处理错误
- 修复 UI 显示错误

**边界判断：**
- 修复测试代码 → test（而非 fix）
- 修复 CI 配置 → ci（而非 fix）
- 修复构建脚本 → build（而非 fix）
```

`references/commit-examples.md` 则提供了十余种场景的完整示例，包含正确和错误写法的对比。例如常见错误：

```markdown
### ❌ 错误示例
feat: added dark mode toggle.
→ 问题：时态错误（应为 add），多余句号

### ✅ 正确示例
feat: add dark mode toggle
```

这些参考文件让 Agent 在实际使用 Skill 时，能够根据用户的具体场景精准匹配类型、给出符合规范的建议。

### 执行效果

当你在 OpenCode 中使用这个 Skill 后，在 Skill-insight 看板上可以看到：

**✅ 执行流程全追溯** —— Agent 是否按照 5 步工作流完整执行，有没有跳过"选择类型"或"填写 Scope"等关键环节，是否正确引用了 references 中的参考资料，都以流程图的形式清晰呈现。

**📊 量化评测数据** —— Skill 召回率（Agent 是否在"写 commit message"的场景下正确召回了这个 Skill）、执行时延、Token 消耗等指标一目了然。

**🔄 一键优化闭环** —— 如果评测发现 description 部分的格式说明不够清晰、Agent 生成的 commit message 经常超长，使用 `/si-optimizer` 即可自动定位并修补 Skill 中的薄弱环节。

***

# 五、Skill-insight vs Skill Creator：多文档场景下的关键差异

如果你之前用过Anthropic内置的**Skill Creator**，可能会好奇：既然已经有了Skill创建工具，为什么还需要Skill-insight？

对于单个Skill的创建场景，两者都能胜任。但当你面对的是**大量相似文档**时，差异就非常明显了。

## 一个真实的场景

假设你是一个运维团队的负责人，团队积累了20份MySQL慢查询相关的故障处理记录——不同时间、不同主机、不同业务，但根因都指向索引缺失或锁等待。你希望把这些经验沉淀为Agent可用的Skill。

**用Skill Creator的做法：**Skill Creator采用的是"逐个创建"的工作模式——你需要基于每份文档分别创建一个Skill，最终得到 20 个高度相似的Skill。当这些Skill进入 Skill库后，Agent面对一个新的MySQL慢查询问题时，需要从 20 个几乎相同的 Skill 中做选择。研究表明，**当Skill数量超过 40~50 个时，召回率会从 95% 骤降至 30% 以下**——冗余的Skill不仅没有帮助，反而严重拖累了 Agent 的表现。

**用Skill-insight的做法：**Skill-insight的Doc-Pipeline引擎支持**Merge Mode（特征模式归纳）**。它不是把每份文档各生成一个Skill，而是在底层提取这 20 份文档的共性故障模式（`FailurePattern`）——哪些排查步骤是共通的、哪些命令是必须执行的、哪些参数需要泛化——最终归纳生成**一个**覆盖整类故障的通用Skill。

## 对比总结

| 维度 | Skill Creator | Skill-insight |
|:-----|:-------------|:-------------|
| **多文档输入** | 逐个创建，N篇文档 → N个Skill | 支持Merge Mode，N篇相似文档 → 1个泛化Skill |
| **Skill召回率** | Skill膨胀后召回率下降 | 精简Skill数量，召回率显著提升 |
| **任务准确率** | 召回不准导致任务执行偏差 | 召回精准，任务准确率随之提高 |
| **Token消耗** | Agent需遍历大量相似Skill，Token开销高 | Skill库精简，检索和执行的Token消耗更低 |
| **执行可观测性** | 创建闭环（创建 → 测试 → eval），但无执行过程追溯 | 全链路可观测（生成 → 执行追溯 → 评测 → 归因 → 优化） |

简单来说，**Skill Creator更像是一个精致的"单品工坊"，适合逐个打磨高质量Skill；而Skill-insight更像是一条"智能产线"，尤其擅长从大量同类经验中提炼出精简、高召回的Skill集合。** 在运维场景中，后者的优势尤为突出——因为运维知识天然是"同一类问题、不同案例"的结构，正是Merge Mode发挥价值的理想场景。

***

# 六、小结

让我们回顾一下这篇文章走过的完整路径：

```
安装平台 → 注册工具包 → 一句话生成 Skill → 质量验证 → 执行任务 → 查看评测 → 优化迭代
```

这就是Skill-insight带来的核心价值：**把 Skill的生命周期从"写完就丢"变成了一个可观测、可量化、可持续改进的闭环。**

作为入门篇，我们只触及了Skill-insight能力的冰山一角。在后续的进阶文章中，我们还会深入探讨：

- **批量生成与去重**：如何从一整套文档库中高效提炼出精简的Skill集合
- **多维交叉对比**：如何从Skill、框架、模型、任务四个维度分析性能瓶颈
- **团队协作模式**：如何在团队内共建共享Skill库并持续优化

想第一时间获取后续内容？欢迎关注项目仓库并Star：

🔗 **项目地址**：<https://gitcode.com/openeuler/witty-skill-insight>

📦 **npm包**：[@witty-ai/skill-insight](https://www.npmjs.com/package/@witty-ai/skill-insight)

有问题？提Issue，社区一起搞定：[Issue 入口](https://atomgit.com/openeuler/witty-skill-insight/issues)

***

# 七、常见问题

**Q：安装时报npm权限错误怎么办？**

A：尝试使用`sudo npm i -g @witty-ai/skill-insight`，或者配置npm的全局安装路径到用户目录下（推荐 `npm config set prefix ~/.npm-global`），避免权限问题。

**Q：`localhost:3000`** **打不开看板？**

A：确认Skill-insight服务是否正常启动。可以检查终端输出是否有报错，也可以尝试`bash scripts/restart.sh`重启服务。如果端口被占用，检查是否有其他服务占用了3000端口。

**Q：生成的Skill验证不通过，L2检查报脚本语法错误？**

A：这通常是LLM生成代码时出现了"幻觉"（比如引用了不存在的命令或变量）。根据报错提示定位到具体脚本和行号，手动修正后重新运行`validate_skill.sh`即可。这也是为什么我们强烈建议在使用前先跑验证——它就是为了拦截这类问题而设计的。

**Q：支持Claude Code吗？**

A：支持。Skill-insight通过日志旁路的方式采集Claude Code的执行数据，不需要额外插件。具体接入方式参考项目文档中的Agent框架适配章节。

**Q：生成的Skill可以跨框架使用吗？**
A：可以。Skill-insight生成的Skill遵循统一的标准规范（YAML Frontmatter + Markdown + 脚本），不绑定特定Agent框架。你可以在OpenCode中生成，然后迁移到Claude Code或OpenClaw中使用。

# 相关链接：

1. 项目地址：<https://gitcode.com/openeuler/witty-skill-insight>
2. npm包：[@witty-ai/skill-insight](https://www.npmjs.com/package/@witty-ai/skill-insight)
