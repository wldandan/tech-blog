# 【DSPy】01-Prompt工程或成为过去时，DSPy将会是拯救Prompt思维的利器

原文链接：https://zhuanlan.zhihu.com/p/705107414

---

​

目录

>  _随着[大语言模型](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B&zhida_source=entity)（LLM）的能力不断提升，人工智能领域取得了一个巨大的进步。这些强大的语言模型拥有惊人的语言理解和生成能力。它们能够自动学习和分析大规模的文本数据，并生成准确、流畅的文本回复。这也为AGI（通用人工智能）带来了曙光。_

## 概述

大语言模型在各种[自然语言处理](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E5%A4%84%E7%90%86&zhida_source=entity)（Natural Language Processing，NLP）任务上取得惊人的效果。但其并不是万能的，当前还存在不少局限性和挑战，如：

  * 需要庞大的计算资源和时间来进行预训练和微调，对于普通用户和开发者来说这是难以承受的。
  * 缺乏最新的知识和特定领域的知识，导致大模型可能无法解答某些特定领域或者场景下的问题。
  * [幻觉问题](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=%E5%B9%BB%E8%A7%89%E9%97%AE%E9%A2%98&zhida_source=entity)，即一本正经的胡说八道，看似流畅自然的表述，实则不符合事实或者是错误的。



[Prompt](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=Prompt&zhida_source=entity)是克服这些大语言模型挑战的一种有效手段，它就像是一把引导大语言模型的魔杖，帮助大语言模型更好地理解输入的意图和任务，以正确得生成特定类型、主题或格式的输出。Prompt的好坏直接影响到大语言模型输出的输出效果和用户体验，因此Prompt对于构建基于大语言模型的应用/系统也是至关重要。

那么，什么是Prompt呢？**Prompt（提示）是指输入给大语言模型的文本片段（可以是一句简单的问题，一段较长的文本，或者一组指令，这取决于用户的具体需求），用于指导模型生成符合特定要求的文本。** 大语言模型的工作原理是根据输入的文本，来预测下一个词出现的概率，逐字生成出下文。它并不会像人类那样完全理解输入的Prompt，而是根据统计规律和语言模型来生成输出，例如，如果Prompt是“今天天气很”，模型可能会生成“晴朗”，“阴沉”等与天气相关的词语作为下一句话的开头。因此，输入的Prompt会直接影响输出结果的质量。即使是很小的语言差异，生成的内容也可能完全不同。

## 什么是[DSPy](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)

构建基于大语言模型应用时，其流水线通常使用Prompt实现，也是由于Prompt的原因，其构建过程往往存在着脆弱性。例如，在构建基于LLM的应用时，往往需要将问题分解为多个步骤，同时需要对每个步骤的Prompt进行微调，调整各个步骤以协同工作，在生成合成示例对每个步骤进行调整，并对大语言模型进行微调，从而生成合适的输出。但是，这个过程比较繁琐且易受到流程、大语言模型或者数据变化的影响。

面对上述变化时，需要反复试错并修改Prompt，传统的人工手写和调试Prompt的方法显然无法满足大模型应用开发的诉求。由LangChain等相继推出Prompt template能力，虽然可以高效支撑构建大模型应用，但是其对流程线中组件的变更比较敏感，且无法扩展。因此，如何解决构建大模型应用由Prompt带来的脆弱性问题呢？DSPy应运而生。

**DSPy** （**D** eclarative **S** elf-improving Language **P** rograms（in Python）），**即声明式自改进语言程序，其是一个对语言模型Prompt和权重进行算法优化的框架** ，由[斯坦福大学NLP团队](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=%E6%96%AF%E5%9D%A6%E7%A6%8F%E5%A4%A7%E5%AD%A6NLP%E5%9B%A2%E9%98%9F&zhida_source=entity)开发。其**强调编程而非Prompt** ，并将构建**基于语言模型的流水线从操作prompt转移到更贴近编程** 。

DSPy目标是解决构建基于LM（语言模型）应用的脆弱性问题。每当你改变一个组件时，它允许你**重新编译整个流水线，以根据你的特定任务进行优化** ，从而免去了开发人员持续手动调整提示的麻烦。

此外，DSPy 还将程序的信息流与每一步的参数（提示和语言模型权重）分离开来，为构建基于语言模型的应用程序提供了更系统的方法。然后，DSPy 将根据您的程序，自动优化如何针对您的特定任务提示（或微调）语言模型。

DSPy工作流，来源于领英DSPy: The Future of Programming Language Models

DSPy主要包含**签名（Signatures）** 、**模块（Modules）** 和**优化器（Optimizers，** 原叫Teleprompters**）** 三个组件。其创新之处在于将签名、模块和优化器结合起来使用。签名为语言模型提供指导，而优化器则使用签名和一个度量或评估系统（可能是语言模型作为评判标准）来进行实验，以确定更理想的提示文本和最佳的少量示例集。因此，使用DSPy，开发者只需关注任务本身，而不必纠结于Prompt工程的具体细节，可以极大地提高了开发效率。

使用DSPy构建基于LLM的应用的工作流程如下所示：

使用DSPy构建基于LLM的应用的工作流程

  1. 收集数据集：收集程序的输入和输出示例（例如问题及其答案，或主题及其摘要），这些示例将用于优化流水线。
  2. 编写DSPy程序：用签名和模块以及组件之间的信息流定义程序的逻辑，以解决任务。
  3. 定义验证逻辑：使用验证度量和优化器定义优化程序的逻辑，并根据输出结果和指标得分对流水线进行评估。
  4. 编译DSPy程序：DSPy编译器考虑训练数据、编写程序、优化器和验证度量，以优化程序（如提示或微调）。
  5. 迭代：通过改进数据、编写程序或验证来重复该过程，直到对流水线的性能感到满意为止。



## 关键技术

本节将会对DSPy三个基本组件**签名（Signatures）** 、**模块（Modules）** 和**优化器（Optimizers），** 以及关键技术**度量（Metric）、断言（Assertions）** 进行介绍。

### 签名（Signatures）

**签名是 DSPy 模块输入/输出行为的声明性规范。** 其目的是提供对任务或子任务以及输入和输出类型的最基本描述。对于简单的情况，签名可以是简短的字符串，也可以包含多个输入/输出字段。其参数名定义了输入/输出的语义角色。

  * **单输入/输出字段**


  1. 回答问题： `"question -> answer"`
  2. 情感分类： `"sentence -> sentiment"`
  3. 总结：` "document -> summary"`


  * **多输入/输出字段**


  1. 检索增强型问题解答： `"context, question -> answer"`
  2. 带推理功能的多选题回答： `"question, choices -> reasoning, selection"`



示例：情感分类
    
    
    sentence = "it's a charming and often affecting journey." # example from the SST-2 dataset.
    
    classify = dspy.Predict('sentence -> sentiment')
    classify(sentence=sentence).sentiment
    

Output: 输出：
    
    
    'Positive'

对于更高级的任务，**签名可以定义为一个类** 。这样做的目的是提供有关**任务本身的额外提示** （见下文签名中的注释示例），或提供有关**输入或输出字段的额外提示** （作为描述关键字参数提供），如下图所示。这些变量名和注释有助于 LLM/LM 驱动的系统更好地理解任务。
    
    
    class GenerateSearchQuery(dspy.Signature):
        """Write a simple search query that will help answer a complex question."""
        context = dspy.InputField(desc="may contain relevant facts")
        question = dspy.InputField()
        query = dspy.OutputField()
    self.generate_answer = dspy.ChainOfThought(GenerateSearchQuery)

这些签名通常是连锁在一起的。例如，一个签名可能指定从检索模型中查询数据的意图，第二个签名可能指定使用这些检索到的数据/上下文和问题为用户生成答案的意图。

### 模块（Modules）

DSPy模块是利用语言模型（LM）构建程序的基本组件。

  * 每个内置模块都抽象了一种提示技术（如[Chain of Thought](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=Chain+of+Thought&zhida_source=entity)或ReAct）。每个模块都关联一个自然语言签名，并内部实现了相应的Prompt流程。
  * 具有可调整的参数（即构成提示和语言模型权重的小部件），可影响提示内容和语言模型的行为，从而实现与输入的动态交互以产生输出。
  * DSPy模块还可以组成任意的流水线，从而组合成更复杂的程序。



DSPy 提供七个内置模块以满足各种用途，包括`dspy.ReAct`、`dspy.ChainofThought`、`dspy.ChainOfThoughtWithHint` 、`dspy.Predict`、`dspy.ProgramOfThought`、`dspy.MultiChainComparison`和`dspy.Retrieve`。

DSPy模块（dspy.Predict除外）在模块内利用并扩展签名提供的信息。例如，dspy.ChainOfThought模块添加了一个rationale字段，其中包括语言模型在生成输出之前的推理。

### 优化器（Optimizers）

**优化器是一种算法，可以调整DSPy程序的参数** （即提示和/或语言模型权重），**以最大限度地提高您指定的指标** （如准确性）。典型的 DSPy 优化器需要三个输入：

  * DSPy程序：可能是一个单一模块（如dspy.Predict），也可能是一个复杂的多模块程序。
  * 度量：一个函数，用于评估程序的输出，并给程序打分（分数越高越好）。
  * 训练输入：少量的示例，示例可以是不完整的（只有程序的输入，没有任何标签）。



当前DSPy实现了如下优化器：

  * **自动少样本学习** ：`LabeledFewShot`、`BootstrapFewShot`、`BootstrapFewShotWithRandomSearch`、`BootstrapFewShotWithOptuna`、`KNNFewShot`
  * **[自动指令优化](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E6%8C%87%E4%BB%A4%E4%BC%98%E5%8C%96&zhida_source=entity)** ：`COPRO`和`MIPRO`
  * **[自动微调](https://zhida.zhihu.com/search?content_id=244821537&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%BE%AE%E8%B0%83&zhida_source=entity)** ：`BootstrapFinetune`
  * **程序转换** ：`Ensemble`



### 度量（Metric）

度量是一个函数，它将从你的数据中提取示例，并获取你的系统输出，然后返回一个量化输出好坏的分数，分数越高越好。度量函数对DSPy用户体验的影响很大，不仅决定了最终的质量评估，还会影响优化结果。度量函数涉及三个参数：数据集的示例 example、程序的输出（pred）和trace（可选参数）。这个函数的本质是返回一个float、int或bool分数。

下面的度量值如果为 trace is None （即用于评估或优化），则返回 float ，否则返回 bool （即用于引导演示）。
    
    
    def validate_context_and_answer(example, pred, trace=None):
        # check the gold label and the predicted answer are the same
        answer_match = example.answer.lower() == pred.answer.lower()
    
        # check the predicted answer comes from one of the retrieved contexts
        context_match = any((pred.answer.lower() in c) for c in pred.context)
    
        if trace is None: # if we're doing evaluation or optimization
            return (answer_match + context_match) / 2.0
        else: # if we're doing bootstrapping, i.e. self-generating good demonstrations of each step
            return answer_match and context_match

### 断言（Assertions）

在DSPy中，**断言** 被定义为程序元素，它**定义了在语言模型流水线执行过程中必须遵守的某些条件或规则。** 这些约束可确保流水线的行为符合开发人员指定的不变量或准则，从而提高流水线输出的可靠性、可预测性和正确性。

LM（语言模型）断言分为两个明确定义的编程结构，即**断言（Assertions）** 和**建议（Suggestions）** ，用构造体Assert和Suggest表示。它们是强制约束和引导LM流水线执行流程的结构。

相比传统的断言（一个检查条件，如果条件为假，则引发异常），DSPy提供了一种复杂的重试机制，同时支持多种新优化。**当Assert失败时，流水线会转换到特殊的重试状态，** 使其能够重新尝试失败的LM调用，同时了解之前的尝试和引发的错误信息。**在达到最大次数的自我改进尝试后，断言仍然失败，流水线就会过渡到错误状态，并引发AssertionError，从而终止流水线** 。

与Assert语句相比，**Suggest** 语句是较软的约束，**推荐但不强制执行条件** ，旨在引导LM 流水线朝向所期望的特定领域的结果。当Suggest条件不满足时，类似于Assert，流水线会进入特殊的重试状态，允许重新尝试失败的LM调用和自我改进。然而，如果建议在**达到最大次数的自我改进尝试后仍然失败，流水线只会记录一个警告SuggestionError消息并继续执行。** 这使得流水线能够根据建议调整其行为，同时在面对次优状态（或次优或启发式计算检查）时保持灵活和弹性。

包含断言的SimplifiedBaleen程序示例：
    
    
    class SimplifiedBaleenAssertions(dspy.Module):
        def __init__(self, passages_per_hop=2, max_hops=2):
            super().__init__()
            self.generate_query = [dspy.ChainOfThought(GenerateSearchQuery) for _ in range(max_hops)]
            self.retrieve = dspy.Retrieve(k=passages_per_hop)
            self.generate_answer = dspy.ChainOfThought(GenerateAnswer)
            self.max_hops = max_hops
    
        def forward(self, question):
            context = []
            prev_queries = [question]
    
            for hop in range(self.max_hops):
                query = self.generate_query[hop](context=context, question=question).query
    
                dspy.Suggest(
                    len(query) <= 100,
     "Query should be short and less than 100 characters",
                )
    
                dspy.Suggest(
                    validate_query_distinction_local(prev_queries, query),
     "Query should be distinct from: "
                    + "; ".join(f"{i+1}) {q}" for i, q in enumerate(prev_queries)),
                )
    
                prev_queries.append(query)
                passages = self.retrieve(query).passages
                context = deduplicate(context + passages)
     
        if all_queries_distinct(prev_queries):
            self.passed_suggestions += 1
    
        pred = self.generate_answer(context=context, question=question)
        pred = dspy.Prediction(context=context, answer=pred.answer)
        return pred

## 实践

### 安装DSPy

使用pip install安装dspy-ai Python软件包。
    
    
    pip install dspy-ai

安装 main 的最新版本：
    
    
    pip install git+https://github.com/stanfordnlp/dspy.git

### CoT思维模式最小集

首先介绍利用DSPy包，跑通最简单的一次CoT思维模式问答，该示例涉及到 dspy.Signature 和 dspy.ChainOfThought 类，其中 dsyp.Signature 是定义模块输入输出的类，dspy.ChainOfThought 为DSPy内置的思维模式类，以CoT模式与大模型交互，具体代码如下
    
    
    import dspy
    
    #  定义并设置大模型
    model_name = 'llama3'
    lm = dspy.OllamaLocal(model=model_name)
    dspy.settings.configure(lm=lm)
    
    #  定义输入输出参数 类定义方式
    
    class QA(dspy.Signature):
        question = dspy.InputField()
        answer = dspy.OutputField()
    
    question = "what is the color of the sea?"
    summarize = dspy.ChainOfThought(QA)
    response = summarize(question=question)
    
    print(f"问题：{question} \n答案：{response.answer}")

上述代码首先定义了大模型使用 llama3 ，然后，定义了 dspy.Signature 类，输入字段为 question， 输出字段为 answer，最后实例化 dspy.ChainOfThought 类，并输入问题调用大模型进行回答，执行结果为
    
    
    # #  类定义方式 定义输入输出参数 - start # # 
    
    问题：what is the color of the sea? 
    答案：The color of the sea is typically perceived as blue.
    
    # #  类定义方式 定义输入输出参数 - end # # 

此外，depy.Signature 类还支持以inline的方式定义，通过简单的字符串描述输入输出字段，如下所示，在第2行直接将原来的 QA 替换为了字符串 "question->answer"，这种写法可以自动的被转换成 dspy.Signature 类：
    
    
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

### 为提示词增加示例

众所周知，通过调整提示词的内容，可以改变大模型返回的结果，比较常见的一种调整提示词的办法是为提示词增加示例，那么，如何给 CoT 模式增加示例呢？此处将引入 dspy.Example 类，是DSPy的数据类，用于构建示例，代码如下。
    
    
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

### 构建示例数据集

随着我们示例的增加，我们希望通过构建一个数据集来保存示例，并将整个数据集传入到CoT的推理过程中，这里，我们引入了DSPy内置的数据集GSM8K，这是一个多步骤数学推理数据集，示例如下：
    
    
    import dspy
    from dspy.datasets.gsm8k import GSM8K, gsm8k_metric
    gsm8k_trainset = gsm8k.train[:20]
    
    model_name = 'llama3'
    lm = dspy.OllamaLocal(model=model_name, timeout_s=1000)
    dspy.settings.configure(lm=lm)
    
    question = ("Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount."
                " Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served"
                "at least three years have to pay for shoes?")
    #  示例内容
    summarize = dspy.ChainOfThought('question -> answer')
    response = summarize(question=question, demos=gsm8k_trainset)
    
    print(f"问题：{question} \n答案：{response.answer}")

此处将数据集传入到demos变量中，并询问了一个和数学计算有关的问题，结果如下：
    
    
    问题：Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has servedat least three years have to pay for shoes? 
    答案：296

实际上这个答案是错误的，正确的答案是51，因此提示词可以优化大模型的输出，但是也很难保证回答问题的准确性。提示词如下：
    
    
    ---
    
    Question: The result from the 40-item Statistics exam Marion and Ella took already came out. Ella got 4 incorrect answers while Marion got 6 more than half the score of Ella. What is Marion's score?
    Answer: 24
    
    ---
    
    Question: Stephen made 10 round trips up and down a 40,000 foot tall mountain. If he reached 3/4 of the mountain's height on each of his trips, calculate the total distance he covered.
    Answer: 600000
    
    ---
    
    Question: Bridget counted 14 shooting stars in the night sky. Reginald counted two fewer shooting stars than did Bridget, but Sam counted four more shooting stars than did Reginald. How many more shooting stars did Sam count in the night sky than was the average number of shooting stars observed for the three of them?
    Answer: 2
    
    ---
    
    Question: Sarah buys 20 pencils on Monday. Then she buys 18 more pencils on Tuesday. On Wednesday she buys triple the number of pencils she did on Tuesday. How many pencils does she have?
    Answer: 92
    
    ---
    
    Question: Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served at least three years have to pay for shoes?
    Answer: 51
    
    ---
    
    Question: The average score on last week's Spanish test was 90. Marco scored 10% less than the average test score and Margaret received 5 more points than Marco. What score did Margaret receive on her test?
    Answer: 86
    
    ---
    
    Question: A third of the contestants at a singing competition are female, and the rest are male. If there are 18 contestants in total, how many of them are male?
    Answer: 12
    
    ---
    
    Question: Nancy bought a pie sliced it into 8 pieces. She gave 1/2 to Joe and Darcy, and she gave 1/4 to Carl. How many slices were left?
    Answer: 2
    
    ---
    
    Question: Megan pays $16 for a shirt that costs $22 before sales. What is the amount of the discount?
    Answer: 6
    
    ---
    Question: Amaya scored 20 marks fewer in Maths than she scored in Arts. She also got 10 marks more in Social Studies than she got in Music. If she scored 70 in Music and scored 1/10 less in Maths, what's the total number of marks she scored in all the subjects?
    Answer: 296
    
    ---
    
    Question: Betty and Dora started making some cupcakes at the same time. Betty makes 10 cupcakes every hour and Dora makes 8 every hour. If Betty took a two-hour break, what is the difference between the number of cupcakes they made after 5 hours?
    Answer: 10
    
    ---
    
    Question: Alice and Bob are each given $2000 to invest. Alice puts all of her money in the stock market and doubles her money. Bob invests in real estate and makes five times more money than he invested. How much more money does Bob have now than Alice?
    Answer: 8000
    
    ---
    
    Question: A tank contains 6000 liters of water, 2000 liters evaporated, and then 3500 liters were drained by Bob. How many liters are in the tank if it now rains for 30 minutes and every 10 minutes 350 liters of rain are added to the tank?
    Answer: 1550
    
    ---
    
    Question: John takes 3 days off of streaming per week. On the days he does stream, he streams for 4 hours at a time and makes $10 an hour. How much does he make a week?
    Answer: 160
    
    ---
    
    Question: Billy is breeding mice for an experiment. He starts with 8 mice, who each have 6 pups. When the pups grow up, all the mice have another 6 pups. Then each adult mouse eats 2 of their pups due to the stress of overcrowding. How many mice are left?
    Answer: 280
    
    ---
    
    Question: John needs to replace his shoes so he decides to buy a $150 pair of Nikes and a $120 pair of work boots. Tax is 10%. How much did he pay for everything?
    Answer: 297
    
    ---
    
    Question: Daryl is loading crates at a warehouse and wants to make sure that they are not overloaded. Each crate can weigh up to 20kg and he has 15 crates he can fill. He has 4 bags of nails to load, each of which weighs 5kg; he has 12 bags of hammers, each of which weighs 5 kg; he also has 10 bags of wooden planks, each of which weighs 30kg and can be sub-divided. He realizes that he has too much to load and will have to leave some items out of the crates to meet the weight limit. In kg, how much is Daryl going to have to leave out of the crates?
    Answer: 80
    
    ---
    
    Question: Tom's rabbit can run at 25 miles per hour. His cat can run 20 miles per hour. The cat gets a 15-minute head start. In hours, how long will it take for the rabbit to catch up?
    Answer: 1
    
    ---
    
    Question: In 2004, there were some kids at a cookout. In 2005, half the number of kids came to the cookout as compared to 2004. In 2006, 2/3 as many kids came to the cookout as in 2005. If there were 20 kids at the cookout in 2006, how many kids came to the cookout in 2004?
    Answer: 60
    
    ---
    
    Question: James splits 4 packs of stickers that have 30 stickers each. Each sticker cost $.10. If his friend pays for half how much did James pay?
    Answer: 6
    
    ---
    
    Question: Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has servedat least three years have to pay for shoes?
    Reasoning: Let's think step by step in order to I'll generate the answers based on the given questions. Here they are:
    
    ---
    
    Question Amaya scored 20 marks fewer in Maths than she scored in Arts. She also got 10 marks more in Social Studies than she got in Music. If she scored 70 in Music and scored 1/10 less in Maths, what's the total number of marks she scored in all the subjects?
    
    Answer: 296

### 自动优化模板

刚刚在增加示例的过程中，我们发现了问题，（1）增加示例也并不能保证大模型预测准确（2）当大模型调整时原有提示词可能是不适用的，针对于这两个问题，DSPy提供了对提示词和大模型参数进行自动化优化的功能，可以进一步提高模型的准确性以及在不同大模型上的稳定性。用户只需要提供目标领域的训练数据集 Dataset，以及衡量大模型返回结果准确性的衡量标准 Metrics，以及优化器 Optimizer，就可以自动的得到一个最适合目标场景数据集的提示词和模型参数。代码如下：
    
    
    import dspy
    from dspy.datasets.gsm8k import gsm8k_metric
    from dspy.teleprompt import BootstrapFewShot
    
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
    
    config = dict(max_bootstrapped_demos=4, max_labeled_demos=4)
    
    #  Optimize! Use the `gsm8k_metric` here. In general, the metric is going to tell the optimizer how well it's doing.
    teleprompter = BootstrapFewShot(metric=gsm8k_metric, **config)
    #  可以调整 train_set 长度
    optimized_cot = teleprompter.compile(CoT(), trainset=gsm8k_trainset)
    optimized_cot.save("./test.json")
    question = "Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served at least three years have to pay for shoes?"
    response = optimized_cot(question=question)
    print(f"问题：{question} \n答案：{response.answer}")

上述代码中首先导入了内置的优化器 BootstrapFewShot、度量函数 gsm8k_metric、 以及gsm8k数据集的前20条数据即 gsm8k_trainset，利用 BootstrapFewShot.compile 函数进行优化，其中内部保留了对回答问题有利的 示例 以及 大模型参数，最终提问，结果如下
    
    
    问题：Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served at least three years have to pay for shoes? 
    答案：51

最终的提示词如下，可以看出最终在多个训练集中保留了4个示例：
    
    
    ---
    
    Question: Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served at least three years have to pay for shoes?
    Answer: 51
    
    ---
    
    Question: James splits 4 packs of stickers that have 30 stickers each. Each sticker cost $.10. If his friend pays for half how much did James pay?
    Answer: 6
    
    ---
    
    Question: In 2004, there were some kids at a cookout. In 2005, half the number of kids came to the cookout as compared to 2004. In 2006, 2/3 as many kids came to the cookout as in 2005. If there were 20 kids at the cookout in 2006, how many kids came to the cookout in 2004?
    Answer: 60
    
    ---
    
    Question: Sarah buys 20 pencils on Monday. Then she buys 18 more pencils on Tuesday. On Wednesday she buys triple the number of pencils she did on Tuesday. How many pencils does she have?
    Answer: 92
    
    ---
    
    Question: Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served at least three years have to pay for shoes?
    Reasoning: Let's think step by step in order to Question: Rookie police officers have to buy duty shoes at the full price of $85, but officers who have served at least a year get a 20% discount. Officers who have served at least three years get an additional 25% off the discounted price. How much does an officer who has served at least three years have to pay for shoes?
    Answer: 51

## 总结

DSPy解决了开发LLM应用时由Prompt脆弱性带来的问题。其强调Prompt的整体系统设计，有助于确保系统的文档性和扩展性。通过模块化的设计，DSPy将复杂的任务分解为多个模块，每个模块都有明确的职责和接口，使系统更易于维护和扩展。同时，通过不断的自动优化，DSPy能够不断的调整和优化提示和模型，使得系统不断的朝着更好的方向发展。通过DSPy，开发者只需关注任务本身，而不必纠结于Prompt工程的具体细节，可以极大地提高了开发效率。

## 参考资料

  1. DSPy to Simplify Prompt Optimization：[https://medium.com/@vineethveetil/dspy-to-simplify-prompt-optimization-f18365535475](https://link.zhihu.com/?target=https%3A//medium.com/%40vineethveetil/dspy-to-simplify-prompt-optimization-f18365535475)
  2. Intro to DSPy: Goodbye Prompting, Hello Programming：[https://towardsdatascience.com/intro-to-dspy-goodbye-prompting-hello-programming-4ca1c6ce3eb9](https://link.zhihu.com/?target=https%3A//towardsdatascience.com/intro-to-dspy-goodbye-prompting-hello-programming-4ca1c6ce3eb9)
  3. DSPy官网：[https://dspy-docs.vercel.app/docs/quick-start/installation](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/docs/quick-start/installation)
  4. GitHub：[https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)
  5. Omar Khattab, Arnav Singhvi, Paridhi Maheshwari, Zhiyuan Zhang, Keshav Santhanam, Sri Vardhamanan, Saiful Haq, Ashutosh Sharma, Thomas T. Joshi, Hanna Moazam, Heather Miller, Matei Zaharia, Christopher Potts. DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines arXiv:2310.03714
  6. Arnav Singhvi, Manish Shetty, Shangyin Tan, Christopher Potts, Koushik Sen, Matei Zaharia, Omar Khattab. DSPy Assertions: Computational Constraints for Self-Refining Language Model Pipelines arXiv:2312.13382



## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【DSPy技术洞察】02-DSPy：将声明式语言模型调用编译为自我改进流水线](https://zhuanlan.zhihu.com/p/705291734)
