# 5. Context Window 与上下文管理

## 重要程度
**S 级** - 必须掌握，不会就很难高效用

## 学习目标
- 理解 Claude Code 为什么会"越聊越重"
- 学会控制上下文，而不是无限拉长会话

## 学什么
- context window
- compact
- resume
- session 管理
- 长会话策略

## 你需要掌握
- 会话开始前，CLAUDE.md、auto memory、MCP tool names、skill descriptions 等就会进入上下文
- Claude 工作过程中，每次读文件也会继续消耗上下文
- Claude 会自动 compact，必要时也可手动 /compact
- VS Code 中有可视化 context indicator

## 应学的具体方法
- 什么时候新开 session
- 什么时候 /resume
- 什么时候 /compact
- 如何把流程说明从对话里挪到 CLAUDE.md / skills
- 如何把高噪音任务交给 subagents

## 学完标准
- 你不会再把所有任务都塞进一个长 session
- 你能主动设计上下文负载
