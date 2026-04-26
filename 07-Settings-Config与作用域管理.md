# 7. Settings、Config 与作用域管理

## 重要程度
**A 级** - 强烈建议掌握。Settings 决定 Claude Code 怎么运行、默认能做什么、哪些行为应该共享给团队。不会分层，项目配置很快会混乱。

## 学习目标
- 分清 memory 和 settings 的职责
- 知道用户级、项目级、本地项目级、组织级配置分别放什么
- 能判断一项配置应该个人保存、提交到仓库，还是交给组织托管

## 先建立一个场景
你在团队项目里用 Claude Code，希望它：
- 默认使用某个模型
- 自动带上项目需要的环境变量
- 允许跑测试和 lint
- 禁止读敏感文件
- 每次改完自动触发格式化 hook

这些不应该写在聊天里，也不应该全塞进 `CLAUDE.md`。因为它们不是“知识”，而是 Claude Code 的运行边界。

这就是 settings 要解决的问题。

## 一、先分清两句话
`CLAUDE.md` 解决的是：

**Claude 应该知道什么。**

settings 解决的是：

**Claude Code 应该如何运行。**

例如：
- “本项目后端在 `apps/api`”是 memory
- “允许执行 `pnpm test`”是 settings
- “不要引入新 UI 库”是 memory
- “禁止读取 `.env`”是 permissions，也属于 settings 体系
- “每次编辑后运行 formatter”是 hook，也属于 settings 体系

只要涉及权限、环境变量、hooks、默认模式、模型、额外目录，就不要只靠 memory。

## 二、配置层级怎么理解
Claude Code 的配置不是一个文件说了算，而是多层叠加。

常见层级可以这样记：
- 用户级：影响你所有项目
- 项目级：跟着仓库走，团队共享
- 项目本地级：只影响你当前机器上的这个项目
- CLI 参数：只影响本次启动
- 组织级 managed settings：企业或组织强制策略

实际使用时，你不需要死记优先级，先问一个更重要的问题：

**这条配置应该由谁承担？**

## 三、真实场景：哪些配置该提交到仓库
适合放进项目级 `.claude/settings.json` 的，是团队希望所有人一致的行为。

例如：
- 常用测试命令 allowlist
- 敏感路径 denylist
- 项目默认 permission mode
- 项目需要的共享环境变量
- 团队统一 hooks
- 允许 Claude 访问的项目内额外目录

这类配置应该可审计、可复用、可讨论。新人 clone 仓库后，也应该得到同样的基础行为。

反过来，不要把这些内容提交到项目级 settings：
- 你的本地绝对路径
- 你的个人 token
- 你的临时实验开关
- 只适合你机器的调试命令

这些应该放进 `.claude/settings.local.json` 或你的用户级配置。

## 四、三个最常见的 settings 文件
### `~/.claude/settings.json`
用户级配置。

适合放：
- 你所有项目都想保留的默认偏好
- 通用环境变量
- 通用通知配置
- 个人常用权限偏好

### `.claude/settings.json`
项目共享配置。

适合放：
- 团队共享权限
- 项目 hooks
- 项目默认模式
- 项目级环境变量
- 应该提交到仓库的 Claude Code 行为

### `.claude/settings.local.json`
项目本地配置。

适合放：
- 本机路径
- 临时试验
- 个人例外
- 不该提交的私有设置

这个文件通常应该被 git ignore。

## 五、`/config` 是入口，不是全部
`/config` 很适合快速查看和修改常见选项，但它不是 Claude Code 配置系统的全部。

你真正要关心的是：
- 改动最终落在哪个文件
- 这个文件是不是会提交
- 这项设置影响个人还是团队
- 它是否会被 CLI 参数或 managed settings 覆盖

如果只会点 `/config`，但不知道作用域，很容易把团队规则改成个人偏好，或者把本地私有配置提交出去。

## 六、`claude config` 适合做什么
CLI 配置命令适合检查和脚本化管理配置：

```bash
claude config list
claude config get <key>
claude config set <key> <value>
claude config add <key> <value>
claude config remove <key> <value>
```

它的价值不在于“多一个改配置的方法”，而在于你能明确看到某项配置当前是什么，而不是靠猜。

## 七、真实场景：个人和团队配置怎么分
假设你在一个后端项目里工作。

你个人想让 Claude 默认用某种输出风格，这应该放用户级。

团队希望所有人都允许执行：

```text
pnpm test
pnpm lint
git status
git diff
```

这应该放项目级。

你本机有一个私有调试目录：

```text
C:\Users\you\local-fixtures
```

这应该放项目本地级，不应该提交。

公司要求所有项目都禁止读取 secrets 目录，这应该放 managed settings，而不是让每个仓库自己约定。

## 八、哪些内容典型属于 settings
最常见的是这几类：

- 权限：allow、ask、deny、defaultMode
- 路径：additionalDirectories
- 环境变量：env
- hooks：工具执行前后的自动动作
- 模型和输出偏好：model、outputStyle、statusLine
- 插件、MCP、sandbox 相关行为

如果一项内容会影响 Claude Code 的实际运行方式，它大概率属于 settings。

## 九、几个常见错放案例
### 把安全规则写进 `CLAUDE.md`
问题：`CLAUDE.md` 只是提示，不能强制阻止。

更合适：写进 permissions、hooks 或 managed settings。

### 把本地路径提交到项目级 settings
问题：别人的机器上路径不存在，还会污染仓库。

更合适：放 `.claude/settings.local.json`。

### 把团队默认命令只放在个人配置
问题：新人和其他协作者不会继承。

更合适：放 `.claude/settings.json`。

### 用 CLI 参数承担长期规则
问题：只对本次启动生效，不稳定也不可审计。

更合适：长期默认值放 settings，单次实验才用 CLI。

## 十、最小配置分层原则
你可以长期按这个规则判断：

- 只属于我：用户级
- 属于这个项目的所有人：项目级
- 只属于我在这个项目的机器环境：项目本地级
- 只影响这一次启动：CLI 参数
- 组织安全底线：managed settings

如果你拿不准，优先不要提交到仓库。先放 local，等团队确认后再提升到项目级。

## 十一、和前后章节的关系
- 第 6 章讲 memory：让 Claude 知道项目信息
- 第 7 章讲 settings：让 Claude Code 按正确方式运行
- 第 8 章会继续展开 settings 里最关键的一部分：权限体系

## 学完标准
- 你能区分 memory 和 settings
- 你知道哪些配置该个人保存，哪些该随仓库提交
- 你不会把私有路径、个人 token 或临时实验提交到项目级 settings
- 你能用“谁应该承担这条约束”来判断配置作用域
