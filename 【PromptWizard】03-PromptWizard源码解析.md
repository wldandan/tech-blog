# 【PromptWizard】03-PromptWizard源码解析

原文链接：https://zhuanlan.zhihu.com/p/27109942600

---

​

目录

## 概述

[PromptWizard](https://zhida.zhihu.com/search?content_id=254444606&content_type=Article&match_order=1&q=PromptWizard&zhida_source=entity)为微软研究院提出的新型、完全自动化的离散提示词优化框架。PromptWizard利用自我进化、自我适应的机制，通过反馈驱动的批评和综合过程，在探索和开发之间实现了有效平衡，迭代地优化提示指令和上下文示例，生成可读、特定任务的提示。这种引导式方法系统地提高了提示质量，此外，成本分析显示API调用、Token使用和总成本大幅减少。本文将深入探讨PromptWizard的核心组件源码实现。

进入PromptWizard代码仓，核心代码存储在promptwizard目录中，目录结构如下：
    
    
    promptwizard/
    └── glue/
        ├── common/
        ├── paramlogger/
        └── promptopt/

目前promptwizard目录中的主要代码在glue目录中，glue目录中包含了 common、paramlogger、promptopt三个目录，分别对应了公共功能（如大模型调用、异常处理）、日志功能以及优化器。本文的讨论的重点为优化器的执行原理。

## 核心代码解析

当用户使用PromptWizard优化提示词时，需要编写如下代码并执行，在GluePromptOpt实例化时设置了大模型配置、优化参数配置、实验设置、训练文件、数据集处理器以及提示词池路径（可选），然后调用get_best_prompt()即可完成提示词的自动优化。
    
    
    gp = GluePromptOpt(args.llm_config_path,
                        args.promptopt_config_path,
                        args.setup_config_path,
                        args.train_file_name,
                        args.dataset_processor_pkl_path,
                        args.prompt_pool_path)
    
    best_prompt, expert_profile = gp.get_best_prompt()

因此GluePromptOpt为PromptWizard的核心功能类，主要包含四个成员函数（功能在注释中标记）其代码结构如下。
    
    
    class GluePromptOpt:
           """
        This class is trigger point for any prompt optimization method. Different prompt optimization techniques are
        represented by different classes. This class collates all the user configs present in different yaml files and
        other boilerplate code. Any of supported prompt optimization techniques can be triggered by this class.
        """
        BEST_PROMPT = None
        EXPERT_PROFILE = None
        data_processor = None
        iolog = [ParamLogger](https://zhida.zhihu.com/search?content_id=254444606&content_type=Article&match_order=1&q=ParamLogger&zhida_source=entity)()
        ...
        def __init__(self,
                     prompt_config_path: str,
                     setup_config_path: str,
                     dataset_jsonl: str,
                     data_processor: [DatasetSpecificProcessing](https://zhida.zhihu.com/search?content_id=254444606&content_type=Article&match_order=1&q=DatasetSpecificProcessing&zhida_source=entity),
                     dataset_processor_pkl_path: str = None,
                     prompt_pool_path: str = None):
            #  初始化函数，用于将所有优化相关配置导入
     
        def get_best_prompt(self,use_examples=False,run_without_train_examples=False,generate_synthetic_examples=False) -> (str, Any):
            ...
            #  提示词优化函数，包含提示词优化的全流程
        def evaluate(self, test_dataset_jsonl: str) -> float:
            ...
            #  
        @iolog.log_io_params
        def predict_and_access(self, question: str, gt_answer: str) -> (bool, str, str):
            ...
            #  预测函数，用于使用优化后的提示词对新问题进行预测

对于提示词优化流程，我们进一步查看 get_best_prompt函数。
    
    
    def get_best_prompt(self,use_examples=False,run_without_train_examples=False,generate_synthetic_examples=False) -> (str, Any):
        """
        Call get_best_prompt() method of class [PromptOptimizer](https://zhida.zhihu.com/search?content_id=254444606&content_type=Article&match_order=1&q=PromptOptimizer&zhida_source=entity) & return its value.
        :return: (best_prompt, expert_profile)
            best_prompt-> Best prompt for a given task description
            expert_profile-> Description of an expert who is apt to solve the task at hand. LLM would be asked to take
            identity of described in expert_profile.
        """
        start_time = time.time()
        self.BEST_PROMPT, self.EXPERT_PROFILE = self.prompt_opt.get_best_prompt(self.prompt_opt_param,use_examples=use_examples,run_without_train_examples=run_without_train_examples,generate_synthetic_examples=generate_synthetic_examples)
    
        self.logger.info(f"Time taken to find best prompt: {(time.time() - start_time)} sec")
        return self.BEST_PROMPT, self.EXPERT_PROFILE

核心逻辑调用了self.promtp_opt 变量中的get_best_prompt函数生成 BEST_PROMPT 和 EXPERT_PROFILE，其中self.prompt_opt 为优化器类的实例，在初始化函数中创建，目前PromptWizard仅提供了一种优化器类，即[CritiqueNRefine](https://zhida.zhihu.com/search?content_id=254444606&content_type=Article&match_order=1&q=CritiqueNRefine&zhida_source=entity)类。

进一步查看CritiqueNRefine类，他是由__init__、get_best_prompt、等大量的其他函数实现。
    
    
    class CritiqueNRefine(PromptOptimizer, UniversalBaseClass):
        def __init__(self, dataset: List, base_path: str, setup_config: SetupConfig,
                     prompt_pool: CritiqueNRefinePromptPool, data_processor: DatasetSpecificProcessing, logger):
            self.dataset = dataset
            self.setup_config = setup_config
            self.data_processor = data_processor
            self.logger = logger
            self.prompt_pool = prompt_pool
            base_path = join(base_path, LogLiterals.DIR_NAME)
            self.iolog.reset_eval_glue(base_path)
     
        def get_best_prompt(self, params: PromptOptimizationParams,use_examples=False,run_without_train_examples=False,generate_synthetic_examples=False) -> (str, Any):
            ...

其中，__init__ 函数中定义了 数据集、实验设置、数据集处理器、模板池等，其中模板池是用来放置优化工程中调用大模型所需要的提示词模板，默认的模板池文件路径为 promptwizard/glue/promptopt/techniques/critique_n_refine/prompt_pool.yaml，里面包含大量提示词模板。

get_best_prompt 函数逻辑如下面代码及图1所示，代码逻辑清晰，每一个步骤都是获取相关内容后，利用self.prompt_pool中的对应模板填充，填充后请求大模型完成的。
    
    
        def get_best_prompt(self, params: PromptOptimizationParams,use_examples=False,run_without_train_examples=False,generate_synthetic_examples=False) -> (str, Any):
            """
            Perform `params.max_iterations` iterations for optimizing your prompt. And return the best prompt found so far.
    
            :params: Object of class PromptOptimizationParams, that has all hyper-parameters needed for prompt optimization.
            :return: Best prompt for the given task and dataset.
            """
            #  let think step by step 初始的 instruction
            current_base_instruction = params.base_instruction
            #  判断是不是要生成examples
            if not generate_synthetic_examples:
                #  实际的生成提示词的逻辑
                print("\nMutating Task Description....")
                #  Mutate and refine task description
                #  一共迭代多少次
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
                        #  如果没有exmaple则不产生best提示词
                        return "",""
                    #  如果是有 example的 往下走
                    #  给prompt 打分， list 每个元素是一个三元组
                    prompt_score_list = self.get_prompt_score(candidate_prompts, params)
                    #  选得分topn 个
                    prompt_score_list = self.select_top_prompts(prompt_score_list, params.top_n)
                    #  如果要refine instruction
                    if params.refine_instruction:
                        #  评论 + 生成
                        refined_prompts = self.refine_prompts(prompt_score_list, params)
                        #  再验证一次 这块可以看一下分数是否有提升
                        refined_prompt_score_list = self.get_prompt_score(refined_prompts, params)
                        prompt_score_list = self.select_top_prompts(refined_prompt_score_list + prompt_score_list,
                                                                    params.top_n)
                    #  提示词
                    current_base_instruction = prompt_score_list[0][self.GetPromptScoreIndex.PROMPT_STR]
                    self.iolog.append_dict_to_chained_logs({"round_num": round_num,
                                                            "best_prompt": current_base_instruction,
                                                            "score": prompt_score_list[0][self.GetPromptScoreIndex.SCORE]
                                                            })
    
                examples = []
                #  更新 base_instruction, best prompt的第一部分完成
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
                    examples = random.sample(self.dataset, params.few_shot_count - len(examples))
    
                #  Refine task description and examples iteratively
                print("\nRefining Task description and Examples iteratively....")
                for i in tqdm(range(params.refine_task_eg_iterations)):
                    refine_task_desc = random.choice([True, False])
                    if refine_task_desc:
                        #  带着示例 再一次更新  base instruction
                        refined_instruction = self.get_best_instr_by_critique(examples, params)
                        if refined_instruction:
                            #  更新
                            params.base_instruction = refined_instruction
                    #  comment this to turn off synthetic examples
                    elif use_examples:
                        #  生成示例
                            examples = self.generate_best_examples(examples, params)
            #  如果生成示例的话，就下面这个else
            else:
                print("Generating Sythetic Examples....")
                #  生成之后返回
                train_examples = self.generate_best_examples_zero_shot(params)
                with open("train_synthetic.jsonl", 'w') as file:
                    for record in train_examples:
                        json.dump(record, file)
                        file.write('\n')
    
                print("Synthetic examples saved at train.jsonl....")
                return "",""
     
            #  是否生成 推理，推理指的是得到答案的推理逻辑
            if params.generate_reasoning:
                print("\nGenerating CoT Reasoning for In-Context Examples....")
                for example in tqdm(examples):
                    reason = self.generate_reasoning(params.task_description,
                                                     params.base_instruction,
                                                     example[DatasetSpecificProcessing.QUESTION_LITERAL],
                                                     example[DatasetSpecificProcessing.FINAL_ANSWER_LITERAL])
    
                    example[DatasetSpecificProcessing.ANSWER_WITH_REASON_LITERAL] = f"{reason} " + \
                                                                                    f"{DatasetSpecificProcessing.ANSWER_START}" + \
                                                                                    f"{example[DatasetSpecificProcessing.FINAL_ANSWER_LITERAL]}" + \
                                                                                    f"{DatasetSpecificProcessing.ANSWER_END}"
            if self.data_processor != None:
                #  self.prompt_pool.quest_reason_ans 这是一个模板，把 examples 填到这个模板里面
                example_string = self.data_processor.collate_to_str(examples, self.prompt_pool.quest_reason_ans)
            else:
                example_string = ""
                for example in examples:
                    answer = example[DatasetSpecificProcessing.FINAL_ANSWER_LITERAL]
                    if DatasetSpecificProcessing.ANSWER_WITH_REASON_LITERAL in example:
                        answer = example[DatasetSpecificProcessing.ANSWER_WITH_REASON_LITERAL]
    
                    example_string += self.prompt_pool.quest_reason_ans.format(question=example[DatasetSpecificProcessing.QUESTION_LITERAL],
                                                                answer=answer)
            #  生成最终的提示词，分为有example 和 没有 example
            if params.few_shot_count == 0:
                final_best_prompt = self.prompt_pool.final_prompt.format(
                instruction=params.base_instruction,
                answer_format=params.answer_format,
                few_shot_examples="")
            else:
                final_best_prompt = self.prompt_pool.final_prompt.format(
                    instruction=params.base_instruction,
                    answer_format=params.answer_format,
                    few_shot_examples=example_string)
    
            expert_identity = self.prompt_pool.system_prompt
            if params.generate_expert_identity:
                print("\nGenerating Expert Identity....")
                expert_identity = self.generate_expert_identity(params.task_description)
                self.logger.info(f"Expert Identity: {expert_identity}")
    
            #  如果需要生成关键词 就加到 提示词的最下面
            if params.generate_intent_keywords:
                print("\nGenerating Intent Keywords....")
                intent_keywords = self.generate_intent_keywords(params.task_description,
                                                                params.base_instruction)
    
                final_best_prompt += "Keywords: " + intent_keywords
     
            self.iolog.dump_chained_log_to_file("best_prompt")
            self.logger.info(f"Final best prompt: {final_best_prompt}")
    
            return final_best_prompt, expert_identity

图1 PromptWizard优化流程图

结合图1可以看出，get_best_prompt函数会根据输入的任务描述等信息（params变量中）输入到 iterative Refinement Of Prompt Instructions 模块中，对应到代码中第11-68行判断逻辑内的流程（代码解释见注释）。该模块通过迭代的方式优化提示词中的指令部分，首先生成多种角度的thinking styles，然后利用不同的thinking style生成正对当前任务的多个提示词指令变种，让每个变种指令请求大模型，并评估当前指令的分支，通过评论的方式给出指令优化建议，可参考图2流程。

图2 Iterative Refinement of Prompt Instruction 流程

如图1，经过指令迭代优化后，并列有一个Diverse Examples Selection 模块，然后优化的指令 和 Example示例同时输入到第3部分Sequential Optimization中，其中 Diverse Examples Selection 模块代码为上述get_best_prompt函数中第69-86行，其主要的目标是找出利用提示词指令预测失败的示例，并达到目标数量。

接下来进入第三部分（Sequential Optimization），这一部分的目标是在引入示例后，通过迭代的方式，进一步用户提示词指令和示例（注意 示例和指令都会被优化）。优化流程对应上述 get_best_prompt代码中第90-101 行，从代码中看出每次迭代会先更新指令，再更新示例。

最后，进入第四部分（self-generated reasoning and validation），这一步的主要内容在 第116-127行，通过 self.generate_reasoning 函数生成示例的推理过程。

至此，最终提示词所需要的内容均已完成，进入到提示词组装步骤，参考 上述 get_best_prompt 函数第141-169行，返回值为 final_best_prompt 和 expert_identity，其中 final_best_prompt 由 指令、示例（问题、答案、推理内容）、格式化输出、关键词拼接组成。 expert_indenty由self.generate_expert_identity 函数生成。

提示词优化的可视化步骤如图3 所示。

图3 提示词优化流程可视化

那么，expert_indentiy 和 final_best_prompt最终如何使用呢？

如 GluePromptOpt 类的predict_and_access 函数所示，在最终预测时，输入的问题（question）和BEST_PROMPT（即final_best_prompt）会利用eval_prompt进行组装为final_prompt，而final_prompt和EXPERT_PROFILE（expert_identity）作为系统提示词和 用户提示词请求大模型得到预测结果。
    
    
    class GluePromptOpt:
    
        @iolog.log_io_params
        def predict_and_access(self, question: str, gt_answer: str) -> (bool, str, str):
            """
            For the given input question, get answer to it from LLM, using the BEST_PROMPT & EXPERT_PROFILE
            computes earlier.
    
            :param question: Question to be asked to LLM, to solve
            :param gt_answer: Ground truth, final answer.
            :return:  (is_correct, predicted_ans, llm_output)
                    is_correct -> Tells if prediction by LLM was correct.
                    predicted_ans -> is the actual predicted answer by LLM.
                    llm_output -> Output text generated by LLM for the given question
            :rtype: (bool, str, str)
            """
            final_prompt = self.prompt_pool.eval_prompt.format(instruction=self.BEST_PROMPT,
                                                               question=question)
            llm_output = self.prompt_opt.chat_completion(user_prompt=final_prompt, system_prompt=self.EXPERT_PROFILE)
     

## 总结

本文介绍了PromptWizard提示词优化框架的核心优化代码逻辑，核心代码主要是实现了 CritiqueNRefine 类，类中通过迭代的方式优化提示词，提示词的返回主要分为两个部分，分别是 final_best_prompt,和 expert_identity， 其中 final_best_prompt 包含由 指令、示例（问题、答案、推理内容）、格式化输出、关键词拼接组成，并且指令、示例、关键词都是经过优化得到；expert_identity 主要是当前任务的角色描述，也是通过访问大模型优化得到。在提示词各部分的优化过程中，都使用了 模板池（prompt_pool）中的模板，在流程中填入对应内容并请求大模型得到优化过程中的结果。

## 相关链接

  1. PromptWizar GitHub： _[https:// github.com/microsoft/PromptWizard](https://link.zhihu.com/?target=https%3A//github.com/microsoft/PromptWizard)_



> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【PromptWizard技术洞察】02-Prompt又一选择，从0-1快速上手PromptWizard](https://zhuanlan.zhihu.com/p/26154065107)  
> 下一篇：[【PromptWizard技术洞察】04-PromptWizard和DSPy上手实操案例&实验数据对比](https://zhuanlan.zhihu.com/p/27861853778)
