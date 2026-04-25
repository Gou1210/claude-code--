# 6. Memory：CLAUDE.md、auto memory 与 .claude 目录

## 重要程度
**S 级** - 必须掌握，不会就很难高效用

## 学习目标
- 知道 Claude Code 如何记住项目
- 知道项目级配置和个人级配置如何分层

## 学什么
- Memory
- CLAUDE.md vs auto memory
- .claude 目录结构
- ~/.claude 与项目内 .claude
- CLAUDE.local.md

## 你需要掌握
- Claude Code 有两套互补记忆系统：CLAUDE.md 与 auto memory
- 它们都会在会话开始时加载，但属于"上下文"，不是硬性配置
- Claude Code 会从项目目录和 ~/.claude 中读取 instructions、settings、skills、subagents、memory 等
- CLAUDE.local.md 是当前项目私有偏好文件，适合 .gitignore
- 子目录下的 CLAUDE.md 会在 Claude 读到该目录文件时按需加载，不一定一开始就进上下文

## 最值得先学的内容
- 怎么写项目根目录 CLAUDE.md
- 什么适合写进 CLAUDE.md
- 什么适合用 auto memory
- 什么适合单独写到 skill / subagent / hook 里

## 学完标准
- 你能写出一份像样的项目 CLAUDE.md
- 你知道哪些规则应该长期加载，哪些不该
