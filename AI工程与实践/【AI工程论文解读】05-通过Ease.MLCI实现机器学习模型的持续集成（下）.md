# 【AI工程论文解读】05-通过Ease.ML/CI实现机器学习模型的持续集成（下）

原文链接：https://zhuanlan.zhihu.com/p/581384885

---

​

目录

>  _[持续集成](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity) 是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，[自动化测试](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95&zhida_source=entity))来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。_

在上一篇《[【AI工程论文解读】04-通过Ease.ML/CI实现机器学习模型的持续集成（上）](https://zhuanlan.zhihu.com/p/581822019)》文中，我们介绍了ease.ml/ci的[系统设计](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1&zhida_source=entity)和基本实现，本篇将主要分享基于ease.ml/ci进行的优化和实验。

## 优化

如前所述，ease.ml/ci基本实现无法为低容错和或自适应场景提供可用的方案。因此，需要对进一步优化[样本量](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%A0%B7%E6%9C%AC%E9%87%8F&zhida_source=entity)估计器。

**High-level Intuition**

本节中我们提出的所有技术都基于相同的洞察：在最坏的情况下，收紧样本[估计量](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E4%BC%B0%E8%AE%A1%E9%87%8F&zhida_source=entity)很难比epsilon%5E%7B2%7D%29" alt="O(1/\epsilon^{2})" eeimg="1"/>更好；相反，我们采取经典的系统思维方式—对普遍的测试条件进行样本估计量的改进。因此，对于不同形式的测试条件，ease.ml/ci采用不同的[优化技术](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E4%BC%98%E5%8C%96%E6%8A%80%E6%9C%AF&zhida_source=entity)。

**技术观察1**

当新模型和旧模型的预测差异仅为 （）（100×p）（100×p）% %（这可能是测试条件的一部分）时，对于数据点 ii ，[随机变量](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E9%9A%8F%E6%9C%BA%E5%8F%98%E9%87%8F&zhida_source=entity) ni−oin_{i} - o_{i}  具有较小的[方差](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%96%B9%E5%B7%AE&zhida_source=entity)： E[(ni−oi)2]<pE[(n_{i}-o_{i})^{2}]<p ，其中 nin_{i}  和 oi o_{i}  是新模型和老模型对数据点i的预测。这允许我们应用标准Bennett’s[不等式](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E4%B8%8D%E7%AD%89%E5%BC%8F&zhida_source=entity)。

命题1（Bennett’s不等式）：假设 X_{1},…,X_{n} 是独立且平方可积的随机变量，使得对于某个非负常数 b ， \left| X_{i} \right|\leq b 几乎适用于所有 i<n 。得出如下公式：

Pr\left[ \left| \frac{\sum_{i}^{}{X_{i}-E\left[ X_{i} \right]}}{n} \right|>\epsilon \right]\leq2exp(-\frac{v}{b^{2}}h(\frac{nb\epsilon}{v})) ,其中 v=\sum_{i}^{}{\left[ X_{i}^{2} \right]} ,对于左右的正 u ，h(u)=(1+u)\ln_{}{(1+u)}-u 。

**技术观察2**

如果要估计新模型和旧模型之间的预测差异，不需要有标记。相反，来自未标记数据集的样本就足以估计差异。此外，当只有10%的[数据点](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=3&q=%E6%95%B0%E6%8D%AE%E7%82%B9&zhida_source=entity)具有不同的预测时，为了估计 n-o ，只需为整个测试集中的10%提供标记。

### 模式1：基于差异的优化（Difference-based Optimization）

ease.ml/ci在公式中搜索的第一个模式是它是否符合以下形式

d < A +/- B  /\ n - o > C +/- D

上述限制了新模型允许的变更量，并同时确保新模型不比旧模型差。这两个子句经常出现在用户的测试条件中：对于生产级系统，开发人员从部署模型开始，并花费大部分时间微调机器学习模型。因此，[持续集成测试](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95&zhida_source=entity)必须具有低至单个精度点的容错。另一方面，新模型与旧模型不会有明显的区别，否则会花费需要更多的调试和研究。

假设一种优化方案，获取无标记的数据样本相对便宜，而提供标记则比较昂贵。当此假设有效时，[分层测试](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E5%88%86%E5%B1%82%E6%B5%8B%E8%AF%95&zhida_source=entity)和主动标记中两种优化都可以应用于此模式；否则，两种优化仍然适用，但只会对一个子集进行改进。

### 分层测试（Hierarchical Testing）

第一个优化是测试以 d < A +/- B 为条件的其余子句，这导致了一个具有两级测试的[算法](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E7%AE%97%E6%B3%95&zhida_source=entity)。第一级测试新模型和旧模型之间的差异是否足够小，而第二级测试 （n-o） 。

该算法分两步运行：

  1. (过滤)用 n^{'} 个样本得到 (\epsilon^{'},\frac{\delta}{2}) —估计量 d 。测试：  \hat{d}>A+\epsilon^{'} 如果是，返回False；
  2. (测试)测试 F ，如在系统实现 (1-\frac{\delta}{2}概率) 中，以 d<A+2\epsilon^{'} 为条件。



不难看出上述算法的工作原理——第一步只需要未标记的数据点，不需要人工干预。第二步，以 d < p 为条件，已经知道每个数据点的E[(n_{i}-o_{i})^{2}]<p。结合\left| n_{i} - o_{i} \right|<1，应用Bennett's inequality得到如下公式。

Pr\left[ \left| \widehat{n-o} -(n-o)\right| >\epsilon\right]\leq2exp(-nph\left( \frac{\epsilon}{p} \right)) 。

因此，第二步需要的样本量(对于非自适应场景) n=\frac{\ln_{}{H}-\ln_{}{\frac{\delta}{4}}}{ph(\frac{\epsilon}{p})} 。

当 p = 0.1,1-δ = 0.9999,d < 0.1 时，非自适应场景下， H 为 32 时只需要29K样本；自适应场景下， H 为 32 时只需要67K样本，就可以达到比[基线](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E5%9F%BA%E7%BA%BF&zhida_source=entity)少10倍的单精度点容错(图2)。

### 主动标记（Active Labeling）

前面的示例为用户提供了一种只用67K样本执行32个完全自适应微调步骤的方法。假设开发人员每天执行一次提交，这意味着我们每个月需要67K个示例来支持[持续集成服务](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E6%9C%8D%E5%8A%A1&zhida_source=entity)。

该策略的一个潜在挑战是，在持续集成服务开始工作之前，需要对所有67K个样例进行标记。在理想的情况下，我们希望将开发工作与标记工作交织在一起，并随着时间的推移对标记工作进行分摊。

系统使用的第二种技术依赖于估计 (n-o) ，只需要标记在新模型和旧模型之间具有不同预测的数据点。当我们知道新模型的预测与旧模型只相差10%时，我们只需要标记所有数据点的10%。很容易看出，每当开发人员提交一个新模型时，我们只需要提供n=\frac{-\ln_{}{\frac{\delta}{4}}}{ph(\frac{\epsilon}{p})}\times p个标记。

当 p = 0.1,1 - δ = 0.9999 时，单个精度点的误差容限 n = 2188 。如果开发人员每天提交一个模型，那么标记团队第二天只需要标记2188个样本。考虑到一个设计良好的界面，可以实现每5秒的标记一个标记，则只需要3小时即可完成。对于拥有多个工程师的团队来说，考虑到系统提供的保证，这种开销通常是可以接受的。

主动标记假定是一个平稳的底层分布。在系统中实现该目的的方法是要求用户同时提供一个未标记的数据点池，然后仅在需要时要求提供标记。这样，就不需要随着时间的推移绘制新的样本。

### 模式2：隐式方差边界（Implicit Variance Bound）

在很多情况下，用户没有对新模型和旧模型之间的差异提供明确的约束。然而，许多机器学习模型的预测结果并没有那么大的差异。以AlexNet、ResNet、GoogLeNet、AlexNet（Batch Normalized）和VGG为例：当应用于ImageNet测试集时，这五个开发的模型只会在top-1的正确性上产生至多25%的不同答案，而在前5的正确性上只会产生15%的不同答案。因此，对于持续集成工作，期望多次提交的差异小于经过多年开发的ImageNet的差异并非不合理。

受此启发，ease.ml/ci将自动匹配以下模式：

n - o > C +/- D.

当无标记测试集的获取成本较低时，系统将使用一个测试集来估计 d [高达](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E9%AB%98%E8%BE%BE&zhida_source=entity) ϵ = 2D ：对于二项分类任务，系统可以使用无标记测试集；对于多类任务，既可以在未标记的测试集上测试预测的差异，也可以在标记的测试集上测试正确性的差异。这给了一个 n - o 的上限。然后，系统在另一个测试集(与测试d所用的测试集不同)上测试 n – o 直到 ϵ = D 。当这个上限足够小时，系统将触发类似于模式1中的优化。

需要注意的是在执行该方法之前，系统不知道第二个测试集的大小。如果需要，系统会使用一种类似于主动标记的技术，即在每次提交新模型时通过递增的方式增加标记的测试集。具体来说，根据[模式优化](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%A8%A1%E5%BC%8F%E4%BC%98%E5%8C%96&zhida_source=entity)测试条件

n > A +/- B,

当 A 较大（如0.9或0.95）时，可以先对n的下限进行粗略预估，然后根据该下限来实现。这种改进方法只能在下限较大（例如0.9）时引入。

### 严格的数值边界（Tight Numerical Bounds）

在（Langford,2005）之后，可以从Bernoulli分布中提取的 n i.i.d随机变量组成的测试条件，人们可以简单地导出达到 (ϵ,δ) 精度所需的样本数。样本数的计算需要使用Binomial distribution (sum of i.i.d Bernoulli variables)。Tight Bound是通过取所需样本数 n 的最小值，而不是未知的最大真平均值 p 来解决的。这种技术也可以扩展到更复杂的查询，其中二项分布必须被[多模态分布](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E5%A4%9A%E6%A8%A1%E6%80%81%E5%88%86%E5%B8%83&zhida_source=entity)取代。对于简单的情况，精确的分析没有封闭的解，推导有效的[近似值](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E8%BF%91%E4%BC%BC%E5%80%BC&zhida_source=entity)留待后续工作。

图3 基本实现和优化实现中样本量估计器的比较

## 实验

### 样本量估计器

通过[样本方差](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%A0%B7%E6%9C%AC%E6%96%B9%E5%B7%AE&zhida_source=entity)的上限，相对于Hoeffding边界，能够使用更严格的边界。上限可以通过使用未标记的数据点来估计新模型和旧模型之间的差异来获取；也可以在使用标记的数据点前先进行粗略估计获取。本文中，验证了我们的理论界限及其对提高标记复杂性的影响。

图3中，通过假设不同的上限 p ，对于精度在98%左右的模型，给出了估计误差和经验误差。通过在无限MNIST数据集（Bottou,2016）上运行GoogLeNet（贾等人，2014），并估计真实精度 c 。假设在非自适应场景，通过随机抽取 n 个数据点获得了一系列精度。然后，用给定的样本数n和概率 1-δ 估计区间 ϵ 。可以得出，基本实现和ease.ml/ci都具有经验误差，正如预期的那样，ease.ml/ci使用的样本数量明显较少。

图4中展示了上限对提高标记复杂性的影响。可以看到，当 p 相当小时，改进显著增加；当 p = 0.1 时，在标记复杂性上几乎实现10倍的改进。而主动标记更进一步进行了改善，正如预期的那样，又增加了10倍。

图4 _、_和p对标记复杂度的影响

### 进行中的ease.ml/ci

作者根据SemEval-2019任务3竞赛的机器学习模型开发了基于ease.ml/ci的三种不同测试条件，在开发测试条件时采用了非自适应场景。图5展示了三个相似但不同的测试条件，在非自适应场景下，前两个条件检查新模型是否比旧模型至少好2个百分点。第三个条件模拟了用户在每次提交后都会得到反馈，没有任何[false negative](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=false+negative&zhida_source=entity)。ease.ml/ci使用模式2优化了所有三个查询，任意两个提交之间的预测差异不超过10%。

简单地使用[Hoeffding不等式](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=Hoeffding%E4%B8%8D%E7%AD%89%E5%BC%8F&zhida_source=entity)并不能得到一个实际的解决方案，对于 ϵ= 0.02、δ= 0.002、H = 7 时，需要 n>\frac{r_{v}^{2}(\ln_{}{H}-\ln_{}{\frac{\delta}{2}})}{2\epsilon^{2}}=44,268 个样本。在完全适应的情况下，这个数字甚至高达58K。

图5 ease.ml/ci中的持续集成

前两个条件可以在2个百分点的容错范围内和0.998可靠性内解答。第三种情况下的全自适应查询只能实现2.2个百分点的容错性，由于所需的标记数量将超过6K，具有与前两个查询相同的容错性。

可以看到，在所有三种情况下ease.ml/ci返回的是直观意义的通过/失败信息。如果细看在8个迭代中开发和测试精度的演变（见图6），理想的情况是开发人员希望ease.ml/ci接受其最后一个提交，而所有三个查询都将选择倒数第二个模型作为生产模型，主要和测试精度的演变相关。

图6 开发和测试精度的演变

## 结论

ease.ml/ci，一个用于机器学习的持续集成系统，提供了一种声明性[脚本语言](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%80&zhida_source=entity)，允许用户用严格的概率保证来声明丰富的测试条件类。同时，作者们还研究了针对于机器学习模型的标记管理工作的实用性问题。基于作者们的持续集成技术，可以将实际[生产系统](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E7%94%9F%E4%BA%A7%E7%B3%BB%E7%BB%9F&zhida_source=entity)中使用的测试条件所需的标记数量降低两个[数量级](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%95%B0%E9%87%8F%E7%BA%A7&zhida_source=entity)。作者们对技术的可靠性已做了验证，并展示了其在实际生产场景中的应用。

## 附录A 语法和语义

### A.1 语法

为了指定条件，每当提交新模型时，将通过ease.ml/ci进行测试，用户使用以下语法：
    
    
    c   :- floating point constant
    v   :- n | o | d
    op1 :- + | -
    op2 :- *
    EXP :- v | v op1 EXP | EXP op2 c
    cmp :- > | <
    C   :- EXP cmp c +/- c
    F   :- C | C /\ F

F 作为最终条件，它是一组子句 C 的集合。每个子句都是基于{ n, o, d }的表达式和常数之间的比较，符号 +/- 后面有一个容错。例如，重点优化的两个[表达式](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=2&q=%E8%A1%A8%E8%BE%BE%E5%BC%8F&zhida_source=entity)如下：

n - o > 0.02 +/- 0.01 /\ d < 0.1 +/- 0.01

其中第一个子句

n - o > 0.02 +/- 0.01

要求新模型的精度比旧模型高两分，[容错率](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E5%AE%B9%E9%94%99%E7%8E%87&zhida_source=entity)为1个点，而子句

d < 0.1 +/- 0.01

要求新模型只能改变旧预测的10%，容错率为1%。

### A.2 语义

与传统的[连续积分](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E8%BF%9E%E7%BB%AD%E7%A7%AF%E5%88%86&zhida_source=entity)不同，ease.ml/ci、{ n, o, d }中使用的三个变量都是随机变量。因此，对ease.ml/ci条件的评估具有内在的[概率性](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E6%A6%82%E7%8E%87%E6%80%A7&zhida_source=entity)。用户还需要提供2个额外的参数来定义测试条件的语义：（1） δ ，允许测试过程出错的概率，通常选择小于 0.001 或 0.0001 (即 0.999 或 0.9999 成功率)；（2）从{ fp-free, fn-free }中选择模式，它指定测试是无 false-positive 或者无 false-negative 。其语义是，在概率为 1-δ 的情况下，ease.ml/ci的输出不存在 false positive" eeimg="1"/> 或者 false negative 。

false positive 或 false negativ 的概念与统计假设实验中“I型”错误和“II型”错误之间的基本权衡有关。考虑

x < 0.1 +/- 0.01.

假设 x 的实际未知值是 x^{\ast} 。给定一个[估计值](https://zhida.zhihu.com/search?content_id=217330645&content_type=Article&match_order=1&q=%E4%BC%B0%E8%AE%A1%E5%80%BC&zhida_source=entity) \hat{x} ，其概率 1-δ 满足：

\hat{x}\in\left[ x^{\ast}-0.01,x^{\ast}+0.01 \right] 

这种情况的测试结果有三种情况：

  1. 当 \hat{x}>0.11 时，测试条件返回 False ，因为给定 x^{\ast} < 0.1 ， \hat{x}> 0.11 > x^{\ast}+0.01 的概率小于 δ 。
  2. 当 \hat{x}<0.09 时，测试条件返回 True ，因为给定 x^{\ast} > 0.1 ， \hat{x}< 0.09 < x^{\ast}-0.01 的概率小于 δ 。
  3. 当 0.09<\hat{x}<0.11 时，无法确定结果：即使 \hat{x}>0.1 ，也无法判断 x^{\ast} 的实际值是大于还是小于0.1。在本例中，测试条件的计算结果为 Unknown 。



参数模式允许系统处理条件评估为未知的情况。在 fp-free 模式中，ease.ml/ci将 Unknown 视为 False (因此拒绝提交)，以确保每次使用 \hat{x} 评估测试条件为 True 时，对于x^{\ast}来说，相同的测试条件总是返回 True 。类似地，在 fn-free 模式下，ease.ml/ci将 Unknown 视为 True (因此接受提交)。在 fn-free (resp. fp-free)模式下，false positive rate ((resp. false negative rate)由容错率决定。

## 参考资料

  1. Cedric Renggli, Bojan Karlaš, Bolin Ding, Feng Liu,[Wentao Wu](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/people/wentwu/), Ce Zhang, [Continuous Integration of Machine Learning Models with ease.ml/ci: Towards a Rigorous Yet Practical Treatment](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/publication/continuous-integration-of-machine-learning-models-with-ease-ml-ci-towards-a-rigorous-yet-practical-treatment/) SysML Conference (SysML 2019)



  


> 上一篇：[【AI工程论文解读】04-通过Ease.ML/CI实现机器学习模型的持续集成（上）](https://zhuanlan.zhihu.com/p/581822019)  
> 下一篇：[【AI工程论文解读】06-AI4SE和SE4AI：研究路线图](https://zhuanlan.zhihu.com/p/598460506)
