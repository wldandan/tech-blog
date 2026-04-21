# 【Agent入门到精通】12-Multi-Agent系统：协作、竞争与涌现

原文链接：https://zhuanlan.zhihu.com/p/2003212333846118909

---

​

目录

## Multi-Agent系统：协作、竞争与涌现

> **本系列简介** ：这是一套系统性的AI Agent技术教程，覆盖从基础概念到生产级应用的完整知识体系。本文是系列的第12篇，全面更新至2026年最新实践。

**系列目录** ：

  1. [AI Agent的本质：从自动化到自主智能](https://zhuanlan.zhihu.com/p/1998733226689202164)
  2. [Agent架构设计：ReAct、ReWOO与思维链](https://zhuanlan.zhihu.com/p/1998817328603878485)
  3. [工具调用（Function Calling）：Agent的手和脚](https://zhuanlan.zhihu.com/p/1999201160813384313)
  4. [MCP协议深度解析：连接AI与数据源的标准化桥梁](https://zhuanlan.zhihu.com/p/1999203686707130939)
  5. [Workflow架构：可视化Agent编排平台](https://zhuanlan.zhihu.com/p/1999206497670947974)
  6. [Skills系统：Claude Code的模块化能力封装](https://zhuanlan.zhihu.com/p/1999207466928477997)
  7. [记忆系统：让Agent拥有上下文感知能力](https://zhuanlan.zhihu.com/p/2000654068553651951)
  8. [规划与推理：Agent如何分解复杂任务](https://zhuanlan.zhihu.com/p/2000654382052689377)
  9. [多模态Agent：视觉、语音与文本的融合](https://zhuanlan.zhihu.com/p/2000655295198823249)
  10. [上下文工程：Agent系统的”神经系统”](https://zhuanlan.zhihu.com/p/2003783359764112722)
  11. [Agent评估与优化：如何衡量Agent性能](https://zhuanlan.zhihu.com/p/2003203050169463280)
  12. [Multi-Agent系统：协作、竞争与涌现](https://zhuanlan.zhihu.com/p/2003212333846118909)
  13. [生产级Agent架构：可靠性、安全性与可观测性](https://zhuanlan.zhihu.com/p/2003761515438811109)
  14. [AI Agent的未来：AGI之路上的关键一步](https://zhuanlan.zhihu.com/p/2007391883593283178)



* * *

> 本文是《AI Agent系列教程》的第12篇（2026版），将深入探讨现代Multi-Agent系统的设计模式、协作机制、最新框架（[LangGraph Swarm](https://zhida.zhihu.com/search?content_id=270083842&content_type=Article&match_order=1&q=LangGraph+Swarm&zhida_source=entity)、CrewAI、AutoGen等）和实际应用案例，这是构建复杂AI系统的关键进阶技术。

## 上一篇回顾

在前面11篇文章中，我们学习了如何构建单个功能强大的Agent。但面对更复杂的任务，单个Agent往往力不从心：

  * **知识局限** ：单个Agent难以掌握所有领域的专业知识
  * **能力瓶颈** ：某些任务需要并行处理多个子任务
  * **可靠性问题** ：单点故障风险
  * **扩展性限制** ：难以处理超大规模任务



**Multi-Agent系统** 通过多个专业Agent协作，能够突破这些限制，实现1+1>2的效果。

## 引言：从单体到多体

### 单Agent vs Multi-Agent
    
    
    场景：构建一个智能数据分析系统
    
    单Agent方案：
    ┌─────────────────────────────────┐
    │         Data Analyst Agent       │
    │  - 数据清洗                      │
    │  - 统计分析                      │
    │  - 可视化                        │
    │  - 报告生成                      │
    │  - 领域知识（金融/医疗/...）      │
    └─────────────────────────────────┘
    问题：Agent需要掌握所有技能，复杂度高
    
    Multi-Agent方案：
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │Cleaner   │  │Analyzer  │  │Visualizer│
    │  Agent   │  │  Agent   │  │  Agent   │
    └────┬─────┘  └────┬─────┘  └────┬─────┘
         │             │             │
         └──────────┬──┴─────────────┘
                    │
             ┌──────┴──────┐
             │Coordinator  │
             │   Agent     │
             └─────────────┘
    优势：专业分工、并行执行、易于扩展

### Multi-Agent的核心价值

  1. **专业分工** ：每个Agent专注特定领域
  2. **并行处理** ：多个Agent同时工作
  3. **容错能力** ：单个Agent失败不影响整体
  4. **可扩展性** ：灵活添加新的专业Agent
  5. **涌现智能** ：协作产生超出个体的能力



## 一、Multi-Agent架构模式

### 1.1 架构分类
    
    
    ┌─────────────────────────────────────────┐
    │      Multi-Agent架构模式                  │
    ├─────────────────────────────────────────┤
    │  1. 层次式（Hierarchical）               │
    │     Manager-Agent模式                   │
    ├─────────────────────────────────────────┤
    │  2. 平面式（Flat）                      │
    │     对等协作                             │
    ├─────────────────────────────────────────┤
    │  3. 网络式（Network）                   │
    │     动态拓扑                             │
    ├─────────────────────────────────────────┤
    │  4. 竞争式（Competitive）               │
    │     多Agent竞争                          │
    └─────────────────────────────────────────┘

### 1.2 [层次式架构](https://zhida.zhihu.com/search?content_id=270083842&content_type=Article&match_order=1&q=%E5%B1%82%E6%AC%A1%E5%BC%8F%E6%9E%B6%E6%9E%84&zhida_source=entity)
    
    
    # 核心类定义（完整代码见GitHub仓库）
    from typing import List, Dict, Optional, Any
    from abc import ABC, abstractmethod
    from enum import Enum
    import asyncio
    
    class AgentRole(Enum):
        """Agent角色枚举"""
        MANAGER = "manager"
        WORKER = "worker"
        SPECIALIST = "specialist"
    
    class Message:
        """Agent间通信消息"""
        def __init__(self, sender: str, receiver: str, content: Any,
                     message_type: str = "task"):
            self.sender = sender
            self.receiver = receiver
            self.content = content
            self.message_type = message_type  # task, result, error, query等
            self.timestamp = time.time()
    
    class BaseAgent(ABC):
        """Agent基类 - 定义所有Agent的共同接口"""
        def __init__(self, name: str, role: AgentRole):
            self.name = name
            self.role = role
            self.inbox = asyncio.Queue()  # 接收消息队列
            self.outbox = asyncio.Queue()  # 发送消息队列
            self.running = False
    
        @abstractmethod
        async def process(self, message: Message) -> Optional[Message]:
            """处理消息的核心方法（子类必须实现）"""
            pass
    
        async def send(self, receiver: str, content: Any,
                       message_type: str = "task"):
            """发送消息到指定Agent"""
            message = Message(self.name, receiver, content, message_type)
            await self.outbox.put(message)
    
        async def run(self):
            """Agent主循环 - 持续处理消息"""
            self.running = True
            while self.running:
                message = await self.inbox.get()
                response = await self.process(message)
                if response:
                    await self.outbox.put(response)
    
    class ManagerAgent(BaseAgent):
        """管理Agent - 负责任务分配和协调"""
        def __init__(self, name: str, workers: List[str]):
            super().__init__(name, AgentRole.MANAGER)
            self.workers = workers  # 可用的工作Agent列表
            self.completed_tasks = {}  # 任务完成记录
    
        async def assign_task(self, task: Dict) -> str:
            """分配任务给合适的Worker"""
            worker_id = self._select_worker(task)  # 基于任务类型、负载等选择
            await self.send(worker_id, task, "task")
            return worker_id
    
        async def process(self, message: Message) -> Optional[Message]:
            """处理Worker返回的结果或错误"""
            if message.message_type == "result":
                task_id = message.content.get("task_id")
                self.completed_tasks[task_id] = message.content
                print(f"[Manager] 任务 {task_id} 完成")
            elif message.message_type == "error":
                print(f"[Manager] 错误：{message.content}")
            return None
    
    class WorkerAgent(BaseAgent):
        """工作Agent - 执行具体任务"""
        def __init__(self, name: str, specialty: str, skills: List[str]):
            super().__init__(name, AgentRole.WORKER)
            self.specialty = specialty  # 专业领域
            self.skills = skills        # 具备的技能
    
        async def process(self, message: Message) -> Optional[Message]:
            """处理任务并返回结果"""
            if message.message_type == "task":
                return await self._execute_task(message.content)
            return None
    
        async def _execute_task(self, task: Dict) -> Message:
            """执行具体任务逻辑"""
            task_id = task.get("id")
            task_type = task.get("type")
    
            try:
                # 实际业务逻辑（这里简化）
                result = f"完成 {task_type} 任务"
    
                return Message(
                    sender=self.name,
                    receiver=message.sender,
                    content={"task_id": task_id, "status": "completed", "result": result},
                    message_type="result"
                )
            except Exception as e:
                return Message(
                    sender=self.name,
                    receiver=message.sender,
                    content={"task_id": task_id, "status": "failed", "error": str(e)},
                    message_type="error"
                )
    
    class HierarchicalMultiAgentSystem:
        """层次化Multi-Agent系统"""
        def __init__(self):
            self.agents: Dict[str, BaseAgent] = {}
            self.message_router = None
    
        def add_agent(self, agent: BaseAgent):
            """添加Agent到系统"""
            self.agents[agent.name] = agent
    
        async def start(self):
            """启动所有Agent"""
            tasks = [asyncio.create_task(agent.run()) for agent in self.agents.values()]
            await asyncio.gather(*tasks)
    
    # 使用示例（简化）
    async def main():
        system = HierarchicalMultiAgentSystem()
    
        # 创建专业Agent
        cleaner = WorkerAgent("cleaner", "data_cleaning", ["clean", "normalize"])
        analyzer = WorkerAgent("analyzer", "analysis", ["statistics", "ml"])
        manager = ManagerAgent("manager", ["cleaner", "analyzer"])
    
        # 添加Agent并启动系统
        system.add_agent(cleaner)
        system.add_agent(analyzer)
        system.add_agent(manager)
    
        # 分配任务
        await manager.assign_task({
            "id": "task_001",
            "type": "clean",
            "data": "raw_data.csv"
        })
    
    if __name__ == "__main__":
        asyncio.run(main())

> **说明** ：以上是层次式架构的核心代码片段，完整实现（包括消息路由器、错误处理、负载均衡等）可在GitHub仓库查看：`ai-agent-tutorial-series/multi-agent/hierarchical.py`

### 1.3 [平面式协作](https://zhida.zhihu.com/search?content_id=270083842&content_type=Article&match_order=1&q=%E5%B9%B3%E9%9D%A2%E5%BC%8F%E5%8D%8F%E4%BD%9C&zhida_source=entity)架构
    
    
    class CollaborativeAgent(BaseAgent):
        """平面式协作Agent - 对等协作模式"""
        def __init__(self, name: str, expertise: List[str]):
            super().__init__(name, AgentRole.WORKER)
            self.expertise = expertise      # 专业技能领域
            self.peers = []                 # 协作伙伴列表
            self.shared_memory = {}         # 共享记忆（用于协作）
    
        async def collaborate(self, task: Dict) -> Dict:
            """协作工作流：分析→找伙伴→分配→执行→整合"""
            # 1. 分析任务需求
            required_skills = task.get("required_skills", ["general"])
    
            # 2. 寻找具备相关技能的伙伴
            collaborators = self._find_collaborators(required_skills)
    
            # 3. 分解任务并分配
            subtasks = self._decompose_task(task, collaborators)
    
            # 4. 并行执行（使用asyncio.gather）
            results = await self._execute_parallel(subtasks)
    
            # 5. 整合结果
            return self._integrate_results(results)
    
        def _find_collaborators(self, skills: List[str]) -> List['CollaborativeAgent']:
            """基于技能匹配寻找协作伙伴"""
            return [peer for peer in self.peers
                    if any(skill in peer.expertise for skill in skills)]
    
        async def _execute_parallel(self, subtasks: Dict) -> Dict:
            """并行执行子任务"""
            # 创建并行任务
            tasks = []
            for agent, subtask in subtasks.items():
                task = agent.process(Message(
                    sender=self.name,
                    receiver=agent.name,
                    content=subtask,
                    message_type="collaboration"
                ))
                tasks.append(task)
    
            # 等待所有任务完成
            results = await asyncio.gather(*tasks, return_exceptions=True)
            return dict(zip(subtasks.keys(), results))

> **说明** ：平面式架构中所有Agent地位平等，通过共享记忆和消息传递直接协作。完整实现见GitHub仓库。

### 1.4 [竞争式架构](https://zhida.zhihu.com/search?content_id=270083842&content_type=Article&match_order=1&q=%E7%AB%9E%E4%BA%89%E5%BC%8F%E6%9E%B6%E6%9E%84&zhida_source=entity)
    
    
    class CompetitiveAgent(BaseAgent):
        """竞争式Agent - 通过竞争机制完成任务"""
        def __init__(self, name: str, strategy: str):
            super().__init__(name, AgentRole.WORKER)
            self.strategy = strategy  # 竞争策略：speed/quality/balanced
            self.performance_score = 0.0  # 性能评分
            self.completed_tasks = []     # 完成任务记录
    
        async def compete(self, task: Dict) -> Dict:
            """根据策略执行任务竞争"""
            # 根据策略选择执行方式
            if self.strategy == "speed":
                return await self._fast_execution(task)
            elif self.strategy == "quality":
                return await self._quality_execution(task)
            else:
                return await self._balanced_execution(task)
    
        async def _fast_execution(self, task: Dict) -> Dict:
            """快速执行（侧重响应速度）"""
            start_time = time.time()
            await asyncio.sleep(0.5)  # 模拟快速处理
            return {
                "agent": self.name,
                "strategy": "speed",
                "execution_time": time.time() - start_time,
                "quality_score": 0.7,
                "result": "快速完成"
            }
    
        async def _quality_execution(self, task: Dict) -> Dict:
            """高质量执行（侧重结果质量）"""
            start_time = time.time()
            await asyncio.sleep(2.0)  # 模拟详细处理
            return {
                "agent": self.name,
                "strategy": "quality",
                "execution_time": time.time() - start_time,
                "quality_score": 0.95,
                "result": "高质量完成"
            }
    
    class CompetitiveArena:
        """竞争竞技场 - 管理多个Agent的竞争"""
        def __init__(self, agents: List[CompetitiveAgent]):
            self.agents = agents
            self.history = []  # 竞争历史记录
    
        async def run_competition(self, task: Dict) -> Dict:
            """运行一轮竞争"""
            print(f"🏁 开始竞争：{len(self.agents)}个Agent参与")
    
            # 并行执行所有Agent
            tasks = [agent.compete(task) for agent in self.agents]
            results = await asyncio.gather(*tasks)
    
            # 评估获胜者
            winner = self._evaluate_winner(results, task)
    
            # 更新分数
            for agent, result in zip(self.agents, results):
                if result["agent"] == winner["agent"]:
                    agent.performance_score += 1
    
            # 记录结果
            competition_result = {
                "task": task,
                "results": results,
                "winner": winner,
                "timestamp": time.time()
            }
            self.history.append(competition_result)
    
            print(f"🏆 获胜者：{winner['agent']}（策略：{winner['strategy']}）")
            return competition_result
    
        def _evaluate_winner(self, results: List[Dict], task: Dict) -> Dict:
            """根据任务优先级评估获胜者"""
            priority = task.get("priority", "balanced")
    
            if priority == "speed":
                return min(results, key=lambda r: r["execution_time"])
            elif priority == "quality":
                return max(results, key=lambda r: r["quality_score"])
            else:
                # 综合评分：质量70% + 速度30%
                for r in results:
                    r["final_score"] = r["quality_score"] * 0.7 + (1 / (r["execution_time"] + 1)) * 0.3
                return max(results, key=lambda r: r["final_score"])

> **说明** ：竞争式架构通过多个Agent并行执行相同任务，根据评估标准选择最优结果。适用于需要多样性和择优的场景。

## 二、协作模式

### 2.1 协作模式分类
    
    
    class CollaborationPatterns:
        """协作模式库 - 提供三种标准协作模式"""
    
        @staticmethod
        async def sequential(task: Dict, agents: List[BaseAgent]) -> Dict:
            """顺序协作：Agent流水线式处理"""
            result = task
            trace = []
    
            for agent in agents:
                # 创建消息传递给下一个Agent
                message = Message(
                    sender="system",
                    receiver=agent.name,
                    content=result,
                    message_type="collaboration"
                )
                response = await agent.process(message)
    
                if response:
                    result = response.content
                    trace.append({"agent": agent.name, "output": result})
    
            return {"final_result": result, "trace": trace}
    
        @staticmethod
        async def parallel(task: Dict, agents: List[BaseAgent]) -> Dict:
            """并行协作：多个Agent同时处理相同任务"""
            # 创建并行任务
            tasks = []
            for agent in agents:
                message = Message(
                    sender="system",
                    receiver=agent.name,
                    content=task,
                    message_type="collaboration"
                )
                tasks.append(agent.process(message))
    
            # 等待所有任务完成
            results = await asyncio.gather(*tasks, return_exceptions=True)
    
            # 合并结果（根据任务类型定制）
            valid_results = [r.content for r in results if hasattr(r, 'content')]
            return CollaborationPatterns._merge_results(valid_results)
    
        @staticmethod
        async def divide_and_conquer(task: Dict, agents: List[BaseAgent]) -> Dict:
            """分治协作：分解任务 → 并行处理 → 合并结果"""
            # 1. 分解任务
            subtasks = CollaborationPatterns._decompose_task(task, len(agents))
    
            # 2. 分配给各个Agent并行执行
            agent_tasks = []
            for agent, subtask in zip(agents, subtasks):
                message = Message(
                    sender="system",
                    receiver=agent.name,
                    content=subtask,
                    message_type="collaboration"
                )
                agent_tasks.append(agent.process(message))
    
            # 3. 等待所有子任务完成
            results = await asyncio.gather(*agent_tasks)
    
            # 4. 合并子任务结果
            valid_results = [r.content for r in results if hasattr(r, 'content')]
            return CollaborationPatterns._merge_results(valid_results)
    
        @staticmethod
        def _decompose_task(task: Dict, num_agents: int) -> List[Dict]:
            """任务分解策略（可根据业务逻辑定制）"""
            data = task.get("data", [])
            batch_size = max(1, len(data) // num_agents)
    
            subtasks = []
            for i in range(num_agents):
                start = i * batch_size
                end = start + batch_size if i < num_agents - 1 else len(data)
                subtasks.append({**task, "data": data[start:end]})
    
            return subtasks
    
        @staticmethod
        def _merge_results(results: List[Dict]) -> Dict:
            """结果合并策略（可根据业务逻辑定制）"""
            merged_data = []
            for result in results:
                if isinstance(result, dict) and "data" in result:
                    merged_data.extend(result["data"])
    
            return {"data": merged_data, "count": len(merged_data)}

> **说明** ：三种基础协作模式可组合使用，形成复杂的Multi-Agent工作流。完整实现见GitHub仓库。

## 三、实战案例：软件开发Multi-Agent系统
    
    
    class SoftwareDevelopmentTeam:
        """软件开发Multi-Agent团队 - 模拟真实开发流程"""
        def __init__(self):
            self.agents = {}
            self._setup_team()
    
        def _setup_team(self):
            """组建跨职能Agent团队"""
            self.agents["pm"] = ProductManagerAgent("pm")           # 产品经理
            self.agents["architect"] = ArchitectAgent("architect")   # 架构师
            self.agents["dev_frontend"] = DeveloperAgent("dev_frontend", "frontend")
            self.agents["dev_backend"] = DeveloperAgent("dev_backend", "backend")
            self.agents["tester"] = TesterAgent("tester")           # 测试工程师
            self.agents["reviewer"] = CodeReviewerAgent("reviewer") # 代码审查
    
        async def develop_feature(self, requirement: str) -> Dict:
            """端到端功能开发流程"""
            print(f"🎯 开始开发：{requirement}")
    
            # 1. 需求分析 → 2. 架构设计 → 3. 并行开发 → 4. 代码审查 → 5. 测试
            spec = await self.agents["pm"].analyze_requirement(requirement)
            architecture = await self.agents["architect"].design_architecture(spec)
    
            # 并行开发（前端、后端）
            dev_tasks = [
                ("frontend", self.agents["dev_frontend"].implement(architecture["frontend"], spec)),
                ("backend", self.agents["dev_backend"].implement(architecture["backend"], spec))
            ]
    
            development_results = {}
            for component, task in dev_tasks:
                development_results[component] = await task
    
            # 质量保障流程
            review_results = await self._conduct_reviews(development_results)
            test_results = await self.agents["tester"].test(development_results, spec)
    
            return {
                "specification": spec,
                "architecture": architecture,
                "implementation": development_results,
                "quality_check": {"reviews": review_results, "tests": test_results}
            }
    
        async def _conduct_reviews(self, implementations: Dict) -> Dict:
            """并行代码审查"""
            reviews = {}
            for component, code in implementations.items():
                reviews[component] = await self.agents["reviewer"].review(code, component)
            return reviews
    
    # 专业Agent实现（简化版）
    class ProductManagerAgent(BaseAgent):
        async def analyze_requirement(self, requirement: str) -> Dict:
            """需求分析 -> 产品规格"""
            return {
                "description": requirement,
                "user_stories": [f"作为用户，我需要{requirement}"],
                "acceptance_criteria": ["功能完整", "性能达标", "安全合规"]
            }
    
    class ArchitectAgent(BaseAgent):
        async def design_architecture(self, spec: Dict) -> Dict:
            """架构设计 -> 技术方案"""
            return {
                "pattern": "microservices",
                "frontend": {"framework": "React", "state_management": "Redux"},
                "backend": {"framework": "FastAPI", "database": "PostgreSQL"}
            }
    
    class DeveloperAgent(BaseAgent):
        def __init__(self, name: str, specialty: str):
            super().__init__(name, AgentRole.WORKER)
            self.specialty = specialty
    
        async def implement(self, design: Dict, spec: Dict) -> Dict:
            """代码实现"""
            await asyncio.sleep(1)  # 模拟开发时间
            return {
                "component": self.specialty,
                "status": "completed",
                "files": [f"{self.specialty}_main.py"],
                "summary": f"完成{self.specialty}模块开发"
            }
    
    class TesterAgent(BaseAgent):
        async def test(self, implementations: Dict, spec: Dict) -> Dict:
            """自动化测试"""
            results = {comp: {"passed": True, "coverage": 0.9}
                      for comp in implementations.keys()}
            return {
                "results": results,
                "all_passed": all(r["passed"] for r in results.values()),
                "summary": f"测试通过率：{len(results)}/{len(results)}"
            }
    
    class CodeReviewerAgent(BaseAgent):
        async def review(self, code: Dict, component: str) -> Dict:
            """代码质量审查"""
            return {
                "component": component,
                "approved": True,
                "score": 95,
                "comments": "代码结构清晰，符合规范"
            }
    
    # 使用示例
    async def main():
        team = SoftwareDevelopmentTeam()
        result = await team.develop_feature("用户登录系统")
    
        print("\n" + "="*50)
        print("开发完成总结：")
        print(f"📋 需求：{result['specification']['description']}")
        print(f"🏗️ 架构：{result['architecture']['pattern']}")
        print(f"✅ 组件：{list(result['implementation'].keys())}")
        print(f"🧪 测试：{result['quality_check']['tests']['summary']}")
    
    if __name__ == "__main__":
        asyncio.run(main())

> **说明** ：这是一个简化的软件开发Multi-Agent案例，展示了专业分工、并行协作和质量保障流程。完整实现（包含更多角色和详细逻辑）见GitHub仓库。

## 四、现代Multi-Agent框架

### 4.1 LangGraph Swarm：专业化Agent编排框架

LangGraph Swarm是LangChain生态中用于构建专业化Agent团队的框架，支持动态任务分配和协调。
    
    
    from langgraph_swarm import create_swarm
    from langchain_openai import ChatOpenAI
    from langgraph.prebuilt import create_react_agent
    
    # 1. 定义专业化Agent
    def create_specialist_agent(name: str, role: str, tools: list):
        """创建专业Agent模板"""
        return create_react_agent(
            ChatOpenAI(model="gpt-4o"),
            tools=tools,
            prompt=f"你是{role}专家，请专注完成相关任务。"
        )
    
    # 2. 构建Swarm团队
    async def build_content_creation_team():
        """构建内容创作团队"""
        research_agent = create_specialist_agent(
            "researcher", "研究", [DuckDuckGoSearchRun()]
        )
        writing_agent = create_specialist_agent("writer", "写作", [])
        analysis_agent = create_specialist_agent("analyst", "分析", [])
    
        # 3. 创建Swarm协调系统
        swarm = create_swarm(
            agents={
                "researcher": research_agent,
                "writer": writing_agent,
                "analyst": analysis_agent
            },
            coordinator_prompt="你负责协调研究、写作、分析专家完成内容创作任务。"
        )
        return swarm
    
    # 4. 使用Swarm执行任务
    async def create_article_with_swarm(topic: str):
        """使用Swarm团队创作文章"""
        swarm = await build_content_creation_team()
        result = await swarm.ainvoke({
            "messages": [("user", f"创作一篇关于{topic}的专业文章")]
        })
        return result["messages"][-1].content
    
    # 示例：生成AI趋势文章
    article = await create_article_with_swarm("2026年AI Agent发展趋势")
    print(f"生成文章：{article[:200]}...")

> **关键特性** ：  
> 

  * **专业化分工** ：每个Agent专注特定领域
  * **智能协调** ：Swarm自动分配任务给最适合的Agent
  * **状态管理** ：跟踪整个团队的工作进度
  * **灵活扩展** ：轻松添加新的专业Agent



  


完整示例见GitHub仓库。

### 4.2 Supervisor模式：分层任务协调

Supervisor模式采用”主管-专家”分层结构，是最成熟的Multi-Agent协作模式之一。
    
    
    from langgraph_supervisor import create_supervisor
    from langgraph.prebuilt import create_react_agent
    from langchain_openai import ChatOpenAI
    
    # 1. 创建专业化Agent团队
    def build_expert_team():
        """构建内容创作专家团队"""
        research_agent = create_react_agent(
            ChatOpenAI(model="gpt-4o"),
            tools=[search_tool],
            name="researcher",
            prompt="研究专家：负责信息检索和资料收集"
        )
    
        writing_agent = create_react_agent(
            ChatOpenAI(model="gpt-4o"),
            tools=[],
            name="writer",
            prompt="写作专家：负责内容创作和编辑优化"
        )
    
        review_agent = create_react_agent(
            ChatOpenAI(model="gpt-4o"),
            tools=[],
            name="reviewer",
            prompt="审核专家：负责质量评估和改进建议"
        )
    
        return [research_agent, writing_agent, review_agent]
    
    # 2. 创建Supervisor主管
    def create_content_supervisor(agents):
        """创建内容创作主管"""
        return create_supervisor(
            agents,
            model=ChatOpenAI(model="gpt-4o"),
            prompt="""
            你是内容创作团队主管，管理三个专家：
            1. researcher - 信息检索
            2. writer - 内容创作
            3. reviewer - 质量审核
    
            标准工作流程：
            研究 → 写作 → 审核 → 修改（如需）
    
            根据任务状态智能调度专家。
            """
        )
    
    # 3. 编译并运行工作流
    content_agents = build_expert_team()
    supervisor = create_content_supervisor(content_agents)
    app = supervisor.compile()
    
    # 4. 执行任务
    async def create_technical_article(topic: str):
        """创建技术文章工作流"""
        result = await app.ainvoke({
            "messages": [("user", f"写一篇关于{topic}的技术文章")]
        })
        return result["messages"][-1].content
    
    # 示例：创建量子计算文章
    article = await create_technical_article("量子计算基础原理")
    print(f"文章生成完成，长度：{len(article)}字符")

> **模式特点** ：  
> 

  * **明确分工** ：主管负责调度，专家负责执行
  * **流程可控** ：预设工作流程，确保任务有序完成
  * **质量保障** ：多阶段审核机制
  * **易于调试** ：清晰的责任边界和状态追踪



  


### 4.3 高级特性概览

现代Multi-Agent框架提供了多种高级特性，以下是关键特性的简要介绍：

### 4.3.1 共享记忆系统

多个Agent通过共享存储（如InMemoryStore）交换信息和上下文，确保协作过程的信息一致性。典型应用场景包括：

  * **研究-写作工作流** ：研究Agent收集资料存入共享记忆，写作Agent读取并创作
  * **多阶段审核** ：各阶段结果存入共享记忆，供后续阶段使用
  * **上下文传递** ：避免重复工作，提高协作效率



### 4.3.2 动态Agent创建

根据任务需求实时创建专业化Agent，实现资源的最优分配：

  * **技能分析** ：LLM分析任务所需技能组合
  * **动态配置** ：基于技能选择工具和提示词
  * **按需实例化** ：任务完成后可回收资源，适合临时性任务



### 4.3.3 Agent通信协议

标准化消息格式和通信机制确保Agent间可靠交互：

  * **消息队列** ：异步通信，支持优先级和确认机制
  * **广播机制** ：一对多消息分发
  * **错误处理** ：超时重试、死信队列等可靠性保障



### 4.4 2026年主流Multi-Agent框架对比

### 4.4.1 CrewAI：面向生产的Agent编排框架

CrewAI专注于企业级应用，提供清晰的角色定义和任务编排：
    
    
    from crewai import Agent, Task, Crew, Process
    
    # 定义专业化Agent
    researcher = Agent(
        role="市场研究专家",
        goal="分析AI Agent市场趋势",
        backstory="资深行业分析师",
        tools=[search_tool],
        verbose=True
    )
    
    writer = Agent(
        role="技术文档作家",
        goal="撰写专业分析报告",
        backstory="前科技记者，擅长技术内容创作"
    )
    
    # 创建任务流水线
    research_task = Task(
        description="收集2026年AI Agent市场数据",
        agent=researcher,
        expected_output="结构化市场分析数据"
    )
    
    writing_task = Task(
        description="基于研究数据撰写分析报告",
        agent=writer,
        expected_output="完整的市场分析报告"
    )
    
    # 组建团队并执行
    crew = Crew(
        agents=[researcher, writer],
        tasks=[research_task],
        process=Process.sequential  # 顺序执行
    )
    
    result = crew.kickoff()
    print(f"CrewAI执行结果：{result}")

### 4.4.2 Microsoft AutoGen：多Agent对话框架

AutoGen支持复杂的多轮对话和工具调用，适合需要深入交互的场景：
    
    
    from autogen import AssistantAgent, UserProxyAgent, GroupChat, GroupChatManager
    
    # 创建多个Agent
    engineer = AssistantAgent(
        name="engineer",
        system_message="软件工程师，负责代码实现"
    )
    
    scientist = AssistantAgent(
        name="scientist",
        system_message="数据科学家，负责算法设计"
    )
    
    product_manager = AssistantAgent(
        name="product_manager",
        system_message="产品经理，负责需求分析"
    )
    
    # 创建群聊协调
    group_chat = GroupChat(
        agents=[engineer, scientist, product_manager],
        messages=[],
        max_round=10
    )
    
    manager = GroupChatManager(groupchat=group_chat)
    
    # 启动协作会话
    user_proxy = UserProxyAgent(name="user_proxy")
    user_proxy.initiate_chat(
        manager,
        message="设计一个智能推荐系统，需要技术方案和实现计划"
    )

### 4.4.3 框架选择指南

框架| 适用场景| 核心优势| 学习曲线  
---|---|---|---  
LangGraph Swarm| 复杂专业化任务编排| LangChain生态集成，生产就绪| 中等  
CrewAI| 企业级工作流自动化| 角色定义清晰，任务管理完善| 低  
AutoGen| 研究型多轮对话| 对话协调能力强，灵活度高| 中等  
自定义框架| 特定领域需求| 完全控制，可深度定制| 高  
  
> **建议** ：新项目可从CrewAI开始，需要深度定制时考虑LangGraph，研究场景使用AutoGen。

## 五、总结

### 核心要点

  1. **Multi-Agent架构** ：层次式、平面式、网络式、竞争式
  2. **协作模式** ：顺序、并行、分治
  3. **通信机制** ：消息传递、共享内存
  4. **任务分配** ：基于能力、负载、策略
  5. **实际应用** ：软件开发、数据分析、内容创作



### 最佳实践

  * ✅ **明确分工** ：每个Agent有清晰的职责
  * ✅ **高效通信** ：优化消息传递机制
  * ✅ **容错设计** ：处理Agent失败
  * ✅ **动态调整** ：根据负载调整Agent数量
  * ✅ **监控调试** ：追踪Agent间交互



### 常见陷阱

  * ❌ **过度复杂** ：简单任务不需要Multi-Agent
  * ❌ **通信瓶颈** ：消息传递成为性能瓶颈
  * ❌ **死锁问题** ：Agent互相等待
  * ❌ **一致性差** ：Agent间信息不一致



* * *

## 推荐阅读

  * [AutoGen: Enabling Next-Gen LLM Applications](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/blog/autogen-enabling-next-gen-large-language-model-applications/)
  * [MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2308.00352)
  * [CAMEL: Communicative Agents for “Mind” Exploration](https://link.zhihu.com/?target=https%3A//www.camel-ai.org/)



## 关于本系列

这是《AI Agent系列教程》的第12篇（2026版），共14篇。

> **上一篇** ：[【Agent入门到精通】11-Agent评估与优化：现代框架与实践指南](https://zhuanlan.zhihu.com/p/2003203050169463280)  
> **下一篇** ：[【Agent入门到精通】13-生产级Agent架构：可靠性、安全性与可观测性](https://zhuanlan.zhihu.com/p/2003761515438811109)

* * *

 _如果这篇文章对你有帮助，欢迎点赞、收藏和分享！有任何问题欢迎在评论区讨论。_

* * *

**系列说明** ：

  * 本系列文章正在持续更新中，欢迎关注！
  * 所有代码示例将在GitHub仓库开源：`ai-agent-tutorial-series`
  * 有问题欢迎在评论区讨论，我会及时回复


