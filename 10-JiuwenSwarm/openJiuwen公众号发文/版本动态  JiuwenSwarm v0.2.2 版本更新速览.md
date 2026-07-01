# 版本动态 | JiuwenSwarm v0.2.2 版本更新速览
openJiuwen openJiuwen 2026年6月16日 22:32 上海 听全文

发布日期：2026 年 6 月 16 日
JiuwenSwarm v0.2.2：本次版本重点引入**SwarmFlow 可控工作流编排、 ****Symphony**** 智能编排与技能检索、多源技能分发体系、****A2UI**** 模块化渲染**，强化 TUI 交互体验、新增 CircuitBreaker 安全断路器，以及大量稳定性与易用性改进。
# ✨ 新功能（Features）
### SwarmFlow 可控工作流编排
• **SwarmFlow**：全新推出面向多智能体团队的可控工作流编排系统，将协作编排从 Leader 的临场判断中解耦，由系统按程序稳定执行，大幅提升复杂任务的确定性与可靠性。
• **声明式装配体系**：基于纯声明式 Spec（RailSpec / BuiltinToolSpec / SubAgentSpec）的装配设计，支持从 config 源派生能力配置，实现团队成员 harness 的可序列化、可复用。
• **SwarmSkill Creator 适配**：swarmskill-creator 升级新版本，完整支持 swarmflow 工作流生成。
### Symphony 智能编排
• **Symphony 核心组件**：引入交响乐编排核心组件，实现复杂任务的智能拆解与协调执行。
• **Agentic**** 技能检索**：引入本地 skill 的 Agentic 检索能力，支持智能技能匹配与推荐。
• **增量构建能力**：支持 Symphony 总谱增量构建、指纹/关系缓存、断点续构建，大幅提升构建效率。
• **可视化展示**：在聊天界面可视化展示 Agentic 搜索的技能树路径，skill 能力树支持增量构建机制。
### TUI 交互体验
• **/resume 快捷键**：新增 Ctrl+A、Ctrl+B、Space、Ctrl+R 等快捷键支持。
• **部分对话压缩**：新增"从此处开始总结"/"总结到此处"的部分对话压缩功能。
• **日志折叠展示**：TUI 支持日志折叠与展开功能。
• **Recap 双语输出**：支持双语 recap 输出及自动 recap 配置开关。
### 权限与安全
• **CircuitBreaker 断路器**：新增 Agent 工具循环调用检测断路器，支持 4 种循环检测类型（相同工具重复、未知工具重复、全局无进展兜底、Ping-Pong 交替循环），主动中断防止死循环。
### 前端 / Web
• **Markdown 增强**：增强 markdown 渲染、持久化历史记录、成员面板布局优化。
• **A2UI 模块化集成**：改进模块化 A2UI 集成能力，提升 Agent-to-UI 交互体验。
• **临时团队解散**：支持解散临时团队。
• **团队右侧面板优化**：调整 team 成员右侧面板样式以及 mermaid 样式。
• **成员状态处理**：新增成员状态处理，过滤活跃团队成员。
### Model
• **DeepSeek V4 支持**：在运行时从 reasoning_level 注入 DeepSeek V4 thinking 参数。
# 🛠️ 缺陷修复（Bug Fixes）
### TUI
• 修复 jiuwenswarm.md 过大后，告警字符过长报错。
• 修复运行 /compact 时按 Esc 无法中断进程的问题。
• 修复 /statusline 支持 agent 自动写脚本并生效。
### Skill 技能
• 修复 SkillNet 技能搜索 bug。
• 修复 skills.get 在 SKILL.md 缺少 frontmatter 时找不到技能的问题。
• 修复 ClawHub 搜索显示 slug 字段，下掉源管理 ClawHub 显示/隐藏明文按钮。
• 技能列表在刷新后自动调整到当前视图。
• 未托管的技能自动注册为本地技能。
### 前端 / Web
• 修复预置成员加载时不显示的问题。
• 修复修改 agents 配置图标，team 配置团队名称长度限制 32，去除成员名称的前后空格。
• 修复删除多 agents 和 team 配置的缓存，修复删除 agents/team 配置后保存恢复问题优化。
• 修复预置成员删除后保存按钮依旧置灰问题，删除 agents/team 配置后保存恢复问题优化。
• 移除技能禁用按钮的悬浮文字。去掉外链字体加载，统一字体设置。
• 支持切换同名模型。
• 修复 team 模式下 leader 上下文窗口显示。
### Cron 定时任务
• 修复创建多个同时触发的 /cron 任务后退出直接删除 cron 任务持久化文件，清空定时任务列表的问题。
### Session 会话
• 修复发送消息时重置历史加载卡住的问题。
• /resume 添加 session_id 信息，修复切换会话错误。
### 文档
• 文档更新与优化• A2UI介绍：https://atomgit.com/openJiuwen/jiuwenswarm/blob/develop/docs/zh/A2UI.md• AutoHarness介绍：https://atomgit.com/openJiuwen/jiuwenswarm/blob/develop/docs/zh/AutoHarness.md• SwarmFlow使用指南：https://gitcode.com/openJiuwen/jiuwenswarm/blob/develop/docs/zh/TUI%E4%BD%BF%E7%94%A8SwarmFlow%E6%8C%87%E5%8D%97.md

❤️最后感谢@ldstyle8 @lin-xiaoyu @ray_le @z2tng @chensheng12345 @xiaotian077 @haowenzhong1 @cycing_1 @xiaoyao42 @xutuo @xiaoxiawang @wncbdj @min_gitcode @zhengkailan 等各位开发者的反馈与贡献（排名不分先后），欢迎大家体验JiuwenSwarm新版本！
如有问题欢迎大家通过 Issues反馈，我们会持续迭代优化。
https://atomgit.com/openJiuwen/jiuwenswarm/issues
https://github.com/openJiuwen-ai/jiuwenswarm/issues
# 📚 相关资源
• JiuwenSwarm官网：
https://www.openjiuwen.com/jiuwenswarm
• AtomGit代码仓：
https://atomgit.com/openJiuwen/jiuwenswarm
• GitHub代码仓：
https://github.com/openJiuwen-ai/jiuwenswarm
