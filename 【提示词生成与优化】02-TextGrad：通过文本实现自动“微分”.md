# 【提示词生成与优化】02-TextGrad：通过文本实现自动“微分”

原文链接：https://zhuanlan.zhihu.com/p/9192012739

---

​

目录

## **概述**

由于大型语言模型（[LLM](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=LLM&zhida_source=entity)s）的突破，AI系统的构建范式正在发生转变。新一代的AI应用程序越来越多地采用复合系统，涉及多个复杂组件的，每个组件都可能是基于LLM的Agent、模拟器或网络搜索工具。然而，这些突破很多都来自于由应用领域的专家手工创建并通过启发式方法进行调整的系统。因此，开发有原则的自动化优化方法来优化AI系统，是构建LLM复合系统的关键挑战之一，并且是释放AI潜力的必要条件。  
**[TextGrad](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=TextGrad&zhida_source=entity) ，用于优化新一代的AI系统**，不仅仅局限于Prompt。  
TextGrad，这是一种**通过文本进行自动“微分”的框架，** 其**通过LLM提供的文本反馈来优化复合AI系统的各个组件** 。在TextGrad 中，每个AI系统被转换为一个[计算图](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=%E8%AE%A1%E7%AE%97%E5%9B%BE&zhida_source=entity)，其中变量是复杂（不一定可微分）函数调用的输入和输出。对变量的反馈（被称为“[文本梯度](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=%E6%96%87%E6%9C%AC%E6%A2%AF%E5%BA%A6&zhida_source=entity)”）是以信息丰富且可解释的自然语言批评的形式进行提供，描述了如何改变变量以优化系统。  
TextGrad遵循[PyTorch](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=PyTorch&zhida_source=entity)的语法和抽象，灵活且易于使用。它适用于各种任务，用户只需提供目标函数，无需调整框架的组件或提示。  
作者们在论文中相继以编码、问题解决、化学、医学等领域以示例的方式展现了TextGrad的灵活性，也展示了TextGrad通过文本反向传播反馈自动优化复合AI系统的潜力。项目在GitHub进行了开源，[https://github.com/zou-group/TextGrad](https://link.zhihu.com/?target=https%3A//github.com/zou-group/TextGrad)。

## **关键技术**

### **通用范式**

该抽象适用于任意复杂的系统。通过如下方式定义计算图：

υ=fυ(PredecessorsOf(υ))\upsilon=f_{\upsilon}\left( PredecessorsOf\left( \upsilon \right) \right)  ∀υ∈ν\forall_{\upsilon}\in \nu （10）

其中， υ\upsilon 是图中的一个变量， 是图中所有变量的集合，SuccessorsOf是返回变量的后继变量，PredecessorsOf是返回变量的前继变量。一般来说，υ\upsilon的值可以是非结构化数据，如自然语言文本或图像。在本文的大部分结果和论述中，υ\upsilon是自然语言文本。  
进一步，假设为消费一组变量并生成变量υ\upsilon的变换。例如，可以使用LLM或数值模拟器作为变换。由于不同的函数将有不同的计算梯度和收集反馈的方法，通常使表示函数的梯度函数。为了叙述方便，当函数显而易见时，将省略下标。  
梯度计算公式为：

从 的所有后继中收集梯度集，即从变量 被使用的每个上下文中获取反馈，并将它们汇总。  
方程11递归计算图中相对于所需变量 的下游目标的梯度。函数将相对于给定变量 的后继的梯度、变量 的值和后继本身作为输入。请注意，最终的梯度变量包含了变量在任何地方使用时的上下文和批评。  
最后，要更新图中任何所需的变量 ，可以使用优化器：

该优化器根据变量的当前值和梯度更新 的值。对于有n条边的计算图，每次优化迭代最多需要额外调用n次语言模型来计算梯度（每条边都需要调用一次梯度运算符）。

### **目标函数**

在数值优化和[自动微分](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%BE%AE%E5%88%86&zhida_source=entity)中，目标函数通常是可微分函数，如均方误差或交叉熵。在TextGrad中，目标可以是复杂且可能不可微分的函数，函数的定义域和值域可以是非结构化数据。这种选择为框架增加了重要的通用性和灵活性。例如，代码片段的简单损失函数可以如下：

使用这一评估信号来优化代码片段，这得益于LLM模拟人类反馈、自我评估和自我改进的能力。

### **实例优化vs提示优化**

探索的优化问题分为两类，这两类问题都可以在无需手工调整框架的情况下解决。。

  * **在实例优化中** ，直接将问题的解决方案（例如，代码片段、问题的解决方案或分子）视为优化变量。
  * **在提示优化中** ，目标是找到一个提示，以提高LLM在多个任务上的性能。



### 优化技术

自动微分是TextGrad的强类比，并为框架中实现的优化技术提供概念支持。以下是一些示例。

  * **批量优化：** 为提示优化实现了随机小批量梯度下降。具体来说，在对批次中的多个实例进行前向传递并评估各个损失项后，使用tg.sum函数对损失进行求和（类似于 torch.sum）。在反向传递中，对变量的梯度通过各个损失项串联起来，类似于通过加法反向传播。
  * **约束优化：** 使用自然语言约束，以约束优化作为类比。特别是，使用自然语言约束（例如，“你的回答的最后一行应是如下格式：‘答案：$LETTER’，其中LETTER是ABCD中的之一。”）来引导优化器的行为。作者们注意到，得益于指令微调，语言模型可以遵循这些简单约束条件，尽管约束过多时可能会降低它们的可靠性。
  * **动量：** 使用梯度下降中的动量类比。在优化变量时，TGD优化器在进行更新时可以选择性地查看变量的早期迭代。



### 示例

假设一个简单系统的计算图如下：

> Prediction = LLM(Prompt + Question), (1)   
> Evaluation = LLM(Evaluation Instruction + Prediction), (2)

  
图1 通过文本自动“微分”（a, b）。梯度的反向传播是深度学习的驱动力。TextGrad的抽象（c）。TextGrad的应用（d, e, f, g）。  
在这里，使用绿色表示要优化的自由参数Prompt，并用+表示两个字符串的连接，用 LLM(x) 表示将x作为提示传递给语言模型以收集响应。链式表示法如下

在这个系统中，进行一次LLM调用用于使用Prompt为Question生成Prediction，并进行另一次LLM调用以评估这个Prediction。

**梯度计算**

在这个示例系统中，为了改进Prompt以提高评估结果，通过以下方式实例化类似反向传播算法：

下面是一种灵活的方法来实例化∇，以收集的反馈：

**优化器**

在标准梯度下降中，当前变量值与梯度通过减法结合，例如：

延续基于梯度的优化类比，使用文本梯度下降（TGD）：

具体地实例化 TGD 的方式如下：

## 实验与数据

本文TextGrad在多种应用中的灵活性。

### **代码优化**

代码优化的计算图：

  * **任务：** 使用 [LeetCode Hard](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=LeetCode+Hard&zhida_source=entity) 数据集作为代码优化的基准。
  * **基线：** 采用Reflexion，其是LeetCode Hard数据集上最先进的方法。
  * **结果：** 结果显示，GPT-4零样本的通过率为7%，使用Reflexion的通过率为15%。使用gpt-4o零样本时的通过率为23%，使用Reflexion的通过率为31%。使用TextGrad，通过率可以达到36%的性能。  
表1：使用gpt-4o在LeetCode Hard上进行代码优化。结果取5个用例的平均值。



### 解决方案优**化**

通过TextGrad进行解决方案优化的任务，其计算图如下：

该方案中，优化的参数是解决方案，损失函数通过对解决方案的评估获取，如使用LLM。  
改进的目标函数如下所示：

下面是TextGrad中解决方案优化的一个代表性实现示例。
    
    
    # Assume we have the test_dataset
    question , answer = test_dataset [i ]
    # Initialize the test time loss function
    # This has the system prompt provided above as ‘Solution Refinement Objective ’
    test_time_loss_fn = MultipleChoiceTestTime ()
    # Get a zero - shot solution from an LLM
    solution : Variable = zero_shot_llm ( question )
    # Optimize the solution itself
    optimizer = tg. TextualGradientDescent ( parameters =[ solution ])
    
    for iteration in range ( max_iterations ):
        optimizer . zero_grad ()
        # Note how the loss is self - supervised ( does not depend on any ground truth .)
        loss = test_time_loss_fn ( question , solution )
        # Populate the gradients , and update the solution
        loss . backward ()
        optimizer . step ()

**任务** ：使用Google-proof Question Answering (GPQA)基准，其中物理、生物和化学方面的多选题由具有或正在攻读博士学位的领域专家创建并标注。此外，使用[MMLU](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=MMLU&zhida_source=entity)问题回答基准所包含的两个子集，机器学习和大学物理，该基准用于跟踪语言模型的进展及其是否达到人类水平的表现。  
**方法** ：包含两个基线。首先，gpt-4o使用了0.5的生成温度。对于TEXTGRAD，执行3次测试时更新（即更新解决方案三次）并在所有解决方案中进行多数投票以获得最终答案。  
**结果** ：如下表所示，在Google-proof QA场景下，TEXTGRAD达到了55%；在MMLU-ML和MMLU-College Physics场景下，分别达到了88.4%和95.1%。  
表2 gpt-4o的零样本问答解决方案优化

### 提示优化

在提示优化中，目标是找到一个提示或指令来指导LLM的行为，以便其在给定任务中表现良好。其计算图如下：

该方案中，包含任务的问题、问题的答案以及表示输出质量的评估指标（比如根据标准答案评估答案的准确性）。  
本方案通过给定少量的训练示例来优化提示，目标是最大化LLM在给定任务中的表现。在实验中，目标是通过使用更强大的模型（例如gpt-4o）生成的反馈来提高较弱且成本较低的模型（例如gpt-3.5-turbo-0125）的性能。如下是使用TextGrad进行提示优化的简单实现代码。
    
    
    # Initialize the system prompt
    system_prompt = tg. Variable ("You are a helpful language model . Think step by step .",
    requires_grad = True ,
    role_description =" system prompt to the language model ")
    
    # Set up the model object ’ parameterized by ’ the prompt .
    model = tg. BlackboxLLM ( system_prompt = system_prompt )
    
    # Optimize the system prompt
    optimizer = tg. TextualGradientDescent ( parameters =[ system_prompt ])
    
    for iteration in range ( max_iterations ):
        batch_x , batch_y = next ( train_loader )
        optimizer . zero_grad ()
        # Do the forward pass
        responses = model ( batch_x )
        losses = [ loss_fn ( response , y ) for ( response , y) in zip ( responses , batch_y ) ]
        total_loss = tg. sum( losses )
        # Perform the backward pass and compute gradients
        total_loss . backward ()
        # Update the system prompt
        optimizer . step ()

如代码中所示，在小批量随机梯度下降设置中使用TextGrad。在每次迭代中，使用一些训练示例来运行方程16中的前向传递。上面的伪代码和简短实现展示了这一过程。  
**任务：** 在多个数据集中探索提示优化，包括Big Bench Hard中的两个标准推理任务（对象计数和单词排序）（随机分为50/100/100训练/验证/测试样本），GSM8k小学数学问题解决（使用[DSPy](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)的训练/验证/测试分割）。对于GSM8k和对象计数，使用基于字符串的精确匹配指标来量化准确性（即，响应中提供的最终数字是否与标准答案相同）。对于单词排序，使用LLM比较响应和标准答案。  
**方法：** 探索使用gpt-4o在反向传播期间提供反馈以提高gpt-3.5-turbo-0125的性能。  
基线：两个主要基线，（1）零样本链思维（CoT），我将所有提示初始化为零样本CoT提示，其中模型被指示“逐步思考”以在给出答案之前解释其推理过程。（2）DSPy是一个先进的语言模型编程和提示优化框架，将其作为参考基线。实例化DSPy的BootstrappedFewShotRandomSearch（BFSR）优化器，使用10个候选程序和8个少样本示例。  
**结果** ：在三个任务中，TEXTGRAD显著提高了零样本提示的性能。对于单词排序和GSM8k，它的表现与DSPy相似，而对于对象计数，它比DSPy提高了7%。  
表3：推理任务的提示优化  


## **总结**

在提示优化领域，作者们参考并借鉴了DSPy和[ProTeGi](https://zhida.zhihu.com/search?content_id=250861008&content_type=Article&match_order=1&q=ProTeGi&zhida_source=entity)。DSPy开创了将复杂LLM系统视为具有潜在多层的程序的想法，并提出了以编程方式构建和优化它们的方法。该框架改善了LLM在各种问答、推理和提示优化任务中的性能。其次，Prompt Optimization with Textual Gradients（ProTeGi）在提示优化的背景下定义了文本梯度，其中梯度是LLM给出的关于任务中出现的错误的自然语言反馈。虽然ProTeGi建立在文本梯度的类比上，但作者们将这一类比更广泛地扩展到自动微分，并远远超越了提示优化任务。特别是，DSPy和ProTeGi都专注于提示优化，而TextGrad的显著进步在于其是针对多样化应用开展优化，是针对实例的优化。

TextGrad基于三个关键原则构建：

  * 通用的高性能框架，不是为特定应用领域量身定制的。
  * 易于使用，类似于PyTorch抽象概念，便于知识迁移。
  * 完全开源。通过TextGrad，作者们在代码优化和博士级问题解答、优化提示方面取得了最先进的成果，并在科学应用（如开发分子和优化治疗计划）中提供了概念验证结果。



随着AI范式从训练单个模型转向优化涉及多个交互LLM组件和工具的复合系统，因此，需要新一代的自动化优化器。TextGrad将LLM的推理能力与反向传播的可分解效率结合起来，创建了一个优化AI系统的通用框架。后续，将会把TextGrad扩展到更多领域，并继续探索集成更多的技术，如更多的组件、方差减少技术、自适应梯度等，来提升框架的稳定性。

## 相关链接

  1. Mert Yuksekgonul, Federico Bianchi, Joseph Boen, Sheng Liu, Zhi Huang, Carlos Guestrin, James Zou. TextGrad: Automatic "Differentiation" via Text. arXiv preprint arXiv:2406.07496
  2. [https://github.com/zou-group/TextGrad](https://link.zhihu.com/?target=https%3A//github.com/zou-group/TextGrad)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 下一篇：[【Prompt洞察】03-ell：将提示视为函数的轻量级提示工程库](https://zhuanlan.zhihu.com/p/9941505513)
