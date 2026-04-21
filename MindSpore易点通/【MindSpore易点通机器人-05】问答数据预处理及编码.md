# 【MindSpore易点通机器人-05】问答数据预处理及编码

原文链接：https://zhuanlan.zhihu.com/p/561830878

---

​

目录

在上一篇[【MindSpore易点通机器人-04】MLOps 环境搭建过程](https://zhuanlan.zhihu.com/p/514996638)，我们为大家讲述迭代0中具体的MLOps 环境搭建过程，本篇为大家介绍**问答数据[预处理](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=%E9%A2%84%E5%A4%84%E7%90%86&zhida_source=entity)及编码**。易点通机器人处理的目标数据范围包括MindSpore官网的FAQ文档、官网其它文档、论坛等问答数据，在机器人开发的第一个迭代中，我们把数据的范围限制的官网的QA数据。我们通过自动化脚本将官网的QA文档转换为期望的数据格式，然后通过Bert模型对所有的问题进行分词编码，作为后续相似度模型判别的数据。

## 问答数据初步处理

本项目是基于官网FAQ文档整理的数据集，用于做问答机器人的语料库。采用python语言，从官网中下载FAQ文档，并对其各项属性进行划分所得到的格式为.[csv](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=csv&zhida_source=entity)的数据集。其headline一共分为如下部分：title、link、content、process和tag。

FAQ数据来源于[MindSpore官网](https://link.zhihu.com/?target=http%3A//www.mindspore.cn/)，我们通过Python脚本从官网中下载FAQ文档，并对其各项属性进行划分所得到的格式为.csv的数据集，数据分为五列：

  * `title`：**FAQ中的questions** ，是可能出现的问题。采用的是文本的格式，去掉了一些xml标签，本身这些标签对于问题的表述没有较大影响。
  * `link`：**FAQ[语料库](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=2&q=%E8%AF%AD%E6%96%99%E5%BA%93&zhida_source=entity)的源网页地址链接**，共有十一个链接。
  * `content`:**FAQ中的answers** ，是对应问题的解答模块。与title一致，处理时去掉了一些xml标签，同时，还去掉的部分名词自带的网页链接（多是用于展开介绍）。
  * `process`: **问题类型** ，主要分为安装、[数据处理](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)、执行问题、编译、第三方框架迁移、调优、分布式配置、推理和特性咨询。其中“执行问题”，“第三方框架迁移”，“分布式配置”和“特性咨询”为新添加的process。其分类标准参照的是语料库的分类。
  * `tag`：**FAQ所涉及到的领域** ，暂时将tag设置为GPU、Ubuntu、MindInsight、MIndArmour、CPU、Windows、WSL、Ascend、Conda和Serving等。



数据处理的核心实现逻辑如下：
    
    
    #定义列表和特征字符
    url = ['https://mindspore.cn/docs/zh-CN/master/faq/installation.html', 'https://[mindspore](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=3&q=mindspore&zhida_source=entity).cn/docs/zh-CN/master/faq/data_processing.html', 'https://mindspore.cn/docs/zh-CN/master/faq/implement_problem.html', 'https://mindspore.cn/docs/zh-CN/master/faq/network_compilation.html', 'https://mindspore.cn/docs/zh-CN/master/faq/operators_compile.html', 'https://mindspore.cn/docs/zh-CN/master/faq/usage_migrate_3rd.html', 'https://mindspore.cn/docs/zh-CN/master/faq/performance_tuning.html', 'https://mindspore.cn/docs/zh-CN/master/faq/precision_tuning.html', 'https://mindspore.cn/docs/zh-CN/master/faq/distributed_configure.html', 'https://mindspore.cn/docs/zh-CN/master/faq/inference.html', 'https://mindspore.cn/docs/zh-CN/master/faq/feature_advice.html']
    process = ['#安装', '#数据处理', '#执行问题', '#网络编译', '#[算子编译](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=%E7%AE%97%E5%AD%90%E7%BC%96%E8%AF%91&zhida_source=entity)', '#第三方框架迁移使用', '#性能调优', '#精度调优', '#分布式配置', '#推理', '#特性咨询']
    process1 = ['安装', '数据处理', '执行问题', '编译', '编译', '第三方[框架迁移](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=4&q=%E6%A1%86%E6%9E%B6%E8%BF%81%E7%A7%BB&zhida_source=entity)', '[性能调优](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=2&q=%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98&zhida_source=entity)', '精度调优', '分布式配置', '推理', '特性咨询']
    tag = ['GPU', 'CuBLAS库', 'Linux', 'arm', 'macOS', 'SDK版本', 'SciPy', 'whl包', 'protobuf', 'Ubuntu', 'MindInsight', 'MIndArmour', 'CPU', 'Windows', 'WSL', 'Ascend', 'Conda', 'Serving', 'gmp', 'cuda', 'MindIR', 'API', 'MindRecord', 'JupyterLab', 'Generator Dataset', 'MindData', 'pipeline', 'Datalouder', 'Dataset', 'eval', 'Model', 'SGD', 'loss', 'PyTorch', 'NLP', 'ModelZoo', 'HCLL', 'ModelArts', 'PyNative', 'Graph', 'TensorFlow', 'C++', 'AIPP', 'OpenMPI', 'NCLL', 'RDMA', 'IB', 'RoCE', 'taichi', 'Caffe', 'NPU']
    tag_change = []
    str1 = "Q:"
    str2 = "A:"
    str3 = "Q："
    str4 = "A："
    qa = []
    #转义tag中的特殊符号
    for i in tag:
        tag_change.append(re.escape(i))
    #十一个语料库大循环
    for numbers in range(0, len(url)):
        #获取网页文本
        req = requests.get(url=url[numbers])
        req.encoding = 'utf-8'
        html = req.text
        bs = BeautifulSoup(html, "html.parser")#html.parser是解析器
        result = bs.select(process[numbers])
        body = result[0].get_text()
        #定义存放[特征值](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E5%80%BC&zhida_source=entity)位置的列表，其中i1、j1存放英文冒号的QA，wq1、wq2存放中文冒号的QA
        question_en = []
        answer_en = []
        wrong_question_en = []
        wrong_answer_en = []
        #开始查找，得到位置的列表
        question_en = find_en(str1, body)
        answer_en = find_en(str2, body)
        wrong_question_en = find_en(str3, body)
        wrong_answer_en = find_en(str4, body)
        #如果找到存在中文冒号的Q和A，将其插入
        correct(wrong_question_en, question_en)
        correct(wrong_answer_en, answer_en)
        #在完成上述补充后，还存在不对齐的现象，断定为缺少'A'字符
        if len(question_en) != len(answer_en):
            #print('存在缺少A的现象')
            for m in range(0, len(question_en)-2):
                if question_en[m+1] < answer_en[m]:
                    a = body.find('？', answer_en[m-1]+1, len(body))
                    answer_en.insert(m-1, a)
        #通过特征位置构造存放QA的列表
        question = []
        answer = []
        for m in range(0, len(question_en)-1):
            str5 = body[question_en[m]:answer_en[m]]
            question.append(str5)
        for n in range(0, len(answer_en)-2):
            str6 = body[answer_en[n]:question_en[n+1]]
            answer.append(str6)
        #最后的A一直到文本结束
        last = len(answer_en)-2
        str7 = body[answer_en[last]:]
        answer.append(str7)
        #在A中可能会出现小标题，进行去除
        delete(answer)
        #进行不区分大小写的查找tag
        tag1 = []
        tag2 = []
        for m in range(0, len(question)):
            str8 = question[m]+answer[m]
            for i in tag_change:
                t = re.search(i, str8, flags=re.IGNORECASE)
                if t != None:
                    tag2.append(t.group(0))
            tag1.append(tag2)
            tag2 = []
        #将列表元素合并为字符串
        FinalTag = []
        for m in range(0, len(question)):
            ft_str = '、'.join(tag1[m])
            FinalTag.append(ft_str)
        #将各个FAQ元素合并，用于写入csv文件
        for n in range(0, len(question)):
            qa.append([question[n], url[numbers],  answer[n], process1[numbers], FinalTag[n]])

数据整理完后的结果如下图：

图：数据集

其中tile表示对应的问题，content表示对应的答案，process表示数据的类型（FAQ，算子等）。

在获取到数据集的基础上，需要对数据进行筛选和编码。

## 数据筛选

读取csv文件，提取数据，并将问题和答案组织为K-V结构字典，具体代码实现如下：
    
    
    def load_data(file_path):
        result = {}
        with open(file_path) as f:
            reader = csv.reader(f)
            for item in reader:
                if reader.line_num == 1:
                    continue
                result[item[0]] = item[1]
        return result

## 对FAQ数据中的问题进行编码

使用Bert模型对所有的问题K进行编码，输出[向量](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=%E5%90%91%E9%87%8F&zhida_source=entity)。将其组织称为[K-V](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=2&q=K-V&zhida_source=entity)结构的字典，具体的代码实现如下：
    
    
    def encode_resource_sentences(resource_sentences):
        tokenizer = BertTokenizer.load('sentence-transformers/all-MiniLM-L6-v2')
        model = BertModel.load('sentence-transformers/all-MiniLM-L6-v2')
        resource_sentences_encode = {}
        for sentence in resource_sentences:
            resource_token = mindspore.Tensor([tokenizer.encode(sentence, add_special_tokens=True)],
                                              mindspore.int32)
            resource_sentences_encode[sentence] = model(resource_token)[1][0].asnumpy().tolist()
        with open("../data/resource_sentence_encode.json", "w", encoding='utf-8') as f:
            json.dump(resource_sentences_encode, f)
        with open("../qaRobot/qa_api/data/resource_sentence_encode.json", "w", encoding='utf-8') as f:
            json.dump(resource_sentences_encode, f)

将步骤数据筛选后的数据导出为json文件。
    
    
    def dump_q_a(result):
        with open("../data/q_a.json", "w", encoding='utf-8') as f:
            json.dump(result, f)
        with open("../qaRobot/qa_api/data/q_a.json", "w", encoding='utf-8') as f:
            json.dump(result, f)

使用[main函数](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=1&q=main%E5%87%BD%E6%95%B0&zhida_source=entity)将上述三个步骤串联起来：
    
    
    if __name__ == '__main__':
        r = load_data("../../data/data.csv")
        encode_resource_sentences(r.keys())
        dump_q_a(r)

对数据梳理结束后我们得到了两个json文件，作为[模型判别](https://zhida.zhihu.com/search?content_id=212983923&content_type=Article&match_order=2&q=%E6%A8%A1%E5%9E%8B%E5%88%A4%E5%88%AB&zhida_source=entity)的输入。

  * `q_a.json`: 存储问题K和答案V
  * `resource_sentence_encode.json`: 存储问题K和对应的经过Bert模型编码向量。



> 完整的代码逻辑见链接 [prepare_data.py](https://link.zhihu.com/?target=https%3A//gitee.com/msu-sig/robot/blob/master/src/model/prepare_data.py)

完成数据处理和编码后，我们就可以进入下一个阶段，通过`all-MiniLM-L6-v2`模型判别提问的相似度来返回准确的答案。

  


> 上一篇：[【MindSpore易点通机器人-04】MLOps 环境搭建过程](https://zhuanlan.zhihu.com/p/514996638)  
> 下一篇：[【MindSpore易点通机器人-06】基于相似度模型实现问答匹配及推荐功能](https://zhuanlan.zhihu.com/p/562778149)
