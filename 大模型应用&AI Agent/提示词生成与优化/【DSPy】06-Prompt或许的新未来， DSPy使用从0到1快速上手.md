# 【DSPy】06-Prompt或许的新未来， DSPy使用从0到1快速上手

原文链接：https://zhuanlan.zhihu.com/p/707925423

---

​

目录

## 概述

[DSPy](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)，即声明式语言模型编程（Declarative Language Model Programming），旨在简化复杂语言模型应用的构建过程。由[斯坦福大学](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=%E6%96%AF%E5%9D%A6%E7%A6%8F%E5%A4%A7%E5%AD%A6&zhida_source=entity)的研究人员开发，DSPy允许开发者专注于应用程序的高级逻辑，同时抽象掉许多低级细节。创造了一种提示词工程的新范式，对目前提示词工程中的共识问题（如提示词的脆弱性、迭代成本高、缺乏系统化方法、范式繁多、依靠人类经验 ）等诸多问题，给出了新的解决思路。

本文将首先以示例驱动的方式，由浅入深的介绍DSPy框架自身的使用流程，然后结合知名大模型应用框架[LangChain](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=LangChain&zhida_source=entity)，进一步介绍两个框架结合的案例。

## DSPy从0到1使用教程

DSPy主要包含了 Signature、Data、[Optimizer](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=Optimizer&zhida_source=entity)、Module、Predict等几个模块，在下面的例子中将依次介绍。

准备工作：

1、在使用DSPy之前，请先确保已经安装了 DSPy的python包，可参考[Installation | DSPy (dspy-docs.vercel.app)](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/docs/quick-start/installation)

2、本文代码涉及本地部署的ollama相关模型，可参考[ollama/ollama: Get up and running with Llama 3, Mistral, Gemma, and other large language models. (github.com)](https://link.zhihu.com/?target=https%3A//github.com/ollama/ollama)

DSPy工作流

### 迭代1-跑通极简的CoT思维模式（无示例）

目标：三分钟跑通极简demo

首先介绍利用DSPy包，跑通最简单的一次CoT思维模式问答，该示例涉及到 **dspy.Signature** 和 **dspy.[ChainOfThought](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=ChainOfThought&zhida_source=entity)** 类，其中 **dsyp.Signature** 是定义模块输入输出的类**，dspy.ChainOfThought** 为DSPy内置的思维模式类，以CoT模式与大模型交互，具体代码如下
    
    
    import dspy
    
    #  定义并设置大模型
    model_name = 'llama3'
    lm = dspy.[OllamaLocal](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=OllamaLocal&zhida_source=entity)(model=model_name)
    dspy.settings.configure(lm=lm)
    
    #  定义输入输出参数 类定义方式
    
    class QA(dspy.Signature):
        question = dspy.InputField()
        answer = dspy.OutputField()
    
    question = "what is the color of the sea?"
    summarize = dspy.ChainOfThought(QA)
    response = summarize(question=question)
    
    print(f"问题：{question} \n答案：{response.answer}")

上述代码首先定义了大模型使用 llama3 ，然后，定义了 **dspy.Signature** 类，输入字段为 question， 输出字段为 answer，最后实例化 **dspy.ChainOfThought** 类，并输入问题调用大模型进行回答，执行结果为
    
    
    # #  类定义方式 定义输入输出参数 - start # # 
    
    问题：what is the color of the sea? 
    答案：The color of the sea is typically perceived as blue.
    
    # #  类定义方式 定义输入输出参数 - end # # 

此外，depy.Signature 类还支持以inline的方式定义，通过简单的字符串描述输入输出字段，如下所示，在第2行直接将原来的 QA 替换为了字符串 "question->answer"，这种写法可以自动的被转换成 **dspy.Signature** 类：
    
    
    question = "what is the color of the sky?"
    summarize = dspy.ChainOfThought('question -> answer')
    response = summarize(question=question)

结果为：
    
    
    # #  inline方式 定义输入输出参数 - start # # 
    
    问题：what is the color of the sky? 
    答案：Blue
    
    # #  inline方式 定义输入输出参数 - end # # 

如果希望查看 CoT模式的详细提示词，可以运行如下代码：
    
    
    lm.inspect_history(n=1)

结果为
    
    
    Question: what is the color of the sky?
    Reasoning: Let's think step by step in order to Question: what is the color of the sky?
    Reasoning: Let's think step by step in order to determine the color of the sky. The sky appears blue due to the scattering of light waves in the atmosphere. The blue light is scattered more efficiently than other colors of light, which is why the sky appears blue.
    Answer: Blue

### 迭代2-为CoT模式提示词增加示例

目标：通过增加示例提升CoT的成功率

众所周知，通过调整提示词的内容，可以改变大模型返回的结果，比较常见的一种调整提示词的办法是为提示词增加示例，那么，如何给 CoT 模式增加示例呢？此处将引入 dspy.Example 类，是DSPy的数据类，用于构建示例，代码如下
    
    
    import dspy
    
    model_name = 'llama3'
    lm = dspy.OllamaLocal(model=model_name)
    dspy.settings.configure(lm=lm)
    
    question = "what is the color of sky at night?"
    #  示例内容
    example = dspy.Example(question="what is the color of sky?", answer="the color of sky is blue, even at night")
    summarize = dspy.ChainOfThought('question -> answer')
    response = summarize(question=question, demos=[example])
    
    print(f"问题：{question} \n答案：{response.answer}")

可以看出，上述代码构建了 example实例，并将该实例加入到 dspy.ChainOfThought 的运行函数中，结果如下：
    
    
    问题：what is the color of sky at night? 
    答案：...the color of the sky at night is still blue!

进一步查看提示词内容，
    
    
    ---
    
    Question: what is the color of sky?
    Answer: the color of sky is blue, even at night
    
    ---
    
    Question: what is the color of sky at night?
    Reasoning: Let's think step by step in order to Question: what is the color of sky at night?
    Reasoning: Let's think step by step in order to answer this question. We know that during the day, the color of the sky is blue, and we also know that the color of the sky remains relatively consistent even after sunset. Therefore...
    Answer: ...the color of the sky at night is still blue!

可以看出，提示词中增加了示例的内容，这影响了问题最终的答案。

### 迭代3-构建示例数据集

目标：通过引入标准数据集，进一步提升CoT的成功率

随着我们示例的增加，我们希望通过构建一个数据集来保存示例，并将整个数据集传入到CoT的推理过程中，这里，我们引入了DSPy内置的数据集[GSM8K](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=GSM8K&zhida_source=entity)，这是一个多步骤数学推理数据集，示例如下：
    
    
    import dspy
    from get_dataset import custom_trainset as trainset
    #   example of transet :  Example({'question': '1+5=?', 'answer': '6'}) (input_keys={'question'})
    gsm8k_trainset = gsm8k.train[:10]
    
    model_name = 'llama3'
    lm = dspy.OllamaLocal(model=model_name, timeout_s=1000)
    dspy.settings.configure(lm=lm)
    
    question = "3+3+5=?"
    #  示例内容
    summarize = dspy.ChainOfThought('question -> answer')
    response = summarize(question=question, demos=gsm8k_trainset)
    
    print(f"问题：{question} \n答案：{response.answer}")

此处将数据集传入到demos变量中，并询问了一个和数学计算有关的问题，结果如下：
    
    
    问题: 3+3+5=?
    答案: 8

实际上这个答案是错误的，正确的答案是11，因此提示词可以优化大模型的输出，但是也很难保证回答问题的准确性。提示词如下：
    
    
    Given the fields `question`, produce the fields `answer`.
    
    ---
    
    Follow the following format.
    
    Question: ${question}
    Reasoning: Let's think step by step in order to ${produce the answer}. We ...
    Answer: ${answer}
    
    ---
    
    Question: 1+1=?
    Answer: 2
    
    ---
    
    Question: 5*5=?
    Answer: 25
    
    ---
    
    Question: 1+5=?
    Answer: 6
    
    ---
    
    Question: 3+3=?
    Answer: 6
    
    ---
    
    Question: 6+6=?
    Answer: 12
    
    ---
    
    Question: 3+3+5=?
    Reasoning: Let's think step by step in order to Here is the completed response:
    
    ---
    
    Question: 3+3+5=?
    Reasoning: Let's think step by step in order to get the answer. We need to add 3 and 3 first, which gives us 6, then we can add 5 to that result.
    Answer: 8

### 迭代4-自动优化模板

目标：通过自优化的方式，调整访问模型参数（如Temperature）、筛选影响度较高的示例，进一步提升CoT的准确率

通过扩大示例可以提高大模型回答问题的准确性，但是存在2个问题，（1）增加示例也并不能保证大模型预测准确（2）当大模型调整时原有提示词可能是不适用的，针对于这两个问题，DSPy提供了对提示词和大模型参数进行自动化优化的功能，可以进一步提高模型的准确性以及在不同大模型上的稳定性。用户只需要提供目标领域的训练数据集 Dataset，以及衡量大模型返回结果准确性的衡量标准 Metrics，以及优化器 Optimizer，就可以自动的得到一个最适合目标场景数据集的提示词和模型参数。

首先，在优化之前，我们使用dspy的Evaluate类以及Metrics测试当前CoT模块在测试集上的准确率：
    
    
    from get_dataset import gsm_testset as test_set
    import dspy
    from dspy.datasets.gsm8k import gsm8k_metric
    from dspy.teleprompt import [BootstrapFewShot](https://zhida.zhihu.com/search?content_id=245448406&content_type=Article&match_order=1&q=BootstrapFewShot&zhida_source=entity)
    
    #  定义并设置大模型
    model_name = 'llama3'
    lm = dspy.OllamaLocal(model=model_name, timeout_s=1000)
    dspy.settings.configure(lm=lm)
    
    class CoT(dspy.Module):
        def __init__(self):
            super().__init__()
            self.prog = dspy.ChainOfThought("question -> answer")
    
        def forward(self, question):
            return self.prog(question=question)
    
    cot = CoT()
    evaluate = Evaluate(devset=test_set, metric=gsm8k_metric, num_threads=4, display_progress=True, display_table=0)
    evaluate(cot)

结果如下:

可见，在优化之前，基于llama3模型在测试集上的准确率为80%，接下来利用dspy对CoT进行优化，
    
    
    from get_dataset import custom_trainset as trainset
    
    config = dict(max_bootstrapped_demos=4, max_labeled_demos=4)
    
    #  Optimize! Use the `gsm8k_metric` here. In general, the metric is going to tell the optimizer how well it's doing.
    teleprompter = BootstrapFewShot(metric=gsm8k_metric, **config)
    #  可以调整 train_set 长度
    optimized_cot = teleprompter.compile(CoT(), trainset=trainset)
    optimized_cot.save("./test.json")
    question = "3+3+5=?"
    response = optimized_cot(question=question)
    print(f"问题：{question} \n答案：{response.answer}")

上述代码中首先导入了内置的优化器 **BootstrapFewShot** 、度量函数 **gsm8k_metric** 、 以及gsm8k数据集的前20条数据即 **gsm8k_trainset** ，利用 **BootstrapFewShot.compile** 函数进行优化，其中内部保留了对回答问题有利的 示例 以及 大模型参数，最终提问，结果如下
    
    
    问题: 3+3+5=?
    答案: 11

接下来，我来再次利用 dspy.Evaluate 验证优化后的CoT实例的准确率，
    
    
    evaluate(optimized_cot)

结果在测试集上达到了100%的预测准确率

### 迭代5-加载训练参数

目标：通过加载预训练参数，省去重复训练流程
    
    
    optimized_cot = CoT()
    optimized_cot.load("./test.json")
    optimized_cot(question=question)

## DSPy 与 LangChain 配合使用示例

DSPy有着强大的自动化提示词优化能力，在使用过程中，可能有一部分开发者已经使用LangChain制作过智能体，也有优化的需求。因此，DSPy官方对LangChain的部分组件进行了适配，使得利用DSPy 可以优化由LangChain 创建的“链 (Chain)”；此外DSPy的部分功能如Retriever，也可以融合到LangChain的链中。综上所述，LangChain用户将获得通过任何DSPy优化器优化链的能力。DSPy用户将能够将DSPy程序导出到LCEL中。

示例链接：dspy/examples/tweets/compiling_langchain.ipynb at main · stanfordnlp/dspy ([http://github.com](https://link.zhihu.com/?target=http%3A//github.com))

本文逐步介绍如何利用DSPy优化LangChain的部分模块

首先，我们可以通过LangChain的基本语法定义一条链，此链融入了dspy.Retrieve模块，代码如下
    
    
    import dspy
    from dspy.evaluate.evaluate import Evaluate
    from dspy.teleprompt import BootstrapFewShotWithRandomSearch
    from langchain_openai import OpenAI
    from langchain.globals import set_llm_cache
    from langchain.cache import SQLiteCache
    # From LangChain, import standard modules for prompting.
    from langchain_core.prompts import PromptTemplate
    from langchain_core.output_parsers import StrOutputParser
    from langchain_core.runnables import RunnablePassthrough
    colbertv2 = dspy.ColBERTv2(url='http://20.102.90.50:2017/wiki17_abstracts')
    dspy.configure(rm=colbertv2)
    set_llm_cache(SQLiteCache(database_path="cache.db"))
    llm = OpenAI(model_name="gpt-3.5-turbo-instruct", temperature=0)
    retrieve = lambda x: dspy.Retrieve(k=5)(x["question"]).passages
    # Just a simple prompt for this task. It's fine if it's complex too.
    prompt = PromptTemplate.from_template("Given {context}, answer the question `{question}` as a tweet.")
    # This is how you'd normally build a chain with LCEL. This chain does retrieval then generation (RAG).
    init_chain = RunnablePassthrough.assign(context=retrieve) | prompt | llm | StrOutputParser()

上述代码中展示了如何利用LangChain创建检索、提示词、大模型、输出处理模块，并串成了一条链，其中检索模块使用了DSPy提供的基础检索功能。

那么，如何将上述的LangChain的链，封装为DSPy的Module，使得这条链可以被优化呢？

如下面代码所示：
    
    
    # From DSPy, import the modules that know how to interact with LangChain LCEL.
    from dspy.predict.langchain import LangChainPredict, LangChainModule
    # This is how to wrap it so it behaves like a DSPy program.
    # Just Replace every pattern like `prompt | llm` with `LangChainPredict(prompt, llm)`.
    zeroshot_chain = RunnablePassthrough.assign(context=retrieve) | LangChainPredict(prompt, llm) | StrOutputParser()

如上述代码，首先需要导入LangChainPredict类，该类集成自 langchain_core.runnables.Runnable 类 以及 dspy.Predict类，因此可以串联到链中。利用LangChainPredict包装原来的 提示词和大模型模块 并串联到链中，目前这条链与init_chain 中实现的功能完全一致。

然后，需要导入LangChainModule，并将整条链传入LangChainModule中初始化，zeroshot_chain，如下面代码所示
    
    
    zeroshot_chain = LangChainModule(zeroshot_chain)  # then wrap the chain in a DSPy module.
    question = "In what region was Eddy Mazzoleni born?"
    zeroshot_chain.invoke({"question": question})

LangChainModule集成自dspy.Module，因此可以被优化器优化，此外LangChainModule实现了invoke方法，与Langchain的使用习惯一致。

至此，我们已经得到了一个被DSPy修饰过的LangChain的链。

接下来，我们可以使用优化器来优化这条链。在优化之前，我们先利用Metrics 计算zero_shot 在特定数据集上的准确率
    
    
    from tweet_metric import metric, trainset, valset, devset
    evaluate = Evaluate(metric=metric, devset=devset, num_threads=8, display_progress=True, display_table=5)
    evaluate(zeroshot_chain)

结果为42.7%，在150个测试的问题中，有64个是回答符合Metrics的。

接下来，利用优化器进行优化，并再一次调用 evaluate函数进行验证，如下面代码和图片所示
    
    
    # Set up the optimizer. We'll use very minimal hyperparameters for this example.
    # Just do random search with ~3 attempts, and in each attempt, bootstrap <= 3 traces.
    optimizer = BootstrapFewShotWithRandomSearch(metric=metric, max_bootstrapped_demos=3, num_candidate_programs=3)
    # Now use the optimizer to *compile* the chain. This could take 5-10 minutes, unless it's cached.
    optimized_chain = optimizer.compile(zeroshot_chain, trainset=trainset, valset=valset)
    evaluate(optimized_chain)

可看出，优化之后optimized_chain的准确率有着明显的提高，达到了52.4%。此外，用户可以通过调整优化器的参数，使得得到的结果进一步提高。

## 总结

本文首先通过一个简单的例子，解释了DSPy如何从最基础的无示例CoT模块问答、到有示例CoT模块、再到构建数据集，并利用Metrics和Optimizer优化Module获得更高的预测准确率的预测和验证过程。然后展示了利用DSPy对LangChain的链的优化、测试、验证过程。DSPy可以有效降低提示词在切换大模型时的脆弱性、并且将提示词开发从手工开发转为抽象的代码开发，提供了更加系统化的方法。DSPy所提出的新范式，将对未来的大模型应用平台技术产生深远影响。

## 相关链接

  1. [dspy官网](https://link.zhihu.com/?target=https%3A//wiki.huawei.com/domains/30974/wiki/49918/WIKI202406133768336) [https://dspy-docs.vercel.app](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/)
  2. Dspy github仓库地址 [https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)
  3. 示例代码地址 [https://github.com/ziqi-jin/dspy-examples](https://link.zhihu.com/?target=https%3A//github.com/ziqi-jin/dspy-examples)
  4. Langchain 介绍文档[https://python.langchain.com/v0.1/docs/integrations/providers/dspy/](https://link.zhihu.com/?target=https%3A//python.langchain.com/v0.1/docs/integrations/providers/dspy/)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】05-DSPy的“前世今生”，从DSPy的核心论文解析其技术演进之路](https://zhuanlan.zhihu.com/p/707184607)  
> 下一篇：[【DSPy技术洞察】07-丝分缕解！带你了解DSPy核心模块的源码实现之Signature类](https://zhuanlan.zhihu.com/p/708332838)
