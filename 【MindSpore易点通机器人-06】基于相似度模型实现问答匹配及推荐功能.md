# 【MindSpore易点通机器人-06】基于相似度模型实现问答匹配及推荐功能

原文链接：https://zhuanlan.zhihu.com/p/562778149

---

​

目录

在上一篇[【MindSpore易点通机器人-05】问答数据[预处理](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=%E9%A2%84%E5%A4%84%E7%90%86&zhida_source=entity)及编码](https://zhuanlan.zhihu.com/p/561830878)，我们为大家讲述了机器人问答数据预处理及编码，本篇为大家介绍**机器人基于什么模型实现问答匹配及推荐功能** 。

答案搜索的核心逻辑是使用用户的输入去匹配知识库中的问题，然后返回匹配度最高的问题的答案。在第一个迭代开发中，我们的数据集规模比较局限，因此，不需要使用复杂的模型来实现QA和推荐功能。这里我们使用了一个**基于相似度的简单模型all-MiniLM-L6-v2** ，通过对相似度的判断来**实现问答以及推荐功能** ：

  1. 问题和知识库中的问题**相似度超过70%** ，**返回最匹配的答案** ； 
  2. 问题和知识库中的问题**相似度低于70%** ，**返回相似度最高的两个问题，作为推荐提问** 。



## 关于模型

`[all-MiniLM-L6-v2](https://link.zhihu.com/?target=https%3A//huggingface.co/sentence-transformers/all-MiniLM-L6-v2)`是`sentence-transformers库`(将文本，图片等向量化并进行文本相似性、语义搜索、同义词挖掘等任务的库)的一个SOTA模型，它可以将句子或者段落映射到384维的[向量空间](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=%E5%90%91%E9%87%8F%E7%A9%BA%E9%97%B4&zhida_source=entity)中，常用于聚类和语义搜索任务。

[MiniLM-V2](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2012.15828)论文本身介绍了面向Transformer-based预训练语言模型的深度自注意力知识蒸馏（Deep Self-Attention Distillation）MiniLM的通用压缩方法，MiniLM的蒸馏方法简单有效，由不同预训练大模型压缩得到的单语和多语MiniLM预训练模型不仅更小更快，而且在多语言理解和生成任务上效果显著。

MiniLM-V2结构，引入了多头注意力机制的蒸馏策略

## 具体实现

为了快速实现构建原型能力，我们在实现时使用MindSpore的Bert模型直接加载了预训练好的`sentence-transformers/all-MiniLM-L6-v2`模型权重，并对输入的句子进行编码。然后将完成编码的句子和[数据处理](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)中编码好的语料（FAQ中的QA数据集）进行相似度对比，根据对比的结果来判断返回最佳的匹配答案或者推荐其它相关的问题。

如下是代码的具体实现，首先调用encode_sentence对用户的输入进行向量化编码。
    
    
    def encode_sentence(input_sentence):
        tokenizer = BertTokenizer.load('sentence-transformers/all-MiniLM-L6-v2')
        model = BertModel.load('sentence-transformers/all-MiniLM-L6-v2')
        model.set_train(False)
        input_token = [mindspore](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=mindspore&zhida_source=entity).Tensor([tokenizer.encode(input_sentence, add_special_tokens=True)], mindspore.int32)
        return model(input_token)

其次使用该[向量](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=4&q=%E5%90%91%E9%87%8F&zhida_source=entity)去知识库中进行匹配并获取对应的问题，在这里，我们通过`cosine_simalarity`（[余弦相似度](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=%E4%BD%99%E5%BC%A6%E7%9B%B8%E4%BC%BC%E5%BA%A6&zhida_source=entity)方法）计算两个向量的相似度，相似度最高，且超过0.7的，就是我们需要返回的最佳答案。
    
    
    def compute_similarity(input_encode):
        with open("../data/resource_sentence_encode.json") as f:
            resource_sentences_encode = json.load(f)
            question_similarity = {}
            for k, v in resource_sentences_encode.items():
                similarity = [cosine_similarity](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=cosine_similarity&zhida_source=entity)(
                    [input_encode[1][0].asnumpy()], [np.asarray(v)])
                question_similarity[k] = similarity
            sorted_similarity = sorted(question_similarity.items(), key=lambda x: x[1], reverse=True)
            if not sorted_similarity or len(sorted_similarity) == 0:
                return None
            elif sorted_similarity[0][1] >= 0.7:
                return [sorted_similarity[0][0]]
            elif len(sorted_similarity) > 1 and sorted_similarity[0][1] < 0.7:
                return [sorted_similarity[0][0], sorted_similarity[1][0]]

如果用户输入的问题在知识库中所匹配到的问题相似度都小于0.7，则会返回相似度最高的前两个问题，作为推荐的提问问题，供用户参考。

## 功能验证

将上述步骤进行组合后，我们就能在本地通过命令行来验证[问答机器人](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=%E9%97%AE%E7%AD%94%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity)，完整步骤如下：

  1. 获取用户输入。
  2. 对用户输入进行编码得到对应向量。
  3. 从知识库中使用输入向量进行匹配，获取匹配到的问题。
  4. 从知识库中获取匹配到的问题的答案，输出给用户。



对应的代码实现如下：
    
    
    if __name__ == '__main__':
        input_sentence = sys.argv[1]
        input_encode = encode_sentence(input_sentence)
        match_keys = compute_similarity(input_encode)
        q_a_data = load_q_a_data("../data/q_a.json")
        if match_keys:
            for k in match_keys:
                if k in q_a_data.keys():
                    print(q_a_data[k])

在本地，通过命令行执行脚本，我们就可以测试模型的效果了，如我们提问`是否可以转AIR模型`，则会返回一个最接近的答案，这个答案会和FAQ中的问题/答案进行匹配：
    
    
    python robot/src/model/q_a.py 是否可以转AIR模型
    Ascend 310不能导出AIR，需要在Ascend 910加载训练好的[checkpoint](https://zhida.zhihu.com/search?content_id=213194344&content_type=Article&match_order=1&q=checkpoint&zhida_source=entity)后，导出AIR，然后在Ascend 310转成OM模型进行推理。Ascend 910的安装方法可以参考官网MindSpore安装指南(https://www.mindspore.cn/install)。

## 总结

至此，我们基于`sentence-transformers/all-MiniLM-L6-v2`预训练模型，通过相似度对比，快速实现了对于易点通机器人QA以及简单推荐功能的支持。那么下一步就要把这个功能封装成一个REST API，在云上部署起来，提供给IDE或者其它客户端使用。

## 参考资料

  1. [5分钟 NLP系列 — SentenceTransformers 库介绍](https://zhuanlan.zhihu.com/p/457876366)
  2. [NeurIPS 2020 | MiniLM：预训练语言模型通用压缩方法](https://zhuanlan.zhihu.com/p/298390577)



  


> 上一篇：[【MindSpore易点通机器人-05】问答数据预处理及编码](https://zhuanlan.zhihu.com/p/561830878)  
> 下一篇：[【MindSpore易点通机器人-07】部署易点通机器人](https://zhuanlan.zhihu.com/p/566066575)
