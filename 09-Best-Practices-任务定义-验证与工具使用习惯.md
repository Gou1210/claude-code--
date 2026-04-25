# 9. Best Practices：任务定义、验证与工具使用习惯

## 重要程度
**S 级** - 必须掌握，不会就很难高效用

## 学习目标
- 学会让 Claude Code 输出更稳
- 掌握官方推荐的高收益使用习惯

## 学什么
- Best Practices 全文
- 给 Claude 验证方式
- 善用 CLI 工具
- 管理 permissions
- 并行与拆分会话

## 你需要掌握
- 官方明确推荐：给 Claude 一个可验证结果的方式
- 官方也建议 Claude Code 与 gh、aws、gcloud、sentry-cli 等 CLI 工具配合使用
- permissions 的核心不是一味保守，而是通过 auto mode / allowlists / sandboxing 降低审批疲劳

## 学完标准
- 你写任务时会同时给出目标、范围、约束、验证方式
- 你不再只说"帮我改一下"，而会说"改完跑哪些测试"
