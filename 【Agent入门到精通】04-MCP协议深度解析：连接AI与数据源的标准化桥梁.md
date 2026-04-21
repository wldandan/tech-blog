# 【Agent入门到精通】04-MCP协议深度解析：连接AI与数据源的标准化桥梁

原文链接：https://zhuanlan.zhihu.com/p/1999203686707130939

---

​

目录

> **本系列简介** ：这是一套系统性的AI Agent技术教程，覆盖从基础概念到生产级应用的完整知识体系。本文是系列的第4篇。

**系列目录** ：

  1. [AI Agent的本质：从自动化到自主智能](https://zhuanlan.zhihu.com/p/1998733226689202164)
  2. [Agent架构设计：ReAct、ReWOO与思维链](https://zhuanlan.zhihu.com/p/1998817328603878485)
  3. [工具调用（Function Calling）：Agent的手和脚](https://zhuanlan.zhihu.com/p/1999201160813384313)
  4. [[MCP协议](https://zhida.zhihu.com/search?content_id=269598940&content_type=Article&match_order=1&q=MCP%E5%8D%8F%E8%AE%AE&zhida_source=entity)深度解析：连接AI与数据源的标准化桥梁](https://zhuanlan.zhihu.com/p/1999203686707130939)
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

> 本文是《AI Agent系列教程》的第4篇，将深入解析MCP（Model Context Protocol）协议，这是[Anthropic](https://zhida.zhihu.com/search?content_id=269598940&content_type=Article&match_order=1&q=Anthropic&zhida_source=entity)推出的开放协议，用于标准化AI应用与外部数据源、工具的连接。

## 上一篇回顾

在第3篇《工具调用》中，我们学习了如何用Markdown封装AI能力。但当Agent需要连接大量外部数据源时，如何标准化这些连接呢？

**每个工具都要单独集成、数据格式不统一、上下文难以共享。**

这就是**MCP协议** 要解决的问题。

## 引言：为什么需要标准化数据连接？

### 当前AI开发的数据访问困境
    
    
    现状：
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ AI应用A  │  │ AI应用B  │  │ AI应用C  │
    │(Claude)  │  │ (自定义)  │  │ (GPT)    │
    └────┬─────┘  └────┬─────┘  └────┬─────┘
         │             │             │
         │各自不同的数据连接方式        │
         │             │             │
    ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐
    │数据库驱动  │  │ API客户端  │  │ 文件读取  │
    └──────────┘  └──────────┘  └──────────┘
    
    问题：
    - 每个数据源需要单独编写连接代码
    - 数据格式不统一，转换成本高
    - 上下文难以在不同应用间共享
    - 安全和权限管理各自为政
    
    MCP的目标：
    ┌─────────────────────────────────────┐
    │         MCP协议层                    │
    └──────┬──────────────┬───────────────┘
           │              │
    ┌──────┴─────┐  ┌────┴────────┐
    │ 所有AI应用  │  │ 所有数据源   │
    └────────────┘  └─────────────┘
    
    统一接口，即插即用

### MCP的核心价值

  1. **统一数据访问** ：标准化连接各种数据源（数据库、API、文件系统等）
  2. **上下文共享** ：在不同AI应用间共享数据上下文
  3. **工具标准化** ：统一工具的定义和调用方式
  4. **安全可控** ：标准化的权限和安全管理
  5. **可扩展性** ：易于添加新的Agent和工具
  6. **降低成本** ：减少重复开发



## 一、MCP协议基础

### 1.1 什么是MCP？

**MCP（Model Context Protocol）** 是一个开放协议，用于：

  * AI应用与外部数据源/工具之间的标准化连接
  * 统一访问各种资源（数据库、文件系统、API等）
  * 在不同AI应用间共享上下文和数据



**核心设计原则** ：

  * **简单性** ：易于理解和实现
  * **灵活性** ：支持多种使用场景
  * **可扩展性** ：允许自定义扩展
  * **安全性** ：内置安全机制



### 1.2 MCP架构
    
    
    ┌─────────────────────────────────────────┐
    │         应用层                           │
    │  ┌──────────┐  ┌──────────┐            │
    │  │Agent App │  │ Tool App │            │
    │  └────┬─────┘  └────┬─────┘            │
    ├───────┼────────────┼────────────────────┤
    │       │  MCP客户端  │                    │
    ├───────┼────────────┼────────────────────┤
    │       │            │                    │
    │    ┌──┴────────────┴──┐                │
    │    │   MCP传输层       │                 │
    │    │  ([JSON-RPC 2.0](https://zhida.zhihu.com/search?content_id=269598940&content_type=Article&match_order=1&q=JSON-RPC+2.0&zhida_source=entity))  │                │
    │    └──────────────────┘                │
    ├─────────────────────────────────────────┤
    │         传输层                          │
    │  stdio, HTTP, WebSocket, etc.           │
    └─────────────────────────────────────────┘

## 二、MCP核心概念

### 2.1 基本数据结构
    
    
    from typing import Dict, List, Optional, Any, Union
    from dataclasses import dataclass, field
    from enum import Enum
    import json
    import uuid
    
    class MessageType(Enum):
        """MCP消息类型"""
        REQUEST = "request"
        RESPONSE = "response"
        NOTIFICATION = "notification"
    
    @dataclass
    class MCPMessage:
        """MCP消息基类"""
        jsonrpc: str = "2.0"
        id: Union[str, int] = None
        method: Optional[str] = None
        params: Optional[Dict] = None
        result: Optional[Any] = None
        error: Optional[Dict] = None
    
        def to_dict(self) -> Dict:
            """转换为字典"""
            data = {"jsonrpc": self.jsonrpc}
    
            if self.id is not None:
                data["id"] = self.id
    
            if self.method:
                data["method"] = self.method
    
            if self.params:
                data["params"] = self.params
    
            if self.result is not None:
                data["result"] = self.result
    
            if self.error:
                data["error"] = self.error
    
            # 移除None值
            return {k: v for k, v in data.items() if v is not None}
    
        def to_json(self) -> str:
            """转换为JSON字符串"""
            return json.dumps(self.to_dict(), ensure_ascii=False)
    
        @classmethod
        def from_dict(cls, data: Dict) -> 'MCPMessage':
            """从字典创建"""
            return cls(
                jsonrpc=data.get("jsonrpc", "2.0"),
                id=data.get("id"),
                method=data.get("method"),
                params=data.get("params"),
                result=data.get("result"),
                error=data.get("error")
            )
    
        @classmethod
        def from_json(cls, json_str: str) -> 'MCPMessage':
            """从JSON字符串创建"""
            return cls.from_dict(json.loads(json_str))
    
    @dataclass
    class MCPRequest(MCPMessage):
        """MCP请求消息"""
        method: str = ""
        params: Dict = field(default_factory=dict)
    
        def __post_init__(self):
            self.id = self.id or str(uuid.uuid4())
    
    @dataclass
    class MCPResponse(MCPMessage):
        """MCP响应消息"""
        result: Any = None
        error: Dict = None
    
    @dataclass
    class MCPNotification(MCPMessage):
        """MCP通知消息（无需响应）"""
        method: str = ""
        params: Dict = field(default_factory=dict)

### 2.2 资源（Resources）
    
    
    @dataclass
    class [MCPResource](https://zhida.zhihu.com/search?content_id=269598940&content_type=Article&match_order=1&q=MCPResource&zhida_source=entity):
        """MCP资源定义"""
        uri: str  # 统一资源标识符
        name: str
        description: str
        mime_type: Optional[str] = None
        metadata: Dict = field(default_factory=dict)
    
    class MCPResourceHandler:
        """资源处理器"""
    
        def __init__(self):
            self.resources: Dict[str, MCPResource] = {}
    
        def register_resource(self, resource: MCPResource):
            """注册资源"""
            self.resources[resource.uri] = resource
    
        def get_resource(self, uri: str) -> Optional[MCPResource]:
            """获取资源"""
            return self.resources.get(uri)
    
        def list_resources(self) -> List[MCPResource]:
            """列出所有资源"""
            return list(self.resources.values())
    
    # 使用示例
    resource_handler = MCPResourceHandler()
    
    # 注册文件资源
    file_resource = MCPResource(
        uri="file:///workspace/document.txt",
        name="文档",
        description="项目文档",
        mime_type="text/plain"
    )
    resource_handler.register_resource(file_resource)
    
    # 注册API资源
    api_resource = MCPResource(
        uri="api:///weather",
        name="天气API",
        description="天气查询接口",
        mime_type="application/json"
    )
    resource_handler.register_resource(api_resource)

### 2.3 工具（Tools）
    
    
    @dataclass
    class [MCPTool](https://zhida.zhihu.com/search?content_id=269598940&content_type=Article&match_order=1&q=MCPTool&zhida_source=entity):
        """MCP工具定义"""
        name: str
        description: str
        input_schema: Dict  # JSON Schema格式
        handler: callable
    
    class MCPToolHandler:
        """工具处理器"""
    
        def __init__(self):
            self.tools: Dict[str, MCPTool] = {}
    
        def register_tool(self, tool: MCPTool):
            """注册工具"""
            self.tools[tool.name] = tool
    
        def get_tool(self, name: str) -> Optional[MCPTool]:
            """获取工具"""
            return self.tools.get(name)
    
        def list_tools(self) -> List[MCPTool]:
            """列出所有工具"""
            return list(self.tools.values())
    
        async def execute_tool(self, name: str, params: Dict) -> Any:
            """执行工具"""
            tool = self.get_tool(name)
            if not tool:
                raise ValueError(f"工具不存在：{name}")
    
            # 验证参数
            self._validate_params(tool.input_schema, params)
    
            # 执行工具
            result = await tool.handler(params)
    
            return result
    
        def _validate_params(self, schema: Dict, params: Dict):
            """验证参数（简化版）"""
            # 实际应用中应使用完整的JSON Schema验证库
            required = schema.get("required", [])
            for field in required:
                if field not in params:
                    raise ValueError(f"缺少必需参数：{field}")
    
    # 工具定义示例
    async def search_tool(params: Dict) -> str:
        """搜索工具实现"""
        query = params["query"]
        # 实际搜索逻辑
        return f"搜索结果：{query}"
    
    async def calculate_tool(params: Dict) -> float:
        """计算工具实现"""
        expression = params["expression"]
        try:
            result = eval(expression)
            return float(result)
        except:
            raise ValueError("表达式错误")
    
    # 注册工具
    tool_handler = MCPToolHandler()
    
    # 添加搜索工具
    search_tool_def = MCPTool(
        name="search",
        description="搜索互联网信息",
        input_schema={
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "搜索关键词"
                }
            },
            "required": ["query"]
        },
        handler=search_tool
    )
    tool_handler.register_tool(search_tool_def)
    
    # 添加计算工具
    calculate_tool_def = MCPTool(
        name="calculate",
        description="执行数学计算",
        input_schema={
            "type": "object",
            "properties": {
                "expression": {
                    "type": "string",
                    "description": "数学表达式"
                }
            },
            "required": ["expression"]
        },
        handler=calculate_tool
    )
    tool_handler.register_tool(calculate_tool_def)

### 2.4 Prompts（提示词模板）
    
    
    @dataclass
    class MCPPrompt:
        """MCP提示词模板"""
        name: str
        description: str
        arguments: List[Dict]  # 参数定义
        template: str  # 提示词模板
    class MCPPromptHandler:
        """提示词处理器"""
        def __init__(self):
            self.prompts: Dict[str, MCPPrompt] = {}
        def register_prompt(self, prompt: MCPPrompt):
            """注册提示词"""
            self.prompts[prompt.name] = prompt
        def get_prompt(self, name: str, args: Dict = None) -> str:
            """获取渲染后的提示词"""
            prompt = self.prompts.get(name)
            if not prompt:
                raise ValueError(f"提示词不存在：{name}")
            # 渲染模板
            return self._render_template(prompt.template, args or {})
        def _render_template(self, template: str, args: Dict) -> str:
            """渲染模板"""
            # 简单的字符串替换（实际可用更强大的模板引擎）
            result = template
            for key, value in args.items():
                result = result.replace(f"{{{{ {key} }}}}", str(value))
            return result
    # 提示词模板示例
    code_review_prompt = MCPPrompt(
        name="code_review",
        description="代码审查提示词",
        arguments=[
            {
                "name": "code",
                "description": "要审查的代码",
                "required": True
            },
            {
                "name": "language",
                "description": "编程语言",
                "required": False
            }
        ],
        template="""请审查以下{{ language }}代码：
    {{ language }}
    {{ code }}
    
    请检查：
    - 代码质量
    - 潜在BUG
    - 性能问题
    - 安全漏洞
    - 提供改进建议。"""
    )
    
    prompt_handler = MCPPromptHandler()
    prompt_handler.register_prompt(code_review_prompt)

## 三、MCP服务器实现

### 3.1 基础MCP服务器
    
    
    import asyncio
    import json
    from typing import Dict, List
    
    class [MCPServer](https://zhida.zhihu.com/search?content_id=269598940&content_type=Article&match_order=1&q=MCPServer&zhida_source=entity):
        """MCP服务器"""
    
        def __init__(self, name: str, version: str = "1.0.0"):
            self.name = name
            self.version = version
            self.resource_handler = MCPResourceHandler()
            self.tool_handler = MCPToolHandler()
            self.prompt_handler = MCPPromptHandler()
    
            # 注册标准方法
            self._setup_standard_methods()
    
        def _setup_standard_methods(self):
            """设置标准MCP方法"""
            self.methods = {
                "initialize": self._handle_initialize,
                "tools/list": self._handle_list_tools,
                "tools/call": self._handle_call_tool,
                "resources/list": self._handle_list_resources,
                "resources/read": self._handle_read_resource,
                "prompts/list": self._handle_list_prompts,
                "prompts/get": self._handle_get_prompt
            }
    
        async def _handle_initialize(self, params: Dict) -> Dict:
            """处理初始化请求"""
            return {
                "protocolVersion": "2024-11-05",
                "serverInfo": {
                    "name": self.name,
                    "version": self.version
                },
                "capabilities": {
                    "tools": {},
                    "resources": {},
                    "prompts": {}
                }
            }
    
        async def _handle_list_tools(self, params: Dict) -> Dict:
            """列出可用工具"""
            tools = []
            for tool in self.tool_handler.list_tools():
                tools.append({
                    "name": tool.name,
                    "description": tool.description,
                    "inputSchema": tool.input_schema
                })
    
            return {"tools": tools}
    
        async def _handle_call_tool(self, params: Dict) -> Dict:
            """调用工具"""
            name = params["name"]
            arguments = params.get("arguments", {})
    
            try:
                result = await self.tool_handler.execute_tool(name, arguments)
                return {
                    "content": [
                        {
                            "type": "text",
                            "text": str(result)
                        }
                    ]
                }
            except Exception as e:
                return {
                    "content": [
                        {
                            "type": "text",
                            "text": f"错误：{str(e)}"
                        }
                    ],
                    "isError": True
                }
    
        async def _handle_list_resources(self, params: Dict) -> Dict:
            """列出可用资源"""
            resources = []
            for resource in self.resource_handler.list_resources():
                resources.append({
                    "uri": resource.uri,
                    "name": resource.name,
                    "description": resource.description,
                    "mimeType": resource.mime_type
                })
    
            return {"resources": resources}
    
        async def _handle_read_resource(self, params: Dict) -> Dict:
            """读取资源"""
            uri = params["uri"]
            # 实际应用中应该读取真实资源
            return {
                "contents": [
                    {
                        "uri": uri,
                        "mimeType": "text/plain",
                        "text": f"资源内容：{uri}"
                    }
                ]
            }
    
        async def _handle_list_prompts(self, params: Dict) -> Dict:
            """列出提示词"""
            prompts = []
            for prompt in self.prompt_handler.prompts.values():
                prompts.append({
                    "name": prompt.name,
                    "description": prompt.description,
                    "arguments": prompt.arguments
                })
    
            return {"prompts": prompts}
    
        async def _handle_get_prompt(self, params: Dict) -> Dict:
            """获取提示词"""
            name = params["name"]
            args = params.get("arguments", {})
    
            try:
                prompt_text = self.prompt_handler.get_prompt(name, args)
                return {
                    "description": f"提示词：{name}",
                    "messages": [
                        {
                            "role": "user",
                            "content": {
                                "type": "text",
                                "text": prompt_text
                            }
                        }
                    ]
                }
            except Exception as e:
                return {"error": str(e)}
    
        async def handle_message(self, message: MCPMessage) -> MCPMessage:
            """处理MCP消息"""
            if message.method not in self.methods:
                return MCPResponse(
                    id=message.id,
                    error={
                        "code": -32601,
                        "message": f"未知方法：{message.method}"
                    }
                )
    
            try:
                handler = self.methods[message.method]
                result = await handler(message.params or {})
    
                return MCPResponse(id=message.id, result=result)
    
            except Exception as e:
                return MCPResponse(
                    id=message.id,
                    error={
                        "code": -32603,
                        "message": str(e)
                    }
                )

### 3.2 HTTP传输层
    
    
    from aiohttp import web
    import aiohttp
    
    class MCPHTTPServer:
        """基于HTTP的MCP服务器"""
    
        def __init__(self, mcp_server: MCPServer, host: str = "0.0.0.0", port: int = 8080):
            self.mcp_server = mcp_server
            self.host = host
            self.port = port
            self.app = web.Application()
            self._setup_routes()
    
        def _setup_routes(self):
            """设置路由"""
            self.app.router.add_post("/mcp", self._handle_mcp_request)
            self.app.router.add_get("/health", self._health_check)
    
        async def _handle_mcp_request(self, request: web.Request) -> web.Response:
            """处理MCP请求"""
            try:
                # 解析请求
                body = await request.json()
                mcp_message = MCPMessage.from_dict(body)
    
                # 处理消息
                response = await self.mcp_server.handle_message(mcp_message)
    
                # 返回响应
                return web.json_response(response.to_dict())
    
            except Exception as e:
                return web.json_response(
                    {
                        "jsonrpc": "2.0",
                        "error": {
                            "code": -32700,
                            "message": f"解析错误：{str(e)}"
                        }
                    }
                )
    
        async def _health_check(self, request: web.Request) -> web.Response:
            """健康检查"""
            return web.json_response({"status": "ok"})
    
        async def start(self):
            """启动服务器"""
            runner = web.AppRunner(self.app)
            await runner.setup()
            site = web.TCPSite(runner, self.host, self.port)
            await site.start()
            print(f"MCP服务器运行在 http://{self.host}:{self.port}")
    
        async def stop(self):
            """停止服务器"""
            pass
    
    class MCPHTTPClient:
        """基于HTTP的MCP客户端"""
    
        def __init__(self, server_url: str):
            self.server_url = server_url
            self.session = None
    
        async def __aenter__(self):
            self.session = aiohttp.ClientSession()
            return self
    
        async def __aexit__(self, exc_type, exc_val, exc_tb):
            if self.session:
                await self.session.close()
    
        async def send_request(self, method: str, params: Dict = None) -> Any:
            """发送MCP请求"""
            request = MCPRequest(
                method=method,
                params=params or {}
            )
    
            async with self.session.post(
                f"{self.server_url}/mcp",
                json=request.to_dict()
            ) as response:
                data = await response.json()
                mcp_response = MCPMessage.from_dict(data)
    
                if mcp_response.error:
                    raise Exception(f"MCP错误：{mcp_response.error}")
    
                return mcp_response.result
    
        async def initialize(self) -> Dict:
            """初始化连接"""
            return await self.send_request("initialize")
    
        async def list_tools(self) -> List[Dict]:
            """列出可用工具"""
            result = await self.send_request("tools/list")
            return result.get("tools", [])
    
        async def call_tool(self, name: str, arguments: Dict) -> Any:
            """调用工具"""
            result = await self.send_request(
                "tools/call",
                {"name": name, "arguments": arguments}
            )
            return result
    
        async def list_resources(self) -> List[Dict]:
            """列出资源"""
            result = await self.send_request("resources/list")
            return result.get("resources", [])
    
        async def read_resource(self, uri: str) -> str:
            """读取资源"""
            result = await self.send_request(
                "resources/read",
                {"uri": uri}
            )
            return result

## 四、MCP客户端实现
    
    
    class MCPClient:
        """通用MCP客户端"""
    
        def __init__(self):
            self.server_info = None
            self.capabilities = None
            self.transport = None
    
        async def connect(self, transport):
            """连接到MCP服务器"""
            self.transport = transport
    
            # 初始化
            init_request = MCPRequest(
                method="initialize",
                params={
                    "protocolVersion": "2024-11-05",
                    "capabilities": {},
                    "clientInfo": {
                        "name": "MCP-Python-Client",
                        "version": "1.0.0"
                    }
                }
            )
    
            response = await self.transport.send(init_request)
            self.server_info = response.result.get("serverInfo")
            self.capabilities = response.result.get("capabilities")
    
            print(f"已连接到MCP服务器：{self.server_info}")
    
        async def list_tools(self) -> List[Dict]:
            """列出可用工具"""
            request = MCPRequest(method="tools/list")
            response = await self.transport.send(request)
            return response.result.get("tools", [])
    
        async def call_tool(self, name: str, arguments: Dict) -> Any:
            """调用工具"""
            request = MCPRequest(
                method="tools/call",
                params={
                    "name": name,
                    "arguments": arguments
                }
            )
    
            response = await self.transport.send(request)
            return response.result
    
        async def list_resources(self) -> List[Dict]:
            """列出资源"""
            request = MCPRequest(method="resources/list")
            response = await self.transport.send(request)
            return response.result.get("resources", [])
    
        async def read_resource(self, uri: str) -> str:
            """读取资源"""
            request = MCPRequest(
                method="resources/read",
                params={"uri": uri}
            )
    
            response = await self.transport.send(request)
            return response.result
    
        async def list_prompts(self) -> List[Dict]:
            """列出提示词"""
            request = MCPRequest(method="prompts/list")
            response = await self.transport.send(request)
            return response.result.get("prompts", [])
    
        async def get_prompt(self, name: str, arguments: Dict = None) -> str:
            """获取提示词"""
            request = MCPRequest(
                method="prompts/get",
                params={
                    "name": name,
                    "arguments": arguments or {}
                }
            )
    
            response = await self.transport.send(request)
            return response.result

## 五、实战案例：构建MCP工具服务器
    
    
    class WeatherMCPServer(MCPServer):
        """天气MCP服务器示例"""
    
        def __init__(self):
            super().__init__("weather-server", "1.0.0")
            self._setup_tools()
            self._setup_resources()
    
        def _setup_tools(self):
            """设置工具"""
            # 当前天气工具
            current_weather_tool = MCPTool(
                name="get_current_weather",
                description="获取当前天气",
                input_schema={
                    "type": "object",
                    "properties": {
                        "city": {
                            "type": "string",
                            "description": "城市名称"
                        },
                        "unit": {
                            "type": "string",
                            "enum": ["celsius", "fahrenheit"],
                            "description": "温度单位"
                        }
                    },
                    "required": ["city"]
                },
                handler=self._get_current_weather
            )
            self.tool_handler.register_tool(current_weather_tool)
    
            # 天气预报工具
            forecast_tool = MCPTool(
                name="get_weather_forecast",
                description="获取天气预报",
                input_schema={
                    "type": "object",
                    "properties": {
                        "city": {"type": "string"},
                        "days": {
                            "type": "integer",
                            "minimum": 1,
                            "maximum": 7
                        }
                    },
                    "required": ["city"]
                },
                handler=self._get_weather_forecast
            )
            self.tool_handler.register_tool(forecast_tool)
    
        def _setup_resources(self):
            """设置资源"""
            # 天气数据资源
            weather_resource = MCPResource(
                uri="weather:///current",
                name="当前天气数据",
                description="实时天气信息",
                mime_type="application/json"
            )
            self.resource_handler.register_resource(weather_resource)
    
        async def _get_current_weather(self, params: Dict) -> str:
            """获取当前天气"""
            city = params["city"]
            unit = params.get("unit", "celsius")
    
            # 模拟API调用
            weather_data = {
                "city": city,
                "temperature": 20,
                "unit": unit,
                "condition": "晴天",
                "humidity": 65,
                "wind_speed": 3.5
            }
    
            return json.dumps(weather_data, ensure_ascii=False)
    
        async def _get_weather_forecast(self, params: Dict) -> str:
            """获取天气预报"""
            city = params["city"]
            days = params.get("days", 3)
    
            # 模拟预报数据
            forecast = []
            for i in range(days):
                forecast.append({
                    "date": f"2024-03-{i+1:02d}",
                    "high": 25 + i,
                    "low": 15 + i,
                    "condition": "多云" if i % 2 else "晴天"
                })
    
            return json.dumps({
                "city": city,
                "forecast": forecast
            }, ensure_ascii=False)
    
    # 使用示例
    async def main():
        # 启动MCP服务器
        server = WeatherMCPServer()
        http_server = MCPHTTPServer(server, port=8080)
        await http_server.start()
    
        # 客户端连接
        async with MCPHTTPClient("http://localhost:8080") as client:
            # 初始化
            await client.initialize()
    
            # 列出工具
            tools = await client.list_tools()
            print("可用工具：")
            for tool in tools:
                print(f"  - {tool['name']}: {tool['description']}")
    
            # 调用工具
            weather = await client.call_tool(
                "get_current_weather",
                {"city": "北京", "unit": "celsius"}
            )
            print(f"\n当前天气：{weather}")
    
            # 获取预报
            forecast = await client.call_tool(
                "get_weather_forecast",
                {"city": "上海", "days": 5}
            )
            print(f"\n天气预报：{forecast}")
    
    if __name__ == "__main__":
        asyncio.run(main())

## 六、总结

### 核心要点

  1. **MCP协议** ：AI应用与数据源的标准化连接协议
  2. **三大核心** ：Resources、Tools、Prompts
  3. **传输层** ：支持多种传输方式（HTTP、stdio等）
  4. **服务器** ：实现MCP接口的服务端（提供数据源）
  5. **客户端** ：连接和使用MCP服务（AI应用）



### 最佳实践

  * ✅ **遵循规范** ：严格按照MCP协议实现
  * ✅ **错误处理** ：完善的错误处理机制
  * ✅ **类型安全** ：使用JSON Schema验证
  * ✅ **文档完善** ：提供清晰的工具和资源说明
  * ✅ **性能优化** ：缓存、批量操作



### 常见陷阱

  * ❌ **版本不匹配** ：客户端和服务器版本不一致
  * ❌ **参数验证缺失** ：导致运行时错误
  * ❌ **资源泄漏** ：未正确关闭连接
  * ❌ **并发问题** ：多线程/异步安全问题



* * *

## 推荐阅读

  * [MCP Protocol Specification](https://link.zhihu.com/?target=https%3A//spec.modelcontextprotocol.io/)
  * [Model Context Protocol SDK](https://link.zhihu.com/?target=https%3A//github.com/modelcontextprotocol)
  * [Building MCP Tools](https://link.zhihu.com/?target=https%3A//modelcontextprotocol.io/basics)



* * *

 _如果这篇文章对你有帮助，欢迎点赞、收藏和分享！有任何问题欢迎在评论区讨论。_

* * *

这是《AI Agent系列教程》的第4篇，共14篇。

> 上一篇：[【Agent入门到精通】03-工具调用（Function Calling）：Agent的手和脚](https://zhuanlan.zhihu.com/p/1999201160813384313)  
> 下一篇：[【Agent入门到精通】05-Workflow架构：可视化Agent编排平台](https://zhuanlan.zhihu.com/p/1999206497670947974)

* * *

**系列说明** ：

  * 本系列文章正在持续更新中，欢迎关注！
  * 所有代码示例将在GitHub仓库开源：`ai-agent-tutorial-series`
  * 有问题欢迎在评论区讨论，我会及时回复


