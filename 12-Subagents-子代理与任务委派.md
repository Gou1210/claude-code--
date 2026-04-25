# 12. Subagents：子代理与任务委派

## 重要程度
**A 级** - 强烈建议掌握。Subagents 是 Claude Code 从“单线程会话”升级到“隔离上下文、角色化执行、并行委派”的关键机制。

## 学习目标
- 理解 subagent 的本质不是“多开一个 Claude”，而是“把某类任务隔离出去”
- 掌握 subagent 的定义方式、能力控制、调用方式和典型场景
- 学会判断什么时候该主会话做，什么时候该交给 subagent

## 学什么
- built-in subagents
- custom subagents
- `.claude/agents/`
- frontmatter 字段
- tools / permissionMode / mcpServers / hooks / memory / worktree
- 自动委派、显式调用、前后台运行
- subagent 与 skills / main conversation / agent teams 的边界

## 你需要掌握
- subagent 的核心价值是 **隔离上下文污染**
- subagent 可以有自己的 system prompt、工具权限、模型、memory、hooks、MCP 范围
- 截至 **2026-04-25** 的官方文档里，subagents 已经不是简单实验功能，而是一整套成熟能力
- 第 12 章真正要学会的不是“怎么写一个 agent 文件”，而是“什么任务应该委派”

## 一、先纠正误解：subagent 不是“更强的 Claude”
很多人初看 subagents，会以为它的价值是：
- 多一个助手
- 更专业的模型
- 更自动化

这些都只是表层。

真正的价值是：

**把高噪音、高输出、高搜索量、强角色化的任务，从主会话上下文里拆出去。**

主会话负责：
- 目标定义
- 综合判断
- 最终实施路线

subagent 负责：
- 调查
- 验证
- 专项分析
- 受限执行

## 二、官方为什么强调 subagents
官方文档现在对 subagents 的定位非常明确：
- 每个 subagent 有独立 context window
- 可以有专门 system prompt
- 可以限制可用工具
- 可以独立 permission mode
- 可以选择不同模型

这直接带来四个收益：
- 保持主会话干净
- 让角色更专注
- 限制能力边界更安全
- 便于跨项目复用

## 三、Claude Code 自带哪些 built-in subagents
官方文档当前列出的 built-in subagents 里，最重要的是这些：

### `Explore`
偏研究型。

特点：
- 用于搜索、理解代码库
- 更适合做调查而不是实施

### `Plan`
plan mode 下的研究型子代理。

特点：
- 只读调查
- 为计划产出上下文

### `general-purpose`
通用执行型。

特点：
- 兼顾探索和行动
- 适合复杂多步任务

此外还有一些辅助型内建 agent，例如：
- 配置状态栏的 helper
- 回答 Claude Code 自身文档问题的 guide 型 agent

## 四、什么时候最该用 subagent
### 1. 大量读文件的调查
例如：
- 查认证链路
- 找复用组件
- 梳理模块历史

### 2. 输出很多的工作
例如：
- 跑测试
- 分析日志
- 抓 PR diff
- 检查一堆错误

### 3. 需要角色化视角
例如：
- security reviewer
- debugger
- db analyst
- code reviewer

### 4. 需要受限工具
例如：
- 只允许读，不允许写
- 只允许 Bash，不允许 Edit
- 只允许特定 MCP

## 五、什么时候不要急着上 subagent
官方也明确给出反面边界。

更适合主会话的情况：
- 需要频繁来回确认
- 多个阶段共享大量上下文
- 只是快速小改
- 对延迟很敏感

一句话判断：

**如果任务适合“拿到摘要后再决策”，很适合 subagent；如果任务需要你和 Claude 高频来回磨细节，更适合主会话。**

## 六、subagent 放在哪，决定作用域
官方支持这些来源：

### 项目级
- `.claude/agents/`
- 适合团队共享

### 用户级
- `~/.claude/agents/`
- 适合你自己跨项目复用

### CLI 临时定义
- `claude --agents '{...}'`
- 适合快速实验和脚本场景

### 插件提供
- 来自 plugin 的 `agents/`

### 优先级
官方给出的顺序是：
- managed
- CLI
- project
- user
- plugin

如果同名，优先级高的覆盖低的。

## 七、subagent 文件怎么写
本质上是：
- YAML frontmatter
- 后面跟 Markdown 形式的 system prompt

最小示例：

```md
---
name: code-reviewer
description: 专门做代码审查。代码修改后应主动使用。
tools: Read, Glob, Grep, Bash
model: sonnet
---

你是资深代码审查工程师。
优先关注代码质量、安全性、可维护性和测试覆盖。
```

这里要注意：

**subagent 拿到的是你给它的 system prompt，不是完整主会话系统提示。**

## 八、最重要的 frontmatter 字段
官方字段不少，但第一轮必须先掌握这些。

### `name`
唯一标识。

### `description`
Claude 会根据它判断何时自动委派。

### `tools`
白名单。只允许这些工具。

### `disallowedTools`
黑名单。从继承工具集中剔除。

### `model`
可选 `sonnet`、`opus`、`haiku`、完整模型 ID 或 `inherit`。

### `permissionMode`
可设：
- `default`
- `acceptEdits`
- `auto`
- `dontAsk`
- `bypassPermissions`
- `plan`

### `skills`
把某些 skill 的全文预注入 subagent 上下文。

### `mcpServers`
给 subagent 单独配置或限定 MCP。

### `hooks`
给这个 subagent 生命周期挂 hooks。

### `memory`
启用跨会话 memory。

### `background`
是否默认后台运行。

### `effort`
该 subagent 的思考深度。

### `isolation: worktree`
在独立 worktree 里运行，避免和主工作区冲突。

### `initialPrompt`
当它作为主 session agent 运行时的初始提示。

## 九、工具控制是 subagent 设计的第一原则
官方最佳实践里反复强调：

**不要默认给满权限。**

### 好的例子
- code reviewer：只读
- debugger：读 + edit + bash
- db-reader：只允许 Bash，再用 hook 限定只读 SQL

### 不好的例子
- 一个 agent 什么都能干
- 既能部署又能删文件又能改数据库又能发消息

subagent 设计越聚焦，自动委派越稳定，行为越可预测。

## 十、自动委派是怎么发生的
官方现在的机制是：
- Claude 看你的任务描述
- 看 subagent 的 `description`
- 看当前上下文
- 决定是否委派

所以你写 `description` 时，不要写成抽象口号，而要写成：
- 做什么
- 什么时候用
- 是否主动使用

例如：
```text
Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
```

这种写法明显比“review code”更容易被正确触发。

## 十一、显式调用 subagent 的三种方式
官方给了三个层级。

### 1. 自然语言提及
```text
Use the code-reviewer subagent to check these auth changes
```

Claude 一般会理解并委派，但仍保留自主判断。

### 2. `@` 指定
这是更强制的方式。

概念上类似：
- `@"code-reviewer (agent)"`
- 或手动 `@agent-code-reviewer`

适合你明确知道要跑哪个 agent。

### 3. 整个 session 直接作为某个 agent 运行
```bash
claude --agent code-reviewer
```

或者在 settings 里设置 `agent`。

这时主线程本身就带着那个 agent 的 system prompt、模型和工具限制运行。

## 十二、前台与后台：subagent 不一定要堵住主会话
官方明确区分：

### Foreground subagent
- 阻塞主会话
- 权限提示和澄清问题会传给你

### Background subagent
- 后台并发执行
- 启动前先做所需权限的预批准
- 运行中缺权限通常会自动拒绝，而不是反复打断

### 什么时候适合后台
- 跑测试
- 长日志分析
- 平行调查多个模块

### 什么时候适合前台
- 可能需要你澄清
- 需要实时观察
- 风险较高

## 十三、subagent 的典型高收益模式
### 1. 隔离高输出操作
```text
用一个 subagent 跑测试，只把失败项和错误摘要返回给我
```

### 2. 并行研究
```text
并行调查认证、数据库、API 三个模块，然后汇总给我
```

### 3. 链式处理
```text
先让 code-reviewer 找性能问题，再让 optimizer 处理这些问题
```

### 4. 实现后复核
```text
实现完成后，用 subagent 审查边界情况和一致性风险
```

这类用法能明显降低主会话上下文噪音。

## 十四、subagent 和 skill 的根本区别
这是最容易混淆的地方。

### Skill
- 重点是复用知识或流程
- 默认运行在主会话上下文
- 本质是按需加载的 prompt / playbook

### Subagent
- 重点是隔离上下文和角色化执行
- 有自己独立上下文
- 可以有独立模型、工具、权限、memory

一句话区分：

**skill 解决“怎么复用套路”，subagent 解决“怎么隔离做事”。**

## 十五、`skills` 字段：让 subagent 一出生就带上某些知识
官方允许在 subagent frontmatter 里写：

```yaml
skills:
  - api-conventions
  - error-handling-patterns
```

这意味着：
- 这些 skill 的全文会在 subagent 启动时直接注入
- subagent 不会自动继承主会话当前已加载的 skills

这很适合：
- 专职 API agent
- 专职 reviewer
- 带团队规范的特定 agent

## 十六、memory：让 subagent 变成持续积累经验的角色
官方支持三种 memory scope：
- `user`
- `project`
- `local`

### 推荐理解
- `project`：默认最实用，能团队共享
- `user`：跨项目通用经验
- `local`：只给你自己，不入库

当 memory 开启后，subagent 会得到：
- 自己的 memory 目录
- `MEMORY.md` 内容注入
- 自动管理 memory 所需的读写工具

这意味着某些长期角色，例如：
- reviewer
- debugger
- architecture analyst

可以逐步积累项目知识。

## 十七、hooks 与 MCP：subagent 可以有更细的执行边界
### hooks
可以用来：
- 校验 Bash 命令
- 编辑后自动 lint
- 启停时做 setup / cleanup

### MCP
可以让某个 subagent 独享特定外部工具，而不暴露给主会话。

这两个能力放在一起非常强：

**你可以做一个会查库、但只能读；会用浏览器、但只在自己上下文里；会跑脚本、但每次都先经过钩子校验的 agent。**

## 十八、worktree isolation：并行执行时避免相互踩文件
官方支持：

```yaml
isolation: worktree
```

它的作用是：
- 给 subagent 一个隔离的代码副本
- 适合并行修改、互不干扰
- 没有改动时可自动清理

这对多代理并行尤其关键，也会为下一章的 Agent Teams 埋下基础。

## 十九、官方给出的典型 subagent 示例，值得直接借鉴
### code-reviewer
只读审查型。

### debugger
可分析也可修复，强调根因。

### data-scientist
面向 SQL / BigQuery / 数据分析。

### db-reader
通过 Bash + PreToolUse hook 限定只读查询。

这些例子说明：

**最好的 subagent 不是“全能型”，而是“职责极清晰”。**

## 二十、这一章最该建立的委派判断
推荐你脑子里有这样一条判断链：

1. 这件事会不会产生很多无用输出？
2. 这件事是不是适合一个专门角色处理？
3. 这件事需不需要更严格的工具限制？
4. 这件事做完后，我是不是只想要摘要而不是全过程？

如果四个问题里有两个以上回答“是”，通常就该考虑 subagent。

## 二十一、和前后章节的关系
- 第 11 章讲的是“把知识和流程封装起来”
- 第 12 章讲的是“把任务和上下文隔离出去”
- 第 13 章会继续升级到多个代理之间如何协同，也就是 Agent Teams

## 学完标准
- 你知道 subagent 的第一价值是隔离上下文，而不是炫技式多代理
- 你会写基础 `.claude/agents/*.md`，并理解 `description`、`tools`、`permissionMode`、`skills`、`memory`、`isolation` 的作用
- 你知道何时该主会话处理，何时该委派给 subagent
- 你已经能设计出 2 到 4 个适合自己项目的专职 subagents
