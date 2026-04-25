# 12. Subagents：子代理与任务委派

## 重要程度
**A 级** - 强烈建议掌握，属于进阶分水岭

## 学习目标
- 学会把高噪音工作拆出去
- 学会并行调查、分工执行

## 学什么
- built-in subagents
- custom subagents
- automatic delegation
- foreground / background
- subagent memory
- subagent hooks
- subagent capability control

## 你需要掌握
- Claude 会根据 subagent 描述决定何时委派任务
- Claude Code 自带若干 built-in subagents，也支持自定义 subagents
- 文档地图显示 subagents 已覆盖：自动委派、前后台运行、上下文管理、恢复、auto-compaction、persistent memory、可用工具限制、MCP 范围、hooks 等完整能力
- 文档还给了典型样例，如 code reviewer、debugger、data scientist、database query validator 等

## 学完标准
- 你能设计出 2 到 4 个适合自己项目的 subagents
- 你知道哪些任务该主会话做，哪些该丢给 subagent
