# 15. MCP、Plugins 与扩展体系

## 重要程度
**A 级** - 强烈建议掌握。学完这一章，你才算真正把 Claude Code 从“会写代码”升级成“能接系统、能装扩展、能沉淀能力”的平台。

## 学习目标
- 搞清楚 Claude Code 扩展层的整体地图
- 理解 MCP、skill、hook、subagent、plugin 各自处在哪一层
- 学会判断一个需求到底该用哪种扩展方式

## 学什么
- Extend Claude Code / features overview
- MCP 文档
- Plugins 文档
- Plugins reference
- Claude Code 作为 MCP server
- `.mcp.json`
- `.claude-plugin/plugin.json`

## 你需要掌握
- 官方当前已经把扩展能力明确分层：`CLAUDE.md`、skills、MCP、subagents、agent teams、hooks、plugins
- MCP 负责 **连接外部工具和数据**
- plugin 负责 **把 skills、agents、hooks、MCP 等组件打包分发**
- plugin 不是 MCP 的替代，也不是 skill 的替代，而是更高一层的 **封装与分发机制**

## 一、先建立扩展地图
截至 **2026-04-25**，官方的 `Extend Claude Code` 页面已经把扩展层讲得很清楚。

可以把它压缩成下面这张图：

### `CLAUDE.md`
给 Claude 长期稳定上下文。

### Skills
给 Claude 按需知识和工作流。

### MCP
给 Claude 外部工具和数据访问。

### Subagents
给 Claude 隔离上下文的专职 worker。

### Agent Teams
给 Claude 多会话协作能力。

### Hooks
在生命周期外做确定性脚本控制。

### Plugins
把上面这些能力打包、命名空间化、安装和分发。

所以 plugin 不是“又多一种扩展能力”，而是：

**一个把多种扩展组件组织起来的容器。**

## 二、为什么 MCP 是最关键的外部连接层
官方对 MCP 的定义是：

**MCP 是一个开放协议，用来把 Claude Code 连接到外部工具、数据库和 API。**

你可以把它理解成：
- skill 负责教 Claude “怎么想”
- MCP 负责给 Claude “能连什么”

没有 MCP 时，Claude 主要能做的是：
- 读本地代码
- 跑本地命令
- 访问有限内建能力

有 MCP 之后，它能：
- 查 Jira
- 读 Figma
- 看 Slack
- 查数据库
- 连监控系统
- 操作第三方平台

这就是 Claude Code 从“本地编码代理”升级到“工作流代理”的关键。

## 三、MCP 最适合解决什么问题
官方列出的典型场景很有代表性。

### 1. 从 issue tracker 拉需求
例如：
- 读取 Jira / GitHub issue
- 基于 issue 实现功能

### 2. 从 observability 系统查数据
例如：
- 查 Sentry
- 查 Statsig
- 看部署日志

### 3. 直接查数据库
例如：
- 找符合条件的用户
- 核对 feature 使用情况

### 4. 接设计与知识系统
例如：
- Figma
- Slack
- Notion
- Google Drive

### 5. 自动化跨系统流程
例如：
- 读需求 -> 查监控 -> 改代码 -> 写 PR -> 发通知

## 四、MCP 的作用边界：它不是 prompt，不是插件，不是脚本
这是最容易混淆的点。

### MCP 不是 prompt
它提供的是工具和数据，不是行为指令。

### MCP 不是 plugin
plugin 可以包含 MCP 配置，但 plugin 本身不是连接协议。

### MCP 不是 hook
hook 是生命周期脚本，MCP 是给 Claude 增加外部可调用工具。

一句话：

**MCP 解决“Claude 能连什么”，不是“Claude 应该怎么做”。**

## 五、MCP 怎么配
官方当前推荐通过 `claude mcp` 命令来管理。

例如：
- `claude mcp add ...`
- `/mcp`

MCP server 还可以按 scope 配置。

官方文档当前说明的 scope 包括：
- `local`
- `project`
- `user`

### `local`
适合当前项目、个人、实验性配置。

### `project`
适合团队共享，通常会生成或更新 `.mcp.json`。

### `user`
适合跨项目的个人常用工具。

## 六、`.mcp.json` 是团队共享 MCP 的核心落点
官方说明 project-scoped MCP 配置会写入：
- `.mcp.json`

而且这个文件设计上就是为了进版本控制。

它的意义是：
- 团队成员获得一致的 MCP 工具配置
- 服务名称、启动方式、环境变量占位都可以共享

这和单纯在本机临时加一个 server，完全不是一个层级。

## 七、MCP 的两个很重要的工程细节
### 1. 同名 server 有优先级
官方当前的 precedence 是：
- local
- project
- user

### 2. `.mcp.json` 支持环境变量展开
支持：
- `${VAR}`
- `${VAR:-default}`

这非常适合团队共享配置但保留机器差异和密钥差异。

## 八、Claude Code 甚至可以反过来当 MCP server
这是一个很容易被忽略、但非常强的能力。

官方支持：

```bash
claude mcp serve
```

也就是说：

**Claude Code 不只是 MCP client，还可以作为 MCP server 暴露自己的工具给其他应用。**

例如可以接到：
- Claude Desktop
- 其他支持 MCP 的客户端

这时外部应用能调用 Claude Code 的本地工具能力。

## 九、MCP prompts：外部 server 还能直接变成 slash commands
官方当前还支持：

**MCP server 暴露 prompts，Claude Code 会把它们变成 slash commands。**

格式类似：
- `/mcp__servername__promptname`

这意味着：
- MCP 不只带来工具
- 还可能直接带来一组可调用命令入口

所以它已经不只是“后台连接协议”，而是开始进入交互层。

## 十、什么时候你需要的是 MCP，而不是继续手写上下文
出现这些情况时，通常该考虑 MCP：
- 你总在手动贴 issue 内容
- 你总在复制粘贴监控截图
- 你总在来回切 Slack / Jira / Figma / DB
- 你希望 Claude 直接从真实系统读数据

一句话判断：

**只要外部信息是动态的、真实的、常用的，就更该接 MCP，而不是手喂文本。**

## 十一、再看 plugin：它解决的是“打包与分发”
官方最新的 `Create plugins` 和 `Plugins reference` 已经把插件讲得很清楚。

plugin 的核心定义是：

**一个自包含目录，里面可以装 skills、agents、hooks、MCP servers、LSP servers、settings 等组件。**

所以 plugin 的价值不是新增某种能力，而是：
- 组织能力
- 命名空间
- 安装
- 更新
- 分发

## 十二、什么时候用 standalone `.claude/`，什么时候做 plugin
官方给出的建议非常明确。

### 用 standalone `.claude/`
适合：
- 单项目
- 个人实验
- 快速迭代
- 不需要共享

例如：
- `.claude/skills/`
- `.claude/agents/`
- 项目本地 hooks

### 用 plugin
适合：
- 需要跨项目复用
- 需要团队共享
- 需要 marketplace 分发
- 需要版本化发布

一句话：

**`.claude/` 适合本地生长，plugin 适合打包流通。**

## 十三、plugin 的最小结构
官方当前的核心是：
- 插件根目录
- `.claude-plugin/plugin.json`

最小 manifest 类似：

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0"
}
```

然后在插件根目录下可以放：
- `skills/`
- `commands/`
- `agents/`
- `hooks/`
- `.mcp.json`
- `.lsp.json`
- `bin/`
- `settings.json`

官方特别提醒：

**除了 `plugin.json`，其他目录都不要放进 `.claude-plugin/` 里面。**

## 十四、plugin 会带来什么额外能力
### 1. 命名空间
plugin 里的 skill 会变成：
- `/plugin-name:hello`

这样能避免命名冲突。

### 2. 可安装
而不是手动复制目录。

### 3. 可更新
适合团队统一版本。

### 4. 可分发
可面向团队，也可面向 marketplace。

### 5. 可组合
一个 plugin 可以同时带：
- skills
- agents
- hooks
- MCP
- LSP

这就是插件真正的系统价值。

## 十五、plugin 和 skill / subagent / hook / MCP 的关系
这是本章最核心的结构题。

### Skill
插件里可以装 skill。

### Subagent / Agent
插件里可以装 agent。

### Hook
插件里可以装 hooks 配置。

### MCP
插件里可以装 `.mcp.json`。

所以 plugin 是“装这些东西的包”，不是这些东西本身。

## 十六、plugin 还能带默认设置
官方当前说明，plugin 根目录可以放 `settings.json`。

当前支持的重要项包括：
- `agent`
- `subagentStatusLine`

这意味着启用某个 plugin 后，它甚至可以：
- 默认激活某个 agent
- 改变 Claude Code 的部分默认行为

这让 plugin 从“资源包”变成了更像“可安装能力模块”。

## 十七、plugin 的安装 scope
官方 reference 当前给出的安装 scope 包括：
- `user`
- `project`
- `local`
- `managed`

这和 Claude Code 整体配置体系是一致的。

所以 plugin 不只是“装在哪里”的问题，还关系到：
- 个人使用
- 团队共享
- 本地实验
- 企业托管

## 十八、什么时候应该把东西从 `.claude/` 升级成 plugin
很实用的判断标准是：

### 继续留在 `.claude/`
- 只在本项目有效
- 还在试验
- 还没稳定

### 升级成 plugin
- 多项目都要用
- 团队里多人要用
- 你希望版本化
- 你要发布出去

推荐路径是：

**先在 `.claude/` 里长出来，再打包成 plugin。**

## 十九、这一章最该形成的“选型脑图”
当你遇到需求时，可以这样问自己：

### 需要长期稳定规则？
用 `CLAUDE.md`

### 需要按需知识和工作流？
用 skill

### 需要外部系统连接？
用 MCP

### 需要隔离上下文执行？
用 subagent

### 需要强制生命周期控制？
用 hook

### 需要跨项目共享和分发？
用 plugin

这张脑图一旦立起来，后面很多选型都会非常清晰。

## 二十、和前后章节的关系
- 第 14 章讲的是 hooks：生命周期自动化
- 第 15 章讲的是整个扩展体系和外部连接层
- 第 16 章会回到具体使用面：VS Code 深度集成

## 学完标准
- 你知道 MCP 和 plugin 分别解决什么问题
- 你能说清 skill / hook / MCP / subagent / plugin 的边界
- 你知道什么时候该继续写 `.claude/`，什么时候该做 plugin
- 你知道 `.mcp.json`、`.claude-plugin/plugin.json`、plugin namespace 各自的作用
