# 【Agent入门到精通】06-Skills系统：Agent的模块化能力

原文链接：https://zhuanlan.zhihu.com/p/1999207466928477997

---

​

目录

## [Skills系统](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=Skills%E7%B3%BB%E7%BB%9F&zhida_source=entity)：Claude Code的模块化能力封装

> **本系列简介** ：这是一套系统性的AI Agent技术教程，覆盖从基础概念到生产级应用的完整知识体系。本文是系列的第6篇。

**系列目录** ：

  1. [AI Agent的本质：从自动化到自主智能](https://zhuanlan.zhihu.com/p/1998733226689202164)
  2. [Agent架构设计：ReAct、ReWOO与思维链](https://zhuanlan.zhihu.com/p/1998817328603878485)
  3. [工具调用（Function Calling）：Agent的手和脚](https://zhuanlan.zhihu.com/p/1999201160813384313)
  4. [MCP协议深度解析：连接AI与数据源的标准化桥梁](https://zhuanlan.zhihu.com/p/1999203686707130939)
  5. [Workflow架构：可视化Agent编排平台](https://zhuanlan.zhihu.com/p/1999206497670947974)
  6. [Skills系统：Claude Code的模块化能力封装](https://zhuanlan.zhihu.com/p/1999207466928477997)
  7. [记忆系统：让Agent拥有上下文感知能力](https://zhuanlan.zhihu.com/p/2000654068553651951)
  8. [规划与推理：Agent如何分解复杂任务](https://zhuanlan.zhihu.com/p/2000654382052689377)
  9. [多模态Agent：视觉、语音与文本的融合](https://zhuanlan.zhihu.com/p/2000655295198823249)
  10. [上下文工程：Agent系统的”神经系统”](https://zhuanlan.zhihu.com/p/2003783359764112722)
  11. [Agent评估与优化：如何衡量Agent性能](https://zhuanlan.zhihu.com/p/2003203050169463280)
  12. [Multi-Agent系统：协作、竞争与涌现](https://zhuanlan.zhihu.com/p/2003212333846118909)
  13. [生产级Agent架构：可靠性、安全性与可观测性](https://zhuanlan.zhihu.com/p/2003761515438811109)
  14. [AI Agent的未来：AGI之路上的关键一步](https://zhuanlan.zhihu.com/p/2007391883593283178)



* * *

> 本文是《AI Agent系列教程》的第6篇，将深入讲解Anthropic Claude Code的Skills系统——一种用[Markdown](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=Markdown&zhida_source=entity)描述文件封装AI能力的革命性方式。

## 上一篇回顾

在第5篇《Workflow架构：可视化Agent编排平台》中，我们学习了如何通过可视化方式快速构建Agent。但Workflow也有其局限性，当需要高度定制化和灵活性时，Skills系统展现了更强大的能力。

**每次调用工具都要手动配置，如何复用这些能力？**

这就是**Skills系统** 要解决的核心问题。

* * *

## 引言：从工具调用到能力封装

### 问题的演进

**第一代：硬编码**
    
    
    # 每次都要写代码
    def process_pdf(file_path):
        # 实现逻辑
        pass
    
    result = process_pdf("doc.pdf")

**第二代：工具调用**
    
    
    # 每次都要配置工具
    tools = [pdf_tool]
    llm_response = call_llm(tools=tools)

**第三代：Skills**
    
    
    # 一次配置，自动使用
    ~/.claude/skills/pdf-processor/SKILL.md
    → Claude自动发现并使用

### Skills的核心价值

**传统方式的问题** ：

  * ❌ 每次都要手动配置工具
  * ❌ 能力难以在团队间共享
  * ❌ 维护成本高



**Skills的解决方案** ：

  * ✅ 一次配置，长期使用
  * ✅ Claude自动发现并调用
  * ✅ 通过Git轻松共享



* * *

## 一、什么是Skills？

### 1.1 官方定义

根据Anthropic官方文档：

> **Skills are modular capabilities that extend Claude’s functionality through organized folders containing instructions, scripts, and resources.**

翻译：**Skills是通过有组织的文件夹（包含指令、脚本和资源）来扩展Claude功能的模块化能力。**

### 1.2 核心特点
    
    
    Skills = Markdown描述 + 可选脚本 + Claude自动发现

**关键特性** ：

  1. **[Model-Invoked](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=Model-Invoked&zhida_source=entity) （模型自动调用）**：Claude根据请求自动决定是否使用
  2. **[Progressive Disclosure](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=Progressive+Disclosure&zhida_source=entity) （渐进式加载）**：只加载必要的文件
  3. **[Git-Native](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=Git-Native&zhida_source=entity) （Git原生）**：纯文本，易于版本管理
  4. **Granular Control（细粒度控制）** ：可限制使用的工具



### 1.3 Skills vs 其他方案

维度| Function Calling| Plugins| Skills  
---|---|---|---  
定义方式| 代码配置| 独立包| Markdown文件  
调用方式| User-Invoked| User-Invoked| Model-Invoked  
发现机制| 手动指定| 手动安装| 自动发现  
共享方式| 代码库| 包管理器| Git仓库  
学习成本| 高（需编程）| 中（需开发）| 低（Markdown）  
  
* * *

## 二、Skills的文件结构

### 2.1 基本结构
    
    
    my-skill/
    ├── SKILL.md          # 必需：[YAML frontmatter](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=YAML+frontmatter&zhida_source=entity) + Markdown内容
    ├── reference.md      # 可选：详细文档
    ├── examples.md       # 可选：使用示例
    ├── scripts/          # 可选：辅助脚本
    │   └── helper.py
    └── templates/        # 可选：模板文件
        └── template.txt

### 2.2 SKILL.md详解

**SKILL.md是核心文件** ，包含两部分：

**第一部分：YAML Frontmatter**
    
    
    ---
    name: pdf-processor
    description: Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction.
    allowed-tools: Read, Write, Bash
    ---

**字段说明** ：

  * `name`：Skill名称（小写字母、数字、连字符，最多64字符）
  * `description`：功能描述和何时使用（最多1024字符）
  * `allowed-tools`：可选项，限制Claude能使用的工具



**第二部分：Markdown内容**
    
    
    # PDF Processor
    
    ## Instructions
    Step-by-step instructions for Claude...
    
    ## Examples
    Concrete examples...
    
    ## Best Practices
    Tips and recommendations...

### 2.3 完整示例

**示例：Commit Message Helper**

**文件：`~/.claude/skills/commit-helper/SKILL.md`**
    
    
    ---
    name: generating-commit-messages
    description: Generates clear commit messages from git diffs. Use when writing commit messages or reviewing staged changes.
    ---
    
    # Generating Commit Messages
    
    ## Instructions
    
    1. Run `git diff --staged` to see changes
    2. I'll suggest a commit message with:
       - Summary under 50 characters
       - Detailed description
       - Affected components
    
    ## Best Practices
    
    - Use present tense ("Add feature" not "Added feature")
    - Explain what and why, not how
    - Reference related issues
    
    ## Examples
    
    **Example 1: Bug fix**

diff –staged

  * return True
  * return False


    
    
    Commit message:

fix(auth): correct login validation

The validation logic was inverted, causing legitimate logins to fail.

Fixes #123
    
    
    **Example 2: New feature**

diff –staged

  * def export_data():
  * ”““Export data to CSV”“”
  * …


    
    
    Commit message:

feat(data): add CSV export functionality

Users can now export their data to CSV format for external analysis.

Closes #456

* * *

## 三、Skills的工作原理

### 3.1 自动发现机制

**Claude如何发现Skills？**
    
    
    用户请求
        ↓
    Claude扫描Skills目录
        ↓
    读取所有SKILL.md的description
        ↓
    匹配请求与description
        ↓
    加载匹配的Skill
        ↓
    按照Instructions执行

**关键点** ：

  * ✅ Claude完全自主决策
  * ✅ 用户无需手动调用
  * ✅ 匹配基于description的语义



### 3.2 渐进式加载

**Claude不会一次性加载所有文件** ：
    
    
    第一次请求：
      加载 SKILL.md
          ↓
    需要更多信息？
      是 → 加载 reference.md
          ↓
    需要执行脚本？
      是 → 加载 scripts/helper.py
          ↓
    需要模板？
      是 → 加载 templates/template.txt

**好处** ：

  * ✅ 节省Token
  * ✅ 提高性能
  * ✅ 只加载必要信息



### 3.3 Model-Invoked vs User-Invoked

**对比** ：
    
    
    User-Invoked（用户调用）：
    /commit-message
    /data-analysis
    /pdf-processing
    
    Model-Invoked（模型自动调用）：
    "帮我写个commit message"
    "分析这个Excel数据"
    "从这个PDF提取表格"

**Skills的优势** ：

  * ❌ 不需要记住命令
  * ❌ 不需要手动调用
  * ✅ 完全自动化



* * *

## 四、创建你的第一个Skill

### 4.1 Personal Skills（个人Skills）

**存储位置** ：`~/.claude/skills/`

**步骤** ：
    
    
    # 1. 创建Skill目录
    mkdir -p ~/.claude/skills/my-first-skill
    
    # 2. 创建SKILL.md
    cat > ~/.claude/skills/my-first-skill/SKILL.md << 'EOF'
    ---
    name: my-first-skill
    description: A brief description of what this skill does and when to use it
    ---
    
    # My First Skill
    
    ## Instructions
    Provide clear, step-by-step guidance for Claude.
    
    ## Examples
    Show concrete examples of using this Skill.
    EOF
    
    # 3. 测试
    claude
    # 问：What skills are available?
    # 或者：匹配你description的场景

**使用场景** ：

  * ✅ 个人工作流
  * ✅ 实验性Skills
  * ✅ 个人生产力工具



### 4.2 Project Skills（项目Skills）

**存储位置** ：`.claude/skills/`（项目根目录）

**步骤** ：
    
    
    # 1. 在项目中创建Skill
    mkdir -p .claude/skills/team-skill
    
    # 2. 创建SKILL.md
    cat > .claude/skills/team-skill/SKILL.md << 'EOF'
    ---
    name: team-convention
    description: Follows team coding conventions. Use when writing, reviewing, or refactoring code.
    ---
    
    # Team Conventions
    
    ## Code Style
    - Use 4 spaces for indentation
    - Max line length: 100 characters
    - Follow PEP 8 guidelines
    
    ## Naming Conventions
    - Variables: snake_case
    - Classes: PascalCase
    - Constants: UPPER_SNAKE_CASE
    
    ## Documentation
    - All functions must have docstrings
    - Complex logic needs inline comments
    EOF
    
    # 3. 提交到Git
    git add .claude/skills/
    git commit -m "Add team convention skill"
    git push

**团队成员获得Skills** ：
    
    
    git pull
    claude  # Skills自动可用

**使用场景** ：

  * ✅ 团队工作流
  * ✅ 项目规范
  * ✅ 共享能力



### 4.3 高级Skills

### 示例1：带工具限制的Skill
    
    
    ---
    name: safe-file-reader
    description: Read files without making changes. Use when you need read-only file access.
    allowed-tools: Read, Grep, Glob
    ---
    
    # Safe File Reader
    
    This Skill provides read-only file access.
    
    ## Instructions
    1. Use Read to view file contents
    2. Use Grep to search within files
    3. Use Glob to find files by pattern
    
    ## Security
    This Skill cannot modify files, ensuring safe read-only access.

### 示例2：多文件Skill

**文件结构** ：
    
    
    pdf-tools/
    ├── SKILL.md
    ├── FORMS.md
    ├── REFERENCE.md
    └── scripts/
        ├── extract.py
        └── fill.py

**SKILL.md** ：
    
    
    ---
    name: pdf-tools
    description: Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction.
    ---
    
    # PDF Tools
    
    ## Quick Start
    
    Extract text:
    ```bash
    python scripts/extract.py doc.pdf

Fill forms: See [FORMS.md](https://zhuanlan.zhihu.com/p/1999207466928477997/FORMS.md) for detailed instructions.

For advanced usage, see [REFERENCE.md](https://zhuanlan.zhihu.com/p/1999207466928477997/REFERENCE.md).
    
    
    **Claude的使用**：

用户：”从这个PDF提取表格”

Claude：

  1. 加载SKILL.md
  2. 发现需要详细说明
  3. 加载FORMS.md
  4. 执行scripts/extract.py


    
    
    ---
    
    ## 五、Skills的最佳实践
    
    ### 5.1 Keep Skills Focused
    
    **好的粒度**：
    
    ✅ **Focused（聚焦）**：
    - `pdf-form-filler` - PDF表单填写
    - `excel-analyzer` - Excel数据分析
    - `git-commit-helper` - Git提交信息
    
    ❌ **Too Broad（太宽泛）**：
    - `document-processor` - 太宽泛
    - `data-tools` - 不明确
    - `helper` - 没意义
    
    ### 5.2 Write Clear Descriptions
    
    **好的description**：
    
    ```yaml
    # ✅ 具体、明确
    description: Analyze Excel spreadsheets, create pivot tables, and generate charts. Use when working with Excel files, spreadsheets, or .xlsx format.
    
    # ❌ 太模糊
    description: Helps with data

**关键点** ：

  * **说明功能** ：做什么
  * **说明场景** ：何时使用
  * **包含关键词** ：用户可能提到的词



### 5.3 Test Your Skills

**测试步骤** ：
    
    
    # 1. 创建Skill
    mkdir -p ~/.claude/skills/pdf-tester
    vim SKILL.md
    
    # 2. 启动Claude Code
    claude
    
    # 3. 测试
    # 问：我有个PDF文件，能帮我提取数据吗？
    
    # Claude应该：
    # - 自动发现pdf-tester Skill
    # - 按照Instructions执行

**如果Claude没用Skill** ：

检查清单：

  * [ ] `description`是否足够具体？
  * [ ] YAML语法是否正确？
  * [ ] 文件路径是否正确？
  * [ ] Skill是否在正确位置？



### 5.4 Document Skill Versions

**在SKILL.md中添加版本历史** ：
    
    
    # My Skill
    
    ## Version History
    
    - v2.0.0 (2025-01-15): Added support for new data format
    - v1.1.0 (2024-12-01): Improved error handling
    - v1.0.0 (2024-11-15): Initial release

* * *

## 六、Skills vs Workflow深度对比

### 6.1 开发体验对比

**场景：构建一个数据处理Agent**

**Workflow方式** ：
    
    
    1. 打开Workflow平台（Dify/Coze）
    2. 创建新流程
    3. 添加节点：数据读取
    4. 添加节点：数据清洗
    5. 添加节点：数据分析
    6. 添加节点：结果展示
    7. 配置每个节点的参数
    8. 连接节点
    9. 测试
    10. 调试
    11. 部署
    
    耗时：1-2天

**Skills方式** ：
    
    
    1. 创建Skill目录
    mkdir -p ~/.claude/skills/data-processor
    
    2. 编写SKILL.md
    vim SKILL.md
    # 写清楚Instructions
    
    3. 测试
    claude "分析这个CSV文件"
    
    耗时：1-2小时

### 6.2 维护对比

**需求变更** ：
    
    
    Workflow：
      修改流程图 → 重新配置节点 → 重新测试
      耗时：1天
    
    Skills：
      编辑SKILL.md → 保存
      耗时：10分钟

**团队协作** ：
    
    
    Workflow：
      导出配置 → Git提交 → 难以阅读diff → Code Review困难
      合并冲突：难以解决
    
    Skills：
      编辑SKILL.md → Git提交 → diff清晰可见 → Code Review容易
      合并冲突：标准merge

### 6.3 扩展性对比

**新增功能** ：
    
    
    Workflow：
      添加新节点 → 可能影响现有流程 → 需要全面测试
    
    Skills：
      创建新Skill → 独立于现有Skills → 无影响

* * *

## 七、Skills的高级应用

### 7.1 Skill组合

**场景：文档处理流水线**
    
    
    ~/.claude/skills/
    ├── pdf-extractor/
    │   └── SKILL.md
    ├── text-cleaner/
    │   └── SKILL.md
    ├── translator/
    │   └── SKILL.md
    └── formatter/
        └── SKILL.md

**Claude的使用** ：
    
    
    # 用户请求
    "处理这个中文PDF，提取表格并翻译成英文"
    
    # Claude自动：
    pdf-extractor → 提取内容
    text-cleaner → 清洗文本
    translator → 翻译成英文
    formatter → 格式化输出

**关键点** ：

  * ✅ 每个Skill独立
  * ✅ Claude自主组合
  * ✅ 无需手动编排



### 7.2 条件逻辑

**在SKILL.md中定义逻辑** ：
    
    
    ---
    name: smart-responder
    description: Intelligently respond to different types of messages. Use for customer support, email classification, or triage.
    ---
    
    # Smart Responder
    
    ## Decision Tree
    
    1. **Is it a complaint?**
       - If yes: Prioritize and escalate
       - If no: Continue to 2
    
    2. **Is it a feature request?**
       - If yes: Log to product backlog
       - If no: Continue to 3
    
    3. **Is it a question?**
       - If yes: Check knowledge base
       - If no: Standard response
    
    ## Examples
    
    **Complaint about billing:**

User: “你们收费太贵了！” Response: Escalate to human support within 1 hour
    
    
    **Feature request:**

User: “能不能添加导出功能？” Response: Log to product backlog, thank user
    
    
    **General inquiry:**

User: “支持哪些支付方式？” Response: Check knowledge base and answer

### 7.3 错误处理

**在SKILL.md中定义错误处理** ：
    
    
    ---
    name: robust-api-caller
    description: Call external APIs with retry logic. Use for integrations, webhooks, or API requests.
    ---
    
    # Robust API Caller
    
    ## Error Handling
    
    1. **Timeout**
       - Wait: 5 seconds
       - Retry: Up to 3 times
       - Then: Log error and notify
    
    2. **Rate Limit**
       - Wait: 60 seconds
       - Retry: Once
       - Then: Log and queue for later
    
    3. **Server Error**
       - Log details
       - Notify monitoring
       - Return graceful message
    
    ## Instructions
    
    When calling APIs:
    1. Check rate limits first
    2. Implement exponential backoff
    3. Log all errors for debugging

* * *

## 八、Skills的跨平台演进

### 8.1 从Claude Code到通用标准

**2024年的情况** ：

  * Skills是Claude Code的专属功能
  * 只能在Claude Code平台使用
  * 格式和规范由Anthropic定义



**2025年的趋势** ：

  * Skills的理念正在被其他平台采纳
  * 跨平台的Skills格式开始出现
  * Skills逐渐成为通用的能力封装标准



### 8.2 不同平台的Skills实现

### OpenAI的GPTs

**定义** ：

  * GPTs是ChatGPT的自定义AI助手
  * 通过配置和对话来定制能力
  * 可以分享给其他用户



**与Skills的对比** ：

特性| Claude Skills| [OpenAI GPTs](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=OpenAI+GPTs&zhida_source=entity)  
---|---|---  
定义方式| Markdown文件| 对话配置  
调用方式| Model-Invoked| User-Invoked  
可移植性| 可导出| 平台绑定  
编程| 可选脚本| Actions（API调用）  
分享| Git| 平台市场  
  
**相似性** ：

  * ✅ 都是能力封装
  * ✅ 都可以定制化
  * ✅ 都可以共享



**差异性** ：

  * Skills更开放（Git管理）
  * GPTs更易用（无代码）



### LangChain的Tools

**定义** ：

  * [LangChain Tools](https://zhida.zhihu.com/search?content_id=269599427&content_type=Article&match_order=1&q=LangChain+Tools&zhida_source=entity)是可重用的函数封装
  * 可以被Agent调用
  * 支持多种LLM平台



**与Skills的对比** ：

特性| Claude Skills| LangChain Tools  
---|---|---  
定义方式| Markdown描述| Python代码  
抽象层次| 能力描述| 函数封装  
跨平台| Claude专用| 多LLM支持  
版本管理| Git友好| 代码库  
  
**演进趋势** ：

  * LangChain开始支持类似Skills的描述方式
  * 社区出现”LangChain Skills”项目
  * 两者在融合



### 自研平台的Skills

**很多AI团队开始实现类似Skills的系统** ：
    
    
    # 通用Skills架构示例
    class SkillsRegistry:
        def __init__(self):
            self.skills = {}
    
        def register_skill(self, skill_def):
            """注册Skill"""
            name = skill_def.get("name")
            description = skill_def.get("description")
            instructions = skill_def.get("instructions")
    
            self.skills[name] = {
                "description": description,
                "instructions": instructions,
                "tools": skill_def.get("tools", [])
            }
    
        def find_matching_skills(self, user_request):
            """查找匹配的Skills"""
            matches = []
            for name, skill in self.skills.items():
                if self._match(skill["description"], user_request):
                    matches.append(name)
            return matches
    
        def _match(self, description, request):
            """语义匹配"""
            # 使用LLM判断匹配度
            return similarity(description, request) > threshold
    
    # 使用
    registry = SkillsRegistry()
    registry.register_skill({
        "name": "data-analyzer",
        "description": "Analyze Excel data and generate insights",
        "instructions": "...",
        "tools": ["pandas", "matplotlib"]
    })
    
    # 现在可以在任何LLM平台上使用
    matches = registry.find_matching_skills("分析这个Excel")

### 8.3 Skills的互操作性

**跨平台Skills的未来** ：

**通用Skills格式（可能在2025-2026年出现）** ：
    
    
    # universal-skill.yaml
    name: pdf-processor
    version: "1.0"
    description: Extract text, fill forms, merge PDFs
    
    # 通用元数据
    capabilities:
      - pdf_processing
      - form_filling
      - document_merge
    
    # 平台适配
    adapters:
      claude:
        format: "SKILL.md"
        path: "~/.claude/skills/"
      openai:
        format: "GPTs Actions"
        api: "openai-actions"
      langchain:
        format: "Python Tool"
        module: "langchain.tools"
    
    # 能力定义
    skills:
      - name: extract-text
        tools: ["pdf-reader"]
      - name: fill-form
        tools: ["form-filler"]

**好处** ：

  * ✅ 一次定义，多平台使用
  * ✅ 降低迁移成本
  * ✅ 促进Skills生态繁荣



### 8.4 Skills的标准化趋势

**三个层次的标准化** ：

**层次1：格式标准化**

  * YAML frontmatter格式
  * Markdown内容结构
  * 文件夹组织方式



**层次2：接口标准化**

  * 如何定义能力
  * 如何声明工具需求
  * 如何处理错误



**层次3：协议标准化**

  * Skills发现协议
  * Skills调用协议
  * Skills组合协议



**预测** ：

  * 2025年：Skills格式事实标准出现
  * 2026年：Skills协议标准化
  * 2027年：Skills生态系统成熟



### 8.5 跨平台Skills的实战案例

**案例：企业内部的自研平台**

**背景** ：

  * 某金融公司有自己的AI平台
  * 希望Skills能在多平台使用



**解决方案** ：
    
    
    # company-skills/data-analyzer/
    ├── skill.yaml          # 通用定义
    ├── claude/              # Claude适配
    │   └── SKILL.md
    ├── openai/              # OpenAI适配
    │   └── gpts_config.json
    └── langchain/           # LangChain适配
        └── tool.py

**效果** ：

  * ✅ 一次定义，多平台使用
  * ✅ 团队成员选择喜欢的平台
  * ✅ 降低平台锁定风险



* * *

## 九、Skills生态系统

### 8.1 Skills来源

**Skills的三个来源** ：
    
    
    1. Personal Skills
       ~/.claude/skills/
       ↓
    2. Project Skills
       .claude/skills/
       ↓
    3. Plugin Skills
       来自已安装的插件
       ↓
    All Available Skills

**查看所有可用的Skills** ：
    
    
    claude
    > What skills are available?
    
    # 或者
    > List all available skills

### 8.2 分享Skills

**方式1：通过Git共享**
    
    
    # 项目包含Skills
    my-project/
    └── .claude/skills/
        ├── code-reviewer/
        └── commit-helper/
    
    # 团队成员
    git pull
    # Skills自动可用

**方式2：通过Plugin共享**
    
    
    # Plugin结构
    my-plugin/
    ├── plugin.json
    └── skills/
        ├── skill1/
        └── skill2/
    
    # 安装Plugin
    claude plugin install my-plugin
    # Skills自动可用

### 8.3 Skills市场（未来趋势）

**预测：2025-2026年**

  * Skills Hub：集中分享Skills的平台
  * Skills Registry：可搜索的Skills仓库
  * Skills Marketplace：买卖Skills的市场



**就像** ：

  * npm包之于JavaScript
  * PyPI包之于Python
  * Skills之于Claude Code



* * *

## 九、Skills调试与优化

### 9.1 常见问题

**问题1：Claude不用我的Skill**

**原因** ：

  * description太模糊
  * YAML语法错误
  * 文件路径不对



**解决** ：
    
    
    # 检查YAML
    cat ~/.claude/skills/my-skill/SKILL.md | head -n 10
    
    # 检查路径
    ls ~/.claude/skills/my-skill/SKILL.md
    
    # 改进description
    # 从："Helps with files"
    # 到："Read, search, and analyze files"

**问题2：Skill加载失败**

**原因** ：

  * YAML格式错误
  * 缺少`---`分隔符
  * 使用了Tab而不是空格



**解决** ：
    
    
    # 验证YAML
    cat SKILL.md | head -n 15
    
    # 确保：
    # - 第1行是---
    # - YAML中使用空格不是Tab
    # - 关闭---在正确位置

### 9.2 性能优化

**优化1：减少Token使用**
    
    
    ---
    name: lightweight-skill
    description: Brief description
    ---
    
    # Instructions
    
    Keep this section concise. Move lengthy details to reference.md.
    
    For advanced usage, see [reference.md](reference.md).

**优化2：延迟加载**
    
    
    # SKILL.md（主文件）
    
    ## Quick Start
    Simple instructions here...
    
    See [examples.md](examples.md) for detailed examples.
    
    ## Advanced Usage
    See [reference.md](reference.md) for API reference.

**好处** ：

  * ✅ 只加载主文件
  * ✅ 按需加载其他文件
  * ✅ 节省Token



* * *

## 十、实战案例

### 案例1：构建代码审查Skill

**需求** ：

  * 自动审查代码质量
  * 检查安全漏洞
  * 遵循团队规范



**实现** ：

**文件：`.claude/skills/code-reviewer/SKILL.md`**
    
    
    ---
    name: code-reviewer
    description: Review code for best practices, security, and team conventions. Use when reviewing PRs, checking code quality, or analyzing code.
    allowed-tools: Read, Grep, Glob
    ---
    
    # Code Reviewer
    
    ## Review Checklist
    
    ### 1. Code Organization
    - [ ] File structure follows conventions
    - [ ] Functions are focused and single-purpose
    - [ ] Classes follow SRP principle
    
    ### 2. Security
    - [ ] No hardcoded secrets
    - [ ] Input validation present
    - [ ] SQL injection prevention
    - [ ] XSS prevention (for web code)
    
    ### 3. Performance
    - [ ] No unnecessary database queries
    - [ ] Proper caching
    - [ ] Efficient algorithms
    
    ### 4. Testing
    - [ ] Unit tests present
    - [ ] Edge cases covered
    - [ ] Tests are readable
    
    ### 5. Documentation
    - [ ] Functions have docstrings
    - [ ] Complex logic has comments
    - [ ] README is updated
    
    ## Instructions
    
    1. Read the target files
    2. Search for patterns using Grep
    3. Find related files using Glob
    4. Check against team conventions
    5. Provide actionable feedback
    
    ## Output Format
    
    For each issue found:

## [SEVERITY] Issue Title

**Location** : `file.py:123`

**Problem** : Description

**Suggestion** : How to fix

**Impact** : Why it matters
    
    
    ## Examples
    
    ### Example 1: Security Issue
    
    Input:
    ```python
    password = get_password()  # Hardcoded
    query = f"SELECT * FROM users WHERE name='{name}'"

Review:
    
    
    ## [CRITICAL] Security Vulnerabilities
    
    **Location**: `app.py:45`
    
    **Problem**:
    1. Hardcoded password
    2. SQL injection vulnerability
    
    **Suggestion**:
    ```python
    # Use environment variables
    password = os.getenv('DB_PASSWORD')
    
    # Use parameterized queries
    query = "SELECT * FROM users WHERE name=?"
    cursor.execute(query, (name,))

**Impact** : These vulnerabilities can lead to data breaches.
    
    
    ### Example 2: Performance Issue
    
    Input:
    ```python
    for user in users:
        result = db.query(User).filter_by(id=user.id)

Review:
    
    
    ## [MEDIUM] N+1 Query Problem
    
    **Location**: `views.py:78`
    
    **Problem**:
    Executes one query per user, causing N+1 queries.
    
    **Suggestion**:
    ```python
    user_ids = [u.id for u in users]
    results = db.query(User).filter(User.id.in_(user_ids)).all()

**Impact** : Slows down page load significantly.

### 案例2：构建数据分析Skill

**需求** ：

  * 分析Excel数据
  * 生成图表
  * 创建报告



**实现** ：

**文件：`~/.claude/skills/data-analyzer/SKILL.md`**
    
    
    ---
    name: data-analyzer
    description: Analyze Excel/CSV data, generate insights, and create visualizations. Use when working with spreadsheets, data analysis, or business intelligence.
    ---
    
    # Data Analyzer
    
    ## Capabilities
    
    1. **Data Loading**
       - Support Excel (.xlsx, .xls)
       - Support CSV
       - Support JSON
    
    2. **Analysis**
       - Descriptive statistics
       - Trend analysis
       - Correlation analysis
       - Outlier detection
    
    3. **Visualization**
       - Bar charts
       - Line charts
       - Pie charts
       - Scatter plots
       - Heatmaps
    
    4. **Reporting**
       - Summary statistics
       - Key insights
       - Recommendations
    
    ## Instructions
    
    When analyzing data:
    
    1. **Understand Context**
       - Ask clarifying questions if needed
       - Identify data source and domain
    
    2. **Data Exploration**
       - Check data structure
       - Identify data types
       - Look for missing values
    
    3. **Analysis**
       - Calculate statistics
       - Find patterns and trends
       - Detect anomalies
    
    4. **Visualization**
       - Choose appropriate chart type
       - Create clear, labeled charts
       - Highlight key insights
    
    5. **Reporting**
       - Summarize findings
       - Provide actionable insights
       - Make data-driven recommendations
    
    ## Best Practices
    
    - Always start with exploratory data analysis
    - Check data quality before analysis
    - Use appropriate statistical methods
    - Visualizations should be clear and misleading-free
    - Always provide context for insights
    
    ## Example Usage
    
    **Example 1: Sales Analysis**

Input: sales_2024.xlsx

Analysis:

  * Total sales: 1.2M - Best month: December (180K)
  * Best product: Product A ($500K)
  * Trend: Growing 15% YoY



Recommendations:

  * Increase inventory for Product A
  * Run promotion in Q1
  * Expand to new markets


    
    
    **Example 2: Customer Churn**

Input: customer_data.csv

Key Findings:

  * Churn rate: 12%
  * At-risk customers: 234
  * Main churn reason: Price sensitivity



Recommendations:

  * Offer discounts to at-risk segment
  * Improve onboarding process
  * Add loyalty program


    
    
    ## Output Format
    
    Provide analysis in this structure:
    
    ```markdown
    # Data Analysis Report
    
    ## 📊 Data Overview
    - Data source: ...
    - Rows: ...
    - Columns: ...
    - Time period: ...
    
    ## 📈 Key Metrics
    1. Metric 1: ...
    2. Metric 2: ...
    3. Metric 3: ...
    
    ## 🔍 Insights
    ### Insight 1
    **Finding**: ...
    **Impact**: ...
    **Recommendation**: ...
    
    ### Insight 2
    **Finding**: ...
    **Impact**: ...
    **Recommendation**: ...
    
    ## 📉 Visualizations
    [Charts will be created]
    
    ## 💡 Recommendations
    1. Recommendation 1
    2. Recommendation 2
    3. Recommendation 3
    **使用**：
    
    ```bash
    # 分析销售数据
    claude "分析这个Excel文件的2024年销售数据，生成报告"
    
    # 分析客户流失
    claude "从这个CSV分析客户流失率，找出主要原因"

* * *

## 十一、Skills vs 传统方案总结

### 11.1 全面对比

维度| 传统方案| Skills  
---|---|---  
定义方式| 代码/可视化| Markdown描述  
调用方式| 手动调用| 自动发现  
学习成本| 高/中| 低  
维护成本| 高| 低  
团队协作| 中/难| 易  
版本管理| 中/难| 易  
扩展性| 中| 高  
适用场景| 所有| 重复性任务  
  
### 11.2 选择指南

**何时使用Skills** ：

✅ **重复性任务** ：频繁使用的功能 ✅ **团队共享** ：多人需要的能力 ✅ **需要维护** ：长期演进的功能 ✅ **需要版本控制** ：频繁更新的能力

**何时不用Skills** ：

❌ **一次性任务** ：只用一次的功能 ❌ **简单操作** ：直接调用更简单 ❌ **快速原型** ：Workflow可能更合适

* * *

## 十二、总结与展望

### 12.1 核心要点

**Skills是什么** ：

  * Markdown描述文件系统
  * Claude自动发现并使用
  * 一次配置，长期复用



**Skills的核心价值** ：

  1. **简单性** ：Markdown描述，无需编程
  2. **自动化** ：Claude自动发现，无需手动调用
  3. **可维护性** ：文本文件，易于修改
  4. **可共享性** ：Git管理，团队协作



### 12.2 最佳实践

  * ✅ **Keep Skills Focused** ：一个Skill做一件事
  * ✅ **Write Clear Descriptions** ：具体、明确
  * ✅ **Test Thoroughly** ：确保Claude能正确使用
  * ✅ **Document Versions** ：版本历史
  * ✅ **Use Progressive Disclosure** ：主文件简洁，详情放参考文件



### 12.3 未来趋势

**2025年** ：

  * Skills生态爆发
  * Skills Hub/Gallery出现
  * 团队普遍采用



**2026年** ：

  * Skills成为标准配置
  * Workflow平台被迫支持Skills
  * IDE集成



**2027+** ：

  * AI自动生成Skills
  * 自然语言创建Skills
  * Skills市场成熟



* * *

## 推荐资源

**官方文档** ：

  * [Anthropic Skills Documentation](https://link.zhihu.com/?target=https%3A//docs.anthropic.com/en/docs/claude-code/skills)
  * [Skills Best Practices Guide](https://link.zhihu.com/?target=https%3A//docs.anthropic.com/en/docs/claude-code/skills/best-practices)



**实战示例** ：

  * [Official Skills Repository](https://link.zhihu.com/?target=https%3A//github.com/anthropics/claude-code-skills)
  * [Community Skills Examples](https://link.zhihu.com/?target=https%3A//github.com/topics/claude-skills)



**相关文章** ：

这是《AI Agent系列教程》的第6篇，共14篇。

> 上一篇：[【Agent入门到精通】05-Workflow架构：可视化Agent编排平台](https://zhuanlan.zhihu.com/p/1999206497670947974)  
> 下一篇：[【Agent入门到精通】07-记忆系统：让Agent拥有上下文感知能力](https://zhuanlan.zhihu.com/p/2000654068553651951)

* * *

 _如果这篇文章对你有帮助，欢迎点赞、收藏和分享！有任何问题欢迎在评论区讨论。_

* * *

**系列说明** ：

  * 本系列文章正在持续更新中，欢迎关注！
  * 所有代码示例将在GitHub仓库开源：`ai-agent-tutorial-series`
  * 有问题欢迎在评论区讨论，我会及时回复


