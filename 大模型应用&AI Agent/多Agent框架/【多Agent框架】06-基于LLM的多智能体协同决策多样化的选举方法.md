# 【多Agent框架】06-基于LLM的多智能体协同决策多样化的选举方法

原文链接：https://zhuanlan.zhihu.com/p/11075409117

---

​

目录

## 论文摘要

现代[大语言模型](https://zhida.zhihu.com/search?content_id=251237851&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）在解决复杂任务和CDM（Collaborative Decision Making，协同决策）方面表现出协同效应，CDM是LLM驱动的多智能体（multi-agent）协作系统的核心部分。本文作者通过对近期相关的52个[多智能体协作系统](https://zhida.zhihu.com/search?content_id=251237851&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E5%8D%8F%E4%BD%9C%E7%B3%BB%E7%BB%9F&zhida_source=entity)的调查发现其CDM依赖的方法缺乏多样性且过分依赖“独裁投票法”和“相对多数投票法”。因此，本文作者首先通过[社会选择理论](https://zhida.zhihu.com/search?content_id=251237851&content_type=Article&match_order=1&q=%E7%A4%BE%E4%BC%9A%E9%80%89%E6%8B%A9%E7%90%86%E8%AE%BA&zhida_source=entity)的视角指出了常用CDM方法的局限性，并提出了**一个整合了多种序数偏好投票法的选举模块——[GEDI](https://zhida.zhihu.com/search?content_id=251237851&content_type=Article&match_order=1&q=GEDI&zhida_source=entity)，并基于该模块研究了CDM方法的多样性对多智能体协作系统的影响。**研究结果表明：某些CDM方法的整合可以显著提高一些LLM的推理能力和鲁棒性，且不需要复杂的系统设计。此外，即使只有三个Agent，一些CDM方法也能产生积极的协同效应。

## 背景&问题

随着大语言模型（LLM）的快速发展，以LLM为核心构建的Agent系统受到了广泛的关注。Agent旨在利用LLM的归纳推理能力，并通过引入有效的工具和插件，从而完成复杂任务。与此同时，越来越多的Agent框架开始聚焦于多智能体协作场景，通过对Agent赋予不同的角色，并由Agent之间进行通信和协作从而达成更具有挑战性的目标。

在多智能体协作场景中，CDM是其关键的组成部分，很大程度影响系统的输出质量。而现有研究主要集中在Agent之间如何更高效更高质量地进行通信和交互，但CDM往往被忽视和过度简化。此外，由下表也可以看出，单一的CDM方法（如目前最常用的“独裁投票法”和“多数投票法”）均存在一定局限性，可能导致系统输出的结果并不合理和准确。

## 思路和方案

### 常用的CDM方法分析

通过作者对近期52个相关的多智能体协作系统的调研发现，大部分系统的CDM方法缺乏多样性且过分依赖“独裁投票法”和“相对多数投票法”，如下图所示：

由图可知，目前常用的CDM方法主要包括：Dictatorial（独裁投票法）和Plurality Voting（相对多数投票法），还有少数用到Utilitarian（功利主义投票法），以及部分不需要CDM的多智能体协作场景。

  * **Dictatorial** ：独裁投票法是目前最常用的CDM方法之一。大多数独裁框架都会**指定一个特别决策者** （可以称为领导者、决定者、法官或指挥官等），负责监督协作，评估结果，并对系统级别的决策拥有最终发言权。尽管如此，这种系统在某种意义上可以是“集体的”，因为“独裁者”可以通过其他选民进行咨询并与之沟通。在独裁投票机制中，也可以再细分为盲目独裁投票法、知情独裁投票法以及错误信息独裁投票法等。
  * **Plurality Voting** ：相对多数投票法也是目前最常用的CDM方法之一。其是一种简单的投票机制，核心规则是：在单轮投票中，**获得最多票数的候选人即当选为胜者** ，该候选人不需要获得超过半数的票即可获胜。此外，绝对多数（获得半数以上投票）和共识（需要每个选民一致同意）可以视作该方法的两种变体。
  * **Utilitarian** ：功利主义方法是一种基于功利主义原则的投票机制，量化了可能的决策影响，旨在**选择一个群体获得的集体“效用”或“报酬”最大化的决策，** 强调结果的最大化，而不是过程中的平等或其他伦理考量。值得注意的是，功利主义因其非自治性而区别于其他方法，即其“功利”是外部预先确定或更新的。



### 其他CDM方法分析

除上述已经应用在多智能体协作系统的CDM方法外，随着社会经济发展和环境变化，也演化了不少其他集体决策方案，例如：

**Range Voting（范围投票法）** ：范围投票法是一种**计分投票制度** ，允许选民给每个选项评分，以反映他们对每个选项的偏好程度。选民可以给每个选项分配一个数值评分，这个评分通常在一个预定的范围内，如从0到5的评分。这种方法类似于功利主义方法，但“效用”（即总分）是由选民给出的，而不是外部预先确定或更新的。

**Bucklin Voting（巴克林投票法）** ：在巴克林投票法中，选民通常需要基于自身的偏好**对所有选项进行排序** ，允许选民投出空白票或无效票。在所有选项中：1.首先计算第一选择的票数，如果一个选项获得多数票（即超过一半的票数），则该选项为最终输出结果；2.如果没有选项在第一选择中获得多数票，那么第二选择的票数会被加到第一选择上，如果此时有选项获得多数票，则该选项为最终输出结果；3.如果仍然没有候选人获得多数票，会继续添加更低排名的票数，直到有候选人获得多数票或所有排名的票数都被计算在内。

**Borda Count（波达计数投票法）** ：在波达计数投票法中，允许选民根据偏好**对选项进行排序，并通过分配分数来确定胜者** 。具体为：如果总共有N个选项，那么排在第一位的选项获得N-1分，排在第二位的选项获得N-2分，以此类推，直到排在最后一位的候选人获得0分。这种分数分配规则满足单调递减，即分数随着排名的降低而减少，且为非负值；进而，对于每个选项，计算其在所有选票上的分数总和，得分最高的选择即为最终输出结果。

**IRV （Instant-Runoff Voting，即时决策投票法）** ：在即时决策投票法中，允许选民对他们支持的选项进行排序，可以是第一选择、第二选择等。计票开始时：首先计算每位选民最高排名的选择。如果某个选项获得超过50%的第一选择票数，则该选项直接是最终输出结果；如果没有选项获得超过半数的第一选择票，那么得票最少的选项被淘汰，其票数会根据选民的下一个最高排名重新分配给其他选项；这个过程会一直重复，直到某个选项获得超过50%的票数，或者只剩下两个选项，此时票数多的选项视为最终输出结果。

**Minimax（极小化极大算法）** ：[Minimax算法](https://zhida.zhihu.com/search?content_id=251237851&content_type=Article&match_order=1&q=Minimax%E7%AE%97%E6%B3%95&zhida_source=entity)是一种重要的博弈搜索算法，它通过构建决策树并计算每个节点的minimax值来找到最优策略。其目标是在给定的局面下，通过选择最优的策略来最小化对手可能获得的最大收益。

**Ranked Pairs（排名配对投票法）** ：排名配对投票法通过史密斯标准（Smith criterion）和孔多塞胜者标准（Condorcet winner criterion），因此它是一种“孔多塞方法”。具体的：对每对选项进行投票计数，确定每对中的胜者（假设没有平局）。考虑每个选民的偏好，例如，如果一个选民表示“A > B > C”，则在A与B的比较中为A增加一票，在A与C的比较中也为A增加一票，在B与C的比较中为B增加一票。将胜者对（称为“多数”）从最大的多数排序到最小的多数。如果一个多数的支持度大于另一个，则将其排在前面；如果两个多数的支持度相等，则将拥有较小少数反对的多数排在前面

### GEDI模块设计

为了改进现有多智能体协作系统的CDM方案，作者提出了**GEDI，一个集成了多种序数偏好投票方法的通用选举决策模块** 。基于GEDI，可以研究CDM方法的多样化对多智能体协作系统的影响。此外，相较于独裁投票法和范围投票法而言，序数偏好投票法允许Agent对所有选项进行排序，并表达自己的偏好程度，更为灵活和细致。

如下图所示，集成了**GEDI模块** 的多智能体协作系统，会首先让每个Agent基于自身偏好独立进行序数排序投票；将所有Agent的投票结果收集汇总后，由GEDI模块选定一种序数偏好投票方法，并基于该方法最终输出集体优先的排序结果。通过引入多种投票策略，GEDI模块能够根据不同的任务场景和LLM模型类型等来选择更优的决策方法，提高了决策的灵活性和鲁棒性。

具体来说，**GEDI的输入** 由两部分组成：（1）每个Agent对所有选项的偏好排序（投票结果）集合，表示为一个配置文件，其中代表第i个Agent的偏好排序；（2）投票系统（社会选择函数，SCF），定义为一个映射，它根据每个严格偏好的配置文件返回一组备选方案。**GEDI的输出** 是选项集合的一个非空有序子集。

## 度量和评估

### 实验设置

**数据集：** 论文所采用的验证数据集包括MMLU、 MMLU-Pro以及ARC-Challenge。下面来简单介绍一下这几个数据集的特点：

  * MMLU（Massive Multitask Language Understanding）是一个大规模、多任务的语言理解验证数据集，旨在评估和提升LLM在各种语言理解任务上的能力。该数据集涵盖了广泛的主题和领域，如历史、文学、科学、数学等，通过这些多样化的主题和领域挑战LLM的理解能力和知识广度。在MMLU数据集中，共收集了15908个问题，并将其分为few-shot开发集、验证集和测试集。few-shot开发集每个学科有5个问题，验证集由1540个问题组成，测试集有14079个问题。每个学科至少包含100个测试问题。
  * MMLU-Pro数据集是MMLU的一个更具挑战性的版本（问题更复杂、选项更多），旨在评估和提升LLM在各种语言理解任务上的能力。在MMLU Pro数据集中，共包含了14个不同领域的学科子集，总计12032个问题，覆盖了数学、物理、化学、法律、工程、心理学和健康等领域。
  * ARC-Challenge数据集是AI2 Reasoning Challenge（ARC）的一部分，它包含了7,787个多项选择科学问题，旨在评估和提升LLM在各种语言理解任务上的能力。这些问题被分为“简易集”和“挑战集”，包含了需要复杂推理和科学知识，难以通过简单的信息检索或词频统计方法来解答，需要更高级的推理和理解能力。



**LLM：** 论文用到的测试LLM包括6个开源大模型以及2个商业上可用的专有大模型，分别为：Mistral-7B-v0.3、glm-4-9b-chat、Llama-3-8B/70B-Instruct、Qwen1.5-72B/110B-Instruct，以及gpt-3.5-turbo-1106和gpt-4-0125-preview。此外，所有使用到的LLM参数均一致且不再额外进行微调。

**实验方法：** （1）在每组实验中，基于相同的LLM来构建Agent，并通过在问题前面加上一段简短指令“你是{随机数}位评分者”以识别每个Agent；（2）对于MMLU和MMLU-Pro数据集，选择每个主题（即学科）的前100个案例来作为测试数据集（包含MMLU的5700个问题和MMLU-Pro的1400个问题），对于ARC-Challenge，使用了1172个案例的整个测试集。每个Agent被要求在上述数据集的每个问题上独立地提供选择的偏好排序（即投票结果）；（3）在收集所有Agent的排序结果并形成配置文件后，由GEDI模块输出符合所选投票规则的最终集体偏好排序；（4）如果输出列表的第一个元素与数据集设定的正确答案相匹配，则认为问题得到了正确的回答，计算数据集中所有问题的答准率（总体准确率）；（5）共计运行5次验证实验，取5次实验的正确率的平均值作为最终正确率。

### 实验结果

论文所进行的验证实验如下表所示。表中“Rand.”和“Dicta.”分别表示“random”和“dictatorial”。括号中的数字以盲目独裁投票法的正确率为基线，性能提升（正确率比基线高）则标记为红色 ，性能下降（正确率比基线低）则标记为蓝色。注：由于MMLU和ARC-Challenge数据集的题目为“四选一”，所以随机基线在25%左右；MMLU-Pro数据集的题目为“十选一”，所以随机基线在10%左右。

由表格数据可以总结以下几个**主要结论** ：

  * 基于社会选择理论的视角，目前常见CDM方法均具有一定局限性。
  * 绝大部分的序数偏好投票法的准确率高于积分投票法和独裁法。（尤其表现在gpt-3.5、gpt-4这两个专有模型上）
  * 各种序数偏好投票法在应对不同数据集、不同LLM时，其准确率也表现出较大差别。
  * 针对多智能体协同系统的集体决策（CDM）质量，需要重视和考虑选举方法的多样性和适用性。



此外，论文作者还研究了**构成有效决策群体的最少Agent数量** 。下图反映了在基于ARC-Challenge数据集的llama-3-8b、glm-4-8b，以及基于MMLU数据集的gpt-3.5、gpt-4的条件下，不同Agent数量对准确性产生的显著影响。由图可知，大多数CDM方法在涉及超过两个Agent的情况下，开始表现出显著的改进，并超过盲目独裁投票法基线。且即使只有三个Agent，一些CDM方法也能产生积极的协同效应。

## 局限性及展望

对于此论文的研究工作可能存在的局限性以及后续工作的展望，作者提到：

  * 本文的验证实验是基于MCQA（Multi-Choice Question-Answering）基准开展的，并在MMLU、MMLU-Pro和ARC-Challenge上的实验也反映出了显著的结果，但实际上MCQA并未完全符合集体决策场景，这表现在：不仅LLM在多选排序任务中表现出不一致性，且大多数MCQA基准测试都有预定的“正确”答案；然而，在没有绝对对错的情况下，CDM过程也可能是相关的。因此，未来的进一步研究可能是构建一个衡量偏好代表性的基准，而不是基于对错判断的标准。
  * 在自包含测试方面，本文也没有测试任何包含基于不同LLM的多智能体协作和集体决策，后续也可以开展该方面的研究和测试。
  * 尽管本文设计了GEDI模块，通过集成多种序数偏好投票机制作为多智能体协作系统集体决策方法多样化研究的基础，但GEDI中的CDM方法列表仍是不详尽的（例如组合多种投票策略的复合机制）。
  * 此外，尽管与LLM推理所需的巨大计算资源相比，CDM方法所需的计算资源（即“投票税”）要少得多，但仍需进一步权衡投票税与实际收益。



## 相关链接

  1. Zhao X , Wang K , Peng W .An Electoral Approach to Diversify LLM-based Multi-Agent Collective Decision-Making[J]. 2024.
  2. 论文中的GEDI模块代码GitHub地址：[https://github.com/xiutian/GEDI](https://link.zhihu.com/?target=https%3A//github.com/xiutian/GEDI)
  3. MMLU的GitHub地址：[https://github.com/SophieWu9824/MMLU-dataset](https://link.zhihu.com/?target=https%3A//github.com/SophieWu9824/MMLU-dataset)
  4. MMLU-Pro的GitHub地址：[https://github.com/TIGER-AI-Lab/MMLU-Pro](https://link.zhihu.com/?target=https%3A//github.com/TIGER-AI-Lab/MMLU-Pro)
  5. ARC-Challenge的GitHub地址：[https://github.com/intersun/ARC-Challenge](https://link.zhihu.com/?target=https%3A//github.com/intersun/ARC-Challenge)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【多Agent洞察】05-AgentScope：灵活而强大的多Agent平台](https://zhuanlan.zhihu.com/p/695452823)  
> 下一篇：[【多Agent洞察】07-通过学习感知策略梯度算法实现多智能体合作](https://zhuanlan.zhihu.com/p/11985058911)

## 附录

**MMLU数据集的实际例子**

Mathematics（数学）：问题为“What is the square root of 16?”，选项为“A) 2, B) 4, C) 16, D) 8”，正确答案为“D) 8”；

Computer Science（计算机科学）：问题为“What is the difference between CPU and GPU?”，选项为“A) CPU stands for Central Processing Unit, B) GPU stands for Graphics Processing Unit, C) Both are the same, D) None of the above”，答案为“B) GPU stands for Graphics Processing Unit”。

**MMLU-Pro数据集的实际例子**

Business（金融）：问题为“Ms. Chen purchased a used car, worth $1650, on the installment plan, paying $50 down and $1,840 in monthly installment payments over a period of two years. What annual interest rate did she pay?”，选项为“A) 10%, B) 17.5%, C) 15.2%, D) 13.3%, E) 20%, F) 19.8%, G) 18%, H) 16%, I) 12%, J) 14.4%”，正确答案为“J. 14.4%”；

Physics（物理）：问题为“Which of the following is not a requirement for a valid contract?”，选项为“A) Offer, B) Acceptance, C) Consideration, D) Intention to create legal relations, E) Capacity of parties, F) Formalities”，答案为“F) Formalities”。

**ARC-Challenge数据集的实际例子**

Physics（物理）：问题为“A student riding a bicycle observes that it moves faster on a smooth road than on a rough road. This happens because the smooth road has: ”，选项为“A) less gravity, B) more gravity, C) less friction, D) more friction”，答案为“C) less friction”；

Geology（地质）：问题为“Which property of a mineral can be determined just by looking at it?”，选项为“A) luster, B) mass, C) weight, D) hardness”，答案为“A) luster”。
