# 【提示词生成与优化】06-CoD：少写快想，一种高准确率、低延迟、低成本的新提示技术

原文链接：https://zhuanlan.zhihu.com/p/1898757867579900226

---

​

目录

## 概述

随着[OpenAI](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=OpenAI&zhida_source=entity) o1、DeepSeek R1和[Gemini 2.0](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=Gemini+2.0&zhida_source=entity)等推理模型模型的突破，大语言模型通过思维链（[Chain-of-Thought](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=Chain-of-Thought&zhida_source=entity), CoT）提示等技术，在解决复杂任务中表现出卓越的性能。该技术强调逐步、详细的推理机制，将问题分解为逐步推导的过程，模拟人类结构化推理的方式。然而，该类方法在推理阶段需消耗大量计算资源，导致输出冗长、延迟更高。这与人类的思维和推理方式形成鲜明的对比，我们只思考时，通常不会采用冗长的语言进行推理，而是记下关键信息。

因此，Zoom的研究人员提出了一种开源的、新的思维推理技术——**[草稿链](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=%E8%8D%89%E7%A8%BF%E9%93%BE&zhida_source=entity) （Chain of Draft, CoD），一种更贴近人类推理的高效极简的提示策略。不同于冗长的中间步骤，CoD要求LLMs在每步生成简洁、信息密集的输出，并将每个推理步骤限制为数个字来描述**。该方法在不牺牲准确率的前提下，降低延迟和计算成本，使LLMs更适合对效率要求极高的现实场景应用。

图1 依据[Claude 3.5 Sonnet](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=Claude+3.5+Sonnet&zhida_source=entity)在不同任务中使用直接回答、CoT、CoD提示技术的准确率与Token消耗对比

## CoD提示词

人类通常不会详尽说明每个细节，而是仅记录关键的中间结果——即简要草稿——以辅助思考。CoD借鉴人类在处理复杂问题时采用的方法，不详尽说明每个细节，而是仅记录关键的中间结果，即简要草稿以辅助思考。该方法通过限制每个推理步骤中使用的词数，聚焦于推进所需的核心计算或转换，从而减少冗长性。

为说明标准提示、CoT提示和CoD提示之间的区别，使用以下简单的算术问题示例：
    
    
    Q: Jason had 20 lollipops. He gave Denny some lollipops. Now Jason has 12 lollipops. How many lollipops did Jason give to Denny?

标准提示法直接输出答案，通常缺乏推理过程的透明性，模型需要在没有中间结果帮助的情况下直接进行多步骤推理，易产生幻觉。

CoT提示生成详细推理过程，尽管准确且可解释，但包含冗余情景细节，导致Token消耗膨胀并增加响应延迟。

CoD提示则将推理过程浓缩为最小化的抽象表示，在这里，推理被简化为一个简洁的方程式，专注于得出答案所需的核心数学运算。通过抽象掉无关的上下文细节，CoD在保证透明性和正确性的前提下，显著减少了Token消耗。

## 实验验证

实验使用[GPT-4o](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=GPT-4o&zhida_source=entity)和Claude 3.5 Sonnet模型，分别基于算术推理、常识推理和符号推理场景进行实验评估。

**算术推理场景下** ，如下所示，标准提示在算术推理[GSM8K](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=GSM8K&zhida_source=entity)数据集的准确率分别为53.3%和64.6%。应用CoT后，两模型准确率均超95%，但单响应平均消耗约200 Token量。相较之下，CoD在保持91%准确率的同时，仅需约40 Token量，实现平均输出Token量降低80%，延迟分别减少76.2%和48.4%，在降低延迟的同时，其准确率没有任何重大损失。

GSM8K针对不同提示技术的评估结果

**常识推理场景下** ，基于BIG-bench的Date理解和sports理解任务中，如下所示，CoD在保持或超越CoT准确率的同时，消耗了更少的Token，显著降低延迟和成本。在sports理解任务中，Claude 3.5 Sonnet的CoT响应平均Token量达189.4，而CoD降至14.3，减少幅度达 92.4%。

Date理解BIG-bench任务的评估结果

Sports理解BIG-bench任务的评估结果

**符号推理场景下** ，在对硬币翻转（预测一系列翻转之后硬币的最终状态）的符号推理任务进行评估时，GPT-4o和Claude 3.5 Sonnet在标准提示下分别取得73.2%和85.2%的准确率，而采用CoT和CoD后均达100%准确率。CoD相较CoT的令牌降幅从GPT-4o的68%到Claude的86%不等。

研究人员创建的硬币翻转数据集中的问题示例

研究人员在定制数据集上对 250 个测试用例进行硬币翻转评估的结果

## 使用[DSPy](https://zhida.zhihu.com/search?content_id=256907842&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)实现CoD

DSPy通过引入签名，实现以简洁的方式定义LLM任务。签名定义了任务的输入输出。在本例中，将使用一个简单的签名 "question->answer"，即输入是一个问题，输出是一个答案。

### **使用CoD实现**
    
    
     import dspy
    
    question = '''Jason had 20 lollipops. He gave Denny some
    lollipops. Now Jason has 12 lollipops. How many
    lollipops did Jason give to Denny?'''
    # DSPy configurations
    task_description = "question->answer"
    rationale_type = dspy.OutputField(prefix="desc", desc="Think step by step, but only keep a minimum draft for each thinking step, with 5 words at most")
    model = dspy.ChainOfThought(task_description, rationale_type=rationale_type)
    lm = dspy.LM("openai/gpt-4o-mini")
    dspy.configure(lm=lm)
    
    print(model(question=question))
    # Output: Prediction(
    #    reasoning='20 lollipops initially. 12 left.',
    #    answer='8 lollipops'
    # )

运行改代码后，将产品如下输出结果：
    
    
    System message:
    
    Your input fields are:
    1. `question` (str)
    
    Your output fields are:
    1. `reasoning` (str): Think step by step, but only keep a minimum draft for each thinking step, with 5 words at most
    2. `answer` (str)
    
    All interactions will be structured in the following way, with the appropriate values filled in.
    
    [[ ## question ## ]]
    {question}
    
    [[ ## reasoning ## ]]
    {reasoning}
    
    [[ ## answer ## ]]
    {answer}
    
    [[ ## completed ## ]]
    
    In adhering to this structure, your objective is: 
            Given the fields `question`, produce the fields `answer`.
    
    
    User message:
    
    [[ ## question ## ]]
    Jason had 20 lollipops. He gave Denny some
    lollipops. Now Jason has 12 lollipops. How many
    lollipops did Jason give to Denny?
    
    Respond with the corresponding output fields, starting with the field `[[ ## reasoning ## ]]`, then `[[ ## answer ## ]]`, and then ending with the marker for `[[ ## completed ## ]]`.
    
    
    Response:
    
    [[ ## reasoning ## ]]
    20 lollipops initially. 12 left. 
    
    [[ ## answer ## ]]
    8 lollipops
    
    [[ ## completed ## ]]

### **使用DSPy实现CoT**
    
    
     import dspy
    
    question = '''Jason had 20 lollipops. He gave Denny some
    lollipops. Now Jason has 12 lollipops. How many
    lollipops did Jason give to Denny?'''
    
    # DSPy configurations
    task_description = "question->answer"
    model = dspy.ChainOfThought(task_description)
    lm = dspy.LM("openai/gpt-4o-mini")
    dspy.configure(lm=lm)
    print(model(question=question))
    
    # Output: Prediction(
    #    reasoning='Jason initially had 20 lollipops. After giving some to Denny, he has 12 lollipops left. To find out how many lollipops Jason gave to Denny, we can subtract the number of lollipops he has now from the number he had initially. This can be calculated as follows: \n\n20 (initial lollipops) - 12 (lollipops left) = 8 lollipops given to Denny.',
    #    answer='Jason gave Denny 8 lollipops.'
    # )

运行改代码后，将产品如下输出结果：
    
    
    System message:
    
    Your input fields are:
    1. `question` (str)
    
    Your output fields are:
    1. `reasoning` (str)
    2. `answer` (str)
    
    All interactions will be structured in the following way, with the appropriate values filled in.
    
    [[ ## question ## ]]
    {question}
    
    [[ ## reasoning ## ]]
    {reasoning}
    
    [[ ## answer ## ]]
    {answer}
    
    [[ ## completed ## ]]
    
    In adhering to this structure, your objective is: 
            Given the fields `question`, produce the fields `answer`.
    
    
    User message:
    
    [[ ## question ## ]]
    Jason had 20 lollipops. He gave Denny some
    lollipops. Now Jason has 12 lollipops. How many
    lollipops did Jason give to Denny?
    
    Respond with the corresponding output fields, starting with the field `[[ ## reasoning ## ]]`, then `[[ ## answer ## ]]`, and then ending with the marker for `[[ ## completed ## ]]`.
    
    
    Response:
    
    [[ ## reasoning ## ]]
    Jason initially had 20 lollipops. After giving some to Denny, he has 12 lollipops left. To find out how many lollipops Jason gave to Denny, we can subtract the number of lollipops he has now from the number he had initially. This can be calculated as follows: 
    
    20 (initial lollipops) - 12 (lollipops left) = 8 lollipops given to Denny.
    
    [[ ## answer ## ]]
    Jason gave Denny 8 lollipops.
    
    [[ ## completed ## ]]

### **小结**

由上两个示例，可以看到输出结果的Response中，CoD的推理和答案简洁，而CoT的显得冗余啰嗦。

上述两个代码唯一的变化在于，在CoD的提示中，为推理添加了描述，如下所示：
    
    
    Your output fields are:
    1. `reasoning` (str): Think step by step, but only keep a minimum draft for each thinking step, with 5 words at most
    2. `answer` (str)

在代码中，定义rationale_type，并对rationale_type添加输出约束，如下所示：
    
    
    task_description = "question->answer"
    rationale_type = dspy.OutputField(prefix="desc", desc="Think step by step, but only keep a minimum draft for each thinking step, with 5 words at most")
    model = dspy.ChainOfThought(task_description, rationale_type=rationale_type)

## 总结

延迟问题在LLM推理能力研究中常被忽视，但对实时应用至关重要。本文提出的CoD技术在保持或超越CoT准确率的同时，显著降低了推理延迟。通过精简推理步骤，CoD减少输入提示Token量和输出Token长度，直接降低成本，在成本敏感场景中具有显著优势。

CoD证明有效的LLM推理无需冗长输出，为保持推理深度提供新思路。未来可将CoD与自适应并行推理或多轮验证等方法结合，进一步优化性能。其精简推理原则可启发通过训练数据优化来提升模型推理效率，在保持可解释性和高效性的同时，弥合理论研究与实际系统需求间的差距。

## 相关链接

  1. [https://arxiv.org/abs/2502.18600](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2502.18600)
  2. [https://github.com/sileix/chain-of-draft](https://link.zhihu.com/?target=https%3A//github.com/sileix/chain-of-draft)
  3. [https://pub.towardsai.net/implementing-chain-of-draft-prompt-technique-with-dspy-ca231c58114f](https://link.zhihu.com/?target=https%3A//pub.towardsai.net/implementing-chain-of-draft-prompt-technique-with-dspy-ca231c58114f)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Prompt洞察】05-Prompt自优化框架洞察分析](https://zhuanlan.zhihu.com/p/29133814672)
