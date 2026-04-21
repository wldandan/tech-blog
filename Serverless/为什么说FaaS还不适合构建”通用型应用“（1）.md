# 为什么说FaaS还不适合构建”通用型应用“（1）

原文链接：https://zhuanlan.zhihu.com/p/108609862

---

2019年3月，[伯克利](https://zhida.zhihu.com/search?content_id=112266117&content_type=Article&match_order=1&q=%E4%BC%AF%E5%85%8B%E5%88%A9&zhida_source=entity)的一篇论文《A Berkeley View on Serverless Computing》对Serveless做了全面的分析，并预测**它将成为未来[云计算](https://zhida.zhihu.com/search?content_id=112266117&content_type=Article&match_order=1&q=%E4%BA%91%E8%AE%A1%E7%AE%97&zhida_source=entity)的主宰**。

>  _“We predict that serverless computing will grow to dominate the future of cloud computing” – Berkeley CS Dept_

2019年12月，AWS REinvent上发布的数个Serverless更新，反响热烈，业界对Serverless的**重视与期待可见一斑** 。

实际上， **当今的Serverless虽然发展迅猛，但其更关注底层基础设施的自动化与效率，对应用的多样性需求，却支撑不足。**

典型如Lambda，作为FaaS的主要代表之一，其核心特征是**无状态、不可寻址、短生命周期** ，这非常适合互联网领域**高吞吐、快速伸缩、高可用** 的应用场景，但对于**低延时、强一致等** 的应用（如实时分析与预测、机器学习的模型训练、[事务处理](https://zhida.zhihu.com/search?content_id=112266117&content_type=Article&match_order=1&q=%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86&zhida_source=entity)等），却并不太友好。

> 正如Adzic在论文《Serveless Computing: Economic and Architectural Impact》中所描述 “当今Serverless可用于某些重要的任务。这些任务中，**高[吞吐量](https://zhida.zhihu.com/search?content_id=112266117&content_type=Article&match_order=1&q=%E5%90%9E%E5%90%90%E9%87%8F&zhida_source=entity)是关键**，同时单个请求也需要在较短时间内完成处理。通过在Serverless环境中运行此类任务，能显著降低成本并加快新功能的交付，这是令人信服的”

无状态的FaaS目前在Serverless中占据了重要的地位，但距离Serverless的宏伟愿景还远不够。相信随着Serverless的快速发展，对**[有状态](https://zhida.zhihu.com/search?content_id=112266117&content_type=Article&match_order=1&q=%E6%9C%89%E7%8A%B6%E6%80%81&zhida_source=entity) 的探索与优化**将会成为重要的研究之一。

本文先抛个砖，后面几篇文章将探讨Serverless与有状态的话题。
