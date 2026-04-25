# 28. 我的个人 Claude Code 作战手册：给中国单兵开发者的最小高杠杆配置

## 重要程度
**S 级** - 极其重要。第 27 章回答的是“高手世界里默认知道什么”，这一章回答的是：

**如果你现在就要把 Claude Code 用起来，而且你是中国小公司里的单兵开发者，你到底该怎么配、怎么用、先做什么、不要做什么。**

## 学习目标
- 建立一套适合单兵开发者的最小 Claude Code 系统
- 知道哪些配置先做最值，哪些以后再做
- 拿到一组可以直接照抄再按需修改的工作模板

## 你需要掌握
- 你现在不需要大而全方案，你需要的是 **最小但高杠杆**
- 先把主工作流打磨好，比先装很多 skill / MCP / subagent 更重要
- 你的目标不是“会几个命令”，而是 **形成一套稳定作战方法**

## 一、先给你最终答案：你现在只需要搭 5 样东西
如果我是你，我不会让你一上来做十几件事。

你现在先搭这 5 样：

1. 一个短而硬的全局 `~/.claude/CLAUDE.md`
2. 一个项目级 `CLAUDE.md`
3. 三个 skill：`catchup`、`bugfix`、`review`
4. 两个 hook：`Notification`、提交边界校验
5. 一套固定 prompt 模板

把这五样做好，你的水平就会明显从“会用 Claude Code”进入“开始掌控 Claude Code”。

## 二、你的全局 `~/.claude/CLAUDE.md` 应该怎么写
它的作用不是介绍项目，而是规定你在所有项目里的工作偏好。

### 原则
- 控制在 80-120 行
- 只写所有项目都通用的偏好
- 不写项目事实
- 不写太长流程

### 推荐骨架
```md
# Working Style
- First explore, then implement.
- Prefer minimal changes over rewrites.
- Always propose a validation path for non-trivial tasks.

# Tool Use
- Prefer rg / rg --files for search.
- Read relevant files before editing.
- Do not make assumptions when the codebase can answer the question.

# Coding Style
- Follow existing project patterns.
- Avoid unnecessary new dependencies.
- Keep changes small and easy to verify.

# Session Discipline
- If the task drifts after two corrections, suggest a reset.
- For long tasks, summarize completed work, remaining work, and risks before compacting or handing off.

# Output
- Be concise.
- Summarize: what changed, how it was validated, remaining risks.
```

### 你应该额外加上的两条
因为你是单兵开发者，我建议再加：

- 默认先给我一个计划，再做多文件改动
- review 时优先指出风险、回归和测试缺口，而不是先写总结

## 三、你的项目 `CLAUDE.md` 应该怎么写
项目 `CLAUDE.md` 不是“写给未来读者的文档”，而是“写给 Claude 的项目事实卡片”。

### 原则
- 控制在 100-150 行
- 只放这个项目每次都该知道的事实
- 流程太长就拆到 skill

### 推荐骨架
```md
# Project Overview
- What this project does
- Main directories

# Core Commands
- install:
- dev:
- test:
- lint:
- build:

# Constraints
- Follow existing patterns
- Do not modify generated files
- Keep public API backward compatible unless explicitly requested

# Validation
- Run related tests after code changes
- Run build for production-impacting changes

# Risk Areas
- auth/
- billing/
- database migrations/

# Documentation
- Update README or docs when behavior changes
```

### 什么应该放进去
- 构建命令
- 测试命令
- 目录结构
- 风险目录
- 禁止事项
- 最低验证要求

### 什么不该放进去
- 你个人的输出偏好
- 很长的发布流程
- 一次性任务打法
- 只对某个目录生效的细则

## 四、只做三个 skill，先不要贪多
这是最重要的建议之一。

你现在最容易犯的错，是做 12 个 skill，然后没有一个真正稳定好用。

先做三个。

## 1. `catchup`
作用：
- 读当前分支 diff
- 读相关文件
- 总结项目现状
- 告诉你下一步做什么

### 它解决什么问题
- 你隔了半天或一天回来，不知道做到哪了
- 你切了新 session，不想从头讲一遍
- 你想快速让 Claude 进入当前上下文

### 推荐内容
```md
---
name: catchup
description: Summarize current branch status, recent changes, relevant files, and propose next steps.
---

1. Read current git diff/status.
2. Read the most relevant changed files.
3. Summarize:
   - current goal
   - what is already done
   - what is incomplete
   - likely next step
   - validation status
4. Keep it concise.
```

## 2. `bugfix`
作用：
- 把修 bug 变成固定节奏

### 它解决什么问题
- Claude 很容易拍脑袋改
- 你自己也容易急着“先修了再说”

### 推荐内容
```md
---
name: bugfix
description: Reproduce, isolate root cause, add a failing test when possible, fix minimally, then verify.
---

When fixing a bug:
1. Reproduce or restate the failing behavior clearly.
2. Narrow down likely files and execution path.
3. Explain the most likely root cause before editing.
4. Add a failing test if practical.
5. Fix with the smallest change that addresses the root cause.
6. Run relevant validation and summarize remaining risks.
```

## 3. `review`
作用：
- 把 review 固定成风险导向

### 它解决什么问题
- Claude 很容易把 review 写成总结
- 你自己也容易被“看起来差不多”骗过去

### 推荐内容
```md
---
name: review
description: Review changes for regressions, edge cases, missing validation, and overreach. Focus on findings first.
---

Review with this order:
1. Behavioral regressions
2. Error handling issues
3. Test gaps
4. Unnecessary scope expansion
5. Maintainability concerns

Output findings first. Keep summary brief.
```

## 五、只做两个 hook，先把最值钱的做起来
你现在不需要复杂自动化。

先做两个。

## 1. `Notification`
用来提醒：
- 权限请求
- 长任务完成
- Claude 需要你回应

为什么值钱：
- 你可以一边让 Claude 干活，一边做别的事
- 不需要一直盯着终端

## 2. 提交边界校验
用来：
- 没跑测试不允许提交
- 改到敏感目录时额外提醒

为什么值钱：
- 这类错误最常见
- 又不该靠记性

### 我不建议你现在先做的 hook
- 大量 `block-at-write`
- 很复杂的多阶段自动审核
- 到处注入长提示

因为你现在最需要的是稳定，而不是花哨。

## 六、你的标准开发工作流，应该固定成这样
这是本章最核心的部分。

## 场景 1：接手一个新项目
### 第一步
```text
/init
```

### 第二步
```text
/memory
```

### 第三步
发这条 prompt：

```text
先不要改代码。
先给我这个仓库的整体地图：目录结构、入口、主流程、构建方式、测试方式、风险目录。
然后告诉我如果要改 xxx，应该先看哪几处文件。
```

### 第四步
把真正“以后每次都值得知道”的事实写进项目 `CLAUDE.md`

## 场景 2：接到一个新需求
固定模板：

```text
目标：...
范围：重点看哪些目录 / 文件
约束：不要怎么做，要沿用什么模式
验证：改完如何确认是对的
```

例如：

```text
目标：新增邮箱登录失败时的错误提示
范围：重点看 auth 页面、提交逻辑和错误展示组件
约束：沿用现有 UI 组件和错误处理模式，不引入新库
验证：补一个相关测试，手动验证错误提示出现且不影响正常登录
```

## 场景 3：需求复杂，不要直接改
固定模板：

```text
先不要改代码。
先分析现状、相关文件、可能改动范围、风险和验证方式。
给我一个实施计划。
```

## 场景 4：真正开始开发
固定模板：

```text
先读相关文件并确认当前实现方式。
按最小改动实现，不要重写。
改完跑相关验证，并总结改动、验证结果和剩余风险。
```

## 场景 5：修 bug
固定模板：

```text
这里有一个可复现错误：...
先解释最可能的根因，不要拍脑袋改。
如果可行，先写失败用例，再最小修改修复，最后跑相关验证。
```

## 场景 6：补测试
固定模板：

```text
找出这个模块里最容易出问题、但还没被覆盖的逻辑。
按现有测试风格补测试。
重点覆盖边界条件、错误路径和回归风险。
```

## 场景 7：做 review
固定模板：

```text
现在不要继续实现。
请按 reviewer 视角检查当前改动。
优先指出行为回归、错误处理问题、测试缺口和是否存在过度改动。
发现的问题先列出来，最后再做简短总结。
```

## 场景 8：准备 commit / PR
固定模板：

```text
先总结当前改动。
然后列出我实际做过的验证。
再列出剩余风险和回滚点。
最后写一个简洁的 commit message 和 PR 描述。
```

## 七、长任务失控时，你该怎么处理
这是你最该建立的能力之一。

## 症状 1：只是上下文变长
用：

```text
/compact 只保留目标、已完成项、未完成项、验证结果和已排除路径
```

## 症状 2：已经明显跑偏
用：

```text
/clear
```

然后新开一轮。

## 症状 3：任务很长，不能直接丢
先让 Claude 写交接：

```text
把当前任务的目标、已完成内容、未完成项、失败尝试、下一步建议整理成 markdown 交接文档。
```

然后新开 session：

```text
读这个交接文档，继续做，不要重复已经排除的路径。
```

## 八、哪些东西该进 memory，哪些该做成 skill
这是你现在最需要彻底想清楚的。

## 应该进项目 memory 的
- build / test / lint 命令
- 项目结构
- 架构约束
- 风险目录
- 团队固定规范
- 每次都需要知道的事实

## 应该做成 skill 的
- 修 bug 流程
- review 流程
- catchup 流程
- 发布检查清单
- 某类问题的固定排查剧本

## 应该做成 hook 的
- 每次都必须发生的动作
- 每次都必须提醒的风险
- 每次都不能漏掉的边界检查

一句话记忆：

**事实进 memory，流程进 skill，强制动作进 hook。**

## 九、你现在最不该做的事
### 1. 不要先做很多 skill
三个先做稳，比十个做得半吊子强。

### 2. 不要把所有项目规则都放 skill
这会让 Claude 每次都靠“有没有触发 skill”来决定是否知道项目事实。

### 3. 不要迷信长 session
会 reset 的人才是高手。

### 4. 不要一上来装很多 MCP
先把主工作流打顺，再决定哪里真的需要接外部系统。

### 5. 不要只让 Claude 干活，不让它总结
你应该让 Claude 帮你做：
- handoff
- review
- changelog
- risk summary
- memory draft

## 十、你的每周固定动作
如果你想真正进步，我建议你每周固定做一次很短的复盘。

### 每周五花 15-20 分钟看 5 件事
1. 这一周 Claude 最常犯什么错
2. 哪些纠偏我重复说了两次以上
3. 哪些事实应该补进项目 `CLAUDE.md`
4. 哪些流程应该做成 skill
5. 哪些动作该做成 hook

这一步会决定你是一直在原地用工具，还是在不断升级自己的系统。

## 十一、如果你现在只做 7 件事
1. 写全局 `~/.claude/CLAUDE.md`
2. 写项目 `CLAUDE.md`
3. 做 `catchup`
4. 做 `bugfix`
5. 做 `review`
6. 做 `Notification` hook
7. 做提交边界校验 hook

先做完这 7 件，再谈更复杂的能力。

## 十二、这一章真正想给你的，不是配置，而是作战方法
你现在最需要的，不是更多知识点，而是一套能每天重复执行的作战方法。

这套方法可以压缩成一句话：

**先把规则搭好，再把任务说清，再让 Claude 去做，再独立 review，再把经验沉淀回系统。**

做到这一点，你和高手的差距就不会再主要是“英语”或“信息差”，而会越来越变成时间和练习量的差距。

## 十三、和前后章节的关系
- 第 27 章讲的是高手世界里的默认常识和高层工作方法
- 第 28 章讲的是：如果你就是那个中国小公司里的单兵开发者，你今天到底该怎么搭第一套真能用的 Claude Code 系统

## 学完标准
- 你知道自己的第一套 Claude Code 系统该怎么搭
- 你知道全局 memory、项目 memory、skill、hook 各自先做什么
- 你拿到了可以直接复用的 prompt 模板
- 你已经有了一套适合自己长期使用的最小工作流

## 关键英文来源（截至 2026-04-25）
### 官方文档
- How Claude remembers your project  
  https://code.claude.com/docs/en/memory
- Extend Claude with skills  
  https://code.claude.com/docs/en/skills
- Hooks reference  
  https://code.claude.com/docs/en/hooks
- Commands  
  https://code.claude.com/docs/en/commands
- Common workflows  
  https://code.claude.com/docs/en/common-workflows
- Best Practices for Claude Code  
  https://code.claude.com/docs/en/best-practices

### Anthropic 工程博客
- Effective context engineering for AI agents  
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Harness design for long-running application development  
  https://www.anthropic.com/engineering/harness-design-long-running-apps

### 高质量实践者经验
- Shrivu Shankar, *How I Use Every Claude Code Feature*  
  https://blog.sshh.io/p/how-i-use-every-claude-code-feature
- Simon Willison, *How I Use Every Claude Code Feature*  
  https://simonwillison.net/2025/Nov/2/how-i-use-every-claude-code-feature/
- Jesse Vincent / obra, *superpowers*  
  https://github.com/obra/superpowers

## 我为什么采用这些来源
- 官方文档：确认 memory、skills、hooks、commands、workflows 的正式边界
- 工程博客：确认长任务、context、reset、handoff 这些高手最在意的问题
- Shrivu / Simon / Jesse：把高手真实工作流里的“作战方法”补全，而不是只停留在官方说明层
