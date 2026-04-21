# 一小时上手MindSpore

原文链接：https://zhuanlan.zhihu.com/p/457516369

---

本文通过一个手写数字识别的例子，探索了快速上手[MindSpore](https://zhida.zhihu.com/search?content_id=189800974&content_type=Article&match_order=1&q=MindSpore&zhida_source=entity)的过程，完整代码请参考[这里](https://link.zhihu.com/?target=https%3A//gitee.com/leon-wang2021/awesome-mindspore/tree/master/learn-mindspore-in-60-min)。

* * *

**什么是MindSpore？**

> MindSpore(昇思)是一个易开发、高效执行、支持全场景部署的AI框架，更多详情请访问[这里](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/)。

  1. **安装MindSpore：**



1) 下载安装[miniconda](https://link.zhihu.com/?target=https%3A//docs.conda.io/en/latest/miniconda.html)

2)基于conda创建环境，并安装MindSpore。具体可参考[官网安装指南](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/install)
    
    
    conda create -n ms_37 python=3.75 
    conda install mindspore-cpu=1.5.0 -c mindspore -c conda-forge 
    

2\. **问题描述与思路：**

深度学习可以很好的解决图片分类问题。

本文使用MindSpore，基于Lenet-5神经网络完成手写数字识别的任务。

> [LeNet-5](https://zhida.zhihu.com/search?content_id=189800974&content_type=Article&match_order=1&q=LeNet-5&zhida_source=entity) 诞生于 1994 年，是最早的[卷积神经网络](https://zhida.zhihu.com/search?content_id=189800974&content_type=Article&match_order=1&q=%E5%8D%B7%E7%A7%AF%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C&zhida_source=entity)之一，被用于手写数字识别并取得了巨大的成功，可以达到低于1%的错误率。同时推动了深度学习领域的发展。

**期望效果：** 输入一张含手写数字的图像，由程序识别图像中的手写数字。

整体的实现思路如下所示：

**3\. 数据处理**

**3.1 下载[MNIST数据集](https://zhida.zhihu.com/search?content_id=189800974&content_type=Article&match_order=1&q=MNIST%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)**

> MNIST数据集是机器学习领域中经典的数据集，由6W个训练样本和1W个测试样本组成，每个样本是28 * 28像素的灰度手写数字图片，共10类（0-9）。整个数据集约为50M，可从[这里](https://link.zhihu.com/?target=http%3A//yann.lecun.com/exdb/mnist/)下载。
    
    
    //使用python下载数据
     import os     
     import wget       
     test_path = os.path.join(os.getcwd(), "datasets/test/")       
     train_path = os.path.join(os.getcwd(), "datasets/train/")       
     os.makedirs(test_path, exist_ok=True)       
     os.makedirs(train_path, exist_ok=True)       
     base_url="https://mindspore-website.obs.myhuaweicloud.com/notebook/datasets/mnist/"       
     train_files = ["train-labels-idx1-ubyte", "train-images-idx3-ubyte"]       
     test_files = ["t10k-labels-idx1-ubyte", "t10k-images-idx3-ubyte"]       
                 
     def download(url, path, filename):       
         if os.path.isfile(path+filename):       
             return       
         wget.download(url,path)       
            
     [download(base_url+x,  train_path, x) for x in train_files]       
     [download(base_url+x,  test_path, x) for x in test_files] 

下载后的目录结构如下： 
    
    
     └─datasets        
         ├─test        
         │      t10k-images.idx3-ubyte        
         │      t10k-labels.idx1-ubyte        
         │        
         └─train        
                train-images.idx3-ubyte        
                train-labels.idx1-ubyte 

Mnist数据集已经做好了测试和训练数据的划分，其中标记为**-images-** 的为图片，**-labels-** 的为标签。 

**3.2 数据处理**

数据集对于模型训练非常重要，好的数据集可以有效提高训练精度和效率。

MindSpore提供了数据处理模块 [mindspore.dataset](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/docs/api/zh-CN/r1.5/index.html)，用于数据集的预处理。  
  
本例中，我们创建data.py文件，并添加如下内容：
    
    
    import mindspore.dataset as ds        
    import mindspore.dataset.transforms.c_transforms as C    #数据处理中针对图像处理的高性能增强模块      
    import mindspore.dataset.vision.c_transforms as CV                
    from mindspore.dataset.vision import Inter      
    from mindspore import dtype as mstype  # MindSpore的数据类型对象      

数据集处理主要分为四个步骤，如下所示：  
**①定义数据集：** 将图片从28*28扩充为32*32，然后把图片数据范围缩减到 [0,1)，让数据满足正态分布  
**②数据增强和处理操作**  
**③数据集预处理**  
**④对数据shuffle、batch操作，保证随机性**
    
    
    def create_dataset(data_path, batch_size=32, repeat_size=1):    # 需要传入数据集的文件路径，批量大小      
         # ① 定义数据集      
         mnist_ds = ds.MnistDataset(data_path)                
         # ②定义所需要操作的map映射      
         resize_op = CV.Resize((32, 32), interpolation=Inter.LINEAR)     # 目标将图片大小调整为32*32，这样特征图能保证28*28，和原图一致        
         rescale_nml_op = CV.Rescale(1 / 0.3081 , -1 * 0.1307 / 0.3081)  # 数据集的标准化系数        
         rescale_op = CV.Rescale(1.0 / 255.0, 0.0)                       # 数据做标准化处理，所得到的数值分布满足正态分布        
         hwc2chw_op = CV.HWC2CHW()                                       # 转置操作        
         type_cast_op = C.TypeCast(mstype.int32)        
         # ③使用map映射函数，将数据操作应用到数据集      
         mnist_ds = mnist_ds.map(operations=type_cast_op, input_columns="label")      
        mnist_ds = mnist_ds.map(operations=[resize_op, rescale_op, rescale_nml_op, hwc2chw_op], input_columns="image")      
         # ④进行shuffle和取数据批量的操作      
         buffer_size = 10000      
         mnist_ds = mnist_ds.shuffle(buffer_size=buffer_size)      
         mnist_ds = mnist_ds.batch(batch_size)      
         return mnist_ds      

**4\. 开发模型：**

接下来，[基于MindSpore的nn.Cell](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/docs/api/zh-CN/r1.5/api_python/nn/mindspore.nn.Cell.html%23mindspore.nn.Cell)，构建Lenet5神经网络，并配置网络运行所需的优化器和损失函数。

> LeNet-5网络不包括输入层的情况下，共有7层：2个卷积层、2个下采样层（池化层）、3个全连接层。每层都包含不同数量的训练参数，如下图所示：

> ①c1卷积层：通过6种5*5大小的卷积核，将32*32的输入生成6个28*28的特征图谱，和图谱的原始大小一致  
> ②S2池化层：通过下采样，即保留图像的局部特征代表该部位的特征，如通过取2*2个单位元素中最大/最小值来表征该位置的图像特征，使用6种采样种类，获得6个14*14的特征图  
> ③C3卷积层：和CI相同，通过16个5*5的卷积核，生成16个10*10的特征图  
> ④S4池化层：将C3卷积层通过16个2*2采样种类转换为16个5*5的特征图  
> ⑤C5全连接层：使用120个5*5的卷积核生成120个卷积结果  
> ⑥F6全连接层：输入为C5的120维向量，矩阵运算后，通过激活函数输出  
> ⑦输出层：第七层，全连接输出层
    
    
     import mindspore.nn as nn      
     from mindspore.common.initializer import Normal      
     class LeNet5(nn.Cell):      
         """      
         Lenet-5网络结构      
         """      
         def __init__(self, num_class=10, num_channel=1):      
             super(LeNet5, self).__init__()      
             # 定义所需要的运算      
             self.conv1 = nn.Conv2d(num_channel, 6, 5, pad_mode='valid')      
             self.conv2 = nn.Conv2d(6, 16, 5, pad_mode='valid')      
             self.fc1 = nn.Dense(16 * 5 * 5, 120, weight_init=Normal(0.02))      
             self.fc2 = nn.Dense(120, 84, weight_init=Normal(0.02))      
             self.fc3 = nn.Dense(84, num_class, weight_init=Normal(0.02))      
             self.relu = nn.ReLU()      
             self.max_pool2d = nn.MaxPool2d(kernel_size=2, stride=2)      
             self.flatten = nn.Flatten()      
    
         def construct(self, x):      
             # 基于已有的算子构建Lenet-5的神经网络         
             x = self.conv1(x)         
             x = self.relu(x)      
             x = self.max_pool2d(x)          
             x = self.conv2(x)      
             x = self.relu(x)        
             x = self.max_pool2d(x)      
             x = self.flatten(x)       
             x = self.fc1(x)      
             x = self.relu(x)        
             x = self.fc2(x)      
             x = self.relu(x)        
             x = self.fc3(x)      
             return x      

**5\. 训练 &模型生成**

有了前面的网络构建和数据处理，接下来执行训练。  
  
**5.1 执行训练**
    
    
     import argparse     
     import os          
     import mindspore.nn as nn     
     from mindspore import Model     
     from mindspore.nn import Accuracy     
     from mindspore.train.callback import LossMonitor     
     from mindspore.train.callback import ModelCheckpoint, CheckpointConfig     
         
         
     from data import create_dataset     
     from lenet import LeNet5     
                
     data_path = "./datasets"     
     train_epoch = 1     
     dataset_size = 1     
         
         
     def train_net(args, model, epoch_size, data_path, repeat_size, ckpoint_cb):     
         """定义训练的方法"""     
         # 加载训练数据集     
         ds_train = create_dataset(os.path.join(data_path, "train"), 32, repeat_size)     
         model.train(epoch_size, ds_train, callbacks=[ckpoint_cb, LossMonitor(125)]) 

**5.2 定义损失函数和优化器**

损失函数用于评估预测值和真实值的差距。  
本例使用了MindSpore中的SoftMax交叉熵函数及Momentum优化器，加速随机梯度下降训练。  
相关代码如下：
    
    
     # 定义损失函数      
     net_loss = nn.SoftmaxCrossEntropyWithLogits(sparse=True, reduction='mean')      
     # 定义优化器      
     net_opt = nn.Momentum(net.trainable_params(), learning_rate=0.01, momentum=0.9)  

**5.3 配置模型**
    
    
     net = LeNet5()       
     # 设置模型保存参数       
     config_ck = CheckpointConfig(save_checkpoint_steps=1875, keep_checkpoint_max=10)       
     # 应用模型保存参数       
     ckpoint = ModelCheckpoint(prefix="checkpoint_lenet", config=config_ck)       
     model = Model(net, net_loss, net_opt, metrics={"Accuracy": Accuracy()})       
     train_net( model, train_epoch, data_path, dataset_size, ckpoint)       

保存代码到脚本train.py中，运行脚本`python train.py`，训练的过程如下所示：
    
    
    epoch: 1 step: 125, loss is 2.3083377
    epoch: 1 step: 250, loss is 2.3019726
    ...
    epoch: 1 step: 1500, loss is 0.028385757
    epoch: 1 step: 1625, loss is 0.0857362
    epoch: 1 step: 1750, loss is 0.05639569
    epoch: 1 step: 1875, loss is 0.12366105
    {'Accuracy': 0.9663477564102564} //精确度的数据表明模型在验证数据集上的识别字符的准确度。

**6\. 模型评估**

模型训练完成后，可以从测试集中抽取出一张图片来验证模型是否能准确识别.
    
    
    import os
    import numpy as np
    from matplotlib import pyplot as plt
    from mindspore import load_checkpoint, load_param_into_net, Tensor, Model
    import mindspore.dataset.transforms.c_transforms as C
    import mindspore.dataset.vision.c_transforms as CV
    from mindspore.dataset.vision import Inter
    from mindspore import dtype as mstype
    import mindspore.dataset as ds
    from lenet import LeNet5
     
    data_path = "./datasets/"
    data = ds.MnistDataset(os.path.join(data_path, "test"), num_samples=1, shuffle=False)
     
    def draw_data(data):
        for item in data.create_dict_iterator(num_epochs=1, output_numpy=True):
            image = item["image"]
            label = item["label"]
        plt.imshow(image, cmap=plt.cm.gray)
        plt.title(label)
        plt.show()
     
    def transform_data(data):
         # 对测试数据集实施相同的数据变换操作      
         resize_op = CV.Resize((32, 32), interpolation=Inter.LINEAR)     # 目标将图片大小调整为32*32，这样特征图能保证28*28，和原图一致        
         rescale_nml_op = CV.Rescale(1 / 0.3081 , -1 * 0.1307 / 0.3081)  # 数据集的标准化系数        
         rescale_op = CV.Rescale(1.0 / 255.0, 0.0)                       # 数据做标准化处理，所得到的数值分布满足正态分布        
         hwc2chw_op = CV.HWC2CHW()                                       # 转置操作        
         type_cast_op = C.TypeCast(mstype.int32)        
         # 使用map映射函数，将数据操作应用到数据集      
        data = data.map(operations=type_cast_op, input_columns="label")
        data = data.map(operations=[resize_op, rescale_op, rescale_nml_op, hwc2chw_op], input_columns="image")
        # 进行batch操作
        return data.batch(1)
     
    # 加载模型参数到网络中
    param_dict = load_checkpoint("checkpoint_lenet-1_1875.ckpt")
    net = LeNet5()
    load_param_into_net(net, param_dict)
    model = Model(net)
    # 定义测试数据集，batch_size设置为1，取出一张图片 
    for item in transform_data(raw_data, 1): 
         image = item[0] 
         label = item[1] 
    # 使用函数model.predict预测image对应分类 
    output = model.predict(Tensor(image)) 
    predicted = np.argmax(output.asnumpy(), axis=1)
    # 输出预测分类与实际分类
    print(f'Predicted: "{predicted[0]}", Actual: "{labels[0]}"')
    draw_data(raw_data)

将如上代码保存到'testing.py'文件中。执行'python testing.py' 可以看到如下的结果：

Predicted: 7; Actual: 7

> 完整代码请参考[这里](https://link.zhihu.com/?target=https%3A//gitee.com/leon-wang2021/awesome-mindspore/tree/master/learn-mindspore-in-60-min)

**参考资料：**

[ https://1187100546.github.io/2020/01/14/lenet-5/](https://link.zhihu.com/?target=https%3A//1187100546.github.io/2020/01/14/lenet-5/)

[https://www.jianshu.com/p/d20e293a0d34](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/d20e293a0d34)

[MindSpore教程 - MindSpore master documentation](https://link.zhihu.com/?target=https%3A//www.mindspore.cn/tutorials/zh-CN/r1.5/index.html)
