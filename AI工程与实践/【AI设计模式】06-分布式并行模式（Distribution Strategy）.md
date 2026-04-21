# 【AI设计模式】06-分布式并行模式（Distribution Strategy）

原文链接：https://zhuanlan.zhihu.com/p/534570078

---

​

目录

在前一篇文章分享了[《检查点模式（CheckPoints）》](https://zhuanlan.zhihu.com/p/521311168)，本篇将带大家了解分布式并行模式。互联网应用通常面临的最大挑战是流量，而传统的[单体架构](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%8D%95%E4%BD%93%E6%9E%B6%E6%9E%84&zhida_source=entity)无法应对流量的变化，因此需要采用分布式的[微服务架构](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84&zhida_source=entity)进行改造。同理，在AI领域，当前最大的挑战也是数据量和模型参数规模不断的增长，单卡的资源要么无法支撑训练，要么效率较低。因此，也需要采用分布式的方式，将负载分摊到不同的硬件设备上来应对[大数据](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%A4%A7%E6%95%B0%E6%8D%AE&zhida_source=entity)/参数规模化，从而提升模型训练效率。这种通过分布式方式解决模型训练效率的模式被称为**分布式并行模式** 。

## **模式定义**

**分布式并行模式** 就是将训练的过程扩展到多个worker，并借助缓存、硬件加速、并行等技术手段，加速模型训练效率，通常有**[数据并行](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%B9%B6%E8%A1%8C&zhida_source=entity)** 和**模型并行** 以及**混合并行** 的方式。

## **问题**

AI领域的模型参数和训练的数据规模正呈现越来越大的趋势。从2018年Google发布Bert模型开始，参数规模从几亿到百亿、千亿（如华为盘古大模型）甚至万亿规模（Google Switch Transformer）。同时，训练数据的规模也越来越大，以GPT系列为例：

  1. GPT-1是上亿规模的参数量，数据集使用了1万本书的BookCorpus，25亿单词量；
  2. [GPT-2](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=GPT-2&zhida_source=entity)参数量达到了15亿规模，其中数据来自于互联网，使用了800万在Reddit被链接过的网页数据，清洗后越40GB（WebText)；
  3. [GPT-3](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=GPT-3&zhida_source=entity)参数规模首次突破百亿，数据集上将语料规模扩大到570GB的CC数据集(4千亿词)+WebText2(190亿词)+BookCorpus(670亿词)+[维基百科](https://www.zhihu.com/search?q=%E7%BB%B4%E5%9F%BA%E7%99%BE%E7%A7%91&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2221187242%7D)(30亿词)。



> 引自：[2022，大模型还能走多远](https://link.zhihu.com/?target=https%3A//www.51cto.com/article/697186.html)

对于如此大规模的模型及训练数据，使用单卡的方式完全无法完成训练。以GPT-3模型训练为例 ，使用 8 张 V100 [显卡](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%98%BE%E5%8D%A1&zhida_source=entity)，训练时长预计要 36 年， 512 张 V100显卡 ，训练时间接近 7 个月，而1024 张A100的训练时长可以减少到 1 个月。时间越长，意味着成本越高，大模型的训练可能是普通人难以负担的。因此，**需要[分布式并行](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=4&q=%E5%88%86%E5%B8%83%E5%BC%8F%E5%B9%B6%E8%A1%8C&zhida_source=entity)的方式来增强算力、加速[数据处理](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)和模型训练**。 

## **解决方案**

在模式定义中提到分布式并行可以分为数据并行、模型并行、混合并行三种方式。其中**数据并行** 是将训练的样本数据分成不同的批量，在不同的硬件设备上运行相同的模型进行训练，而后[聚合梯度](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E8%81%9A%E5%90%88%E6%A2%AF%E5%BA%A6&zhida_source=entity)，更新参数，进行下一轮的训练。**数据并行** 根据通信的方式又可以分为**同步和异步** （参数服务器）方式。**[混合并行](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=3&q=%E6%B7%B7%E5%90%88%E5%B9%B6%E8%A1%8C&zhida_source=entity)** 指数据并行和模型并行结合的方式。

### **数据并行**

如下图，假设我们使用8张GPU卡或者[昇腾](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%98%87%E8%85%BE&zhida_source=entity)的NPU卡来训练图片分类的模型，训练的批量为160，那么每张卡上面分到的批量数据（min-batch）为20，每张卡基于样本数据完成训练。因为各张卡上处理的数据样本不同，所以获得的梯度会有些差别。因此，需要通过对梯度进行聚合（求和、均值）等计算来保持和单卡训练相同的结果，最后再更新参数。

数据并行依赖于集合通信的操作，上面的例子中使用到了Broadcast和AllReduce通信原语，其中Broadcast将数据分发到不同的卡，而AllReduce操作则完成不同卡上的[梯度聚合](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%A2%AF%E5%BA%A6%E8%81%9A%E5%90%88&zhida_source=entity)操作。

除了BroadCast和AllReduce算子外，还有AllGather（将每张卡的输入Tensor在第0维度上进行拼接，最终每张卡输出是相同的数值）、ReduceScatter（将每张卡的输入先进行求和，然后在第0维度按卡数切分，将数据分发到对应的卡上）、AlltoAll（将输入数据在特定的维度切分成特定的块数，并按顺序发送给其它rank，同时从其它rank接收输入，按顺序在特定的维度拼接数据）等[算子](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=2&q=%E7%AE%97%E5%AD%90&zhida_source=entity)，方便多种数据并行的需求。

上面提到的都是**同步的方式** 完成数据并行的训练，即不同分片上的梯度计算完成后会汇总一起更新模型参数，然后再启动下一轮的训练，而异步的方式是模型的权重和参数异步更新，就是单卡上的worker不需要等待其它worker，梯度聚合更新模型参数后即可开始下一轮训练。**[异步方式](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%BC%82%E6%AD%A5%E6%96%B9%E5%BC%8F&zhida_source=entity)** 的典型应用是参数服务器架构，即有一个参数[服务器管理](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AE%A1%E7%90%86&zhida_source=entity)模型权重，并根据worker训练的梯度，持续更新模型，并推送新模型，触发worker进行下一轮训练。

  


相比同步的数据并行模式，[异步并行](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%BC%82%E6%AD%A5%E5%B9%B6%E8%A1%8C&zhida_source=entity)的吞吐会高一些，因为不用等待训练比较慢的worker，但如果过程中有worker失败，部分数据可能没有训练，导致有时候无法准确的知道处理了多少个epoch数据。配合[检查点模式](https://zhuanlan.zhihu.com/p/521311168)，通过定期保存或者利用[断点续训](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%96%AD%E7%82%B9%E7%BB%AD%E8%AE%AD&zhida_source=entity)的功能，避免这样的问题。[参数服务器](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=4&q=%E5%8F%82%E6%95%B0%E6%9C%8D%E5%8A%A1%E5%99%A8&zhida_source=entity)的具体实现也可以使用同步的方式。

### **模型并行**

对于GPT-3上千亿、万亿参数的模型，数据并行无法解决模型过大，单卡无法承载的问题。因此，需要通过层间并行（模型以层为单位切分到多个硬件设备）或者层内并行（每层的模型参数切分到多个硬件设备）的模型[并行方式](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%B9%B6%E8%A1%8C%E6%96%B9%E5%BC%8F&zhida_source=entity)来解决。

如上图，层间模型并行，每卡执行的[网络模型](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E7%BD%91%E7%BB%9C%E6%A8%A1%E5%9E%8B&zhida_source=entity)存在差异，如P0卡上执行embedding层，p1，p2，p3分别计算后面的[全连接层](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%85%A8%E8%BF%9E%E6%8E%A5%E5%B1%82&zhida_source=entity)，而层内模型并行不会改变[网络结构](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E7%BD%91%E7%BB%9C%E7%BB%93%E6%9E%84&zhida_source=entity)，而是将每层的模型参数切分到不同的设备上实现并行，如右图中把每层的[矩阵](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E7%9F%A9%E9%98%B5&zhida_source=entity)计算切分到p0-p3卡中。

## **案例**

AI框架普遍都提供了分布式并行的能力，以**MindSpore** 框架为例，它支持**数据并行、模型并行** 和**混合并行** 三种分布式并行模式，同时在模型并行中，根据**自动化程度** 又支持**自动** ，**半自动** 和**手动** 的方式。

在MindSpore中可以通过context模块来设置不同的并行策略：
    
    
    from [mindspore](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=mindspore&zhida_source=entity).communication import init, get_rank, get_group_size
    from mindspore import context
    init()
    device_num = get_group_size()
    rank = get_rank()
    print("rank_id is {}, device_num is {}".format(rank, device_num))
    context.reset_auto_parallel_context()
    # 下述的并行配置用户只需要配置其中一种模式
    # 数据并行模式
    context.set_auto_parallel_context(parallel_mode=context.ParallelMode.DATA_PARALLEL)
    # 半自动并行模式
    # context.set_auto_parallel_context(parallel_mode=context.ParallelMode.SEMI_AUTO_PARALLEL)
    # 全并行模式
    # context.set_auto_parallel_context(parallel_mode=context.ParallelMode.AUTO_PARALLEL)
    # 手动并行模式
    # context.set_auto_parallel_context(parallel_mode=context.ParallelMode.HYBRID_PARALLEL)

完成策略配置后，就可以根据不同的场景来使用MindSpore提供的各种分布式并行模式。

### **数据并行**

如下为一个简单网络的数据并行代码样例。不难看出，开发者如需在数据并行时对网络有特殊的修改，只需要通过context设置ParallelMode.DATA_PARALLEL 为数据并行模式即可。
    
    
    import numpy as np
    from mindspore import Tensor, context, Model, Parameter
    from mindspore.communication import init
    from mindspore import ops, nn
    
    class DataParallelNet(nn.Cell):
        def __init__(self):
            super(DataParallelNet, self).__init__()
            # [初始化权重](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%88%9D%E5%A7%8B%E5%8C%96%E6%9D%83%E9%87%8D&zhida_source=entity)
            weight_init = np.random.rand(512, 128).astype(np.float32)
            self.weight = Parameter(Tensor(weight_init))
            self.fc = ops.MatMul()
            self.reduce = ops.ReduceSum()
    
        def construct(self, x):
            x = self.fc(x, self.weight)
            x = self.reduce(x, -1)
            return x
    
    init()
    # 设置并行模式为数据并行，其他方式一致
    context.set_auto_parallel_context(parallel_mode=context.ParallelMode.DATA_PARALLEL)
    net = DataParallelNet()
    model = Model(net)
    model.train(*args, **kwargs)

在MindSpore 1.7版本中，引入了数据并行自动加速的功能，它可以在训练过程中，主动监测训练瓶颈是否在数据侧，如果在数据侧，则根据资源（内存，CPU）状态，自动调节数据处理算子的[并行度](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%B9%B6%E8%A1%8C%E5%BA%A6&zhida_source=entity)以及算子队列长度（影响内存占用）来加速数据并行处理过程。启用自动调优只需要引入dataset模块，使用1行代码启用该功能：
    
    
    import mindspore.dataset as ds
    ds.config.set_enable_autotune(True)

如下为[Resnet-50](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=Resnet-50&zhida_source=entity)引入自动调优后的效果，单epoch耗时从72s降低到了17s：
    
    
    [WARNING] [auto_tune.cc:297 Analyse] Op (MapOp(ID:3)) is slow, input connector utilization=0.975806, output connector [utilization](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=2&q=utilization&zhida_source=entity)=0.298387, diff= 0.677419 > 0.35 threshold.
    [WARNING] [auto_tune.cc:253 RequestNumWorkerChange] Added request to change "num_parallel_workers" of Operator: MapOp(ID:3)From old value: [2] to new value: [4].
    [WARNING] [auto_tune.cc:309 Analyse] Op (BatchOp(ID:2)) getting low average worker cpu utilization 1.64516% < 35% threshold.
    [WARNING] [auto_tune.cc:263 RequestConnectorCapacityChange] Added request to change "prefetch_size" of Operator: BatchOp(ID:2)From old value: [1] to new value: [5].
    epoch: 1 step: 1875, loss is 1.1544309
    epoch time: 72110.166 ms, per step time: 38.459 ms
    [epoch](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=5&q=epoch&zhida_source=entity): 2 step: 1875, loss is 0.64530635
    epoch time: 24519.360 ms, per step time: 13.077 ms
    epoch: 3 step: 1875, loss is 0.9806979
    epoch time: 17116.234 ms, per step time: 9.129 ms

数据并行的另一种实践是参数服务器，MindSpore基于自身通信原语，实现了同步的参数服务器。**参数服务器** 通常包含三个组件，**Server** 、**Worker** 和**Scheduler** ，作用分别是：

  * Server：保存模型的权重和反向计算的梯度值，并使用优化器通过Worker上传的梯度值对模型进行更新。
  * Worker：执行网络的正反向计算，反向计算的梯度值通过Push接口上传至Server中，通过Pull接口把Server更新好的模型下载到Worker本地。
  * Scheduler：用于建立Server和Worker的通信关系。



下面介绍如何使用MindSpore在[昇腾910](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E6%98%87%E8%85%BE910&zhida_source=entity)上通过参数服务器的方式完成LeNet的分布式训练，其基本流程如下：

  1. 准备训练脚本：直接使用MindSpore模型代码仓的[LeNet训练代码](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/models/tree/r1.6/official/cv/lenet)，使用MNIST[数据集](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=4&q=%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity).



2\. 设置参数服务器训练模式的参数：通过如下代码，先启动[参数服务器模式](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E5%8F%82%E6%95%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%A8%A1%E5%BC%8F&zhida_source=entity)，然后初始化网络，最后控制训练权重通过参数服务器更新。
    
    
    context.set_ps_context(enable_ps=True)
    network = LeNet5(cfg.num_classes)
    network.set_param_ps()

3\. 配置三个组件的拉起的脚本。

Scheduler[组件脚本](https://zhida.zhihu.com/search?content_id=206926842&content_type=Article&match_order=1&q=%E7%BB%84%E4%BB%B6%E8%84%9A%E6%9C%AC&zhida_source=entity)（scheduler.sh）：
    
    
    #!/bin/bash
    export MS_SERVER_NUM=1
    export MS_WORKER_NUM=1
    export MS_SCHED_HOST=XXX.XXX.XXX.XXX
    export MS_SCHED_PORT=XXXX
    export MS_ROLE=MS_PSERVER
    python train.py --device_target=Ascend --data_path=path/to/dataset

Server组件脚本（server.sh）：
    
    
    Server组件脚本（server.sh）：
    #!/bin/bash
    export MS_SERVER_NUM=1
    export MS_WORKER_NUM=1
    export MS_SCHED_HOST=XXX.XXX.XXX.XXX
    export MS_SCHED_PORT=XXXX
    export MS_ROLE=MS_PSERVER
    python train.py --device_target=Ascend --data_path=path/to/dataset

Worker组件脚本（worker.sh）：
    
    
    #!/bin/bash
    export MS_SERVER_NUM=1
    export MS_WORKER_NUM=1
    export MS_SCHED_HOST=XXX.XXX.XXX.XXX
    export MS_SCHED_PORT=XXXX
    export MS_ROLE=MS_WORKER
    python train.py --device_target=Ascend --data_path=path/to/dataset

4\. 启动训练并查看结果：分别执行`sh Scheduler.sh > scheduler.log 2>&1 &`、`sh Server.sh > server.log 2>&1 &`以及`sh Worker.sh > worker.log 2>&1 &`启动三个组件，然后在scheduler.log可以看到Server和Worker的通信日志：
    
    
    The server node id:b5d8a47c-46d7-49a5-aecf-d29d7f8b6124,node ip: 10.90.53.118,node port:46737 assign rank id:0
    The worker node id:55e86d4b-d717-4930-b414-ebd80082f541 assign rank id:1
    Start the scheduler node is successful！

在worker.log可以看到训练结果，说明参数服务器方式运行LeNet训练成功：
    
    
    epoch: 1 step: 1, loss is 2.302287
    epoch: 1 step: 2, loss is 2.304071
    epoch: 1 step: 3, loss is 2.308778
    epoch: 1 step: 4, loss is 2.301943
    ...

### 模型并行

在[【AI工程】05-基于MindSpore的Resnet-50模型分布式训练实践 ](https://zhuanlan.zhihu.com/p/531465000) 一文中，我们已经详细介绍了基于MindSpore，使用模型自动并行策略完成Resnet-50模型的分布式训练的案例，这里不再赘述。

## **总结**

按照当前模型参数和训练数据发展的趋势，分布式并行将成为开发者在加速模型训练时唯一的选择。分布式训练时，可以从硬件和实践的角度进一步优化，如使用专用硬件，NPU（如昇腾），TPU（Google）。分布式并行模式的选择也会影响到训练的速度，此外，训练的数据批量的大小也需要合理选择，如果min-batch批量数据大，可以减少训练迭代次数，但会影响梯度下降收敛的速度以及最终的进度。在实际的模型训练过程中，需要选择合适的硬件、并行模式和超参来加速模型训练的过程。

## **参考资料**

[1] [什么是大模型？超大模型？Foundation Model?](https://www.zhihu.com/question/498275802/answer/2221187242)

[2] [MindSpore分布式自动并行训练](https://link.zhihu.com/?target=https%3A//www.jiqizhixin.com/articles/2020-04-24-5)

[3] [机器学习设计模式](https://link.zhihu.com/?target=https%3A//www.oreilly.com/library/view/machine-learning-design/9781098115777/)

[4] [分布式通信原语](https://link.zhihu.com/?target=https%3A//mindspore.cn/docs/programming_guide/zh-CN/r1.6/distributed_training_ops.html)

[5] [分布式并行训练案例](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/docs/tree/r1.6/docs/sample_code/distributed_training)

  


> 上一篇：[【AI设计模式】05-检查点模式（CheckPoints）：如何定期存储模型？](https://zhuanlan.zhihu.com/p/521311168)
