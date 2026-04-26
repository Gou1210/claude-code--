# 8. 权限体系：Permission Modes、Allow/Ask/Deny、Sandboxing 与 Auto

## 重要程度
**S 级** - 必须掌握。Claude Code 好不好用，不只看模型能力，还看你能不能把权限设到“低风险少打断，高风险不放飞”。

## 学习目标
- 理解权限不是一个总开关，而是多层控制
- 知道 mode、allow/ask/deny、目录边界、sandbox 分别管什么
- 能为日常开发配置一套安全但不烦人的默认策略

## 先建立一个场景
你让 Claude Code 修一个测试失败问题。它需要：
- 读相关源码
- 改几个文件
- 跑测试
- 查看 git diff
- 可能安装依赖
- 绝不能读取 `.env`
- 绝不能直接 `git push`

如果每一步都问你，效率很低。
如果什么都不问，风险很高。

权限体系要解决的，就是在这两者之间找到边界。

## 一、权限不是一个开关
Claude Code 的权限至少要分四层看：

- mode：默认遇到动作时怎么处理
- rules：哪些动作允许、询问或拒绝
- directory boundary：能访问哪些目录
- sandbox：系统层面还能碰到什么资源

很多配置错误，都来自只看 mode，不看其他层。

例如 auto mode 并不等于“随便做任何事”。它仍然会受到 deny 规则、目录边界、managed settings、sandbox 的限制。

## 二、mode 解决默认倾向
permission mode 决定的是：当一个动作没有被明确规则覆盖时，Claude Code 默认怎么处理。

常见理解方式：
- `default`：稳妥模式，遇到高风险动作会询问
- `acceptEdits`：更少打断地接受文件编辑
- `plan`：只分析和规划，不改文件，不执行命令
- `dontAsk`：没被允许的动作倾向于直接拒绝
- `bypassPermissions`：跳过权限提示，风险最高
- auto / auto-accept：减少常规动作打断，但不是越权模式

实际名称可能因版本和界面略有变化，以当前 `/permissions`、`/config` 和 `claude --help` 为准。

## 三、rules 决定细粒度行为
真正日常好不好用，主要靠 allow、ask、deny。

- `allow`：命中后直接执行
- `ask`：命中后询问
- `deny`：命中后拒绝

规则匹配要记住一个核心顺序：

**deny 优先，其次 ask，最后 allow。**

这意味着安全底线应该写进 deny，而不是指望 Claude 记得某条说明。

## 四、真实场景：个人开发的实用权限
对一个普通代码仓库，比较实用的思路是：

低风险高频动作直接 allow：
- `git status`
- `git diff`
- `pnpm test`
- `pnpm lint`
- `pnpm typecheck`

高风险动作保留 ask：
- 安装依赖
- 访问外网
- 删除文件
- 修改配置文件
- 执行数据库迁移

明确危险动作直接 deny：
- 读取 `.env`
- 读取 secrets 目录
- `git push`
- 删除大目录
- 修改生产配置

这样 Claude 能顺畅完成常规开发，但不会越过关键安全线。

## 五、`Bash` 规则不要乱放宽
权限规则里最容易误配的是 Bash。

你要区分：
- 允许某条具体命令
- 允许某类命令前缀
- 允许所有 Bash

通常不要一上来就全局允许 Bash。更好的方式是从高频低风险命令开始放行。

例如先允许：

```text
Bash(git status)
Bash(git diff:*)
Bash(pnpm test:*)
Bash(pnpm lint)
```

而不是直接允许所有 shell 命令。

原因很简单：Bash 能做的事太多，权限一旦放宽，很难靠提示词兜底。

## 六、`/permissions` 是日常入口
`/permissions` 比直接手改 JSON 更适合日常查看和调整。

它适合用来：
- 看当前 mode
- 看已有 allow、ask、deny
- 看规则来自哪个作用域
- 快速调整常用命令的权限

如果你发现 Claude 总是在同一条安全命令上反复问你，不要每次手动点允许，应该把它变成明确 allow。

如果你发现某个危险动作只靠口头提醒阻止，也应该把它变成 deny。

## 七、目录边界是另一层权限
Claude Code 默认主要围绕当前工作目录工作。

如果任务需要访问其他目录，可以通过：
- `/add-dir`
- `--add-dir`
- settings 里的 `additionalDirectories`

但要注意：扩展目录访问范围，不等于自动加载那个目录里的全部配置，也不等于允许任意修改。

它只是告诉 Claude Code：这个路径也在可访问范围里。真正的读写和命令执行，仍然受权限规则和 sandbox 影响。

## 八、sandbox 不是礼貌提醒
sandbox 是系统执行层的限制，不是提示词。

它和 permissions 的区别是：
- permissions 决定 Claude Code 能不能发起一个动作
- sandbox 决定这个动作即使发起了，在系统里还能碰到什么

例如，你允许 Claude 执行测试命令，但 sandbox 仍然可以限制它不能访问工作区外的路径，或者不能使用网络。

越是自动化程度高的场景，越应该重视 sandbox，而不是只靠 allowlist。

## 九、auto mode 的正确用法
auto mode 的价值不是“彻底不用管”，而是减少低风险操作的确认成本。

适合 auto 的场景：
- 你在干净工作区里做常规开发
- 常用命令已经有明确 allow
- 敏感路径已经 deny
- 你愿意让 Claude 连续完成修改和验证

不适合 auto 的场景：
- 仓库里有敏感数据
- 要操作生产环境
- 任务涉及删除、迁移、推送
- 你还没配置基本 deny 规则

一句话：先设边界，再提高自动化。

## 十、团队共享权限怎么配
团队项目里，权限配置不应该只靠个人习惯。

适合写进项目级 settings 的：
- 允许查询状态和 diff
- 允许跑项目标准测试
- 禁止读取敏感文件
- 禁止修改特定目录
- 对发布、部署、迁移保留 ask 或 deny

不适合提交的：
- 某个人的本机路径
- 过于宽松的全局 Bash allow
- 只适合临时任务的权限

团队共享权限的目标不是“最严格”，而是“默认安全，并且减少无意义打断”。

## 十一、最小可用权限策略
如果你不知道怎么开始，可以按这个顺序：

1. 默认用 `default` 或较保守的自动接受编辑模式
2. allow 常用只读命令和测试命令
3. deny `.env`、secrets、生产配置
4. 对安装依赖、外网访问、删除、推送保留 ask
5. 自动化场景再配 sandbox

先小范围放行，再按实际打断点补 allow。不要反过来一开始就全放开。

## 十二、和前后章节的关系
- 第 7 章讲 settings 的作用域
- 第 8 章讲 settings 里最关键的安全边界
- 第 9 章会讲如何把权限、验证、上下文管理组合成稳定习惯

## 学完标准
- 你知道 mode、rules、目录边界、sandbox 是四层不同控制
- 你不会把 auto mode 当成无条件放行
- 你能把高频低风险动作 allow，把高风险动作 ask 或 deny
- 你知道权限配置应该先设边界，再谈自动化
