# 7. Settings、Config 与作用域管理

## 重要程度
**A 级** - 强烈建议掌握。它不如 memory 那么“天天可见”，但一旦不会配，项目共享、团队治理、权限默认值、环境变量注入都会很乱。

## 学习目标
- 建立 Claude Code 配置层的完整地图
- 分清 `/config`、`claude config`、`settings.json`、环境变量分别干什么
- 掌握用户级、项目级、本地项目级、组织级配置的边界

## 先纠正一个常见误区
很多人把 Claude Code 的“配置”都理解成：
- 进 `/config` 改几项开关
- 或者在 `CLAUDE.md` 写点说明

这两种理解都不完整。

Claude Code 里至少有三类不同东西：
- **提示与说明**：`CLAUDE.md`、rules、skills
- **硬配置**：`settings.json`、managed settings、CLI flags、环境变量
- **交互入口**：`/config`、`claude config ...`

其中真正决定行为边界的，是第二类。

## 学什么
- `/config`
- `claude config list/get/set/add/remove`
- `~/.claude/settings.json`
- `.claude/settings.json`
- `.claude/settings.local.json`
- managed settings
- 环境变量与 CLI 参数

## 你需要掌握
- `settings.json` 才是官方的层级化配置机制
- `/config` 是配置入口，不是唯一配置存储
- 有些全局 UI/交互偏好保存在 `~/.claude.json`，不在 `settings.json`
- 项目共享配置和个人本地偏好应该分层，不要混写
- CLI 参数是单次 session 覆盖
- managed settings 优先级最高，通常不能被用户覆盖

## 一、先建立“配置层次图”
你可以把 Claude Code 的配置理解成五层，从低到高逐步覆盖：

1. 用户级 `~/.claude/settings.json`
2. 项目级 `.claude/settings.json`
3. 项目本地级 `.claude/settings.local.json`
4. CLI 启动参数
5. 组织级 managed settings

截至 **2026-04-25**，官方文档给出的优先级是“从高到低”：
- managed settings
- command line arguments
- `.claude/settings.local.json`
- `.claude/settings.json`
- `~/.claude/settings.json`

所以不要想当然认为“命令行永远最大”。在 Claude Code 里，组织级托管策略可以压过命令行。

## 二、`/config` 是入口，不是全部
`/config` 的价值在于：
- 查看当前主要配置状态
- 修改常见设置
- 在交互中快速切换模型、模式或偏好

但它只是一个入口层。

你真正要理解的是：这些配置最终落在哪个文件、属于哪个作用域、是否应提交到仓库。

所以，学习路径不要停在“会点 `/config`”。

## 三、真正该记住的三个 settings 文件
### 1. `~/.claude/settings.json`
用户级全局配置。

适合放：
- 你在所有项目都想保留的默认偏好
- 你的全局模型默认值
- 通用通知、环境变量、权限偏好

### 2. `.claude/settings.json`
项目共享配置。

适合放：
- 团队共享的权限规则
- 项目默认模型
- 共享 hooks
- 项目统一环境变量
- 团队约定的 `defaultMode`

这是应该随仓库提交的。

### 3. `.claude/settings.local.json`
项目本地私有配置。

适合放：
- 你自己在这个项目里的试验性设置
- 个人本地路径
- 本地调试临时默认值
- 不适合提交到仓库的偏好

它通常应被 git ignore。

## 四、还有一个容易漏掉的文件：`~/.claude.json`
官方文档指出，一部分 **全局交互/UI 配置** 存在 `~/.claude.json`，而不是 `settings.json`。

比如部分版本里：
- IDE 自动连接
- IDE 扩展自动安装
- 外部编辑器上下文偏好

这意味着：
- 不是所有配置都在 `settings.json`
- 遇到“明明在 `/config` 改了，但没写进你以为的那个文件”的情况，不要惊讶

## 五、哪些内容典型地属于 settings
最值得优先掌握的有这几类：

### 1. 权限
例如：
- `permissions.allow`
- `permissions.ask`
- `permissions.deny`
- `defaultMode`
- `additionalDirectories`

这是第 8 章的核心基础。

### 2. 环境变量
可以放在 `env` 里，为每次 session 注入固定环境变量。

适合：
- 团队统一的一组运行参数
- Claude Code 本身需要的行为开关

### 3. hooks
Claude Code 支持在工具执行前后挂钩，这属于硬配置，不属于 memory。

### 4. model / outputStyle / statusLine
这些决定的是 Claude Code 的默认行为和体验，而不是“提示词记忆”。

### 5. 插件、worktree、sandbox 相关设置
这些更偏进阶，但本质上也属于 settings 层。

## 六、什么时候该写进 settings，什么时候不该
### 适合写进 settings 的
- 需要程序层强制生效的行为
- 团队共享的默认权限与安全策略
- 运行时环境变量
- 默认模型与模式
- hooks、插件、工作目录扩展

### 不适合写进 settings 的
- 编码规范说明
- 项目背景介绍
- 架构原则解释
- 发布步骤文档

这些更适合 `CLAUDE.md` 或 skill。

一句话：

**settings 管“系统怎么运行”，memory 管“Claude 应该知道什么”。**

## 七、作用域管理的核心不是“文件位置”，而是“谁应该承担这个约束”
这是很多团队配置混乱的根源。

### 应该放用户级的
- 只属于你个人的长期偏好
- 所有项目通用，但不值得强推给团队的设置

### 应该放项目级的
- 团队所有人都该一致的行为
- 与仓库绑定的权限、hooks、默认模型、共享目录规则

### 应该放项目本地级的
- 只在你本机生效的试验配置
- 本地路径、个人临时开关、未成熟方案

### 应该放组织级 managed settings 的
- 安全底线
- 合规要求
- 不允许被仓库作者绕开的企业策略

## 八、几个最常见的错放案例
### 错放 1：把共享安全策略写进个人 settings
问题：
- 团队不一致
- 新人接手即失效

正确做法：
- 放到项目级或 managed settings

### 错放 2：把个人本地路径提交到 `.claude/settings.json`
问题：
- 别人机器上失效
- 配置污染仓库

正确做法：
- 放到 `.claude/settings.local.json`

### 错放 3：把“必须执行的禁止项”只写在 `CLAUDE.md`
问题：
- 它只是提示，不是强约束

正确做法：
- 真要禁止，配 `permissions.deny`、managed settings 或 hooks

## 九、CLI 参数与 settings 的关系
CLI 参数适合：
- 一次性覆盖
- 临时实验
- 脚本化场景

例如：
- 临时切模型
- 临时切 permission mode
- 启动时加额外目录

但它不适合承担长期团队约束，因为：
- 不稳定
- 不可审计
- 不会自动共享给其他协作者

所以：
- 临时行为，用 CLI
- 稳定默认值，用 settings

## 十、建议你最先学会的几个命令
### 交互入口
- `/config`

### CLI 管理入口
- `claude config list`
- `claude config get <key>`
- `claude config set <key> <value>`
- `claude config add <key> <value>`
- `claude config remove <key> <value>`

它们的价值不只是“会改”，更重要的是让你能明确地看见当前有效配置，而不是靠猜。

## 十一、建议你在脑子里形成的“配置分层原则”
最推荐的分法是：

### 个人长期偏好
放 `~/.claude/settings.json`

### 团队共享默认值
放 `.claude/settings.json`

### 本地试验和个人例外
放 `.claude/settings.local.json`

### 企业强制底线
放 managed settings

### 单次会话覆盖
用 CLI 参数

如果你按这个原则配置，后面接权限、hooks、MCP、插件时就不容易混乱。

## 十二、和前后章节的关系
- 第 6 章解决“Claude 该知道什么”
- 第 7 章解决“Claude Code 该如何运行”
- 第 8 章会在这个基础上展开 permission modes、allow/ask/deny、sandbox 与 working directories

## 学完标准
- 你知道 `/config` 只是入口，不是完整配置系统
- 你能区分 `settings.json`、`settings.local.json`、`~/.claude.json`
- 你知道什么该放用户级，什么该放项目级，什么只能放 managed
- 你不再把 memory 和 settings 混为一谈
