# 【DSPy】12-DSPy Optimizer进阶分析之BootstrapFewShot优化原理

原文链接：https://zhuanlan.zhihu.com/p/716163290

---

​

目录

[DSPy](https://zhida.zhihu.com/search?content_id=247277477&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)是开源社区比较有影响力的大模型提示词、参数调优仓库，使得用户以一种编程的方式，调整和优化大模型模板、参数。其中提示词优化的核心模块为优化器（[dspy.Optimizer](https://zhida.zhihu.com/search?content_id=247277477&content_type=Article&match_order=1&q=dspy.Optimizer&zhida_source=entity)）模块，典型的 DSPy 优化器需要三个输入：

  * **DSPy程序** ：可能是一个单一模块（如[dspy.Predict](https://zhida.zhihu.com/search?content_id=247277477&content_type=Article&match_order=1&q=dspy.Predict&zhida_source=entity)），也可能是一个复杂的多模块程序。
  * **度量** ：一个函数，用于评估程序的输出，并给程序打分（分数越高越好）。
  * **训练输入** ：少量的示例，示例可以是不完整的（只有程序的输入，没有任何标签）。



在优化器模块中包含了很多子优化器，如自动少样本学习[BootstrapFewShot](https://zhida.zhihu.com/search?content_id=247277477&content_type=Article&match_order=1&q=BootstrapFewShot&zhida_source=entity)、LabeledFewShot等，不同的优化器可以使用不同的算法或策略优化提示词的内容，本文主要介绍优化器中常用的 BootstrapFewShot优化器的优化原理。该优化器主要是利用了Bootstrap的思想，先优化一个教师程序，再利用教师程序去训练学生程序。

## BootstrapFewShot使用示例
    
    
    #  创建程序主体
    class CoT(dspy.Module):
        def __init__(self):
            super().__init__()
            self.prog = dspy.ChainOfThought("question -> answer")
    
        def forward(self, question):
            return self.prog(question=question)
    #  设置优化参数
    config = dict(max_bootstrapped_demos=4, max_labeled_demos=4)
    #  实例化优化器
    teleprompter = BootstrapFewShot(metric=your_metric, **config)
    #  开始优化
    teleprompter.compile(CoT(), trainset=your_train_set)

## BootstrapFewShot代码解析

从示例可以看出，BootstrapFewShot优化器对外使用的方法为初始化和compile方法，因此本文从这两个方法出发，逐步分析代码执行逻辑

### 初始化方法

如下为初始化方法，初始化方法主要是定义了优化器的相关配置，具体参数描述可见下面代码注释。
    
    
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
            """
    
            参数
            ----------
            metric: Callable
                一个函数，用于比较预期值和预测值，并输出该比较的结果。
            metric_threshold: 可选 float，默认 `None`
                如果metric返回一个数值，则在决定是否接受引导示例时，将其与此阈值进行比较。
            teacher_settings: dict, 可选
                `teacher`模型的设置。
            max_bootstrapped_demos: int, 默认 4
                包含的最大引导演示数。
            max_labeled_demos: int, 默认 16
                包含的最大标记演示数。
            max_rounds: int, 默认 1
                尝试生成所需引导示例的迭代次数。如果在`max_rounds`后仍未成功，程序将结束。
            max_errors: int, 默认 5
                允许的最大错误次数，超过此次数程序将结束。
            """
            self.metric = metric
            self.metric_threshold = metric_threshold
            self.teacher_settings = {} if teacher_settings is None else teacher_settings
    
            self.max_bootstrapped_demos = max_bootstrapped_demos
            self.max_labeled_demos = max_labeled_demos
            self.max_rounds = max_rounds
            self.max_errors = max_errors
            self.error_count = 0
            self.error_lock = threading.Lock()

### compile()方法

compile() 方法为优化器的优化方法，贯穿了整个优化逻辑，从下面代码可以看出主要分为几个步骤

1）准备训练集 2）准备 teacher 和 student 程序主体，这里面的 teacher 和 student 为相同类实例**即** **[dspy.Module](https://zhida.zhihu.com/search?content_id=247277477&content_type=Article&match_order=1&q=dspy.Module&zhida_source=entity) 类别**，在第一章的例子中为传入的CoT实例 3）获取teacher和student中Predictor的映射关系 4）调用 _bootrap()方法，优化teacher模块 5）调用 _train()方法将teacher优化的信息教给 student程序，最后返回 student 程序。
    
    
        def compile(self, student, *, teacher=None, trainset):
            self.trainset = trainset
    
            self._prepare_student_and_teacher(student, teacher)
            self._prepare_predictor_mappings()
            self._bootstrap()
    
            self.student = self._train()
            self.student._compiled = True
    
            #  set assert_failures and suggest_failures as attributes of student w/ value 0
            self.student._assert_failures = 0
            self.student._suggest_failures = 0
    
            return self.student

接下来我们分别看这些步骤的具体逻辑

**准备teacher和student程序主体**

从代码可以看出，该方法支持传入的student 和 teacher实例，并对他们进行拷贝，如果teacher实例为空，泽使用student实例代替。
    
    
        def _prepare_student_and_teacher(self, student, teacher):
            self.student = student.reset_copy()
            self.teacher = teacher.deepcopy() if teacher is not None else student.reset_copy()
    
            assert getattr(self.student, "_compiled", False) is False, "Student must be uncompiled."
    
            if self.max_labeled_demos and getattr(self.teacher, "_compiled", False) is False:
                teleprompter = LabeledFewShot(k=self.max_labeled_demos)
                self.teacher = teleprompter.compile(self.teacher.reset_copy(), trainset=self.trainset)

**获取teacher和student中Predictor的映射关系**

从代码可以看出，该函数主要构建了student和 teacher实例中包含的predictor的id 与 名字的映射关系，方便后面使用。
    
    
        def _prepare_predictor_mappings(self):
            name2predictor, predictor2name = {}, {}
            student, teacher = self.student, self.teacher
            ...
            for (name1, predictor1), (name2, predictor2) in zip(student.named_predictors(), teacher.named_predictors()):
                ...
                name2predictor[name1] = None  #  dict(student=predictor1, teacher=predictor2)
                predictor2name[id(predictor1)] = name1
                predictor2name[id(predictor2)] = name2
    
            self.name2predictor = name2predictor
            self.predictor2name = predictor2name

**调用 _bootstrap()方法，优化教师程序**

这里的主要逻辑为两层for循环，分别对最大循环次数和数据集进行便利，并在循环中调用 self._bootstrap_one_example 方法。
    
    
        def _bootstrap(self, *, max_bootstraps=None):
            max_bootstraps = max_bootstraps or self.max_bootstrapped_demos
    
            bootstrapped = {}
            self.name2traces = {name: [] for name in self.name2predictor}
    
            for round_idx in range(self.max_rounds):
                for example_idx, example in enumerate(tqdm.tqdm(self.trainset)):
                    if len(bootstrapped) >= max_bootstraps:
                        break
    
                    if example_idx not in bootstrapped:
                        success = self._bootstrap_one_example(example, round_idx)
    
                        if success:
                            bootstrapped[example_idx] = True
    
            dspy.logger.info(
                f"Bootstrapped {len(bootstrapped)} full traces after {example_idx + 1} examples in round {round_idx}.",
            )
    
            #  Unbootstrapped training examples
    
            self.validation = [x for idx, x in enumerate(self.trainset) if idx not in bootstrapped]
            random.Random(0).shuffle(self.validation)
    
            self.validation = self.validation

首先，在 self._bootstrap_one_example 中传入了遍历到的数据集示例，以及循环轮数，根据传入的 round_idx 调整大模型参数存入到 new_settings中，并按照此配置利用 teacher对 传入的example进行预测，预测结果存入到 prediction变量中。然后使用 metric 对预测结果和正确结果进行计算，判断预测是否成功。如果成功则用这一步中的输入输出构建一个demo，并存入到 name2traces中。name2traces 为 self.name2traces的引用，因此可以同步修改。
    
    
        def _bootstrap_one_example(self, example, round_idx=0):
            name2traces = self.name2traces
            teacher = self.teacher  #  .deepcopy()
            predictor_cache = {}
    
            try:
                with dsp.settings.context(trace=[], **self.teacher_settings):
                    lm = dsp.settings.lm
                    lm = lm.copy(temperature=0.7 + 0.001 * round_idx) if round_idx > 0 else lm
                    #  调整大模型的设置
                    new_settings = dict(lm=lm) if round_idx > 0 else {}
    
                    with dsp.settings.context(**new_settings):
                        for name, predictor in teacher.named_predictors():
                            predictor_cache[name] = predictor.demos
                            predictor.demos = [x for x in predictor.demos if x != example]
    
                        prediction = teacher(**example.inputs())
                        trace = dsp.settings.trace
    
                        for name, predictor in teacher.named_predictors():
                            predictor.demos = predictor_cache[name]
    
                    if self.metric:
                        metric_val = self.metric(example, prediction, trace)
                        if self.metric_threshold:
                            success = metric_val >= self.metric_threshold
                        else:
                            success = metric_val
                    else:
                        success = True
            except Exception as e:
                success = False
                with self.error_lock:
                    self.error_count += 1
                    current_error_count = self.error_count
                if current_error_count >= self.max_errors:
                    raise e
                dspy.logger.error(f"Failed to run or to evaluate example {example} with {self.metric} due to {e}.")
    
            if success:
                for step in trace:
                    predictor, inputs, outputs = step
    
                    if "dspy_uuid" in example:
                        demo = Example(augmented=True, dspy_uuid=example.dspy_uuid, **inputs, **outputs)
                    else:
                        #  TODO: FIXME: This is a hack. RandomSearch will complain for now in this edge case.
                        demo = Example(augmented=True, **inputs, **outputs)
    
                    try:
                        predictor_name = self.predictor2name[id(predictor)]
                    except KeyError:
                        continue            
                    name2traces[predictor_name].append(demo)
    
            return success

**将teacher优化结果教给student**

从_train方法可以看出，会循环的将student的所有预测器遍历，并将两种demo赋值给对应预测器的demos变量中，两种demo分别为 raw_demos 和 augmented_demos，其中raw_demos 来自self.validation，augmented_demos来自self.name2traces（即从teacher优化过程中得到的）
    
    
        def _train(self):
            rng = random.Random(0)
            raw_demos = self.validation
            #  循环所有的预测器
            for name, predictor in self.student.named_predictors():
                augmented_demos = self.name2traces[name][: self.max_bootstrapped_demos]
    
                sample_size = min(self.max_labeled_demos - len(augmented_demos), len(raw_demos))
                sample_size = max(0, sample_size)
    
                raw_demos = rng.sample(raw_demos, sample_size)
    
                if dspy.settings.release >= 20230928:
                    predictor.demos = raw_demos + augmented_demos
                else:
                    predictor.demos = augmented_demos + raw_demos
    
            return self.student

## 总结

本文介绍了dspy优化器的 BootstrapFewShot执行原理，主要从初始化函数和compile函数展开分析，在初始化时，BootstrapFewShot会配置好优化所需参数，在运行compile时执行优化流程，其中优化流程又分为1）准备教师和学生模块，此处的教师和学生模块就是优化时传入的dspy.Module类实例 2）获取教师学生模块与对应预测器的映射关系，3）调用bootstrap算法优化教师模块，以及4）将教师的优化结果教给学生模块 4个子步骤。至此学生模型已优化完成，返回得到优化后的dspy.Moudle 实例。

## 相关链接

  1. [https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)
  2. [https://dspy-docs.vercel.app/docs/intro](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/docs/intro)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】11-DSPy和Langchain的无缝集成](https://zhuanlan.zhihu.com/p/714868095)
