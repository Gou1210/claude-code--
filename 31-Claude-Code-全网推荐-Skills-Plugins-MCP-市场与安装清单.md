# 31-Claude-Code-全网推荐 Skills、Plugins、MCP、市场与安装清单

> 更新时间：2026-04-25  
> 这一章只回答四个问题：装什么、什么时候装、怎么装、第一句怎么用。  
> 资料来源以 Claude Code 官方文档、官方插件页，以及少量社区仓库为主。

## 一、先给结论：大多数人真正值得装的，不超过 10 个

如果你主要是真实开发，不是玩生态，优先级直接按这一层装：

1. 官方 marketplace
2. `Context7`
3. `GitHub`
4. `Playwright`
5. `Code Review`
6. `Commit Commands`
7. `Sentry`
8. `Vercel`
9. `Linear`
10. `Superpowers`

如果你是前端偏重，再加：

1. `Frontend Design`

如果你是安全、审计、逆向、区块链偏重，再加：

1. `trailofbits/skills`

如果你是企业 PHP、Go、Docker、Jira、TYPO3 这类场景偏重，再看：

1. `netresearch/claude-code-marketplace`

## 二、先把官方插件体系装对

Claude Code 官方文档现在已经把插件体系讲得很清楚了：插件可以带 `skills`、`agents`、`hooks`、`MCP servers`，官方 marketplace 默认可用。  
先确认你会这几条命令：

```bash
/plugin
/plugin install github@claude-plugins-official
/plugin install context7@claude-plugins-official
/plugin install playwright@claude-plugins-official
/reload-plugins
```

装完以后，实际可用命令名以 `/help` 和 `/plugin` 里显示的结果为准，不要靠文章记忆硬敲。

如果官方市场索引不新，先更新：

```bash
/plugin marketplace update claude-plugins-official
```

官方文档：

- 插件发现与安装：<https://code.claude.com/docs/en/discover-plugins>
- 插件结构与能力边界：<https://code.claude.com/docs/en/plugins>
- 技术参考：<https://code.claude.com/docs/en/plugins-reference>

## 三、先别急着装第三方，内建 skills 先吃透

官方 skills 文档已经把 Claude Code 自带的几类 bundled skills 放出来了。真正常用的是这 5 个：

| 内建 skill | 什么时候用 | 第一种用法 |
| --- | --- | --- |
| `/debug` | 卡在 bug、复现不稳定、原因不清 | 直接让它按调试流程追根因 |
| `/batch` | 一次性处理多个独立改动 | 批量跑多个小修复或检查 |
| `/simplify` | 代码已经能跑，但太绕 | 让它在不改行为前提下收敛复杂度 |
| `/loop` | 需要重复验证直到满足条件 | 跑修复-验证循环 |
| `/claude-api` | 你在写 Anthropic/Claude 相关接入 | 查 Claude API 用法和模式 |

直接起手：

```text
/debug 这个接口偶发 500，先不要修，先给我最可能的 3 个根因和最小复现路径
```

```text
/simplify 只整理最近改过的文件，不改变行为，不扩大改动面
```

```text
/batch 把这 6 个 warning 按最小风险逐个修掉，每修完一个就自检
```

官方文档：

- Skills：<https://code.claude.com/docs/en/skills>
- Commands：<https://code.claude.com/docs/en/commands>

## 四、最值得先装的 8 个官方插件

这一节不讲“都很强”，只讲真实开发里最值回票价的。

| 插件 | 解决什么问题 | 怎么装 | 第一句怎么用 |
| --- | --- | --- | --- |
| `Context7` | 当前文档和版本 API 容易幻觉 | `/plugin install context7@claude-plugins-official` | `给我写 Next.js 认证中间件，use context7` |
| `GitHub` | issue、PR、actions、repo 查询都要来回切窗口 | `/plugin install github@claude-plugins-official` | `看这个仓库最近失败的 GitHub Actions` |
| `Playwright` | 浏览器自动化、E2E、截图、表单流验证 | `/plugin install playwright@claude-plugins-official` | `打开 localhost:3000，跑一遍登录流程并截图` |
| `Code Review` | 需要并行 reviewer 对 PR 做结构化审查 | `/plugin install code-review@claude-plugins-official` | `/code-review` |
| `Commit Commands` | 提交、推送、PR 描述老是写得粗糙 | `/plugin install commit-commands@claude-plugins-official` | `基于当前改动生成 commit 和 PR 描述` |
| `Sentry` | 线上报错、堆栈、影响面分析 | `/plugin install sentry@claude-plugins-official` | `/seer 最近 24 小时最严重的 5 个错误` |
| `Vercel` | 部署、日志、域名和线上构建排障 | `/plugin install vercel@claude-plugins-official` | `/vercel-logs 看最近一次部署为什么失败` |
| `Linear` | issue 跟踪、状态同步、开发和任务系统对齐 | `/plugin install linear@claude-plugins-official` | `把这个 bug 建成 Linear issue，并附修复建议` |

## 五、如果你只能先装 3 个，优先这 3 个

### 1. `Context7`

这是最稳的“降幻觉”插件之一。它的价值不是“会查文档”，而是把版本化文档直接拉进当前工作流。

你应该在这些场景默认带上它：

- 新框架或冷门库
- 升级大版本
- SDK 经常变动
- 你不接受“我记得应该是这样”

起手句：

```text
给我按 Next.js 15 当前文档写 middleware 鉴权，use context7
```

```text
用 /supabase/supabase 的最新 API 写一个服务端查询例子，use context7
```

官方页：<https://claude.com/plugins/context7>

## 六、`GitHub` 是仓库外部系统接入的第一优先级

只要你工作流里有 GitHub，这个插件几乎必装，因为它直接打通了：

- issue
- PR
- review comments
- Actions
- repo 搜索
- code scanning / Dependabot 等安全信息

能直接用的句子：

```text
看看这个仓库最新失败的 Actions，告诉我最可能的修复入口
```

```text
把这个修复做成 PR，并生成一版简洁的变更说明
```

```text
列出分配给我的 open issues，并按紧急程度排序
```

官方页：<https://claude.com/plugins/github>

## 七、`Playwright` 是本地 UI 验证和 E2E 的高杠杆插件

它适合的不是“我要浏览网页”，而是下面这些高价值动作：

- 让 Claude 自己验证一个前端流程到底通没通
- 重现表单、登录、checkout 这类长链路问题
- 跑截图回归
- 在修复后做最小闭环验收

直接用：

```text
打开 localhost:3000，登录 test 账号，进入订单页，下一个测试单，最后给我一张成功页截图
```

```text
验证这个修复有没有解决移动端菜单无法展开的问题，给我操作步骤和结果
```

如果网站有反爬、验证码、地理限制，再考虑 `Stagehand`，但默认先装 `Playwright`，因为它更偏工程测试，而不是偏浏览器代操作。

官方页：

- Playwright：<https://claude.com/plugins/playwright>
- Stagehand：<https://claude.com/plugins/stagehand>

## 八、`Code Review` 适合把“让 Claude 帮我看一眼”升级成正式审查

这个插件值得装，不是因为它会提意见，而是因为它把 review 变成了多 reviewer 并行、带置信度过滤的流程。

适合的场景：

- PR 比较大
- 你怕回归
- 你想把 review 和 implement 分离
- 你准备在团队里半自动化代码审查

最直接的用法：

```text
/code-review
```

更合理的配套用法：

1. 先让 Claude 改代码
2. 再开新会话或新 worktree 跑 review
3. 最后只处理高置信度问题

官方页：<https://claude.com/plugins/code-review>

## 九、`Commit Commands` 不是偷懒插件，是交付插件

很多人把它理解成“帮我写 commit message”，这太低估了。  
它更适合用在这 3 类动作：

1. 一次改动做完后，统一生成 commit
2. 顺手生成 PR 描述和 test plan
3. 清理本地已失效分支

第一句：

```text
基于当前改动生成一个符合仓库风格的 commit，并给出 PR 描述草稿
```

如果你已经安装了 GitHub CLI 并完成认证，直接把它接到交付尾部最省脑力。

官方页：<https://claude.com/plugins/commit-commands>

## 十、`Sentry` 是线上排障插件，不是报错查询插件

当你面对的是生产问题，它的价值在于：

- 先抓最严重错误
- 看堆栈和影响用户范围
- 让 Claude 帮你归因
- 再回到本地代码修

第一轮直接这样开：

```text
/seer 最近 24 小时最严重的错误是什么，按用户影响和修复优先级排序
```

```text
看这个 Sentry issue，告诉我最可能的代码入口、复现路径和修复建议
```

官方页：<https://claude.com/plugins/sentry>

## 十一、`Vercel` 适合把“看部署日志”这件事交给 Claude

如果你前端或全栈项目走 Vercel，这个插件比手动翻 dashboard 快很多。

适合的动作：

- 看最新部署失败原因
- 跟进 build log
- 检查某次上线是否成功
- 直接做部署动作

第一句：

```text
/vercel-logs 看最近一次部署失败在哪里，只保留最关键的 3 个报错
```

```text
帮我部署当前项目到 Vercel，并确认环境变量是否缺失
```

官方页：<https://claude.com/plugins/vercel>

## 十二、`Linear` 适合把“做代码”和“对任务系统交账”接起来

你如果经常出现“代码改了，但 issue 没更新”，那就值得装。

可直接做的事：

- 建 bug / feature issue
- 查分配任务
- 更新状态
- 把代码调查结果回写到任务系统

第一句：

```text
根据我刚修的这个 bug，创建一个 Linear issue，总结复现、根因、修复方案和验证结果
```

官方页：<https://claude.com/plugins/linear>

## 十三、`Frontend Design` 只推荐给真的有 UI 输出压力的人

如果你主要是服务端、脚本、基础设施，这个插件优先级不高。  
但如果你经常让 Claude 直接出页面，它很值，因为它的目标不是“生成能看”，而是“生成不那么 AI 味的页面”。

适合：

- landing page
- dashboard
- marketing site
- 需要更明确审美方向的后台界面

第一句：

```text
做一个金融数据分析后台，不要通用 AI 审美，做出明确视觉方向
```

官方页：<https://claude.com/plugins/frontend-design>

## 十四、`Superpowers` 不是功能插件，而是方法论插件

如果你已经用了一个月 Claude Code，这个插件反而更值得看。  
它的核心不是接外部系统，而是给 Claude 一套更强的工作套路，比如：

- brainstorming
- writing plans
- executing plans
- systematic debugging
- test-driven development
- subagent-driven development
- using git worktrees
- verification before completion

这个插件更适合这些人：

- 你觉得 Claude 会做，但不稳定
- 你想把工作流“模板化”
- 你需要一个比较成体系的 skill 集合

第一句：

```text
/brainstorm 我准备重构认证模块，先把方案空间、约束、风险列出来
```

```text
/write-plan 基于当前仓库，写一个最小可执行的迁移计划
```

官方页：<https://claude.com/plugins/superpowers>

社区市场里也普遍把它当作第一批应装插件之一。

## 十五、如果你做安全、审计、逆向，直接看 `trailofbits/skills`

这是我这次搜下来最值得单列的一类社区市场，因为它不是泛泛 skill 仓库，而是明确面向：

- 安全审计
- 漏洞分析
- 智能合约安全
- 变异测试
- reverse engineering
- GitHub Actions 安全审计

直接安装：

```bash
/plugin marketplace add trailofbits/skills
/plugin
```

它里边值得重点看的不是“装满”，而是按场景选，比如：

- `agentic-actions-auditor`
- `audit-context-building`
- `variant-analysis`
- `mutation-testing`
- `property-based-testing`
- `second-opinion`

如果你比较在意第三方生态的安全性，再看它们的 curated 市场：

```bash
/plugin marketplace add trailofbits/skills-curated
```

来源：

- 市场仓库：<https://github.com/trailofbits/skills>
- curated 市场：<https://github.com/trailofbits/skills-curated>
- Trail of Bits 官方配置说明：<https://github.com/trailofbits/claude-code-config>

## 十六、如果你做企业项目、语言栈明确，`netresearch/claude-code-marketplace` 值得看

这个市场不适合“我就想先装最热门的”。  
它适合下面这种情况：

- 你是企业团队
- 你有比较稳定的技术栈
- 你想装的是“行业化 skill”，不是通用 skill

它收的东西比较偏：

- TYPO3
- PHP modernization
- security audit
- enterprise readiness
- Docker
- Concourse CI
- Go development
- Jira integration
- GitHub release
- data tools

直接安装：

```bash
/plugin marketplace add netresearch/claude-code-marketplace
/plugin
```

来源：<https://github.com/netresearch/claude-code-marketplace>

## 十七、如果你想看“大而全”的高手配置，研究 `Everything Claude Code`，但不要整包照抄

这个项目这段时间讨论度很高，它真正有价值的点是：

- 把 `agents`、`skills`、`hooks`、`rules`、`MCP`、memory persistence 放到一个体系里
- 明确讲了 token optimization、parallelization、verification loops、subagent orchestration
- 有不少“系统化而不是单点 prompt”的实现思路

但它不适合作为“先装再说”的第一选择。更适合这两种用途：

1. 研究别人怎么把 Claude Code 做成完整执行系统
2. 挑其中一两个模块学，不要整仓库全抄

我更建议你只学习这些模块思路：

- hooks 怎样保存/恢复 session 状态
- 什么时候用 worktree 并行
- 怎么把 continuous learning 抽成 skill
- 怎样做 verification loops

来源：<https://github.com/worldflowai/everything-claude-code>

## 十八、社区市场可以逛，但安装时只看 4 个问题

看到一个新 plugin，不要先问“火不火”，先问：

1. 它有没有解决一个你每天都在重复的动作
2. 它是接系统，还是只是换一种 prompt 包装
3. 它有没有脚本、hooks、MCP，源码能不能看
4. 它会不会把你的权限面和攻击面扩大太多

如果只是“写计划更优雅”“思考更系统”，但你自己已经有稳定 workflow，那种插件优先级就很低。

## 十九、最值得你自己补的，不是再装一个市场，而是 4 个个人 skill

官方 skills 文档已经足够成熟了。  
如果你已经不是新手，我更建议你自己补这 4 类 skill，而不是继续囤插件：

### 1. `repo-map`

用途：进新仓库先拉仓库地图、主链路、测试入口、风险点。  
建议用 `context: fork`，让它跑在 `Explore` agent 里。

### 2. `strict-review`

用途：固定输出 review 结构，只报高价值问题。  
建议输出固定字段：`Problem`、`Trigger`、`Impact`、`Evidence`、`Suggested check`。

### 3. `handoff-artifact`

用途：每个任务结束时沉淀固定 artifact。  
建议输出：`Problem`、`Files touched`、`Checks passed`、`Open risks`、`Next action`。

### 4. `release-check`

用途：发布前固定做一遍配置、迁移、环境变量、回滚点检查。

一个最小 skill 壳：

```markdown
---
name: repo-map
description: Build a repo map before implementation. Use for unfamiliar repositories or multi-module tasks.
context: fork
agent: Explore
---

Map this repository before implementation:

1. Identify entrypoints
2. Identify core modules
3. Identify test commands and existing test locations
4. Identify likely files to modify for: $ARGUMENTS
5. Return only concrete file paths and short explanations
```

官方 skills 文档：<https://code.claude.com/docs/en/skills>

## 二十、插件、skills、MCP，到底优先装谁

直接按这个判断：

| 你的问题 | 该用什么 |
| --- | --- |
| 我总在重复一段流程 | 写 `skill` |
| 我需要接 GitHub / Sentry / Vercel / 浏览器 / 文档系统 | 装 `plugin` 或 `MCP` |
| 我要团队共享这一套能力 | 做 `plugin` 或 project-scope skill |
| 我只是在当前仓库临时试一套玩法 | 先写 `.claude/skills/` |
| 我想把一堆 skill、agents、hooks 打包传播 | 做 `plugin marketplace` |

## 二十一、我建议的 4 套最小安装方案

### 1. 通用全栈开发者

```text
Context7 + GitHub + Playwright + Code Review + Commit Commands
```

### 2. 前端 / SaaS

```text
Context7 + GitHub + Playwright + Vercel + Frontend Design
```

### 3. 生产排障 / 全栈维护

```text
GitHub + Sentry + Playwright + Context7
```

### 4. 安全 / 审计 / 高风险代码

```text
GitHub + Code Review + trailofbits/skills + second-opinion
```

## 二十二、安装顺序也有讲究，不要一口气装十几个

我的建议顺序：

1. 先装官方市场里的系统接入插件
2. 再装一个方法论 skill 包，比如 `Superpowers`
3. 再按行业或团队需求加社区市场
4. 最后把真正常用的流程沉淀成你自己的 skill

不要反过来。  
先装一堆社区包，最后你只会得到一个噪音很大的 `/` 菜单。

## 二十三、装完以后第一时间做 3 个动作

### 1. 运行 `/help`

看新暴露出来的命令和 skill 名字，别靠记忆猜。

### 2. 运行 `/plugin`

确认 scope、启用状态、描述、来源。

### 3. 开一轮真实任务验证

不要做 demo，直接用真实任务测：

- `Context7` 查一次正在用的库
- `Playwright` 跑一次本地页面
- `GitHub` 查一次失败 workflow
- `Sentry` 查一次线上错误

## 二十四、哪些东西我不建议你盲装

### 1. 只会“包装提示词”的大 skill 包

如果没有明确脚本、hook、MCP、agents，只是大量 workflow 文案，价值通常没你想的那么高。

### 2. 带重权限 hooks 的未知插件

尤其是会自动执行 shell、改 git、写文件、碰密钥的。

### 3. 你当前根本没有外部系统要接的插件

比如你不用 Linear、不用 Vercel，却先装它们，只会增加菜单噪音。

### 4. 一整套高手仓库原样照搬

这类仓库适合拆着学，不适合整包抄。

## 二十五、这一章最后给你的直接执行版清单

如果你现在就要开始，直接按这个顺序：

```bash
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
/plugin install playwright@claude-plugins-official
/plugin install code-review@claude-plugins-official
/plugin install commit-commands@claude-plugins-official
/reload-plugins
```

然后分别跑这 5 句：

```text
给我按最新文档写一个 Next.js 认证中间件，use context7
```

```text
看看这个仓库最近失败的 GitHub Actions，并指出最可能的修复入口
```

```text
打开 localhost:3000，跑一遍登录流程并截图
```

```text
/code-review
```

```text
基于当前改动生成 commit 和 PR 描述草稿
```

做到这一步，你的 Claude Code 已经不是“会聊天”，而是开始接真实工程系统了。

## 参考来源

- Claude Code Skills：<https://code.claude.com/docs/en/skills>
- Claude Code Commands：<https://code.claude.com/docs/en/commands>
- Claude Code Plugins：<https://code.claude.com/docs/en/plugins>
- Claude Code Discover Plugins：<https://code.claude.com/docs/en/discover-plugins>
- Claude Code Plugins Reference：<https://code.claude.com/docs/en/plugins-reference>
- Claude Code MCP：<https://code.claude.com/docs/en/mcp>
- GitHub plugin：<https://claude.com/plugins/github>
- Context7 plugin：<https://claude.com/plugins/context7>
- Playwright plugin：<https://claude.com/plugins/playwright>
- Code Review plugin：<https://claude.com/plugins/code-review>
- Commit Commands plugin：<https://claude.com/plugins/commit-commands>
- Sentry plugin：<https://claude.com/plugins/sentry>
- Vercel plugin：<https://claude.com/plugins/vercel>
- Linear plugin：<https://claude.com/plugins/linear>
- Frontend Design plugin：<https://claude.com/plugins/frontend-design>
- Superpowers plugin：<https://claude.com/plugins/superpowers>
- Trail of Bits skills marketplace：<https://github.com/trailofbits/skills>
- Trail of Bits curated marketplace：<https://github.com/trailofbits/skills-curated>
- Trail of Bits claude-code-config：<https://github.com/trailofbits/claude-code-config>
- Netresearch marketplace：<https://github.com/netresearch/claude-code-marketplace>
- Everything Claude Code：<https://github.com/worldflowai/everything-claude-code>
