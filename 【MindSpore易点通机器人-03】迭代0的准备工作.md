# 【MindSpore易点通机器人-03】迭代0的准备工作

原文链接：https://zhuanlan.zhihu.com/p/509494866

---

​

目录

在上一篇[【MindSpore易点通机器人-02】[设计与技术](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E8%AE%BE%E8%AE%A1%E4%B8%8E%E6%8A%80%E6%9C%AF&zhida_source=entity)选型](https://zhuanlan.zhihu.com/p/507746139)，我们介绍了MindSpore易点通机器人整体的设计，本篇将会向大家介绍进入迭代0所需要的准备工作。在[03-迭代0：机器学习项目开始前要做哪些准备工作？](https://zhuanlan.zhihu.com/p/500536548)中，提到了迭代0中需要准备的内容：

  1. 选择计算资源
  2. 搭建[开发环境](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)
  3. 建立[版本控制](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6&zhida_source=entity)
  4. 搭建机器学习流水线
  5. 安装Jupyter Notebook（包含在开发环境搭建中）



从MindSpore易点通机器人这个项目出发，按照实际的需求依次完成以上准备内容。

## 选择计算资源

在机器学习的流程中，开发者需要在本地进行网络的开发调试，在CI上完成集成和部署，在训练流水线上完成持续训练和模型验证，在生产环境完成部署推理。因此，在这些不同的环境中，可能需要不同类型的计算资源。

  1. **开发环境** ：开发者需要在开发环境运行MindSpore GPU版本来完成模型的开发和功能调试，因此在计算资源上通常需要Nvidia的GPU支持。
  2. **CI/CD** ：运行MindSpore易点通机器人的集成和部署任务，普通的x86 CPU资源即可，在云上可以考虑[容器集群](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E5%AE%B9%E5%99%A8%E9%9B%86%E7%BE%A4&zhida_source=entity)，如华为云的CCI/CCE。
  3. **持续训练流水线（CT）** ：MLOps流水线需要完成模型的训练、验证等任务，本地运行完整训练流水线选择消费级GPU，云侧使用ModelArts提供的昇腾或者GPU集群，并且需要多卡完成分布式训练以加速训练过程。
  4. **生产环境** ：需要[昇腾310](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E6%98%87%E8%85%BE310&zhida_source=entity)完成推理，可以部署在ModelArts或者CCE上。



训练和部署的具体资源数量可以根据实际的测试验证来选择规格。

## 搭建开发环境

开发环境准备涉及Python环境准备、MindSpore开发相关依赖安装、IDE安装配置等工作。其中Python的环境可以基于Miniconda完成，可以很方便的完成Python版本切换。对于MindSpore框架的开发，当前也有一系列自动化的脚本，帮助开发者快速完成安装配置工作。其它的依赖及IDE安装配置，都可以在[MindSpore易点通机器人代码仓](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot)的`README`找到。

开发所需的具体的组件及作用如下：

  1. MindSpore GPU版本：用于本地训练和调试模型。
  2. MindInsight：用于本地的精度调优。
  3. XAI：模型可解释性评估。
  4. MindArmour：可靠性和安全性工具。
  5. Serving：云侧部署工具。
  6. Lite：端侧（IDE）部署工具。
  7. jupyter Notebook：用于前期的技术探索。
  8. MindSpore Dev Toolkit：基于PyCharm IDE，增加智能编码，搜索等功能。



## 建立版本控制

版本控制包括三个部分，一个代码的版本控制，这部分通过gitee来完成，另外的包括数据和模型的版本控制，我们选择[对象存储](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A8&zhida_source=entity)服务来承载，保证数据和模型的一对一的一致性，同时，对象存储服务也可以管理数据和模型的[元数据](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E5%85%83%E6%95%B0%E6%8D%AE&zhida_source=entity)信息（如版本等）。MindSpore易点通机器人项目使用了华为云的对象存储服务（OBS）作为训练数据和模型的版本管理工具。OBS是一个基于对象的存储服务，适合训练数据这种大规模但碎片化的类型的[数据存储](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8&zhida_source=entity)，同时不用担心扩容和数据可靠性问题。

OBS的使用很简单，通过华为云的Console选择OBS服务后，直接创建标准的OBS桶即可。训练流水线使用的Argo工具，可以在[配置文件](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&zhida_source=entity)中直接对接OBS读写训练数据和模型。在OBS桶的概览中可以看到相关的连接信息，结合华为云用户的AK/SK完成读写操作。

## 搭建机器学习流水线

MindSpore易点通机器人的MLOps流水线包含两个部分，CI/CD流水线以及持续训练流水线，其中本地我们基于[WSL+Ubuntu+Minikube搭建K8S集群](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot/blob/master/docs/k8s.md)，在K8S集群上构建CI/CD流水线（Jenkins）以及持续训练流水线（参考MindSpore DX Sig的 [MLOps-Tool](https://link.zhihu.com/?target=https%3A//github.com/iambowen/MLops-Tools/)项目）。

整个MLOps流水线运作的流程如下：

  * **训练流水线**


  1. OBS上上传原始数据，触发训练。
  2. 训练首先[处理数据](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E5%A4%84%E7%90%86%E6%95%B0%E6%8D%AE&zhida_source=entity)，将处理完的数据上传到OBS桶中。
  3. 而后开始模型的训练，使用刚处理完的数据作为输入，产出模型，同样归档OBS桶。
  4. 进行模型评估，并行完成精度、可解释性和可靠性评估。
  5. 给出总结，并触发CI/CD流水线。


  * **CI/CD流水线：逐次执行如下任务**


  1. [代码检查](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E4%BB%A3%E7%A0%81%E6%A3%80%E6%9F%A5&zhida_source=entity)。
  2. 单元测试。
  3. [接口测试](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E6%8E%A5%E5%8F%A3%E6%B5%8B%E8%AF%95&zhida_source=entity)（部署到云上NLP服务的Rest接口）。
  4. 从OBS下载本次训练的模型，打包成Docker镜像。
  5. 完成生产环境部署。



在代码仓库的[Workflow目录](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot/blob/master/workflows/examples/mnist-train-eval.yaml)给出了当前初步的训练例子的编排配置，使用`argo summbit mnist-train-eval.yaml`提交例子训练任务，从Argo的UI中看到运行的结果。

持续训练流水线完成时，会触发[持续集成](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=1&q=%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90&zhida_source=entity)和部署流水线，由[代码仓](https://zhida.zhihu.com/search?content_id=201353500&content_type=Article&match_order=3&q=%E4%BB%A3%E7%A0%81%E4%BB%93&zhida_source=entity)的Jenkinsfile基于[Pipeline as Code](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot/blob/master/Jenkinsfile)完成初始流水线的构建。

## **总结**

迭代0主要看重端到端流程的打通，以保证后续功能的持续交付。迭代0的准备工作完成后，我们就可以开始下一步的训练数据准备工作了。

  


> 上一篇：[【MindSpore易点通机器人-02】设计与技术选型](https://zhuanlan.zhihu.com/p/507746139)  
> 下一篇：[【MindSpore易点通机器人-04】MLOps 环境搭建过程](https://zhuanlan.zhihu.com/p/514996638)
