# 【AI设计模式】01-数据表示-特征哈希（Feature Hashed）模式

原文链接：https://zhuanlan.zhihu.com/p/504100961

---

​

目录

## **摘要**

数据是机器学习的生命线，对数据的有效管理是AI中重要的工程实践。在《[【AI设计模式】机器学习设计模式概述](https://zhuanlan.zhihu.com/p/502911554)》中，我们介绍了机器学习领域设计模式的背景，以及设计模式解决的问题等。本篇文章将介绍AI设计模式中的一种数据表示模式 - [特征哈希](https://zhida.zhihu.com/search?content_id=200154236&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E5%93%88%E5%B8%8C&zhida_source=entity)（Feature Hashed）模式，并探讨如何使用[MindSpore](https://zhida.zhihu.com/search?content_id=200154236&content_type=Article&match_order=1&q=MindSpore&zhida_source=entity)实践该模式。

AI设计模式总览

## **模式定义**

特征哈希是AI设计模式中的一种数据表示模式，能够有效解决分类数据不完整、高基数（特征类别不均）、以及冷启动问题（推理时无法处理新出现的类别）。结合MindSpore提供的数据处理接口，开发者可以很容易的应用该实践。

## **问题**

机器学习在数据处理时，通常使用**[独热编码](https://zhida.zhihu.com/search?content_id=200154236&content_type=Article&match_order=1&q=%E7%8B%AC%E7%83%AD%E7%BC%96%E7%A0%81&zhida_source=entity)** （one-hot encoding）的方式将分类数据转换为数值数据。独热编码是用N个状态对N个分类数据编码，这样在任意时刻，只有一位是有效的。比如，假设我们有6个邮政编码[1,2,3,4,5,6]，然后通过独热编码对这些分类数据进行编码：
    
    
    import numpy as np
    import mindspore.dataset.transforms.c_transforms as c_transforms
    import mindspore.dataset as ds
    
    code = [1,2,3,4,5,6]
    data = np.array(code)   # 将结果列表转为Numpy的数组
    dataset = ds.NumpySlicesDataset(data, column_names=["clz"], shuffle=False)  # 基于MindSpore的Dataset接口把Numpy数组转为Dataset对象
    onehot_op = c_transforms.OneHot(num_classes=7)                                  # 定义操作，这里num_class要大于code中最大数的值
    dataset = dataset.map(operations=onehot_op, input_columns=["clz"])          # 应用独热编码
    
    for item in dataset:
        print(item)

执行后可以看到编码的结果：
    
    
    [Tensor(shape=[7], dtype=Int32, value= [0, 1, 0, 0, 0, 0, 0])]
    [Tensor(shape=[7], dtype=Int32, value= [0, 0, 1, 0, 0, 0, 0])]
    [Tensor(shape=[7], dtype=Int32, value= [0, 0, 0, 1, 0, 0, 0])]
    [Tensor(shape=[7], dtype=Int32, value= [0, 0, 0, 0, 1, 0, 0])]
    [Tensor(shape=[7], dtype=Int32, value= [0, 0, 0, 0, 0, 1, 0])]
    [Tensor(shape=[7], dtype=Int32, value= [0, 0, 0, 0, 0, 0, 1])]

这样可以确保分类数据的输入的唯一性。

处理分类输入需要提前知道所有的类别，语言、日期等相对确定的数据很容易处理，而对于比较难预测的数据会存在一些问题：

### 1\. **数据不完整**

训练数据中没有包含所有的特征类别。如果训练数据不完整，可能无法提前获得所有可能的单词，导致编码以后的数据也不完整。比如，针对医疗方面的一些模型，训练数据的词汇表中无法包含所有的医院和医生信息。

### 2\. **高基数（某个分类特征的类别特别多）**

单个分类特征的不同值很多，可能需要长度数百万的特征向量， 如IP地址、家庭住址等，导致模型也需要很大空间，无法在小设备上部署。

### 3\. **冷启动问题（推理时无法处理新出现的类别）**

对于新的分类数据，生产环境中的模型无法正确的预测，会出现错误，需要专门的服务来处理这种冷启动的问题。

## **解决方案**

以参考图书中预测航班的准点率模型场景为例，美国约有350个机场，机场间的差别会比较大，有些机场航班很多，有些机场航班很少，同时，每年会有新的机场出现。这个场景同时存在了独热编码时的数据不完整、高基数和冷启动问题。

通过特征哈希模式来解决分类数据在**独热编码** 存在的问题。具体的操作如下：

  1. 将机场的分类数据，把输入转化为唯一的字符串，如把机场名称数据改为缩写并保证数据不重复；
  2. 对字符串使用稳定可移植（训练和推理场景都可用）的哈希算法进行哈希；
  3. 对哈希结果取余数。



通过[farmhash](https://zhida.zhihu.com/search?content_id=200154236&content_type=Article&match_order=1&q=farmhash&zhida_source=entity)算法，对于这些机场进行哈希，然后分别放入10，1000个桶中，结果如下：
    
    
    >> airports = ["DTW", "LBB", "SNA", "MSO", "ANC"]
    >>> list(map(lambda x: farmhash.hash64withseed(x, 10) % 10, airports))
    [9, 9, 4, 0, 1]
    >>> list(map(lambda x: farmhash.hash64withseed(x, 1000) % 1000, airports))
    [416, 532, 193, 538, 971]

机场缩写| hash 10| hash 1000  
---|---|---  
DTW| 9| 416  
LBB| 9| 532  
SNA| 4| 193  
MSO| 0| 538  
ANC| 1| 971  
  
特征哈希如何解决分类数据的问题:

  1. **针对数据不完整问题** ：即使有些机场数据不在训练数据集中，但它通过特征值哈希后在桶的大小范围内，不用担心数据不完整的情况。
  2. **针对高基数问题** ：通过哈希的方式可以将数据的规模降低，减少了系统内存占用和模型大小，即便有百万的数据规模，哈希后也只会落入到有限的桶中。
  3. **针对冷启动问题** ：如果新的分类数据添加到系统中，它在哈希后落入和其它机场相同的桶，所以不用担心在生产环境中预测时会出错的情况。之后通过训练更新的模型获得更好的预测。比如，对于350个机场，哈希桶设置为70，大约每个桶有5个机场，每个桶都有数据，生产环境预测就不会落空，只是预测的数据可能不会特别精确，需要后续训练来优化模型。



## **案例**

这里沿用了上面提到的预测机场航班准点率的例子，首先对机场数据应用模式，而后通过MindSpore的独热编码接口，完成数据的编码准备。其中依赖哈希算法库需要通过`pip install pyfarmhash`安装。
    
    
    import farmhash
    import numpy as np
    import mindspore.dataset.transforms.c_transforms as c_transforms
    import mindspore.dataset as ds
    
    airports = ["DTW", "LBB", "SNA", "MSO", "ANC", "ABC", "CDE", "FGH"]    #  将机场名称缩写
    hashed_data = list(map(lambda x: farmhash.hash64withseed(x, 1000) % 4, airports))  #  对字符串应用特征哈希模式
    
    data = np.array(hashed_data)   # 将结果列表转为Numpy的数组
    dataset = ds.NumpySlicesDataset(data, column_names=["airport_name"], shuffle=False)  # 基于MindSpore的Dataset接口把Numpy数组转为Dataset对象
    onehot_op = c_transforms.OneHot(num_classes=4)                                       # 定义独热编码操作，这里num_class的数量和桶的数量保持一致
    dataset = dataset.map(operations=onehot_op, input_columns=["airport_name"])          # 对机场信息数据应用编码
    
    for item in dataset:
        print(item)

编码的输出结果如下：
    
    
    [Tensor(shape=[4], dtype=Int32, value= [1, 0, 0, 0])]
    [Tensor(shape=[4], dtype=Int32, value= [1, 0, 0, 0])]
    [Tensor(shape=[4], dtype=Int32, value= [0, 1, 0, 0])]
    [Tensor(shape=[4], dtype=Int32, value= [0, 0, 1, 0])]
    [Tensor(shape=[4], dtype=Int32, value= [0, 0, 0, 1])]
    [Tensor(shape=[4], dtype=Int32, value= [0, 0, 0, 1])]
    [Tensor(shape=[4], dtype=Int32, value= [0, 0, 1, 0])]
    [Tensor(shape=[4], dtype=Int32, value= [1, 0, 0, 0])]

## **总结**

特征哈希模式在使用时有它适用的场景，它的主要问题是损失了模型精度。特征哈希模式不适合分类数据明确，词汇表大小相对较小（1000量级），并且不存在冷启动的场景。取模是有损操作，特征哈希模式将不同的分类放到了同一个桶中，损失了数据的准确性。在分类的数据特别不平衡时，会导致推理的误差比较大。比如榆林机场的流量比较小，西安机场的流量比它大两个量级，如果它们被放到同一个桶中，当成一种编码处理。模型的结果将更偏向于西安的场景，导致对于起飞等待时间等预测出现偏差。

有两种方式可以缓解模式造成的模型精度损失，可以在实践时考虑应用：

  1. **添加[聚合特征](https://zhida.zhihu.com/search?content_id=200154236&content_type=Article&match_order=1&q=%E8%81%9A%E5%90%88%E7%89%B9%E5%BE%81&zhida_source=entity)**：如果分类变量的分布偏斜，或者桶的数量少导致冲突多，可以通过添加聚合特征作为模型的输入来缓解。比如，对于每个机场，都可以在训练数据集中找到准时航班的概率，并将其作为一个特征添加到模型中。避免在散列机场代码时丢失与个别机场相关的信息。在某些情况下，可以完全避免将机场名称作为一个特征，因为有航班准点的相对频率数据可能就够了。
  2. 把桶的数量作为超参来调整，以达到精度的平衡。



在下一篇，我们讲介绍数据表示的嵌入模式（Embeddings）。

## **参考**

1\. [《Machine Learning Design Patterns》](https://link.zhihu.com/?target=https%3A//www.oreilly.com/library/view/machine-learning-design/9781098115777/)

  


> 上一篇：[【AI设计模式】机器学习设计模式概述](https://zhuanlan.zhihu.com/p/502911554)  
> 下一篇：[【AI设计模式】02-数据表示-嵌入(Embeddings)模式](https://zhuanlan.zhihu.com/p/511770353)
