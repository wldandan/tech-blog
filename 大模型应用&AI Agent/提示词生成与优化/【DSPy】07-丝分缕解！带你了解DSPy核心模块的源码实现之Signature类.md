# 【DSPy】07-丝分缕解！带你了解DSPy核心模块的源码实现之Signature类

原文链接：https://zhuanlan.zhihu.com/p/708332838

---

​

目录

DSPy是[斯坦福 NLP 组](https://zhida.zhihu.com/search?content_id=245539025&content_type=Article&match_order=1&q=%E6%96%AF%E5%9D%A6%E7%A6%8F+NLP+%E7%BB%84&zhida_source=entity)推出的一个用于优化和生成Prompt的框架，DSPy将传统手工编写提示词的方式抽象为高代码编程方式，其核心思路为：

  1. 将整体流程与单步骤分开。每个步骤聚焦具体的工作，协同完成Prompt的优化
  2. 引入多样的优化器，这些优化器是由LM驱动的算法，可以根据定义的阈值来调整调用LM的提示及访问参数。



上述步骤主要由 Signature、Module、Optimizer三个核心模块实现，本文将通过示例和源码解析的方式，解读DSPy运行原理。

## DSPy 实现原理之[Signature类](https://zhida.zhihu.com/search?content_id=245539025&content_type=Article&match_order=1&q=Signature%E7%B1%BB&zhida_source=entity)

DSPy的Signature类，是DSPy核心模块之一，Signature是声明性的规范，定义了 DSPy 模块的输入/输出行为，用于告诉语言模型应执行哪些任务，而不是我们应如何设置 prompt 语言模型。

一个签名包括三个基本元素：

  * 语言模型旨在解决的子任务的简洁描述。
  * 我们提供给语言模型的一个或多个输入字段的描述（例如，输入问题）。
  * 我们期望从语言模型得到的一个或多个输出字段的描述（例如，问题的答案）。



### Signature的基本用法示例

首先来看一个示例来解释Signature的用法，需要准备签名所需的三要素，下面代码进行三步操作：

  1. 将子任务描述填入函数下面的注释中。
  2. 定义输入字段（必选），并对输入字段设置描述（可选）。
  3. 定义输出字段（必选），并对输出字段设置描述（可选）。


    
    
    class QA(dspy.Signature):
        """answer the question of user"""
    
        user_question = dspy.InputField(desc="用户的问题")
        answer = dspy.OutputField()

然后我们使用dspy.[ChainOfThought](https://zhida.zhihu.com/search?content_id=245539025&content_type=Article&match_order=1&q=ChainOfThought&zhida_source=entity) 进行一次推理，并查看提示词最终的内容
    
    
    question = "what is the color of the sea?"
    summarize = dspy.ChainOfThought(QA)
    response = summarize(question=question)
    #  查看提示词
    lm.inspect_history(n=1)

提示词的内容如下，通过提示词内容，可以看出Signature可以将类定义中的注释内容，转换为对这个子任务的描述填写到提示词开头部分，然后将输入输出字段，分别以统一的格式（首字母大写，单词用空格分开）排布在 Reasoning的前后，其中Reasoning的内容为 CoT模式的固定提示词。
    
    
    answer the question of user
    
    ---
    
    Follow the following format.
    
    User Question: 用户的问题
    Reasoning: Let's think step by step in order to ${produce the answer}. We ...
    Answer: ${answer}
    
    ---
    
    Question：what is the color of the sea?
    Reasoning: Let's think step by step in order to Question: what is the color of the sky?
    Reasoning: Let's think step by step in order to determine the color of the sky. The sky appears blue due to the scattering of light waves in the atmosphere. The blue light is scattered more efficiently than other colors of light, which is why the sky appears blue.
    Answer: Blue

通过上面的例子，我们了解了如何通过写代码的方式，产生提示词，并使得提示词包括我们对任务描述和输入输出参数描述。

此外，Signature还可以通过字符串定义输入输出方式，代替通过继承方式定义，代码如下：
    
    
    summarize = dspy.ChainOfThought('question -> answer')

但是这种方式只能定义输入输出的字段名，无法定义任务描述 和 字段描述，此时产生的提示词的任务描述是默认任务描述，提示词如下：
    
    
    Given the fields `question`, produce the fields `answer`.
    
    ---
    
    Follow the following format.
    
    Question: ${question}
    Reasoning: Let's think step by step in order to ${produce the answer}. We ...
    Answer: ${answer}
    
    ---

至此，我们介绍了Signature的两种用法。

### Signature源码解析

那么，Signature类是如何实现上述功能的呢？

我们将通过源码解释以下问题：

  * Signature执行流程
  * 字符串如何转换为Signature类
  * Signature的变量如何转换为提示词中的内容



我们深入Signature类的代码中进行简单的解释（文件位置：dspy\signatures）：
    
    
    # signature.py 部分删减
    import ast
    import re
    import types
    import typing
    from copy import deepcopy
    from typing import Any, Dict, Tuple, Type, Union  # noqa: UP035
    from [pydantic](https://zhida.zhihu.com/search?content_id=245539025&content_type=Article&match_order=1&q=pydantic&zhida_source=entity) import BaseModel, Field, create_model
    from pydantic.fields import FieldInfo
    import dsp
    from dspy.signatures.field import InputField, OutputField, new_to_old_field
    
    class SignatureMeta(type(BaseModel)):   # Signature元类
        def __call__(cls, *args, **kwargs):  # noqa: ANN002
            if cls is Signature:
                return make_signature(*args, **kwargs)
            return super().__call__(*args, **kwargs)
        def __new__(mcs, signature_name, bases, namespace, **kwargs):  # noqa: N804     # 初始化Signature时调用该函数
            # Set `str` as the default type for all fields
            raw_annotations = namespace.get("__annotations__", {})
            for name, field in namespace.items():
                if not isinstance(field, FieldInfo):
                    continue  # Don't add types to non-field attributes
                if not name.startswith("__") and name not in raw_annotations:
                    raw_annotations[name] = str
            namespace["__annotations__"] = raw_annotations
            # Let Pydantic do its thing
            cls = super().__new__(mcs, signature_name, bases, namespace, **kwargs)
            # If we don't have instructions, it might be because we are a derived generic type.
            # In that case, we should inherit the instructions from the base class.
            if cls.__doc__ is None:
                for base in bases:
                    if isinstance(base, SignatureMeta):
                        doc = getattr(base, "__doc__", "")
                        if doc != "":
                            cls.__doc__ = doc
            # The more likely case is that the user has just not given us a type.
            # In that case, we should default to the input/output format.
            if cls.__doc__ is None:
                cls.__doc__ = _default_instructions(cls)
            # Ensure all fields are declared with InputField or OutputField
            cls._validate_fields()
            # Ensure all fields have a prefix
            for name, field in cls.model_fields.items():
                if "prefix" not in field.json_schema_extra:
                    field.json_schema_extra["prefix"] = infer_prefix(name) + ":"
                if "desc" not in field.json_schema_extra:
                    field.json_schema_extra["desc"] = f"${{{name}}}"
            return cls
        ...
    
    class Signature(BaseModel, metaclass=SignatureMeta):
        ""  # noqa: D419
        # Note: Don't put a docstring here, as it will become the default instructions
        # for any signature that doesn't define it's own instructions.
        pass
    def make_signature(         # 根据给定的参数创建Signature 实例
        signature: Union[str, Dict[str, Tuple[type, FieldInfo]]],
        instructions: str = None,
        signature_name: str = "StringSignature",
    ) -> Type[Signature]:
        """Create a new Signature type with the given fields and instructions.
        Note:
            Even though we're calling a type, we're not making an instance of the type.
            In general, instances of Signature types are not allowed to be made. The call
            syntax is provided for convenience.
        Args:
            signature: The signature format, specified as "input1, input2 -> output1, output2".
            instructions: An optional prompt for the signature.
            signature_name: An optional name for the new signature type.
        """
        fields = _parse_signature(signature) if isinstance(signature, str) else signature
        # Validate the fields, this is important because we sometimes forget the
        # slightly unintuitive syntax with tuples of (type, Field)
        fixed_fields = {}
        for name, type_field in fields.items():
            if not isinstance(name, str):
                raise ValueError(f"Field names must be strings, not {type(name)}")
            if isinstance(type_field, FieldInfo):
                type_ = type_field.annotation
                field = type_field
            else:
                if not isinstance(type_field, tuple):
                    raise ValueError(f"Field values must be tuples, not {type(type_field)}")
                type_, field = type_field
            # It might be better to be explicit about the type, but it currently would break
            # program of thought and teleprompters, so we just silently default to string.
            if type_ is None:
                type_ = str
            # if not isinstance(type_, type) and not isinstance(typing.get_origin(type_), type):
            if not isinstance(type_, (type, typing._GenericAlias, types.GenericAlias)):
                raise ValueError(f"Field types must be types, not {type(type_)}")
            if not isinstance(field, FieldInfo):
                raise ValueError(f"Field values must be Field instances, not {type(field)}")
            fixed_fields[name] = (type_, field)
        # Fixing the fields shouldn't change the order
        assert list(fixed_fields.keys()) == list(fields.keys())  # noqa: S101
        # Default prompt when no instructions are provided
        if instructions is None:
            sig = Signature(signature, "")  # Simple way to parse input/output fields
            instructions = _default_instructions(sig)
        return create_model(
            signature_name,
            __base__=Signature,
            __doc__=instructions,
            **fixed_fields,
        )
    def _parse_signature(signature: str) -> Tuple[Type, Field]:     # 将字符串形式的输入输出转为对象
        if signature.count("->") != 1:
            raise ValueError(f"Invalid signature format: '{signature}', must contain exactly one '->'.")
        fields = {}
        inputs_str, outputs_str = map(str.strip, signature.split("->"))
        inputs = [v.strip() for v in inputs_str.split(",") if v.strip()]
        outputs = [v.strip() for v in outputs_str.split(",") if v.strip()]
        for name_type in inputs:
            name, type_ = _parse_named_type_node(name_type)
            fields[name] = (type_, InputField())
        for name_type in outputs:
            name, type_ = _parse_named_type_node(name_type)
            fields[name] = (type_, OutputField())
        return fields
    def infer_prefix(attribute_name: str) -> str:       # 同意在提示词中的格式，如首字母大写等
        """Infer a prefix from an attribute name."""
        # Convert camelCase to snake_case, but handle sequences of capital letters properly
        s1 = re.sub("(.)([A-Z][a-z]+)", r"\1_\2", attribute_name)
        intermediate_name = re.sub("([a-z0-9])([A-Z])", r"\1_\2", s1)
        # Insert underscores around numbers to ensure spaces in the final output
        with_underscores_around_numbers = re.sub(
            r"([a-zA-Z])(\d)",
            r"\1_\2",
            intermediate_name,
        )
        with_underscores_around_numbers = re.sub(
            r"(\d)([a-zA-Z])",
            r"\1_\2",
            with_underscores_around_numbers,
        )
        # Convert snake_case to 'Proper Title Case', but ensure acronyms are uppercased
        words = with_underscores_around_numbers.split("_")
        title_cased_words = []
        for word in words:
            if word.isupper():
                title_cased_words.append(word)
            else:
                title_cased_words.append(word.capitalize())
        return " ".join(title_cased_words)

**1、执行流程**

从代码中代码中可以看出，`Signature`类集成了`pydantic.BaseModel`，并设置了元类`SignatureMeta`；`pydantic.BaseModel`是格式校验的类；`SignatureMeta`是自定义类，`Signature`的大部分提示词逻辑都来自`SignatureMeta`类。

在`Signature`类及子类初始化时，首先会调用`SignatureMeta`的`__call__ `函数，`__call__`中调用了 `make_signature`函数，该函数主要是解析输出的字段，最终调用`pydantic.create_model`函数创建pydantic的格式类`（__call__ -> make_signature -> pydantic.create_model）`，格式类中已经包含了输入输出字段以及对应的字段描述。

然后调用`SignatureMeta`的` __new__`函数，该函数将子任务描述，传入到`cls.__doc__`变量中，并且利用`infer_prefix`函数，修改了输入输出字段的格式，使他们统一为首字母大写的形式，存入到 `field.json_schema_extra["prefix"]`中，将字段的描述，存入`field.json_schema_extra["desc"]`字段中，并返回类。

**2、字符串如何转为Signature类**

如果输入为字符串类型，则`make_signature`中的`_parse_signature`函数会 格式化这个字符串并转换为 `dspy.InputFiled`或`dspy.OutputField`类型，至此，通过字符串和`Signature`类定义的方式都统一成了相同类型。

**3、Signature的变量如何转换为提示词中的内容**

在初始化时，调用了`SignatureMeta`类的`__call__ `函数和` __new__`函数，两个函数创建了pydantic.BaseModal类，并将输入输出字段进行格式化，将任务描述存入`cls.__doc__`，至此，所有代码描述的注释和变量，都转变为了提示词中将要用到的字符串内容，在`[dspy.Module](https://zhida.zhihu.com/search?content_id=245539025&content_type=Article&match_order=1&q=dspy.Module&zhida_source=entity)`执行时，则会对提示词进行填充。

后续将会对Module（模块）类、Optimizer（优化）类进行源码解析，欢迎大家持续关注。下一篇将会解析Module（模块）类，敬请关注~感谢本文作者“阿拉赫莫拉”供稿。

## 相关链接

  1. [dspy官网](https://link.zhihu.com/?target=https%3A//wiki.huawei.com/domains/30974/wiki/49918/WIKI202406133768336) [https://dspy-docs.vercel.app](https://link.zhihu.com/?target=https%3A//dspy-docs.vercel.app/)
  2. Dspy github仓库地址 [https://github.com/stanfordnlp/dspy](https://link.zhihu.com/?target=https%3A//github.com/stanfordnlp/dspy)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【DSPy技术洞察】06-Prompt或许的新未来， DSPy使用从0到1快速上手](https://zhuanlan.zhihu.com/p/707925423)  
> 下一篇：[【DSPy技术洞察】08-丝分缕解！带你了解DSPy核心模块的源码实现之Module类](https://zhuanlan.zhihu.com/p/709324353)
