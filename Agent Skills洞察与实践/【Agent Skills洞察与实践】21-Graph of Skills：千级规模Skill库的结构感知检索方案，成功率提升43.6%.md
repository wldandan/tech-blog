# 【Skills】19-Graph of Skills：千级规模Skill库的结构感知检索方案，成功率提升43.6%

原文链接：https://zhuanlan.zhihu.com/p/2028146364282880663

---

​

目录

## 一、海量Skill时代：Agent面临的两大核心痛点

在Agent系统中，调用外部工具、API和可复用Skill已经成为完成复杂任务的标配能力。但当企业级Skill库规模从几十个快速扩张到上千甚至上万个时，Skill检索已经从原来的边缘辅助环节，变成了直接制约Agent任务成功率和运行效率的核心瓶颈。目前行业主流的两种Skill检索方案都存在明显缺陷：

  1. **[全量Skill加载](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=%E5%85%A8%E9%87%8FSkill%E5%8A%A0%E8%BD%BD&zhida_source=entity) （Vanilla Skills）**：直接把所有Skill描述塞进上下文窗口，不仅token成本随Skill库规模线性增长，还会导致模型淹没在无关信息中，关键Skill的使用约束被忽略；
  2. **[向量检索](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=%E5%90%91%E9%87%8F%E6%A3%80%E7%B4%A2&zhida_source=entity) （Vector Skills）**：虽然能有效控制返回的Skill数量，但仅基于语义相似度匹配，无法识别Skill之间的依赖关系，经常出现“检索到了核心功能Skill，但缺少前置的解析、转换、环境配置等依赖Skill”的问题，最终任务仍然无法跑通。



图1 全量Skill加载、向量检索与Skill图谱的对比

简单来说：全量加载太 “重”，纯语义检索太 “偏”，二者都无法兼顾相关性、执行完整性、上下文效率三大目标。针对这一问题，来自宾夕法尼亚大学、马里兰大学等机构的研究团队在论文《[Graph of Skills](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=Graph+of+Skills&zhida_source=entity): Dependency-Aware Structural Retrieval for Massive Agent Skills 》中提出了**Graph of Skills (GoS)** 框架，这是一套**推理时结构感知检索** 系统，通过构建**Skill依赖图** 的方式，一次性返回完整的可执行Skill包，同时兼顾token效率和任务成功率。

## 二、核心设计：用有向依赖图解决Skill检索的依赖缺失问题

GoS的核心设计思路是把所有**Skill及其依赖关系组织成一张结构化的有向图** ，检索时不仅返回语义相关的Skill，还会自动补全所有需要的前置依赖，形成完整的可执行Skill包。整套体系分为**离线建图** 和**在线检索** 两个核心阶段，全程不需要人工标注依赖关系。

图2 GoS架构

### 2.1 离线建图：自动生成结构化的Skill依赖网络

离线阶段，GoS会把所有本地Skill包解析成标准化的Skill节点，每个节点包含Skill名称、能力描述、输入输出格式、依赖工具、示例场景、脚本路径等结构化字段。解析过程以确定性规则为主，只有当Skill文档不完整时，才会用轻量大模型补全检索需要的核心字段，避免语义幻觉。

完成节点标准化后，系统会自动生成四种类型的边，构建完整的**有向类型Skill图** ：

  1. **依赖边** ：如果ASkill的输出格式正好匹配BSkill的输入要求，就生成一条A→B的依赖边，表示A是B的前置依赖，这是图中最核心的边类型；
  2. **工作流边** ：表示两个Skill经常在同一个业务流程中被先后使用，比如“数据提取”和“数据清洗”；
  3. **语义边** ：表示两个Skill的能力高度相似，属于同一个功能领域；
  4. **替代边** ：表示两个Skill可以互相替换实现同一个功能，比如不同厂商的OCR识别Skill。



为了控制图的规模和精度，非依赖边的生成采用稀疏验证机制：先通过词汇相似度、语义邻居、输入输出扩展生成候选池，再用大模型仅验证候选池内的边关系，避免全量计算的成本问题，同时保证边的准确率。

### 2.2 在线检索：[反向扩散算法](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=%E5%8F%8D%E5%90%91%E6%89%A9%E6%95%A3%E7%AE%97%E6%B3%95&zhida_source=entity)生成完整的可执行Skill包

在线检索阶段，GoS没有采用传统的top-k语义匹配，而是通过三层逻辑生成完整的Skill包：

  1. **混合语义-词汇种子检索** ：先把用户查询重写成包含目标、操作、依赖、关键词的结构化检索 schema，同时用语义向量和词汇匹配两种方式找到最相关的核心Skill作为种子，避免单一检索方式的漏召问题；
  2. **反向加权个性化PageRank** ：以种子Skill为起点，沿着依赖边反向扩散评分，自动找到所有需要的前置依赖Skill。不同类型的边有不同的反向权重：依赖边权重最高1.0，工作流边0.5，语义边0.2，替代边0.1，保证核心依赖优先被召回；
  3. **预算重排与水化** ：把扩散得到的候选Skill结合查询匹配度重新排序，按照上下文窗口的预算限制，只返回最相关的Skill组合，每个Skill都附带可直接调用的本地路径和执行说明，Agent拿到后可以直接使用，不需要额外解析。



和传统检索方案最大的区别是：GoS返回的不是孤立的Skill列表，而是完整的可执行Skill组合，Agent不需要再额外寻找依赖，也不需要处理Skill之间的兼容性问题，大幅降低了推理阶段的复杂度。

## 三、实验效果：成功率提升43.6%，token消耗减少37.8%

研究团队在两个主流基准测试集（SkillsBench，1000个技能，11个技术领域；[ALFWorld](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=ALFWorld&zhida_source=entity)，140个家庭任务）上验证了GoS的效果，覆盖[Claude Sonnet 4.5](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=Claude+Sonnet+4.5&zhida_source=entity)、MiniMax M2.7、[GPT-5.2 Codex](https://zhida.zhihu.com/search?content_id=273225970&content_type=Article&match_order=1&q=GPT-5.2+Codex&zhida_source=entity)三个主流大模型，实验结果已经被行业从业者用公开数据集复现验证，真实可信。

### 核心指标表现

  1. 相比Vanilla Skills：平均奖励提升**43.6%** ，输入Token减少**37.8%** ，5/6组实验中运行时间更短。
  2. 相比Vector Skills：SkillsBench奖励提升**10.97%** ，ALFWorld提升**2.87%** ，同时保持同等Token效率。
  3. 规模鲁棒性：在200-2000个技能的范围内，GoS始终优于基线，且技能库越大，优势越明显。



## 四、现存局限性与未来发展方向

GoS并非完美的解决方案，目前仍存在几个需要改进的局限性：

  1. **复杂开放场景效果不佳** ：目前提取的Skill主要针对工具使用类场景，对于特别复杂的开放场景、需要强推理的任务，Skill提取和依赖识别的效果仍不理想；
  2. **跨领域迁移能力有限** ：在A领域提取的Skill依赖关系，迁移到差异较大的B领域时，准确率会有明显下降，需要额外的适配工作；
  3. **Skill组合智能化程度不足** ：目前多Skill的执行流程仍然需要Agent自主判断，还不能自动生成最优的执行方案；
  4. **安全风险仍然存在** ：自动提取的Skill可能隐藏恶意指令或安全漏洞，需要额外的安全审计流程。



从行业发展趋势来看，未来AgentSkill生态会向三个方向发展：

  * 将会出现跨平台、跨Agent的通用Skill市场，开发者可以共享和复用Skill，不需要每个公司都从零开始建设；
  * Skill会像现在的软件包一样，拥有成熟的版本管理、依赖管理、安全审计体系，使用起来会像npm安装包一样简单；
  * Skill会实现自动进化，使用人数越多、覆盖场景越广，效果就会越好，形成正向循环。



## 五、总结

GoS的出现填补了大规模Skill库结构感知检索的技术空白，解决了Agent Skill库带来的核心可扩展性问题。通过显式建模Skill依赖并使用结构化检索，它相比两种传统方案同时实现了更高的任务成功率和更低的Token成本。随着Agent体系统能力不断提升，承担的任务越来越复杂，Skill图谱这类方案将成为充分释放大规模技能生态价值的关键技术。

## 相关链接

  1. [https://arxiv.org/pdf/2604.05333](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2604.05333)
  2. [https://github.com/davidliuk/graph-of-skills](https://link.zhihu.com/?target=https%3A//github.com/davidliuk/graph-of-skills)


