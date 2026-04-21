# 【Agent实践】05-构建多Agent系统的8个最佳实践

原文链接：https://zhuanlan.zhihu.com/p/1954596883117871493

---

​

目录

在生成式人工智能时代，[多智能体系统](https://zhida.zhihu.com/search?content_id=263498109&content_type=Article&match_order=1&q=%E5%A4%9A%E6%99%BA%E8%83%BD%E4%BD%93%E7%B3%BB%E7%BB%9F&zhida_source=entity)是最具前景的架构之一。从个人辅助工具到复杂的企业自动化系统，正迅速超越单一查询聊天机器人，迈向由智能体组成的智能网络 —— 每个智能体都有自己的角色、工具和目标。但在构建多智能体系统时，设计多个智能体很简单，让它们高效且安全的协作却是一大挑战。

本文是基于实战提炼了8条最佳实践，包含可立即应用的代码模式、架构建议与[可观测性](https://zhida.zhihu.com/search?content_id=263498109&content_type=Article&match_order=1&q=%E5%8F%AF%E8%A7%82%E6%B5%8B%E6%80%A7&zhida_source=entity)实践。

## 什么是多智能体系统？

首先，回顾一下什么是多智能体系统（MAS）。多智能体系统是由独立[LLM智能体](https://zhida.zhihu.com/search?content_id=263498109&content_type=Article&match_order=1&q=LLM%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity)组成的网络，其中每个智能体通常：

  * 有明确的角色和目标
  * 使用记忆或上下文进行操作
  * 能调用一个或多个工具/API
  * 可按序列、并行或通过协商循环工作
  * 协作解决复杂的多步任务



后文，逐一对8条实践进行介绍。

## 1\. 明确分配角色与职责

智能体应该像一个跨职能团队一样工作：有明确界定的范围，职责不重复，并具备优雅的回退行为。

✅正面示例：
    
    
    {
    "agent": "Researcher",
    "goal": "Find 3 relevant academic papers",
    "tools": ["WebSearch", "PDFReader"],
    "memory": "query intent, keywords"
    }

❌反面示例：两个智能体同时尝试规划并执行任务，易互相覆盖或相互矛盾。

建议：在每个智能体的配置中加入一行系统提示，例如：“你是规划者。请勿执行或验证，只创建步骤。”

## 2\. 保持记忆本地化，而非全局化

并非所有智能体都需要完整的对话或执行历史。因为，完整的上下文 = token过载 + 延迟上升 + 推理混乱。

✅示例：
    
    
    # Instead of injecting full session:
    context = history.get_last_steps(agent="Researcher", max_tokens=300)
    
    # Or create summarized inputs
    context = summarize_results(last_3_documents)

## 3\. 明确的处理工具访问权限

并非每个智能体都需要访问所有工具。

最佳实践：

  * 仅为真正需要的智能体赋予相应工具的使用权限。
  * 记录工具输出，并只将相关内容提供给下一个智能体。
  * 使用工具调用日志以提高可解释性。



✅示例：只有 “Implementer” 代理应调用CodeExecutor，而非Planner或Validator。

## 4\. 预先设置终止条件

智能体容易陷入循环。必须在拖垮整个系统之前阻止它们。

✅正面示例：
    
    
    MAX_RETRIES = 3
    if iteration > MAX_RETRIES or confidence > 0.9:
        break

❌反面示例：
    
    
    while True:
        result = agent.run()

## 5\. 记录一切，然后迭代

没有结构化可观测性，调试多智能体系统将非常痛苦。

✅ 日志示例：
    
    
    {
      "agent": "Validator",
      "input": "Generated JSON schema",
      "tool_used": "SchemaLinter",
      "output": "2 critical schema mismatches",
      "confidence": 0.78,
      "duration_ms": 2200,
      "next_action": "Retry"
    }

构建轻量级仪表盘（例如Streamlit或[Supabase UI](https://zhida.zhihu.com/search?content_id=263498109&content_type=Article&match_order=1&q=Supabase+UI&zhida_source=entity)）以便日志回放与检查。

## 6\. 优先考虑可中断性和安全性

用户需要能够暂停、检查或重定向智能体，尤其是在处理自动化时。

应构建的UI控件：

  * 暂停执行（Pause Execution）
  * 编辑计划（Edit Plan）
  * 重新运行智能体（Rerun Agent）
  * 覆盖工具输入（Override Tool Input）
  * 回滚到上一个安全状态（Rollback）



✅ 示例：在UI中展示智能体状态和临时记录。
    
    
    {
      "step": "3/5",
      "current_agent": "Researcher",
      "thought": "Still missing one source from a verified domain",
      "plan": ["Find sources", "Summarize", "Send email"]
    }

## 7\. 运行时使行为可配置

避免对智能体参数（如温度、角色描述或允许的工具）进行硬编码。

✅ 示例：
    
    
    Planner:
      temperature: 0.3
      system_prompt: "Break down tasks clearly"
      tools: ["TaskListGenerator", "DependencyChecker"]

构建管理面板或YAML加载器，允许非开发人员在无需更改代码的情况下调整智能体的性能。

## 8\. 对所有内容进行版本控制

多智能体系统是动态的。你会：

  * 修改提示词模板
  * 更新工具schema
  * 调整记忆窗口
  * 优化决策逻辑



没有版本控制，系统会悄然崩溃。

最佳实践示例：

  * 使用[语义化版本号](https://zhida.zhihu.com/search?content_id=263498109&content_type=Article&match_order=1&q=%E8%AF%AD%E4%B9%89%E5%8C%96%E7%89%88%E6%9C%AC%E5%8F%B7&zhida_source=entity)（如Planner v1.2.3）
  * 记录每次运行的配置哈希值（config hash）
  * 在测试用例中存储智能体的输入/输出示例（类似pytest fixtures）
  * 将智能体视作API：保持稳定、可测试并有版本控制



## 相关链接

> 技术全景图：[大模型应用&Agent应用开发技术栈-全景图](https://zhuanlan.zhihu.com/p/694428893)  
> 上一篇：[【Agent实践】04-为什么大多数AI Agent会失败](https://zhuanlan.zhihu.com/p/1952099345617908257)
