# LangSmith Engine：LangChain造了一个"专门给Agent看病"的Agent

> 你的Agent上线之后，你多久回看一次它的执行日志？一周一次？出了问题才看？还是根本不看？
>
> LangChain团队在吃了自己的狗粮之后，造了一个叫**LangSmith Engine**的东西——一个专门盯着生产环境Agent运行轨迹、发现反复出现的故障模式、然后自动生成修复方案的"元Agent"。上个月刚公测，已经有三家公司在用了。拆开来看看它怎么做的。

![alt text](image-2.png)

![alt text](image-3.png)

# 一、Agent上线之后，真正的痛苦才刚开始

做Agent开发的都经历过这个循环：看trace找问题 → 修prompt/代码 → 造数据集 → 跑实验 → 上线 → 接着看trace。每一轮都是人肉驱动，而且有几个环节特别容易掉链子：

- **单条trace看不出模式**：一个用户说Agent答错了，你点进去看，可能是因为tool description写糊了，也可能只是偶发幻觉。但如果**12% 的客服会话**都在同一个问题上翻车，单条看根本不知道这是系统性问题。
- **从生产数据造ground truth太繁琐**：把一条失败的trace手工转成测试用例，要写预期输出、标关键断言——这活在忙起来的时候第一个被跳过。
- **修完之后没针对性的回归守卫**：上线了一个修复，但没有针对这个具体failure mode的evaluator，下次再翻车只能等用户投诉。

LangChain自己在内部被这套循环搞烦了，做了一个决定：**与其让人盯着trace做这些事，不如让Agent自己来。**

于是有了LangSmith Engine。

它的定位很直白：**Engine本身就是一个Agent —— 一个专门"给Agent看病"的Agent。** 它接在LangSmith的tracing和evaluation基础设施之上，持续扫描生产trace，把重复出现的失败聚合成issue，然后针对每个issue自动生成evaluator、回归用例、甚至直接提PR修代码。

![alt text](image.png)

# 二、三个核心任务：找问题、讲清楚、变成可执行的改进

Engine的工作流可以概括为三步：

**第一步：找重复出现的失败。** 表面看是"监控告警"，但Engine做的不是盯着latency和error rate发报警——它往深一层走，从trace的结构里识别**模式化的失败**：Agent在同一个工具调用上反复循环、用错了参数、该调的工具漏掉了没调、同类用户请求在多条trace中一致性地返回了错误答案。

**第二步：把失败变成"可执行"的issue。** 这不是一句"Agent有时会答错"就完事了。每个issue带上了：归属类别（预定义的taxonomy，比如`agent_looping`、`incorrect_tool_args`、`missing_tool`、`pii_leak`）、严重程度（low/medium/high）、关联的证据trace、以及建议的处理动作。

**第三步：把issue变成持久化的改进。** 这是Engine跟普通监控工具最大的不同。它会为每个issue自动生成三样东西：一个**自定义 evaluator**（代码逻辑检查 trace 结构，或 LLM-as-judge 评估语义问题）、一组**回归断言**（不需要完整 ground truth，只声明正确答案必须满足什么条件）、一个`needs_fix`标签触发后续的修复Agent介入。

最终效果是：**每修一个问题，eval套件就自增长一层。** 同类型失败下次再出现，不用等用户投诉，在线evaluator直接拦住。


# 三、核心架构：一个六步闭环

Engine本身是一个**deep agent**（LangChain自己的长运行Agent框架），跑在**LangSmith Sandboxes**（Docker环境，带文件系统和代码执行能力）里。它把一个完整的"巡检 → 诊断 → 修复 → 记忆"循环拆成了六个步骤。

![alt text](image-1.png)

从宏观上看，引擎主要靠这几块驱动：

- **系统提示与指令**：里面包含了Agent概览。
- **沙箱**：引擎实际干活的环境。
- **LangSmith CLI**：引擎用来拉取数据、把更新推回LangSmith的主要交互界面。
- **自定义工具**：尤其是用来测试评估器、推荐回归示例的那些工具。
- **子Agent**：负责筛选链路追踪、排查潜在问题，避免撑爆主Agent的上下文。
- **记忆**：靠Agent概览来维护，并会根据用户操作动态更新。

## Step 1：准备上下文

每个run启动时，Engine先拉Sandbox环境（基础Docker镜像 + LangSmith CLI），可选地checkout项目仓库的指定分支和子目录。然后加载它的**记忆文件 —— Agent Overview**。这个文件类似于`AGENTS.md`，记录了Agent的职责描述、预期的trace结构、已知的常见失败模式、项目上下文，以及一个会随着Engine运行而自适应更新的"用户偏好"段落。

## Step 2：大规模trace初筛

这是整套架构里最值得讲的设计。生产环境的trace数量可能非常大，而且每条完整trace塞进上下文又贵又慢。Engine的解决办法分两层：

- **轨迹压缩（trajectory compression）：** 不读完整trace内容，而是把它压成骨架——每一轮只保留角色、可选工具名、延迟和内容大小。一条原本几千token的trace被压成几十个token。
- **专用初筛子Agent（Screener）：** 用**Claude Haiku**驱动一个专门做初筛的subagent，批量处理约20条轨迹压缩后的骨架。它输出的是一个结构化的三元组`trace_id | 问题类别 | 一行原因`，外加一个"干净"的trace计数。

这里的split很妙：**初筛追求规模和速度，用便宜模型 + 压缩信息；深度调查追求准确性，用强模型 + 完整上下文。**

## Step 3：深入调查可疑问题

初筛挑出来的问题trace，交给**调查子 Agent（Investigator）** 做深度分析。Investigator不是固定的system prompt —— 它是动态prompt的通用Agent，根据不同类型的问题用不同策略去查。它可以拉取完整trace内容，也可以选择性地读项目源码，理解工具定义和业务逻辑之后再下判断。

## Step 4：创建Issue、Evaluator和数据集

主Agent上场做"产出"：根据调查结果创建issue，然后逐一生成evaluator和回归用例。两个关键工具：

- `test_evaluator`：把新生成的evaluator直接在证据trace上跑一遍，返回`PASS | FAIL | SKIPPED`。不通过就回去改，不瞎交付。
- `propose_regression_example`：为每条证据trace生成一个回归断言。这里有个设计取舍 —— **生成的是"断言"而不是完整ground truth**。断言只声明"正确答案必须满足什么条件"（比如"回复中必须包含退款时间窗口"），而不约束具体措辞。好处是不会把正确答案的写法卡死。

## Step 5：分离修复链路

Engine的主Agent **不直接修代码**。它创建完issue后只打`needs_fix`标签。修复工作交给一个**独立的fix Agent**去提PR。

LangChain团队发现，如果让同一个Agent既定位问题又写修复，它在这两件性质完全不同的事情之间切换效果很差。拆开之后两边都更稳定。

## Step 6：更新记忆

每次运行结束后，Engine把它从这次调查中学到的东西 —— 哪些是新发现的failure pattern，用户对哪些issue做了什么操作（忽略、关闭、采纳） —— 写回Agent Overview文件。下轮运行时，它已经不是同一个Engine 了：它更了解你的Agent容易在哪类场景下出错，也更了解你作为开发者对什么类型的问题更敏感。


# 四、几个值得琢磨的设计取舍

| 决策 | 做法 | 原因 |
|------|------|------|
| **CLI作为主界面** | 不封装custom tool，直接调用LangSmith CLI | 灵活、可复现、方便debug |
| **轨迹压缩** | 初筛只看骨架，不看全文 | 全量trace在生产规模下token开销不可接受 |
| **预定义问题分类** | 不给Agent自由发挥issue类别 | 可控、可度量、防止输出五花八门 |
| **Haiku做初筛** | 专用subagent用便宜模型批量过 | 初筛是模式匹配任务，不需要深度推理 |
| **断言替代完整输出** | 回归用例只声明约束条件 | 不卡死正确答案的措辞，同时保证关键事实被验证 |
| **分离修复Agent** | issue创建和代码修复不同Agent处理 | 单一Agent同时做两件事效果显著下降 |

这些取舍背后有一个共同的原则：**不让任何一个环节变成"模型自由发挥"的黑盒。** 分类是预定义的，评估器要跑测试验证，修复要提 PR 给人审，记忆要持久化到文件里随时能看。每一步都留了人类介入的入口。

# 五、这意味着什么

LangSmith Engine做的事，本质上跟LangChain内部工程师每天手工做的事一模一样——看trace、找规律、写evaluator、建数据集、修代码。区别在于，Engine把这个循环从"人肉一周转一次"变成了**持续运行的闭环**。

这不是说人就被替代了。Engine的定位很务实：它负责把"能从trace里自动提取的模式"捞出来、整理好、生成初版修复方案；人在关键决策点介入 —— 审PR、确认issue、校准 evaluator的覆盖范围。LangChain明确说了，当前让Engine"完全不经过人审查就自动合并"还不行，但**对于已经被充分理解的问题类型，未来可能会朝这个方向走**。

对正在做Agent生产的团队来说，值得关注的不是Engine本身的功能清单，而是它背后的设计思路：**把Agent运维从"手工debug"升级为"Agent巡检 + 人审关键决策"的混合模式**。轨迹压缩、初筛分离、断言替代完整输出、修复链路独立 —— 这些设计是你即使不用LangSmith，自己搭类似系统时也能直接借鉴的工程经验。

话说回来，LangChain这次发布Engine，还释放了一个更重要的信号：**Agent基础设施的下半场，拼的不再是"能不能调用工具"，而是"上线之后能不能持续变好"**。Tracing是眼睛，Eval是尺子，Engine是把两者看到的和量到的自动转成下一步行动的闭环器。这个方向，才刚刚开始卷。


# 参考链接

1. Introducing LangSmith Engine: [https://www.langchain.com/blog/introducing-langsmith-engine](https://www.langchain.com/blog/introducing-langsmith-engine)
2. How We Built LangSmith Engine: [https://www.langchain.com/blog/how-we-built-langsmith-engine-our-agent-for-improving-agents](https://www.langchain.com/blog/how-we-built-langsmith-engine-our-agent-for-improving-agents)
