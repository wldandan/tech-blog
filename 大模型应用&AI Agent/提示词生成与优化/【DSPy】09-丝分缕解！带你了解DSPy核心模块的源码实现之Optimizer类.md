# 【DSPy】09-丝分缕解！带你了解DSPy核心模块的源码实现之Optimizer类

原文链接：https://zhuanlan.zhihu.com/p/709694815

---

​

目录

书接上文《[【[DSPy](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)技术洞察】08-丝分缕解！带你了解DSPy核心模块的源码实现之[Module](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=Module&zhida_source=entity)类](https://zhuanlan.zhihu.com/p/709324353)》，在该文章中，通过示例和源码解析的方式对[Optimizer](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=Optimizer&zhida_source=entity)类进行了详细的解读，本文将带你了解DSPy核心模块-Optimizer类的源码实现。

## 概述

DSPy是斯坦福 NLP 组推出的一个用于优化和生成Prompt的框架，DSPy将传统手工编写提示词的方式抽象为高代码编程方式，其核心思路为：

  1. 将整体流程与单步骤分开。每个步骤聚焦具体的工作，协同完成Prompt的优化
  2. 引入多样的优化器，这些优化器是由LM驱动的算法，可以根据定义的阈值来调整调用LM的提示及访问参数。



上述步骤主要由 [Signature](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=Signature&zhida_source=entity)、Module、Optimizer三个核心模块实现，本文将通过示例和源码解析的方式，解读DSPy运行原理。

## DSPy 实现原理之Optimizer

DSPy 优化器（此前被称为 [Teleprompters](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=Teleprompters&zhida_source=entity)）是用来微调 DSPy 程序参数的算法，如 prompts 和 LLM 权重，以达到某些指标（如准确性）的最大值。一个典型的 DSPy 优化器需要三个输入：

  1. 您的 DSPy 程序：可以是单一模块（例如 dspy.Predict）或复杂的多模块程序。
  2. 您选择的指标：一个评估程序输出并为其打分的函数（分数越高表示结果越好）。
  3. 一组训练输入：通常只需要 5 到 10 个示例。



一旦您定义了训练数据、模块和指标，优化器将优化 LLM 权重、prompt 指令和少数示例以提高程序效率。例如，[BootstrapFewShot](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=BootstrapFewShot&zhida_source=entity) 优化器生成与指定度量一致的答案，而像 COT（Chain of Thought）这样的模块生成结构化推理以得出准确的答案。DSPy 记录这些成功的实例和理由作为处理未来测试查询的少数示例演示。

本文以DSPy中常用的 BootstrapFewShot 优化器为例，解读优化器内部实现的原理。

### 优化器使用示例

首先来看一个示例来解释优化器的用法，需要准备DSPy 程序、衡量指标 Metrics 以及 训练集。
    
    
    import dspy
    from dspy.datasets.gsm8k import [GSM8K](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=GSM8K&zhida_source=entity), gsm8k_metric
    # Set up the LM
    turbo = dspy.OpenAI(model='gpt-3.5-turbo-instruct', max_tokens=250)
    dspy.settings.configure(lm=turbo)
    # Load math questions from the GSM8K dataset
    gsm8k = GSM8K()
    gsm8k_trainset, gsm8k_devset = gsm8k.train[:10], gsm8k.dev[:10]
    class CoT(dspy.Module):
        def __init__(self):
            super().__init__()
            self.prog = dspy.[ChainOfThought](https://zhida.zhihu.com/search?content_id=245841588&content_type=Article&match_order=1&q=ChainOfThought&zhida_source=entity)("question -> answer")
        def forward(self, question):
            return self.prog(question=question)

然后，初始化优化器，并调用compile函数进行优化
    
    
    from dspy.teleprompt import BootstrapFewShot
    # Set up the optimizer: we want to "bootstrap" (i.e., self-generate) 4-shot examples of our CoT program.
    config = dict(max_bootstrapped_demos=4, max_labeled_demos=4)
    # Optimize! Use the `gsm8k_metric` here. In general, the metric is going to tell the optimizer how well it's doing.
    teleprompter = BootstrapFewShot(metric=gsm8k_metric, **config)
    teleprompter.compile(CoT(), trainset=train_set)

### 优化器源码解析

那么在优化器是如何进行优化的，都优化了什么参数？  
本小节通过源代码解析方式解释 BootstrapFewShot 优化器利用bootstrap算法的优化流程。  
如下面代码所示， BootStrapFewShot 类初始化时，设置了训练参数，通过参数的调整，可以改变优化器最终的优化效果，例如，max_rounds 参数代码在优化过程中迭代次数，metric_threshold参数代表优化过程中，判定某次预测结果计算Metrics的阈值。

如示例所示，首先导入 BootstrapFewShot 类并实例化，内部代码逻辑如下：
    
    
    class BootstrapFewShot(Teleprompter):
        def __init__(
            self,
            metric=None,
            metric_threshold=None,
            teacher_settings: Optional[Dict]=None,
            max_bootstrapped_demos=4,
            max_labeled_demos=16,
            max_rounds=1,
            max_errors=5,
        ):
            self.metric = metric
            self.metric_threshold = metric_threshold
            self.teacher_settings = {} if teacher_settings is None else teacher_settings
            self.max_bootstrapped_demos = max_bootstrapped_demos
            self.max_labeled_demos = max_labeled_demos
            self.max_rounds = max_rounds
            self.max_errors = max_errors
            self.error_count = 0
            self.error_lock = threading.Lock()

在初始化之后，调用compile函数即可开始优化，compile函数定义如下：
    
    
    def compile(self, student, *, teacher=None, trainset, valset=None):
        self.trainset = trainset
        self.valset = valset
        self._prepare_student_and_teacher(student, teacher)
        self._prepare_predictor_mappings()
        self._bootstrap()
        self.student = self._train()
        self.student._compiled = True
        # set assert_failures and suggest_failures as attributes of student w/ value 0
        self.student._assert_failures = 0
        self.student._suggest_failures = 0
    return self.student

上述代码按照 1）获取数据集，2）准备教师、学生模块（属于bootstrap算法概念，teacher和student变量均为CoT类）3）准备预测器（预测器就是Predict类对象，此处为ChainOfThought对象）4）开始运行 bootstrap算法，5）将训练结果传递给student对象并返回，共计5个步骤运行。其中，核心代码在 self._bootstrap函数中。
    
    
    def _bootstrap(self, *, max_bootstraps=None):
           ...
           for round_idx in range(self.max_rounds): # 循环轮数
               for example_idx, example in enumerate(tqdm.tqdm(self.trainset)):   # 循环数据集
                   if len(bootstrapped) >= max_bootstraps:
                       break
                   if example_idx not in bootstrapped:
                       success = self._bootstrap_one_example(example, round_idx)
                       if success:
                           bootstrapped[example_idx] = True
             ...

在 _bootstrap 函数中，执行了 bootstrap算法核心逻辑，通过迭代的方式，循环训练集 trainset，将trainset中的示例数据不断的传入 bootstrap_one_example 函数中。
    
    
    def _bootstrap_one_example(self, example, round_idx=0):
        name2traces = self.name2traces
        teacher = self.teacher #.deepcopy()
        predictor_cache = {}
        try:
            with dsp.settings.context(trace=[], **self.teacher_settings):
                lm = dsp.settings.lm
                lm = lm.copy(temperature=0.7 + 0.001 * round_idx) if round_idx > 0 else lm
                new_settings = dict(lm=lm) if round_idx > 0 else {}
                with dsp.settings.context(**new_settings):
                    ...
                    prediction = teacher(**example.inputs())
                    trace = dsp.settings.trace
                if self.metric:
                    metric_val = self.metric(example, prediction, trace)
                    if self.metric_threshold:
                        success = metric_val >= self.metric_threshold
                    else:
                        success = metric_val
                else:
                    success = True
                # print(success, example, prediction)
        if success:
            for step in trace:
                predictor, inputs, outputs = step
                if 'dspy_uuid' in example:
                    demo = Example(augmented=True, dspy_uuid=example.dspy_uuid, **inputs, **outputs)
                else:
                    demo = Example(augmented=True, **inputs, **outputs)
                ...
                name2traces[predictor_name].append(demo)
    return success

在 _bootstrap_one_example 函数中，主要执行逻辑为

  1. 根据round_idx 调整大模型的 temperature值，修改 new_settings 内容，并使用 new_settings的配置进行预测，得到预测结果 prediction。
  2. 使用 输入的 example 和 预测的 prediction，利用 Metrics函数计算结果，如果结果大于 metric_threshold，则代码该示例预测成功。
  3. 如果示例预测成功，则将示例加入 name2traces变量中，name2traces变量为 self.name2traces，为类属性。



经过上述几个步骤，compile函数利用 trainset和bootstrap算法以及Metrics，不断的调整 temperture和 name2traces中的 示例内容，最后这些内容将 调用 self._train 函数传递给 学生模块。
    
    
    def _train(self):
           rng = random.Random(0)
           raw_demos = self.validation
           for name, predictor in self.student.named_predictors():
               augmented_demos = self.name2traces[name][:self.max_bootstrapped_demos] #获取优化后的示例
               sample_size = min(self.max_labeled_demos - len(augmented_demos), len(raw_demos))
               sample_size = max(0, sample_size)
               raw_demos = rng.sample(raw_demos, sample_size)
               if dspy.settings.release >= 20230928:
                   predictor.demos = raw_demos + augmented_demos
               else:
                   predictor.demos = augmented_demos + raw_demos

从代码中可以看出，_train函数最终修改了 student的 predictor.demos属性（raw_demos + augmented_demos），其中这里的predictior只有一个，就是CoT定义的ChainOfThought类。而student（CoT类实例）最终作为compile函数的返回。因此我们可以理解为整个compile流程是将传入的 CoT 对象 student，利用 bootstrap算法、Metrics、trainset得到了一些示例，并用这些示例更新了 student 中 predictor的demos等属性，以及全局的大模型配置（temperature）等。

在上一节中，我们提到过 Module类在进行推理时，会首先加载示例到提示词中，因此这些优化得到的示例就成功的加入到优化后Module子类中，从而影响最终的推理结果。

## 总结

本文通过示例和源码解析的方式，解释DSPy核心模块的工作原理。其中Signature为通过格式化的方式定义了提示词模板中需要填写的内容，Module中将Signature的内容配合提示词模板组成完整的提示词，并输出预测结果。Optimizer可以利用算法和Metrics衡量标准调整Module中的示例和参数，进而调整提示词和模型参数，使得Module可以有更好的预测结果。

## 相关链接

  1. [dspy官网](https://link.zhihu.com/?target=https%3A//wiki.huawei.com/domains/30974/wiki/49918/WIKI202406133768336) [https://dspy-docs.vercel.app](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/)
  2. Dspy github仓库地址 [https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】08-丝分缕解！带你了解DSPy核心模块的源码实现之Module类](https://zhuanlan.zhihu.com/p/709324353)  
> 下一篇：[【DSPy技术洞察】10-DSPy Visualizer：可视化Prompt优化过程](https://zhuanlan.zhihu.com/p/714277212)
