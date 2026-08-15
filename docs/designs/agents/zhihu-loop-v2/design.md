---
topic: zhihu-loop-v2
title: 知乎运营闭环 v2 — hermes 驱动 + Claude Code 写作
status: implemented
created: 2026-08-09
spike_dir: .spike/zhihu-loop-v2/
related_code:
  - ~/.claude/skills/zhihu-loop/SKILL.md
  - ~/.claude/skills/zhihu-loop/state.json
  - ~/.claude/skills/zhihu-loop/scripts/
  - ~/.hermes/scripts/zhihu-daily.sh
  - ~/.hermes/scripts/zhihu-cycle.sh
human_summary: ../../humans/zhihu-loop-v2/index.html
---

# 知乎运营闭环 v2 — hermes 驱动 + Claude Code 写作

## What this is

把现有 zhihu-loop（已跑 2 个 cycle）升级为一套全自动运营系统：hermes cron 负责调度和通知，Claude Code 负责写高质量答案，目标 3 个月涨到 1W 知乎关注者。

核心突破：把原来每次手动触发的"一次性答题 spike"升级为 hermes 定时驱动、带三道用户确认门的持续运营闭环。

## Goal & context

**目标**：90 天知乎粉丝从 0 涨到 10,000（zhihu-1w-followers-2026Q4）。

保底指标：
- 30 篇回答（约每 3 天一篇）
- 10 篇 ≥100 赞
- 2 篇 ≥500 赞
- 总获赞 5,000+

背景：用户是 openEuler SIG AI Agent Maintainer，tech-blog 有 388+ 篇 AI Agent / Harness / LLM 工程化文章作为素材金矿。垂直领域专家在知乎的涨粉杠杆：2 篇爆款（≥500 赞）≈ 4000 粉，10 篇高赞（≥100 赞）≈ 3000 粉，长尾补齐 3000，合计 1W 可达。

**scope in**：SCOUT 选题、WRITE 答案、MEASURE 指标、daily-report 推送、TOPIC-SCOUT 主动发问。
**scope out**：账号级粉丝数自动测量（opencli 无此能力，待升级后验证）、自动发布答案（合规红线）。

## Alignment conclusions

- 三道用户确认门（强制，不可绕过）：门1选题、门2发布确认、门3发起问题
- WRITE/HUMANIZER 由 Claude Code（Sonnet 4.6）执行，不交给 hermes LLM，保证答案质量
- hermes cron 使用 `--no-agent` 模式跑 bash 脚本，不消耗 LLM token
- 通知渠道：已有 HERMES_FEISHU_TARGET（飞书一人群）+ HERMES_WEIXIN_TARGET（微信），notify.sh 自动选通道
- opencli zhihu v1.7.12 无 `profile/stats/create-question` 命令；每日快报用回答赞变化聚合代替账号粉丝数
- topic-scout.sh 每周一由 run-daily.sh 内嵌触发，生成问题草稿推飞书，用户网页手动发起
- 所有答案存 tech-blog/zhihu-answers/，git push 归档，答案须引用至少 1 篇旧文 + 加「延伸阅读」段

## What we tried — decision log

1. **检查现有脚本是否真实实现** → 确认 scout.sh（opencli hot+search）、measure.sh（opencli question）均为真实调用，非伪代码。notify.sh 已有飞书 webhook + hermes send 多通道架构。

2. **每日粉丝数方案调研** → opencli zhihu 命令集：`answer, collection, collections, comment, download, favorite, follow, hot, like, question, search`，无 profile/stats/me。测试升级路径后决定：先用回答赞变化（现有能力）作每日快报，等 opencli 升级到 v1.8.6 后 re-check。

3. **发起问题方案** → opencli 无 `create-question`，playwright 自动化有账号安全风险，决定：agent 起草问题草稿（标题+背景），加门3由用户网页手动发起。

4. **hermes 调度语法确认** → 测试发现 hermes 用 `cron`（不是 `schedule`）子命令，正确语法：`hermes cron add "<cron>" "<prompt>" --name <n> --script <s> --no-agent --workdir <d>`。wrapper 脚本放 `~/.hermes/scripts/`。

5. **WRITE 执行者** → 用户明确选 Claude Code（非 hermes LLM），WRITE/HUMANIZER 阶段保留在 Claude Code，质量优先。

## The approach

### 架构分工

```
hermes cron (--no-agent, bash)           Claude Code (LLM, Sonnet 4.6)
──────────────────────────────           ───────────────────────────────
每天 08:00 → run-daily.sh               用户飞书选题后触发
  ├─ measure.sh                           WRITE (起草答案)
  ├─ daily-report.sh                      HUMANIZER (去 AI 味)
  └─ notify.sh → 飞书/微信推快报           → 门2：推飞书请用户确认

周一/周四 09:00 → run-cycle.sh           用户确认「发」后
  ├─ scout.sh (hot + search)              publish.sh → git push
  ├─ corpus-search.sh (tech-blog)         → 用户网页粘贴发布
  └─ notify.sh → 飞书推候选               → fill-back-url.sh
     ⏸ 门1：等用户选「答 <qid>」

每周一 (run-daily.sh 内嵌)              门3确认后
  topic-scout.sh                          用户网页发起问题
  → 2 道问题草稿 → 飞书                   → fill-back-question.sh
  ⏸ 门3：等用户确认
```

### 文件布局

```
~/.claude/skills/zhihu-loop/
├── SKILL.md              # 完整流程文档 v2
├── state.json            # v1.1.0，维护 cycle/history/goal/corpus_hints
├── scripts/
│   ├── run-daily.sh      # 每日 orchestrator（新增）
│   ├── run-cycle.sh      # 答题循环 orchestrator（新增）
│   ├── daily-report.sh   # 每日快报生成（新增）
│   ├── corpus-search.sh  # tech-blog 全库搜索（新增）
│   ├── topic-scout.sh    # 每周话题调研 + 问题草稿（新增）
│   ├── fill-back-question.sh  # 问题 qid 回填（新增）
│   ├── hermes-setup.sh   # hermes cron 注册（更新语法）
│   ├── scout.sh          # opencli hot+search（已有）
│   ├── measure.sh        # opencli question 回收（已有）
│   ├── notify.sh         # 飞书/微信多通道推送（已有）
│   ├── publish.sh        # 打包发布 + git push（已有）
│   ├── fill-back-url.sh  # 回答 URL 回填（已有）
│   └── learn.sh          # metrics 聚合 → learned.md（已有）
├── answers/              # 已发布回答存档
├── drafts/               # 草稿 + topic-scout 问题草稿
├── metrics/              # measure.sh 输出的 JSON 快照
├── patterns/             # learn.sh 输出的 learned.md
└── corpus/               # corpus-search 输出缓存（可选）

~/.hermes/scripts/
├── zhihu-daily.sh        # → run-daily.sh wrapper
└── zhihu-cycle.sh        # → run-cycle.sh wrapper
```

### state.json 关键字段

```json
{
  "current_cycle": 2,
  "loop_state": "awaiting_publish",
  "goal": {
    "name": "zhihu-1w-followers-2026Q4",
    "metrics": [
      {"key": "follower_delta", "target": 10000, "current": 0},
      {"key": "answer_count", "target": 30, "current": 3},
      {"key": "high_vote_count", "target": 10, "current": 0},
      {"key": "high_vote_count_500", "target": 2, "current": 0},
      {"key": "total_votes", "target": 5000, "current": 0}
    ]
  },
  "strategy": {
    "big_words": ["AI Agent 落地", "Harness Engineering", "Claude Code", ...],
    "my_question_qids": []
  },
  "cycle_context": {
    "corpus_hints": [...]
  },
  "history": [...],
  "my_questions": [...]
}
```

### SCOUT 选题策略

- 优先：热度 ≥ 100 + 回答数 ≤ 50（竞争小，能排前）
- 关键词命中用户专长（AI Agent / Harness / Claude / DeepSeek）
- 标题含钩子词（为什么 / 本质 / 怎么 / 如何 / 踩坑），收藏率高
- 优先回答 my_question_qids 中自己发起的问题（自产自销流量闭环）

### daily-report.sh 输出格式

```markdown
# 知乎每日快报 · 2026-08-09
📅 第 2 天（还剩 88 天）  目标：3 个月涨到 1W 粉

## Goal 进度
- 粉丝  [░░░░░░░░░░░░░░░░░░░░] 0/10000
- 回答  [██░░░░░░░░░░░░░░░░░░] 3/30
- 高赞  [░░░░░░░░░░░░░░░░░░░░] 0/10

## 今日回答表现
| 回答 | 赞 | ±今日 | 排名 |
...
## 建议
...
```

### hermes cron 已注册任务

| ID | Name | Schedule | Mode |
|----|------|----------|------|
| cd2ba54231f2 | zhihu-daily | 每天 08:00 | --no-agent |
| 96130ab56756 | zhihu-cycle | 周一/周四 09:00 | --no-agent |

### 错误处理

- run-cycle.sh 检查 `loop_state == awaiting_publish`，避免重复触发（除非 `--force`）
- 所有 opencli 调用有 timeout（measure: 60s，scout: 10s/词），超时退出而非挂起
- notify.sh 降级：webhook → hermes send → 终端打印，不因通道缺失而崩溃
- corpus-search.sh 找不到 tech-blog 目录时安全退出，不阻断主流程

## Open questions & risks

1. **opencli 无账号粉丝数**：每日快报只有回答赞，无法直接追踪粉丝增量。缓解：升级 opencli v1.8.6 + extension v1.0.22 后重新检查；若仍无，后期考虑 playwright 抓账号页。

2. **scout.sh perl timeout 脆弱**：macOS 缺 gtimeout，当前用 perl fork/alarm 技巧，依赖 IO::Popen。风险：某些 macOS 版本上挂死。缓解：后续改为 Python `subprocess.run(timeout=...)`。

3. **opencli v1.7.12 → v1.8.6 + extension 跳版大**：extension 从 v1.0.5 → v1.0.22，有可能新增 `create-question` 或 `profile` 命令。需升级验证。

4. **cycle 2 仍在 awaiting_publish**：DeepSeek V4 Flash 回答（v2）未发布，旧 v1 需在知乎网页删除。这是用户侧手动操作，不阻断新 cycle。

5. **1W 粉目标压力**：90 天需平均每天涨 111 粉。前期低（正常），依赖 2-3 篇爆款拉升。若第 30 天粉丝仍 < 500，需重新评估 cadence 和选题策略。

## Implementation plan

本 spike 已完成实现，以下是运营阶段的操作清单：

### 立刻（用户侧）
- [ ] 知乎网页删除 cycle 2 旧 v1 答案（answer id `2069508313591690890`）
- [ ] 发布 v2（`tech-blog/zhihu-answers/2026-08-08-deepseek-v4-flash-agent-harness-v2.md`）
- [ ] 回填 URL：`bash ~/.claude/skills/zhihu-loop/scripts/fill-back-url.sh 2026-08-08-deepseek-v4-flash-agent-harness-v2 <新URL>`

### 本周内
- [ ] 升级 opencli：`npm install -g @jackwener/opencli` → re-check `opencli zhihu --help`
- [ ] 配置飞书 webhook（可选，已有 hermes 通道）：`echo 'export FEISHU_WEBHOOK="..."' >> ~/.zhihu-loop.env`
- [ ] 验证 hermes cron 已启动：`hermes cron status`

### 持续运营
- 每天 08:00：hermes 自动触发 daily-report → 飞书推送
- 周一/周四 09:00：hermes 自动触发 SCOUT → 飞书推候选 → 用户选题 → Claude Code 写答案
- 每周一：hermes 顺带触发 topic-scout → 飞书推问题草稿 → 用户发起

### 常用命令
```bash
SKILL=~/.claude/skills/zhihu-loop

# 查看状态
bash $SKILL/scripts/status.sh

# 手动触发
bash $SKILL/scripts/run-daily.sh
bash $SKILL/scripts/run-cycle.sh

# 回填
bash $SKILL/scripts/fill-back-url.sh <slug> <url>
bash $SKILL/scripts/fill-back-question.sh <qid> "标题"
```
