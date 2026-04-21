# 【模型路由】02-RouteLLM：利用偏好数据学习路由LLMS

原文链接：https://zhuanlan.zhihu.com/p/21408688093

---

​

目录

## 简介

大型语言模型（LLMs）在多种自然语言处理任务中展现了卓越的能力。然而，在实际应用中选择使用哪个模型时，往往需要在性能和成本之间做出权衡。更强大的模型虽然响应质量高，但成本也更高；而能力较弱的模型则具有更低的服务成本。

为了解决这个困境，论文提出了一个**新的[RouteLLM](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=RouteLLM&zhida_source=entity)框架**，**用于在强模型和弱模型之间进行查询路由选择。通过学习用户偏好数据，预测强模型获胜的概率，并根据成本阈值α来决定使用哪种模型处理查询。** 该研究主要应用于大规模语言模型（LLMs）的实际部署中，通过智能路由在保证响应质量的前提下显著降低成本。

## 评估指标

  * [PGR](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=PGR&zhida_source=entity)：性能差距恢复（performance gap recovered），定义整体性能增益。  




  * [APGR](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=APGR&zhida_source=entity)：平均性能差距恢复（average performance gap recovered），定义路由器在不同成本约束（通过改变阈值α来实现）下恢复性能差距的总体衡量标准。  




  * [CPT](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=CPT&zhida_source=entity)：调用性能阈值（call-performance threshold）。给定所需的路由器性能（以 x% 的 PGR 来衡量），CPT(x%) 是指为了获得所需的 PGR，需要调用强模型（如[GPT-4](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=GPT-4&zhida_source=entity)）的最小百分比。例如，CPT(50%)，即实现 50% 的性能增益（PGR）所需的 GPT-4 调用百分比。



## 方法论

### 偏好数据

论文使用来自[Chatbot Arena](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=Chatbot+Arena&zhida_source=entity)的80k对战数据作为训练数据，在该平台上，用户与聊天机器人界面交互并提交他们选择的提示，并在收到两个匿名模型的响应后，投票选出获胜模型或平局。由此产生的数据集 D_{arena} 包含用户查询、两个模型的回答以及基于人类判断的成对比较标签。

在路由模型的训练过程中，研究人员探索了2种数据增强技术：

  * 利用带有黄金标签的数据集 D_{gold} ，如[MMLU验证集](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=MMLU%E9%AA%8C%E8%AF%81%E9%9B%86&zhida_source=entity)。这些数据集包含自动计算的正确答案，例如多项选择题的答案，这些答案被用来生成偏好数据，从而增强训练数据的质量和数量。
  * 使用LLM评审数据集 D_{judge} ，这些数据集通过一个强大的语言模型（如GPT-4）作为评审者，对一系列用户查询进行响应，并生成强模型和弱模型的对比标签。



### 路由方法

论文介绍了4种从偏好数据学习获取预测模型的方法。

  * **相似度加权（SW）排名** ：通过计算训练集中每个相似查询的加权排名，最终得到模型获胜概率。
  * **矩阵分解方法** ：通过训练偏好数据来揭示隐藏的评分函数，从而确定不同模型对查询的质量。
  * **[BERT分类器](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=BERT%E5%88%86%E7%B1%BB%E5%99%A8&zhida_source=entity)** ：使用了标准的文本分类方法，通过全参数微调BERT模型来进行查询路由决策。
  * **[因果LLM分类器](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=%E5%9B%A0%E6%9E%9CLLM%E5%88%86%E7%B1%BB%E5%99%A8&zhida_source=entity)** ：通过参数化Llama 3模型，并采用指令跟随范式来预测查询的胜率。



## 实验结果

使用来自Chatbot Arena的80k对战数据作为训练数据，进一步通过增强数据集（如MMLU验证集和GPT-4评审数据）来提高路由器性能。在三个广泛使用的基准（MT Bench、MMLU和[GSM8K](https://zhida.zhihu.com/search?content_id=253303970&content_type=Article&match_order=1&q=GSM8K&zhida_source=entity)）上评估路由模型的性能，包括开放式问答、文科和数学问题，展示了路由模型在不同数据集上的广泛适应性和优越性能。

### MT Bench基准

在MT Bench基准测试中，对于使用 arena数据集训练路由器，可以发现矩阵分解和相似性加权排名的强大性能，两个路由器在所有指标上的性能都明显优于随机路由器。与随机路由器相比，矩阵分解路由器仅需要一半的GPT-4调用次数就能实现50%的PGR。

使用增强数据集（ judge）可以在所有路由器上带来显著改进。与随机路由器相比，BERT分类器的APGT提高了50%以上。当在此增强数据集上进行训练时，矩阵分解是性能最好的路由器，因为与随机路由器相比，其CPT(80%)几乎减半，需要调用的GPT-4减少了50%。

### MMLU基准

在MMLU测试中，所有路由器在仅使用 arena数据集训练时表现较差，但通过增强数据集（ gold），性能显著提升。CPT (50%)场景下，所有路由器所需的 GPT-4 调用比随机路由器少大约 20%。 重要的是，尽管事实上大约 1500 个样本的增强数据集（ gold）只占整个训练数据的不到 2%，这证明了即使样本数量很少时数据集增强的有效性。

### GSM8K基准

在GSM8K测试中，使用增强数据集后的因果LLM路由器，CPT(50%)从56.09%下降到33.64%，APGR从0.461提高到0.622，展现了25.3%的显著改进。矩阵分解路由器的CPT(50%)也从53.59%降至38.82%，APGR提高到0.565，表现出13.8%的提升。

### 迁移学习能力

在上述实验中，研究人员选择 gpt-4-1106-preview 和 Mixtral 8x7B 作为强弱模型的代表。但是，为了证明这个框架对不同模型对的泛化能力，研究人员还使用 Claude 3 Opus 和 Llama 3 8B 作为强弱模型，分析了不同路由器在 MT Bench基准 上的性能。重要的是，使用相同的路由器没有任何重新训练，并且仅替换强模型和弱模型。这两个模型也不存在于训练数据中。

替换强弱模型后，仍然能观察到强劲的结果。新模型对和原始模型对的结果仍然明显强于随机模型，在使用增强数据集（ judge）后，所有路由模型比随机模型少 30% 的 GPT-4 调用就能实现 CPT (80%)。这些结果表明，论文提出的路由器已经学习了可以区分强模型和弱模型的问题的一些共同特征，这些特征可以推广到新的强模型和弱模型对，而无需额外的训练。

## 结论

总结来说，这篇论文通过提出一种动态路由框架，能够根据任务对模型性能的需求，智能地在不同的LLM之间进行选择，以达到成本和性能的最佳平衡。

路由模型的迁移学习能力也得到了验证，即使在测试时强弱模型发生变化，模型仍能保持稳定的性能表现。这主要归功于模型在训练过程中学习到的通用特性和丰富多样的训练数据。研究人员计划在未来引入更多不同的模型组合进行测试，以进一步验证和提高模型的迁移学习能力。

## 参考资料

  1. [https://arxiv.org/pdf/2406.18665](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2406.18665)
  2. [https://developer.aliyun.com/article/1560386](https://link.zhihu.com/?target=https%3A//developer.aliyun.com/article/1560386)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【模型路由技术洞察】01-OrchestraLLM：高效协调对话状态跟踪语言模型](https://zhuanlan.zhihu.com/p/700883845)
