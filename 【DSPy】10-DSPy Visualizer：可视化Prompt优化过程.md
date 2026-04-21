# 【DSPy】10-DSPy Visualizer：可视化Prompt优化过程

原文链接：https://zhuanlan.zhihu.com/p/714277212

---

​

目录

[DSPy](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)是开源社区比较有影响力的大模型提示词、参数调优仓库，使得用户以一种编程的方式，调整和优化大模型模板和参数。DSPy的设计思想借鉴了[Torch](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=Torch&zhida_source=entity)，使得Torch的使用者可以很快的理解DSPy的使用方式。

使用Torch时，研究人员往往会使用[Tensorboard](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=Tensorboard&zhida_source=entity)等可视化工具来直观的查看模型的训练情况，那么是否有适配DSPy优化过程的可视化工具呢？答案是Yes，本文将首先介绍带有DSPy优化器的优化示例，然后使用 [langwatch](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=langwatch&zhida_source=entity) 仓库提供的DSPy Visualizer 查看具体的优化过程，包括显示每个优化步骤的Predictior、Example、大模型调用内容等。

## 安装

在使用DSPy Visualizer之前，需要确保已经安装了langwatch 以及 dspy 的python包。
    
    
    pip install dspy-ai==2.4.12
    pip install langwatch

## 编写DSPy程序

本示例将编写简单的优化器使用示例，如图-1所示，DSPy的优化流程需要准备数据集、程序主体、优化器以及衡量指标，然后在固定数据集合衡量指标的情况下，调整程序主体和算法以达到优化的目的，具体步骤如下  


图-1 DSPy 优化流程

  
  
1）获取数据集，本示例中通过导入方式获取训练集和测试集。
    
    
    import dspy
    from get_dataset import custom_trainset as train_set
    from get_dataset import custom_testset as test_set

2）导入衡量指标以及优化器，本示例中使用基本的 [BootstrapFewShot](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=BootstrapFewShot&zhida_source=entity)优化器。
    
    
    from dspy.datasets.gsm8k import gsm8k_metric
    from dspy.teleprompt import BootstrapFewShot

3）定义大模型配置，本示例中使用了 [ollama](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=ollama&zhida_source=entity) 启动的本地llama3模型服务。
    
    
    #   定义并设置大模型
    model_name = 'llama3'
    lm = dspy.OllamaLocal(model=model_name)
    dspy.settings.configure(lm=lm)

4）定义程序主体（Module），程序主体包含初始化（__init__）以及推理（forward）两部分逻辑，使用了dspy.[ChainOfThought](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=ChainOfThought&zhida_source=entity)思维模式实现推理过程。
    
    
    class CoT(dspy.Module):
        def __init__(self):
            super().__init__()
            self.prog = dspy.ChainOfThought("question -> answer")
    
        def forward(self, question):
            return self.prog(question=question)
    
    cot = CoT()

5）配置优化器参数，通过调整优化器参数，可以调整最终的提示词效果。
    
    
    config = dict(max_bootstrapped_demos=4, max_labeled_demos=4, max_rounds=2)
    
    #   Optimize! Use the `gsm8k_metric` here. In general, the metric is going to tell the optimizer how well it's doing.
    teleprompter = BootstrapFewShot(metric=gsm8k_metric, **config)
    

至此，本示例已经定义好了优化之前的所有准备工作。

## 优化与可视化

为了实现可视化DSPy的优化过程，本实例引入langwatc包，langwatc需要在优化开始前初始化，如下面代码所示。

DSPy Visualizer的可视化服务可以通过在线的方式访问，也可以通过本地部署安装[Docker](https://zhida.zhihu.com/search?content_id=246858247&content_type=Article&match_order=1&q=Docker&zhida_source=entity)镜像的方式使用。
    
    
    import langwatch
    langwatch.login()
    langwatch.dspy.init(experiment="test", optimizer=teleprompter)
    optimized_cot = teleprompter.compile(cot, trainset=train_set)

至此，本示例代码已完成，接下来需要做好可视化准备工作，可视化准备主要包含启动可视化程序和获取langwatch的API-Key。

### 在线访问方式

由于langwatch已经提供了可视化的在线服务，因此在线方式无需启动可视化程序，直接获取API-Key即可，如下步骤所示：  
1）登录[langwatch在线dashboard](https://link.zhihu.com/?target=https%3A//app.langwatch.ai/authorize) ，获取API-Key。  


  
2）本地运行程序，根据提示输入API-Key，并在 dashboard查看结果。  
本地界面：  


  
dashboard界面：  


  
如上界面展示的为训练两步得到的可视化数据，可以看出在第一步时优化器得到了3个demo，第二步时，优化器得到了4个demo。  
也可以点击具体的 Predictors、Examples、LLM Calls（图中LLM Call无信息，因为使用了LLama3本地访问，如果是OpenAI接口则有数据显示）查看详细信息。  
如下图所示，显示了变量名为 prog 的Predictor类的信息，包括输入输出、得到的示例等。  


  
下图显示了在优化过程中所测试的示例，以及得到的分数，可以看出前两个示例预测结果格式不符合要求，判定为错误，后三个示例预测结果正确。

### 本地访问

本地启动可视化程序需要安装Docker，然后做如下3步动作：

1）下载github langwatch 仓库并 复制`langwatch/.env.example`中的文件内容到`langwatch/.env` 。

2）运行`docker compose up --build` 。·

3）打开LangWatch通过网址[http://localhost:3000](https://link.zhihu.com/?target=http%3A//localhost%3A3000/)

其他步骤与在线访问类似。

## 相关链接

  1. [https://github.com/langwatch/langwatch](https://link.zhihu.com/?target=https%3A//github.com/langwatch/langwatch)
  2. [https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)
  3. 代码：[ziqi-jin/dspy-examples: Running DSPy examples by llama3 (github.com)](https://link.zhihu.com/?target=https%3A//github.com/ziqi-jin/dspy-examples)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】09-丝分缕解！带你了解DSPy核心模块的源码实现之Optimizer类](https://zhuanlan.zhihu.com/p/709694815)  
> 下一篇：[【DSPy技术洞察】11-DSPy和Langchain的无缝集成](https://zhuanlan.zhihu.com/p/714868095)
