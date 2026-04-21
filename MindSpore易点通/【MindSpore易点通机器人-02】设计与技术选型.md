# 【MindSpore易点通机器人-02】设计与技术选型

原文链接：https://zhuanlan.zhihu.com/p/507746139

---

​

目录

2022年3月， _[MindSpore易用性SIG](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig)_ 成立后，很快吸引了不少开发者加入SIG群中进行交流。作为和开发者进行连接的桥梁，易用性SIG的目标是**和开发者共同打造易学易用、灵活高效的AI框架，持续提升MindSpore易用性，助力开发者成功** 。

通过易用性SIG，我们不仅倾听开发者的声音，为开发者提供帮助，也希望能和开发者一起围绕“**MindSpore的学习与实践** ”这个主题，打造一些有趣的[开源项目](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE&zhida_source=entity)。

**MindSpore易点通机器人** 是易用性SIG发起的一个开源项目，**希望能够基于MindSpore框架提供的能力，与社区一起从0到1打造易用性问答机器人，帮助提升MindSpore易用性，** 更多背景请参考 [01-MindSpore易点通机器人介绍](https://zhuanlan.zhihu.com/p/492438570)

本篇文章，我们将梳理需求、设计初始架构、定义MLOps，为后续迭代工作提供输入。

## 一、需求梳理

### **1\. 功能概述**

通过提供面向MindSpore开发者学习和使用的智能机器人服务，接受用户问题查询，自动帮助开发者找到问题答案，覆盖80%以上常见问题，提升社区开发者体验，提升用户满意度。

主要包括三大功能：

  * 问题答案查询：直接回答用户关于MindSpore官网/资料/指南等问题。
  * 多轮对话交互：通过多轮交互引导用户补充问题上下文，最终精准找到答案。
  * 问答推荐：如果没有准确命中的答案，可以给用户推荐相似的3个问题和答案。



### 2\. **需求故事(Epic Story)**

1) 问题答案查询场景 ： 直接回答用户关于MindSpore官网/资料/指南等问题，举例如下所示：

  * 作为`学习型用户`，当我提出关于`学习教程`的问题，期望机器人回答`教程清单`


    
    
    用户提问：[图像分类](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E5%9B%BE%E5%83%8F%E5%88%86%E7%B1%BB&zhida_source=entity)相关的教程有哪些？
    响应：返回图像分类相关的教程地址。

  * 作为`教学/科研型用户`，当我提出`从Pytorch到MindSpore迁移`的问题时，期望机器人回答`API映射信息`


    
    
    用户提问：pytorch torch.chunk api 对应的MindSpore API是什么？
    响应：对应的接口为 mindspore.ops.split，详细地址为(https://mindspore.cn/docs/api/zh-CN/r1.6/api_python/ops/[mindspore](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=3&q=mindspore&zhida_source=entity).ops.Split.html#mindspore.ops.Split)。

  * 作为`生产型用户`，当我提出`精度调试调优的问题`，期望机器人回答`精度调优以及错误定位相关清单`


    
    
    用户提问：精度问题如何定位？ 
    响应：返回精度调优、定位相关的案例以及指南文章。

**2) 多轮对话交互场景** ：通过多轮对话缩小用户问题范围，从而给出答案，举例如下所示：

  * 作为`学习型用户`，当我提出`安装`的问题，期望机器人`交互式获取详情并回答`


    
    
    用户提问：安装mindspore，提示该版本不支持当前平台。
    响应：请输入您的操作系统版本及处理器。
    用户回答：win10、intel处理器。
    响应：请输入您的[显卡](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%98%BE%E5%8D%A1&zhida_source=entity)相关信息。
    用户回答：AMD[集成显卡](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E9%9B%86%E6%88%90%E6%98%BE%E5%8D%A1&zhida_source=entity)。
    响应：请输入您安装的MindSpore版本。
    用户回答：mindspore_gpu 1.6.0。
    响应：mindspore当前不支持 AMD显卡，请安装CPU版本。

  * 作为`教学科研型用户`，当我提出`MindInsight无法运行`问题，期望机器人`交互式获取详情并回答`


    
    
    用户提问：请问在Ubuntu上安装了MindInsight运行时提示无法使用，应该怎么解决？
    响应：请问使用的是什么操作系统？
    用户回答：Ubuntu。
    响应：请输入提示的错误信息。
    用户回答：提示[8080端口](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=8080%E7%AB%AF%E5%8F%A3&zhida_source=entity)无法使用。
    响应：8080端口可能已被占用，建议切换启动端口，如 mindinsight start --port 8080 --summary-base-dir xxx。

  * 作为`生产型用户`，当我提`如何部署`问题，期望机器人`交互式获取详情并回答`


    
    
    用户提问：模型如何部署到生产环境？
    响应：请问部署的目标硬件是什么？
    用户回答：GPU。
    响应：请问部署的[操作系统](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=3&q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&zhida_source=entity)是什么？
    用户回答：Windows。
    响应：请问是离线还是在线部署？
    用户回答：离线。
    响应：给出MindSpore Lite Windows版本下载地址以及相应的指南。

**3) 问答推荐场景** ：推荐相关的问题和答案
    
    
    用户提问：[龙芯](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E9%BE%99%E8%8A%AF&zhida_source=entity)支持的版本？ 
    响应：您可能想了解以下问题： 
    1. 如何在青松服务器上编译MindSpore？ 
    2. 如何在[海光服务器](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%B5%B7%E5%85%89%E6%9C%8D%E5%8A%A1%E5%99%A8&zhida_source=entity)上编译MindSpore？

* * *

## 二、 [架构设计](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1&zhida_source=entity)

该部分从四个主要视图([逻辑视图](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E9%80%BB%E8%BE%91%E8%A7%86%E5%9B%BE&zhida_source=entity)、开发视图、部署视图和运行视图)展开，指导后续实现。

### 1\. 逻辑视图

主要包括 [业务模型](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E4%B8%9A%E5%8A%A1%E6%A8%A1%E5%9E%8B&zhida_source=entity)、技术模型（技术选型）、API设计、[数据模型](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B&zhida_source=entity)。

**1) 业务模型**  
逻辑架构如下图所示，NLP服务接受来自多客户端的请求，返回生成的应答内容。

逻辑架构

主要模块如下所示 ：

  * 客户端：表征以REST方式发起请求的用户，包括IDE、MindSpore主页、公众号，掌中宝、Gitee等。
  * NLP服务（业务层）：提供语义理解、内容生成、会话管理等能力。
  * [数据服务](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%9C%8D%E5%8A%A1&zhida_source=entity)（数据层）：提供模型版本存储、会话状态、知识图谱的[数据管理](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86&zhida_source=entity)能力。
  * 监控服务：提供基础的生产环境、基础设施监控以及业务[监控能力](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E7%9B%91%E6%8E%A7%E8%83%BD%E5%8A%9B&zhida_source=entity)。



**2) 技术模型**

我们将[NLP服务](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=3&q=NLP%E6%9C%8D%E5%8A%A1&zhida_source=entity)作为独立的[微服务](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E5%BE%AE%E6%9C%8D%E5%8A%A1&zhida_source=entity)在云上以容器形式部署运行，同时利用云侧提供的服务，完成NLP服务的监控、高可用和安全性保障。

技术模型

主要技术方案：

**客户端**

  * IDE：基于PyCharm，提供UI节目。
  * 主站：基于Reactjs实现对话框；
  * Gitee：通过Webhook方式对接；



**NLP服务**

  * 基于Bert模型完成意图识别、内容生成。
  * REST API基于Serving实现。



**数据服务**

  1. 模型版本：数据和模型打包在一起，通过SWR管理。
  2. 会话状态：基于Redis管理。
  3. 知识图谱：基于Gauss数据库管理。



**监控服务**

基于APM实现基础监控，通过自定义指标实现业务监控。

**3) API设计**

针对问答过程，客户端使用REST API完成提问及获取答案： 
    
    
    GET /query
    Request：{
        question: string, 
        tag: string  //标记问题的类型或者领域
    }
    Response: 
    {
        question: string, 
        answer: string, 
        tag: string, 
        ref: string  //承载url链接，针对某些问答可能需要给出参考链接的场景
    }

**4) 数据模型**

数据模型主要考虑[数据集](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)的格式。数据集使用csv格式进行存储，主要包含以下字段：文档内容，流程，标签。

  * content：文档内容，为文档所有的文字内容，以便匹配用户的搜索关键词。
  * process：流程主要是tag信息，用于聚类，反问；当前定义流程为（不建议自行增加流程）：安装，数据处理，代码编写，编译，调试，调优，推理。
  * tag：内容可能归属于多个tag（领域），环境也属于tag信息，只是从不同角度进行定义，如系统信息（Windows、Mac），[芯片架构](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E8%8A%AF%E7%89%87%E6%9E%B6%E6%9E%84&zhida_source=entity)（CPU、GPU），AI领域（NLP、CV）。



样例

title| link| content| process| tag  
---|---|---|---|---  
id| [http://...](https://link.zhihu.com/?target=http%3A//...).| Conda方式安装MindSpore GPU版本| 安装| GPU、Linux  
  
### 2\. 开发视图

开发视图通常承载代码结构、流水线设计等内容。

**1) 代码结构**

当前[MindSpore易点通机器人](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot)的代码仓按照如下目录组织：
    
    
    代码仓结构：
    data：源数据目录
    src: 源码目录
       - Data：数据处理
       - Train: 训练代码
       - inference ：[推理代码](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%8E%A8%E7%90%86%E4%BB%A3%E7%A0%81&zhida_source=entity)
    tests：自动化测试代码
    scripts: CI/CD相关脚本
    infra：基础设施即代码的配置
    workflow：Argo workflow的[配置文件](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&zhida_source=entity)
    notebooks：Jupyter notebook，记录案例教程
    Jenkinsfile：CI/CD流水线配置文件
    Dockerfiles：镜像打包配置文件

**2) 流水线设计**

MLOps流水线设计

**流水线分类**

  * 构建（CI/CD）流水线：包含代码规范检查、单元测试、[接口测试](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E6%8E%A5%E5%8F%A3%E6%B5%8B%E8%AF%95&zhida_source=entity)，完成代码打包和自动化部署。
  * 训练（ CT ）流水线：[数据处理](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=3&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)（生成新语料库）、训练模型、[评估模型](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E8%AF%84%E4%BC%B0%E6%A8%A1%E5%9E%8B&zhida_source=entity)、发布新版本模型。



**构建元素**

  * 模型及推理代码包（容器）。
  * 语料数据集（CSV/MindRecord）。



**版本管理**

  * 数据版本管理：可分为原始数据以及语料库数据（mindrecord格式），使用OBS进行存储及版本管理。
  * 模型版本管理：同样通过OBS托管模型，也可以考虑和推理代码打包在一起发布到到镜像仓库进行版本管理。



### 3\. 部署视图

对于NLP服务云侧部署，参考微服务的[云原生](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E4%BA%91%E5%8E%9F%E7%94%9F&zhida_source=entity)部署方式，最大程度的利用云服务来保障系统的高可用，弹性，并降低维护成本。

部署视图

云侧部署：

  * 极简部署：[负载均衡](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1&zhida_source=entity)（ELB），NLP服务器（CCE 容器），状态存储（Redis）。
  * 高可用：NLP服务多AZ部署，Redis主从。
  * 安全性：NLP服务只接收ELB[安全组](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E5%AE%89%E5%85%A8%E7%BB%84&zhida_source=entity)请求，Redis只接收NLP服务安全组请求。
  * [可维护性](https://zhida.zhihu.com/search?content_id=200964640&content_type=Article&match_order=1&q=%E5%8F%AF%E7%BB%B4%E6%8A%A4%E6%80%A7&zhida_source=entity)：监控指标聚合到APM服务，NLP服务上传自定义指标到APM。



### 4\. 运行视图

运行视图介绍NLP服务以及在端侧IDE运行时的进程详情。

运行视图

NLP服务以及客户端的运行视图比较简单，NLP服务和HTTP服务运行在一个进程中，而IDE客户端则作为插件和IDE进程运行在一起。

## 三、总结

该篇文章主要介绍了设计的4类主要视图：逻辑视图、开发视图、部署视图、运行视图。

完成初步设计后，我们接下来就可以参考《[03-迭代0：机器学习项目开始前要做哪些准备工作？](https://zhuanlan.zhihu.com/p/500536548)》开始迭代0

  


> 上一篇：[【MindSpore易点通机器人-01】你也许见过很多知识问答机器人，但这个有点不一样](https://zhuanlan.zhihu.com/p/492438570)  
> 下一篇：[【MindSpore易点通机器人-03】迭代0的准备工作](https://zhuanlan.zhihu.com/p/509494866)
