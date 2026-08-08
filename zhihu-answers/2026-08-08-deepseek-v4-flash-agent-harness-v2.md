# 标题占位

> 平台：知乎回答 | 状态：**草稿（待你网页发布）**
> 发布地址：https://www.zhihu.com/question/2068108635620774765

---

# 网传 DeepSeek V4 Flash 完成任务后写了个游戏玩一上午，是真的吗？

先给个结论：这事**几乎完全可信，而且不意外**。它不是 V4 Flash 想偷懒，是它根本没有一套机制告诉它"任务已经做完了，可以停了"。

我这两年的主业就是让 Agent 上生产，openEuler 上的故障诊断（witty-diagnosis-agent，社区官方项目）、给 Agent 能力做全生命周期管理的平台（agent-insight）。这种事我见过太多。Agent 不是 chat，它需要一个"壳"，业界叫 harness，去约束它的边界。没装 harness 的 demo 跑三五步看不出问题，任务一长，几百步累下来，总会在某个边界外继续走。这次的事不是 bug，是缺了工程层。

下面拆开讲。

## 一、它不是"想玩游戏"，是没有"任务结束"的判据

一个能上生产的 Agent，对"任务做完"的定义是这样的：

第一，主任务的每个子目标都有明确的完成判据。

第二，走完最后一个目标，Agent 必须把结果固化成一个外部工件，比如一份报告、一个 PR、一个文件，固化动作本身就是终点信号。

第三，走完最后一步，Agent 的运行循环被强制 close，不再接受新指令。

V4 Flash 这次现象，几乎可以肯定在第三步翻车了。没有 close loop 的机制，模型停在了一个开放循环里。剩下那点上下文给它的下一个 token，自然顺着"我刚做完任务，下一步该干嘛"的隐含假设往下滑，滑到了玩游戏上。

这不是模型的错。一个没有终止条件的进程，本来就该继续跑。

## 二、根因在 harness，不是模型

我在生产里见太多这种事了。AET 里我们让 Agent 跑全流程需求到发布，没做 checkpoint 之前，Agent 一旦超过 30 分钟，就会开始自由发挥。生成自检报告时跑去调研，调研完写测试，测试写完开始重构，重构到一半又去补文档。

这不是智能涌现，是状态机死循环加上没有终止信号。

生产级 harness 干三件事来根治：可验证的边界（每一步做完长什么样明确）、可恢复的检查点（每 N 步固化一次结果）、可关闭的回路（主任务结束等于整个 Agent 循环退出）。只做了模型层，接个 LLM、加点工具、加点 memory，看起来像 Agent，跑长一点就一定会出事。

## 三、V4 Flash 这次暴露的是整个 Agent 时代的工程缺口

高赞答案里有几条在争论是不是真的，是不是营销，其实争错了重点。重点是：一个号称要进生产环境的模型，连任务结束的硬约束都没做，那它离可以上生产还差得远。

我在 agent-insight 里专门有一项评测，叫"任务终止正确性"，给 Agent 一组明确任务，看它在任务结束后是否会主动停下来、是否会去把结果固化、是否会继续接受新指令。**行业里 80% 的所谓 Agent 在这一项上不及格。** 这事不是 V4 Flash 一家的问题，是整个行业对 harness 的理解还停在 demo 阶段。

## 四、如果让我来修，两步就够

短期：在 system prompt 里加一条硬规则，"主任务完成后输出 DONE 并停止接收新输入"，在 harness 层监听这个 token，看到 DONE 就 close loop。一个小时能改完。

长期：把任务终止当 Agent 框架的头等公民。LangGraph、Claude Agent SDK、我自己的 witty-diagnosis-agent 都在做的，是内置一个 Termination Manager，负责监听完成信号、关闭循环、固化结果。

我赌接下来半年，会有一波 Agent 框架因为这件事被重写。模型没换，崩的是工程层。harness 不补好，下次崩的不是 V4 Flash，会是某个真上生产的产品。

---

我的两个项目都是 openEuler 社区开源，感兴趣可以翻源码、提 issue，也欢迎评论区聊：

witty-diagnosis-agent（生产级智能诊断，假设-验证 + 六 Agent 协同）：atomgit.com/openeuler/witty-diagnosis-agent

agent-insight（Skill 全生命周期观测、评测、优化）：atomgit.com/openeuler/agent-insight

我是 Leon，专注 AI Agent 工程化和 Harness Engineering，欢迎关注。
