# 25. 高手到底是怎么用 Claude Code 的：给中文单兵开发者的一封进阶信

## 重要程度
**S 级** - 极其重要。这一章不是继续介绍功能，而是直接回答一个更现实的问题：

**如果你在一个小公司里、没有英文社区、没有会用 Claude Code 的同事，你该怎么追上真正的高手？**

## 学习目标
- 看清英文高质量资料里，高手和普通用户的真正差距到底在哪里
- 知道你现在最可能缺的不是“更多插件”，而是“更成熟的使用系统”
- 建立一套适合单兵开发者的 Claude Code 进阶路线，而不是模仿大厂的复杂配置

## 学什么
- Anthropic 官方 `Best Practices`、`Memory`、`Skills`、`Hooks`、`Commands`、`Security`
- Anthropic 工程博客里关于 context engineering、auto mode、sandboxing、long-running harness 的一手结论
- 英文开发者圈少数真正值得看的高手经验，尤其是 Shrivu Shankar 和 Simon Willison

## 你需要掌握
- 高手和普通用户最大的差别，不在“提示词更花”，而在 **是否把 Claude Code 用成了一套工作系统**
- 你大概率不是缺“更多功能认知”，而是缺 **分层记忆、可复用流程、强制验证、重启策略、复盘机制**
- 小公司单兵用户不该照抄大厂全家桶；你需要的是 **最小但高杠杆** 的那套配置和习惯

## 一、先告诉你结论：你现在的迷茫是正常的
如果你已经会基础使用，也明显感受到 Claude Code 很强，但总觉得自己和高手差很远，这个感觉是对的。

因为英文互联网里真正高水平的 Claude Code 用法，已经不再是：
- 会不会 `/clear`
- 会不会写 `CLAUDE.md`
- 会不会开 subagent

而是：
- 怎么管理上下文预算
- 怎么分层管理记忆
- 怎么把高频流程沉淀成 skill
- 怎么用 hook 把“必须发生的动作”强制下来
- 怎么在长任务里做交接、重启、QA、复盘
- 怎么把 Claude Code 变成一个持续运转的个人操作系统

这类内容在中文互联网里确实很少，而且很多二手资料只会教你：
- “试试这个提示词”
- “装这个插件”
- “这个模式很厉害”

但高手真正重视的，不是这些表面动作。

## 二、高手和普通用户，真正差在哪
我把英文高质量资料反复对照后，发现差距主要在下面五层。

### 第一层：高手管理的是 context，不是 prompt
Anthropic 官方在 `Best Practices` 和 **2025-09-29** 的《Effective context engineering for AI agents》里讲得非常清楚：

- Claude 的 context window 很快会满
- 性能会随着 context 增长而下降
- 真正的工程问题，不只是“提示词怎么写”，而是“**什么信息应该进入上下文**”

这直接对应一个现实：

**普通用户喜欢把需求讲长，高手喜欢把上下文做薄、做准、做有结构。**

普通用户常见写法：
- 我之前做过很多尝试
- 这个项目大概是这样
- 你先看看哪里有问题

高手常见写法：
- 先看 `@src/auth` 和 `@src/session.ts`
- 只追 token refresh 相关逻辑
- 先复现这个失败用例，再修，再跑测试

高手不是“更会说”，而是更会 **做上下文裁剪**。

## 三、第二层：高手一定会给验证闭环
官方 `Best Practices` 写得很重：

**给 Claude 一个可以验证自己工作的方式，是最高杠杆动作。**

这意味着高手几乎不会只说：
- 帮我修一下
- 帮我重构一下
- 帮我优化一下

他们更常说：
- 先写失败用例复现
- 最后跑相关测试
- 截图前后对比
- build 成功才算完成
- 输出里要包含这个字段

这件事非常重要，因为它直接决定 Claude Code 是在：
- 产出“看起来像对的东西”

还是在：
- 通过反馈回路自我修正，逐步收敛到真的对

如果你只学一个高手习惯，我最建议你学这个。

## 四、第三层：高手不会把所有规则都塞进一个 `CLAUDE.md`
这是你现在最值得立刻升级的点。

Anthropic 官方 `Memory` 文档已经把这个体系讲得很清楚：
- `CLAUDE.md` 是你写的持久指令
- auto memory 是 Claude 自己记下来的 learnings
- 它们都只是 context，不是强制配置

而且官方给了几个特别关键的细节：
- auto memory 每次只加载前 **200 行或 25KB**
- 每个 `CLAUDE.md` 最好控制在 **200 行以内**
- 多个 `CLAUDE.md` 会被**拼接进上下文**，不是覆盖
- 如果某条内容是多步骤流程，或者只适用于代码库某一部分，就应该移到 **skill 或 path-scoped rule**

这几条合在一起，会导出一个高手式结论：

### 高手的记忆是分层的
#### 1. `~/.claude/CLAUDE.md`
放你个人在所有项目都通用的偏好。

例如：
- 先探索再修改
- 回答要简洁
- 优先用 `rg`
- 先给验证方式

#### 2. 项目根 `CLAUDE.md`
放这个仓库所有 session 都该知道的事实。

例如：
- 构建命令
- 测试命令
- 目录结构
- 团队约定
- 风险目录

#### 3. `.claude/rules/`
放局部规则。

例如：
- `frontend/` 下遵循什么 UI 规范
- `docs/` 下写什么文风
- `tests/` 下用什么测试风格

#### 4. auto memory
放 Claude 通过你纠偏逐渐学到的偏好和经验。

例如：
- 这个项目里哪些命令常用
- 某类错误怎么排查
- 你经常修正它的哪些行为

一句话记忆：

**高手不会写一本大而全的 `CLAUDE.md`，而是把记忆拆成“全局偏好 + 项目事实 + 局部规则 + 自动沉淀”。**

## 五、第四层：高手把“流程”做成 skills，把“硬约束”做成 hooks
这是中文互联网最容易讲虚、但英文高手真正用得很实的一层。

### 1. skill 不是“高级提示词收藏夹”
Anthropic 的 `Skills` 文档和工程博客《Equipping agents for the real world with Agent Skills》讲得很明确：

- skill 适合承载 **procedural knowledge**
- 也就是“怎么做一类事”的流程知识
- 只在调用或被判定相关时才加载，比把所有流程都塞进 `CLAUDE.md` 更省上下文

高手做 skill，通常不是为了炫，而是为了减少重复解释。

### 真正值得做成 skill 的，通常只有少数几类
- `bugfix`：复现 -> 定位 -> 写失败用例 -> 修复 -> 回归验证
- `review`：按风险、回归、边界、测试缺口审查
- `catchup`：读取当前分支 diff，总结现状和下一步
- `trace-flow`：从入口一路追执行链路
- `ship-doc`：把代码变更整理成 PR / README / release notes

官方现在还内置了一批 bundled skills，例如：
- `/debug`
- `/simplify`
- `/batch`
- `/loop`
- `/claude-api`

你可以把这件事理解成：

**Claude Code 官方自己也在往“流程 skill 化”这个方向走。**

### 2. hook 不是锦上添花，而是确定性控制
官方 `Hooks` 文档已经很成熟了，事件覆盖到：
- session 开始 / 结束
- user prompt 提交
- `PreToolUse` / `PostToolUse`
- `Notification`
- `ConfigChange`
- `SubagentStart`

英文高手对 hook 的评价普遍很高，因为它解决的是模型最不擅长的事情：

**“每次都必须执行”“绝对不能漏掉”的动作。**

例如：
- commit 前测试必须通过
- 改敏感目录必须提醒
- 出现 permission prompt 要桌面通知
- 改配置文件时插入额外上下文

### 一个很重要的高手共识
Shrivu Shankar 这篇文章里有个判断，我认为非常值钱：

- 少做 `block-at-write`
- 多做 `block-at-submit` 和 `hint hooks`

原因很简单：
- 在 `Edit` / `Write` 中途拦截，容易打断代理当前计划
- 在 commit、submit、PR、deploy 边界校验，通常更稳

这不是官方硬规则，但它和官方 hook 体系非常一致，也很符合工程直觉。

## 六、第五层：高手知道什么时候该重启，而不是一直硬聊
这是你现在特别值得补的一点。

官方 `Best Practices` 明确建议：
- 无关任务之间用 `/clear`
- 同一问题纠偏两次还不对，就重开
- 长任务里积极管理 context

Anthropic **2026-03-24** 的工程博客《Harness design for long-running application development》则进一步把这个问题讲透了：

- compaction 只是保留连续性
- reset 才是真正的 clean slate
- 长任务中，结构化 handoff artifact 非常重要

这背后的高手习惯是：

### 普通用户
- 同一个 session 一路硬聊到底
- 越聊越乱，再继续补充说明

### 高手
- session 脏了就重开
- 长任务先写交接文档再重开
- 把“重启”看成一种正常工作流，而不是失败

如果你最近经常遇到这些问题：
- Claude 开始复读旧思路
- 明明没验证就宣布完成
- 被纠正后还在同一路径打转
- 越聊质量越差

那通常不是你不够会提问，而是 **这个 session 该结束了。**

## 七、第六层：高手不会迷信 subagent、插件、MCP 越多越强
这一点很反直觉，但越看英文资料越明显。

### 1. subagent 并不是越多越好
官方当然支持 subagents、agent teams、多代理协作。

但 Anthropic 自己在《How we built our multi-agent research system》里也承认：
- 宽度优先的研究任务更适合多代理
- coding 并不总是天然适合多代理
- 协调和 token 成本都很高

Shrivu 甚至直接说：
- 很多自定义 specialist subagents 是脆弱的
- 更稳的做法，反而是把核心上下文写进 `CLAUDE.md`
- 再让主代理自己决定什么时候派 clone 出去

你不一定要完全照抄这个观点，但它至少提醒你：

**不要因为“这个能力高级”就急着上。**

### 2. 插件也不是越多越好
官方在 `Best Practices` 里提过：
- 如果你用的是 typed language，可以考虑 code intelligence plugin

但高手真正的态度通常更克制：
- 先把主工作流打磨好
- 只有当某个插件能明显提升导航、诊断或自动化时才接入

### 3. MCP 更适合做“安全边界”，不适合做“大而全工具箱”
Simon Willison 转评 Shrivu 时专门强调了一个很成熟的观点：

MCP 最有价值的角色，往往不是把整个外部系统 API 全搬进来，而是做成：
- 少数几个高价值动作
- 清晰的 auth / networking / security boundary
- 让代理能安全地接触外部世界

这和很多中文资料喜欢讲的“把所有系统都接成 MCP”完全不是一个方向。

## 八、如果我是你，我不会先追“装什么插件”，我会先做这套最小系统
你现在的处境，不适合做大而全方案。我更建议你先搭一个 **单兵版 Claude Code operating system**。

### 第一步：做一个短而硬的全局 `~/.claude/CLAUDE.md`
目标：
- 不超过 80-120 行
- 只写你所有项目都通用的偏好

建议包含：
- 先探索后执行
- 默认给验证方式
- 优先使用现有模式，不乱引库
- 修改前先定位影响范围
- 输出简洁，不要废话
- 连续两次纠偏失败就建议重开 session

### 第二步：每个项目有一个项目级 `CLAUDE.md`
目标：
- 100-150 行左右
- 只放这个项目每次都需要的事实

建议包含：
- build / test / lint 命令
- 目录结构
- 代码规范
- 文档规范
- 绝对不要动的路径
- 提交前最低验证标准

### 第三步：只做 3 个 skill
先别做十几个。

我建议你先做：
- `catchup`
- `bugfix`
- `review`

这三个已经足够让你和过去的自己拉开明显差距。

### 第四步：只做 2 个 hook
- 一个 `Notification`：权限请求或空闲时提醒你
- 一个提交边界 hook：例如 commit 前没跑测试就阻断

### 第五步：每周做一次复盘
复盘最近 5-10 个高价值 session：
- 哪些错误重复出现
- 哪些说明值得写进 `CLAUDE.md`
- 哪些流程值得做成 skill
- 哪些动作必须做成 hook
- 哪些任务其实应该早重开

高手不是“每次临场发挥更神”，而是会不断把经验 **沉淀回系统**。

## 九、你最该马上开始用的几个命令
英文高质量资料里，很多高手不是靠插件，而是把内置能力用到位。

### `/init`
官方现在支持更强的初始化流。设置 `CLAUDE_CODE_NEW_INIT=1` 后，`/init` 不只会帮你生成 `CLAUDE.md`，还会把 skills、hooks 一起纳入初始化讨论。

### `/memory`
用来检查和编辑：
- `CLAUDE.md`
- `CLAUDE.local.md`
- auto memory

这对“我到底记住了什么、为什么没生效”特别重要。

### `/context`
可视化当前 context 使用情况。

这对培养高手最关键的直觉之一非常有用：

**哪些行为在偷偷吃掉你的上下文预算。**

### `/insights`
这是很容易被忽视的命令，但对你这种“一个人用、没有同伴交流”的场景反而很值。

它会分析你的 Claude Code sessions、interaction patterns 和 friction points。你可以把它当成：

**让 Claude 反过来帮你分析“我到底怎么在用 Claude Code”。**

### `/skills`
这里有个小细节很值钱：官方支持按 token count 排序。

这能帮助你识别：
- 哪些 skill 太胖
- 哪些 skill 正在偷偷拖累上下文

### `/fewer-permission-prompts`
官方现在直接做成 bundled skill 了。它会分析 transcript，并帮你生成优先级更高的 allowlist 建议。

这特别适合你这种已经明显在高频使用、但又不想手工调一堆权限的人。

## 十、你现在最不该做的三件事
### 1. 不要再追“神 prompt”
真正的高手差距已经不在这里。

### 2. 不要一上来装很多 MCP / 插件 / subagents
你现在更需要的是更稳定的主工作流，而不是更复杂的能力面。

Anthropic 在 context engineering 博客里直接点名：

**bloated tool sets** 是很常见的失败模式。  
如果连人类都说不清什么时候该用哪个工具，代理更不可能稳定选对。

### 3. 不要把所有纠偏都留在聊天里
如果一个纠偏值得说第二次，它就该开始考虑：
- 写进 `CLAUDE.md`
- 做成 rule
- 做成 skill
- 做成 hook

否则你永远在重复劳动。

## 十一、真正像高手那样使用 Claude Code，意味着什么
如果把我这次搜索英文高质量资料后的结论压缩成一句话，那就是：

**高手不是更会“问 AI”，而是更会设计一个让 agent 稳定工作的环境。**

这个环境至少包括：
- 干净的 context
- 短而硬的持久规则
- 少量高价值 skills
- 强制验证的 hooks
- 及时重启的习惯
- 会复盘、会沉淀的使用方式

当这些搭起来以后，你和“高手”的差距就不再主要是英语，而是时间和练习量。

这也是我最想告诉你的事：

**你现在不是缺天赋，也不是缺某个神秘技巧。你缺的是一套更成熟的 Claude Code 工作系统。**

## 十二、如果你现在只做三件事
1. 写好你的全局 `~/.claude/CLAUDE.md`，并且控制在短而硬的范围内。
2. 只做三个 skill：`catchup`、`bugfix`、`review`。
3. 每周复盘最近几个 session，把重复纠偏沉淀成规则、skill 或 hook。

如果你能连续做四周，你会明显感觉自己从“会用 Claude Code”进入“开始掌控 Claude Code”。

## 十三、和前后章节的关系
- 第 23 章讲的是高手技巧
- 第 24 章讲的是真实工作实践
- 第 25 章讲的是：如果你是一个没有英文环境、没有高手同伴、但想快速进阶的单兵开发者，真正该怎么追上

## 学完标准
- 你知道高手差距主要不在提示词，而在系统设计
- 你知道该如何分层管理记忆、skills、hooks
- 你知道为什么 session 重启和复盘是进阶关键
- 你知道现在最该补的是哪几项，而不是继续盲目搜功能
- 你已经能开始建立自己的 Claude Code operating system

## 关键英文来源（截至 2026-04-25）
### 官方文档
- Best Practices for Claude Code  
  https://code.claude.com/docs/en/best-practices
- How Claude remembers your project  
  https://code.claude.com/docs/en/memory
- Extend Claude with skills  
  https://code.claude.com/docs/en/skills
- Hooks reference  
  https://code.claude.com/docs/en/hooks
- Commands  
  https://code.claude.com/docs/en/commands
- Security  
  https://code.claude.com/docs/en/security

### Anthropic 工程博客
- Effective context engineering for AI agents  
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Harness design for long-running application development  
  https://www.anthropic.com/engineering/harness-design-long-running-apps
- Claude Code auto mode: a safer way to skip permissions  
  https://www.anthropic.com/engineering/claude-code-auto-mode
- Making Claude Code more secure and autonomous with sandboxing  
  https://www.anthropic.com/engineering/claude-code-sandboxing
- Equipping agents for the real world with Agent Skills  
  https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- How we built our multi-agent research system  
  https://www.anthropic.com/engineering/multi-agent-research-system

### 高质量实践者经验
- Shrivu Shankar, *How I Use Every Claude Code Feature*  
  https://blog.sshh.io/p/how-i-use-every-claude-code-feature
- Simon Willison, *How I Use Every Claude Code Feature*  
  https://simonwillison.net/2025/Nov/2/how-i-use-every-claude-code-feature/
- Simon Willison, *Using Claude Code to build a GitHub Actions workflow*  
  https://simonwillison.net/2025/Jul/1/claude-code-github-actions/
- Simon Willison, *Clinejection*  
  https://simonwillison.net/2026/Mar/6/clinejection/

### 我为什么采用这些来源
- 官方文档：确认功能边界、最佳实践、记忆结构、hooks、skills、命令与安全模型
- 工程博客：确认 Anthropic 团队自己已经验证过的方法，而不是二手转述
- Shrivu / Simon：补上“高手实际怎么用”的高价值经验，但只采用与官方材料不冲突、且被实践验证的部分
