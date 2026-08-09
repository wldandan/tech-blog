# 大家做 Multi-Agent 时，是怎么做意图识别的？

> qid: 1907930479778300189
> slug: 2026-08-09-multi-agent-intent-routing

---

做了几年生产级 Multi-Agent 系统（witty-diagnosis-agent 故障诊断、AET 全流程 AI 研发底座），说说我的实际做法，以及踩过的坑。

意图识别在 Multi-Agent 里，本质就是个路由问题——把用户的话分给对的 Agent 处理。听起来不难，但生产里坑很深。

---

## 路由方案：没有银弹，看情况组合

### LLM 路由器

把每个 Agent 的边界写进 system prompt，让一个独立的 LLM 来裁判：

```python
ROUTER_PROMPT = """
你是一个 Multi-Agent 系统的调度器，根据用户输入选择最合适的 Agent：

- diagnosis_agent：处理系统故障诊断、日志分析、性能问题排查
- deploy_agent：处理服务部署、发版、回滚操作  
- monitor_agent：处理监控配置、告警规则、指标查询

只返回 agent 名称，不解释。
"""

def route(user_input: str) -> str:
    response = llm.chat(ROUTER_PROMPT, user_input)
    return response.strip()
```

Agent 数量少（10 个以内）、意图边界模糊的场景用这个没问题。但 Agent 一多，prompt 会越写越长，延迟也跟着涨，成本开始肉疼。

### Embedding + 向量检索

把每个 Skill 的描述向量化，用户输入也向量化，找最近邻：

```python
skill_embeddings = {
    skill_name: embed(skill_description)
    for skill_name, skill_description in skills.items()
}

def route(user_input: str) -> str:
    query_vec = embed(user_input)
    scores = {
        name: cosine_similarity(query_vec, vec)
        for name, vec in skill_embeddings.items()
    }
    return max(scores, key=scores.get)
```

Skill 数量上了 50+、对延迟敏感的场景，这个比 LLM 路由便宜快很多。坑在于：**描述写差了，召回就直接垮**。微软生产环境验证过，一次优化 Skill 描述能让误召回率下降 40%（来源：[Skill Collision 研究](https://zhuanlan.zhihu.com/p/2017640491264483370)），写描述这件事没想象中简单。

### 规则 + 关键词

对于边界清晰的高频意图，别迷信 LLM：

```python
RULES = [
    (["重启", "重新启动", "restart"], "ops_agent"),
    (["日志", "log", "报错", "error"], "diagnosis_agent"),
    (["部署", "发布", "上线", "deploy"], "deploy_agent"),
]

def route(user_input: str) -> str | None:
    for keywords, agent in RULES:
        if any(k in user_input for k in keywords):
            return agent
    return None  # 回退到下一层
```

零延迟、零成本、可解释，就是覆盖率有限。

我现在生产里用的是**三层瀑布**：规则先过一遍（命中率约 40%，不花钱）→ Embedding 兜底（再拿 40%）→ LLM 处理剩下模糊的 20%。大部分请求不用碰 LLM。

---

## 真正麻烦的几个问题

**意图模糊**是最常见的。用户说"帮我看看系统"，到底是查日志、看监控还是做诊断？

我在 witty-diagnosis-agent 里用的是假设-验证范式：先路由到最可能的 Agent，执行前做置信度评估，低于阈值就反问：

```python
if routing_confidence < 0.7:
    return clarify("你是想查【日志报错】还是【性能监控】？")
```

多问一句比跑错方向强。

**意图溢出**是另一个。"帮我分析昨天的故障原因，然后告诉我以后怎么避免"——这需要 diagnosis_agent 和 knowledge_agent 串联，不是路由到一个 Agent 就完事的。

这种情况，路由器输出的不是单个 Agent，而是一个执行计划（DAG），再交给编排器执行。SkillOrchestra 论文（[见这篇分析](https://zhuanlan.zhihu.com/p/2014393687812110022)）干的就是这事——把"路由哪个 Agent"扩展为"规划一条技能执行链"，性能比强化学习编排器最高提升了 22.5%。

**多轮里的意图漂移**也容易被忽视。"它报什么错" 这句话的路由完全取决于前文谈的是哪个系统，不带上下文就乱了。我的做法是路由器输入带上最近 3-5 轮的摘要，让路由有上下文感知。

---

## 路由建好了，还没完

这是很多团队容易犯的错：路由跑起来，以为就结束了。

路由系统需要持续运营：

```
Trace 记录每次路由决策
  → Observability 监控路由失败率和用户反馈
  → 定期从线上抽样 + 人工标注
  → 扩充 Benchmark
  → 发布新版前跑 Regression Test
  → 上线
  → 回到 Trace
```

witty-diagnosis-agent 每次升版前，我们都用 500+ 真实故障案例跑回归，确认新逻辑解决了旧问题、没带来新的误路由。这套流程是生产级和 Demo 级之间最大的差距，没有之一。

---

总结一下我的选择逻辑：Agent 少就直接上 LLM 路由器；Skill 多了换 Embedding 检索；高频明确的意图用规则兜底；生产级部署三层组合跑，再配上 Trace 持续迭代。

---

**延伸阅读**

- [Multi-Agent系统：协作、竞争与涌现](https://zhuanlan.zhihu.com/p/2003212333846118909)
- [Agent Skill 分析：如何实现技能的编排和评估](https://zhuanlan.zhihu.com/p/2014393687812110022)
- [Skill 描述优化实战：一次重写能降低多少误召回](https://zhuanlan.zhihu.com/p/2017640491264483370)
