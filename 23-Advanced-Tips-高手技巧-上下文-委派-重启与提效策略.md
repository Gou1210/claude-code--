# 23. Advanced Tips：高手技巧、上下文、委派、重启与提效策略

## 一、先纠正一个误区：高手技巧不是“神秘提示词”
截至 **2026-04-25**，官方和高质量实践者给出的共识都很一致：

真正拉开差距的，不是某句神奇 prompt，而是以下四件事：

1. 让 Claude 进入上下文的信息尽量少、尽量准
2. 让任务自带验证闭环，而不是靠你肉眼兜底
3. 让流程里“必须发生”的动作由 hooks 或环境保证，而不是靠模型记性
4. 在 session 变脏之前及时重启，而不是盲目在同一会话里硬拖

官方 `Best Practices` 甚至把这件事讲得很直接：

**大多数 best practices，本质上都在解决同一个约束：context window 很快会被塞满，而且随着它变满，表现会下降。**

## 二、第一高手习惯：把 context 当预算，不当垃圾桶
Anthropic 在 **2025-09-29** 的工程博客《Effective context engineering for AI agents》里，把 context 讲得非常明确：

- context 是有限资源，不是越多越好
- 好的 context engineering，是找到“**最小但高信号**”的信息组合
- 工具、提示、例子、消息历史，都要围绕这个原则收缩

这对 Claude Code 的日常使用有三个直接含义。

### 1. 少解释“背景故事”，多给直接可消费材料
差写法：
- 我记得这个问题可能在某个认证模块里
- 之前有人改过，反正现在不太对

高手写法：
- 看 `@src/auth` 和 `@src/session.ts`
- 重点检查 token refresh 和 session timeout
- 验证方式是复现这个失败用例并跑相关测试

### 2. 少让 Claude 背整本手册，多让它按需读
Anthropic 官方在 `Skills` 文档里明确说：

- 如果某段 `CLAUDE.md` 已经长成“流程、剧本、检查清单”，它更适合变成 skill
- skill 的正文只会在使用时加载，不会像 `CLAUDE.md` 一样每次启动都进上下文

一句话记忆：

**长期稳定事实放 `CLAUDE.md`，按需执行的程序化知识放 skill。**

### 3. 工具和能力面不要膨胀
Anthropic 在 context engineering 博客里专门点名：

- 工具能力重叠太多，会让代理更难选
- 如果人类都说不清某个场景到底该用哪个工具，模型更不可能稳定选对

所以高手不是“把能接的 MCP 都接上”，而是只给当前任务真正会用到的能力面。

## 三、第二高手习惯：把 `CLAUDE.md` 当宪法，不当操作手册
英文高手实践里，几乎所有人都会强调根目录 `CLAUDE.md` 的重要性；但真正的分歧不在“要不要写”，而在“写到什么程度”。

### 推荐放进去的东西
- 项目长期稳定规则
- 架构约束
- 测试与验证命令
- 团队提交 / PR 规范
- 容易从代码外部才知道的坑

### 不推荐放进去的东西
- 会频繁变化的操作流程
- 超长教程
- 某个偶发任务的临时打法
- 大量互斥场景混在一起的规则

Shrivu Shankar 的经验很有代表性：他把 `CLAUDE.md` 当代理的“constitution”，但同时又强调，真正企业环境里需要把它 **严格维护**，不能让它无节制膨胀。

你可以把这一点理解成：

**`CLAUDE.md` 不是越厚越好，而是越像“稳定制度层”越好。**

## 四、第三高手习惯：流程化知识做成 skill，不要塞回主提示
官方 `Skills` 和 Anthropic **2025-10-16** 的工程博客《Equipping agents for the real world with Agent Skills》给出的结论很清晰：

- 真实工作不只需要模型知识，还需要 **procedural knowledge** 和 **organizational context**
- skill 的优势，不是“它更高级”，而是它能把知识做成 **按需渐进加载**

这意味着：

### 适合做 skill 的内容
- 代码 review checklist
- 某类 bug 的排查剧本
- 某种文档产出的固定模板
- 需要顺带调用脚本、模板、示例文件的流程

### 不适合做 skill 的内容
- 一个仓库全员默认都该知道的基础规则
- 只是一句很短的偏好
- 纯粹为了偷懒而复制一个大提示词，但它本身没有复用价值

### 一个关键细节
官方文档还提醒：

- skill 一旦被调用，其内容会进入会话并在本 session 里持续存在

所以 skill 也不是“越大越好”。如果你把 skill 写成第二本说明书，它照样会吃掉上下文预算。

## 五、第四高手习惯：`/compact` 是工具，不是救命稻草
这是英文社区里最值得翻成中文、但很多中文资料没讲透的一个点。

Anthropic 在 **2026-03-24** 的工程博客《Harness design for long-running application development》里专门区分了：

- `compaction`：在原 session 内做摘要，继续同一代理
- `context reset`：清空上下文，换一个新代理继续，但靠结构化交接带状态

官方结论很重要：

**compaction 能保连续性，但不是真正的 clean slate。**

也就是说，长任务里你经常会遇到一种情况：
- session 还没爆
- 但已经开始“有点糊”“有点急着收尾”“有点忘前文”

这时只 compact，不一定够。

### 更稳的两种重启法
#### 1. 简单重启
适用：
- 改动还比较局部
- 你主要担心 session 被脏上下文污染

做法：
- 让 Claude 重新读当前分支改动或关键文件
- `/clear` 后开新轮

#### 2. 文档化交接后重启
适用：
- 任务已经很长
- 中间产生了很多决策、未完成项和已知坑
- 你担心新 session 会丢失真实进度

做法：
1. 让 Claude 把当前目标、已完成内容、待办、失败尝试、下一步写进一个 `.md`
2. 新开 session
3. 让 Claude 读这个交接文件继续

这其实就是 Anthropic long-running harness 里“结构化 handoff artifact”的人类版。

### 什么情况下你该果断重开
- 连续两次纠偏还在原地绕
- 开始提前收尾、假装完成
- 老是引用过期信息
- 把已经被否掉的路径继续往下推
- 输出里混入很多无关历史

## 六、第五高手习惯：强制规则交给 hooks，不交给模型记忆
如果你问很多英文高手“最被低估的 Claude Code 能力是什么”，hooks 几乎一定会排前列。

官方 hooks 文档现在已经很成熟，事件覆盖也很全：
- session 级
- turn 级
- tool call 级
- 还包括 `Notification`、`ConfigChange`、`WorktreeCreate` 这类异步事件

### 该怎么理解 hooks 的地位
- `CLAUDE.md` 是建议层
- skill 是流程层
- hook 是执行控制层

也就是：

**凡是“必须发生”或“必须阻止”的东西，优先考虑 hook，而不是写在提示里祈祷模型记住。**

### 最值钱的两类 hook
#### 1. Block-at-submit / block-at-commit
典型例子：
- 测试没过不允许 commit
- 关键目录变更前必须触发额外检查
- 改配置文件时必须给出理由

#### 2. Hint hooks
典型例子：
- 正在做次优操作时给提示
- 检测到危险目录或危险命令时提醒
- 输出一个团队 checklist

### 为什么很多高手不喜欢 block-at-write
这点在 Shrivu 的实践总结里讲得非常直白：

- 在 `Edit` / `Write` 阶段频繁阻断，会打断代理当前计划
- 模型会被打乱节奏，甚至开始“绕规则”

所以更稳的做法通常是：

**允许它先完成一轮实现，再在 commit、submit、deploy、PR 这些边界点做强校验。**

## 七、第六高手习惯：自定义命令要极简，别把系统做成“命令宇宙”
很多人一上来就喜欢堆大量 `/foo`、`/bar`、`/baz`。

但官方现在其实已经把很多“自定义命令”的角色收进了 skills：
- `.claude/commands/` 还兼容
- 但推荐格式是 `.claude/skills/<name>/SKILL.md`

高手一般遵循一个克制原则：

### 真值得做成命令 / skill 的
- 高频重复
- 输入输出模式稳定
- 能明显减少你每次重新解释任务

### 不值得做成命令 / skill 的
- 只会用一次
- 只是一个很短的 prompt shortcut
- 功能边界模糊，最后变成万能入口

一句话记忆：

**命令和 skill 应该是高价值套路的压缩器，不是把所有想法都包装成按钮。**

## 八、第七高手习惯：知道何时用 CLI，何时用 MCP，何时用 skill
这是最容易被讲乱的一组关系。

### 1. 优先 CLI
当目标是：
- 跑测试
- 查 git
- 调现有开发工具
- 走团队本来就成熟的命令行流程

CLI 往往最直接、最接近真实工程环境，也最少额外抽象。

### 2. 优先 MCP
官方 `MCP` 文档给的适用条件很清楚：

- 你开始反复把 issue、监控、数据库结果、外部系统数据复制到聊天里
- 你希望 Claude 直接读和操作那个系统

但高质量实践里还有一个更成熟的理解：

**MCP 最有价值的角色，往往不是“再做一套 API 封装”，而是做受控的认证、网络和权限边界。**

Shrivu 的这点被 Simon Willison 明确点名赞同过：
- MCP 更适合做“安全网关”
- 给几个高价值、高权限、边界清晰的动作
- 而不是暴露一堆碎工具让代理自己拼

### 3. 优先 skill
当你要复用的是：
- 一段流程
- 一套团队做事方法
- 一种任务剧本

不是去接外部系统，而是把“怎么做这类事”固化下来，这时 skill 最合适。

## 九、第八高手习惯：委派不是越多越强，而是越准越强
很多人一听到 subagent、agent teams，就会本能觉得“更多代理 = 更高级”。

这不对。

### 官方给出的边界
- subagent：适合聚焦、隔离、降噪、保主上下文
- agent teams：适合多独立 session 协作，但 **截至 2026-04-25 仍是 experimental**

### Anthropic 工程博客给出的更深结论
在 **2025-06-13** 的《How we built our multi-agent research system》里，Anthropic 明确指出：

- 多代理在“宽度优先、强并行”的研究任务上收益很大
- 但 coding 并不是最天然适合多代理的领域
- 很多 coding 任务真正可并行的部分没有你想象得多
- 协调与 token 成本会上升很快

### 高手怎么判断值不值得委派
适合委派：
- 大量搜索、归档、对比、总结
- 会制造很多噪音的调查
- 可分成几块独立推进的任务
- 主任务需要保干净上下文

不适合委派：
- 眼前一步就卡住，结果立刻要用
- 高频往返澄清
- 强耦合、需要持续共享同一上下文

一句话记忆：

**委派的目标不是“显得高级”，而是把噪音和并行收益从主线程里剥离出去。**

## 十、第九高手习惯：自动化的前提是安全边界足够清楚
这部分如果只看中文二手资料，很容易被讲成“少点几次审批”。其实官方最新文档和工程博客讲得更深。

### Auto mode 的真正定位
Anthropic 在 **2026-03-25** 的《Claude Code auto mode: a safer way to skip permissions》里给出的定义是：

- 不是无限放权
- 是让分类器代替人审一部分权限决策
- 默认只信任你当前正在工作的 git 仓库

也就是说，auto mode 的核心不是“更快”，而是：

**在审批疲劳和完全裸奔之间，找一个带防线的中间层。**

### Sandboxing 的真正价值
Anthropic 在 **2025-10-20** 的 sandboxing 博客里给了一个非常硬的数字：

- 内部使用里，sandboxing 安全地把 permission prompts 降低了 **84%**

为什么这件事值钱？
- 少审批 = 更少疲劳
- 有文件系统与网络边界 = 被 prompt injection 命中时也更难出大事

所以高手不是简单追求“全自动”，而是优先追求：

1. 明确 trust boundary
2. 让代理在边界内自由行动
3. 越出边界时再让人接手

## 十一、第十高手习惯：把“会话变脏”当成一类可观测故障
高手会把 session 质量当成运行状态，而不是把所有问题都怪模型。

### 常见脏会话症状
- 开始复读旧结论
- 用高置信度包装低质量判断
- 明明没验证却急着宣布完成
- 连续出现与当前任务无关的历史痕迹
- 在几个失败路径之间打转

### 常见修法
- 缩任务边界
- 切回可验证闭环
- 清掉无关上下文
- 外部化当前状态再重开
- 把高噪音调查拆给 subagent

这也是为什么高手很少迷信“同一个 session 聊到底”。他们更像在管理一个会工作的系统，而不是在维持一段聊天。

## 十二、这一章最值得沉淀下来的十条技巧
1. 先想“哪些信息必须进入上下文”，再开始说话。
2. `CLAUDE.md` 写事实与规则，skill 写流程，hook 管强制。
3. 不要把所有可复用内容都堆回根提示。
4. `/compact` 用来续命，不用来掩盖坏 session。
5. 长任务优先考虑“文档化交接后重启”。
6. 强验证放在提交边界，而不是编辑中途乱打断。
7. 工具面越宽，代理越容易犹豫；能收就收。
8. 委派给 subagent 是为了降噪，不是为了显得复杂。
9. auto mode 和 sandbox 的重点是边界，不只是少点按钮。
10. 一旦会话变脏，越早重开，返工越少。

## 十三、哪些是官方结论，哪些是实践者经验
### 更接近官方结论的部分
- context window 是首要约束
- 给 Claude 验证方式是最高杠杆动作
- skill 适合承载程序化知识
- hook 适合强制执行规则
- auto mode、sandboxing、MCP、agent teams 都有明确边界和安全模型

### 更接近实践者经验的部分
- `/compact` 应该保守使用
- 长任务更稳的方式是“外部交接 + 新 session”
- block-at-submit 往往比 block-at-write 更稳
- MCP 最值得做成“安全网关型高价值工具”，而不是大而全 API 镜像

这些不是官方硬规则，但它们在英文开发者圈里已经形成了很强的共识，而且和官方一手资料并不冲突。

## 十四、和前后章节的关系
- 第 9 章讲的是 best practices 总原则
- 第 10 章讲的是常见工作流模板
- 第 23 章讲的是：当你已经会用以后，怎样把使用质量拉高到“高手稳定输出”的级别
- 第 24 章会继续把这些技巧，落到个人、团队、长任务和自动化的真实工作实践

## 学完标准
- 你知道为什么高水平使用的核心不是 prompt 魔法，而是 context engineering
- 你知道 `CLAUDE.md`、skills、hooks 三者的分工
- 你知道为什么 `/compact` 不能替代重启
- 你知道什么时候应该委派，什么时候不该
- 你知道自动化的前提不是胆子大，而是边界清楚

## 关键英文来源（截至 2026-04-25）
### 官方文档
- Best Practices for Claude Code  
  https://code.claude.com/docs/en/best-practices
- Skills  
  https://code.claude.com/docs/en/skills
- Hooks  
  https://code.claude.com/docs/en/hooks
- MCP  
  https://code.claude.com/docs/en/mcp
- Security  
  https://code.claude.com/docs/en/security
- Configure auto mode  
  https://code.claude.com/docs/en/auto-mode-config
- Agent teams  
  https://code.claude.com/docs/en/agent-teams
- Commands  
  https://code.claude.com/docs/en/commands

### Anthropic 工程博客
- Effective context engineering for AI agents  
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Equipping agents for the real world with Agent Skills  
  https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- Beyond permission prompts: making Claude Code more secure and autonomous  
  https://www.anthropic.com/engineering/claude-code-sandboxing
- Claude Code auto mode: a safer way to skip permissions  
  https://www.anthropic.com/engineering/claude-code-auto-mode
- Harness design for long-running application development  
  https://www.anthropic.com/engineering/harness-design-long-running-apps
- How we built our multi-agent research system  
  https://www.anthropic.com/engineering/multi-agent-research-system

### 高质量实践者经验
- Shrivu Shankar, *How I Use Every Claude Code Feature*  
  https://blog.sshh.io/p/how-i-use-every-claude-code-feature
- Simon Willison 对上文的评注  
  https://simonwillison.net/2025/Nov/2/how-i-use-every-claude-code-feature/

### 我为什么采用这些来源
- 官方文档：用于确认功能边界、配置方式、安全模型、当前产品状态
- Anthropic 工程博客：用于提炼官方团队已经验证过的工程方法，而不是纯 marketing 说法
- Shrivu / Simon：用于补上“高手到底怎么用”的细节，但只采用与官方材料互相印证的部分
