# 【记忆】05-SRMT：面向多智能体终身路径规划的共享内存技术

原文链接：https://zhuanlan.zhihu.com/p/1920569731095696375

---

​

目录

## 1 概述

### 1.1 背景介绍

[多智能体系统](https://zhida.zhihu.com/search?content_id=259457702&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E7%B3%BB%E7%BB%9F&zhida_source=entity)在分布式智能和协作方面具有巨大潜力，但协调多个智能体之间的交互仍然是一个挑战。在[多智能体路径规划](https://zhida.zhihu.com/search?content_id=259457702&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E8%B7%AF%E5%BE%84%E8%A7%84%E5%88%92&zhida_source=entity)（MAPF）中，每个智能体的目标是到达其目标位置，同时只能局部观察环境状态，包括其他智能体的位置和动作。现有的方法通常需要复杂的通信协议和决策机制，但这些方法在面对稀疏奖励和未见过的地图时往往表现不佳。

### 1.2 主要解决的问题

论文主要解决的问题是**多智能体强化学习（MARL）中智能体之间的协调问题** ，特别是在多智能体路径规划（MAPF）任务中，如何在没有明确通信协议的情况下实现智能体之间的有效协作，具体如下：

  * 在多智能体环境中，每个智能体需要独立地做出决策以完成任务，但同时又需要与其他智能体协调行动以避免冲突（如碰撞）。
  * 传统的MARL方法要么依赖于复杂的通信协议，要么在没有通信的情况下难以有效协调，显式地预测其他智能体的行为对于实现合作至关重要，但是当前没有很有效的解决方案。
  * 传统的MARL方法往往依赖于复杂的通信协议或中央控制器，难以扩展到大规模、去中心化的环境中。
  * 在部分可观测的多智能体路径规划（PO-MAPF）任务中，智能体只能局部观察环境状态，这进一步增加了协调的难度。



### 1.3 相关工作

**共享内存与多智能体强化学习中的通信**

  * **集中式方法**
    * 概念：有一个中央控制器收集所有智能体的信息并做出决策，例如LaCAM和RHCR。
    * 局限性：依赖于中央控制器，难以扩展到大规模环境，同时缺乏灵活性。
  * **完全去中心化方法**
    * 概念：智能体仅根据局部观察做出决策，例如IQL、VDN、QMIX、QPLEX、Follower和MATS-LP。
    * 局限性：不支持智能体之间的直接通信，导致在复杂环境中容易出现死锁和冲突。同时在稀疏奖励环境或未训练过的环境中表现较差，泛化性较弱。
  * **去中心化且允许通信的方法**
    * 概念：智能体之间可以共享局部信息，例如DCC、MAMBA和SCRIMP。
    * 局限性：通信机制增加了计算和通信开销，可能导致系统性能下降；通信策略复杂且可能导致死锁。



这些方法在一定程度上提高了智能体之间的协调能力，但是也各自存在一些局限性。SRMT通过共享[循环记忆](https://zhida.zhihu.com/search?content_id=259457702&content_type=Article&match_order=1&q=%E5%BE%AA%E7%8E%AF%E8%AE%B0%E5%BF%86&zhida_source=entity)提供了一种新的隐式信息交换方式，避免了这些局限性。

**[Memory Transformer](https://zhida.zhihu.com/search?content_id=259457702&content_type=Article&match_order=1&q=Memory+Transformer&zhida_source=entity)**

  * **Memory Transformer (MT)** ：在Transformer 中引入记忆token，为模型提供了额外的操作空间。
  * **Recurrent Memory Transformer (RMT)** ：通过循环更新记忆token，使Transformer能够处理长序列输入。
  * **Agent Transformer Memory (ATM)** ：每个智能体维护自己的记忆缓冲区，用于存储过去N个记忆状态。
  * **Recurrent Action Transformer with Memory (RATE)** ：通过记忆保留阀门更新记忆状态，有效地处理长分割轨迹。
  * **Relational Recurrent Neural Network (RRNN)** ：使用多头点积注意力来更新记忆状态。



ATM、RATE和RRNN专注于为每个智能体维护单独的记忆状态，没有考虑智能体之间的信息共享。与这些方法不同，SRMT 将循环记忆扩展到多智能体强化学习，并通过共享内存促进智能体之间的协调。

## 2 本文方案与关键技术简介

论文提出的**共享循环记忆Transformer（SRMT）是一种创新的多智能体路径规划方法，其核心思路是通过引入共享记忆机制** ，使智能体能够在没有显式通信的情况下隐式地交换信息，从而提高智能体之间的协调能力。

### **2.1 核心思想**

将「全局工作空间理论」与「Memory Transformer」应用于多智能体系统，即让智能体通过共享的内存空间进行隐式信息交互，实现更好的协调。

  * 全局工作空间理论：大脑中不同的功能模块通过一个全局工作空间进行信息广播和协作。
  * Memory Transformer：通过记忆token为Transformer模型提供额外的操作空间和长期记忆能力。



### **2.2 基础流程**

  * 每个智能体维护自己的循环记忆 (memory vector)。
  * 在每个时间步，所有智能体的循环记忆被合并到一个共享内存空间中。
  * 每个智能体通过交叉注意力机制读取共享内存，并将全局信息融入到自己的决策中。
  * 每个智能体使用一个记忆头（memory head）来更新自己的循环记忆，为下一步做准备。
  * 整个过程是去中心化的，智能体独立进行决策，不依赖于中心控制器。



### **2.3 SRMT架构解析**

SRMT架构示意图

**模型结构**

    * **空间编码器** ：使用ResNet和MLP处理智能体的局部观测信息。
    * **循环记忆模块**
      * 每个智能体维护自己的循环记忆向量。
      * 在每个时间步，所有智能体的记忆向量被合并，形成一个共享内存。
      * 智能体通过交叉注意力机制读取共享内存中的信息。
      * 智能体使用记忆头更新自己的循环记忆向量。
    * **动作解码器** ：根据智能体的隐藏状态生成动作。
    * **评论家头 (Critic Head)** ：用于评估当前状态的价值。



**关键技术**

    * **交叉注意力机制** ：允许智能体从共享内存中获取全局信息，实现隐式信息交换。
    * **循环记忆** ：使智能体能够存储和利用历史信息，提高决策的连贯性。
    * **去中心化训练** ：每个智能体独立训练，不依赖于中央控制器。



**使用的工具**

    * [POGEMA](https://zhida.zhihu.com/search?content_id=253094970&content_type=Article&match_order=1&q=POGEMA&zhida_source=entity)：一个用于多智能体路径规划的基准平台。
    * [Sample Factory](https://zhida.zhihu.com/search?content_id=253094970&content_type=Article&match_order=1&q=Sample+Factory&zhida_source=entity)：一个用于强化学习的工具库。
    * [Huggingface GPT-2](https://zhida.zhihu.com/search?content_id=253094970&content_type=Article&match_order=1&q=Huggingface+GPT-2&zhida_source=entity)：用于实现注意力机制。



## 3 实验与数据

### **3.1 实验设置**

**实验环境**

    * **瓶颈环境** ：一个简单的两智能体协调任务，要求智能体通过一个狭窄的通道，用于测试SRMT的基本性能和泛化能力。
    * **[POGEMA 基准](https://zhida.zhihu.com/search?content_id=259457702&content_type=Article&match_order=1&q=POGEMA+%E5%9F%BA%E5%87%86&zhida_source=entity)** ：二维环境被表示为一个由障碍物和空闲单元格组成的网格，包括迷宫 (Maze)、随机 (Random)、仓库 (Warehouse) 和MovingAI等多种地图，用于评估SRMT在复杂环境中的性能。



**实验设置**

    * **奖励函数** ：使用了方向性奖励 (Directional)、移动负奖励 (Moving Negative) 和稀疏奖励 (Sparse) 三种奖励函数，以评估SRMT在不同奖励机制下的性能。 
      * 方向性奖励 (Directional)：智能体在到达目标以及每一步向目标靠近时都会获得奖励。
      * 移动负奖励 (Moving Negative)：移动会被轻微惩罚以最小化通往目标的路径，但不会提供关于目标位置的信息。
      * 稀疏奖励 (Sparse)：只在智能体成功达到目标单元格时给予奖励，不提供任何中间奖励。
    * **对比方法** ：与MAMBA、QPLEX、ATM、RATE、RRNN等多种MARL方法进行了对比，同时进行了RMT、Attention、RNN等消融实验。
    * **评估指标** ： 
      * **合作成功率 (CSR)** ：所有智能体是否在规定时间内到达目标。
      * **个体成功率 (ISR)** ：到达目标的智能体比例。
      * **总成本 (SoC)** ：所有智能体到达目标所花费的总时间步数。
      * **平均吞吐量** ：在终身 MAPF 中，所有智能体在每个时间步达到的平均目标数。
      * **POGEMA 基准指标** ：包括性能 (Performance)、路径规划 (Pathfinding)、拥堵 (Congestion)、合作 (Cooperation)、分布外泛化 (Out-of-Distribution) 和可扩展性 (Scalability)。



### 3.2 实验结果 

**基于瓶颈任务的经典多智能体路径规划**

SRMT 在三种奖励函数下都优于其他方法，尤其是在稀疏奖励下表现突出，并展现了良好的泛化能力。  


  
SRMT在所考虑的奖励函数下成功解决了任务。特别是，SRMT在其他方法难以应对的具有挑战性的移动负值和稀疏奖励场景中始终表现良好。通过共享内存协调智能体证明至关重要，尤其是在环境反馈极少的情况下。  
为了进一步评估训练好的策略的泛化能力，论文在走廊长度显著大于训练时使用的长度的瓶颈环境中评估了它们，范围从5到1000个单元格。  


  
SRMT 智能体在长度高达 1000 的走廊上泛化。在训练了从 3 到 30 个单元的走廊尺寸后，所有方法都在更长的通道上进行了评估，长度高达 1000。所有非零性能的模型都显示出良好的扩展性，直到走廊长度为 100。对于稀疏奖励，SRMT 最高达到 400，然后低于RMT的集体性能。对于移动负奖励，SRMT 在所有三个指标上都显示出第一名的性能。阴影区域表示 95%的置信区间。  
  
在 POGEMA 基准地图中，SRMT 在迷宫、随机和 MovingAI 地图中都优于其他 MARL 方法，但在仓库环境中，SRMT 结合启发式路径规划方法 (SRMT-FlwrPlan) 后，才能与规划方法 RHCR 竞争。  


**终身多智能体路径规划（LMAPF）任务**

终身多智能体路径规划（LMAPF）是 MAPF 的扩展，其中智能体在完成当前目标后接收新的目的地。这个任务设置的主要质量指标是平均吞吐量，计算为每个回合步骤中所有智能体达到的目标数量的平均值。  


在不同环境中，SRMT 比其他多智能体强化学习方法表现更优。在迷宫环境中训练的 SRMT 在未见过训练地图的评估中表现出强大的泛化能力。除了仓库环境外，SRMT 在所有地图上都优于 MARL 基线方法 MAMBA 和 QPLEX。混合训练使用 64 或 128 个智能体（SRMT 64-128）不会影响该方法的泛化能力。在仓库环境中，基于跟随者启发式路径搜索的奖励函数的 SRMT（SRMT-FlwrPlan）的平均吞吐量超过了 MAMBA、MATS-LP、QPLEX 和 RHCR 方法。误差线表示 95%置信区间。

  * POGEMA 基准测试中的高级 MAPF 指标 
    * Performance：显示了每种方法的平均吞吐量与所考虑方法中的最佳值之间的关系。该指标在 Random（带有随机放置障碍物的 20×20 网格）和 Mazes（大小为 21×21 的迷宫状网格）地图集上获得的结果上进行平均计算。
    * Pathfinding：一个二进制值，它展示了单个智能体在大型地图上的路径是否最优。
    * Congestion：每个智能体在观察到的智能体密度与仓库地图上智能体的整体密度相比的平均值。
    * Cooperation：解决复杂情况的能力。
    * Out-of-Distribution：训练期间未出现的地图上的性能指标值。
    * Scalability：描述了每种方法的运行时间随着仓库地图上智能体数量的增加而变化的情况。  




  
在多智能体路径规划中的关键性能指标上，SRMT 与其他方法的比较。条形图比较了 SRMT 及其变体（SRMT 64-128、SRMT-FlwrPlan）与其他方法——MAMBA、QPLEX、Follower、MATS-LP 和 RHCR——在六个指标上的性能：性能、路径规划、拥堵、合作、分布外和可扩展性。SRMT 及其变体表现出具有竞争力的性能，特别是在可扩展性和路径规划方面。当与 Follower 规划集成时，SRMT 在拥堵管理方面表现最佳。集中式规划方法 RHCR 在多个指标上领先，特别是在合作、分布外、性能和路径规划方面，达到近 100%。MAMBA 在拥堵管理和可扩展性方面表现出强大的性能。

## 4 总结

本文提出来一种新颖的基于共享循环记忆的Transformer架构SRMT，可以用于提高多智能体系统的协调能力。

  * **验证了共享循环记忆的有效性** ：实验结果表明，通过共享循环记忆，智能体可以隐式地交换信息，从而实现更好的决策。
  * **展示了 SRMT 的泛化能力和可扩展性** ：SRMT 在多种不同的环境中都表现良好，并且可以应用于大规模的多智能体系统。
  * **提供了对智能体记忆表示的分析** ：分析结果表明，SRMT 学习到的记忆表示与智能体之间的空间关系对齐，这为理解 SRMT 的工作原理提供了新的视角。



## 相关链接

  1. [https://arxiv.org/abs/2501.13200](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2501.13200)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【记忆】04-Letta，为AI智能体解锁记忆潜能的开发框架](https://zhuanlan.zhihu.com/p/29629889237)
