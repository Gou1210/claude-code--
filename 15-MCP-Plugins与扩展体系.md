# 15. MCP、Plugins 与扩展体系

## 重要程度
**A 级** - 强烈建议掌握，属于进阶分水岭

## 学习目标
- 学会连接外部工具与服务
- 搞清楚 Claude Code 的扩展层级

## 学什么
- features overview
- MCP
- plugins
- plugins reference
- 在 VS Code 中接入 MCP / plugins

## 你需要掌握
- features overview 把 Claude Code 能力分成：always-on context、on-demand capabilities、background automation 等不同层
- Claude Code 可以通过 skills 扩展知识，通过 MCP 连外部服务，通过 hooks 自动化，通过 subagents 做任务分拆
- plugin 安装后可自动发现 skills 和 commands，Claude 也可在相关上下文中自动调用它们

## 学完标准
- 你能说清楚 skill / hook / MCP / plugin 的边界
- 你能按目标选对扩展方式
