# 【AI设计模式】04-数据处理-Eager模式

原文链接：https://zhuanlan.zhihu.com/p/517450571

---

​

目录

在前一篇文章[《数据处理-Pipeline模式》](https://zhuanlan.zhihu.com/p/517440240)中提到，资源条件允许的情况下，为了追求更高的性能，可以使用流水线模式加快数据处理速度。但在有些场景下，如训练数据量较小、训练资源不足或者推理（零散样本），无法使用流水线模式或者代码实现较为复杂。在这些场景下，可以直接进行数据处理的算子调用，以串行的方式完成数据处理，这种模式也被称为**Eager模式** 。

AI设计模式总览

## **模式定义**

在流水线模式中，需要定义map算子，并将数据增强的算子交给map算子调度，由其负责启动和执行（可并行）给定的数据增强算子，对数据管道的数据进行[映射](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=1&q=%E6%98%A0%E5%B0%84&zhida_source=entity)变化。从代码实现角度，这种方式需要开发者从构建输入源开始，逐步定义数据流水线中的各个阶段的[算子](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=6&q=%E7%AE%97%E5%AD%90&zhida_source=entity)，只有在进行`map`操作的时候才涉及数据增强算子的执行。对于比较简单的场景，流水线模式增加了开发的负担。

**Eager模式是一种轻量化的数据处理执行方式** ，开发者通过函数式调用的方式执行数据处理算子，不需要构建数据管道。代码编写会更为简洁且能立即执行得到运行结果，适合在小型数据增强实验、[模型推理](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E6%8E%A8%E7%90%86&zhida_source=entity)等轻量化场景中使用。如下图，和流水线模式相比，Eager模式不需要构建数据处理的流水线，开发更加直接简单。

MindSpore目前支持在Eager模式执行通用、[图像处理](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=1&q=%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86&zhida_source=entity)、文本处理的数据算子，MindSpore的vision模块（`mindspore.dataset.vision`）、text模块（`mindspore.dataset.text`）、transform模块（`mindspore.dataset.transforms`）都支持Eager模式。

## **案例**

下面我们使用MindSpore dataset接口，使用Eager模式对图像数据和文本数据进行处理。

### **[图像数据处理](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=1&q=%E5%9B%BE%E5%83%8F%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)**

这里使用香蕉图片，基于MindSpore的Eager模式完成图像的转换，首先从OBS下载香蕉图片：
    
    
    import wget
    
    wget.download("https://obs.dualstack.cn-north-4.myhuaweicloud.com/mindspore-website/notebook/datasets/banana.jpg", ".")

然后混用vision模块中的c_tranforms与py_transforms的算子，对给定图像进行变换（），vision算子的Eager模式支持numpy.array或PIL.Image类型的数据作为入参，过程中不需要构建数据的管道。
    
    
    import numpy as np
    from PIL import Image
    import matplotlib.pyplot as plt
    import mindspore.dataset.vision.c_transforms as C
    import mindspore.dataset.vision.py_transforms as P
    
    banana = Image.open("banana.jpg").convert("RGB")
    print("Image.type: {}, Image.shape: {}".format(type(banana), banana.size))
    
    # 定义Resize操作，并立即执行
    squared_banana = C.Resize(size=(320))(banana)
    print("Image.type: {}, Image.shape: {}".format(type(squared_banana), squared_banana.shape))
    
    # 定义CenterCrop操作并理解执行（图片中部切280*280大小的内容）
    squared_banana = C.CenterCrop((280, 280))(squared_banana)
    print("Image.type: {}, Image.shape: {}".format(type(squared_banana), squared_banana.shape))
    
    # ToPIL()接口将[numpy](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=3&q=numpy&zhida_source=entity)图像转为Pillow图像，方便后面的填充（padding）的图像处理操作
    to_pil = P.ToPIL()
    padding = P.Pad(40)
    squared_banana = padding(to_pil(squared_banana))
    print("Image.type: {}, Image.shape: {}".format(type(squared_banana), squared_banana.size))
    
    # 使用[matplotlib](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=2&q=matplotlib&zhida_source=entity)绘制图片处理前后的对比
    plt.rcParams['font.sans-serif']=['KaiTi']
    plt.subplot(1, 2, 1)
    plt.imshow(banana)
    plt.title("原始图片")
    plt.subplot(1, 2, 2)
    plt.imshow(squared_banana)
    plt.title("处理后图片")
    plt.show()

### **文本数据处理**

下面的例子展示了Eager模式在文本处理场景下的应用，进行分词以及[类型转换](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=1&q=%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2&zhida_source=entity)处理。
    
    
    import mindspore.dataset.text.transforms as text
    from mindspore import dtype as mstype
    
    # 定义WhitespaceTokenizer分词操作并且立即执行
    txt = "Welcome to Beijing !"
    txt = text.WhitespaceTokenizer()(txt)
    print("Tokenize result: {}".format(txt))
    
    # 定义ToNumber操作并立即执行
    txt = ["123456"]
    to_number = text.ToNumber(mstype.int32)
    txt = to_number(txt)
    print("ToNumber result: {}, type: {}".format(txt, type(txt[0])))

输出结果：
    
    
    Tokenize result: ['Welcome' 'to' 'Beijing' '!']   #分词结果
    ToNumber result: [123456], type: <class 'numpy.int32'>  #字符串到整数的转换结果

## **总结**

相比[流水线模式（[Pipeline模式](https://zhida.zhihu.com/search?content_id=203120696&content_type=Article&match_order=2&q=Pipeline%E6%A8%A1%E5%BC%8F&zhida_source=entity)）](https://zhuanlan.zhihu.com/p/517440240)中的实现代码，可以看到Eager模式的代码实现更为简单直接。所以对于不追求性能或者数据量小的场景，可以使用Eager模式快速实现功能。

参考资料

  1. [数据处理的Eager模式](https://link.zhihu.com/?target=https%3A//mindspore.cn/docs/programming_guide/zh-CN/r1.6/eager.html)



  


> 上一篇：[【AI设计模式】03-数据处理-流水线(Pipeline)模式](https://zhuanlan.zhihu.com/p/517440240)  
> 下一篇：[【AI设计模式】05-检查点模式（CheckPoints）：如何定期存储模型？](https://zhuanlan.zhihu.com/p/521311168)
