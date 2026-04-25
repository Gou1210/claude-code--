# 8. 权限体系：Permission Modes、Allow/Ask/Deny、Sandboxing 与 Auto

## 重要程度
**S 级** - 必须掌握。Claude Code 的真正可用性，不取决于它“会不会写代码”，而取决于你能不能把权限边界设到既安全又不烦。

## 学习目标
- 理解 Claude Code 的权限控制是分层的，不是一个总开关
- 掌握 permission mode、permission rules、working directories、sandboxing 各自控制什么
- 能根据任务风险选择合适的默认模式

## 先纠正一个常见误区
很多人会把“权限模式”理解成：
- normal 比较保守
- auto 比较激进
- plan 不能改文件

这只是表层现象。

真正应该建立的理解是：Claude Code 的权限体系至少有四层控制面。

### 第一层：模式
决定“遇到新操作时大体怎么处理”。

### 第二层：规则
决定“哪些工具或命令可以直接过、必须问、或者直接拒绝”。

### 第三层：目录边界
决定“Claude 默认能接触哪些路径”。

### 第四层：sandbox
决定“即使允许执行 Bash，它在系统层面还能碰到什么”。

只盯着 mode，而不看其他三层，基本一定会配错。

## 学什么
- `/permissions`
- `permissions.allow / ask / deny`
- `defaultMode`
- `additionalDirectories`
- permission rule 语法
- sandboxing
- auto / auto-accept / acceptEdits / dontAsk / bypassPermissions 等模式差异

## 你需要掌握
- Claude Code 的读文件、写文件、执行命令、用 MCP / Web 工具，不是一个统一授权点
- 规则匹配顺序是 `deny -> ask -> allow`
- `Bash` 和 `Bash(*)` 不是一回事
- `additionalDirectories` 扩的是工作目录访问范围，不等于自动加载这些目录里的全部 `.claude/` 配置
- sandbox 是系统级隔离，不是提示词层的“温柔提醒”

## 一、先看官方给出的权限基本面
截至 **2026-04-25**，官方文档把 Claude Code 的权限系统描述成“在能力和安全之间做平衡的分层机制”。

其中最基础的分类是：
- 只读操作：通常不需要审批
- Bash 命令：通常需要审批
- 文件修改：通常需要审批

这说明一个关键事实：

**Claude Code 默认不是“什么都能直接做”，而是对高风险动作有显式权限门槛。**

## 二、permission mode 管的到底是什么
mode 解决的是：

**当 Claude 遇到一个“当前没有现成规则覆盖”的动作时，默认怎么处理。**

常见模式可以这样理解：

### `default`
标准模式。第一次遇到相关工具或高风险动作时，按默认策略询问。

### `acceptEdits`
更偏向“自动接受文件修改”，减少写文件阶段的审批打断。

### `plan`
分析模式。Claude 可以看、可以想，但不能改文件，也不能执行命令。

### `dontAsk`
没有被预先允许的动作，倾向于直接拒绝，而不是不停弹确认。

### `bypassPermissions`
跳过权限提示。只能在你明确知道环境安全时用。

另外，官方文档和不同界面里，你还可能看到：
- `auto`
- Auto-Accept Mode
- normal / plan / auto-accept 这类交互标签

不同版本和界面命名可能略有差异。实际以你当前版本的 `/permissions`、`/config` 和 `claude --help` 为准，不要只记旧截图。

## 三、真正决定细粒度行为的是 allow / ask / deny
mode 只是总倾向，规则才是精细控制。

Claude Code 的权限规则主要有三类：

### `allow`
命中的动作可以直接执行。

### `ask`
命中的动作每次都问。

### `deny`
命中的动作直接拒绝。

而且规则顺序不是随便的，官方明确说明：

**匹配顺序是 `deny -> ask -> allow`，第一个命中的规则生效。**

这意味着：
- `deny` 永远优先
- 不要以为后面补一个 `allow` 就能覆盖前面的 `deny`

## 四、permission rule 语法要点
规则格式本质上是：

```text
Tool
Tool(specifier)
```

例如：
- `Bash`
- `Bash(npm run test:*)`
- `Read(./.env)`
- `WebFetch(domain:example.com)`

其中最容易踩坑的是 Bash。

### `Bash` 不等于 `Bash(*)`
官方文档特别强调：
- `Bash` 才表示“所有 Bash 命令”
- `Bash(*)` 并不表示全部 Bash

如果你把这俩搞混，权限会和你预期完全不一样。

### `:*` 和 `*` 也不是一回事
例如：
- `Bash(ls:*)` 更像“以 `ls` 这个词开头的前缀匹配”
- `Bash(ls*)` 则更宽，甚至可能匹配到你没预期到的命令

权限规则不是正则，也不是自然语言。写之前一定要按官方语义理解。

## 五、最常见也最实用的权限设计思路
### 1. 对高频低风险命令做 allow
例如：
- `npm run lint`
- `npm run test:*`
- `git diff`
- `git status`

这样能明显减少打断。

### 2. 对危险外部操作做 ask 或 deny
例如：
- `git push`
- `curl`
- 读取 `.env`
- 读取 `secrets/**`

### 3. 不要一上来就全局 `Bash`
除非你非常确定当前环境就是给 Claude 全自动跑的安全沙箱，否则这通常过于宽松。

## 六、`/permissions` 是日常真正该用的入口
如果说 `/config` 是通用配置入口，那 `/permissions` 就是权限系统的专门控制台。

它的价值在于：
- 查看当前已有规则
- 看规则来自哪个 settings 文件
- 直接调整 allow / ask / deny
- 检查当前默认 mode

这比只盯着某个 JSON 文件要直观得多。

## 七、working directory 是权限边界的一部分
默认情况下，Claude 只对启动目录及其子目录天然有访问权。

如果你要扩展访问范围，可以用：
- 启动参数 `--add-dir`
- 会话里的 `/add-dir`
- settings 里的 `additionalDirectories`

但要注意两点：

### 1. 这扩的是文件访问范围
不是“自动把所有配置、memory、rules 都一起带上”。

官方文档明确提到：额外目录里的大部分 `.claude/` 配置默认并不会自动发现。

### 2. 额外目录也沿用当前权限体系
它不是一个“完全独立的新世界”。

文件可读范围扩大后，编辑权限仍然要遵循当前 mode 与 permission rules。

## 八、sandboxing 管的是系统层安全，不是聊天层礼貌
这是很多人对 sandbox 的误解。

sandbox 不是告诉 Claude “尽量别乱来”，而是直接在执行层隔离 Bash 能碰到的文件系统和网络。

所以它和 permission rules 的关系是：

- permission rules：决定“Claude 被不被允许发起某种动作”
- sandbox：决定“即使这个动作被允许，它在系统里还能碰到什么”

这两层是互补的，不是替代关系。

## 九、auto / auto-accept 的正确理解
很多人一看到 auto，就以为是“完全自动化、彻底不问”。

这往往是错的。

更准确的理解应该是：
- 它是在默认模式层面尽量减少不必要打断
- 但仍可能受 `deny`、`ask`、managed settings、sandbox 等约束
- 它不是“越权模式”

所以不要把 auto 理解成：
- 无条件放行一切命令
- 等同于 `bypassPermissions`

真正彻底危险的是后者。

## 十、这章最容易混淆的几个点
### `plan` vs `dontAsk`
- `plan` 是能力受限：不能改、不能执行
- `dontAsk` 是审批策略受限：没预先允许就拒绝

### `acceptEdits` vs `bypassPermissions`
- 前者主要是减少编辑相关打断
- 后者是整体跳过权限提示，风险高很多

### `allow` vs `additionalDirectories`
- `allow` 管工具/动作
- `additionalDirectories` 管路径范围

### permission rules vs sandbox
- 前者是 Claude Code 自己的权限层
- 后者是系统执行层隔离

## 十一、建议这样给自己配一套“实用但不冒进”的默认策略
### 对个人日常开发
- 默认模式用 `default` 或偏保守的自动接受编辑模式
- 常用低风险命令做 allow
- 对外网访问、密钥文件、推送命令保留 ask 或 deny

### 对团队共享仓库
- 把敏感路径 deny 写进项目级 settings
- 把常用测试 / lint / git 查询命令做共享 allow
- 不要把过于宽松的策略直接提交到仓库

### 对 CI / 安全沙箱
- 才考虑更激进的自动化模式
- 同时配合 sandbox 与 managed settings

## 十二、建议你在脑子里形成的“权限四层图”
最推荐的理解顺序是：

1. mode 决定默认倾向
2. allow / ask / deny 决定细粒度规则
3. working directory / additionalDirectories 决定路径边界
4. sandbox 决定系统执行边界

只学其中一层，都会产生错觉；四层一起看，才知道 Claude 到底“能做什么、会不会问、就算能做还能碰到哪里”。

## 十三、和前后章节的关系
- 第 6 章讲的是长期提示与记忆
- 第 7 章讲的是硬配置与作用域
- 第 8 章在这基础上解决“行动边界与安全边界”
- 第 9 章会进一步讲如何把这些权限机制变成高效、低打断的日常实践

## 学完标准
- 你知道 mode、rules、directories、sandbox 是四层不同控制面
- 你知道 `deny -> ask -> allow` 的匹配顺序
- 你不会再把 `Bash` 和 `Bash(*)` 混为一谈
- 你能给自己的项目配出一套既安全又不频繁打断的权限策略
