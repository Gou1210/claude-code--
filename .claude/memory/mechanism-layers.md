---
name: 机制分层关系
description: Claude Code 各机制的职责边界与强制力层级
type: reference
---

# Claude Code 机制分层关系

## 核心原则

**需要强制执行的东西，交给 settings / permissions / hooks，不要只靠 memory 或 skill。**

## 三层架构

| 层级 | 机制 | 本质 | 强制力 |
|------|------|------|--------|
| **硬配置层** | settings / permissions / hooks | 系统级约束 | 强制执行，不依赖 Claude 理解 |
| **说明层** | `CLAUDE.md` / rules / auto memory | 上下文输入 | 依赖 Claude 理解与配合 |
| **能力层** | skills / agents | 按需调用的扩展 | 同样是输入，不是约束 |

## 各机制职责

### 硬配置层（强制约束）

**settings**
- 模型选择
- 环境变量
- MCP 服务器配置
- 不能被 Claude 忽略

**permissions**
- 允许/禁止的命令
- 文件访问范围
- 工具调用权限
- 系统层面拦截

**hooks**
- 动作前后的自动执行
- 拦截点不可跳过
- 可用于校验、通知、拦截

### 说明层（依赖理解）

**`CLAUDE.md`**
- 你写给 Claude 的长期说明书
- 项目规则、构建命令、架构说明
- 每次会话自动注入

**rules**
- 按路径/主题拆分的规则文件
- 可带 `paths:` 限定生效范围
- 比 `CLAUDE.md` 更模块化

**auto memory**
- Claude 自己积累的工作笔记
- 适合记录反复纠正后的偏好

### 能力层（按需扩展）

**skills**
- 流程封装（发布流程、排查清单）
- 调用时才加载
- 是"扩展能力"不是"施加约束"

**agents / subagents**
- 独立上下文执行任务
- 隔离噪音，主会话只拿结果

## 为什么 memory/skill 不能替代硬配置

```text
memory/skill  →  "建议你这样做"（Claude 可以忽略或理解偏）
settings      →  "模型用这个、环境变量是这些"（系统层面固定）
permissions   →  "这些命令允许/禁止"（无法绕过）
hooks         →  "动作前后自动执行"（拦截点，不可跳过）
```

## 选择决策

| 需求 | 放哪里 |
|------|--------|
| 每次都要知道的规则 | `CLAUDE.md` |
| 只对某些目录生效 | rules + `paths:` |
| 流程步骤（发布/排查） | skill |
| Claude 自己学会的偏好 | auto memory |
| 强制使用某个模型 | settings |
| 禁止某些危险命令 | permissions |
| 每次提交前自动检查 | hooks |
