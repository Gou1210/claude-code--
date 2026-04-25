# 14. Hooks：生命周期自动化

## 重要程度
**A 级** - 强烈建议掌握，属于进阶分水岭

## 学习目标
- 学会把规则做成确定性自动执行，而不是"希望 Claude 记住"

## 学什么
- hooks guide
- hooks reference
- event lifecycle
- async hooks
- HTTP hooks
- prompt hooks
- MCP tool hooks
- 调试 hooks

## 你需要掌握
- Hooks 是用户定义的 shell 命令、HTTP 端点或 LLM prompts，会在 Claude Code 生命周期的特定点自动执行
- hooks 的核心价值是 deterministic control，也就是保证某些动作一定发生，而不是靠模型"想起来"
- hooks 覆盖的事件很多，包括 SessionStart、SessionEnd、UserPromptSubmit、PreToolUse、PostToolUse、Stop、SubagentStart/Stop、PreCompact、FileChanged 等
- reference 页面还包含 async hooks、HTTP hooks、prompt hooks、MCP tool hooks 等高级能力

## 学完标准
- 你能写出至少一个 edit 后自动测试或格式化的 hook
- 你知道 hook 什么时候比 skill 更合适
