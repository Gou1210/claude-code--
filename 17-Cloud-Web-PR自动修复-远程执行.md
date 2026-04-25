# 17. Cloud / Web / PR 自动修复 / 远程执行

## 重要程度
**B 级** - 有场景时重点学。这一章不是 CLI 主线必修，但只要你开始做长任务、离线运行、PR 持续修复或跨设备协作，它就会非常关键。

## 学习目标
- 理解 Claude Code on the web、Remote Control、`/autofix-pr` 分别是什么
- 学会判断一项任务应该跑在本地、云端，还是“本地运行但远程操控”
- 建立对云端 session、PR 自动修复、跨设备接力的整体认知

## 学什么
- Claude Code on the web
- `/web-setup`
- `/autofix-pr`
- `claude --remote`
- `claude --teleport`
- Remote Control
- 移动端 / 浏览器接力

## 你需要掌握
- 截至 **2026-04-25**，Claude Code on the web 运行在 **Anthropic 托管云环境**
- 浏览器关掉后，web session 仍可继续运行
- `/autofix-pr` 会为当前 PR 启动一个 web session，自动响应 CI 失败和 review comments
- Remote Control 看起来也在网页里，但本质上仍然 **跑在你本机**

## 一、这一章最重要的区分：Web 不是 Remote Control
很多人第一次接触时，容易把这两件事混为一谈，因为它们都可能在：
- `claude.ai/code`
- 或手机 App

里出现。

但两者本质完全不同。

### Claude Code on the web
- 任务跑在 Anthropic 云端
- 仓库在云端 fresh clone
- 浏览器关掉后还能继续

### Remote Control
- 任务仍跑在你的本机
- 网页或手机只是远程窗口
- 你的本地文件、MCP、工具链都还在本机上

一句话记忆：

**Web 是“把运行位置搬到云端”，Remote Control 是“把操控界面搬到远端”。**

## 二、为什么 Claude Code on the web 值得学
官方当前把 web 定位得非常明确：

**适合那些不需要你持续盯着、但希望继续运行的任务。**

比如：
- 长时间修 PR
- 大范围探索代码库
- 跑一串复杂分析
- 你离开电脑后还想让它继续

这类任务如果都绑在本地，会遇到几个常见问题：
- 电脑要一直开着
- 终端不能随便关
- 网络中断会打断
- 你离开工位后不方便接力

而 web 正是为这些问题设计的。

## 三、Claude Code on the web 当前的官方状态
截至 **2026-04-25**，官方页面说明：
- Claude Code on the web 是 **research preview**
- 面向 Pro、Max、Team，以及符合条件的 Enterprise 用户开放
- 入口是 `claude.ai/code`

官方还明确提到：
- session 会持续运行，即使你关闭浏览器
- 也能从 Claude 移动端 App 监控这些 session

这意味着它已经不是“仅仅能在网页里聊”，而是一个真正的云端执行面。

## 四、云端 session 到底怎么工作
官方当前的云端模型大致可以理解成：

1. 你给 Claude 一个任务
2. 云端环境 clone 代码仓库
3. Claude 在 Anthropic 托管环境中运行
4. 改动推到分支
5. 你回来看结果、review diff、继续对话

这里最关键的两个现实差异是：

### 1. 不是你的本地工作区
它操作的是云端环境里的仓库副本。

### 2. 不是永久活环境
有环境生命周期和限制，不是“永远开着的一台机器”。

所以你应该把 web 看成：
- 云端任务执行环境
- 而不是远程登录你自己的电脑

## 五、连接 GitHub 的两种方式
官方当前给出两种主要 GitHub 授权方式。

### 1. GitHub App
适合：
- 团队
- 需要按仓库授权
- 需要 Auto-fix

### 2. `/web-setup`
适合：
- 个人开发者
- 已经在本地使用 `gh`
- 想快速把本地 `gh` token 同步到 Claude 账户

官方明确说明：
- `/schedule` 会检查你是否完成过其中一种授权
- Auto-fix 必须依赖 GitHub App，因为它要接 GitHub webhook

## 六、`/autofix-pr` 到底是什么
这是 web 工作流里最有代表性的能力之一。

官方当前定义是：

**Claude 订阅一个 PR 的 GitHub 活动，在 CI 失败或 reviewer 留言时自动调查并修复。**

也就是：
- PR 上有人提 comment
- CI 红了
- Claude 会自己跟进

这已经不是“我手动叫它修一下”，而是更像一个持续盯着 PR 的自动 worker。

## 七、如何启动 Auto-fix
官方当前列出几种方式。

### 1. 从 web 上创建的 PR
在 CI 状态栏里直接开启 Auto-fix。

### 2. 从终端里
在 PR 对应分支运行：

```text
/autofix-pr
```

Claude 会：
- 用 `gh` 检测当前 open PR
- 启动一个 web session
- 开启 auto-fix

### 3. 从移动端
你也可以直接要求 Claude 去 watch 某个 PR 并处理问题。

### 4. 粘贴任意现有 PR URL
也可以让 Claude 对该 PR 启用 auto-fix。

## 八、Auto-fix 最适合什么任务
### 1. CI 红灯修复
最标准的场景。

### 2. reviewer comment 响应
尤其适合明确、局部的问题。

### 3. 需要持续盯 PR 一段时间
比如你不想每次都重新回来手动触发。

### 4. 你已经离开电脑
仍希望 PR 有人盯着。

## 九、什么时候不要用 Auto-fix
虽然很强，但也不是所有 PR 都适合。

不太适合的情况：
- 需求还没定
- comment 很主观、需要大量产品判断
- 改动牵涉高风险架构决策
- 你根本还没信任这个仓库的自动流程

一句话：

**Auto-fix 适合“明确反馈 -> 自动跟进”，不适合“高度模糊 -> 需要大量人工判断”。**

## 十、`--remote` 和 `--teleport`：云端与本地如何接力
官方当前明确支持：
- `claude --remote`
- `claude --teleport`

### `--remote`
从终端直接发起一个云端 session。

### `--teleport`
把某个 web session 拉回本地终端继续。

这两个能力合在一起的意义非常大：

**你不必在“只能本地”或“只能云端”之间二选一，可以中途切换。**

## 十一、什么时候适合把任务放到 web
### 适合放到 web
- 长时间运行
- 不依赖本地未提交文件
- 你想关电脑还继续跑
- 你希望从手机或浏览器跟进
- 任务天然围绕 GitHub PR

### 不太适合放到 web
- 强依赖本地环境
- 强依赖本地 MCP / 本地服务
- 需要访问你本机私有文件
- 需要用你本地已经运行的开发进程

## 十二、Remote Control：真正的价值不是“远程聊天”
官方对 Remote Control 的定位很实用：

**继续一个已经在本地运行的 Claude Code session，但从别的设备接力。**

它适合：
- 你从电脑前离开
- 但又不想把工作搬到云端
- 同时还想保留本地工具链、MCP 和文件系统

这和 web 的差别非常关键：
- web 是新环境
- Remote Control 是同一个本地环境

## 十三、Remote Control 怎么启动
官方当前支持几种形式。

### CLI server mode
```bash
claude remote-control
```

### 交互 session 内
- 通过 `/remote-control`

### VS Code
- 也支持从扩展里启动

启动后，你会得到：
- 一个 session URL
- 还可以显示二维码，方便手机接入

## 十四、Remote Control 的核心优点
### 1. 本地环境原封不动
- 本地文件系统
- 本地工具
- 本地 MCP
- 本地配置

### 2. 多端同步
终端、网页、手机之间会话保持同步。

### 3. 中断后可自动恢复
机器睡眠或网络短断后，会尝试自动重连。

所以 Remote Control 最适合“我已经在本地干到一半了，但我人暂时不在这台机器前”。

## 十五、Web 和 Remote Control 的选型标准
最实用的判断方式是：

### 用 Web
- 任务不依赖本地环境
- 想要云端持续执行
- 想关电脑也继续
- 想围绕 GitHub 自动化

### 用 Remote Control
- 任务强依赖本地环境
- 本地有特殊工具 / MCP / 文件
- 你只是不在电脑前
- 你想继续同一个 local session

## 十六、Cloud session 的限制意味着什么
官方 web 页面明确讲了 limitation、security 和 environment lifecycle。

这对你的实际含义是：

### 1. 别把它当成你自己的持久主机
它是托管执行环境，不是你完全控制的一台远程机器。

### 2. 对本地状态不要想当然
云端 clone 是 fresh 的，不会自动拥有你本地未提交改动。

### 3. 网络与外部访问是有边界的
需要关注 network access、setup scripts、Docker 等配置能力。

## 十七、这一章真正要建立的认知升级
学完这章后，你对 Claude Code 的理解不该再停留在：
- 一个在终端里聊天和改文件的工具

而应该升级成：
- 本地 CLI
- IDE
- 云端 web
- 手机远程接力
- PR 自动修复

这一整套多执行面系统。

## 十八、和前后章节的关系
- 第 16 章讲的是 IDE 深度集成
- 第 17 章讲的是 Web / Cloud / Remote Control / Auto-fix
- 第 18 章会继续讲自动化：Routines、Scheduled Tasks 与周期性运行

## 学完标准
- 你知道 Claude Code on the web 和 Remote Control 的根本差别
- 你知道 `/autofix-pr` 适合什么、不适合什么
- 你知道什么时候该本地跑，什么时候该云端跑
- 你知道 `--remote`、`--teleport`、`/web-setup`、`/autofix-pr` 在整套工作流里的位置
