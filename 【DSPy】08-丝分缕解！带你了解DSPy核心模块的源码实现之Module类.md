# 【DSPy】08-丝分缕解！带你了解DSPy核心模块的源码实现之Module类

原文链接：https://zhuanlan.zhihu.com/p/709324353

---

​

目录

书接上文《[【[DSPy](https://zhida.zhihu.com/search?content_id=245759035&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)技术洞察】07-丝分缕解！带你了解DSPy核心模块的源码实现之[Signature类](https://zhida.zhihu.com/search?content_id=245759035&content_type=Article&match_order=1&q=Signature%E7%B1%BB&zhida_source=entity)](https://zhuanlan.zhihu.com/p/708332838)》，通过示例和源码解析的方式对Signature类进行了详细的解读，本文带你了解DSPy核心模块-[Module类](https://zhida.zhihu.com/search?content_id=245759035&content_type=Article&match_order=1&q=Module%E7%B1%BB&zhida_source=entity)的源码实现。

DSPy是斯坦福 NLP 组推出的一个用于优化和生成Prompt的框架，DSPy将传统手工编写提示词的方式抽象为高代码编程方式，其核心思路为：

  1. 将整体流程与单步骤分开。每个步骤聚焦具体的工作，协同完成Prompt的优化
  2. 引入多样的优化器，这些优化器是由[LM驱动](https://zhida.zhihu.com/search?content_id=245759035&content_type=Article&match_order=1&q=LM%E9%A9%B1%E5%8A%A8&zhida_source=entity)的算法，可以根据定义的阈值来调整调用LM的提示及访问参数。



上述步骤主要由 Signature、Module、[Optimizer](https://zhida.zhihu.com/search?content_id=245759035&content_type=Article&match_order=1&q=Optimizer&zhida_source=entity)三个核心模块实现，本文将通过示例和源码解析的方式，解读DSPy运行原理。

## DSPy 实现原理之Module类

DSPy 模块在 LLM pipeline 中抽象了传统的 prompting 技术。它们具有三个关键特性：

  * 每个内置模块抽象出一个特定的 prompting 技术（如 [Chain of Thoughts](https://zhida.zhihu.com/search?content_id=245759035&content_type=Article&match_order=1&q=Chain+of+Thoughts&zhida_source=entity) 或 ReAct）并处理 DSPy 签名。
  * DSPy 模块具有可学习的参数，包括 prompt 组件和 LLM 权重，使它们能够处理输入并生成输出。
  * DSPy 模块可以组合，从而创建更大、更复杂的模块。



## Module类简单示例

首先来看一个示例来解释Module类的用法，想要创建自定义dspy.Module类，首先需要继承 dspy.Module，然后必须实现 __init__() 方法和forward() 方法，其中，forward方法中实现具体的推理逻辑，代码如下：
    
    
    class CoT(dspy.Module):# 自定义CoT类
        def __init__(self):
            super().__init__()
            self.prog = dspy.ChainOfThought("question -> answer")
        def forward(self, question):
            return self.prog(question=question)
    cot = CoT()
    question = ""
    response = cot(question=question)

提示词和执行结果为
    
    
    Given the fields `question`, produce the fields `answer`.
    ---
    Follow the following format.
    Question: ${question}
    Reasoning: Let's think step by step in order to ${produce the answer}. We ...
    Answer: ${answer}
    ---
    Question: what is the color of the sea?
    Reasoning: Let's think step by step in order to Question: what is the color of the sea?
    Reasoning: Let's think step by step in order to determine the color of the sea. The sea is a vast body of water that covers over 70% of the Earth's surface. It is primarily composed of water, with a small amount of salt. The color of the sea is determined by a number of factors, including the water temperature, the presence of dissolved particles, and the presence of phytoplankton.
    Answer: The color of the sea is typically perceived as blue.

## Module类源码解读

在上一节中，我们介绍了上述提示词中的内容是如何通过Signature类传入并进行格式化转换的，本小节我们介绍Module中是如何组织Signature 以及思维模式来得到提示词，并执行推理流程。

通过本小节通过源码分析的方式，解释如下问题：

  * Module类的推理流程
  * Predict类的执行流程



### **Module类的推理流程**

Module类可以理解为一个数据的集合，它将输入的 Signature的内容，推理用到的 Predict，Retrieve等内容，集中到一起，起到调度的作用，如下面代码所示，Module类继承自BaseMoudle，并且也设置了元类 ProgramMeta，但是目前ProgramMeta中没有具体代码逻辑。  
当Module的子类初始化时，首先会调用Module类的 __init__() 函数，然后调用子类 初始化函数逻辑。  
在推理时，子类实例化对象会调用 父类的 __call__()函数，__call__()函数有调用了 子类的forward()函数（Module.__call__ -> SubClassOfMoudle.forward），因此子类执行时，实际在掉用自身的foward函数。
    
    
    class Module(BaseModule, metaclass=ProgramMeta):# Module类定义
        def _base_init(self):
            self._compiled = False
        def __init__(self):
            self._compiled = False
        def __call__(self, *args, **kwargs):
            return self.forward(*args, **kwargs)

### **Predict类的执行流程**

子类的forward执行时，需要实现对应的执行逻辑，或使用官方提供的类，如 dspy.Predict类及Predict子类，例如，dspy.ChainOfThought,本文以 dspy.ChainOfThougth 和 dspy.Predict 为例，解释具体推理流程。
    
    
    class ChainOfThought(Predict):# ChainOfThought类定义
        def __init__(self, signature, rationale_type=None, activated=True, **config):
            super().__init__(signature, **config)
            self.activated = activated
            signature = ensure_signature(self.signature)
            *_keys, last_key = signature.output_fields.keys()
            rationale_type = rationale_type or dspy.OutputField(
                prefix="Reasoning: Let's think step by step in order to",
                desc="${produce the " + last_key + "}. We ...",
            )
            self.extended_signature = signature.prepend("rationale", rationale_type, type_=str)
        def forward(self, **kwargs):
            new_signature = kwargs.pop("new_signature", None)
            if new_signature is None:
                if self.activated is True or (
                    self.activated is None and isinstance(dsp.settings.lm, dsp.GPT3)
                ):
                    signature = self.extended_signature
                else:
                    signature = self.signature
            else:
                signature = new_signature
                # template = dsp.Template(self.signature.instructions, **new_signature)
            return super().forward(signature=signature, **kwargs)

如上文代码所示，在初始化 ChainOfThought类时，为原有的signature增加了 ratiaonale内容，支持用户输入或使用默认内容。
    
    
    class Predict(Parameter):
        def __init__(self, signature, **config):
           ...
        def __call__(self, **kwargs):
            return self.forward(**kwargs)

然后，在推理时，如上面代码所示，调用过程为，`CoT.forward -> CoT.prog-> Predict.__call__ -> ChainOfThought.forward -> Predict.forward`

因此，实际的执行代码主要在 Predict.forward中，如下面所示为Predict.forward函数代码
    
    
    class Predict(Parameter):
        def forward(self, **kwargs):
           # Extract the three privileged keyword arguments.
           new_signature = ensure_signature(kwargs.pop("new_signature", None))
           signature = ensure_signature(kwargs.pop("signature", self.signature))
           demos = kwargs.pop("demos", self.demos)
           config = dict(**self.config, **kwargs.pop("config", {}))
           # Get the right LM to use.
           lm = kwargs.pop("lm", self.lm) or dsp.settings.lm
           assert lm is not None, "No LM is loaded."
           # If temperature is 0.0 but its n > 1, set temperature to 0.7.
           temperature = config.get("temperature")
           temperature = lm.kwargs["temperature"] if temperature is None else temperature
           num_generations = config.get("n")
           if num_generations is None:
               num_generations = lm.kwargs.get("n", lm.kwargs.get("num_generations", None))
           if (temperature is None or temperature <= 0.15) and num_generations > 1:
               config["temperature"] = 0.7
               # print(f"#> Setting temperature to 0.7 since n={num_generations} and prior temperature={temperature}.")
           # All of the other kwargs are presumed to fit a prefix of the signature.
           # That is, they are input variables for the bottom most generation, so
           # we place them inside the input - x - together with the demos.
           x = dsp.Example(demos=demos, **kwargs)
           if new_signature is not None:
               signature = new_signature
           if not all(k in kwargs for k in signature.input_fields):
               present = [k for k in signature.input_fields if k in kwargs]
               missing = [k for k in signature.input_fields if k not in kwargs]
               print(f"WARNING: Not all input fields were provided to module. Present: {present}. Missing: {missing}.")
           # Switch to legacy format for dsp.generate
           template = signature_to_template(signature)  # 提示词对象
           if self.lm is None:
               x, C = dsp.generate(template, **config)(x, stage=self.stage)# 调用大模型预测
           else:
               # Note: query_only=True means the instructions and examples are not included.
               # I'm not really sure why we'd want to do that, but it's there.
               with dsp.settings.context(lm=self.lm, query_only=True):
                   x, C = dsp.generate(template, **config)(x, stage=self.stage)# 调用大模型预测
           assert self.stage in x, "The generated (input, output) example was not stored"
           completions = []
           for c in C:
               completions.append({})
               for field in template.fields:
                   if field.output_variable not in kwargs.keys():
                       completions[-1][field.output_variable] = getattr(
                           c,
                           field.output_variable,
                       )
           pred = Prediction.from_completions(completions, signature=signature)
           if kwargs.pop("_trace", True) and dsp.settings.trace is not None:
               trace = dsp.settings.trace
               trace.append((self, {**kwargs}, pred))
           return pred

具体步骤如下：

  1. 调用 ensure_signature 函数，得到signature实例，里面包含 rationale、question、answer字段。
  2. 获取加入提示词中的示例放入到 demos变量中，将demos和输入 question 通过 Example类组装成 x变量。
  3. 调用 signature_to_template 函数，获取提示词对象template。
  4. 调用 dsp.generate 函数，将 提示词，输入参数x，组装成完整提示词，并调用大模型返回结果存储到变量 x 和 C中。
  5. 其中C为字段以及内容组成的结构体（类似于字典类型），包括 question、answer等内容，最终调用 from_completion 函数组装为Prediction实例化对象返回。



下一篇将会对Optimizer进行源码解析，欢迎大家持续关注。感谢本文作者“阿拉赫莫拉”供稿。

## 相关链接

  1. [dspy官网](https://link.zhihu.com/?target=https%3A//wiki.huawei.com/domains/30974/wiki/49918/WIKI202406133768336) [https://dspy-docs.vercel.app](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/)
  2. Dspy github仓库地址 [https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】07-丝分缕解！带你了解DSPy核心模块的源码实现之Signature类](https://zhuanlan.zhihu.com/p/708332838)
