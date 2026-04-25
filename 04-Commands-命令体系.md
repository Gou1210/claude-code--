# 4. Commands 命令体系

## 重要程度
**S 级** - 必须掌握，不会就很难高效用

## 学习目标
- 建立完整命令地图，而不是只记几个 `/xxx`
- 理解内建命令、CLI 命令、自定义命令、MCP 命令的区别
- 知道哪些命令解决的是上下文问题，哪些解决的是配置问题，哪些解决的是自动化问题

## 先纠正一个常见误区
- Claude Code 里的“命令”不等于“所有功能都靠 slash command”
- 有些能力是 slash command
- 有些能力是启动参数或 CLI 子命令
- 有些能力来自 `.claude/commands` 里的自定义命令
- 有些能力来自 MCP server 暴露出来的 prompt / command
- 还有一些命令是否可见，取决于平台、版本、plan、IDE 集成状态和当前环境

## 学什么
- 内建 slash commands
- CLI 层命令和启动方式
- custom commands
- MCP commands
- 命令发现与帮助系统
- 命令的权限边界与上下文代价

## 你需要掌握
- `/` 菜单不是“帮助页”，而是 Claude Code 的操作总入口
- 命令的核心价值不是省几个字，而是切换工作状态：清上下文、改配置、拉权限、查状态、进入特定工作流
- 官方文档中的命令集合会演进，学习时应优先以当前官方 docs 和 `/help` 菜单为准
- 截至 **2026-04-25**，官方文档里稳定可见的一批内建命令，重点包括：
  - `/help`
  - `/clear`
  - `/compact`
  - `/config`
  - `/cost`
  - `/doctor`
  - `/init`
  - `/login`
  - `/logout`
  - `/memory`
  - `/mcp`
  - `/model`
  - `/permissions`
  - `/pr_comments`
  - `/review`
  - `/status`
  - `/terminal-setup`
  - `/vim`
  - `/agents`
  - `/add-dir`
- 你在一些文章、截图或旧笔记里看到的 `/resume`、`/rewind`、`/usage`、`/effort`、`/remote-control`、`/schedule`、`/autofix-pr`、`/ultrareview`、`/less-permission-prompts`，不应先假设为“所有环境都稳定存在的通用内建命令”

## 建议先分四层理解
1. **发现与求助层**：`/help`、`/status`、`/doctor`
2. **上下文与会话层**：`/clear`、`/compact`、`/memory`
3. **配置与权限层**：`/config`、`/permissions`、`/model`、`/add-dir`
4. **工作流与扩展层**：`/review`、`/pr_comments`、`/agents`、`/mcp`、custom commands

## 一、最核心的内建命令

### 1. `/help`
- 看当前环境下“你实际可用”的命令，而不是记忆里的命令
- 当你不确定某个命令还在不在、名字是否变了时，先看它
- 它也会让你意识到：命令集合不是一成不变的

### 2. `/clear`
- 清空当前会话上下文，重新开始
- 适合任务已经切换、上下文污染严重、模型开始“带着旧问题思考”时使用
- 它解决的是“换题”问题，不是“压缩”问题

### 3. `/compact`
- 把当前长会话压缩成更小的可继续上下文
- 适合任务还没结束，但上下文已经很重
- 它解决的是“继续当前任务但减负”问题

### 4. `/config`
- 查看或调整配置
- 这是连接 settings、模型、工具行为、默认工作方式的重要入口
- 学 Claude Code 不能只学对话提示，必须学会看配置

### 5. `/permissions`
- 直接管理权限策略
- 当你觉得 Claude 太烦、总在请求批准，或者反过来太放权时，这就是核心控制点
- 它和后面的 sandbox、allowlist、auto mode 是强相关主题

### 6. `/memory`
- 管理记忆与项目说明
- 典型用途不是“存一条聊天记录”，而是把稳定规则沉淀为项目长期上下文
- 它和 `CLAUDE.md`、自动记忆、团队规范是同一套体系

### 7. `/model`
- 切换或确认当前模型
- 这是成本、速度、推理强度之间的直接杠杆
- 很多“Claude 今天怎么变笨/变慢”的问题，本质上是模型与任务不匹配

### 8. `/cost`
- 看成本或用量信息
- 它相当于你对“这段工作到底花了多少”的反馈面板
- 如果你长期用 Claude Code 做重任务，这个命令很重要

### 9. `/status`
- 快速看当前会话或环境状态
- 适合排查“现在到底用了什么配置、接了哪些能力、处于什么状态”

### 10. `/doctor`
- 做诊断与排障
- 当 Claude Code 行为异常、集成异常、工具不可用时，比盲猜更有效

## 二、把命令按“问题类型”去记

### 1. 遇到上下文太重
- 还要继续做当前任务：先想 `/compact`
- 任务已经切了：先想 `/clear`
- 需要把长期规则沉淀下来：先想 `/memory`

### 2. 遇到权限打断太多
- 先看 `/permissions`
- 再去理解后面的 allowlist、sandboxing、auto mode
- 不要一上来怪模型“太笨”，很多时候只是权限模式没配对

### 3. 遇到行为异常或环境不对
- 先看 `/status`
- 再跑 `/doctor`
- 再去检查 `/config`

### 4. 遇到评审、子代理、外部能力接入
- Code review：先看 `/review`
- 处理 PR 评论：先看 `/pr_comments`
- 管理 subagents：先看 `/agents`
- 查看 MCP 服务与命令：先看 `/mcp`

## 三、不是所有“重要操作”都是 slash command

### 1. Session 恢复更常见的是 CLI 入口
- 例如继续最近一次会话、按 session id 恢复，会更多出现在 CLI 启动参数或启动方式里
- 所以“resume”是重要概念，但不一定非得作为 slash command 来记

### 2. 有些能力是 IDE 或平台入口
- 比如 remote control、可视化 context indicator、权限模式切换
- 它们很重要，但不一定表现为通用 slash command

### 3. 有些能力是云端工作流，不应和本地内建命令混为一谈
- 例如 PR 自动修复、持续观察 CI、按计划执行任务等
- 这些更像“产品工作流能力”，而不一定是你本地聊天框里永远存在的一条命令

## 四、Custom Commands 才是“命令体系”真正进阶的地方

### 你要建立的认识
- 内建命令解决的是通用控制问题
- 自定义命令解决的是“把你自己的工作流产品化”
- 当团队开始稳定使用 Claude Code 时，真正提高效率的往往不是多背几个内建命令，而是沉淀自己的 commands

### Custom Commands 的典型来源
- 项目级：`.claude/commands`
- 用户级：`~/.claude/commands`

### 适合做成 custom command 的内容
- 固定格式的 code review
- 固定格式的 bug triage
- 固定格式的重构检查清单
- 固定格式的 PR 总结
- 团队统一的测试、验证、交付流程

### 为什么它比复制提示词更高级
- 你不需要每次重写长提示
- 团队成员调用的是同一套操作入口
- 可以配合参数，让命令像“小工具”一样复用
- 可以配合 frontmatter 约束可用工具、描述用途、减少误用

## 五、MCP Commands 是“把外部系统接进命令菜单”

### 你要理解
- MCP 不是一个单独工具，而是 Claude Code 接入外部能力的标准接口
- 一旦某个 MCP server 暴露了 prompts / commands，它们会出现在命令体系里
- 所以命令菜单不是静态的，它会随着你接的 MCP server 而扩张

### 这意味着什么
- 你的命令系统可以连接 GitHub、设计系统、文档库、数据库、内部平台
- Claude Code 不再只是“会聊天的本地代理”，而是一个可以被命令化的工作台
- 命令体系越成熟，你的团队越接近“自然语言控制的软件工具链”

## 六、这章最容易混淆的几个点

### 1. `/clear` 和 `/compact` 不是一回事
- `/clear` 是换会话思路
- `/compact` 是保留任务、压缩上下文

### 2. `/memory` 不是“聊天历史存档”
- 它更接近长期规则和稳定偏好的沉淀入口

### 3. `/config` 不只是“看看设置”
- 它是连接模型、权限、工作方式的控制面板

### 4. 命令菜单里的东西不一定全是官方内建
- 可能有 custom commands
- 可能有 MCP commands
- 可能受当前集成环境影响

### 5. 旧截图里的命令名不一定还有效
- 学习时优先看当前版本的 `/help`
- 不要把零散帖子里的命令表当作稳定真相

## 七、建议这样学

### 第一阶段：先练最常用 6 个
- `/help`
- `/clear`
- `/compact`
- `/config`
- `/permissions`
- `/memory`

### 第二阶段：补齐系统感
- `/model`
- `/cost`
- `/status`
- `/doctor`
- `/mcp`
- `/agents`

### 第三阶段：开始做你自己的命令层
- 把重复提示词迁移到 custom commands
- 把团队规范写进项目命令
- 把外部系统能力通过 MCP 接进来

## 八、建议你在脑子里形成的“命令地图”
- **求助与发现**：`/help`
- **看状态**：`/status`、`/cost`、`/doctor`
- **控会话**：`/clear`、`/compact`
- **控规则**：`/memory`
- **控行为**：`/config`、`/permissions`、`/model`
- **控工作流**：`/review`、`/pr_comments`、`/agents`
- **控扩展**：`/mcp`、custom commands

## 九、和后续章节的关系
- 这章回答的是：**Claude Code 里有哪些命令入口**
- 第 5 章回答的是：**为什么上下文会膨胀，以及怎么管**
- 第 6 章回答的是：**memory / CLAUDE.md / auto memory 到底怎么分工**
- 第 7 章回答的是：**config 和作用域到底怎么管理**
- 第 8 章回答的是：**permissions / sandbox / allowlist / auto mode 怎么配**
- 第 11、15 章回答的是：**skills、MCP、plugins 怎么把命令体系继续扩展**

## 学完标准
- 你能区分内建命令、CLI 命令、自定义命令、MCP 命令
- 你知道“该清空会话”时用 `/clear`，“该减重继续”时用 `/compact`
- 你知道权限问题优先看 `/permissions`，配置问题优先看 `/config`
- 你不会再把旧截图里的命令名默认当成当前版本标准答案
- 你开始把命令看作“工作流控制面板”，而不是“聊天快捷键”
