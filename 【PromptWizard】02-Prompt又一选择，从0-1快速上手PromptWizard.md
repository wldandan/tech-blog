# 【PromptWizard】02-Prompt又一选择，从0-1快速上手PromptWizard

原文链接：https://zhuanlan.zhihu.com/p/26154065107

---

​

目录

[PromptWizard](https://zhida.zhihu.com/search?content_id=254253280&content_type=Article&match_order=1&q=PromptWizard&zhida_source=entity)为微软研究院提出的新型、完全自动化的[离散提示词优化框架](https://zhida.zhihu.com/search?content_id=254253280&content_type=Article&match_order=1&q=%E7%A6%BB%E6%95%A3%E6%8F%90%E7%A4%BA%E8%AF%8D%E4%BC%98%E5%8C%96%E6%A1%86%E6%9E%B6&zhida_source=entity)。PromptWizard利用自我进化、自我适应的机制，通过反馈驱动的批评和综合过程，在探索和开发之间实现了有效平衡，迭代地优化提示指令和上下文示例，生成可读、特定任务的提示。这种引导式方法系统地提高了提示质量，此外，成本分析显示API调用、[token使用](https://zhida.zhihu.com/search?content_id=254253280&content_type=Article&match_order=1&q=token%E4%BD%BF%E7%94%A8&zhida_source=entity)和总成本大幅减少。

本教程将以数学问题场景为例，循序渐进地介绍PromptWizard的基本功能、简单原理及使用场景。

## 准备工作

下载PromptWizard，并安装环境。
    
    
    git clone https://github.com/microsoft/PromptWizard
    cd PromptWizard
    
    pip install -e .

PromptWizard 简单易用，在完成环境安装之后，需要简单的设置环境文件和实验的配置文件，PromptWizard根据配置文件即可开始提示词的优化，配置文件。

1、.env 文件，用于配置大模型服务相关内容、如API KEY等。
    
    
    USE_OPENAI_API_KEY="False"
    
    OPENAI_API_KEY=""
    OPENAI_MODEL_NAME =""
    
    OPENAI_API_VERSION=""
    AZURE_OPENAI_ENDPOINT=""
    AZURE_OPENAI_DEPLOYMENT_NAME=""

2、setup_config.yaml，用于配置实验参数，如log目录名称、实验名称等，在每次运行优化后相关的内容将会根据本配置文件内容记录。
    
    
    assistant_llm:
      #  put the unique_model_id that you specified in llm_config.yaml
      prompt_opt: gpt-4o
    dir_info:
      #  Base directory for everything
      base_dir: logs
      log_dir_name: glue_logs
    experiment_name: gsm8k
    #  Many features are different for mode: online/offline. For eg
    #  1) Print of logs happens on console for offline mode
    #  2) LLM Queue gets instantiated only in online mode
    mode: offline
    #  Full length description of the experiment. This would be logged.
    description:

3、promptout_config.yaml，用于配置优化提示词过程中的参数，如使用的模型、优化轮数、任务描述等。
    
    
    #  Specify one or more prompt refinement technique to be used. If you specify more than one prompt refinement techniques,
    #  all these technique would run on same seed data. Result, iterations needed & cost incurred for each of these
    #  technique would be logged. And winning technique for each data instance and overall would be logged.
    
    #  Supported prompt refinement techniques: Basic, RecursiveEval, MedPrompt
    #  Uncomment techniques that you want to use
    # # # # # # # # # # # # # # # # # # # # # # # # # # # #  Critique Task Description Start # # # # # # # # # # # # # # # # # # # # # # # # # # # # 
    prompt_technique_name: "critique_n_refine"
    #  unique_model_id of model defined in llm_config.yaml
    unique_model_id: gpt-4o
    #  Number of iterations for conducting <mutation_rounds>  rounds of mutation of task description
    #  followed by refinement of instructions
    mutate_refine_iterations: 3
    #  Number of rounds of mutation to be performed when generating different styles
    mutation_rounds: 3
    #  Refine instruction post mutation
    refine_instruction: true
    #  Number of iterations for refining task description and in context examples for few-shot
    refine_task_eg_iterations: 3
    #  Number of variations of prompts to generate in given iteration
    style_variation: 5
    #  Number of questions to be asked to LLM in a single batch, during training step
    questions_batch_size: 1
    #  Number of batches of questions to correctly answered, for a prompt to be considered as performing good
    min_correct_count: 3
    #  Max number of mini-batches on which we should evaluate our prompt
    max_eval_batches: 6
    #  Number of top best performing prompts to be considered for next iterations
    top_n: 1
    #  Description of task. This will be fed to prompt
    task_description: "You are a mathematics expert. You will be given a mathematics problem which you need to solve"
    #  Base instruction, in line with your dataset. This will be fed to prompt
    base_instruction: "Lets think step by step."
    #  Instruction for specifying answer format
    answer_format: "For each question present the reasoning followed by the correct answer."
    #  Number of samples from dataset, set aside as training data. In every iteration we would be drawing
    #  `questions_batch_size` examples from training data with replacement.
    seen_set_size: 25
    #  Number of examples to be given for few shots
    few_shot_count: 5
    #  Number of synthetic training examples to be generated
    num_train_examples: 20
    #  Generate synthetic reasoning
    generate_reasoning: true
    #  Generate description of an expert which can solve the task at hand
    generate_expert_identity: true
    #  Generate keywords that describe the intent of the task
    generate_intent_keywords: false
    # # # # # # # # # # # # # # # # # # # # # # # # # # # #  Critique Task Description End # # # # # # # # # # # # # # # # # # # # # # # # # # # # 

4、准备[GSM8K 数据集](https://zhida.zhihu.com/search?content_id=254253280&content_type=Article&match_order=1&q=GSM8K+%E6%95%B0%E6%8D%AE%E9%9B%86&zhida_source=entity)并编写GSM8K数据对应的类，数据集将会存储在data目录中。
    
    
    from datasets import load_dataset
    from promptwizard.glue.promptopt.techniques.common_logic import DatasetSpecificProcessing
    
    if not os.path.exists("data"):
        os.mkdir("data")
    
    dataset = load_dataset("openai/gsm8k", "main")
    num_samples = 0
    gsm8k_processor = GSM8k()
    
    for dataset_type in ['train', 'test']:
        data_list = []
        for data in dataset[dataset_type]:
            data_list.append({"question": data['question'], "answer": data['answer']})
            if num_samples == 100 and dataset_type == 'train':  #  We sample only 100 train examples and use 25 out them for training randomly
                break
            num_samples += 1
        gsm8k_processor.dataset_to_jsonl("data/" + dataset_type + '.jsonl', dataset=data_list)
    
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
            #  用于access answer
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

## 迭代1-使用手工提示词（不使用PromptWizard优化，得到评估结果）

在不使用PromptWizard的情况下，通常需要手动编写提示词以及请求大模型的相关代码，如提示词为：
    
    
    系统提示词："You are a mathematics expert. You will be given a mathematics problem which you need to solve"
    用户提示词：" {question}. Lets think step by step."

未经封装调用大模型预测的代码为：
    
    
    import openai
    
    #  设置你的 OpenAI API 密钥
    openai.api_key = '你的API密钥'
    
    #  定义系统提示词和用户提示词
    system_prompt = "You are a mathematics expert. You will be given a mathematics problem which you need to solve"
    question = "1 + 1 = ?"
    user_base_prompt = "Lets think step by step."
    user_prompt = question + user_base_prompt
    #  构建消息列表
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ]
    
    #  发送请求
    response = openai.chat_completion.create(
        model="gpt-3.5-turbo",
        messages=messages
    )
    
    #  获取并打印模型的回答
    model_response = response.choices[0].message.content
    print(model_response)

## 迭代2-使用PromptWizard优化提示词，但不使用数据集

迭代1中介绍了使用手工提示词，而不使用PromptWizard优化，进行模型预测得到结果。接下来，我们介绍使用PromptWizard优化提示词但不使用数据集的场景。

此场景依然使用原有配置文件，首先实例化GluePromptOpt类，该类为PromptWizard的核心类，实现了优化提示词的主要功能，由于此场景不使用数据集，因此 dataset_jsonl 和 data_processor 配置为空。
    
    
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl=None,
                       data_processor=None)

通过调用get_best_prompt 函数，设置指定参数，产生提示词示例。
    
    
    best_prompt, expert_profile = gp.get_best_prompt(use_examples=False,run_without_train_examples=True,generate_synthetic_examples=False)

查看结果，可以看到输出了多个优化的提示词和对应的专家角色描述（耗时40s，因未产生大量迭代消耗token较少）。
    
    
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

那么，在迭代2的优化过程中，PromptWizard具体做了哪些工作呢？如下代码所示：
    
    
    def get_best_prompt(self, params: PromptOptimizationParams,use_examples=False,run_without_train_examples=False,generate_synthetic_examples=False) -> (str, Any):
        ...    
        for round_num in tqdm(range(1, params.mutate_refine_iterations+1), desc="Iterations completed: "):
            self.logger.info(f"{CommonLogsStr.LOG_SEPERATOR} + Starting iteration: {round_num} \n "
                            f"current_base_instruction: {current_base_instruction}")
            #  生成 mutation_round 个 提示词
            candidate_prompts = self.gen_different_styles(current_base_instruction,
                                                        params.task_description,
                                                        params.mutation_rounds+1,
                                                        params.style_variation)
            #  如果没有examples
            if run_without_train_examples:
                prompt_index = 1
                print("\nOptimization Finished...")
                print("\nPossible prompt variations:")
                #  这里为什么只循环了 mutation_rounds次
                for candidate in candidate_prompts[:params.mutation_rounds]:
                    final_best_prompt = self.prompt_pool.final_prompt.format(
                    instruction=candidate,
                    answer_format=params.answer_format,
                    few_shot_examples="")
                    expert_identity = self.prompt_pool.system_prompt
                    if params.generate_expert_identity:
                        expert_identity = self.generate_expert_identity(params.task_description)
    
                    # if params.generate_intent_keywords:
                    intent_keywords = self.generate_intent_keywords(params.task_description,
                                                                        params.base_instruction)
    
                    final_best_prompt += "Keywords: " + intent_keywords
                    print("_______________________________________________________________________")
                    print("\nVariations "+str(prompt_index)+":\nExpert Profile:\n"+expert_identity+":\nPrompt:\n"+final_best_prompt)
                    prompt_index += 1
                #  如果没有exmaple则不产生best提示词， 这里写的有点毛病，可能是他没写完
                return "",""
    
            ...

通过查看get_best_prompt`函数中关于`run_without_train_examples开关的逻辑可以看出，首先是迭代mutate_refine_iterations 次数，每次迭代都会通过self.gen_different_styles 函数生成不同多样性类型的提示词，存放到candidate_prompts变量中，循环candidate_prompts并执行四个操作:

  1. 使用self.prompt_pool.final_prompt 模板，初始化final_best_prompt。
  2. 利用 task_description 生成 expert_indentity 也就是对角色的描述。
  3. 利用task_description和base_instruction 生成 intent_keywords。
  4. 组装最终的final_best_prompt。



## 迭代3-使用PrmoptWizard，使用官方数据集

在迭代3中，我们在没有数据集的情况下，生成数据集，再利用生成的数据集自动优化提示词，本小节直接使用GSM8K训练集中的真实数据作为训练集，利用PromptWizard 进行提示词的自动优化。

首先修改 promptopt_config.yaml 文件， 和场景2中配置相同。
    
    
    "task_description": "You are a mathematics expert. You will be given a mathematics problem which you need to solve",
    "base_instruction": "Lets think step by step.",
    "mutation_rounds": 2,
    "few_shot_count": 5,
    "generate_reasoning": True,
    "mutate_refine_iterations" : 3,
    "seen_set_size":20

然后优化生成提示词，可以看出，迭代4和迭代3的优化参数完全一致，唯一的区别为使用的训练集不同。
    
    
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl = os.path.join("data", "train.jsonl"),
                       data_processor = gsm8k_processor)
    best_prompt, expert_profile = gp.get_best_prompt(use_examples=True,run_without_train_examples=False,generate_synthetic_examples=False)

查看优化结果（耗时566s）
    
    
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

此外，在迭代3的自动化执行过程中，best_prompt 是如何产生的呢？

图1

如图1所示，中间PromptWizard部分为生成提示词的主要逻辑，主要分为四步：

  1. 输入任务基本信息。
  2. 对指令的迭代优化，Iterative Optimization of Prompt Instruction。
  3. 对指令和示例的优化 Sequential Optimization。
  4. 推理和验证 Self-generated Reasoning and Validation。


  * 其中，对于1，输入任务相关内容，如任务描述、指令以及训练数据。
  * 对于2，指令的迭代优化，每次迭代的过程如图2所示，首先对于输入的Thinking Styles、Problem Description， Prompt Instruction，去生成具有不同多样性的提示词（N mutated instructions），然后对每一个生成的提示词，从训练数据集中随机选择K个数据进行预测并打分，然后通过 Critique模块对每个提示词给出评论（如分数低的原因、预测不准确的建议等），利用Critique生成的内容，重新调用大模型生成新的提示词指令，这样完成了提示词指令的优化的一次迭代。  
其中 Thinking Styles提供了多个角度思考的提示词，每种Style 都预期可以生成一条指令，最终选取N条指令作为候选指令，示例如下：



图2

在指令的迭代优化后，会有一个示例选择步骤，主要是从训练集中选择容易预测失败的示例，这些示例和指令一起传入下一阶段。

如源代码所示，在完成第一步之后得到 current_base_instruction，然后循环数据集，目标是在数据集中找出params.few_shot_count个examples，寻找的标准如代码所示，即在原始数据集中找到目标数量个错误的例子，如果没有达到目标数量，则随机从数据集中取出补齐。
    
    
    examples = []
    params.base_instruction = current_base_instruction
    for example in self.dataset:
        solve_prompt = self.prompt_pool.solve_template.format(
            questions_batch_size=1,
            instruction=params.base_instruction,
            answer_format=params.answer_format,
            questions=example[DatasetSpecificProcessing.QUESTION_LITERAL])
        generated_text = self.chat_completion(solve_prompt)
        #  self.evaluate 返回的是错误示例
        examples.extend(self.evaluate(generated_text, [example]))
        if len(examples) >= params.few_shot_count:
            #  找到 few shot count 数量的 错误的examples
            break
    if len(examples) < params.few_shot_count:
        examples = examples + random.sample(self.dataset, params.few_shot_count - len(examples))

  * 对于3，对指令和示例的优化，如图3所示，输入为错误的例子，利用Critique模块生成评论，利用评论、例子、原始指令拼接模板调用大模型，生成优化后的指令；在这个迭代过程中，指令、例子均得到了优化。



图3 序列优化

  * 对于4，在每一个示例生成时，加入得出当前示例对应答案的推理逻辑，如代码所示，generate_reasoning 函数描述了推理逻辑生成的过程，其结果将会加入示例中，作为提示词的一部分。


    
    
    def generate_reasoning(self, task_description: str, instruction: str, question: str, answer: str) -> str:
        """
        For the given question return the reasoning that's needed to arrive at the provided answer
    
        :param task_description: Task description of the given task
        :param instruction: Instruction given to LLM for solving the given task
        :param question: Question from the task to be solved
        :param answer: Answer to the question
        :return: Reasoning that went through for getting answer `answer` for question `question`
        """
    
        prompt_template = self.prompt_pool.generate_reason_template.format(task_description=task_description,
                                                                            instruction=instruction,
                                                                            question=question,
                                                                            answer=answer)
        return self.chat_completion(user_prompt=prompt_template)

## 迭代4-使用PromptWizard，使用合成数据集

迭代2中，我们介绍了不使用数据集的情况下自动化的生成提示词并预测结果，PromptWizard在没有数据集时，提供了生成数据集的能力，本小节介绍如何利用PromptWizard生成伪造的数据集并利用生成的数据集自动优化提示词

主要分为两步：

**第一步：生成合成数据**

需要先修改 prompt_config.yaml 文件。
    
    
    "num_train_examples":20

然后执行代码生成合成数据。
    
    
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl=None,
                       data_processor=None)
    best_prompt, expert_profile = gp.get_best_prompt(use_examples=False,run_without_train_examples=False,generate_synthetic_examples=True)

生成的合成数据将会存储在 train_systhetic.jsonl中。
    
    
    {"question": ": Solve the quadratic equation 3x^2 - 4x - 7 = 0.", "answer": ": To solve the quadratic equation, we can use the quadratic formula x = [-b \u00b1 sqrt(b^2 - 4ac)] / (2a). Here, a = 3, b = -4, and c = -7. Substituting these values into the formula, we get x = [4 \u00b1 sqrt((-4)^2 - 4*3*(-7))] / (2*3) = [4 \u00b1 sqrt(16 + 84)] / 6 = [4 \u00b1 sqrt(100)] / 6 = [4 \u00b1 10] / 6. So, the solutions are x = 14/6 = 7/3 and x = -6/6 = -1. <ANS_START>x = 7/3, x = -1<ANS_END>", "final_answer": "x = 7/3, x = -1"}
    {"question": ": What is the derivative of the function f(x) = x^3 + 2x^2 - 3x + 1?", "answer": ": To find the derivative of the function, we apply the power rule for each term. The derivative of x^3 is 3x^2, the derivative of 2x^2 is 4x, and the derivative of -3x is -3. The derivative of the constant term 1 is 0. So, the derivative of the function is f'(x) = 3x^2 + 4x - 3. <ANS_START>f'(x) = 3x^2 + 4x - 3<ANS_END>", "final_answer": "f'(x) = 3x^2 + 4x - 3"}
    {"question": ": Simplify the expression (x^3 - 8) / (x - 2).", "answer": ": The expression (x^3 - 8) is a difference of cubes and can be factored as (x - 2)(x^2 + 2x + 4). So, the given expression simplifies to x^2 + 2x + 4. <ANS_START>x^2 + 2x + 4<ANS_END>", "final_answer": "x^2 + 2x + 4"}
    {"question": ": Solve the system of equations: y = x^2 + 2x + 1 and y = 2x + 3.", "answer": ": To solve the system, we can set the two equations equal to each other: x^2 + 2x + 1 = 2x + 3. Simplifying this equation gives x^2 = 2, so x = \u00b1sqrt(2). Substituting these values into the first equation gives y = (\u00b1sqrt(2))^2 + 2*\u00b1sqrt(2) + 1 = 2 \u00b1 2sqrt(2) + 1 = 3 \u00b1 2sqrt(2). So, the solutions are (sqrt(2), 3 + 2sqrt(2)) and (-sqrt(2), 3 - 2sqrt(2)). <ANS_START>(sqrt(2), 3 + 2sqrt(2)), (-sqrt(2), 3 - 2sqrt(2))<ANS_END>", "final_answer": "(sqrt(2), 3 + 2sqrt(2)), (-sqrt(2), 3 - 2sqrt(2))"}
    {"question": ": What is the integral of x^2 e^x dx?", "answer": ": To find the integral, we can use integration by parts with u = x^2 and dv = e^x dx. Then du = 2x dx and v = e^x. The formula for integration by parts is \u222budv = uv - \u222bvdu, so \u222bx^2 e^x dx = x^2 e^x - \u222b2x e^x dx. The integral \u222b2x e^x dx can be found again using integration by parts with u = 2x and dv = e^x dx, giving \u222b2x e^x dx = 2x e^x - 2\u222be^x dx = 2x e^x - 2e^x. So, the original integral is \u222bx^2 e^x dx = x^2 e^x - 2x e^x + 2e^x + C. <ANS_START>x^2 e^x - 2x e^x + 2e^x + C<ANS_END>", "final_answer": "x^2 e^x - 2x e^x + 2e^x + C"}
    {"question": ": Find the area under the curve y = x^3 from x = 0 to x = 2.", "answer": ": The area under the curve is given by the definite integral from 0 to 2 of x^3 dx. The antiderivative of x^3 is (1/4)x^4, so the area is [(1/4)(2)^4 - (1/4)(0)^4] = (1/4)(16) = 4. <ANS_START>4<ANS_END>", "final_answer": "4"}
    {"question": ": What is the limit of (e^x - 1)/x as x approaches 0?", "answer": ": This limit is an indeterminate form of type 0/0, so we can apply L'Hopital's rule, which states that the limit of a quotient of two functions is equal to the limit of the quotients of their derivatives if the latter limit exists. The derivative of e^x is e^x and the derivative of x is 1, so the limit is the limit as x approaches 0 of e^x/1 = e^x, which is e^0 = 1. <ANS_START>1<ANS_END>", "final_answer": "1"}
    {"question": ": Solve the inequality x^2 - 4x + 3 > 0.", "answer": ": To solve the inequality, we first find the roots of the equation x^2 - 4x + 3 = 0. These are x = 1 and x = 3. The inequality is satisfied when the quadratic is positive, which occurs outside the interval between the roots. So, the solution is x < 1 or x > 3. <ANS_START>x < 1 or x > 3<ANS_END>", "final_answer": "x < 1 or x > 3"}
    {"question": ": Find the roots of the cubic equation x^3 - 6x^2 + 11x - 6 = 0.", "answer": ": The roots of the cubic equation can be found by factoring. The equation factors as (x - 1)(x - 2)(x - 3) = 0, so the roots are x = 1, x = 2, and x = 3. <ANS_START>x = 1, x = 2, x = 3<ANS_END>", "final_answer": "x = 1, x = 2, x = 3"}
    {"question": ": What is the sum of the first 10 terms of the geometric sequence 3, 6, 12, ...?", "answer": ": The sum S of the first n terms of a geometric sequence with first term a and common ratio r is given by S = a*(1 - r^n) / (1 - r). Here, a = 3, r = 2, and n = 10, so S = 3*(1 - 2^10) / (1 - 2) = -3*(1 - 1024) / -1 = 3*1023 = 3069. <ANS_START>3069<ANS_END>", "final_answer": "3069"}
    {"question": ": What is the value of the sum 1 - 1/3 + 1/9 - 1/27 + ...?", "answer": ": This is a geometric series with first term a = 1 and common ratio r = -1/3. The sum S of an infinite geometric series with |r| < 1 is given by S = a / (1 - r). So, S = 1 / (1 - (-1/3)) = 1 / (4/3) = 3/4. <ANS_START>3/4<ANS_END>", "final_answer": "3/4"}
    {"question": ": What is the derivative of the function f(x) = sin(2x)?", "answer": ": The derivative of sin(2x) can be found using the chain rule, which states that the derivative of a composite function is the derivative of the outer function times the derivative of the inner function. The derivative of sin(x) is cos(x), and the derivative of 2x is 2, so the derivative of sin(2x) is cos(2x)*2 = 2cos(2x). <ANS_START>2cos(2x)<ANS_END>", "final_answer": "2cos(2x)"}
    {"question": ": What is the value of the definite integral from 0 to \u03c0/2 of cos(2x) dx?", "answer": ": The antiderivative of cos(2x) is (1/2)sin(2x), so the value of the integral is [(1/2)sin(2*(\u03c0/2)) - (1/2)sin(2*0)] = (1/2)[sin(\u03c0) - sin(0)] = (1/2)[0 - 0] = 0. <ANS_START>0<ANS_END>", "final_answer": "0"}
    {"question": ": Solve the equation 2^x = 5 for x.", "answer": ": To solve the equation for x, we can take the natural logarithm of both sides to get x ln(2) = ln(5), so x = ln(5) / ln(2). <ANS_START>ln(5) / ln(2)<ANS_END>", "final_answer": "ln(5) / ln(2)"}
    {"question": ": Find the area of a regular hexagon with side length 3.", "answer": ": The area A of a regular hexagon with side length s is given by A = (3\u221a3/2)s^2. So, A = (3\u221a3/2)(3)^2 = 27\u221a3/2. <ANS_START>27\u221a3/2<ANS_END>", "final_answer": "27\u221a3/2"}
    {"question": ": What is the volume of a cone with radius 2 and height 3?", "answer": ": The volume V of a cone with radius r and height h is given by V = (1/3)\u03c0r^2h. So, V = (1/3)\u03c0(2)^2(3) = 4\u03c0. <ANS_START>4\u03c0<ANS_END>", "final_answer": "4\u03c0"}
    {"question": ": What is the sum of the interior angles of a pentagon?", "answer": ": The sum S of the interior angles of a polygon with n sides is given by S = (n - 2) * 180 degrees. So, for a pentagon, S = (5 - 2) * 180 = 3 * 180 = 540 degrees. <ANS_START>540 degrees<ANS_END>", "final_answer": "540 degrees"}
    {"question": ": What is the distance between the points (1, 2, 3) and (4, 5, 6)?", "answer": ": The distance d between two points (x1, y1, z1) and (x2, y2, z2) in 3D space is given by d = sqrt[(x2 - x1)^2 + (y2 - y1)^2 + (z2 - z1)^2]. So, d = sqrt[(4 - 1)^2 + (5 - 2)^2 + (6 - 3)^2] = sqrt[3^2 + 3^2 + 3^2] = sqrt[27] = 3sqrt(3). <ANS_START>3sqrt(3)<ANS_END>", "final_answer": "3sqrt(3)"}
    {"question": ": What is the equation of the line passing through the points (1, 2) and (3, 4)?", "answer": ": The slope m of the line passing through two points (x1, y1) and (x2, y2) is given by m = (y2 - y1) / (x2 - x1). So, m = (4 - 2) / (3 - 1) = 1. The equation of the line is then y - y1 = m(x - x1), so y - 2 = 1(x - 1), or y = x + 1. <ANS_START>y = x + 1<ANS_END>", "final_answer": "y = x + 1"}
    {"question": ": What are the coordinates of the centroid of the triangle with vertices at (1, 2), (3, 4), and (5, 6)?", "answer": ": The coordinates (x, y) of the centroid of a triangle with vertices at (x1, y1), (x2, y2), and (x3, y3) are given by x = (x1 + x2 + x3) / 3 and y = (y1 + y2 + y3) / 3. So, x = (1 + 3 + 5) / 3 = 3 and y = (2 + 4 + 6) / 3 = 4. <ANS_START>(3, 4)<ANS_END>", "final_answer": "(3, 4)"}

**第二步：用合成数据优化提示词**

修改 promptopt_config.yaml 文件。
    
    
    "task_description": "You are a mathematics expert. You will be given a mathematics problem which you need to solve",
    "base_instruction": "Lets think step by step.",
    "mutation_rounds": 2,
    "few_shot_count": 5,
    "generate_reasoning": True,
    "mutate_refine_iterations" : 3,
    "seen_set_size":20

优化提示词。
    
    
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl = "train_synthetic.jsonl",
                       data_processor=gsm8k_processor)
    best_prompt, expert_profile = gp.get_best_prompt(use_examples=True,run_without_train_examples=False,generate_synthetic_examples=False)

查看结果，可以看出提示词中增加了很多示例和推理过程。
    
    
    Generating Expert Identity....
    Expert Identity: You are a mathematician with a strong background in various fields of mathematics, including algebra, calculus, geometry, and statistics. You have a deep understanding of mathematical theories and principles, and you are skilled at solving complex problems with precision and clarity. Your analytical skills and logical reasoning enable you to break down problems into manageable steps and find accurate solutions efficiently. You are familiar with a wide range of mathematical techniques and tools, and you can apply them to solve problems in both theoretical and applied contexts. Your expertise allows you to explain your solutions clearly and concisely, making complex concepts accessible to others. Whether the problem involves solving equations, proving theorems, or analyzing data, you are well-equipped to provide a thorough and correct solution.
    Final best prompt: 
    （变化点：生成优化指令）
    Provide a clear and detailed solution, breaking down all necessary steps. Ensure that the final answer is clearly marked and separated from the solution steps. Use proper mathematical notation and formatting throughout. Verify the final answer by checking the solution steps for accuracy. Simplify all expressions and fractions where possible. Handle special cases or edge cases appropriately, and clearly state any assumptions or conditions applied during the solution process. Finally, review the entire solution to ensure logical consistency and correct formatting.
    （变化点：生成示例）
    [Question] Solve for \( x \) in the equation \( 2x + 3 = 11 \).
    [Answer] To solve for \( x \) in the equation \( 2x + 3 = 11 \), we will follow these steps:
    
    1. **Isolate the term containing \( x \)**:
       We start by isolating the term with \( x \) on one side of the equation. To do this, we need to eliminate the constant term on the left side of the equation.
    
       \[
       2x + 3 = 11
       \]
    
       Subtract 3 from both sides of the equation:
    
       \[
       2x + 3 - 3 = 11 - 3
       \]
    
       Simplifying this, we get:
    
       \[
       2x = 8
       \]
    
    2. **Solve for \( x \)**:
       Now, we need to solve for \( x \) by isolating \( x \) itself. Since \( x \) is multiplied by 2, we will divide both sides of the equation by 2 to solve for \( x \).
    
       \[
       \frac{2x}{2} = \frac{8}{2}
       \]
    
       Simplifying this, we get:
    
       \[
       x = 4
       \]
    
    3. **Verify the solution**:
       To ensure our solution is correct, we substitute \( x = 4 \) back into the original equation and check if both sides are equal.
    
       Original equation:
    
       \[
       2x + 3 = 11
       \]
    
       Substitute \( x = 4 \):
    
       \[
       2(4) + 3 = 11
       \]
    
       Simplifying this, we get:
    
       \[
       8 + 3 = 11
       \]
    
       \[
       11 = 11
       \]
    
       Since both sides of the equation are equal, our solution is verified to be correct.
    
    **Final Answer**: \( x = 4 \) <ANS_START> \( x = 4 \) <ANS_END>
    
    ...
    （变化点：答案格式）
    
    For each question present the reasoning followed by the correct answer.

那么合成数据是如何生成的呢？如下面代码所示为生成数据的核心代码，首先利examples_critique_template_zero_shot提示词模板，得到填充后的提示词，填充过程中输入了原始提示词、任务描述以及生成示例的数量，然后利用few_shot_critique_prompt以及self.prompt_pool.expert_profile请求大模型得到生成的数据集，最终再调用examples的提取函数extract_examples_frm_response将数据格式化。
    
    
        def generate_best_examples_zero_shot(self,params: PromptOptimizationParams) -> List:
            """
            Generate best example to be give as few-shots for the given task.
    
            :param params: Object having hyperparameters for this prompt optimization technique.
            :return: List of synthetic examples
            """
            #  这里使用了一个生成 n个exmaple的默认模板 examples_critique_template_zero_shot，角色就是系统模板
            few_shot_critique_prompt = self.prompt_pool.examples_critique_template_zero_shot.\
                format(prompt=params.base_instruction,
                       task_description=params.task_description,
                       num_examples=params.num_train_examples)
            #  这里用到了角色，用角色和prompt生成评论
            critique = self.chat_completion(few_shot_critique_prompt, self.prompt_pool.expert_profile)
            #  用评论产生新的prompt
            few_shot_opt_prompt = self.prompt_pool.examples_optimization_template.\
                format(prompt=params.base_instruction,
                       examples="",
                       gt_example="",
                       critique=critique,
                       task_description=params.task_description,
                       num_examples=params.num_train_examples)
            synthetic_examples = self.chat_completion(few_shot_opt_prompt, self.prompt_pool.expert_profile)
            #  格式化数据集
            synthetic_examples = self.extract_examples_frm_response(synthetic_examples)
            return synthetic_examples

## 实验部分

为了比较迭代1-迭代4产生提示词的质量，本文使用GSM8K测试集的前150个样本，逐个请求大模型获取结果，并计算每个提示词得到预测的准确度。

### 准确度计算方式

对于每个预测样本，与答案完全相同则认为预测正确，因此，准确率 = 正确数量/样本总数。

### 实验代码

对于每种提示词，均使用同样的测试数据集和测试代码，由于迭代1 和 迭代2没有产生 BEST_PROMPT 和 EXPERT_IDENTITY 因此需手动赋值，而迭代3和迭代4则直接调用验证函数即可，代码如下。
    
    
    迭代1：:
    gp = GluePromptOpt(promptopt_config_path,
                       setup_config_path,
                       dataset_jsonl=None,
                       data_processor=GSM8k())
    gp.BEST_PROMPT = "Lets think step by step."
    gp.EXPERT_PROFILE = "You are a mathematics expert. You will be given a mathematics problem which you need to solve"
    test_score = gp.evaluate(test_dataset_jsonl=os.path.join("data", "test.jsonl"))

在log文件中查看准确率计算结果，在150个GSM8K测试集中答对了122个，准确率81%，（测试消耗Token：4万）。
    
    
    {"accuracy": "1/1 : 100%", "predicted": "18", "actual": "18"}
    ...
    {"accuracy": "10/12 : 8333333333333334%", "predicted": "694", "actual": "694"}
    ...
    {"accuracy": "91/115 : 7913043478260869%", "predicted": "6", "actual": "6"}
    ...
    {"accuracy": "122/150 : 8133333333333334%", "predicted": "16", "actual": "16"}

**迭代2**

从日志文件可以看出PromptWizard自动的产生多种类型的提示词，每部分提示词都包含了 Expert Profile 和 Prompt两部分，Expert Profile中描述了 角色背景信息以及Prompt中描述了 指令、结果格式化方式、以及关键词三部分，比手工编写的提示词描述的更充分，为了验证新生成提示词的效果，我们利用PromptWizard，在GSM8K数据集的测试集上进行预测，这里就选取Variations 1 输出的提示词内容进行验证(注意，当前场景PromptWizard只赋值生成了提示词，但没有选择最佳提示词，因此需要手动复制对应的Expert Profile和 Prompt)。
    
    
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

在log文件中查看准确率计算结果，在150个GSM8K测试集中答对了124个，准确率82.6%（测试消耗Token：8万）。
    
    
    {"accuracy": "1/1 : 100%", "predicted": "18", "actual": "18"}
    ...
    {"accuracy": "24/31 : 7741935483870968%", "predicted": "109", "actual": "109"}
    ...
    {"accuracy": "102/126 : 8095238095238095%", "predicted": "10", "actual": "10"}
    ...
    {"accuracy": "124/150 : 8266666666666667%", "predicted": "16", "actual": "16"}

**迭代3**
    
    
     test_score = gp.evaluate(test_dataset_jsonl=os.path.join("data", "test.jsonl"))

最终得到的结果为存到 evaluation 目录中,在150个GSM8K测试集中答对了143个，准确率95.3%（优化+测试消耗总Token：13万）。
    
    
    {"accuracy": "1/1 : 100%", "predicted": "18", "actual": "18"}
    ...
    {"accuracy": "60/63 : 9523809523809523%", "predicted": "25000", "actual": "25000"}
    ...
    {"accuracy": "109/114 : 956140350877193%", "predicted": "25", "actual": "25"}
    ...
    {"accuracy": "143/150 : 9533333333333334%", "predicted": "16", "actual": "16"}

**迭代4**

在150个GSM8K测试集中答对了125个，准确率83.3%（生成数据+优化+测试消耗总Token：20万）。
    
    
    test_score = gp.evaluate(test_dataset_jsonl=os.path.join("data", "test.jsonl"))

最终得到的结果为存到 evaluation 目录中。
    
    
    {"accuracy": "1/1 : 100%", "predicted": "18", "actual": "18"}
    ...
    {"accuracy": "41/44 : 9318181818181818%", "predicted": "48", "actual": "48"}
    ...
    {"accuracy": "113/137 : 8248175182481752%", "predicted": "6", "actual": "6"}
    ...
    {"accuracy": "125/150 : 8333333333333334%", "predicted": "16", "actual": "16"}

### 实验结果

| 迭代1-手工提示词| 迭代2-优化提示词，不使用示例| 迭代3-优化提示词，使用GSM8K训练数据| 迭代4-优化提示词，使用合成数据  
---|---|---|---|---  
准确率| 81%| 82.6%| 95.3| 83.3%  
  
从实验结果可以分析，使用GSM8K原有训练集得到的提示词效果有明显优势，其次是带有合成数据生成的提示词，然后是不带示例的提示词和手工提示词，其中 迭代1、2、4的准确率递增但较为接近。

从所有实验结果可以推测优质的提示词示例对预测结果有较大帮助，此外PromptWizard的提示词优化能力较手工提示词也有着明显的优势。

## 附录：提示词示例

图1中步骤2、3、4的提示词如下所示。

### 2、提示词如下
    
    
    Expert Identity: You are a highly skilled mathematician with expertise in a wide range of mathematical disciplines, including algebra, calculus, geometry, statistics, and more. Your analytical mind and problem-solving abilities allow you to tackle even the most complex mathematical problems with precision and clarity. You have a deep understanding of mathematical theories, formulas, and principles, and you can apply them effectively to solve problems. Whether the problem involves solving equations, proving theorems, or analyzing data, you approach it methodically, breaking it down into manageable steps and arriving at a logical and accurate solution. Your ability to explain your reasoning clearly makes you an excellent teacher and communicator, capable of helping others understand the beauty and logic of mathematics.
    Final best prompt: 
    **You are a mathematics expert tasked with solving a variety of mathematical problems. Follow these steps to ensure accurate and logical solutions:**
    1. **Rephrase and Understand the Problem**: Begin by rephrasing the problem in your own words to confirm comprehension. Identify the given information, what needs to be determined, and categorize the problem type (e.g., ratio, average, fraction, or multi-step calculation).
    2. **Define Variables and Relationships**: Explicitly list and define all variables and their relationships. Pay close attention to proportional relationships (e.g., pages read per unit of time) and unit conversions (e.g., minutes to hours). Ensure unit consistency throughout the problem.
    3. **Break the Problem into Smaller Parts**: If the problem involves multiple components (e.g., weekdays and weekends, milk and eggs), solve each part separately before combining the results. Clearly outline the steps for each part.
    4. **Perform Intermediate Calculations Carefully**: Solve each part step by step, ensuring that each step is explicitly reasoned and builds on the previous one. Verify intermediate results by cross-checking them against the problem context or using alternative methods.
    5. **Combine Results and Simplify**: Combine the results of smaller parts to arrive at the final answer. Simplify the problem by focusing on the key variables and their interactions. Ensure all parts of the problem are accounted for.
    6. **Verify Logical Consistency**: Check that each step logically follows from the previous one and leads to the final solution. Ensure that your calculations are accurate and consistent. Address any problem-specific nuances.
    7. **Present the Solution Clearly**: Present your solution in a clear and concise manner, making sure each step is easy to follow and logically connected to the next. Structure the final answer to meet the problem's requirements (e.g., separate or combined quantities). Always include units in your final answer to ensure clarity.
    **Let’s solve this problem step by step.**
    For each question present the reasoning followed by the correct answer.

### 3、提示词如下
    
    
    Expert Identity: You are a highly skilled mathematician with expertise in a wide range of mathematical disciplines, including algebra, calculus, geometry, statistics, and more. Your analytical mind and problem-solving abilities allow you to tackle even the most complex mathematical problems with precision and clarity. You have a deep understanding of mathematical theories, formulas, and principles, and you can apply them effectively to solve problems. Whether the problem involves solving equations, proving theorems, or analyzing data, you approach it methodically, breaking it down into manageable steps and arriving at a logical and accurate solution. Your ability to explain your reasoning clearly makes you an excellent teacher and communicator, capable of helping others understand the beauty and logic of mathematics.
    Final best prompt: 
    **You are a mathematics expert tasked with solving a variety of mathematical problems. Follow these steps to ensure accurate and logical solutions:**
    1. **Rephrase and Understand the Problem**: Begin by rephrasing the problem in your own words to confirm comprehension. Identify the given information, what needs to be determined, and categorize the problem type (e.g., ratio, average, fraction, or multi-step calculation).
    2. **Define Variables and Relationships**: Explicitly list and define all variables and their relationships. Pay close attention to proportional relationships (e.g., pages read per unit of time) and unit conversions (e.g., minutes to hours). Ensure unit consistency throughout the problem.
    3. **Break the Problem into Smaller Parts**: If the problem involves multiple components (e.g., weekdays and weekends, milk and eggs), solve each part separately before combining the results. Clearly outline the steps for each part.
    4. **Perform Intermediate Calculations Carefully**: Solve each part step by step, ensuring that each step is explicitly reasoned and builds on the previous one. Verify intermediate results by cross-checking them against the problem context or using alternative methods.
    5. **Combine Results and Simplify**: Combine the results of smaller parts to arrive at the final answer. Simplify the problem by focusing on the key variables and their interactions. Ensure all parts of the problem are accounted for.
    6. **Verify Logical Consistency**: Check that each step logically follows from the previous one and leads to the final solution. Ensure that your calculations are accurate and consistent. Address any problem-specific nuances.
    7. **Present the Solution Clearly**: Present your solution in a clear and concise manner, making sure each step is easy to follow and logically connected to the next. Structure the final answer to meet the problem's requirements (e.g., separate or combined quantities). Always include units in your final answer to ensure clarity.
    **Let’s solve this problem step by step.**
    [Question] A bakery sells 120 loaves of bread each day. If they sell twice as many loaves on weekends as they do on weekdays, how many loaves do they sell in a week?
    [Answer] 
    **Final Answer**: 1080 <ANS_START>1080<ANS_END>
    [Question] A car travels 60 miles in the first hour and 75 miles in the second hour. If the car continues to travel at the same average speed, how many miles will it travel in 5 hours?
    [Answer]
    **Final Answer:** 337.5 miles <ANS_START>337.5<ANS_END>
    [Question] A school has 480 students. If 3/8 of the students are in the 6th grade, how many students are in the 6th grade?
    [Answer] 
    **Final Answer:** 180 <ANS_START>180<ANS_END>
    [Question] A farmer has 15 cows and 25 chickens. If each cow produces 8 liters of milk per day and each chicken lays 3 eggs per day, how many liters of milk and eggs does the farmer collect in a day?
    [Answer] 
    **Final Answer**: 75. <ANS_START>75<ANS_END>
    [Question] A train travels 300 kilometers in 3 hours. If the train continues at the same speed, how long will it take to travel 500 kilometers?
    [Answer] 
    **Final Answer**: 5 hours. <ANS_START>5<ANS_END>
    For each question present the reasoning followed by the correct answer.

### 4、提示词如下
    
    
    Expert Identity: You are a highly skilled mathematician with expertise in a wide range of mathematical disciplines, including algebra, calculus, geometry, statistics, and more. Your analytical mind and problem-solving abilities allow you to tackle even the most complex mathematical problems with precision and clarity. You have a deep understanding of mathematical theories, formulas, and principles, and you can apply them effectively to solve problems. Whether the problem involves solving equations, proving theorems, or analyzing data, you approach it methodically, breaking it down into manageable steps and arriving at a logical and accurate solution. Your ability to explain your reasoning clearly makes you an excellent teacher and communicator, capable of helping others understand the beauty and logic of mathematics.
    Final best prompt: 
    **You are a mathematics expert tasked with solving a variety of mathematical problems. Follow these steps to ensure accurate and logical solutions:**
    1. **Rephrase and Understand the Problem**: Begin by rephrasing the problem in your own words to confirm comprehension. Identify the given information, what needs to be determined, and categorize the problem type (e.g., ratio, average, fraction, or multi-step calculation).
    2. **Define Variables and Relationships**: Explicitly list and define all variables and their relationships. Pay close attention to proportional relationships (e.g., pages read per unit of time) and unit conversions (e.g., minutes to hours). Ensure unit consistency throughout the problem.
    3. **Break the Problem into Smaller Parts**: If the problem involves multiple components (e.g., weekdays and weekends, milk and eggs), solve each part separately before combining the results. Clearly outline the steps for each part.
    4. **Perform Intermediate Calculations Carefully**: Solve each part step by step, ensuring that each step is explicitly reasoned and builds on the previous one. Verify intermediate results by cross-checking them against the problem context or using alternative methods.
    5. **Combine Results and Simplify**: Combine the results of smaller parts to arrive at the final answer. Simplify the problem by focusing on the key variables and their interactions. Ensure all parts of the problem are accounted for.
    6. **Verify Logical Consistency**: Check that each step logically follows from the previous one and leads to the final solution. Ensure that your calculations are accurate and consistent. Address any problem-specific nuances.
    7. **Present the Solution Clearly**: Present your solution in a clear and concise manner, making sure each step is easy to follow and logically connected to the next. Structure the final answer to meet the problem's requirements (e.g., separate or combined quantities). Always include units in your final answer to ensure clarity.
    **Let’s solve this problem step by step.**
    [Question] A bakery sells 120 loaves of bread each day. If they sell twice as many loaves on weekends as they do on weekdays, how many loaves do they sell in a week?
    [Answer] **Step 1: Rephrase and Understand the Problem**  
    The bakery sells 120 loaves of bread each day. On weekends, they sell twice as many loaves as they do on weekdays. We need to determine the total number of loaves sold in a week.
    **Step 2: Define Variables and Relationships**  
    - Let \( W \) represent the number of loaves sold on a weekday.  
    - On weekends, the bakery sells twice as many loaves, so the number of loaves sold on a weekend day is \( 2W \).  
    - There are 5 weekdays and 2 weekend days in a week.  
    **Step 3: Break the Problem into Smaller Parts**  
    1. Calculate the total loaves sold on weekdays.  
    2. Calculate the total loaves sold on weekends.  
    3. Combine the results to find the total loaves sold in a week.  
    **Step 4: Perform Intermediate Calculations Carefully**  
    1. **Weekday Sales**:  
       - The bakery sells \( W = 120 \) loaves per weekday.  
       - Total loaves sold on weekdays = \( 5 \times W = 5 \times 120 = 600 \).  
    2. **Weekend Sales**:  
       - On weekends, the bakery sells \( 2W = 2 \times 120 = 240 \) loaves per day.  
       - Total loaves sold on weekends = \( 2 \times 240 = 480 \).  
    **Step 5: Combine Results and Simplify**  
    - Total loaves sold in a week = Loaves sold on weekdays + Loaves sold on weekends.  
    - Total loaves sold in a week = \( 600 + 480 = 1080 \).  
    **Step 6: Verify Logical Consistency**  
    - The calculations are consistent with the problem statement.  
    - The total loaves sold on weekdays and weekends add up correctly to 1080.  
    **Step 7: Present the Solution Clearly**  
    The bakery sells a total of **1080 loaves** in a week.  
    **Final Answer**: 1080 <ANS_START>1080<ANS_END>
    [Question] A car travels 60 miles in the first hour and 75 miles in the second hour. If the car continues to travel at the same average speed, how many miles will it travel in 5 hours?
    [Answer] **Step 1: Rephrase and Understand the Problem**  
    The problem states that a car travels 60 miles in the first hour and 75 miles in the second hour. It then asks how many miles the car will travel in 5 hours if it continues to travel at the same average speed. The goal is to determine the total distance traveled in 5 hours based on the given information.
    **Step 2: Define Variables and Relationships**  
    - Let \( d_1 = 60 \) miles (distance traveled in the first hour).  
    - Let \( d_2 = 75 \) miles (distance traveled in the second hour).  
    - Let \( t = 5 \) hours (total time for which the distance needs to be calculated).  
    - The average speed \( v \) is calculated as the total distance traveled divided by the total time taken.  
    **Step 3: Break the Problem into Smaller Parts**  
    1. Calculate the total distance traveled in the first two hours.  
    2. Calculate the average speed of the car.  
    3. Use the average speed to calculate the distance traveled in 5 hours.  
    **Step 4: Perform Intermediate Calculations Carefully**  
    1. Total distance in the first two hours:  
       \[
       d_{\text{total}} = d_1 + d_2 = 60 + 75 = 135 \text{ miles}
       \]  
    2. Average speed:  
       \[
       v = \frac{d_{\text{total}}}{t_{\text{total}}} = \frac{135}{2} = 67.5 \text{ miles per hour}
       \]  
    3. Distance traveled in 5 hours:  
       \[
       d = v \times t = 67.5 \times 5 = 337.5 \text{ miles}
       \]  
    **Step 5: Combine Results and Simplify**  
    The total distance traveled in 5 hours is 337.5 miles.  
    **Step 6: Verify Logical Consistency**  
    - The average speed calculation is consistent with the given distances and times.  
    - The final distance calculation uses the average speed and total time correctly.  
    - All units are consistent (miles and hours).  
    **Step 7: Present the Solution Clearly**  
    The car travels 60 miles in the first hour and 75 miles in the second hour, totaling 135 miles in 2 hours. The average speed is \( 67.5 \) miles per hour. At this speed, the car will travel \( 337.5 \) miles in 5 hours.  
    **Final Answer:** 337.5 miles <ANS_START>337.5<ANS_END>
    [Question] A school has 480 students. If 3/8 of the students are in the 6th grade, how many students are in the 6th grade?
    [Answer] **Reasoning Chain:**
    1. **Rephrase and Understand the Problem**:  
       - The problem states that a school has 480 students in total. Out of these, \( \frac{3}{8} \) of the students are in the 6th grade. We need to determine how many students are in the 6th grade.
    2. **Define Variables and Relationships**:  
       - Let \( N \) represent the total number of students in the school, which is given as 480.  
       - Let \( G \) represent the number of 6th-grade students, which is \( \frac{3}{8} \) of \( N \).  
       - The relationship is: \( G = \frac{3}{8} \times N \).
    3. **Break the Problem into Smaller Parts**:  
       - The problem involves a single calculation: finding \( \frac{3}{8} \) of 480.  
       - No further breakdown is needed as the problem is straightforward.
    4. **Perform Intermediate Calculations Carefully**:  
       - Calculate \( \frac{3}{8} \times 480 \):  
         \[
         G = \frac{3}{8} \times 480
         \]
       - First, simplify the multiplication:  
         \[
         G = 3 \times \left( \frac{480}{8} \right)
         \]
       - Calculate \( \frac{480}{8} \):  
         \[
         \frac{480}{8} = 60
         \]
       - Now multiply by 3:  
         \[
         G = 3 \times 60 = 180
         \]
    5. **Combine Results and Simplify**:  
       - The calculation directly yields the number of 6th-grade students, which is 180.  
       - No further simplification is needed.
    6. **Verify Logical Consistency**:  
       - The calculation \( \frac{3}{8} \times 480 = 180 \) is logically consistent.  
       - Cross-checking: \( \frac{3}{8} \times 480 = 180 \) is correct, as \( 3 \times 60 = 180 \).
    7. **Present the Solution Clearly**:  
       - The number of 6th-grade students is 180.  
       - The final answer is: **180**.
    **Final Answer:** 180 <ANS_START>180<ANS_END>
    [Question] A farmer has 15 cows and 25 chickens. If each cow produces 8 liters of milk per day and each chicken lays 3 eggs per day, how many liters of milk and eggs does the farmer collect in a day?
    [Answer] **Rephrase and Understand the Problem**:  
    The farmer has 15 cows and 25 chickens. Each cow produces 8 liters of milk per day, and each chicken lays 3 eggs per day. We need to calculate the total liters of milk and eggs the farmer collects in a day. The problem involves calculating two separate quantities (milk and eggs) and then combining them into a single total.
    ---
    **Define Variables and Relationships**:  
    - Let \( C \) represent the number of cows, which is 15.  
    - Let \( M \) represent the liters of milk produced per cow per day, which is 8 liters.  
    - Let \( H \) represent the number of chickens, which is 25.  
    - Let \( E \) represent the number of eggs laid per chicken per day, which is 3 eggs.  
    The relationships are:  
    - Total milk produced per day = \( C \times M \).  
    - Total eggs laid per day = \( H \times E \).  
    ---
    **Break the Problem into Smaller Parts**:  
    1. Calculate the total milk produced by the cows in a day.  
    2. Calculate the total eggs laid by the chickens in a day.  
    3. Combine the results to find the total liters of milk and eggs collected in a day.  
    ---
    **Perform Intermediate Calculations Carefully**:  
    1. **Total milk produced per day**:  
       \( C \times M = 15 \times 8 = 120 \) liters of milk.  
    2. **Total eggs laid per day**:  
       \( H \times E = 25 \times 3 = 75 \) eggs.  
    ---
    **Combine Results and Simplify**:  
    The problem asks for the total liters of milk and eggs collected in a day. However, milk is measured in liters, and eggs are measured in units (not liters). To combine them meaningfully, we need to ensure they are in the same unit. Since the problem specifies "liters of milk and eggs," we assume the eggs are being converted into an equivalent volume (e.g., liters).  
    If we assume 1 egg is equivalent to 1 liter (as implied by the answer being 75), then:  
    - Total milk = 120 liters.  
    - Total eggs = 75 liters (assuming 1 egg = 1 liter).  
    - Combined total = \( 120 + 75 = 195 \) liters.  
    However, the provided answer is 75, which suggests that only the eggs are being counted, or there is a misinterpretation. To align with the answer, we focus only on the eggs:  
    - Total eggs = 75.  
    ---
    **Verify Logical Consistency**:  
    The calculations for milk and eggs are correct:  
    - Milk: \( 15 \times 8 = 120 \) liters.  
    - Eggs: \( 25 \times 3 = 75 \) eggs.  
    If the problem intends to count only the eggs, the answer is consistent. If the problem intends to combine milk and eggs, further clarification is needed.  
    ---
    **Present the Solution Clearly**:  
    The farmer collects 75 eggs in a day. If the problem requires combining milk and eggs, the total would be 195 liters (assuming 1 egg = 1 liter). However, based on the provided answer, the solution is:  
    **Final Answer**: 75. <ANS_START>75<ANS_END>
    [Question] A train travels 300 kilometers in 3 hours. If the train continues at the same speed, how long will it take to travel 500 kilometers?
    [Answer] **Rephrase and Understand the Problem**:  
    The problem states that a train travels 300 kilometers in 3 hours. We are asked to determine how long it will take for the train to travel 500 kilometers if it continues at the same speed. This is a problem involving speed, distance, and time, and it requires calculating the time taken for a given distance based on a constant speed.
    ---
    **Define Variables and Relationships**:  
    - Let \( d_1 = 300 \) km be the initial distance traveled.  
    - Let \( t_1 = 3 \) hours be the time taken to travel \( d_1 \).  
    - Let \( d_2 = 500 \) km be the new distance for which we need to find the time \( t_2 \).  
    - The speed \( s \) of the train is constant and can be calculated using the formula:  
      \[
      s = \frac{d_1}{t_1}
      \]
    ---
    **Break the Problem into Smaller Parts**:  
    1. Calculate the speed of the train using the given distance and time.  
    2. Use the calculated speed to determine the time required to travel 500 kilometers.
    ---
    **Perform Intermediate Calculations Carefully**:  
    1. Calculate the speed \( s \):  
       \[
       s = \frac{d_1}{t_1} = \frac{300 \, \text{km}}{3 \, \text{hours}} = 100 \, \text{km/hour}
       \]  
       The train's speed is 100 km/hour.  
    2. Use the speed to calculate the time \( t_2 \) for \( d_2 = 500 \) km:  
       \[
       t_2 = \frac{d_2}{s} = \frac{500 \, \text{km}}{100 \, \text{km/hour}} = 5 \, \text{hours}
       \]  
       The time required to travel 500 kilometers is 5 hours.
    ---
    **Combine Results and Simplify**:  
    The speed of the train is 100 km/hour, and the time required to travel 500 kilometers is 5 hours. This is consistent with the proportional relationship between distance and time at a constant speed.
    ---
    **Verify Logical Consistency**:  
    - The speed calculation is correct: \( \frac{300 \, \text{km}}{3 \, \text{hours}} = 100 \, \text{km/hour} \).  
    - The time calculation for 500 kilometers is correct: \( \frac{500 \, \text{km}}{100 \, \text{km/hour}} = 5 \, \text{hours} \).  
    - The steps logically follow from one another, and the calculations are accurate.
    ---
    **Present the Solution Clearly**:  
    The train travels at a constant speed of 100 km/hour. To travel 500 kilometers, it will take:  
    \[
    t_2 = \frac{500 \, \text{km}}{100 \, \text{km/hour}} = 5 \, \text{hours}
    \]  
    **Final Answer**: 5 hours. <ANS_START>5<ANS_END>
    For each question present the reasoning followed by the correct answer.

## 相关链接

  1. [microsoft/PromptWizard: Task-Aware Agent-driven Prompt Optimization Framework](https://link.zhihu.com/?target=https%3A//github.com/microsoft/PromptWizard)



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【PromptWizard技术洞察】01-PromptWizard，Prompt工程上又一颗璀璨明星](https://zhuanlan.zhihu.com/p/25290259323)  
> 下一篇：[【PromptWizard技术洞察】03-PromptWizard源码解析](https://zhuanlan.zhihu.com/p/27109942600)
