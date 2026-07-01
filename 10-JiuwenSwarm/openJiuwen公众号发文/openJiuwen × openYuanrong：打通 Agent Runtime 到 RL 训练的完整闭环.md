# openJiuwen × openYuanrong：打通 Agent Runtime 到 RL 训练的完整闭环
openJiuwen104次阅读创建于2026-04-08更新于2026-06-08
图片加载失败

今天我们想分享一个社区重要进展：
**openJiuwen联合openYuanrong ，打通了 openJiuwen Agent Runtime、openYuanrong、RL 训练框架（verl）与昇腾算力之间的全链路。**
这意味着：
Agent 每次真实运行产生的轨迹，不再只是日志；
它会被结构化为可学习样本；
直接进入 GRPO 训练循环；
在昇腾环境上完成推理和训练；
一句话：**从“运行轨迹”到“策略更新”，现在是一条可自动化的闭环路径。**
# 1. 以前为什么难：Agent Runtime、RL常常是割裂的
传统流程里：
Runtime 负责执行任务（含工具调用、多轮交互）；
Trainer 只看静态文本样本；
中间缺少稳定、标准、可复用的桥接层。
结果就是：

训练需要模拟Agent运行环境，与Agent行为脱节；


轨迹数据格式不统一，难直接训练；


多轮决策价值无法体现在奖励里；

# 2. 这个链路做了什么？
### openJiuwen：把 Runtime 原生轨迹变成 RL 可消费数据
轨迹采样：完整采集轨迹并采样高质量数据，构建Rollout数据格式
奖励分配：集成reward函数，对轨迹进行奖励的生成与轨迹中步骤的信用分配
任务管理：任务队列实现任务管理，完成训练batch组装
### openYuanrong提供高性能高可靠RL分布式后端
openYuanrong作为verl分布式计算后端，在昇腾上提供高性能、高资源利用率竞争力
结合硬件实现高性能D2D/D2H/H2D/H2H传输：作为verl中关键组件Transfer Queue分布式存储后端，提升样本传输性能3~4倍
异构分布式多级缓存：支持HBM/DRAM/SSD多级缓存，零冗余拷贝跨节点传输，提供高性能KVC访问能力，提升KVC缓存命中率
实例快速弹性伸缩：使用NPU参数面数据快速传输能力，实现LLM模型实例秒级扩容
异构函数自适应动态调度能力：实现实例自动休眠唤醒，提升RL任务NPU资源利用率
### 如何使用昇腾算力
昇腾为Agent强化学习训练提供底层算力基座，依托 torch-npu、vLLM-Ascend等昇腾生态独有特性实现了强化学习训推一体的深度优化。
在Rollout阶段，昇腾通过 vLLM-Ascend 推理加速插件为Agent提供高效推理服务。
在模型参数更新阶段，昇腾依托 torch-npu 提供的NPU设备抽象层，无缝对接PyTorch原生 FSDP 分布式训练框架。
整体架构图：
图片加载失败
# 3. 验证场景：以计算器任务为例
为了验证“Agent 运行轨迹可直接用于强化学习训练”这条链路，我们选择了一个可控、可复现实验场景：计算器工具调用任务。
在该场景中，模型需要根据用户问题自主决定是否调用计算器工具、如何组织多步计算、以及如何输出最终答案；奖励函数同时考察最终结果正确性、工具调用有效性和轨迹质量。
从训练结果看，随着 rollout 迭代推进，reward 曲线稳定上升，说明 Runtime 轨迹 -> 奖励 -> GRPO 更新的链路在昇腾环境上可以稳定工作。
**reward曲线**
图片加载失败
# 4. 快速上手
基于openJiuwen ReAct Agent完成上述端到端训练任务：
from openjiuwen.core.foundation.prompt import PromptTemplate
from openjiuwen.core.foundation.tool import tool
from openjiuwen.core.common.logging import logger
from openjiuwen.dev_tools.agentrl.coordinator.schemas import RolloutMessage
from openjiuwen.dev_tools.agentrl import RLConfig, RLOptimizer
from openjiuwen.dev_tools.agentrl.config.schemas import (
    AdaConfig,
    AgentRuntimeConfig,
    PersistenceConfig,
    RolloutConfig,
    TrainingConfig,
)

#Step 1: 训练参数配置
config = RLConfig(
    training=TrainingConfig(...), # 训练配置
    rollout=RolloutConfig(...), # Rollout配置
    runtime=AgentRuntimeConfig(...), # Agent运行时配置
    persistence=PersistenceConfig(...), # 训练关键数据保存配置
)

#Step 2：系统提示词设置
CALCULATOR_SYSTEM_PROMPT = PromptTemplate(
    name="calculator_system",
    content=(
        "You are a {{role}}. Use the {{tool_name}} tool to solve "
        "{{task_type}} problems step by step.\n"
        "Output the answer when you are ready. "
        "The answer should be surrounded by three sharps (`###`), "
        "in the form of {{answer_format}}."
    ),
)
system_prompt = CALCULATOR_SYSTEM_PROMPT.format(
    keywords={
        "role": "calculator assistant",
        "tool_name": "calculator",
        "task_type": "math",
        "answer_format": "### ANSWER: <answer> ###",
    }
).content

#Step 3：工具定义
@tool(
    name="calculator",
    description="Perform arithmetic calculations, simplify algebraic expressions, and solve equations.",
)
def calculator(expression: str) -> str:
    """计算器工具，支持表达式计算、方程求解"""
    ...
    return result

#Step 4: 数据集预处理函数定义
def task_data_fn(task_sample: dict) -> dict:
    """将数据集行转为 Agent 输入格式。"""
    ...
    return {
        "query": task_sample.get("question", ""),
        "ground_truth": task_sample.get("result", ""),
    }

#Step 5：奖励函数定义
def calc_reward(msg: RolloutMessage) -> dict:
    """定义奖励函数，例如答案正确返回 1.0，否则返回 0.0，
    返回多轮对话细粒度的奖励列表reward_list和全局奖励值global_reward。"""
    ...
    return {"reward_list": reward_list, "global_reward": global_reward}

#Step 6：优化器实例化与训练
optimizer = RLOptimizer(config)
optimizer.register_reward(calc_reward, name="calc_reward")
optimizer.set_tools([calculator])
optimizer.set_task_data_fn(task_data_fn)
try:
    optimizer.train()
except KeyboardInterrupt:
    logger.info("Training interrupted by user")
finally:
    optimizer.stop()
    logger.info("Training complete")
# 5. 这条打通路径的价值
### 价值一：真实行为可学习
模型不再只学习“怎么回答”，而是学习“怎么行动”：
何时调用工具；
多轮中如何纠错；
如何在环境反馈下调整策略。
### 价值二：训练与线上一致
线上 Agent 的执行轨迹可以直接进入训练管道，形成数据闭环。
评估、优化、发布不再断层。
Agent 的下一阶段，不是“再多几个 prompt 技巧”，而是让 Agent 具备持续进化能力。
而持续进化的核心是：**让真实运行行为进入可优化的学习闭环。**
