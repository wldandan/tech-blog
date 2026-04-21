# 【DSPy】11-DSPy和Langchain的无缝集成

原文链接：https://zhuanlan.zhihu.com/p/714868095

---

​

目录

## 概述

随着[大型语言模型](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLMs）和向量存储的不断强大，出现了一代新的框架，能够通过利用LLMs和向量搜索技术来简化AI应用程序的开发。这些框架简化了从[检索增强生成](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=%E6%A3%80%E7%B4%A2%E5%A2%9E%E5%BC%BA%E7%94%9F%E6%88%90&zhida_source=entity)（RAG）应用程序到具有先进会话能力的复杂[聊天机器人](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=%E8%81%8A%E5%A4%A9%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity)的构建过程，甚至还能支持复杂的推理驱动的AI应用。

其中最知名的框架可能是[LangChain](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=LangChain&zhida_source=entity)。该项目由Harrison Chase于2022年10月推出，并迅速获得了广泛关注，吸引了数百位开发者在GitHub上贡献代码。LangChain在对文档、数据源和API的广泛支持方面表现出色。此外，它与[Qdrant](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=Qdrant&zhida_source=entity)等向量存储的无缝集成以及能够链接多个LLM，使得开发者能够构建复杂的AI应用，而无需重新发明轮子。

然而，尽管像LangChain这样强大的框架已经解锁了许多能力，开发者仍需掌握[提示工程](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=%E6%8F%90%E7%A4%BA%E5%B7%A5%E7%A8%8B&zhida_source=entity)的专业知识，以编写最佳的LLM提示。此外，优化这些提示并适应多阶段推理AI的构建在现有框架中仍然是一项挑战。

事实上，当你开始构建生产级AI应用程序时，你应该会知道单次的LLM调用不足以释放LLMs的全部能力。相反，你需要创建一个工作流程，让模型与外部工具（如网络浏览器）互动，提取文档中的相关片段，并将结果汇总到一个[多阶段推理管道](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=%E5%A4%9A%E9%98%B6%E6%AE%B5%E6%8E%A8%E7%90%86%E7%AE%A1%E9%81%93&zhida_source=entity)中。

这涉及构建一个结合并推理中间输出的体系结构，并根据任务要求调整LLM提示，以生成最终输出。对于这样的场景，手动提示工程的方法很快就会显得不够用。

2023年10月，[斯坦福NLP](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=%E6%96%AF%E5%9D%A6%E7%A6%8FNLP&zhida_source=entity)的研究人员发布了一个库[DSPy](https://zhida.zhihu.com/search?content_id=246989508&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)，该库完全自动化了大型语言模型（LLMs）的提示和权重优化过程，从而消除了手动提示或提示工程的需求。

DSPy的一个关键特性是它能够自动调整LLM提示，这种方法在你的应用需要在管道中多次调用LLM时尤其强大。

因此，在构建基于LLM和向量存储的AI应用时，你应该选择哪个框架？在本文中，我们将深入探讨每个框架的能力，并讨论它们各自的适用场景。

## DSPy vs LangChain

DSPy和LangChain都是构建AI应用程序的强大框架，利用大型语言模型（LLMs）和向量搜索技术。以下是它们的关键特性、性能和使用场景的比较分析：

特点| LangChain| DSPy  
---|---|---  
核心关注点| 提供大量构建模块，简化使用LLMs与用户指定数据源结合的应用程序开发。| 自动化和模块化LLM交互，消除手动提示工程，提高系统可靠性。  
方法| 利用模块化组件和可以使用LangChain表达语言（LCEL, LangChain Expression Language）链接在一起的链。| 通过编程而非提示来简化LLM交互，并自动优化提示和权重。  
复杂管道| 通过LCEL创建链，支持异步执行和与各种数据源及API的集成。| 使用模块和优化器简化多阶段推理管道，并通过减少手动干预确保可扩展性。  
优化| 依赖用户的提示工程和多个LLM调用的链式操作。| 内置优化器自动调整提示和权重，提高LLM管道的效率和效果。  
社区与支持| 拥有庞大的开源社区，文档丰富，示例众多。| 新兴框架，社区支持不断增长，带来LLM提示的新范式。  
  
### LangChain

  
**优势：**

  * **数据源和API** ：LangChain支持多种数据源和API，允许与不同类型的数据无缝集成，非常适用于各种AI应用。
  * **模块化组件** ：LangChain提供的模块化组件可以组合在一起，LangChain表达语言（LCEL）使得使用声明性语法构建和管理工作流程变得更容易。
  * **丰富的文档和示例** ：作为较早的框架，LangChain拥有丰富的文档和成千上万的示例，开发者可以从中获得灵感。



  
**劣势：**

  * **复杂推理任务** ：对于涉及复杂多阶段推理任务的项目，LangChain需要大量的手动提示工程，这既耗时又容易出错。
  * **可扩展性问题** ：管理和扩展需要多个LLM调用的工作流可能非常具有挑战性。
  * **提示工程需求** ：开发者需要对提示工程有深入理解，才能构建需要多



### DSPy

**优势：**

  * DSPy 自动化了提示生成和优化过程，显著减少了手动提示设计的需求，使得使用大型语言模型（LLMs）更为容易，并有助于构建可扩展的AI工作流。
  * 该框架包含内置的优化器，如 BootstrapFewShot 和 MIPRO，可以自动精化提示并将其适配到特定的数据集。
  * DSPy 使用通用模块和优化器简化了提示设计的复杂性，使得创建复杂多步骤推理应用变得更为容易，无需担心处理LLMs的复杂细节。
  * DSPy 支持多种LLMs，并具有在同一程序中使用多个LLMs的灵活性。
  * 通过关注编程而非提示，DSPy 确保了AI应用的可靠性和性能，尤其是在需要复杂多阶段推理的情况下。



**劣势：**

  * 作为一个较新的框架，DSPy 的社区比 LangChain 小，这意味着资源、示例和社区支持的可用性较为有限。
  * 尽管 DSPy 提供了教程和指南，但其文档比 LangChain 的文档要少，这可能会在您开始使用时带来挑战。
  * 在开始使用 DSPy 时，您可能会感到被它提供的模式和模块所限制。



## LangChain和DSPy的结合

下面我们将从两个方向来解析LangChain和DSPy的结合。第一，我们先介绍如何将LangChain集成到DSPy中。其次，我们再分析一下，如何将DSPy集成到LangChain，该方向尚处于一个探索阶段，目前LangChain暂不支持。

### 将LangChain集成到DSPy

为了适配LangChain，使得LCEL可以使用DSPy优化，DSPy官方提供 LangChainModule 和 LangChainPredict两个类，这两个类将LangChain的模块分别继承自 dspy.Module 和 dspy.Predict 类，同时也实现了必要的方法。

LangChainModule：在DSPy中，预测、优化、验证的程序主体为Module，LangChainModule可以将完整的LCEL封装，使得在不改变DSPy原有 Optimizer.compile() 和 Evaluate.evalute() 接口的情况下，无缝传入，完成对LCEL中的提示词和大模型组件的优化。
    
    
    class LangChainModule(dspy.Module):
        def __init__(self, lcel):
            super().__init__()
     
            modules = []
            for name, node in lcel.get_graph().nodes.items():
                if isinstance(node.data, LangChainPredict): modules.append(node.data)
    
            self.modules = modules
            self.chain = lcel
     
        def forward(self, **kwargs):
            output_keys = ['output', self.modules[-1].output_field_key]
            output = self.chain.invoke(dict(**kwargs))
     
            try: output = output.content
            except Exception: pass
    
            return dspy.Prediction({k: output for k in output_keys})
     
        def invoke(self, d, *args, **kwargs):
            return self.forward(**d).output

如上代码所示，LangChainModule继承了dspy.Moudle类，实现了dspy.Module必要的 forward方法，在forward方法中，主要的处理逻辑为 self.chain.invoke，也就是执行LCEL，执行之后再将结果封装为DSPy标准的dspy.Prediction类对象返回。

LangChainPredict：被优化和预测的主要逻辑是在Predict类中，DSPy提供的LangChainPredict可以覆盖Predict类的功能，同时又有LCEL的特定，使得LangChainPredict类可以和其他langChain.Runnable子类一起串联到LCEL中。
    
    
    class LangChainPredict(Predict, Runnable): # , RunnableBinding):
        class Config: extra = Extra.allow  #  Allow extra attributes that are not defined in the model
    
        def __init__(self, prompt, llm, **config):
            Runnable.__init__(self)
            Parameter.__init__(self)
    
            self.langchain_llm = ShallowCopyOnly(llm)
    
            try: langchain_template = '\n'.join([msg.prompt.template for msg in prompt.messages])
            except AttributeError: langchain_template = prompt.template
    
            self.stage = random.randbytes(8).hex()
            self.signature, self.output_field_key = self._build_signature(langchain_template)
            self.config = config
            self.reset()
    
        def reset(self):
            ...
    
        def dump_state(self):
            ...
    
        def load_state(self, state):
            ...
     
        def __call__(self, *arg, **kwargs):
            if len(arg) > 0: kwargs = {**arg[0], **kwargs}
            return self.forward(**kwargs)
     
        def _build_signature(self, template):
            gpt4T = dspy.OpenAI(model='gpt-4-1106-preview', max_tokens=4000, model_type='chat')
    
            with dspy.context(lm=gpt4T): parts = dspy.Predict(Template2Signature)(template=template)
    
            inputs = {k.strip(): OldInputField() for k in parts.input_keys.split(',')}
            outputs = {k.strip(): OldOutputField() for k in parts.output_key.split(',')}
    
            for k, v in inputs.items():
                v.finalize(k, infer_prefix(k))  #  TODO: Generate from the template at dspy.Predict(Template2Signature)
    
            for k, v in outputs.items():
                output_field_key = k
                v.finalize(k, infer_prefix(k))
    
            return dsp.Template(parts.essential_instructions, **inputs, **outputs), output_field_key
    
        def forward(self, **kwargs):
            #  Extract the three privileged keyword arguments.
            signature = kwargs.pop("signature", self.signature)
            demos = kwargs.pop("demos", self.demos)
            config = dict(**self.config, **kwargs.pop("config", {}))
    
            prompt = signature(dsp.Example(demos=demos, **kwargs))
            output = self.langchain_llm.invoke(prompt, **config)
    
            try: content = output.content
            except AttributeError: content = output
    
            pred = Prediction.from_completions([{self.output_field_key: content}], signature=signature)
            dspy.settings.langchain_history.append((prompt, pred))
     
            if dsp.settings.trace is not None:
                trace = dsp.settings.trace
                trace.append((self, {**kwargs}, pred))
    
            return output
     
        def invoke(self, d, *args, **kwargs):
            return self.forward(**d)

如上代码所示，LangChainPredict初始化时传入LangChain的大模型和提示词类，初始化函数会根据传入提示词构建 dspy.Signature。在预测时调用forward函数，将传入的内容利用self.siguature转换为字符串类型，传入self.langchain_llm.invoke函数进行推理，实际是在执行LangChain的推理，再将预测内容包装到Prediction类中，存入dspy.settings中，最终返回预测结果。此外 LangChainPredict类也实现了invoke方法，使得LangChainPredict可以和其他LCEL组件串联在一起。

### DSPy集成到LangChain

从上文可知，利用LangChainPredict和LangChainModule可以利用DSPy优化LCEL中的prompt和LLM，那么已经优化好的提示词和大模型，如果在langchian的LCEL中使用呢？作者以为有两种方法将DSPy优化的结果集成到LangChain中

1）将DSPy的 LangChainPredict类之间串联到 LCEL 中

由于LangChainPredict类继承自 Runnable，所以它天然可以在LangChain中使用，因此只需要将优化好的LangChainPredict实例串联到所需的chain中即可。
    
    
    trained_langchain_predict = LangChainPredict(prompt, llm)
    #  此处省略优化步骤，得到优化后的 trained_langchain_predict
    ...
    chain = trained_langchain_predict | StrOutputParser()

2）提取优化好的提示词，手动加入到LangChain的Prompt中

由于目前业内尚未发现相关探索，在此给出作者的思路，经过上文2.1的源码分析，可以看出 LangChainPredict类包含signature属性，而在调用 LangChainPredict.forward时可以将signature转换为字符串，因此我们可以仿照这这种写法获取到提示词内容，具体步骤为先获取到 signature变量，然后将 demos传入 signature，会得到字符串类型的提示词。
    
    
    #  获取signature
    signature = trained_langchain_predict.signature
    #  获取demos
    demos = trained_langchain_predict.demos
    #  得到提示词字符串
    prompt_content = signature(demos)
    #  转换为提示词类
    prompt = PromptTemplate.from_template(prompt_content)
    #  创建大模型
    llm = OllamaLLM(model="llama3", stream=False)
    #  连接成 LCEL
    chain = prompt | llm

## 实践示例：使用DSPy优化LCEL

这一小节将介绍将LECL集成到DSPy中，利用DSPy优化LCEL的示例，首先将LCEL的主体功能利用 2.1 中提到的 LangChainPredict 与 LangChainModule类封装，然后利用 dspy.Optimizer 以及 dspy.Evaluate 进行优化和验证。

### 准备工作

请确保安装以下python包：

  * langchain
  * langchain_ollama/langchain_openai
  * dspy
  * langchain_core
  * langchain_community  




### 示例介绍

1）模块初始化：

首先导入相关模块，并初始化LangChain的 大模型、检索器以及提示词，这里使用了 Ollama提供的 llama3 大模型以及 维基百科检索器。
    
    
    import dspy
    
    from langchain.cache import SQLiteCache
    from langchain.globals import set_llm_cache
    from langchain_ollama import OllamaLLM
    from langchain_community.retrievers import WikipediaRetriever
    
    set_llm_cache(SQLiteCache(database_path="cache.db"))
    llm =  OllamaLLM(model="llama3", stream=False)
    retriever = WikipediaRetriever(load_max_docs=1)
    prompt = PromptTemplate.from_template(
        "Given {context}, answer the question `{question}` as a tweet."
    )
    def retrieve(inputs):
        return [doc.page_content[:1024] for doc in retriever.get_relevant_documents(query=inputs["question"])]
    
    def retrieve_eval(inputs):
        return [{"text": doc.page_content[:1024]} for doc in retriever.get_relevant_documents(query=inputs["question"])]
    
    question = "where was MS Dhoni born?"

2）创建LangChainModule实例

首先，将llm 和 prompt 封装到 LangChainPredict实例中，再将 检索器、LangChainPredict、输出解析器串联为 LECL，最后将 LECL 封装到 LangChainModule类中。
    
    
    from dspy.predict.langchain import LangChainModule, LangChainPredict
    
    zeroshot_chain = LangChainModule(
        RunnablePassthrough.assign(context=retrieve)
        | LangChainPredict(prompt, llm)
        | StrOutputParser()
    )

3）构建数据集

导入一个数据集，并且按照训练集、测试集、验证集分开
    
    
    from dspy.primitives.example import Example
    from datasets import load_dataset
    
    dataset = load_dataset('hotpot_qa', 'fullwiki')
    
    trainset = [
        Example(dataset['train'][i]).without("id", "type", "level", "supporting_facts", "context").with_inputs("question")
        for i in range(0, 50)
    ]
    valset = [
        Example(dataset['validation'][i]).without("id", "type", "level", "supporting_facts", "context").with_inputs("question")
        for i in range(0, 10)
    ]
    testset = [
        Example(dataset['validation'][i]).without("id", "type", "level", "supporting_facts", "context").with_inputs("question")
        for i in range(10, 20)
    ]
    
    """
    trainset[0]
    Example({'question': "Which magazine was started first Arthur's Magazine or First for Women?", 'answer': "Arthur's Magazine"}) (input_keys={'question'})
    """

4）创建Metrics

本示例构建了一种利用大模型判断的衡量指标（Metrics）,从 correct、engaging、faithful三个维度衡量。
    
    
    class Assess(dspy.Signature):
        """Assess the quality of a tweet along the specified dimension."""
    
        context = dspy.InputField(desc="ignore if N/A")
        assessed_text = dspy.InputField()
        assessment_question = dspy.InputField()
        assessment_answer = dspy.OutputField(desc="Yes or No")
    
    
    optimiser_model = dspy.OpenAI(model="gpt-4-turbo", max_tokens=1000, model_type="chat")
    METRIC = None
    
    
    def metric(gold, pred, trace=None):
        question, answer, tweet = gold.question, gold.answer, pred.output
        context = retrieve_eval({'question': question})
    
        engaging = "Does the assessed text make for a self-contained, engaging tweet?"
        faithful = "Is the assessed text grounded in the context? Say no if it includes significant facts not in the context."
        correct = f"The text above is should answer `{question}`. The gold answer is `{answer}`. Does the assessed text above contain the gold answer?"
    
        with dspy.context(lm=optimiser_model):
            faithful = dspy.Predict(Assess)(
                context=context, assessed_text=tweet, assessment_question=faithful
            )
            correct = dspy.Predict(Assess)(
                            context="N/A", assessed_text=tweet, assessment_question=correct
            )
            engaging = dspy.Predict(Assess)(
                context="N/A", assessed_text=tweet, assessment_question=engaging
            )
    
        correct, engaging, faithful = [
            m.assessment_answer.split()[0].lower() == "yes"
            for m in [correct, engaging, faithful]
        ]
        score = (correct + engaging + faithful) if correct and (len(tweet) <= 280) else 0
    
        if METRIC is not None:
            if METRIC == "correct":
                return correct
            if METRIC == "engaging":
                return engaging
            if METRIC == "faithful":
                return faithful
    
        if trace is not None:
            return score >= 3
        return score / 3.0

5）优化程序

与优化普通的dspy.Module一样，优化 LangChainModule实例。
    
    
    from dspy.teleprompt import BootstrapFewShotWithRandomSearch
    
    optimizer = BootstrapFewShotWithRandomSearch(
        metric=metric, max_bootstrapped_demos=3, num_candidate_programs=3
    )
    
    optimized_chain = optimizer.compile(zeroshot_chain, trainset=trainset, valset=valset)

6）验证结果
    
    
    from dspy.evaluate.evaluate import Evaluate
    
    evaluate = Evaluate(
        metric=metric, devset=testset, num_threads=8, display_progress=True, display_table=5
    )
    evaluate(optimized_chain)
    #  Average Metric: 3.0 / 10  (30.0): 100%|██████████| 10/10 [00:22<00:00,  2.22s/it]

## 相关链接

[1] [DSPy-LangChain (kaggle.com)](https://link.zhihu.com/?target=https%3A//www.kaggle.com/code/ritvik1909/dspy-langchain/notebook)

[2] [GitHub仓库：stanfordnlp/dspy: DSPy: The framework for programming—not prompting—foundation models (github.com)](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)

[3] DSPy官方文档：[About DSPy | DSPy (dspy-docs.vercel.app)](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/docs/intro)

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】10-DSPy Visualizer：可视化Prompt优化过程](https://zhuanlan.zhihu.com/p/714277212)  
> 下一篇：[【DSPy技术洞察】12-DSPy Optimizer进阶分析之BootstrapFewShot优化原理](https://zhuanlan.zhihu.com/p/716163290)
