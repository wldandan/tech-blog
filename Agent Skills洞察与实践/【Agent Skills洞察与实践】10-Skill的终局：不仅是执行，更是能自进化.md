# 【Skills】08-Skill的终局：不仅是执行，更是能自进化

原文链接：https://zhuanlan.zhihu.com/p/2019809405609230673

---

​

目录

随着Agent技术的快速发展，Skill作为行业与领域知识的沉淀，正在被前所未有的速度生产出来。据统计，在主流开源社区与各类平台生态中，Skill规模已达到数十万级（且仍在快速增长）。这是一件好事，但也带来了一个潜在的问题：

**Skill的生产问题正在被解决，而Skill的“失控问题”正在浮现。**

在多个真实项目中，我们反复看到类似的现象：Skill数量快速膨胀，但命中率却在下降；任务可以完成，但执行路径不可解释；优化在持续进行，但效果却越来越不稳定。表面上看，系统在“运转”，本质上却在积累技术债。 业界以[skill-creator](https://zhida.zhihu.com/search?content_id=271939706&content_type=Article&match_order=1&q=skill-creator&zhida_source=entity)为代表的工具，让“生产Skill”这件事变得高效、可规模化，甚至开始具备评测、A/B测试与优化等能力，但：

  * 缺乏去重和归纳能力，相似Skill越积越多
  * 评测只看结果不看过程，结果无法追溯
  * 优化缺乏执行数据支撑，难以持续改进



这也是本文想要讨论的核心：**在skill-creator已经进化的背景下，我们为什么还需要[Skill-insight](https://zhida.zhihu.com/search?content_id=271939706&content_type=Article&match_order=1&q=Skill-insight&zhida_source=entity)？**

答案不是“再造一个工具”，而在于补上一个关键能力——让Skill能够在真实使用中通过数据驱动的闭环实现持续进化。

## 从“功能覆盖”到“闭环优化”：重心正在转移

必须承认，skill-creator的能力已经显著进化，从单一生成工具发展为包含生成、评测、A/B测试与优化在内的工具链。从功能上看，似乎已经接近一个完整闭环。 但在真实工程环境中，问题不在于“有没有这些能力”，而在于这些能力是否围绕同一套执行数据打通。当前体系更像是“功能串联”，而不是“[数据驱动闭环](https://zhida.zhihu.com/search?content_id=271939706&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E9%A9%B1%E5%8A%A8%E9%97%AD%E7%8E%AF&zhida_source=entity)”：生成、评测与优化彼此连接，却没有共享统一的执行语义与数据视图。 这种差异在规模化场景中会迅速放大，主要体现在三个方面：

  * **规模失控** ：缺乏去冗余与模式抽取，相似问题不断生成新的Skill，导致数量与噪声同步增长
  * **评测失真** ：评测仍以结果为中心，无法判断执行路径是否正确或存在风险
  * **优化失效** ：缺乏过程数据支撑，优化依赖结果反馈，难以形成稳定工程方法



本质上，这三个问题指向同一个根因：**缺少以执行过程数据为核心的闭环能力。**

当Skill规模进入企业级后，这种缺失会直接导致系统能力“表面增长、实质下降”：Skill越来越多，但召回变差、成本上升、优化失去方向。 这正是Skill-insight切入的核心问题。

## Skill-insight：让Skill在使用中自我进化

Skill-insight的设计，并不是替代已有工具，而是将它们向前推进一层：从“功能工具链”升级为“数据驱动闭环”。 其核心思路可以概括为一句话：

**让执行过程数据成为Skill演进的核心输入。**

围绕这一点，它构建了三项相互联动的能力。

  1. 首先，是对**Skill规模的重构** 。**通过去冗余、语义聚合与模式抽取，将大量相似Skill压缩为少量具备泛化能力的模式化表达** 。这样，系统面对的就不再是“成百上千的具体解法”，而是“若干稳定的问题模型”。规模下降的同时，召回精度与成本控制能力反而提升。
  2. 其次，是**评测方式的改变** 。系统不再只记录结果，而是对执行过程进行完整追踪：每一步调用、每一次推理、每一段路径选择都会被记录下来，并与预定义流程进行对比。**评测因此从“结果判断”，升级为“结果 + 路径 + 成本”的综合分析** 。 更重要的是，这种评测是可解释的。系统可以明确指出偏差发生在哪一步，并辅助判断是模型推理问题，还是Skill设计问题。 
  3. 最后，是**优化方式的转变** 。当执行过程被结构化记录之后，优化就不再依赖猜测。系统可以基于真实数据定位瓶颈步骤、识别高风险路径，并对Skill结构进行针对性调整。这使得优化从“基于结果的试错”，变成“基于数据的工程过程”。 当这三部分能力被同一套数据串联起来时，就形成了一个真正意义上的闭环：**生成 → 执行 → 评测 → 归因 → 优化。**



Skill不再是静态资产，而成为一个可以在使用中持续演进的系统。

## 案例1：[磁盘故障诊断](https://zhida.zhihu.com/search?content_id=271939706&content_type=Article&match_order=1&q=%E7%A3%81%E7%9B%98%E6%95%85%E9%9A%9C%E8%AF%8A%E6%96%AD&zhida_source=entity)场景，使用Skill-insight有效减少相似Skill数量，显著提升召回率并降低Token消耗

**背景** ：某企业在一个磁盘故障诊断的场景中，由于需要基于大量日志文件进行分析，团队面临着处理效率不高，且存在一定操作风险的问题。 最初，团队使用现有工具生成了一批排障Skill，系统可以完成问题定位，但很快暴露出两个问题：

  * Skill数量多且相似，召回效果不稳定；
  * 执行过程不可见，难以判断是否存在风险操作。



引入Skill-insight后，系统首先对历史案例进行聚合，将数十个相似问题收敛为少量模式化流程，例如资源检查、内核参数分析与日志诊断等标准步骤，形成若干Skill。最终与skill-creator生成的Skill在相同任务上进行对比，结果如下：

工具名称| Token 消耗| Skill 召回率  
---|---|---  
skill-creator| 460k| 20%  
Skill-insight| 355k| 100%  
  
可以看到，经过Skill-insight的模式化抽取后，可以显著降低任务执行的Token消耗，以及显著提升Skill召回率。

## 案例2：[应用卡顿故障诊断](https://zhida.zhihu.com/search?content_id=271939706&content_type=Article&match_order=1&q=%E5%BA%94%E7%94%A8%E5%8D%A1%E9%A1%BF%E6%95%85%E9%9A%9C%E8%AF%8A%E6%96%AD&zhida_source=entity)场景，使用Skill-insight进行优化，自动添加关键备份与回滚步骤

**背景** ：某企业在生产环境中频繁出现 Docker 应用卡顿问题，历史上沉淀了大量故障排除方法文档，但人工处理问题效率低，希望将这些文档固化为Skill。 团队使用现有工具生成了一版Skill，并想在使用的过程中持续的发现问题并优化，但是却遇到一些问题：**没有优化方向** ，Agent使用Skill时的执行流程是什么样的？与预期有什么偏离？这些信息一概不知。 

引入Skill-insight后，系统对Agent执行过程与Skill定义的流程进行可视化呈现与对比，直观展示预期之外的Agent行为，并给出分析、优化建议。最终与skill-creator的评测以及优化能力在相同任务上进行对比，结果如下：

Skill-insight的Skill信息图：

Skill-insight的Skill有效性分析：

skill-creator的评测结果：

正是由于Skill-insight引入了执行过程数据和缺陷原因分析，为优化提供了更多的优化依据，据此进行的优化结果如下： 优化前的Skill.md：
    
    
    ### 步骤 2：检查并修复 kernel.printk 配置
    **目标**：修改 `kernel.printk` 内核参数，以规避已知的内核死锁路径。
    1.  **执行检查**：查看当前 `/etc/sysctl.conf` 文件中的 `kernel.printk` 配置。
        ```bash
        grep \"kernel.printk\" /etc/sysctl.conf
        ```

优化后的Skill.md：
    
    
    ### 步骤 2：检查并修复 kernel.printk 配置
    **目标**：修改 `kernel.printk` 内核参数，以规避已知的内核死锁路径。
    1.  **执行检查与备份**：查看当前 `/etc/sysctl.conf` 文件中的 `kernel.printk` 配置，并备份原始值。
        ```bash
        # 检查当前配置
        grep "kernel.printk" /etc/sysctl.conf
        # 备份当前运行时参数（用于可能的回滚）
        CURRENT_PRINTK=<!--MATH_PH_1-->CURRENT_PRINTK"
        echo "如需回滚，可执行: sysctl -w kernel.printk=\"$CURRENT_PRINTK\""
        ```

## 结语：Skill的终局，不是被生成，而是能够自进化

skill-creator开启了Skill规模化生产的时代，但当系统进入生产环境后，真正决定上限的不再是”能不能生成”，而是能否控制规模、解释行为、持续优化——这些能力的前提是拥有完整可用的执行过程数据。**Skill-insight正是将这部分数据从”不可见”变为”可用”，驱动Skill从静态工具进化为可持续学习的系统** 。Agent能否真正走向生产，分水岭不在于它”会不会做”，而在于它能不能在不断使用中，变得更好。

  


如果你正在构建各类Agent、优化Skill，探索AI在各行业中的落地： 欢迎加入 [openEuler](https://zhida.zhihu.com/search?content_id=271939706&content_type=Article&match_order=1&q=openEuler&zhida_source=entity) 社区，一起让Agent真正落地，降低执行成本、提升执行可靠性。

👉 项目地址：[https://atomgit.com/openeuler/witty-skill-insight](https://link.zhihu.com/?target=https%3A//atomgit.com/openeuler/witty-skill-insight)

  


> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Skills】07-告别Skill’盲盒’！Skill-insight三招让Agent精准可迭代](https://zhuanlan.zhihu.com/p/2019459824727897901)  
> 下一篇：[【Skills】09-SkillProbe：用“魔法”打败“魔法”——用Skill审计Skills](https://zhuanlan.zhihu.com/p/2020165820400054775)
