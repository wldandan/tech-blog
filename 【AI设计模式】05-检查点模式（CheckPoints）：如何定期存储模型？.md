# 【AI设计模式】05-检查点模式（CheckPoints）：如何定期存储模型？

原文链接：https://zhuanlan.zhihu.com/p/521311168

---

​

目录

在前一篇文章[《数据处理-Eager模式》](https://zhuanlan.zhihu.com/p/517450571)中分享了数据处理-Eager模式，那么在模型训练时，有哪些设计模式可以使用呢？在数据库领域，为了防止执行时间较长的存储过程失败重新执行，会将中间的过程状态以检查点的形式持续记录下来，每次失败时不需要重头执行，而是加载最近的检查点，继续执行，避免浪费时间。和存储过程类似，模型的训练时间会更长，如果缺乏一定的可靠性机制，过程中一旦失败，就需要重头开始训练，浪费时间较多。因此，需要实现类似的机制来保证可靠性问题，这种机制被称为**检查点（CheckPoints）模式** 。

AI设计模式总览

## **模式定义**

**[检查点模式](https://zhida.zhihu.com/search?content_id=203979194&content_type=Article&match_order=1&q=%E6%A3%80%E6%9F%A5%E7%82%B9%E6%A8%A1%E5%BC%8F&zhida_source=entity) （CheckPoints）**是指通过周期性（迭代/时间）的保存模型的完整状态，在模型训练失败时，可以从保存的检查点模型继续训练，以避免训练失败时每次都需要从头开始带来的训练时间浪费。检查点模式适用于模型训练时间长、训练需要提前结束、fine-tune等场景，也可以拓展到异常时的断点续训场景。

## **问题**

  1. **训练耗时的网络在训练过程中失败，从头开始训练的代价高** ：对于层数比较深的神经网络，或者需要大规模训练数据的模型，训练的时间会很长。因为有更多的参数以及更多的数据样本需要处理。比如对于VGG16的网络，[cifar-10](https://zhida.zhihu.com/search?content_id=203979194&content_type=Article&match_order=1&q=cifar-10&zhida_source=entity)的数据集，普通的NVIDIA显卡训练需要3-4小时；一旦过程中失败，需要重头开始训练，时间成本高。
  2. **训练时间越长，精度可能不发生变化，或者产生过拟合的现象。** 这种场景时，提前结束（early stopping）获得中间的模型状态收益会更高。
  3. Fine-Tune时，通常需要最终模型前面的一些模型状态进行基础上进行调优，这样可以更好的针对新数据进行训练，获得更好的泛化性。



## **解决方案**

**在每轮训练结束时，都保存当前的模型状态作为检查点，如果下轮训练失败时，可以从这个检查点模型继续训练。** 和训练完成导出的模型（以神经网络为例，最终的模型包含权重、激活函数以及隐藏层信息）相比，这个中间模型状态需要额外的轮、当前的批量计数等信息，以保证基于这个中间模型继续训练。通常这个中间模型被称为检查点（CheckPoints）。检查点的模型状态中通常不包括学习率，因为训练过程中它可能会动态调整。

如果在每个批量数据训练完，权重更新后都保存检查点，中间模型的数量和占用的空间会非常大。所以实践中通常会在每轮结束后保存检查点，或者保留最近的几个检查点。

## **案例**

AI框架通常都提供了模型训练的检查点保存能力。在[MindSpore](https://zhida.zhihu.com/search?content_id=203979194&content_type=Article&match_order=1&q=MindSpore&zhida_source=entity)中，通过训练API提供了**[ModelCheckPoint](https://zhida.zhihu.com/search?content_id=203979194&content_type=Article&match_order=1&q=ModelCheckPoint&zhida_source=entity)** 和**CheckpointConfig** 模块来帮助开发者保存模型训练过程中的检查点。MindSpore提供了三种检查点保存策略，包括**直接保存、周期保存** （迭代次数或者训练时长）、和**异常保存** （在训练失败的异常情况下保存的策略）。

> 说明：检查点文件是一个二进制文件，存储了所有训练参数的值；且检查点的实现上采用了[Protocol Buffers](https://zhida.zhihu.com/search?content_id=203979194&content_type=Article&match_order=1&q=Protocol+Buffers&zhida_source=entity)机制，与开发语言、平台无关，具有良好的可扩展性。

在此，我们重点介绍下如何在MindSpore中周期性保存模型状态、以及在异常情况下保存故障点的模型状态。

### 周期保存

**1）迭代次数方式保存**

下面的MindSpore代码片段展示了使用迭代次数配置检查点保存策略，以及在模型训练时通过回调的方式应用保存策略。训练开始后，会每隔1785个step保存一次检查点模型，并最多保留10个中间模型，模型的名称格式为`checkpoint_lenet-1_1875.ckpt`。
    
    
     from mindspore.train.callback import ModelCheckpoint, CheckpointConfig 
        
     # 设置模型保存参数，设置模型保存的策略，如本例中设置最多保存10个checkpoints，每隔1875个step保存一次 
        
     config_ck = CheckpointConfig(save_checkpoint_steps=1875, keep_checkpoint_max=10) 
        
     # 应用模型保存参数 
        
     ckpoint = ModelCheckpoint(prefix="checkpoint_lenet", config=config_ck) 
        
     #通过回调的方式配置在模型训练的过程中 
        
     model.train(epoch_size, ds_train, callbacks=[ckpoint_cb]) 
          

加载CheckPoint可以通过`load_checkpoint`和`load_param_into_net`方法来完成，如下面的代码，通过`load_checkpoint`方法从保存好的checkpoint中加载网络的参数，再通过`load_param_into_net`将参数导入到具体的网络实例中，方便后面续训或者评估。
    
    
    from mindspore import load_checkpoint, load_param_into_net
    # 加载已经保存的用于测试的模型
    param_dict = load_checkpoint("checkpoint_lenet-1_1875.ckpt")
    # 加载参数到网络中
    load_param_into_net(net, param_dict)

完整的代码可以参考[[1]](https://link.zhihu.com/?target=https%3A//mindspore.cn/tutorials/zh-CN/r1.7/beginner/quick_start.html)中的案例。

**2）周期时间方式保存**

时间策略提供了按照秒和分钟配置参数，如下面的代码，每隔30秒保存一个CheckPoint文件，每隔3分钟保留一个CheckPoint文件。
    
    
    from mindspore import CheckpointConfig
    
    # 每隔30秒保存一个CheckPoint文件，每隔3分钟保留一个CheckPoint文件
    config_ck = CheckpointConfig(save_checkpoint_seconds=30, keep_checkpoint_per_n_minutes=3)

### **异常保存**

如果模型较大，通常会减少梳理保留的检查点模型，间隔的时间会拉长。如盘古大模型的检查点保存间隔在4-5小时，如果在两个检查点之间失败，那么从上个检查点重新训练的时间损失会比较大。MindSpore在1.7版本扩展了检查点功能，提供**断点续训能力** ，保证在训练异常时触发检查点，保证下次可以从发生故障时的模型状态继续训练，训练时间无损失。引入断点续训功能，只需在策略配置时增加“exception_save=True”的参数即可。
    
    
    from mindspore import ModelCheckpoint, CheckpointConfig
    # 配置断点续训功能开启
    config_ck = CheckpointConfig(save_checkpoint_steps=32, keep_checkpoint_max=10, exception_save=True)

## **总结**

**检查点（CheckPoints）模式** 最大的作用在于**保证了模型训练的可靠性** ，同时也可以让开发者**更容易的做早停** 。断点续训能力对于大模型的价值较大，异常状态下续训无时间损失，检查点模式也有利于转移学习时做fine-tune，这也是我们下一个要介绍的模式。

## **参考资料**

[1] MindSpore完整案例：[https://mindspore.cn/tutorials/zh-CN/r1.7/beginner/quick_start.html](https://link.zhihu.com/?target=https%3A//mindspore.cn/tutorials/zh-CN/r1.7/beginner/quick_start.html)

[2] MindSpore模型保存：[https://gitee.com/mindspore/docs/blob/master/tutorials/source_zh_cn/advanced/train/save.ipynb](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/docs/blob/master/tutorials/source_zh_cn/advanced/train/save.ipynb)

[3] 机器学习设计模式：[https://www.oreilly.com/library/view/machine-learning-design/9781098115777/](https://link.zhihu.com/?target=https%3A//www.oreilly.com/library/view/machine-learning-design/9781098115777/)

  


> 上一篇：[【AI设计模式】04-数据处理-Eager模式](https://zhuanlan.zhihu.com/p/517450571)  
> 下一篇：[【AI设计模式】06-分布式并行模式（Distribution Strategy）](https://zhuanlan.zhihu.com/p/534570078)
