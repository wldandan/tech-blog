# 第2篇：Static静态优化模式实战（通用Skill优化场景）

> 在上一篇文章中，我们对Skill-insight优化能力进行了整体的介绍，然后为大家演示了如何基于用户反馈优化Skill。本文将聚焦最基础、最通用的 **Static（静态优化）模式**——它不依赖任何运行时数据，仅基于 Skill 自身代码与配置进行冷启动式质量评估与修复，是 Skill 发布前的必备质量门禁。

# 一、什么是Static静态优化

## 1.1 功能介绍

Static静态优化是Skill-insight四种优化模式中最基础的一种。它的核心特点是：**不依赖任何运行时数据**，仅基于Skill自身代码与配置进行"冷启动式"评估与修复。

当你的Skill刚写好还没跑过，或者你想在发布前做一次全面的质量体检时，Static模式是你的最佳选择。它会从六个维度对Skill进行系统性评估，自动发现潜在问题并生成诊断报告，然后进行定点修补。

## 1.2 六维评估体系

Static模式的核心是一套 **六维质量评估体系**，涵盖Skill的各个方面：

| 评估维度 | 评估内容 | 典型问题示例 |
|---------|---------|-------------|
| **职责明确性** | Skill是否有清晰、单一的职责定位 | 一个Skill同时处理CPU、内存、磁盘三种故障排查，职责边界模糊。 |
| **结构规范性** | SKILL.md的结构是否符合规范，步骤编号是否连续。 | 步骤编号混乱（Step 1 → Step 3 → Step 2），缺少必要的frontmatter元数据。 |
| **指令适配性** | 指令是否足够具体、可执行，是否覆盖边界条件。 | 只写了"排查CPU问题"，没有指定使用`top`还是`htop`，没有说明排序字段。 |
| **内容一致性** | SKILL.md与references目录下的参考文档是否一致。 | SKILL.md 中提到`commit-types.md`定义了10种类型，但文件中只有7种。 |
| **风险可控性** | 是否包含高危操作，是否有备份/回滚机制。 | 包含`rm -rf`命令但未限定路径，修改系统参数前无备份提示。 |
| **脚本及文档质量** | 脚本是否存在语法错误，参考文档是否完整可用。 | Bash脚本缺少`#!/bin/bash`头部，正则表达式存在语法错误。 |

## 1.3 硬规则与软规则

Static模式的评估分为两类规则：

**硬规则（不依赖大模型）**：
- **格式校验**：YAML frontmatter格式正确性检查。
- **安全扫描**：高危指令扫描（如`rm -rf /*`、`chmod 777`等未经限制的危险操作）。
- **结构检查**：必需字段是否存在，文件结构是否完整。

硬规则由代码检查器（Linter）强制执行，不达标会直接报错，必须修复后才能继续。

**软规则（基于大模型评估）**：
- 六维质量评分
- 语义一致性分析
- 指令可执行性判断

软规则由大模型进行语义分析，得分不达标会生成诊断报告并自动修补。

### 1.4 适用场景

Static模式适用于以下场景：

- **Skill发布前的质量门禁**：确保新Skill在上线前符合质量标准。
- **刚写好的Skill需要体检**：不知道Skill是否有问题，想先做一次全面检查。
- **通用Skill优化**：不依赖特定运行环境，适用于任何Skill。
- **团队协作规范化**：统一团队Skill的质量标准，避免低质量Skill进入项目。

> 💡 **与其他模式的区别**：Dynamic模式基于运行失败的Trace日志进行修复，Feedback模式基于用户主观反馈进行迭代，Hybrid模式是static → dynamic的组合。Static模式是唯一不依赖运行数据的"冷启动"优化方式。


# 二、准备工作

在开始Static静态优化之前，请确认以下环境已就绪：

## 2.1 Agent编码终端

你需要一个支持Agent能力的编码终端。目前支持以下框架：

| 框架 | 说明 |
|-----|------|
| **OpenCode** | 开源Agent框架，支持Slash Command调用优化器。 |
| **Claude Code** | Anthropic官方 CLI，支持自然语言调用优化器。 |

确保终端可以正常执行`npx`命令，并且网络可达Skill-insight的仓库地址。

## 2.2 Node.js运行时

Skill-insight基于Node.js运行，请确保本地已安装Node.js（建议v20及以上）和npm：

```bash
node -v   # 应输出 v20.x 或更高
npm -v    # 应输出 10.x.x 或更高
```

## 2.3 待优化的Skill

你需要一个待优化的Skill 目录。Skill目录结构应符合规范：

```
your-skill/
├── SKILL.md              # 核心Skill文件（必需）
└── references/           # 参考文档目录（可选）
    ├── doc1.md
    └── doc2.md
```

**SKILL.md必需结构**：

```markdown
---
name: skill-name
description: Skill的简要描述
---

## Step 1：步骤名称
...
```

# 三、优化操作案例

下面以一个真实案例演示Static静态优化的完整流程。我们使用**vmcore-analysis**Skill作为示例，这是一个Linux内核崩溃转储分析Skill，在 Static优化过程中发现了多个典型的质量问题并进行了修复。

## 示例 Skill：vmcore-analysis

这是一个用于分析Linux内核崩溃转储文件(vmcore)的 Skill，通过crash工具提取关键诊断信息，识别OOM Killer、Kernel Panic、Deadlock等故障模式。

**优化前的目录结构**：

```
vmcore-analysis/
├── SKILL.md              # 核心Skill文件
└── scripts/              # 排查脚本目录
    ├── _lib.sh           # 通用函数库
    ├── collect.sh        # 信息采集脚本
    ├── check_oom.sh      # OOM排查脚本
    ├── check_panic.sh    # Panic排查脚本
    └── ...
```

**存在的问题**：
- **风险可控性缺失**：缺少安全边界说明，没有权限检查和预检流程。
- **指令适配性不足**：Step 1只有简单的采集命令，没有环境验证步骤。
- **脚本质量欠缺**：`_lib.sh`缺少安全检查函数。
- **参考文档缺失**：缺少`references`目录和crash工具使用指南。

## Step 1：环境依赖安装

首先，在Agent终端中安装Skill-insight Meta-Skill，为Agent注入优化能力：

```bash
npx skills add https://gitcode.com/openeuler/witty-skill-insight.git
```

安装过程中，交互式选择要安装的Skill：
- 选择**skill-optimizer**（优化器）
- 选择集成框架（如OpenCode）

安装完成后，优化器将作为Agent可调用的Skill注册到当前环境中。

## Step 2：数据准备

准备好待优化的Skill目录，确认目录结构正确。

**检查Skill目录结构**：

```bash
# 查看Skill目录
ls -la ~/.opencode/skills/vmcore-analysis/

# 确认 SKILL.md 存在
cat ~/.opencode/skills/vmcore-analysis/SKILL.md
```
---

## Step 3：执行优化

现在开始执行Static静态优化，在此，我们通过自然语言触发优化。

在OpenCode终端中直接告诉它你的需求：

```
使用static（静态优化）模式，优化一下 vmcore-analysis Skill。
```

![alt text](image.png)

**优化过程详解**：

引擎执行以下步骤：

1. **硬规则校验**：
   - 检查SKILL.md的YAML frontmatter格式 ✓
   - 扫描是否存在高危指令 ✓
   - 检查必需字段（name、description）是否存在 ✓

2. **软规则评估**：
   - 调用大模型对Skill进行六维评分
   - 生成各维度的诊断报告

3. **定点修补**：
   - 仅针对不达标的维度进行修改
   - 保持原有正确部分不变

**实际优化输出**：

```
✅ vmcore-analysis静态优化完成！

变更摘要：
- 新增安全边界章节（权限、只读分析、数据安全）。
- Step 1新增环境验证与预检流程。
- _lib.sh增加safety_check()函数。
- 新建references/crash_tool_guide.md。

已加载到： ~/.opencode/skills/vmcore-analysis
旧版已归档至： ~/.skill-insight/skill-history/vmcore-analysis-20260428_145242
```

![alt text](image-1.png)


## Step 4：对优化后的技能进行调试、验证

优化完成后，系统进入**Diff Review**环节，展示优化前后的变更对比。

![alt text](image-2.png)

以下是本次优化的关键变更：

**新增安全边界章节**

```diff
+## 安全边界 (Security Boundaries)
+
+**权限要求与风险说明:**
+- 所有诊断脚本需以 root 权限执行 (`sudo`),用于读取 vmcore 文件和调用 crash 工具
+- **只读分析**: 所有脚本仅执行诊断分析操作,不会修改 vmcore、系统配置或运行环境
+- **数据安全**: 不上传数据到外部服务,所有分析在本地完成
+
+**强制预检 (Mandatory Pre-flight Checks):**
+执行任何诊断脚本前,必须验证以下条件:
+
+1. **脚本来源验证**: 确认脚本来自可信来源且未被篡改
+2. **环境依赖检查**: 脚本会自动检查 crash 工具、vmlinux debug symbols 的可用性
+3. **输入验证**: VMCORE_PATH 必须指向有效的 vmcore 文件
+4. **最小权限原则**: 仅在必要时使用 sudo -E 传递环境变量
+
+**执行约束:**
+- 仅对已存在的 vmcore 文件执行分析
+- 不执行任何写操作或系统修改
+- 若安全检查失败,脚本将立即终止并报告错误
```

**新增环境验证与预检流程**

```diff
-### Step 1: 信息采集
-
-运行基础信息采集,验证环境并获取 vmcore 基本信息:
-
-```bash
-VMCORE_PATH=/path/to/vmcore sudo -E bash scripts/collect.sh | tee /tmp/vmcore_collect.json
-```
-
-> 注意: 需要先安装 crash 工具和对应的 kernel-debuginfo 包
+### Step 1: 环境验证与信息采集
+
+**1.1 验证环境依赖:**
+
+```bash
+# 检查 crash 工具是否安装
+command -v crash || echo "Install: yum install crash (RHEL) / apt install linux-crashdump (Debian)"
+
+# 检查 kernel-debuginfo 是否安装
+ls /usr/lib/debug/lib/modules/$(uname -r)/vmlinux 2>/dev/null || \
+  echo "Install: debuginfo-install kernel-$(uname -r) (RHEL) / apt install linux-image-$(uname -r)-dbgsym (Debian)"
+```
+
+**1.2 运行安全预检并采集基础信息:**
+
+```bash
+VMCORE_PATH=/path/to/vmcore sudo -E bash scripts/collect.sh | tee /tmp/vmcore_collect.json
+```
+
+> **重要**: `collect.sh` 内置 `safety_check()` 函数,会自动验证:
+> - root 权限
+> - VMCORE_PATH 环境变量
+> - vmcore 文件存在且可读
+> - crash 工具可用性
+> - vmlinux debug symbols 可用性
+> - 脚本完整性
+>
+> 若预检失败,脚本将终止并输出错误信息。请根据提示修复后再继续。
```

**_lib.sh新增safety_check()函数**

```bash
# Safety check - validate environment before running diagnostic operations
# Usage: safety_check
# Checks:
#   1. Running as root or with sudo
#   2. VMCORE_PATH is set and file exists
#   3. crash tool is available
#   4. vmlinux debug symbols exist
# Returns: 0 on success, 1 on failure
safety_check() {
    local errors=0
    
    # Check root privileges
    if [[ $EUID -ne 0 ]]; then
        error "This script requires root privileges. Run with sudo."
        ((errors++))
    fi
    
    # Check VMCORE_PATH
    if [[ -z "${VMCORE_PATH:-}" ]]; then
        error "VMCORE_PATH environment variable is not set."
        error "Usage: VMCORE_PATH=/path/to/vmcore sudo -E bash $0"
        ((errors++))
    elif [[ ! -f "${VMCORE_PATH:-}" ]]; then
        error "vmcore file not found: ${VMCORE_PATH}"
        ((errors++))
    elif [[ ! -r "${VMCORE_PATH:-}" ]]; then
        error "vmcore file not readable: ${VMCORE_PATH}"
        ((errors++))
    fi
    
    # Check crash tool
    if ! command -v crash &>/dev/null; then
        error "crash tool not found. Install: yum install crash / apt install linux-crashdump"
        ((errors++))
    fi
    
    # Check vmlinux debug symbols
    local kernel_ver="${KERNEL_VERSION:-$(uname -r)}"
    local vmlinux_paths=(
        "/usr/lib/debug/lib/modules/${kernel_ver}/vmlinux"
        "/usr/lib/debug/lib/modules/${kernel_ver}/vmlinux-$(uname -m)"
        "/boot/vmlinux-${kernel_ver}"
    )
    local vmlinux_found=false
    for vmlinux in "${vmlinux_paths[@]}"; do
        if [[ -f "$vmlinux" ]]; then
            vmlinux_found=true
            break
        fi
    done
    if [[ "$vmlinux_found" != "true" ]]; then
        warn "vmlinux debug symbols may be missing for kernel ${kernel_ver}"
        warn "Install: debuginfo-install kernel-${kernel_ver} (RHEL) or linux-image-${kernel_ver}-dbgsym (Debian/Ubuntu)"
    fi
    
    # Check script integrity (detect tampering)
    local script_dir="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
    if [[ ! -f "$script_dir/_lib.sh" ]]; then
        error "Script library integrity check failed. _lib.sh not found in expected location."
        ((errors++))
    fi
    
    if [[ $errors -gt 0 ]]; then
        error "Safety check failed with $errors error(s). Aborting."
        return 1
    fi
    
    info "Safety check passed."
    return 0
}
```

**4新建references/crash_tool_guide.md**

优化后新增了crash工具使用指南参考文档，位于`references/crash_tool_guide.md`。

**优化后的目录结构**：

```
vmcore-analysis/
├── SKILL.md                  # 核心 Skill 文件（已优化）
├── scripts/                  # 排查脚本目录
│   ├── _lib.sh               # 通用函数库（新增 safety_check）
│   ├── collect.sh            # 信息采集脚本
│   └── ...
└── references/               # 参考文档目录（新增）
    └── crash_tool_guide.md   # crash 工具使用指南
```

**交互式确认**：

系统提供以下快捷指令：

- **Accept**：满意，落盘为新版本
- **Revise**：不满意，提供反馈继续修改
- **Revert**：不满意，回滚到原版本

选择**Accept**后，系统保存优化后的版本。

## Step 5：执行优化后的技能

确认Accept后，优化后的Skill已替换原版本。现在可以实际执行来验证效果。

**在OpenCode终端中执行**：

```
分析这个 vmcore 文件，帮我找出内核崩溃的原因。
vmcore 路径：/path/to/vmcore
```

Agent 会基于优化后的 Skill 执行排查，你应该能看到：
- Agent 首先执行环境验证（检查 crash 工具、vmlinux debug symbols）
- Agent 运行安全预检（safety_check）
- Agent 执行基础信息采集
- Agent 根据故障现象选择对应的排查场景
- Agent 输出诊断结论

**预期执行流程**：

```
Step 1: 环境验证与信息采集
  → 检查 crash 工具是否安装 ✓
  → 检查 vmlinux debug symbols ✓
  → 执行 safety_check() 预检
  → 运行 collect.sh 采集基础信息

Step 2: 故障模式识别
  → 根据日志关键字判断故障类型
  → 命中 "Kernel panic" 关键字
  → 选择场景 1：内核崩溃

Step 3: 执行排查
  → 运行 check_panic.sh
  → [HIT] 发现崩溃点在网卡驱动
  → 分析调用栈定位根因

诊断结论:
  故障根因: Kernel Panic
  故障组件: 网卡驱动 xxx
  定位依据: bt 显示崩溃在驱动函数 xxx_send()
```

---

## Step 6：进行评测和对比

优化后，可以通过Skill-insight的评测能力进行量化验证。优化后的Skill会保存在`snapshots/`目录中，旧版本已归档：

```
# 当前版本（优化后）
~/.opencode/skills/vmcore-analysis/

# 旧版本归档
~/.skill-insight/skill-history/vmcore-analysis-20260428_145242/
```

归档目录中包含完整的旧版本文件和优化元数据，方便追溯和回滚。

# 四、小结

本文以vmcore-analysis Skill为真实案例，演示了Static静态优化模式的完整流程：

1. **安装优化器组件**：通过`npx skills add`为Agent注入优化能力。
2. **准备待优化Skill**：确认目录结构和SKILL.md格式正确。
3. **执行Static优化**：引擎自动进行六维评估和定点修补。
4. **Diff Review确认**：可视化查看变更，交互式确认落盘。
5. **执行验证效果**：实际运行Skill，观察执行流程变化。
6. **评测量化对比**：通过六维评分验证优化效果。

**本次优化的关键变更**：

| 变更项 | 变更内容 | 改进维度 |
|-------|---------|---------|
| 新增安全边界章节 | 权限要求、只读分析、数据安全说明 | 风险可控性 |
| Step 1新增预检流程 | 环境验证、安全预检步骤 | 指令适配性 |
| `_lib.sh`增加`safety_check()` | 环境验证函数，执行前强制检查 | 风险可控性 |
| 新建`crash_tool_guide.md` | crash 工具使用参考文档 | 脚本及文档质量 |

**Static 模式的核心价值**：

- **发布前质量门禁**：在Skill上线前发现并修复潜在问题。
- **不依赖运行数据**：适用于任何 Skill，无需等待运行失败。
- **六维全面评估**：从职责、结构、指令、一致性、风险、文档六个维度系统评估。
- **定点修补**：只改需要改的，不碰不该碰的，避免误改正常部分。

**适用场景总结**：

| 场景 | 推荐模式 |
|-----|---------|
| Skill刚写好还没跑过 | ⭐ Static |
| Skill发布前的质量检查 | ⭐ Static |
| Skill运行失败需要修复 | Dynamic |
| Skill能跑但不贴合需求 | Feedback |
| 需要彻底优化 | Hybrid |

掌握了Static模式后，你可以继续探索Dynamic和Feedback模式，应对更复杂的优化场景。

想第一时间获取后续内容？欢迎关注项目仓库并Star：

🔗 **项目地址**：<https://gitcode.com/openeuler/witty-skill-insight>

📦 **npm包**：[@witty-ai/skill-insight](https://www.npmjs.com/package/@witty-ai/skill-insight)

有问题？提Issue，社区一起搞定：[Issue入口](https://atomgit.com/openeuler/witty-skill-insight/issues)

---


# 五、常见问题

**Q1：六维评分的阈值是多少？多少分算达标？**

默认情况下，每个维度的达标阈值是7/10。如果某维度得分低于7，引擎会生成诊断报告并进行修补。你可以通过配置文件自定义阈值。

**Q2：硬规则校验失败怎么办？**

硬规则校验失败会直接报错，必须修复后才能继续。常见硬规则问题：

| 问题 | 解决方法 |
|-----|---------|
| YAML frontmatter格式错误 | 检查`---`分隔符是否正确，字段是否有语法错误。 |
| 缺少必需字段 | 添加`name`和`description`字段。 |
| 高危指令未限定 | 给`rm -rf`等命令添加路径限定，如`rm -rf /tmp/xxx`。 |

**Q3：优化后的版本可以回滚吗？**

可以。所有版本都保存在归档目录中。如果你想回滚到之前的版本：

**方式一**：在Diff Review界面选择Revert。

**方式二**：手动复制历史版本
```bash
cp ~/.skill-insight/skill-history/vmcore-analysis-20260428_145242/SKILL.md ~/.opencode/skills/vmcore-analysis/SKILL.md
```

**方式三**：在OpenCode上通过指令回滚
```
将 Skill 回滚到上一个版本
```

**Q4：我的Skill没有references目录，Static模式还能用吗？**

可以。references目录是可选的。如果Skill没有references，Static模式会跳过"内容一致性"维度的检查（自动判定为满分），只评估其他五个维度。优化过程中，引擎可能会根据需要自动创建references目录并补充参考文档（如本次vmcore-analysis案例）。

**Q7：Static模式可以用于优化别人写的Skill吗？**

可以。无论Skill是谁写的、通过什么方式生成的，只要符合目录结构规范，Static模式都可以进行质量评估和优化。这正是Static模式的价值——为团队提供统一的Skill质量标准。


# 相关链接

1. **项目仓库**：[https://gitcode.com/openeuler/witty-skill-insight](https://gitcode.com/openeuler/witty-skill-insight)
2. **npm 包**：[@witty-ai/skill-insight](https://www.npmjs.com/package/@witty-ai/skill-insight)
3. **Issue 入口**：[https://atomgit.com/openeuler/witty-skill-insight/issues](https://atomgit.com/openeuler/witty-skill-insight/issues)
>
