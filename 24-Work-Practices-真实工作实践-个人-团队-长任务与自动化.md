# 24. Work Practices：真实工作实践、个人、团队、长任务与自动化

## 重要程度
**S 级** - 极其重要。如果说前面很多章节解决的是“Claude Code 能做什么”，这一章解决的是“你怎样把它真正放进工作系统里，而不是只当一个聊天玩具”。

## 学习目标
- 理解个人开发、团队协作、长任务执行、GitHub 自动化和企业治理的真实落地方法
- 学会把 Claude Code 用成一个可持续、可审计、可恢复、可度量的工作系统
- 形成“任务价值 - 风险 - 并行度 - 可验证性”四维判断，而不是一股脑自动化

## 学什么
- Anthropic 官方文档里的 analytics、monitoring、GitHub Actions、security、managed settings
- Anthropic 工程博客关于 long-running harness、多代理研究系统、auto mode 的设计结论
- 官方产品页与客户案例里已经公开披露的团队实践信号
- 英文开发者圈里被证明有用的组织化使用经验

## 你需要掌握
- 真实工作实践的重点不是“单次 prompt 多漂亮”，而是 **持续运行时的组织方式**
- 长任务最怕的不是模型笨，而是缺少阶段目标、交接物、评审角色和可恢复点
- 团队落地最怕的不是功能不够，而是配置分层混乱、权限过宽、日志不可观察、产出不可度量
- 自动化不是把人踢出去，而是把人从低价值盯梢里移走，只保留高风险节点审批

## 一、先建立一个更接近真实工作的心智模型
Anthropic 官方产品页现在对 Claude Code 的定位已经非常明确：

- 它不是 autocomplete
- 它是一个 **agentic coding system**
- 工程师的角色，越来越像是在做 architecture、product thinking、continuous orchestration

你可以把这句话翻译成最务实的工作理解：

**Claude Code 最适合接手“探索、实现、验证、修正、提交流水线里的执行部分”，而人类更适合定义目标、边界、优先级和验收标准。**

所以真正成熟的工作方式，不是你盯着它每一步，而是你把系统搭好，让它在系统里跑。

## 二、个人开发实践：最稳的不是“边聊边改”，而是固定节奏
个人开发最容易犯的错，是把 Claude Code 当成高级聊天框，想到哪说到哪。

更稳的个人节奏通常是：

1. 先限定范围
2. 再定义验证
3. 再允许探索和实现
4. 最后单独做一次 review / 验收

### 一个成熟的个人循环
#### 1. 起手先给任务壳，而不是直接给答案
- 目标：要做成什么
- 范围：看哪些目录、哪些文件
- 约束：不能引入什么、要沿用什么模式
- 验证：测试、构建、截图、日志、预期输出是什么

#### 2. 小任务直做，大任务先 plan
如果你能一句话说清 diff，可以直接做。

如果你说不清：
- 涉及多文件
- 牵涉架构
- 你自己也不确定方案

那就先 plan。

#### 3. 长任务拆成“可停、可接、可验”的段
这一步是很多人缺的。你不应该让 Claude 在一个无限长的 session 里漫无边界地跑，而应该让任务天然分段：
- 当前阶段目标
- 当前阶段完成定义
- 失败后回滚点
- 下一段接力方式

#### 4. 结束前单独拉一次 review
不要把“实现者的自我感觉良好”当验收。

Anthropic 在 long-running harness 博客里反复强调一个现实：

**让代理自己评自己的活，通常会偏宽松。**

所以哪怕是个人使用，你也应该尽量把“实现”和“验收”分成两轮，必要时甚至分成两个 session。

## 三、长任务实践：把任务做成 sprint，不要做成一口气
这一部分是英文资料里真正含金量极高、但中文资料经常没吸收好的地方。

Anthropic 在 **2026-03-24** 的《Harness design for long-running application development》里，把长任务执行拆成了非常有启发性的结构：

- planner
- generator
- evaluator
- 结构化 handoff
- sprint contract

你不一定要照搬三代理架构，但这套思想非常适合翻译成人类团队工作法。

### 1. 先有 planner，不要让实现代理直接脑补完整需求
Anthropic 的 planner 做的不是“细节实现设计”，而是：
- 把简短需求扩成完整产品 spec
- 保持高层交付目标
- 避免一开始就在细节实现上过度承诺

这给我们的启发是：

**需求越大，越不要让第一轮就直接跳进代码。先把“交付物长什么样”写清楚。**

### 2. 每个 sprint 先谈“done 长什么样”
Anthropic 这篇里最值钱的做法之一，是 generator 和 evaluator 在每个 sprint 前先协商一个 **sprint contract**：

- 这一段到底做什么
- 怎么验证
- 什么叫完成

这件事看起来朴素，但特别重要，因为它避免了两种常见失败：
- 生成代理以为自己做完了，其实只是做了一个外壳
- 评估代理不知道该怎么判定“完成”

你在人类工作里也完全应该这么做。

### 3. 交接不要靠口头，要靠 artifact
Anthropic 明说他们的 agent 间通信是通过文件完成的。

翻译成人类可直接套用的方法就是：

- 长任务必须沉淀交接文件
- 里面至少要有：目标、现状、已完成、失败尝试、待验证点、下一步

否则 session 一断、人一换、时间一长，任务质量就会立刻塌。

### 4. 不要迷信 compaction，必要时直接 reset
Anthropic 在这篇文章里明确区分了：
- compaction 保留连续性
- reset 提供 clean slate

对真实工作来说，你可以这样理解：

#### 优先 compact 的情况
- 任务还没跑偏
- 只是上下文有点长
- 当前代理仍然稳定

#### 优先重开的情况
- 开始出现“提前收尾”
- 已知错误路径在反复循环
- 长时间输出质量下降
- 你已经需要解释“忽略前面那几轮”

## 四、评审实践：实现和 QA 最好分离
Anthropic 在 long-running harness 里给出的一个核心经验是：

**把干活的代理和打分的代理分开，是一个很强的杠杆。**

原因很简单：
- 自评容易放水
- 独立 evaluator 更容易形成可执行反馈

### 个人使用时怎么落地
- 第一轮：让 Claude 实现
- 第二轮：新 prompt / 新 session 专门做 QA 或 review
- 第三轮：再让 Claude 按反馈修正

### 团队使用时怎么落地
- Writer session：实现
- Reviewer session：审查风险、边界、兼容性
- Tester / QA session：跑验证、做截图对比、点流程、查边界

这不是形式主义，而是为了降低“同一代理既写又夸”的天然偏差。

## 五、团队实践：把配置分层理清，比堆规则更重要
Claude Code 真正进入团队环境后，最大的混乱通常不是功能，而是“规则到底放哪”。

官方 `Settings`、`Memory`、`Skills`、`MCP`、`Security` 文档给出的分层，其实已经非常实用。

### 1. `CLAUDE.md`
团队共享的项目事实层：
- 架构规则
- 测试方式
- 团队约定
- 长期有效的风险提醒

### 2. `.claude/settings.json`
团队共享的行为配置层：
- permissions
- hooks
- env
- output style
- 组织认可的默认设置

### 3. `.claude/settings.local.json`
个人实验和本地偏好层：
- 不该变成团队标准的东西
- 个人快捷试验
- 本地机环境差异

### 4. `.mcp.json`
项目级外部工具接入层：
- 团队共享 MCP
- 项目依赖的外部系统入口

### 5. `.claude/skills/`
团队复用流程层：
- review 剧本
- 调试剧本
- 文档生成剧本
- 某类模块的专有流程

一句话记忆：

**事实、配置、外部连接、流程脚本，不要混放。**

## 六、团队实践：新成员 onboarding 要产品化
Claude Code 已经把这件事做成了内置能力。

官方 `Commands` 文档里有 `/team-onboarding`：
- 它会分析过去 30 天的 sessions、commands、MCP 使用情况
- 产出一个新成员可直接粘贴的 onboarding guide

这说明 Anthropic 已经在产品层面承认一个现实：

**团队落地的瓶颈，不只是模型能力，而是“怎么让新成员快速对齐使用方式”。**

所以成熟团队应该至少有三样 onboarding 资产：
- 仓库级 `CLAUDE.md`
- 团队共享 settings / hooks / skills
- 一份“我们怎么用 Claude Code”的实际起手说明

## 七、团队实践：并行不是默认多开，而是控制冲突面
官方产品页和客户案例都明显在强调“parallel sessions”带来的效率提升。

比如截至 **2026-04-25** 官方公开信息里：
- Ramp 把 incident investigation time 降到最多 80% 更快
- Rakuten 已公开提到工程师并行跑多个 Claude Code session

但并行真正落地时，关键不是“多开几个窗口”，而是：

### 1. 文件所有权要清楚
- 谁改哪块
- 谁只读
- 谁负责 review

### 2. 用 worktree 或隔离分支
避免多个 session 直接踩同一工作区。

### 3. 并行只做低耦合任务
Anthropic 自己在 multi-agent 博客里也提醒：
- coding 任务并不总是天然可并行
- 协调成本和 token 成本都会上涨

所以并行适合：
- 一个写实现
- 一个写测试
- 一个做 review
- 一个做外部资料调查

不适合：
- 三个代理同时乱改同一模块

## 八、自动化实践：GitHub Actions 真正值钱，但必须收边界
这一块是英文世界里增长非常快、也最容易踩安全坑的部分。

官方 `Claude Code GitHub Actions` 文档现在已经非常明确：
- 可以通过 `@claude` 在 issue / PR 里触发
- 可以自动实现功能、修 bug、创建 PR
- 支持 `claude_args`
- 支持 AWS Bedrock / Google Vertex AI / GitHub App / OIDC

### 它最适合什么
- issue to PR
- PR 自动修复
- review 辅助
- 夜间批处理
- 接收外部系统触发，例如工单、报警、CI 失败

### 它为什么真正值钱
因为它把 Claude Code 放进了 **可审计的 CI 环境**：
- runner 可控
- 权限可控
- 日志可留
- 触发条件可写进仓库

Shrivu 也特别强调这点：GitHub Action 版本的 Claude Code 很强，不只是因为能自动做事，而是因为它把环境、sandbox、日志和审计能力都带进来了。

### 但这里必须加一个安全现实
只要自动化入口会接触到：
- 公开 issue
- 外部评论
- 第三方日志
- 网页内容

你就必须默认存在 **prompt injection** 风险。

Anthropic 官方 `Security` 文档已经在反复强调这类风险。

而英文开发者圈最近最值得警惕的案例之一，是 Simon Willison 在 **2026-03-06** 讨论的 `Clinejection`：公开 issue 标题本身就可能被构造成攻击链入口。

虽然那个案例不是 Claude Code 自身被打穿，但它说明了一件事：

**“公开输入 + 自动执行 + 宽权限” 是所有 coding agent 体系的通用高危组合。**

### 所以 GitHub 自动化的保守做法应该是
1. 最小权限 GitHub App / token
2. 仓库级限制，而不是组织级大权限
3. 对外部输入做更高审慎
4. 尽量把可写范围限制在当前仓库、当前分支、当前工作流
5. 对高风险操作保留人工审批

## 九、组织治理实践：不要只看“好不好用”，要看“看不看得见”
团队规模一大，只靠感觉管理 Claude Code，一定会失真。

官方现在已经把可观测性和度量补得很完整。

### 1. Analytics dashboard
官方 `analytics` 文档显示，团队和企业计划可以看到：
- lines of code accepted
- suggestion accept rate
- daily active users / sessions
- PRs and lines shipped with Claude Code assistance
- leaderboard
- data export

这意味着团队已经不必只靠口碑判断 adoption。

### 2. OpenTelemetry monitoring
官方 `monitoring` 文档支持：
- metrics
- logs / events
- traces
- OTel 导出

而且官方还直接给了 ROI measurement guide。

这非常关键，因为成熟团队最终会问的不是：
- 某个人觉得它酷不酷

而是：
- 哪些工作流真省时间
- 哪些高频失败该治理
- 哪些团队在用
- 哪些设置会让成本异常

### 3. 日志审阅与失败分析
这点官方和实践者都在强调：
- 观察 token、重试、失败事件、hooks 触发、compaction
- 找共性错误，不要只修单点 bug

你可以把这理解成：

**Claude Code 进入组织以后，已经不是“一个人使用工具”的问题，而是“一个系统如何被运营”的问题。**

## 十、安全与合规实践：边界先于效率
安全这部分，最怕的就是“工具太强，所以大家默认全放开”。

官方截至 **2026-04-25** 的安全资料给出的几个重点，建议你直接当工作守则记住：

### 1. 默认只读是对的
Claude Code 默认严格 read-only，不是烦你，而是在帮你把误伤成本降到最低。

### 2. sandboxing 的价值不是“方便”，而是同时隔离文件系统与网络
官方工程博客已经明确说过：
- 只有文件系统隔离，不够
- 只有网络隔离，也不够

因为 prompt injection 真正危险的，是“读取敏感信息后再往外发”。

### 3. MCP 要当第三方可执行边界看待
官方明确写了：
- Anthropic 不审计 MCP 服务器
- 要么自己写，要么只信任可信供应方

所以团队引入 MCP 时，应该像引第三方基础设施一样对待，而不是把它当普通插件。

### 4. 高敏仓库应使用更严格 repo 级设置
- 更保守 permissions
- 审核 hooks
- 更少自动放行
- 必要时 devcontainer / VM / cloud sandbox

## 十一、什么时候该用单会话、subagent、agent team、GHA
这一节可以当一张工作实践决策表来记。

### 单会话
适合：
- 小到中等任务
- 你需要快速来回讨论
- 主要是局部实现和局部验证

### subagent
适合：
- 高噪音调查
- 独立搜索
- 主 session 需要保干净
- 一次性聚焦工作

### agent team
适合：
- 复杂问题拆成几个独立方向
- 方向之间需要相互沟通
- 值得支付更高协调和 token 成本

注意：
- 截至 **2026-04-25** 仍是 experimental
- 官方也明确写了已知限制

### GitHub Action / 远程自动化
适合：
- 周期性任务
- issue / PR 驱动任务
- 需要统一审计和统一环境的自动执行

一句话判断：

**任务越高价值、越可并行、越可验证、越适合异步执行，就越适合往后几种模式升级。**

## 十二、这一章最值得沉淀下来的工作实践
1. 把 Claude Code 当工作系统，不当聊天面板。
2. 长任务必须有阶段目标、完成定义和交接物。
3. 实现和评审尽量拆开，避免自评放水。
4. 团队规则按层放置：`CLAUDE.md`、settings、skills、MCP、hooks 各管各的。
5. onboarding 要产品化，不要口口相传。
6. 并行要先控冲突面，再谈提速。
7. GitHub 自动化的价值巨大，但必须最小权限、最小暴露面。
8. 度量 adoption、成本、PR 产出和失败模式，不要只靠主观感觉。
9. 安全边界先于效率，特别是面对不可信输入时。
10. 自动化的目标不是“完全没人管”，而是“把人从低价值盯梢中解放出来”。

## 十三、哪些是官方结论，哪些是实践者经验
### 更接近官方结论的部分
- Claude Code 已经被定位成 agentic coding system，而不是 autocomplete
- analytics、OTel、GitHub Actions、managed settings、auto mode、sandboxing 都已经是正式工作流组成部分
- 长任务、评审分离、结构化 handoff、多代理架构，这些都已出现在官方工程博客里

### 更接近实践者经验的部分
- 团队应该定期 review agent logs
- 长任务应默认产出交接文档
- 公开 issue 驱动自动化要格外保守
- 企业里应把 GitHub Action 版 Claude Code 当作“可运营系统”而不只是工具

这些经验不是每家都必须照抄，但它们和官方方向高度一致，而且在英文工程圈里已经被反复证明有效。

## 十四、和前后章节的关系
- 第 23 章讲的是高手个人习惯和进阶技巧
- 第 24 章讲的是这些技巧如何真正落到个人、团队、长任务和自动化工作里
- 到这里，你对 Claude Code 的理解应该已经从“功能使用”升级成“系统设计与工作编排”

## 学完标准
- 你知道个人开发时该怎样设计稳定节奏
- 你知道长任务为什么必须有交接物和完成定义
- 你知道团队应该如何分层配置 Claude Code
- 你知道 GitHub 自动化和组织治理最值得关注的边界
- 你已经开始把 Claude Code 当作可运营、可治理、可度量的工程系统

## 关键英文来源（截至 2026-04-25）
### 官方文档
- Claude Code GitHub Actions  
  https://code.claude.com/docs/en/github-actions
- Commands  
  https://code.claude.com/docs/en/commands
- Track team usage with analytics  
  https://code.claude.com/docs/en/analytics
- Monitoring  
  https://code.claude.com/docs/en/monitoring-usage
- Security  
  https://code.claude.com/docs/en/security
- Settings  
  https://code.claude.com/docs/en/settings
- Memory  
  https://code.claude.com/docs/en/memory
- Agent teams  
  https://code.claude.com/docs/en/agent-teams
- MCP  
  https://code.claude.com/docs/en/mcp

### Anthropic 工程博客与产品页
- Harness design for long-running application development  
  https://www.anthropic.com/engineering/harness-design-long-running-apps
- How we built our multi-agent research system  
  https://www.anthropic.com/engineering/multi-agent-research-system
- Claude Code auto mode: a safer way to skip permissions  
  https://www.anthropic.com/engineering/claude-code-auto-mode
- Beyond permission prompts: making Claude Code more secure and autonomous  
  https://www.anthropic.com/engineering/claude-code-sandboxing
- Claude Code product page  
  https://www.anthropic.com/product/claude-code
- Ramp customer story  
  https://www.claude.com/customers/ramp

### 高质量实践者经验
- Shrivu Shankar, *How I Use Every Claude Code Feature*  
  https://blog.sshh.io/p/how-i-use-every-claude-code-feature
- Simon Willison, *Using Claude Code to build a GitHub Actions workflow*  
  https://simonwillison.net/2025/Jul/1/claude-code-github-actions/
- Simon Willison, *Clinejection — Compromising Cline's Production Releases just by Prompting an Issue Triager*  
  https://simonwillison.net/2026/Mar/6/clinejection/

### 我为什么采用这些来源
- 官方文档：用于确认当前功能状态、治理能力、自动化入口和企业能力边界
- 工程博客：用于提炼 Anthropic 团队自己已经跑通的长任务和多代理方法
- 客户案例：用于观察真实组织的落地方向，但不把 marketing 数字直接当普适结论
- Simon / Shrivu：用于补上自动化、安全和日常团队使用中的真实细节
