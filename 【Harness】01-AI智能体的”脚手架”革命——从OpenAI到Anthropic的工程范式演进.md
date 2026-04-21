# 【Harness】01-AI智能体的”脚手架”革命——从OpenAI到Anthropic的工程范式演进

原文链接：https://zhuanlan.zhihu.com/p/2023121420058633079

---

​

目录

## 引言

2026年无疑是AI智能体工程化爆发的元年。当我们还在惊叹于大模型的能力突破时，科技巨头们已经在悄然进行一场更深层次的革命——**Harness工程** 。这不是又一个算法创新，而是彻底改变AI开发范式的工程方法论革命。

从OpenAI年初提出”[Harness Engineering](https://zhida.zhihu.com/search?content_id=272437256&content_type=Article&match_order=1&q=Harness+Engineering&zhida_source=entity)”理念，到Anthropic3月发布[多智能体Harness设计框架](https://zhida.zhihu.com/search?content_id=272437256&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93Harness%E8%AE%BE%E8%AE%A1%E6%A1%86%E6%9E%B6&zhida_source=entity)，再到51万行Claude Code源码意外泄露所展示的工业级Harness实现，整个行业正在快速形成共识：**未来的AI应用，80%的价值将来源于Harness设计，而不是模型本身** 。

本系列博客将深入解析Harness技术的前世今生、核心设计理念、工业级实现方案，以及它将如何重构整个AI开发流程。

* * *

## 一、什么是Harness？

Harness直译为”马具”、”安全带”，在AI工程语境下，我们可以把它理解为**智能体的”脚手架”或”运行环境”** 。它不是一个具体的产品，而是一套工程方法论和技术框架，核心解决三个核心问题：

  1. **如何让AI智能体可靠地完成长周期复杂任务** （从几分钟的简单问答到几小时甚至几天的软件开发）。
  2. **如何让多个智能体高效协同** （像人类团队一样分工合作、互相评审、迭代优化）。
  3. **如何约束智能体的行为边界** （确保输出符合预期、避免幻觉、保证安全性）。



正如Anthropic在其官方博客中所说：”让AI写出好代码的关键不是换更强的模型，而是给模型搭一个脚手架。就像GAN里的生成器和判别器互相对抗一样，让一个agent写代码、另一个agent像严格的QA一样挑毛病，两者对抗着迭代，最终产出远超单agent能力天花板的作品。”

* * *

## 二、Harness的演进历史

Harness并不是一个全新的概念，它的发展经历了三个主要阶段：

### 2.1 第一代：单智能体Wrapper（2023-2024）

最早的Harness雏形是对大模型的简单封装，主要解决API调用、错误重试、上下文管理等基础问题。代表性的工作包括LangChain、LlamaIndex等框架，核心是让开发者更容易调用大模型能力，但还没有涉及多智能体协同和长任务运行。

### 2.2 第二代：多智能体编排（2024-2025）

随着AutoGPT等项目的流行，行业开始探索多智能体协作模式。这一阶段的Harness主要关注任务分解、角色分配、智能体间通信等问题。但这类系统普遍存在稳定性差、容易陷入死循环、任务完成率低等问题，难以应用到生产环境。

### 2.3 第三代：工业级Harness（2026年至今）

2026年成为Harness技术的成熟元年，OpenAI和Anthropic几乎同时公布了经过大规模生产验证的Harness架构：

  * **OpenAI的Harness Engineering** ：提出”智能体优先”的软件工程范式，人类工程师不再直接编写代码，而是设计环境、明确目标、定义约束，让智能体在Harness框架内完成开发工作。OpenAI内部已经用这套方法构建了超过100万行代码，且完全没有人工编写。
  * **Anthropic的Harness Design** ：公开了长运行应用开发的Harness设计方案，通过生成器-评估器的对抗迭代模式，让Claude可以连续跑6小时完成全栈应用开发，代码质量远超人类工程师平均水平。
  * **Claude Code源码泄露** ：51万行TypeScript源码意外曝光，让我们看到了工业级Harness的完整实现，包括状态管理、上下文调度、错误恢复、安全防护等各个层面的工程细节。



* * *

## 三、Harness的核心设计思想

无论是OpenAI还是Anthropic的Harness架构，都遵循几个共同的核心设计原则：

### 3.1 对抗式迭代（GAN-inspired Pattern）

这是Harness最核心的设计理念，从GAN（[生成对抗网络](https://zhida.zhihu.com/search?content_id=272437256&content_type=Article&match_order=1&q=%E7%94%9F%E6%88%90%E5%AF%B9%E6%8A%97%E7%BD%91%E7%BB%9C&zhida_source=entity)）借鉴而来：

  * **生成器（Generator）** ：负责具体的任务执行（写代码、写文档、设计方案等）。
  * **评估器（Evaluator）** ：负责对生成器的输出进行严格评审，发现问题、提出改进建议。
  * **迭代循环** ：两者反复交互，直到输出达到预设的质量标准。



这种模式完美映射了人类的软件开发流程：开发者写代码 → Code Review → 测试 → 修复问题 → 上线，而Harness把这个流程完全自动化了。

### 3.2 角色专业化分工

Harness中的每个智能体都有明确的角色定位，就像人类团队中的不同岗位：

  * **Initializer Agent** ：负责任务初始化，拆解需求、制定计划、配置环境。
  * **Coding Agent** ：专注于代码编写和功能实现。
  * **Review Agent** ：专门做代码评审，检查质量、安全、性能等问题。
  * **Test Agent** ：负责编写测试用例、执行测试、验证功能正确性。
  * **Debug Agent** ：专门处理bug定位和修复。



专业化分工带来的是能力的深度优化，每个智能体只需要专注于特定领域，比通用智能体的效率和质量高出数倍。

### 3.3 会话间上下文管理

长任务运行的最大挑战是上下文的一致性管理。Anthropic的Harness设计中采用了”[上下文重置](https://zhida.zhihu.com/search?content_id=272437256&content_type=Article&match_order=1&q=%E4%B8%8A%E4%B8%8B%E6%96%87%E9%87%8D%E7%BD%AE&zhida_source=entity)”机制：

  * 每个小迭代只保留必要的上下文信息。
  * 迭代之间通过状态存储共享关键信息。
  * 避免上下文过长导致的注意力分散和幻觉问题。



这种设计让Claude可以连续运行6小时而不偏离任务目标，这在传统的单会话模式下是完全不可能的。

### 3.4 [可观测性与可干预性](https://zhida.zhihu.com/search?content_id=272437256&content_type=Article&match_order=1&q=%E5%8F%AF%E8%A7%82%E6%B5%8B%E6%80%A7%E4%B8%8E%E5%8F%AF%E5%B9%B2%E9%A2%84%E6%80%A7&zhida_source=entity)

工业级Harness必须具备完善的可观测能力：

  * 全流程日志记录，每个智能体的每一步操作都可追溯。
  * 实时状态监控，及时发现死循环、性能瓶颈等问题。
  * 人工干预入口，在关键节点可以由人类介入调整方向。
  * 错误恢复机制，出现问题时可以自动回滚到上一个正确状态。



* * *

## 四、Harness带来的行业变革

Harness技术的成熟，将对整个AI行业产生深远的影响：

  1. **生产力的十倍提升**  
一个设计良好的Harness，可以让AI开发效率提升10倍以上。以前需要一个团队几周完成的开发任务，现在只需要一个工程师配合Harness系统几天就能完成，而且质量更高。
  2. **工程师角色的重构**  
未来的软件工程师不再是代码的编写者，而是Harness的设计者和智能体的管理者。工程师的核心能力将从”写代码”转变为”设计任务、定义约束、评估结果”。
  3. **软件开发流程的重构**  
传统的`需求分析→设计→开发→测试→上线`的线性流程，将被Harness驱动的迭代式开发所取代。整个开发过程变成一个持续优化的闭环，交付速度和质量都会得到质的提升。
  4. **AI应用的门槛大幅降低**  
有了成熟的Harness框架，中小企业不再需要自己研究复杂的多智能体技术，只需要基于开源Harness框架进行简单配置，就能开发出复杂的AI应用。



* * *

### 本系列博客预告

本系列将分为3篇，全面深入解析Harness技术：

  1. **第一篇（本篇）** ：AI智能体的”脚手架”革命——从OpenAI到Anthropic的工程范式演进
  2. **第二篇** ：Anthropic Harness架构深度拆解——让Claude连跑6小时的技术秘密
  3. **第三篇** ：OpenAI Harness Engineering方法论解析——如何用智能体构建百万行代码系统



下一篇我们将深入Anthropic公开的Harness设计文档，拆解其核心架构和工作原理，欢迎持续关注。

* * *

### 相关链接

  1. Anthropic官方博客：[Harness design for long-running application development](https://link.zhihu.com/?target=https%3A//www.anthropic.com/engineering/harness-design-long-running-apps)
  2. OpenAI官方博客：[Harness engineering: leveraging Codex in an agent-first world](https://link.zhihu.com/?target=https%3A//openai.com/index/harness-engineering/)
  3. [最会做Harness的Anthropic下场教学了!!深度解读Harness Design](https://zhuanlan.zhihu.com/p/2020070756931880515)
  4. [Claude Code 51万行源码意外泄露：一次.map文件事故背后的AI工程启示录](https://link.zhihu.com/?target=https%3A//segmentfault.com/a/1190000047688721)
  5. [OpenAI提出Harness Engineering：Codex智能体驱动大规模软件开发](https://link.zhihu.com/?target=https%3A//www.infoq.cn/article/MCUXGhyIRqPLkFhljY9v)


