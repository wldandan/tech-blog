# 【提示词生成与优化】03-ell：将提示视为函数的轻量级提示工程库

原文链接：https://zhuanlan.zhihu.com/p/9941505513

---

​

目录

## [ell](https://zhida.zhihu.com/search?content_id=251011099&content_type=Article&match_order=1&q=ell&zhida_source=entity)简介

ell是一个轻量级提示（prompt）工程库，它的核心设计理念是将提示视为函数。ell提供了自动化的版本控制和序列化功能，支持[多模态数据处理](https://zhida.zhihu.com/search?content_id=251011099&content_type=Article&match_order=1&q=%E5%A4%9A%E6%A8%A1%E6%80%81%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)，并配备了丰富的本地开源可视化工具，帮助用户优化提示工程过程。

现在面向LLM的编程框架层出不穷，[LangChain](https://zhida.zhihu.com/search?content_id=251011099&content_type=Article&match_order=1&q=LangChain&zhida_source=entity)几乎为我们封装了所有，但是有些过于重了。ell可以理解为LangChain的升级版，其特点是"轻量化"。使用LangChain或者直接面向[OpenAI API](https://zhida.zhihu.com/search?content_id=251011099&content_type=Article&match_order=1&q=OpenAI+API&zhida_source=entity)的原生调用代码，除了提示词之外还要写很多调用代码，而且对于提示词缺乏版本控制管理，这成为了一个痛点。ell轻量化的特点使它将对LLM的操作简化到了只需写提示词。

## ell设计原则

### 提示词不仅仅是字符串，而是程序（[LMP](https://zhida.zhihu.com/search?content_id=251011099&content_type=Article&match_order=1&q=LMP&zhida_source=entity)）

在ell中，**提示词不仅仅是字符串，而是将字符串发送至LLM的整个过程的代码程序** 。这种特定方式成为LMP（language model program）。LMP是完全封装的函数，用于生成要发送到各种多模态语言模型的字符串提示或消息列表。这种封装为用户创建了一个干净的界面，用户只需指定给LMP必要的数据。
    
    
    import ell
    
    @ell.simple(model="gpt-4o-mini")
    def hello(world: str):
        """You are a helpful assistant""" # System prompt
        name = world.capitalize()
        return f"Say hello to {name}!" # User prompt
    
    hello("sam altman") # just a str, "Hello Sam Altman! ..."

### 提示工程是一个优化过程

与机器学习的过程一样，提示工程的过程涉及许多迭代，因此提示词可视作机器学习模型中的参数，**ell提示词工程则类似于机器学习中的参数优化过程** 。因为LMP是函数，ell为此过程提供了丰富的工具。

ell通过静态和动态分析自动对提示进行版本控制和序列化，并直接将gpt-4o-mini自动生成的commit信息发送到本地存储。这个过程类似于机器学习训练循环中的检查点（checkpoint），但它不需要任何特殊的IDE或编辑器,这一切都是用常规的Python代码完成的。
    
    
     import ell
    
     ell.init(store='./logdir')  # Versions your LMPs and their calls
    
     # ... define your lmps
    
     hello("strawberry") # the source code of the LMP the call is saved to the store

### 监控、版本控制和可视化工具

对LLM的每一次调用都很重要，所以要有日志的跟踪监控。有了合适的工具，提示工程从一门神秘的艺术转变为一门科学。**[Ell Studio](https://zhida.zhihu.com/search?content_id=251011099&content_type=Article&match_order=1&q=Ell+Studio&zhida_source=entity) 是一个本地、开源的工具，用于提示词的版本控制、监控、可视化**。通过使用Ell Studio，可以在实践中不断优化提示效果。
    
    
    ell-studio --storage ./logdir

### 更好地实现测试时计算

从演示到应用，经常需要多次调用这个模型。**ell通过把问题分解成更小的、更易于管理的部分，使得我们在模型使用过程中，能够灵活地调整和优化，从而提高模型的性能** 。

ell相当于一个工具箱，能把复杂的指令拆分成一个个小指令，然后按顺序执行。这样一来，就可以更轻松地让LLM完成复杂任务，而且还能够随时调整这些指令，让LLM做得更好。
    
    
    import ell
    from typing import List
    
    ell.init(verbose=True)
    
    @ell.simple(model="gpt-4o-mini", temperature=1.0)
    def generate_story_ideas(about : str):
        """You are an expert story ideator. Only answer in a single sentence."""
        return f"Generate a story idea about {about}."
    
    @ell.simple(model="gpt-4o-mini", temperature=1.0)
    def write_a_draft_of_a_story(idea : str):
        """You are an adept story writer. The story should only be 3 paragraphs."""
        return f"Write a story about {idea}."
    
    @ell.simple(model="gpt-4o", temperature=0.1)
    def choose_the_best_draft(drafts : List[str]):
        """You are an expert fiction editor."""
        return f"Choose the best draft from the following list: {'\n'.join(drafts)}."
    
    @ell.simple(model="gpt-4-turbo", temperature=0.2)
    def write_a_really_good_story(about : str):
        """You are an expert novelist that writes in the style of Hemmingway. You write in lowercase."""
        # Note: You can pass in api_params to control the language model call
        # in the case n = 4 tells OpenAI to generate a batch of 4 outputs.
        ideas = generate_story_ideas(about, api_params=(dict(n=4)))
    
        drafts = [write_a_draft_of_a_story(idea) for idea in ideas]
    
        best_draft = choose_the_best_draft(drafts)
    
    
        return f"Make a final revision of this story in your voice: {best_draft}."
    
    story = write_a_really_good_story("a dog")

除了存储每个LMP的源代码之外，ell还可选择性地将对LLM的每次调用保存到本地。这允许你生成调用数据集、按版本比较LMP输出，并充分利用提示工程的所有相关产物。

【Tips】**测试时计算**(Test-Time Computation)是机器学习和深度学习中的一个概念，指的是在模型推理阶段(也就是测试时)进行额外的计算或处理，以提高模型的性能或适应性。这种方法通常用于解决训练数据和测试数据之间存在差异的问题，或者在不重新训练模型的情况下提高模型的泛化能力。其核心思路是不重新训练模型，而是在模型实际使用时进行额外的处理，以提高模型的表现。类似于人类在应用所学知识时会根据具体情况做出适当灵活变通处理，而不是僵化执行。

### 每次调用LLM都很重要

在实践中，调用LLM常用于微调、蒸馏、k-shot提示、从人类反馈中强化学习等。一个好的提示工程系统应该将这些作为重要的原则。

除了存储每个LMP的源代码外，ell还可以选择性地保存每次调用LLM的记录。这使你可以生成调用数据集，按版本比较LMP输出，并充分利用提示工程的所有过程。

### 根据需要选择@ell.simple或@ell.complex

使用**@ell.simple** 会让LMP生成**简单的字符串输出** 。

@ell.simple装饰器是ell中的一个关键概念。它将一个常规的Python函数转换为一个大语言模型程序（LMP）:

  * 函数的文档字符串成为系统消息
  * 函数的返回值成为用户消息
  * 装饰器处理API调用并以字符串形式返回模型的响应  
这种封装允许更简洁、更可重用的代码。可以像调用任何其他Python函数一样调用LMP。  
但当需要更复杂或多模态的输出时，可以使用**@ell.complex** ，让语言模型返回**Message对象的响应** 。


    
    
    import ell
    
    @ell.tool()
    def scrape_website(url : str):
       return requests.get(url).text
    
    @ell.complex(model="gpt-5-omni", tools=[scrape_website])
    def get_news_story(topic : str):
       return [
          ell.system("""Use the web to find a news story about the topic"""),
          ell.user(f"Find a news story about {topic}.")
       ]
    
    message_response = get_news_story("stock market")
    if message_response.tool_calls:
       for tool_call in message_response.tool_calls:
          #...
    if message_response.text:
       print(message_response.text)
    if message_response.audio:
       # message_response.play_audio() supprot for multimodal outputs will work as soon as the LLM supports it
       pass

### **多模态是一等公民**

LLM可以处理和生成各种类型的文本、图像、音频和视频等内容。针对各种类型的输出进行提示工程，就像提示生成文本一样简单。**Ell原生支持丰富的多模态输入和输出类型转换** 。可以将PIL图像、音频和其他多模态输入内嵌在LMP返回的Message对象中。
    
    
    from PIL import Image
    import ell
    
    
    @ell.simple(model="gpt-4o", temperature=0.1)
    def describe_activity(image: Image.Image):
        return [
            ell.system("You are VisionGPT. Answer <5 words all lower case."),
            ell.user(["Describe what the person in the image is doing:", image])
        ]
    
    # Capture an image from the webcam
    describe_activity(capture_webcam_image()) # "they are holding a book"

### **提示工程库不干扰工作流程**

ell被设计为一个轻量级的库，它**不需要我们改变编码风格或使用特殊的编辑器** 。  
我们可以继续在IDE中使用常规Python代码来定义和修改提示，同时利用ell的功能来可视化和分析提示词。可以逐步从LangChain迁移到ell，一次一个函数。

## **ell使用**

### **安装**

  * Pip安装


    
    
    pip install -U ell-ai[all]

该指令直接安装了ell、ell-studio、基于SQLite进行版本控制和追踪和程序默认客户端。

  * 安装验证


    
    
    python -c "import ell; print(ell.__version__)"

  * 配置OpenAI Key



Windows：
    
    
    setx ANTHROPIC_API_KEY "your-anthropic-api-key"

macOS/Linux
    
    
    # in your .bashrc or .zshrc
    export ANTHROPIC_API_KEY='your-anthropic-api-key'

### 使用指南

**创建第一个提示词程序（LMP）**

首先，先参考基于openai方式的代码程序：
    
    
    from dotenv import load_dotenv
    import os
    from openai import OpenAI
    
    assert load_dotenv()
    assert os.environ.get("OPENAI_BASE_URL")
    assert os.environ.get("OPENAI_API_KEY")
    
    messages = [
        {
            "role": "system",
            "content": """你是用户的生活助理-小智。
    识别用户的意图并用简洁的语言给出合适的回应。
            """,
        },
        {"role": "user", "content": "我想明天早上5点起床"},
    ]
    
    client = OpenAI()
    
    response = client.chat.completions.create(
        model="gpt-4o-mini", messages=messages, temperature=0.9, max_tokens=1000
    )
    
    print(response.choices[0].message.content)

其可能的输出如下：
    
    
    好的，明天早上5点起床。建议你今晚早点睡，确保充足的休息！需要我设置闹钟提醒吗？

然后，让我们看看如何使用ell实现相同的结果：
    
    
    from dotenv import load_dotenv
    import os
    from openai import OpenAI
    import ell
    
    assert load_dotenv()
    assert os.environ.get("OPENAI_BASE_URL")
    assert os.environ.get("OPENAI_API_KEY")
    
    client = OpenAI()
    
    
    @ell.simple(model="gpt-4o-mini", client=client, temperature=0.9, max_tokens=1000)
    def reconize_intent(user_input: str):
        """你是用户的生活助理-小智。
        识别用户的意图并用简洁的语言给出合适的回应。
        """  # System prompt
        return user_input  # User prompt
    
    
    content = reconize_intent("我想明天早上5点起床")
    print(content)

输出如下：
    
    
    好的，建议你今晚早点睡，设定一个闹钟在5点。确保把手机放在能听到的地方哦！

ell通过将提示定义为函数单元来简化提示。最终的提示可以简单地调用该函数并传参，而不是手动构建消息。这使提示更具可读性、可维护性和可重用性。

可以启用ell的verbose模式，这样可以了解后台的细节。启用verbose模式只需要在开始调用：
    
    
    ell.init(verbose=True)

当需要在LMP中构造更复杂的对话，如希望系统提示词可变时，可以使用如下信息形式：
    
    
    import ell
    
    @ell.simple(model="gpt-4o")
    def hello(name: str):
        return [
            ell.system("You are a helpful assistant."),
            ell.user(f"Say hello to {name}!"),
            ell.assistant("Hello! I'd be happy to greet Sam Altman."),
            ell.user("Great! Now do it more enthusiastically.")
        ]
    
    greeting = hello("Sam Altman")
    print(greeting)

**提示词版本控制**

启用ell版本控制，只需在代码开始处添加以下代码：
    
    
    ell.init(store='./logdir', autocommit=True, verbose=True)

这行代码时将在 ./logdir 目录中设置一个存储库并启用自动提交。ell将在./logdir/ell.db中存储所有的提示及其版本。

启用版本控制生成./logdir/ell.db后，可以使用ell-studio来查看提示。在终端中，运行下面的命令：
    
    
    ell-studio --storage-dir ./logdir

服务默认监听本机8080端口，在浏览器打开可以看到下面的界面：

点击inocations还可以看到调用历史：

让我们接下来修改一下reconize_intent这个LMP：
    
    
    from dotenv import load_dotenv
    import os
    from openai import OpenAI
    import ell
    
    assert load_dotenv()
    assert os.environ.get("OPENAI_BASE_URL")
    assert os.environ.get("OPENAI_API_KEY")
    
    client = OpenAI()
    
    
    @ell.simple(model="gpt-4o-mini", client=client, temperature=0.9, max_tokens=1000)
    def reconize_intent(user_input: str):
        """你是用户的生活助理-小智。
        识别用户的意图并用简洁的语言给出合适的回应。
        回应的格式是一个json数组,每个对象json包含字段intent和parameters
        如果无法识别用户的意图, json对象中intent字段的值固定为unknown, 并将user_input字段设置为用户的输入
        """  # System prompt
        return user_input  # User prompt
    
    
    ell.init(store="./logdir", autocommit=True, verbose=True)
    content = reconize_intent("我想明天早上5点起床")
    print(content)

此时再次查看ell studio，可以看到reconize_intent这个LMP更新到了Version 2：

### API参考

参考[https://docs.ell.so/reference/index.html](https://link.zhihu.com/?target=https%3A//docs.ell.so/reference/index.html)

## 相关链接

  1. ell官方网站：[https://docs.ell.so/index.html](https://link.zhihu.com/?target=https%3A//docs.ell.so/index.html)
  2. ell GitHub仓：[https://github.com/MadcowD/ell](https://link.zhihu.com/?target=https%3A//github.com/MadcowD/ell)
  3. 《初探轻量级LLM应用开发框架ell》：[https://blog.frognew.com/2024/09/intro-ell-framework.html](https://link.zhihu.com/?target=https%3A//blog.frognew.com/2024/09/intro-ell-framework.html)
  4. 《OpenAI前研究科学家开源面向未来的提示工程库 ell，重新定义提示工程》：[https://www.53ai.com/news/tishicikuangjia/2024091246095.html](https://link.zhihu.com/?target=https%3A//www.53ai.com/news/tishicikuangjia/2024091246095.html)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Prompt洞察】02-TextGrad：通过文本实现自动“微分”](https://zhuanlan.zhihu.com/p/9192012739)  
> 下一篇：[【Prompt洞察】04-GRAD-SUM：利用梯度摘要优化提示工程](https://zhuanlan.zhihu.com/p/10673233589)
