# 8. 权限体系：Permission Modes、Allowlists、Sandboxing、Auto Mode

## 重要程度
**S 级** - 必须掌握，不会就很难高效用

## 学习目标
- 理解 Claude Code 的安全与自动化边界
- 让它少烦你，但又不失控

## 学什么
- permission modes
- permission rules
- allowlists
- sandboxing
- auto mode

## 你需要掌握
- Claude 默认会对写文件、Bash 命令、MCP 工具等请求权限
- 官方 best practices 推荐三条减少打断的路径：auto mode、permission allowlists、sandboxing
- auto mode 会让独立分类器判断哪些动作需要拦截
- 最新版本里 auto mode 已不再需要 --enable-auto-mode
- VS Code 支持 normal、plan、auto-accept 等权限模式切换

## 学完标准
- 你知道什么时候用 normal / plan / auto
- 你知道什么时候该用 allowlist，什么时候该用 sandbox
