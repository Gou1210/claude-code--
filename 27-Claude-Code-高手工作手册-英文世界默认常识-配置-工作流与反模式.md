# 27. Claude Code 高手工作手册：英文世界默认常识、配置、工作流与反模式

## 重要程度
**S 级** - 极其重要。前面的章节更像体系化课程，这一章是另一种东西：

**如果你不会英语、身边没有会用 Claude Code 的人，但你又必须把它真的用到高手水准，这一章就是写给你的。**

## 这章不是写给谁的
不是写给只想看几个 prompt 技巧的人。  
不是写给只想知道“这个命令怎么用”的人。  
不是写给把 Claude Code 当聊天工具的人。

这章写给：
- 已经知道 Claude Code 很强
- 但知道自己现在的用法还很浅
- 想补上英文世界里高手默认知道的常识
- 想像真正的 senior engineer 一样，把 Claude Code 放进日常开发工作流的人

## 学习目标
- 搞清楚英文世界里高手到底在怎么用 Claude Code
- 分清哪些是官方硬规则，哪些是高手共识，哪些仍属实验性流派
- 建立一套适合中文单兵开发者的最小高杠杆用法，而不是盲目追生态噪音

## 先说结论
我这次重新查了一轮资料后，结论非常明确：

**高手和普通用户最大的差距，不是模型、不是 prompt，更不是装了多少插件。**

真正的差距是这五件事：

1. 高手会管理 context，而不是放任上下文膨胀
2. 高手会给验证闭环，而不是靠肉眼收尾
3. 高手会把规则、流程、强约束分层，而不是全塞进一个地方
4. 高手会主动重启、分叉、交接 session，而不是在脏上下文里死磕
5. 高手会把自己的工作流“产品化”，而不是每次临场发挥

这五件事，官方文档都有影子；但真正把它们连成体系的，更多来自英文世界里一线重度用户和开源工作流框架。

## 一、先分清三类知识：官方规则、高手共识、实验流派
这是你过去最容易混淆的地方。

### 第一类：官方规则
这类内容以 Anthropic 官方文档和工程博客为准。

例如：
- 给 Claude 验证方式是最高杠杆动作
- `CLAUDE.md` 适合长期事实和规则
- auto memory 是 per-worktree 的
- skill 是按需加载的流程能力
- 多工具重叠太多会让 agent 更容易选错
- sandbox 和 auto mode 的重点是安全边界

这类内容比较稳。

### 第二类：高手共识
这类内容通常不是官方写成“规则”，但在顶级实践者文章、GitHub 讨论和高关注工作流框架里反复出现。

例如：
- `CLAUDE.md` 应该短而硬，不该长成操作手册
- 长任务里 reset 往往比单纯 compaction 更稳
- review 最好和实现分开
- worktree 是多 session 并行的默认搭配
- 好的 skill 是“流程压缩器”，不是“长提示词仓库”
- 高手通常不会一上来就接很多 MCP

这类内容很重要，因为它们往往就是英文世界的“默认常识”。

### 第三类：实验流派
这类内容已经很火，但你不应该把它们当成“所有高手都必须这样做”的标准答案。

例如：
- `obra/superpowers`
- `get-shit-done`
- session driver / manager-worker 编排
- 各种多 agent 自动工厂

它们很值得学，但更适合当“先进流派”和“方法论样本”，不是一上来就照抄。

一句话记忆：

**先学官方规则，再吸收高手共识，最后按需借鉴实验流派。**

## 二、高手默认知道的 18 条常识
下面这些东西，很多高手觉得是常识，但新用户往往并没有清楚建立起来。

## 1. `CLAUDE.md` 不是配置文件，是持久说明
官方 `memory` 文档说得很明确：
- `CLAUDE.md` 是 context
- 不是 enforced configuration

这意味着：
- 写得再认真，也不是 100% 强制
- 真正要强制的事情，应该交给 settings、permissions、hooks

很多人第一步就错在这里。

## 2. auto memory 和 `CLAUDE.md` 不是一回事
`CLAUDE.md` 是你主动写给 Claude 的。  
auto memory 是 Claude 根据你多次纠偏、显式“remember that …”之类内容写下来的。

官方文档明确写了：
- auto memory 每次会话启动时只加载 `MEMORY.md` 的前 200 行或 25KB
- `/memory` 可以看它到底记住了什么
- 你可以直接对 Claude 说 “remember that …”

所以：

**Claude 不是“自动永久记住一切”，它只是会在特定机制下写一份受大小限制的工作笔记。**

## 3. auto memory 是 per-worktree 的，不是全项目大脑
这点很多人不知道。

官方 memory 文档表格直接写了：
- auto memory scope 是 **per working tree**

这意味着：
- 你在 worktree A 里学到的东西，不会天然等于 worktree B 全知道
- 如果某条知识是所有 session 都该知道的，还是应该进入项目 `CLAUDE.md` 或 rules

## 4. `CLAUDE.md` 最好短于 200 行，不是审美问题，是效果问题
官方文档建议：
- target under 200 lines per `CLAUDE.md`

原因是：
- 它启动就进 context
- 太长会吃 token
- 太长会降低遵循度

高手不是不知道可以写很长，而是知道写长往往会害自己。

## 5. imported 文件不省 token，只是更好组织
官方写得很清楚：
- `@path` import 只改善组织
- imported files 依然会在启动时载入

所以很多人以为“我拆成很多文件就不占上下文了”，这是错的。

## 6. rules 的价值不是分类好看，而是 path-scoped lazy loading
如果一条规则只对 `frontend/` 生效，就不要让它每次都全局加载。

官方 memory 文档建议：
- instructions growing large 时用 path-scoped rules

这是高手处理复杂项目的关键动作之一。

## 7. skill 的本体不是“扩展功能”，而是“扩展流程”
官方 `skills` 文档说得很清楚：
- when you keep pasting the same playbook, checklist, or multi-step procedure into chat, create a skill

所以 skill 的真正定位是：
- 按需加载的 playbook
- checklist
- workflow

不是：
- 把所有项目事实塞进去
- 把所有规则再复制一遍

## 8. built-in commands 和 bundled skills 不是一回事
官方 `skills` 文档明确区分了：
- built-in commands：固定逻辑
- bundled skills：prompt-based playbook

内置 bundled skills 包括：
- `/simplify`
- `/batch`
- `/debug`
- `/loop`
- `/claude-api`

这点很重要，因为它说明：

**Claude Code 官方自己也在把“高手工作流”逐渐 skill 化。**

## 9. custom commands 已经并入 skills 体系
官方 `skills` 文档直接写了：
- `.claude/commands/deploy.md` 和 `.claude/skills/deploy/SKILL.md` 都会创建 `/deploy`
- 现有 custom commands 继续兼容，但本质上已经统一到 skills 思路

这意味着你现在学习“命令”时，应该把它们理解成“可被 slash 调起的 skill”。

## 10. 技巧的关键不是更多工具，而是更少的歧义
Anthropic 的 context engineering 博客明确点名：

**bloated tool sets** 是常见失败模式。  
如果人类都说不清什么时候该用哪个工具，agent 更不可能选对。

这条我希望你牢牢记住：

**高手不是接更多工具，而是尽量保持一个最小可判定的工具面。**

## 11. 给验证方式，比写一段花 prompt 更重要
官方 best practices 的原话非常重：

**This is the single highest-leverage thing you can do.**

也就是：
- 给测试
- 给截图
- 给预期输出
- 给 build / lint / typecheck 结果

如果没有验证，Claude 只能生成“看起来像对”的东西。

## 12. 长任务里，reset 往往比 compaction 更稳
Anthropic 的 long-running harness 博客把这件事讲透了：
- compaction 保连续性
- reset 提供 clean slate

他们甚至明确写了：
- 某些模型上 compaction alone 不够
- context reset + structured handoff became essential

这就是为什么高手会做：
- `/compact`
- `/clear`
- 写交接文件后重开

而不是一条会话拖到底。

## 13. 评审和实现最好分开
Anthropic long-running harness 的另一个核心结论是：

**Separating the agent doing the work from the agent judging it proves to be a strong lever.**

翻译成人话就是：
- 写的人评自己，很容易放水
- 独立 reviewer / evaluator 更可靠

所以高手很常见的做法是：
- 一个 session 负责实现
- 一个 session 或第二轮 prompt 专门 review

## 14. 同一个 session 在多个终端恢复，会把历史写乱
官方 `how Claude Code works` 文档明确写了：
- same session in multiple terminals 会 interleave messages
- for parallel work from same starting point, use `--fork-session`

这属于很实战的“隐性常识”：

**并行不是 resume 同一个 session，而是 fork 或 worktree。**

## 15. 高手大量使用 worktree，不是为了炫，而是为了隔离上下文和 diff
官方 docs 说得很清楚：
- sessions are tied to directories
- run parallel Claude sessions by using git worktrees

而 GitHub issues 和 HN 上大量真实工作流都在说明：
- 4-6 个并行 session 已经很常见
- 多 session without worktree 很容易互相踩

worktree 在高手圈已经不是小众技巧，而是并行开发的默认解法。

## 16. `.claude/` 目录在 worktree 里是需要额外留意的
真实 issue 里有人专门报 bug：
- `claude --worktree` 生成的新 worktree 没把 `.claude/skills`、`rules`、`agents` 带过去

这说明一个高手常识：

**别想当然以为 worktree 天然继承你的整套 agent 配置。你要检查。**

## 17. 高手会用 `/insights`、`/memory`、`/context` 这类“自诊断命令”
官方 `commands` 文档现在已经给出：
- `/insights`：分析 sessions、interaction patterns、friction points
- `/fewer-permission-prompts`：扫描 transcripts 生成优先 allowlist
- `/memory`：直接检查加载了哪些记忆

高手并不是只会 `/clear`、`/compact`。  
他们会反过来观察自己是怎么用 Claude Code 的。

## 18. GitHub 自动化很强，但高手会先把本地工作流打顺
Simon Willison 和官方 GitHub Actions 文档都证明：
- Claude Code 很适合自动化 PR / CI / issue workflows

但真正的高手通常不是一上来就把一切 remote 化。  
他们一般先在本地把：
- prompt 模板
- 验证闭环
- review 流程
- worktree 隔离
- permissions

这些跑顺，再把成熟流程搬进 GitHub Actions。

## 三、高手真实在用什么
这部分我不想再给你“什么都推荐一点”的答案。真实情况更像下面这样。

## 第一层：几乎所有高手都在做的
- 写全局 `~/.claude/CLAUDE.md`
- 写项目 `CLAUDE.md`
- 用 `/memory` 管 auto memory
- 把长流程做成 skill
- 用 hooks 处理“必须发生”的动作
- 给验证闭环
- 学会 `/compact`、`/clear`、`--continue`、`--resume`、`--fork-session`
- 用 worktree 跑并行任务

这层是基本盘。

## 第二层：重度用户常用，但不是人人必装
- `superpowers`
- `get-shit-done`
- worktree orchestration 工具
- session driver / manager-worker 编排
- skill marketplace / curated skills

这层是“方法论增强层”。

## 第三层：少数人玩得很深，但你现在不该先学
- 多 agent 工厂
- Telegram / Slack / phone 管理 worker plane
- 完整 spec-driven / multi-phase orchestration
- 复杂的自动 QA、contract review、artifact timeline

这层不是没价值，而是**投入产出比**对你现在不一定最高。

## 四、我认真看完资料后，对 `superpowers` 和 GSD 的判断
这是你最该听“人话结论”的地方。

## 1. `obra/superpowers`
这是当前 Claude Code 生态里最值得认真看的高影响力框架之一。

它代表的不是“多了几个 skill”，而是一整套纪律：
- brainstorming before coding
- spec / plan before implementation
- red/green TDD
- subagent-driven development
- review before completion

它为什么火？
- 因为它解决了一个高手和普通用户都痛的真实问题：
  Claude 会跳过设计、跳过测试、跳过 review，直接开始写码。

Jesse Vincent 自己在文章里写得非常直接：
- 以前 skill 描述得对，但不够 prescriptive
- hook 没把规则真正塞进上下文时，系统看上去健康，其实根本没生效
- 后来修成同步注入后，brainstorming 和 plan 才真正稳定触发

我对它的结论是：

**如果你想学高手“纪律化开发”的方法，它非常值得研究；但你不应该一上来就全盘照搬它的复杂度。**

更好的做法是先学它的思想：
- 先澄清需求
- 先 plan
- 先测试
- 先 review

## 2. `get-shit-done`
GSD 的重点不是 TDD 纪律，而是：
- meta-prompting
- context engineering
- spec-driven development
- context rot 缓解

它解决的是另一个痛点：
- 任务做久了，Claude 开始漂
- session 变脏
- 上下文开始 rot

我对它的结论是：

**如果你经常做多天、多 session、长链路任务，GSD 这种思路非常值得借鉴；但它不是基础用法的起点。**

## 3. 你现在最适合的策略
不是二选一。

而是：
- 学 `superpowers` 的纪律感
- 学 GSD 的 context / spec 感
- 但先自己搭一个最小系统，不急着装满

## 五、如果我是你，我会这样搭一套“单兵高手版” Claude Code
这是这篇最实用的一部分。

## 1. 先搭一套最小结构
```text
~/.claude/
├── CLAUDE.md
└── skills/
    ├── catchup/
    ├── bugfix/
    └── review/

repo/
├── CLAUDE.md
└── .claude/
    ├── rules/
    ├── settings.json
    └── skills/
```

## 2. 你的全局 `~/.claude/CLAUDE.md` 只写这些
- 先探索再修改
- 默认给验证方式
- 优先最小改动
- 遇到长任务主动提出 compact / reset
- 改动后汇报：做了什么、怎么验证、剩余风险
- review 默认按风险而不是按摘要来做

控制在 80-120 行。

## 3. 项目 `CLAUDE.md` 只写这些
- build / test / lint / typecheck 命令
- 目录结构
- 代码规范
- 风险目录
- 提交前最低验证要求
- 关键业务约束

控制在 100-150 行。

## 4. 先只做 3 个 skill
### `catchup`
作用：
- 读当前分支 diff
- 读相关文件
- 总结现状
- 给下一步建议

### `bugfix`
作用：
- 复现
- 定位根因
- 写失败用例
- 修复
- 回归验证

### `review`
作用：
- 按风险、回归、边界条件、测试缺口审查改动

为什么先做这三个？
- 它们是最稳定、最频繁、最能提升质量的套路

## 5. 先只做 2 个 hook
### `Notification`
用来：
- 权限请求提醒
- 长任务完成提醒

### 提交边界 hook
用来：
- 没跑测试不允许提交
- 敏感目录改动时额外提醒

为什么不建议你一开始搞很多 block-at-write？
- 这是高手共识，不是官方硬规则
- 中途频繁阻断经常会打乱 agent 节奏

## 六、一个真实项目，高手会怎么一路用 Claude Code
这部分我不写课本版，我直接按“工作现场”写。

## 1. 新项目第一天
高手会做：

1. 跑 `/init`
2. 看 `/memory`
3. 写项目 `CLAUDE.md`
4. 让 Claude 建代码库地图
5. 补最必要的 rules

第一轮提示通常像这样：

```text
先不要改代码。
先给我这个仓库的整体地图：目录结构、入口、主流程、构建方式、测试方式、风险目录。
然后告诉我如果要改 xxx，应该先看哪几处文件。
```

## 2. 接到一个功能需求
高手不会说“帮我做个功能”。

更像这样：

```text
目标：新增 xxx 能力
范围：重点看 @src/auth 和 @src/session.ts
约束：沿用现有 session 流程，不引入新库，不改动 public API
验证：写失败用例复现目标场景，改完跑相关测试
```

如果功能复杂：

```text
先不要改代码。
先分析现状并给出计划、风险、验证方式。
```

## 3. 开发进行中
高手会不断切换模式：
- implementer
- debugger
- reviewer

他们不会一直用一个模糊人格。

常见提示像这样：

```text
先读相关文件，确认当前实现方式。
按最小改动实现，不要重写。
改完跑相关测试，并总结改动、验证结果和剩余风险。
```

## 4. 一旦 session 开始变脏
高手会选三种处理之一：

### 继续但减重
```text
/compact 只保留目标、已完成内容、未完成项、验证结果和已排除路径
```

### 重开
```text
/clear
```

### 写交接后重开
```text
把当前任务目标、已完成、未完成、失败尝试、下一步建议写成 markdown 交接文档
```

然后：

```text
读这个交接文件，继续做，不要重复失败路径
```

## 5. 做完实现以后
高手会切换成 review：

```text
现在不要继续实现。
按 reviewer 视角，只看行为回归、错误处理、测试缺口、可维护性和过度改动。
```

如果改动大，甚至开新 session 专做 review。

## 6. 提交前
高手不是只要 commit message。

他们通常先要：
- 改动总结
- 验证结果
- 已知风险
- 回滚点

然后才要：
- commit message
- PR 描述

## 七、高手常见反模式
这部分比“技巧”更重要。

## 1. 把所有项目规则都塞进 skill
这是你之前自己就踩中的坑。

为什么不对？
- 项目事实应该常驻
- 流程才该 skill 化

## 2. 把所有纠偏都留在聊天里
如果一个纠偏值得说第二次，它就该考虑进入：
- `CLAUDE.md`
- rule
- auto memory
- skill
- hook

## 3. 迷信长 session
很多人把“不重启”误当成“高手”。

实际上高手更像：
- 主动 compact
- 主动 reset
- 主动 handoff

## 4. 一上来装很多 skill / MCP / subagent
高手不是不会装，而是知道：
- 能力面越宽，歧义越多
- 维护成本越高
- 误用风险越高

## 5. 没有独立 review
这会直接把质量拉低。

## 6. 只让 Claude 干活，不让它总结和沉淀
高手会让 Claude 做：
- handoff
- postmortem
- insight extraction
- memory draft
- skill draft

## 7. 不看成本和权限
这在小公司尤其危险。

高手会看：
- `/cost`
- `/fewer-permission-prompts`
- allowlist
- sandbox / auto mode 边界

## 八、你现在最该补的，不是“更多生态”，而是这 4 个能力
如果我只能挑四个真正决定上限的能力，我会选这四个：

## 1. Context engineering
知道什么该进入上下文，什么不该。

## 2. Workflow engineering
知道什么该进 `CLAUDE.md`，什么该 skill 化，什么该 hook 化。

## 3. Session engineering
知道什么时候 compact、什么时候 clear、什么时候 fork、什么时候 handoff。

## 4. Verification engineering
知道怎么让 Claude 自证，而不是靠你人工兜底。

这四个能力一旦建起来，你就已经不是“会用 Claude Code 的人”，而是在进入高手区间。

## 九、我给你的最终建议
如果你是我的朋友，我不会建议你现在去追：
- 100 个 skill
- 一堆 marketplace
- 多 agent 工厂
- 花哨 orchestrator

我会建议你按这个顺序走：

1. 把全局 `CLAUDE.md` 和项目 `CLAUDE.md` 写对
2. 建 3 个 skill：`catchup`、`bugfix`、`review`
3. 建 2 个 hook：提醒、提交边界校验
4. 学会 `/memory`、`/context`、`/insights`、`/fewer-permission-prompts`
5. 开始用 worktree 跑并行
6. 练习 handoff + reset
7. 再去借鉴 `superpowers` 和 GSD

这条路不花哨，但它最像高手真正走出来的路。

## 十、如果你只记住这一句
**高手不是更会问 AI，而是更会设计一个让 agent 稳定工作的环境。**

这个环境包括：
- 短而硬的长期规则
- 干净的上下文
- 按需加载的 skill
- 强制边界的 hook 和 permissions
- 能 reset 的 session 工作流
- 能复盘、能沉淀、能复用的习惯

你现在最需要的，不是继续看碎片技巧。  
你需要把这套环境搭起来。

## 关键英文来源（截至 2026-04-25）
### 官方文档
- Best Practices for Claude Code  
  https://code.claude.com/docs/en/best-practices
- How Claude remembers your project  
  https://code.claude.com/docs/en/memory
- Extend Claude with skills  
  https://code.claude.com/docs/en/skills
- Commands  
  https://code.claude.com/docs/en/commands
- How Claude Code works  
  https://code.claude.com/docs/en/how-claude-code-works
- Common workflows  
  https://code.claude.com/docs/en/common-workflows
- CLI reference  
  https://code.claude.com/docs/en/cli-usage

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

### 顶级实践者与高影响力开源工作流
- Shrivu Shankar, *How I Use Every Claude Code Feature*  
  https://blog.sshh.io/p/how-i-use-every-claude-code-feature
- Simon Willison, *How I Use Every Claude Code Feature*  
  https://simonwillison.net/2025/Nov/2/how-i-use-every-claude-code-feature/
- Simon Willison, *Claude Skills are awesome, maybe a bigger deal than MCP*  
  https://simonwillison.net/2025/Oct/16/claude-skills/
- Simon Willison, *Using Claude Code to build a GitHub Actions workflow*  
  https://simonwillison.net/2025/Jul/1/claude-code-github-actions/
- Jesse Vincent / obra, *superpowers*  
  https://github.com/obra/superpowers
- Jesse Vincent, *Superpowers v4.3.0*  
  https://blog.fsck.com/releases/2026/02/12/superpowers-v4-3-0/
- Jesse Vincent, *How Claude Code Session Continuation Works*  
  https://blog.fsck.com/releases/2026/02/22/claude-code-session-continuation/
- GSD / Get Shit Done  
  https://github.com/gsd-build/get-shit-done

### 真实用户工作流与痛点证据
- Inter-session communication for multi-Claude workflows  
  https://github.com/anthropics/claude-code/issues/24798
- Better Multi-project workflow  
  https://github.com/anthropics/claude-code/issues/4707
- `claude --worktree` missing `.claude/` subdirectories  
  https://github.com/anthropics/claude-code/issues/28041
- HN: worktrees and orchestrators around Claude Code  
  https://news.ycombinator.com/item?id=47257712
  https://news.ycombinator.com/item?id=47256056
  https://news.ycombinator.com/item?id=46765489

## 我为什么采用这些来源
- 官方文档：确认能力边界、命令、memory、skills、sessions、worktree、permissions 的正式行为
- 工程博客：确认 Anthropic 团队自己验证过的方法，例如 context reset、独立 evaluator、最小工具集
- Shrivu / Simon：补上高手真实使用姿势，尤其是 memory、skills、hooks、GitHub workflows 的实践感
- Jesse Vincent / superpowers / GSD：观察高手生态里最有影响力的方法论，不把它们当官方标准，但把它们当“高手为什么会这么用”的证据
- GitHub issues / HN：验证哪些痛点是真实高频，而不是我主观臆想
