# 【PromptWizard】04-PromptWizard和DSPy上手实操案例&实验数据对比

原文链接：https://zhuanlan.zhihu.com/p/27861853778

---

​

目录

## 背景

PrompWizard和[DSPy](https://zhida.zhihu.com/search?content_id=254595027&content_type=Article&match_order=1&q=DSPy&zhida_source=entity)分别是来自微软和斯坦福大学的优秀提示词优化项目，[PromptWizard](https://zhida.zhihu.com/search?content_id=254595027&content_type=Article&match_order=1&q=PromptWizard&zhida_source=entity)利用自我进化、自我适应的机制，通过反馈驱动的批评和综合过程，在探索和开发之间实现了有效平衡，迭代地优化提示指令和上下文示例，生成可读、特定任务的提示。DSPy创建了提示词优化的新范式，将提示词优化的步骤分离并抽象为具体的代码实现，以一种编程的方式完成提示词的自动优化，同时提供多种优化器满足提示词参数的优化，和大模型参数的优化。

本文将以实验的形式，在[GSM8K](https://zhida.zhihu.com/search?content_id=254595027&content_type=Article&match_order=1&q=GSM8K&zhida_source=entity)数据集上设计实验，分别测试和对比PromptWizard和DSPy的对于Prompt优化的数据表现。

## 1 实验设置

### 1.1 GSM8K数据集介绍

GSM8K（Grade School Math 8K）是一个由OpenAI发布的高质量小学数学应用题数据集，包含8500个语言多样化的数学问题。该数据集有如下特点： 

  * 问题难度适中：GSM8K中的问题均为小学数学应用题，涉及基本的算术运算（加、减、乘、除），需要2到8步的推理和计算。 
  * 语言多样性：问题的语言表达丰富多样，避免了单一模板，增加了模型对语言和数学概念的理解难度。 
  * 提供解决方案：每个问题都配有自然语言形式的详细解题步骤，而不是纯数学表达式。 


    
    
    {
        "question": "Weng earns $12 an hour for babysitting. Yesterday, she just did 50 minutes of babysitting. How much did she earn?",
        "answer": "Weng earns 12/60 = $<<12/60=0.2>>0.2 per minute.\nWorking 50 minutes, she earned 0.2 x 50 = $<<0.2*50=10>>10.\n# # # #  10",
        "final_answer": "10"
      }

### 1.2 评价指标

本文使用GSM8K测试集的前150个样本，逐个请求大模型获取结果，并计算每个提示词得到预测的准确率。

对于每个预测样本，与答案完全相同则认为预测正确，因此，准确率 = 正确数量/样本总数。

## 2 实验过程

本文分别对两个框架分别设计了3个实验:

  1. **获取基线** ：即通过构造普通网络请求的方式，将测试集中的问题和提示词直接发送到大模型服务器，查看结果。
  2. **使用框架优化提示词(不引入示例)** ：即使用DSPy或PromptWizard框架中的预测模块，对测试集的问题进行预测，但是预测过程中不引入示例数据集。
  3. **使用框架优化提示词(引入示例)** ：即加入训练示例数据集，使用DSPy或PromptWizard框架各自的优化方式对提示词进行优化，最终用各自框架的预测模块对测试集进行预测。



其中对于不使用框架的实验，直接请求大模型评估即可，因此与两框架无关。

### 2.1 获取基线

本文通过直接发送网络请求的方式，对测试集进行测试，获取基线结果，不涉及DSPy和PromptWizard框架代码，这里使用[deepseek](https://zhida.zhihu.com/search?content_id=254595027&content_type=Article&match_order=1&q=deepseek&zhida_source=entity)的chat模型进行请求，并使用openai包进行实现，代码如下。
    
    
    def extract_number_from_boxed(text):
        #  定义正则表达式
        pattern = r"boxed\{(\d+)\}"
        #  使用 re.search 查找匹配项
        match = re.search(pattern, text)
        if match:
            #  提取捕获组中的数字
            number = match.group(1)
            return int(number)  #  返回整数
        else:
            return None  #  如果没有匹配到，返回 None
    
    #  加载 JSON 文件
    def load_json_file(file_path: str):
        ...
    
    #  调用 OpenAI API 获取模型回答
    def get_model_answer(question):
        response = client.chat.completions.create(...)
        return response.choices[0].message.content.strip()
    
    #  计算预测成功率
    def calculate_success_rate(data):
        ...
        return success_rate
    
    #  主函数
    def main():
        #  替换为你的 JSON 文件路径
        json_file_path = "./data/test.jsonl"
        #  加载数据
        data = load_json_file(json_file_path)
        #  计算预测成功率
        success_rate = calculate_success_rate(data)
    
    if __name__ == "__main__":
        main()

由于大模型会自动的输出很多推理的内容，实际上输出的结果和要求的内容是不一致的，因此如果按照实际输出与数据集对比，则准确率机几乎为0，经过观察返回内容，deepseek返回的最终结果是包含在特定格式中的，所以定义了`extract_number_from_boxed`函数提取返回结果中的数值，最终处理后获得的实际准确率为81.2%。

### 2.2 使用框架优化提示词(不引入示例)

**2.2.1 PromptWizard**

PromptWizard提供了在没有示例时的优化功能，首先实例化GluePromptOpt类，该类为PromptWizard的核心类，实现了优化提示词的主要功能，由于此场景不使用数据集，因此 `dataset_jsonl`和`data_processor`配置为空。
    
    
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl=None,
                       data_processor=None)

通过调用`get_best_prompt`函数，设置指定参数，产生提示词示例。
    
    
    best_prompt, expert_profile = gp.get_best_prompt(use_examples=False,run_without_train_examples=True,generate_synthetic_examples=False)

查看结果，可以看到输出了多个优化的提示词和对应的专家角色描述。
    
    
    Variations 1:
    Expert Profile:
    （变化点：生成角色）
    You are a mathematician with a strong background in various fields of mathematics, including algebra, calculus, geometry, and statistics. You have a deep understanding of mathematical theories and principles, and you are skilled at solving complex problems with precision and clarity. Your expertise allows you to approach mathematical problems methodically, breaking them down into manageable steps and applying appropriate techniques to find solutions. You are familiar with both theoretical and applied mathematics, and you can explain your reasoning and solutions in a clear and concise manner. Your ability to solve mathematical problems efficiently and accurately makes you an invaluable resource for anyone seeking help with mathematics.:
    Prompt:
    You are a mathematics expert. You will be given a mathematics problem which you need to solve
    Lets think step by step.
    
    （变化点：答案格式）
    For each question present the reasoning followed by the correct answer.
    （变化点：生成关键词）
    Keywords: mathematics, problem-solving, step-by-step, logical reasoning, expert
    _______________________________________________________________________
    
    Variations 2:
    Expert Profile:
    You are a mathematician with a strong background in various fields of mathematics, including algebra, calculus, geometry, and statistics. You have a deep understanding of mathematical theories and principles, and you are skilled at solving complex problems with precision and clarity. Your expertise allows you to approach mathematical problems methodically, breaking them down into manageable steps and applying appropriate techniques to find solutions. You are familiar with both theoretical and applied mathematics, and you can explain your reasoning and solutions in a clear and concise manner. Your ability to solve mathematical problems efficiently and accurately makes you an invaluable resource for anyone seeking help with mathematics.:
    Prompt:
    Let's break this problem down step by step and devise an experiment to help solve it.
    
    
    For each question present the reasoning followed by the correct answer.
    Keywords: mathematics, problem-solving, step-by-step, logical reasoning, expert
    
    ...

从日志文件可以看出PromptWizard自动的产生多种类型的提示词，每部分提示词都包含了`Expert Profile`和`Prompt`两部分，`Expert Profile`中描述了角色背景信息以及Prompt中描述了指令、结果格式化方式、以及关键词三部分，比手工编写的提示词描述的更充分，为了验证新生成提示词的效果，我们利用PromptWizard，在GSM8K数据集的测试集上进行预测，这里就选取`Variations 1`输出的提示词内容进行验证(注意，当前场景PromptWizard只赋值生成了提示词，但没有选择最佳提示词，因此需要手动复制对应的Expert Profile和 Prompt)。
    
    
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl=None,
                       data_processor=GSM8k())
    gp.BEST_PROMPT = """
    You are a mathematics expert. You will be given a mathematics problem which you need to solve
    Lets think step by step.
    
    
    For each question present the reasoning followed by the correct answer.
    Keywords: mathematics, problem-solving, step-by-step, logical reasoning, expert
    """
    gp.EXPERT_PROFILE = """
    You are a mathematician with a strong background in various fields of mathematics, including algebra, calculus, geometry, and statistics. You have a deep understanding of mathematical theories and principles, and you are skilled at solving complex problems with precision and clarity. Your expertise allows you to approach mathematical problems methodically, breaking them down into manageable steps and applying appropriate techniques to find solutions. You are familiar with both theoretical and applied mathematics, and you can explain your reasoning and solutions in a clear and concise manner. Your ability to solve mathematical problems efficiently and accurately makes you an invaluable resource for anyone seeking help with mathematics."
    """
    test_score = gp.evaluate(test_dataset_jsonl=os.path.join("data", "test.jsonl"))

在log文件中查看准确率计算结果，在150个GSM8K测试集中答对了124个，准确率82.6%。
    
    
    {"accuracy": "1/1 : 100%", "predicted": "18", "actual": "18"} ... {"accuracy": "24/31 : 7741935483870968%", "predicted": "109", "actual": "109"} ... {"accuracy": "102/126 : 8095238095238095%", "predicted": "10", "actual": "10"} ... {"accuracy": "124/150 : 8266666666666667%", "predicted": "16", "actual": "16"}

**2.2.2 DSPy**

与PromptWizard不同，DSPy不是直接优化提示词，而是将预测模块封装成了`[dspy](https://zhida.zhihu.com/search?content_id=254595027&content_type=Article&match_order=1&q=dspy&zhida_source=entity).Module`，本文使用了`dspy.[ChainOfThought](https://zhida.zhihu.com/search?content_id=254595027&content_type=Article&match_order=1&q=ChainOfThought&zhida_source=entity)`模块进行测试，由于DSPy需要有示例才可以优化，因此没有设计DSPy优化相关代码，直接使用dspy的CoT模块对测试集进行预测。
    
    
    import dspy
    import httpx
    
    from dspy.datasets.gsm8k import GSM8K, gsm8k_metric
    from dspy.teleprompt import BootstrapFewShot
    from dspy.evaluate import Evaluate
    
    
    PROMPT_SWITCH = True
    
    if __name__ == '__main__':
    
        #   定义并设置大模型
        model_name = 'deepseek-chat'
        lm = dspy.OpenAI(model=model_name, base_url="https://api.deepseek.com", api_key="", http_client=httpx.Client(verify=False), model_type="chat")
        dspy.settings.configure(lm=lm)
    
        gms8k = GSM8K()
    
        trainset, test_set = gms8k.train, gms8k.test
    
        print(f"# #  创建并设置大模型 {model_name} # # ")
        print(f"\n# #  对模板和参数进行调优 - start # # \n")
    
        class CoT(dspy.Module):
            def __init__(self):
                super().__init__()
                self.prog = dspy.ChainOfThought("question -> answer")
    
            def forward(self, question):
                return self.prog(question=question)
    
    
        cot = CoT()
        evaluate = Evaluate(devset=test_set[:150], metric=gsm8k_metric, num_threads=4, display_progress=True, display_table=0)
        evaluate(cot)

结果为59.3%
    
    
    Average Metric: 89.00 / 150 (59.3%): 100%|██████████| 150/150 [00:00<00:00, 670.17it/s]

### 2.3 使用框架优化提示词(引入示例)

**2.3.1 PromptWizard**  
  
本次实验在上一轮代码基础上，引入训练数据集，在初始化`GluePromptOpt`时，输入数据集路径参数以及数据集处理器类，在调用`get_best_prompt`函数时，将`user_exmamples`参数设置为True即可。
    
    
    import sys
    sys.path.insert(0, "../../")
    import promptwizard
    from promptwizard.glue.promptopt.instantiate import GluePromptOpt
    from promptwizard.glue.promptopt.techniques.common_logic import DatasetSpecificProcessing
    from promptwizard.glue.common.utils.file import save_jsonlist
    from typing import Any
    from tqdm import tqdm
    from re import compile, findall
    import os
    from datasets import load_dataset
    import yaml
    from dotenv import load_dotenv
    load_dotenv(override = True)
    
    path_to_config = "configs"
    promptopt_config_path = os.path.join(path_to_config, "promptopt_config.yaml")
    setup_config_path = os.path.join(path_to_config, "setup_config.yaml")
    
    #  GSM8K 数据集处理类
    class GSM8k(DatasetSpecificProcessing):
    
        def dataset_to_jsonl(self, dataset_jsonl: str, **kwargs: Any) -> None:
            def extract_answer_from_output(completion):
                #  Your functions for metrics and prompt building
                ans_re = compile(r"# # # #  (\-?[0-9\.\,]+)")
                self.INVALID_ANS = "[invalid]"
    
                match = ans_re.search(completion)
                if match:
                    match_str = match.group(1).strip()
                    match_str = match_str.replace(",", "")
                    return match_str
                else:
                    return self.INVALID_ANS
    
            examples_set = []
    
            for _, sample in tqdm(enumerate(kwargs["dataset"]), desc="Evaluating samples"):
                example = {
                    DatasetSpecificProcessing.QUESTION_LITERAL: sample['question'],
                    DatasetSpecificProcessing.ANSWER_WITH_REASON_LITERAL: sample['answer'],
                    DatasetSpecificProcessing.FINAL_ANSWER_LITERAL: extract_answer_from_output(sample["answer"])
                }
                examples_set.append(example)
    
            save_jsonlist(dataset_jsonl, examples_set, "w")
    
        def extract_final_answer(self, answer: str):
    
            if not answer:
                return self.INVALID_ANS
    
            model_pred = answer.lower()
            preds = model_pred.split(self.ANSWER_START.lower())
            answer_flag = True if len(preds) > 1 else False
    
            pred = preds[-1].replace(",", "")
            pred = [s for s in findall(r'-?\d+\.?\d*', pred)]
    
            if len(pred) == 0:
                return self.INVALID_ANS
    
            if answer_flag:
                #  choose the first element in list
                pred = pred[0]
            else:
                #  choose the last element in list
                pred = pred[-1]
    
            #  (For arithmetic tasks) if a word ends with period, it will be omitted ...
            if pred[-1] == ".":
                pred = pred[:-1]
            return pred
    
    #  优化代码
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl = os.path.join("data", "train.jsonl"),
                       data_processor = GSM8k())
    
    best_prompt, expert_profile = gp.get_best_prompt(use_examples=True,run_without_train_examples=False,generate_synthetic_examples=False)
    
    test_score = gp.evaluate(test_dataset_jsonl=os.path.join("data", "test.jsonl"))

优化参数为：
    
    
    "task_description": "You are a mathematics expert. You will be given a mathematics problem which you need to solve",
    "base_instruction": "Lets think step by step.",
    "mutation_rounds": 2,
    "few_shot_count": 5,
    "generate_reasoning": True,
    "mutate_refine_iterations" : 3,
    "seen_set_size":20

查看提示词优化结果。
    
    
    Expert Identity: You are a mathematics expert with a strong background in various fields of mathematics, including algebra, calculus, geometry, and statistics. You have a deep understanding of mathematical theories and principles, and you are skilled at solving complex problems with precision and clarity. Your expertise allows you to break down intricate problems into manageable steps, making it easier for others to follow your reasoning. You are familiar with a wide range of mathematical techniques and tools, and you can apply them effectively to find solutions. Whether the problem involves solving equations, proving theorems, or analyzing data, you can provide a clear, accurate, and well-explained solution. Your ability to communicate complex mathematical concepts in an understandable way makes you an invaluable resource for anyone seeking help with mathematics.
    
    Final best prompt: 
    （变化点：优化生成指令）
    
    You are a mathematics expert. Your task is to solve a given mathematics problem accurately and provide a clear, detailed explanation of your solution process. Follow these steps to ensure a comprehensive and well-structured solution:
    
    1. **Understand the Problem**: Carefully read and comprehend the problem statement. Identify the key components and what is being asked.
    
    2. **Identify Components**: Break down the problem into its fundamental components, such as variables, constants, and relevant quantities (e.g., base pay, overtime pay, distances, speeds, etc.).
    
    3. **Apply Relevant Principles**: Use appropriate mathematical principles, formulas, and methods to solve the problem step by step.
    
    4. **Logical Reasoning**: Employ logical reasoning to explain each step of your solution process. Ensure that each step follows logically from the previous one.
    
    5. **Detailed Explanations**: Provide detailed explanations for each step to ensure clarity and understanding. Include intermediate results and how they contribute to the final solution.
    
    6. **Explicit Calculation Steps**: Show each calculation step in detail, including intermediate results. Use proper mathematical notation and symbols.
    
    7. **Verify Each Step**: Recheck each intermediate step of your calculation to verify the correctness of the final answer. Ensure that all arithmetic and algebraic operations are accurate.
    
    8. **Combine Results**: Clearly combine different components of the problem (e.g., base pay and overtime pay) before arriving at the final answer.
    
    9. **Simplify and Notate**: Simplify the final answer where possible, and use proper mathematical notation and symbols.
    
    10. **Mark the Final Answer**: Clearly mark the final answer within <ANS_START> and <ANS_END> tags.
    
    Ensure that your approach is tailored to the specific type of mathematical problem being solved, whether it involves arithmetic, algebra, geometry, calculus, or any other area of mathematics. Present the solutions in a clear and organized manner.
    
    **Additional Guidelines:**
    - **Contextual Understanding**: Pay close attention to the context of the problem to ensure that all relationships and quantities are correctly interpreted.
    - **Correct Application of Arithmetic Operations**: Double-check that all arithmetic operations are applied correctly and align with the problem's requirements.
    - **Verification of Final Answer**: Dedicate a step to verify the final answer by rechecking all intermediate steps and ensuring they logically lead to the correct final result.
    - **Clarity in Marking Final Answer**: Use the <ANS_START> and <ANS_END> tags to clearly mark the final answer.
    
    By following these steps and additional guidelines, you will ensure that the solution is accurate, well-explained, and clearly presented.
    
    （变化点：生成示例）
    [Question] Bella bought stamps at the post office. Some of the stamps had a snowflake design, some had a truck design, and some had a rose design. Bella bought 11 snowflake stamps. She bought 9 more truck stamps than snowflake stamps, and 13 fewer rose stamps than truck stamps. How many stamps did Bella buy in all?
    [Answer] 1. **Understand the Problem**: Bella bought three types of stamps: snowflake, truck, and rose. We need to determine the total number of stamps she bought, given the relationships between the quantities of each type.
    
    2. **Identify Components**:
       - Number of snowflake stamps: 11.
       - Number of truck stamps: 9 more than the number of snowflake stamps.
       - Number of rose stamps: 13 fewer than the number of truck stamps.
    
    3. **Apply Relevant Principles**: Use basic arithmetic operations to find the quantities of truck and rose stamps, and then sum all the quantities to find the total number of stamps.
    
    4. **Logical Reasoning**:
       - Number of snowflake stamps: 11.
       - Number of truck stamps: 11 (snowflake stamps) + 9 = 20.
       - Number of rose stamps: 20 (truck stamps) - 13 = 7.
    
    5. **Detailed Explanations**:
       - Calculate the number of truck stamps: 11 (snowflake stamps) + 9 = 20.
       - Calculate the number of rose stamps: 20 (truck stamps) - 13 = 7.
       - Calculate the total number of stamps: 11 (snowflake) + 20 (truck) + 7 (rose) = 38.
    
    6. **Explicit Calculation Steps**:
       - Truck stamps: 11 + 9 = $<11+9=20>20.
       - Rose stamps: 20 - 13 = $<20-13=7>7.
       - Total stamps: 11 + 20 + 7 = $<11+20+7=38>38.
    
    7. **Verify Each Step**: Recheck each calculation step to ensure correctness:
       - Truck stamps: 11 + 9 = 20.
       - Rose stamps: 20 - 13 = 7.
       - Total stamps: 11 + 20 + 7 = 38.
    
    8. **Combine Results**: Combine the number of each type of stamp correctly to find the total number of stamps.
    
    9. **Simplify and Notate**: The final answer is already simplified.
    
    10. **Mark the Final Answer**: <ANS_START>38<ANS_END>
    
    By following these steps, we ensure that the solution is accurate, well-explained, and clearly presented. <ANS_START>38<ANS_END>
    
    [Question] It takes Roque two hours to walk to work and one hour to ride his bike to work. Roque walks to and from work three times a week and rides his bike to and from work twice a week. How many hours in total does he take to get to and from work a week with walking and biking?
    [Answer] 1. **Understand the Problem**: Roque has two modes of transportation to work: walking and biking. We need to calculate the total time he spends traveling to and from work in a week, considering the different times and frequencies for each mode.
    
    2. **Identify Components**:
       - Time to walk to work: 2 hours (one way).
       - Time to bike to work: 1 hour (one way).
       - Frequency of walking: 3 times a week (to and from work).
       - Frequency of biking: 2 times a week (to and from work).
    
    3. **Apply Relevant Principles**: Use basic arithmetic to calculate the total time spent walking and biking separately, then sum these times to get the total weekly travel time.
    
    4. **Logical Reasoning**:
       - Calculate the total walking time for a week:
         - One round trip (to and from work) by walking takes 2 hours (to work) + 2 hours (from work) = 4 hours.
         - Roque walks to and from work 3 times a week, so the total walking time is 4 hours per round trip * 3 round trips = 12 hours.
       - Calculate the total biking time for a week:
         - One round trip (to and from work) by biking takes 1 hour (to work) + 1 hour (from work) = 2 hours.
         - Roque bikes to and from work 2 times a week, so the total biking time is 2 hours per round trip * 2 round trips = 4 hours.
    
    5. **Detailed Explanations**:
       - Walking time calculation:
         - One round trip walking: 2 hours (to work) + 2 hours (from work) = 4 hours.
         - Total walking time for the week: 4 hours per round trip * 3 round trips = 12 hours.
       - Biking time calculation:
         - One round trip biking: 1 hour (to work) + 1 hour (from work) = 2 hours.
         - Total biking time for the week: 2 hours per round trip * 2 round trips = 4 hours.
       - Combine the total walking and biking times to get the total weekly travel time:
         - Total weekly travel time: 12 hours (walking) + 4 hours (biking) = 16 hours.
    
    6. **Explicit Calculation Steps**:
       - Walking time: 2 hours (one way) * 2 (round trip) * 3 (times a week) = $<2*2*3=12>12 hours.
       - Biking time: 1 hour (one way) * 2 (round trip) * 2 (times a week) = $<1*2*2=4>4 hours.
       - Total time: 12 hours (walking) + 4 hours (biking) = $<12+4=16>16 hours.
    
    7. **Verify Each Step**: Recheck each calculation step to ensure correctness. Confirm that the arithmetic operations and logic used are accurate.
    
    8. **Combine Results**: Combine the total walking and biking times correctly to ensure the final answer is accurate.
    
    9. **Simplify and Notate**: The final answer is already simplified and clearly presented.
    
    10. **Mark the Final Answer**: <ANS_START>16<ANS_END>
    
    By following these steps, we ensure that the solution is accurate, well-explained, and clearly presented. <ANS_START>16<ANS_END>
    
    [Question] Betty is saving money for a new wallet which costs $100. Betty has only half of the money she needs. Her parents decided to give her $15 for that purpose, and her grandparents twice as much as her parents. How much more money does Betty need to buy the wallet?
    [Answer] 1. **Understand the Problem**: Betty is saving money for a wallet that costs $100. She currently has half of the money she needs. Her parents and grandparents are contributing additional amounts to help her reach her goal. We need to determine how much more money Betty needs to buy the wallet.
    
    2. **Identify Components**:
       - Total cost of the wallet: $100.
       - Amount Betty currently has: half of $100.
       - Contribution from parents: $15.
       - Contribution from grandparents: twice the amount given by parents.
    
    3. **Apply Relevant Principles**: Use basic arithmetic to calculate the total amount of money Betty will have after receiving contributions from her parents and grandparents, and then determine how much more she needs to reach $100.
    
    4. **Logical Reasoning**:
       - Calculate the amount Betty currently has: $100 / 2 = $50.
       - Calculate the contribution from grandparents: 2 * $15 = $30.
       - Calculate the total amount of money Betty will have: $50 (current amount) + $15 (parents' contribution) + $30 (grandparents' contribution).
    
    5. **Detailed Explanations**:
       - Betty currently has $50 because half of $100 is $50.
       - Her parents give her $15.
       - Her grandparents give her twice the amount her parents give, which is 2 * $15 = $30.
       - The total amount of money Betty will have is $50 (current amount) + $15 (parents' contribution) + $30 (grandparents' contribution) = $95.
    
    6. **Explicit Calculation Steps**:
       - Current amount: $100 / 2 = $<100/2=50>50.
       - Grandparents' contribution: 2 * $15 = $<2*15=30>30.
       - Total amount: $50 + $15 + $30 = $<50+15+30=95>95.
    
    7. **Verify Each Step**: Recheck each calculation step to ensure correctness.
       - Current amount: $100 / 2 = $50.
       - Grandparents' contribution: 2 * $15 = $30.
       - Total amount: $50 + $15 + $30 = $95.
    
    8. **Combine Results**: Combine the total amount of money Betty will have correctly.
       - Total amount: $50 (current amount) + $15 (parents' contribution) + $30 (grandparents' contribution) = $95.
    
    9. **Simplify and Notate**: The final answer is already simplified.
    
    10. **Mark the Final Answer**: 
       - Amount Betty needs to buy the wallet: $100 - $95 = $<100-95=5>5.
    
    <ANS_START>5<ANS_END> <ANS_START>5<ANS_END>
    
    [Question] A rectangle has a length of 10 cm and a width of 5 cm. What is the area and perimeter of the rectangle?
    [Answer] 1. **Understand the Problem**: We need to find both the area and the perimeter of a rectangle given its length and width.
    
    2. **Identify Components**: 
       - Length of the rectangle (L) = 10 cm
       - Width of the rectangle (W) = 5 cm
    
    3. **Apply Relevant Principles**: 
       - The formula for the area of a rectangle is \( \text{Area} = \text{Length} \times \text{Width} \).
       - The formula for the perimeter of a rectangle is \( \text{Perimeter} = 2 \times (\text{Length} + \text{Width}) \).
    
    4. **Logical Reasoning**:
       - To find the area, multiply the length by the width.
       - To find the perimeter, add the length and the width, then multiply the result by 2.
    
    5. **Detailed Explanations**:
       - Calculate the area: \( \text{Area} = 10 \, \text{cm} \times 5 \, \text{cm} \).
       - Calculate the perimeter: \( \text{Perimeter} = 2 \times (10 \, \text{cm} + 5 \, \text{cm}) \).
    
    6. **Explicit Calculation Steps**:
       - Area: \( 10 \times 5 = 50 \, \text{cm}^2 \).
       - Perimeter: \( 2 \times (10 + 5) = 2 \times 15 = 30 \, \text{cm} \).
    
    7. **Verify Each Step**: 
       - Recheck the area calculation: \( 10 \times 5 = 50 \, \text{cm}^2 \).
       - Recheck the perimeter calculation: \( 2 \times 15 = 30 \, \text{cm} \).
    
    8. **Combine Results**: 
       - The area of the rectangle is \( 50 \, \text{cm}^2 \).
       - The perimeter of the rectangle is \( 30 \, \text{cm} \).
    
    9. **Simplify and Notate**: 
       - The final answers are already simplified.
    
    10. **Mark the Final Answer**: 
       - Area: <ANS_START>50 \, \text{cm}^2<ANS_END>
       - Perimeter: <ANS_START>30 \, \text{cm}<ANS_END>
    
    By following these steps, we ensure that the solution is accurate, well-explained, and clearly presented. <ANS_START>50<ANS_END>
    
    
    ...
    （变化点：答案格式）
    
    For each question present the reasoning followed by the correct answer.

最终得到的结果为存到`evaluation`目录中,在150个GSM8K测试集中答对了143个，准确率95.3%。
    
    
    {"accuracy": "1/1 : 100%", "predicted": "18", "actual": "18"}
    ...
    {"accuracy": "60/63 : 9523809523809523%", "predicted": "25000", "actual": "25000"}
    ...
    {"accuracy": "109/114 : 956140350877193%", "predicted": "25", "actual": "25"}
    ...
    {"accuracy": "143/150 : 9533333333333334%", "predicted": "16", "actual": "16"}

**2.3.2 DSPy**

本轮实验在上一次实验代码中增加了优化器模块，这里使用了`BootstrapFewShot`优化器，初始化优化器时，设置好评价指标、示例个数、迭代次数等参数，然后调用compile函数对CoT模块进行优化。
    
    
    import dspy
    import httpx
    from dspy.datasets.gsm8k import GSM8K, gsm8k_metric
    from dspy.teleprompt import BootstrapFewShot
    from dspy.evaluate import Evaluate
    PROMPT_SWITCH = True
    if __name__ == '__main__':
        #   定义并设置大模型
        model_name = ''
        lm = dspy.OpenAI(model=model_name, base_url="", api_key="", http_client=httpx.Client(verify=False), model_type="chat")
        dspy.settings.configure(lm=lm)
        #  加载数据集
        gms8k = GSM8K()
        trainset, test_set = gms8k.train, gms8k.test
        #  实例化Module
        class CoT(dspy.Module):
            def __init__(self):
                super().__init__()
                self.prog = dspy.ChainOfThought("question -> answer")
    
            def forward(self, question):
                return self.prog(question=question)
        cot = CoT()
        #  运行评估代码
        evaluate = Evaluate(devset=test_set[:150], metric=gsm8k_metric, num_threads=4, display_progress=True, display_table=0)
        #  配置优化参数
        config = dict(max_rounds=3)
        #  初始化优化器
        teleprompter = BootstrapFewShot(
            metric=gsm8k_metric,
            max_bootstrapped_demos=8,
            max_labeled_demos=8,
            max_rounds=10,
        )
        #  开始优化
        optimized_cot = teleprompter.compile(cot, trainset=trainset)
        #  运行评估代码
        evaluate(cot)
        #  保存优化结果
        optimized_cot.save("./test.json")

优化产生的提示词如下：
    
    
    Given the fields `question`, produce the fields `answer`.
    
    ---
    
    Follow the following format.
    
    Question: ${question}
    Reasoning: Let's think step by step in order to ${produce the answer}. We ...
    Answer: ${answer}
    
    ---
    
    Question: Andrew works in a company that provides a generous vacation allotment: for every 10 days worked, you get 1 vacation day. If last year Andrew worked 300 days and took 5 days off in March and twice as many in September, how many more vacation days can Andrew still take?
    Answer: 15
    
    ---
    
    Question: Frank is making hamburgers and he wants to sell them to make $50. Frank is selling each hamburger for $5 and 2 people purchased 4 and another 2 customers purchased 2 hamburgers. How many more hamburgers does Frank need to sell to make $50?
    Answer: 4
    
    ---
    
    Question: There are 40 students in a class. If 1/10 are absent, 3/4 of the students who are present are in the classroom, and the rest are in the canteen, how many students are in the canteen?
    Answer: 9
    
    ---
    
    Question: Betty and Dora started making some cupcakes at the same time. Betty makes 10 cupcakes every hour and Dora makes 8 every hour. If Betty took a two-hour break, what is the difference between the number of cupcakes they made after 5 hours?
    Answer: 10
    
    ---
    
    Question: Daisy is climbing trees all around her neighborhood and starts to notice the number of branches and the height. The first tree she climbs is 50 feet tall and has 200 branches. The second tree she climbs is 40 feet tall and has 180 branches. The third tree she climbs is 60 feet tall and has 180 branches. The final tree she climbs is 34 feet tall and has 153 branches. On average, how many branches do these trees have per foot?
    Answer: 4
    
    ---
    
    Question: Zhang is twice as old as Li. Li is 12 years old. Zhang's brother Jung is 2 years older than Zhang. How old is Jung?
    Answer: 26
    
    ---
    
    Question: Jack is on the phone with a scammer who says the IRS will arrest Jack if he doesn't send them the codes from 6 $500 Best Buy gift cards and 9 $200 Walmart gift cards. After sending the codes for 1 Best Buy gift card and 2 Walmart gift cards, Jack wises up and hangs up. How many dollars' worth of gift cards can he still return?
    Answer: 3900
    
    ---
    
    Question: Matthew drinks 4 glasses of water per day. Each glass is 5 ounces. He decides to just buy a 35 ounces water bottle. How many times will he fill it each week?
    Answer: 4
    
    

  


测试集的结果如下：
    
    
    Average Metric: 133.00 / 150 (88.7%): 100%|██████████| 150/150 [03:39<00:00,  1.47s/it]
    2025/01/14 17:26:21 INFO dspy.evaluate.evaluate: Average Metric: 133 / 150 (88.7%)

## 3 实验结果

  


DSPy和PromptWizard实验结果对比

  1. 如图-1 所示，PromptWizard使用框架，但未使用示例时，优化的表现均好于直接请求大模型得到的结果，而DSPy低于直接请求的处理后的结果，可以推测DSPy对验证流程没有做较好的优化，导致没有处理好大模型实际输出中的内容。
  2. 如图-2 所示，增加示例数据集后，从优化结果看，PromptWizard在GSM8K上的正确率95%略优于DSPy的88%（注意，图1、图2使用的是相同的基线数据）。
  3. 从提示词内容看PromptWizard的提示词示例内容更丰富包含了推理的过程，DSPy的提示词内容结构较简单。
  4. 对于原始训练示例数据集，PromptWizard对原始示例有修改和优化，而DSPy没有对示例进行修改，使用的是原始数据集中的示例。



## 4 总结

  1. PromptWizard更专注于直接优化提示词，DSPy专注于通过Module整体的优化，间接实现提示词的优化，二者的设计思路不同。
  2. 在当前GSM8K数据集上，提示词的优化效果PromptWizard优于DSPy。
  3. DSPy还具备模型参数微调等能力，功能更丰富。

| PromptWizard| DSPy  
---|---|---  
优化目标| 优化提示词| 优化Module  
是否支持修改模型参数| 不支持| 支持  
优化器算法| CritiqueNRefine| BootStrap，MIPRO, COPRO, ...  
预测主体| 可直接调用大模型接口预测或调用自带的预测函数| 实例化Module预测  
使用方法| 配置文件+代码| 代码  
  
## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【PromptWizard技术洞察】03-PromptWizard源码解析](https://zhuanlan.zhihu.com/p/27109942600)
