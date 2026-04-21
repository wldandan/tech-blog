# 【AI工程】05-基于MindSpore的Resnet-50模型分布式训练实践

原文链接：https://zhuanlan.zhihu.com/p/531465000

---

​

目录

## 概述

在上一篇文章《[为什么AI需要分布式并行？](https://zhuanlan.zhihu.com/p/528964900)》中，我们介绍了AI算力为什么需要[分布式并行](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=2&q=%E5%88%86%E5%B8%83%E5%BC%8F%E5%B9%B6%E8%A1%8C&zhida_source=entity)、分布式并行的策略、以及MindSpore框架实现自动、半自动并行的原理。在本篇文章中，我们尝试基于MindSpore，使用自动并行策略完成[Resnet-50](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=Resnet-50&zhida_source=entity)模型的分布式训练，该实践步骤如下所示。

  1. 准备数据集：下载Cifar019数据集作为训练数据集。
  2. 配置分布式环境：配置昇腾910 8卡环境。
  3. 调用集合通信库：引入HCCL，完成多卡间通信初始化。
  4. 加载数据集：基于数据并行模式加载训练数据集。
  5. 定义网络：定义Resnet-50网络。
  6. 定义损失函数及优化器：定义针对分布式并行场景下的[损失函数](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=2&q=%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0&zhida_source=entity)和优化器。
  7. 构建网络训练代码：定义[分布式并行策略](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F%E5%B9%B6%E8%A1%8C%E7%AD%96%E7%95%A5&zhida_source=entity)以及训练代码。
  8. 训练脚本：完成训练脚本并执行训练。



## 第1步：准备[数据集](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=6&q=%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)

首先需要下载ResNet50训练的CIFAR-10数据集，该数据集由10类32*32的彩色图片组成，每类包含6000张图片。其中训练集共50000张图片，测试集共10000张图片。将数据集下载并解压到本地路径下，解压后的文件夹为cifar-10-batches-bin，数据集[下载链接](https://link.zhihu.com/?target=http%3A//www.cs.toronto.edu/~kriz/cifar-10-binary.tar.gz) 。

## 第2步： 配置分布式[环境变量](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F&zhida_source=entity)

在裸机环境进行分布式训练时，需要配置当前多卡环境的组网信息文件。以Ascend 910 AI处理器为例，1个8卡环境的json[配置文件](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6&zhida_source=entity)示例如下，本样例将该配置文件命名为rank_table_8pcs.json。
    
    
     {      
        "board_id": "0x0000",      
        "chip_info": "910",      
        "deploy_mode": "lab",      
        "group_count": "1",      
        "group_list": [      
            {      
                "device_num": "8",      
                "server_num": "1",      
                "group_name": "",      
                "instance_count": "8",      
                "instance_list": [      
                    {"devices": [{"device_id": "0","device_ip": "192.1.27.6"}],"rank_id": "0","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "1","device_ip": "192.2.27.6"}],"rank_id": "1","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "2","device_ip": "192.3.27.6"}],"rank_id": "2","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "3","device_ip": "192.4.27.6"}],"rank_id": "3","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "4","device_ip": "192.1.27.7"}],"rank_id": "4","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "5","device_ip": "192.2.27.7"}],"rank_id": "5","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "6","device_ip": "192.3.27.7"}],"rank_id": "6","server_id": "10.155.111.140"},      
                    {"devices": [{"device_id": "7","device_ip": "192.4.27.7"}],"rank_id": "7","server_id": "10.155.111.140"}      
                    ]      
            }      
        ],      
        "para_plane_nic_location": "device",      
        "para_plane_nic_name": ["eth0","eth1","eth2","eth3","eth4","eth5","eth6","eth7"],      
        "para_plane_nic_num": "8",      
        "status": "completed"      
    }      

其中，以下参数需要根据实际训练环境修改：

  * board_id：表示当前运行的环境，**x86** 设为“0x0000”，**arm** 设为“0x0020”。
  * server_num：表示机器数量， server_id表示本机IP地址。
  * device_num、para_plane_nic_num及instance_count表示卡的数量。
  * rank_id：表示卡逻辑序号，固定从0开始编号，device_id表示卡物理序号，即卡所在机器中的实际序号。
  * device_ip：表示[集成网卡](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E9%9B%86%E6%88%90%E7%BD%91%E5%8D%A1&zhida_source=entity)的IP地址，可以在当前机器执行指令cat /etc/hccn.conf，address_x的键值就是网卡IP地址。
  * para_plane_nic_name：对应网卡名称。



## 第3步：调用[集合通信库](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=2&q=%E9%9B%86%E5%90%88%E9%80%9A%E4%BF%A1%E5%BA%93&zhida_source=entity)

MindSpore分布式并行训练的通信使用了华为集合通信库Huawei Collective Communication Library（以下简称HCCL），可以在Ascend AI处理器配套的软件包中找到。同时mindspore.communication.management中封装了HCCL提供的集合[通信接口](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E9%80%9A%E4%BF%A1%E6%8E%A5%E5%8F%A3&zhida_source=entity)，方便用户配置分布式信息。HCCL实现了基于Ascend AI处理器的多机多卡通信，我们列出使用[分布式服务](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F%E6%9C%8D%E5%8A%A1&zhida_source=entity)常见的一些使用限制，详细的可以查看HCCL对应的使用文档。

下面是调用集合通信库样例代码： 
    
    
    import os      
    from [mindspore](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=2&q=mindspore&zhida_source=entity) import context      
    from mindspore.communication.management import init      
    if __name__ == "__main__":      
         context.set_context(mode=context.GRAPH_MODE, device_target="Ascend", device_id=int(os.environ["DEVICE_ID"]))      
         init()      
         ...      

其中， 

  * mode=context.GRAPH_MODE：使用分布式训练需要指定“运行模式”为“图模式”（PyNative模式不支持并行）。
  * device_id：卡的物理序号，即卡所在机器中的实际序号。
  * init：使能HCCL通信，并完成分布式训练初始化操作。



## 第4步：基于数据并行模式加载数据集

分布式训练时，数据是以数据并行的方式导入的。下面的代码通过数据并行的方式加载了CIFAR-10数据集，代码中的data_path是指数据集的路径，即cifar-10-batches-bin文件夹的路径。 
    
    
    import mindspore.common.dtype as mstype      
    import mindspore.dataset as ds      
    import mindspore.dataset.transforms.c_transforms as C      
    import mindspore.dataset.transforms.vision.c_transforms as vision      
    from mindspore.communication.management import get_rank, get_group_size      
    def create_dataset(data_path, repeat_num=1, batch_size=32, rank_id=0, rank_size=1):      
        resize_height = 224      
        resize_width = 224      
        rescale = 1.0 / 255.0      
        shift = 0.0      
             
        # get rank_id and rank_size      
        rank_id = get_rank()      
        rank_size = get_group_size()      
        data_set = ds.Cifar10Dataset(data_path, num_shards=rank_size, shard_id=rank_id)      
             
        # define map operations      
        random_crop_op = vision.RandomCrop((32, 32), (4, 4, 4, 4))      
        random_horizontal_op = vision.RandomHorizontalFlip()      
        resize_op = vision.Resize((resize_height, resize_width))      
        rescale_op = vision.Rescale(rescale, shift)      
        normalize_op = vision.Normalize((0.4465, 0.4822, 0.4914), (0.2010, 0.1994, 0.2023))      
        changeswap_op = vision.HWC2CHW()      
        type_cast_op = C.TypeCast(mstype.int32)      
        c_trans = [random_crop_op, random_horizontal_op]      
        c_trans += [resize_op, rescale_op, normalize_op, changeswap_op]      
        # apply map operations on images      
        data_set = data_set.map(input_columns="label", operations=type_cast_op)      
        data_set = data_set.map(input_columns="image", operations=c_trans)      
        # apply shuffle operations      
        data_set = data_set.shuffle(buffer_size=10)      
        # apply batch operations      
        data_set = data_set.batch(batch_size=batch_size, drop_remainder=True)      
        # apply repeat operations      
        data_set = data_set.repeat(repeat_num)      
        return data_set   

与单机不同处，在数据集接口需要传入num_shards和shard_id参数，分别对应卡的数量和逻辑序号，建议通过HCCL接口获取： 

  * get_rank：获取当前设备在集群中的ID。
  * get_group_size：获取[集群](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=2&q=%E9%9B%86%E7%BE%A4&zhida_source=entity)数量。



## 第5步：定义网络

数据并行及自动并行模式下，网络定义方式与单机一致。代码请参考[ResNet50实现](https://zhuanlan.zhihu.com/ResNet50%E5%AE%9E%E7%8E%B0)。 

## 第6步：定义损失函数及优化器 

自动并行以[算子](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E7%AE%97%E5%AD%90&zhida_source=entity)为粒度切分模型，通过[算法](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E7%AE%97%E6%B3%95&zhida_source=entity)搜索得到最优并行策略，所以与单机训练不同的是，为了有更好的并行训练效果，损失函数建议使用小算子来实现。 

在Loss部分，我们采用SoftmaxCrossEntropyWithLogits的展开形式，即按照数学公式，将其展开为多个小算子进行实现，样例代码如下： 
    
    
    from mindspore.ops import operations as P      
    from mindspore import Tensor      
    import mindspore.ops.functional as F      
    import mindspore.common.dtype as mstype      
    import mindspore.nn as nn      
    class SoftmaxCrossEntropyExpand(nn.Cell):      
        def __init__(self, sparse=False):      
            super(SoftmaxCrossEntropyExpand, self).__init__()      
            self.exp = P.Exp()      
            self.sum = P.ReduceSum(keep_dims=True)      
            self.onehot = P.OneHot()      
            self.on_value = Tensor(1.0, mstype.float32)      
            self.off_value = Tensor(0.0, mstype.float32)      
            self.div = P.Div()      
            self.log = P.Log()      
            self.sum_cross_entropy = P.ReduceSum(keep_dims=False)      
            self.mul = P.Mul()      
            self.mul2 = P.Mul()      
            self.mean = P.ReduceMean(keep_dims=False)      
            self.sparse = sparse      
            self.max = P.ReduceMax(keep_dims=True)      
            self.sub = P.Sub()      
                 
        def construct(self, logit, label):      
            logit_max = self.max(logit, -1)      
            exp = self.exp(self.sub(logit, logit_max))      
            exp_sum = self.sum(exp, -1)      
            softmax_result = self.div(exp, exp_sum)      
            if self.sparse:      
                label = self.onehot(label, F.shape(logit)[1], self.on_value, self.off_value)      
            softmax_result_log = self.log(softmax_result)      
            loss = self.sum_cross_entropy((self.mul(softmax_result_log, label)), -1)      
            loss = self.mul2(F.scalar_to_array(-1.0), loss)      
            loss = self.mean(loss, -1)      
            return loss

定义优化器：采用Momentum优化器作为参数更新工具，这里定义与单机一致，不再展开，具体可以参考样例代码中的实现。 

## 第7步：训练网络

context.set_auto_parallel_context是配置并行训练参数的接口，必须在Model初始化前调用。如用户未指定参数，框架会自动根据并行模式为用户设置参数的经验值。如数据并行模式下，parameter_broadcast默认打开。主要参数包括： 

  * parallel_mode：分布式并行模式，默认为单机模式ParallelMode.STAND_ALONE。可选数据并行ParallelMode.DATA_PARALLEL及自动并行ParallelMode.AUTO_PARALLEL。
  * parameter_broadcast： 参数初始化广播开关，DATA_PARALLEL和HYBRID_PARALLEL模式下，默认值为True。
  * mirror_mean：反向计算时，框架内部会将数据并行参数分散在多台机器的梯度值进行收集，得到全局梯度值后再传入优化器中更新。默认值为False，True对应allreduce_mean操作，False对应allreduce_sum操作。
  * device_num和global_rank建议采用默认值，框架内会调用HCCL接口获取。



如脚本中存在多个网络用例，请在执行下个用例前调用context.reset_auto_parallel_context将所有参数还原到默认值。在下面的样例中我们指定并行模式为[自动并行](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=6&q=%E8%87%AA%E5%8A%A8%E5%B9%B6%E8%A1%8C&zhida_source=entity)，用户如需切换为[数据并行模式](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=4&q=%E6%95%B0%E6%8D%AE%E5%B9%B6%E8%A1%8C%E6%A8%A1%E5%BC%8F&zhida_source=entity)，只需将parallel_mode改为DATA_PARALLEL。 
    
    
    from mindspore import context      
    from mindspore.nn.optim.momentum import Momentum      
    from mindspore.train.callback import LossMonitor      
    from mindspore.train.model import Model, ParallelMode      
    from resnet import resnet50      
    device_id = int(os.getenv('DEVICE_ID'))      
    context.set_context(mode=context.GRAPH_MODE, device_target="Ascend")      
    context.set_context(device_id=device_id) # set device_id      
    def test_train_cifar(epoch_size=10):      
        context.set_auto_parallel_context(parallel_mode=ParallelMode.AUTO_PARALLEL, mirror_mean=True)      
        loss_cb = LossMonitor()      
        dataset = create_dataset(data_path, epoch_size)      
        batch_size = 32      
        num_classes = 10      
        net = resnet50(batch_size, num_classes)      
        loss = SoftmaxCrossEntropyExpand(sparse=True)      
        opt = Momentum(filter(lambda x: x.requires_grad, net.get_parameters()), 0.01, 0.9)      
        model = Model(net, loss_fn=loss, optimizer=opt)      
        model.train(epoch_size, dataset, callbacks=[loss_cb], dataset_sink_mode=True)      

其中， 

  * dataset_sink_mode=True：表示采用数据集的下沉模式，即训练的计算下沉到硬件平台中执行。
  * LossMonitor：能够通过[回调函数](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0&zhida_source=entity)返回Loss值，用于监控损失函数。



## 第8步：运行脚本

上述已将训练所需的脚本编辑好了，接下来通过命令调用对应的脚本。目前MindSpore[分布式执行](https://zhida.zhihu.com/search?content_id=206236793&content_type=Article&match_order=1&q=%E5%88%86%E5%B8%83%E5%BC%8F%E6%89%A7%E8%A1%8C&zhida_source=entity)采用单卡单进程运行方式，即每张卡上运行1个进程，进程数量与使用的卡的数量一致。其中，0卡在前台执行，其他卡放在后台执行。每个进程创建1个目录，用来保存日志信息以及算子编译信息。下面以使用8张卡的分布式训练脚本为例，演示如何运行脚本： 
    
    
           #!/bin/bash      
    export DATA_PATH=${DATA_PATH：-$1}      
           export RANK_TABLE_FILE=$(pwd) /rank_table_8pcs.json      
           export RANK_SIZE=8      
           for((i=1;i<${RANK_SIZE};i++))      
           do      
               rm -rf device$i      
               mkdir device$i      
               cp ./resnet50_distributed_training.py ./resnet.py ./device$i      
               cd ./device$i      
               export DEVICE_ID=$i      
               export RANK_ID=$i      
               echo "start training for device $i"      
               env > env$i.log      
               pytest -s -v ./resnet50_distributed_training.py > train.log$i 2>&1 &      
               cd ../      
           done      
           rm -rf device0      
           mkdir device0      
           cp ./resnet50_distributed_training.py ./resnet.py ./device0      
           cd ./device0      
           export DEVICE_ID=0      
           export RANK_ID=0      
           echo "start training for device 0"      
           env > env0.log      
           pytest -s -v ./resnet50_distributed_training.py > train.log0 2>&1      
           if [ $? -eq 0 ];then      
               echo "training success"      
           else      
               echo "training failed"      
               exit 2      
           fi      
           cd ../   

脚本需要传入数据集的路径变量DATA_PATH。其中需要设置的环境变量有， 

  * RANK_TABLE_FILE：组网信息文件的路径。
  * DEVICE_ID：当前卡在机器上的实际序号。
  * RANK_ID: 当前卡的逻辑序号。 其余环境变量请参考安装教程中的配置项。



启动脚本运行时间大约在5分钟内，主要时间是用于算子的编译，实际训练时间在20秒内，而单卡的脚本编译加上训练的时间在10分钟左右。device目录的train.log中运行的日志示例如下： 
    
    
    epoch: 1 step: 156, loss is 2.0084016
    epoch: 2 step: 156, loss is 1.6407638
    epoch: 3 step: 156, loss is 1.6164391
    epoch: 4 step: 156, loss is 1.6838071
    epoch: 5 step: 156, loss is 1.6320667
    epoch: 6 step: 156, loss is 1.3098773
    epoch: 7 step: 156, loss is 1.3515002
    epoch: 8 step: 156, loss is 1.2943741
    epoch: 9 step: 156, loss is 1.2316195
    epoch: 10 step: 156, loss is 1.1533381

## 总结

从上面Resnet-50的分布式并行训练的案例，我们不难看出，分布式并行相比单卡引入了一些开发上的复杂性：

  1. 多卡的分布式配置及分布式通信。
  2. 定义网络时需要考虑损失函数的算子选择。
  3. 训练脚本相比单卡更加复杂。



同时，分布式训练下的调试调优也会更难些。虽然引入了这些开发调试的成本，但是分布式训练带来的性能收益可能远高于开发调试的开销，以Resnet-50为例，在昇腾芯片上8卡的性能超过单卡的X倍。因此对于训练要求较高的模型，使用分布式训练是更好的选择。

## **参考资料**

[1] [分布式训练完整样例代码](https://link.zhihu.com/?target=https%3A//gitee.com/mindspore/docs/tree/master/docs/sample_code/distributed_training)

  


> 上一篇：[04-为什么AI需要分布式并行？](https://zhuanlan.zhihu.com/p/528964900)  
> 下一篇：[06-CD4ML-机器学习的持续交付（上）](https://zhuanlan.zhihu.com/p/552173360)
