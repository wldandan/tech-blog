# 【MindSpore易点通机器人-01】你也许见过很多知识问答机器人，但这个有点不一样

原文链接：https://zhuanlan.zhihu.com/p/492438570

---

2022年3月， _[MindSpore易用性SIG](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig)_ 成立后，很快吸引了不少开发者加入SIG群中进行交流。作为和开发者进行连接的桥梁，易用性SIG的目标是**和开发者共同打造易学易用、灵活高效的AI框架，持续提升MindSpore易用性，助力开发者成功** 。

通过易用性SIG，我们不仅倾听开发者的声音，为开发者提供帮助，也希望能和开发者一起围绕“**MindSpore的学习与实践** ”这个主题，打造一些有趣的[开源项目](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE&zhida_source=entity)。

## **一、为什么发起这个开源项目(** 知识问答机器人**)**

在浏览了当前的一些MindSpore应用案例（如[图片识别](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E5%9B%BE%E7%89%87%E8%AF%86%E5%88%AB&zhida_source=entity)、房价预测等）后，我们认为这些案例**简洁易懂，作为学习资源是非常合适的** 。但作为易用性Sig，我们不仅希望发起的项目是一个**学习资源** ，更希望**它能和易用性有点关系** （能让开发者参与其中，也能帮助提升MindSpore易用性）。

在经历了数轮头脑风暴之后，我们想到 何不针对开发者经常反馈的**MindSpore学习资料分散、问题答案寻找困难这个问题** ，基于MindSpore实现一个知识问答机器人，**既帮助开发者学习/实践MindSpore，输出的成果还能让开发者更容易地使用MindSpore** 。

**1\. 为开发者提供便捷的问题解答途径**

开源两年以来，MindSpore的[编程指南](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E7%BC%96%E7%A8%8B%E6%8C%87%E5%8D%97&zhida_source=entity)、API文档、教程、FAQ等学习资料已经积累了很多，面对这些学习资料，我们希望通过问答交互方式让用户快速、直接地获得答案。例如可以将机器人集成到IDE中，让用户“手”不出IDE，通过快捷键操作就可以获得答案。也可以将机器人集成在官网或者社媒中，让开发者直接提问获取答案。

**2\. 与开发者共同打造一套基于MindSpore[全栈](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E5%85%A8%E6%A0%88&zhida_source=entity)的学习教程**

问答机器人**可以作为MindSpore在NLP领域的一个教程** 。同时，在构造这个项目的过程中，我们会将**MindSpore的[全栈能力](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E5%85%A8%E6%A0%88%E8%83%BD%E5%8A%9B&zhida_source=entity)**(MindInsight、MindArmour、MindSpore XAI、MindSpore Lite、MindSpore Dev Toolkit等)应用其中，并将** _[AI工程](https://zhuanlan.zhihu.com/p/484046440)_ （如MLOps、机器学习设计模式等）系统化地实践**。该案例也会录制为训练营课程，在易用性SIG活动中持续推出。

> 注：MindSpore全栈包括MindSpore框架以及相关组件，如MindInsight、MindArmour、MindSpore XAI、MindSpore Lite、MindSpore Dev Toolkit等。

* * *

## **二、知识问答机器人能做什么**

知识问答机器人接受用户提问、查询相关内容、引导开发者快速获取答案，并且可以集成到IDE、官网、微信公众号、Gitee、MindSpore掌中宝等平台，供开发者通过多种方式使用。

  1. **目前初步规划的功能** ：



1) **直接检索答案**

基于用户问题，直接提供答案（包括编程指南、API映射、算子、报错地图等）。

> 例如：开发者希望从PyTorch将代码迁移到MindSpore，需查询API[映射](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=2&q=%E6%98%A0%E5%B0%84&zhida_source=entity)关系：  
> **·** **用户** ：pytorch torch.chunk api 对应的MindSpore API是什么？  
> **·** **机器人** ：对应的接口为 mindspore.ops.split，详细地址为([https://[mindspore](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=5&q=mindspore&zhida_source=entity).cn/docs/api/zh-CN/r1.6/api_python/ops/mindspore.ops.Split.html#mindspore.ops.Split](https://link.zhihu.com/?target=https%3A//mindspore.cn/docs/api/zh-CN/r1.6/api_python/ops/mindspore.ops.Split.html%23mindspore.ops.Split))

2) **多轮对话交互获取答案**

通过多轮交互，引导用户补充上下文，提供答案。

> 例如，某用户在安装MindSpore时遇到了问题:  
> **·** **用户** ：安装mindspore，提示“该版本不支持当前平台”  
> **·** **机器人** ：请输入您的[操作系统](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&zhida_source=entity)  
> **·** **用户** ：macOS  
> **·** **机器人** ：请输入您的处理器类型（Intel/M1）  
> **·** **用户** ：M1  
> **·** **机器人** ：请输入您的python版本  
> **·** **用户** ：3.7  
> **·** **机器人** ：由于M1芯片不支持python 3.7版本，建议您在python 3.9.1版本上进行安装，安装命令是：pip install [https://ms-release.obs.cn-north-4.myhuaweicloud.com/1.6.1/MindSpore/cpu/aarch64/mindspore-1.6.1-cp39-cp39-macosx_11_0_arm64.whl](https://link.zhihu.com/?target=https%3A//ms-release.obs.cn-north-4.myhuaweicloud.com/1.6.1/MindSpore/cpu/aarch64/mindspore-1.6.1-cp39-cp39-macosx_11_0_arm64.whl) \--trusted-host [http://ms-release.obs.cn-north-4.myhuaweicloud.com](https://link.zhihu.com/?target=http%3A//ms-release.obs.cn-north-4.myhuaweicloud.com) -i [https://pypi.tuna.tsinghua.edu.cn/simple](https://link.zhihu.com/?target=https%3A//pypi.tuna.tsinghua.edu.cn/simple)

**2.[逻辑模型](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E9%80%BB%E8%BE%91%E6%A8%A1%E5%9E%8B&zhida_source=entity)与主要功能：**

智能问答机器人逻辑模型

各模块功能大致如下：

1\. **客户端** ：表示以REST方式发起请求的用户，可以包括IDE、MindSpore主页、公众号、Gitee Issue、MindSpore掌中宝等；

2\. **NLP服务** ：基于训练好的模型，如Bert，提供语义理解、内容生成、会话管理等能力；

3\. **[数据服务](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%9C%8D%E5%8A%A1&zhida_source=entity)** ：提供模型版本存储、会话状态、知识图谱的[数据管理](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86&zhida_source=entity)能力；

4\. **监控服务** ：提供基础的生产环境基础设施监控以及业务监控（推理准确率等）能力；

* * *

## **三、加入这个项目，你能收获什么**

1\. **全方位学习MindSpore全栈能力，包括但不限于** ：

  * 使用MindSpore[框架训练](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E6%A1%86%E6%9E%B6%E8%AE%AD%E7%BB%83&zhida_source=entity)、使用ModelZoo预训练模型、使用MindInsight精度调优；
  * 使用XAI验证模型的可解释性、使用MindArmour验证模型的可靠性；
  * 使用Serving在云/端侧部署。模型的构建、发布会基于MLOps流水线(从0到1搭建)；
  * ........



2\. **提升在开源社区中的影响力** ：  
通过参与开源任务，提升在MindSpore开源社区的影响力，并有机会成为易用性SIG核心成员、MindSpore资深开发者（有证书哦），事迹也会记录到易用性SIG的荣誉殿堂......

3\. **获得丰富的奖励：**  
项目中的各任务会根据难度大小，定义相应的积分，获得积分可兑换MindSpore的精美纪念品。

* * *

## 四、如何加入

我们对任务初步作了分解，点击如下表格的链接了解详情。  
欢迎你参与一起实现，或者一起策划新的特性。当然，过程中遇到的任何技术问题，也可以一起探讨。

任务编号| 分类| 需求描述| 任务地址  
---|---|---|---  
1| 环境准备| 提供开发者本地[开发环境](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)配置以及MLOps流水线的环境，交付件包括[配置文件](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&zhida_source=entity)、相关文档；   
1\. 环境配置(MindSpore/Insight、IDE插件配置等)； 2. 初始化代码仓3. MLOps基础流水线搭建| [https://gitee.com/mindspore/community/issues/I50YLS](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/community/issues/I50YLS)  
2| 数据集| 1\. 提供FAQ的[语料库](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E8%AF%AD%E6%96%99%E5%BA%93&zhida_source=entity)   
2\. 提供Pytorch到MindSpore的Q&A语料库 3. 提供TensorFlow到MindSpore的Q&A语料库| [https://gitee.com/mindspore/community/issues/I50YH6](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/community/issues/I50YH6)  
3| 模型| 提供用于MindSpore官网的对话机器人模型，完成问题答案查询的任务，交付件包括：  
1\. 基于Bert的训练代码 2. 基于MindSpore官网文档的调优模型（云侧部署）3. 基于MindSpore文档的调优模型压缩版（端侧部署4. 基于Serving和Lite的 端云推理 5. 端到端的notebook文件（过程需应用相关的AI设计模式） 6. 应用XAI及MindArmour的过程总结 7. 应用MindInsight进行训练调优的过程总结| [https://gitee.com/mindspore/community/issues/I50TPP](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/community/issues/I50TPP)  
4| 模型| 提供用于MindSpore官网的对话机器人模型，完成多轮对话交互的任务，交付件包括：   
1\. 基于Bert的训练代码 2. 基于MindSpore官网文档的调优模型（云侧部署） 3. 基于MindSpore官网文档的调优模型压缩版（端侧部署） 4. 端云模型的推理代码（基于Serving和Lite）| [https://gitee.com/mindspore/community/issues/I50YES](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/community/issues/I50YES)  
5| 集成| IDE集成相关代码和文档| 待模型ready后发布  
  
最后，我们想把知识问答机器人的冠名权交给广大开发者，您可以填写 _[问卷](https://link.zhihu.com/?target=https%3A//jinshuju.net/f/tRRrJ2)_ 留下您喜欢的名称，后续社区会通过评选来确定最终名称。 

期待和您一起来共同打造MindSpore社区的开源知识问答机器人！

  


> 下一篇：[【MindSpore易点通机器人-02】[设计与技术](https://zhida.zhihu.com/search?content_id=197562535&content_type=Article&match_order=1&q=%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%8A%80%E6%9C%AF&zhida_source=entity)选型](https://zhuanlan.zhihu.com/p/507746139)
