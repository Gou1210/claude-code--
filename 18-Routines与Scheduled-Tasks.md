# 18. Routines 与 Scheduled Tasks

## 重要程度
**B 级** - 有场景时重点学。一旦你想让 Claude 在“你不在场时”继续工作，这一章就会直接决定你该选哪种自动化形态。

## 学习目标
- 理解 `Routines`、Desktop scheduled tasks、`/loop` 三者的差异
- 学会根据任务目标选择“云端自动化”“本地自动化”或“当前 session 内轮询”
- 知道定时任务的运行边界、恢复方式、过期机制和权限特点

## 学什么
- Routines
- `/schedule`
- `/loop`
- session-scoped scheduled tasks
- Desktop scheduled tasks
- schedule / API / GitHub triggers
- cloud vs desktop vs `/loop`

## 你需要掌握
- 截至 **2026-04-25**，Routines 运行在 **Anthropic 托管云基础设施**
- 一个 routine 可以同时绑定多种 trigger：`schedule`、`API`、`GitHub`
- `/loop` 和 cron scheduling tools 适合在 **当前 CLI session** 内重复运行 prompt
- scheduled tasks 需要 Claude Code **v2.1.72 或更高版本**

## 一、这一章最重要的区分：有三种“定时工作”
很多人会把这些能力全混成“自动跑 Claude”。

但官方当前已经把它们区分得很清楚：

### 1. Routines
云端长期自动化。

### 2. Desktop scheduled tasks
本机长期自动化。

### 3. `/loop`
当前 session 内短周期轮询。

一句话记忆：

**Routines 管云端，Desktop tasks 管本机，`/loop` 管当前会话。**

## 二、Routines 到底是什么
截至 **2026-04-25**，官方把 routines 定义为：

**一个保存好的 Claude Code 配置：prompt、一个或多个仓库、以及一组 connectors，之后自动按触发器运行。**

它的关键点有三个：
- 运行在云端
- 可长期存在
- 可被多种 trigger 触发

所以 routine 不是“记住一个 prompt”，而是：

**把一套可自动执行的 Claude 工作流保存成一个长期运行单元。**

## 三、Routine 支持哪些 trigger
官方当前明确列出三类。

### `Scheduled`
按固定时间周期运行。

例如：
- 每小时
- 每晚
- 每周

### `API`
通过 HTTP POST 按需触发。

适合：
- 脚本
- 内部平台
- 外部系统回调

### `GitHub`
响应仓库事件触发。

例如：
- PR
- release
- 其他 repo 事件

官方还特别说明：

**一个 routine 可以同时绑定多种 trigger。**

这点很重要，因为它意味着 routine 不只是“定时任务”，而是真正的自动化工作流入口。

## 四、为什么 Routine 和 `/loop` 不是一回事
这是最高频误解之一。

### `/loop`
- 运行在你当前 session
- 终端关了就没了
- 更像短期轮询

### Routine
- 运行在云端
- 电脑关了也能继续
- 更像长期自动化服务

所以：

**`/loop` 适合“盯一会儿”，routine 适合“长期值班”。**

## 五、官方给出的三种调度方式对比
根据当前官方文档，三种方式可以这样理解。

### Cloud
- 运行地：Anthropic cloud
- 机器是否必须开着：不需要
- 是否跨重启持久：是
- 本地文件访问：否，fresh clone

### Desktop
- 运行地：你的机器
- 机器是否必须开着：需要
- 是否跨重启持久：是
- 本地文件访问：是

### `/loop`
- 运行地：你的机器
- 机器是否必须开着：需要
- 是否必须保持 session：是
- 更适合会话内临时反复检查

## 六、什么时候该用 `/loop`
官方把 `/loop` 放在“快速轮询”位置，非常合理。

适合：
- 盯一个 deploy
- 盯 CI 状态
- 盯某个长任务结果
- 在当前 session 里定时提醒

### 典型例子
```text
/loop 5m 检查 deployment 是否完成并告诉我结果
```

### 不写间隔时会怎样
官方说明：
- Claude 会自己选择下一次检查间隔
- 在某些环境里还可能直接使用 Monitor，而不是简单轮询

### 不写 prompt 时会怎样
会运行内建 maintenance prompt，或者使用你的 `loop.md`

## 七、`loop.md` 的意义
官方当前支持：
- `.claude/loop.md`
- `~/.claude/loop.md`

它会替换 bare `/loop` 的默认 prompt。

这很适合：
- 让 `/loop` 变成你的默认维护机器人
- 例如专门盯 release branch、盯 PR、盯 CI

这个机制的价值在于：

**你不需要每次重新写维护提示词。**

## 八、`/loop` 的边界和限制
官方当前写得很清楚：
- 任务只在 Claude Code 运行且空闲时触发
- 新开 fresh conversation 会清掉 session-scoped tasks
- recurring task 7 天后自动过期
- one-shot task 到时执行后自动删除

所以你不该把 `/loop` 理解成 durable automation，而应该理解成：
- 会话内定时器
- 带恢复能力
- 但不是长期后台服务

## 九、Desktop scheduled tasks 是什么
官方 Desktop 文档当前把它定义为：

**由 Claude Code Desktop 在你的机器上按计划启动的新 session。**

这意味着：
- 不依赖你手动开一个 CLI session 挂着
- 只要 Desktop app 在运行且电脑没睡
- 它就会自动起新任务

这比 `/loop` 更持久，但本质上仍然在你本机。

## 十、Desktop scheduled tasks 最适合什么
### 1. 需要访问本地文件
比如：
- 本地仓库
- 本地脚本
- 本地工具链

### 2. 需要本地 MCP / connectors

### 3. 电脑通常都开着

### 4. 你希望可视化管理和查看历史

官方当前还支持：
- 每个 task 独立权限模式
- review 历史
- 允许和撤销保存的权限

所以 Desktop tasks 更像：

**跑在你电脑上的可管理任务面板。**

## 十一、Desktop tasks 和 `/loop` 的差别
### `/loop`
- 当前会话内
- 更轻
- 更临时
- 适合边聊边盯

### Desktop task
- 独立起新 session
- 更持久
- 更像定时 job
- 更适合每天/每周例行工作

## 十二、Routine 最适合什么
### 1. 电脑不在线也要跑
这是 routine 最核心的价值。

### 2. 需要 GitHub 或 API 触发
这些本来就不该靠本地 session 挂着等。

### 3. 需要云端一致执行环境

### 4. 需要真正的长期自动化

例如：
- 每晚 review 某类 PR
- 有 release 事件就跑回归分析
- 内部平台 POST 一个事件就触发 Claude 处理

## 十三、Routine 和 Web 的关系
Routine 本质上属于 Claude Code on the web 体系。

它的关键属性和 web 一样：
- 云端运行
- fresh clone
- 不依赖你本地机器

所以你可以把它理解成：

**Claude Code on the web 从“人工发起 session”延伸到“自动发起 session”。**

## 十四、定时工作要特别注意 prompt guardrails
官方 Desktop tasks 文档里有一个非常实用的提醒：

**任务可能在你不预期的时点补跑。**

例如：
- 电脑白天睡眠
- 晚上醒来补跑

所以 prompt 最好带 guardrails：
- 只处理今天的内容
- 超过某个时间就只总结，不执行
- 如果 PR 已关闭就退出

这类约束非常重要，否则自动化容易做出“技术上正确、业务上不对”的行为。

## 十五、权限模式在自动化场景里更关键
自动化最怕什么？

不是做不了，而是：
- 一直卡在等待授权
- 或太宽松导致风险过大

官方 Desktop tasks 文档明确提到：
- 每个 task 有自己的 permission mode
- 允许规则会继承相关 settings
- Ask 模式下如果缺权限，任务会停住等你批准

这意味着：

**自动化能不能稳定跑，往往取决于你是否提前把权限配好。**

## 十六、怎么选：Routine、Desktop 还是 `/loop`
推荐用这张判断表：

### 只想在当前会话里暂时盯一下
用 `/loop`

### 需要访问本地文件和工具，并且电脑通常开着
用 Desktop scheduled tasks

### 想离线也继续、想靠 GitHub/API 触发、想真正长期运行
用 Routines

## 十七、这一章真正要形成的自动化观
到这里，你对 Claude Code 的理解应该进一步升级：

不是只有“我在终端里发 prompt，它才工作”，而是开始进入：
- 轮询
- 定时
- 事件触发
- 无人值守

这已经很接近真正的 agent automation 了。

## 十八、和前后章节的关系
- 第 17 章讲的是 Web / Cloud / Remote Control / Auto-fix
- 第 18 章讲的是周期运行和自动触发
- 第 19 章会讲如何安全回滚和恢复：Checkpointing

## 学完标准
- 你知道 Routines、Desktop scheduled tasks、`/loop` 三者的根本差别
- 你知道 routine 可以绑定 schedule、API、GitHub 多种 trigger
- 你知道 `/loop` 适合短期轮询，不适合长期 durable automation
- 你能设计一个 nightly review、一个 API-triggered routine、一个 session 内轮询任务
