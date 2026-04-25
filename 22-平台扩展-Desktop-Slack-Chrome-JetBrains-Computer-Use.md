# 22. 平台扩展：Desktop、Slack、Chrome、JetBrains、Computer Use

## 重要程度
**C 级** - 扩展项，按需学。它们不是 CLI 主线，但会决定 Claude Code 在不同工作环境里的可达边界。

## 学习目标
- 对 Claude Code 的外围平台和集成生态建立完整地图
- 理解不同平台表面的价值，不把它们混成“只是换个 UI”
- 有需要时知道该深入哪一块，而不是盲目全学

## 学什么
- Desktop
- Slack
- Chrome
- JetBrains IDEs
- Computer use
- 平台选择与集成边界

## 你需要掌握
- 官方当前已经把这些能力收进 `Platforms and integrations` 体系
- 它们共享 Claude Code 引擎，但解决的是 **不同场景的工作方式问题**
- 有的偏“表面入口”，有的偏“新能力扩展”，有的偏“团队协作接口”
- 不是每个团队都需要全套，但知道边界很重要

## 一、这章最该先建立的总图
截至 **2026-04-25**，官方的 `Platforms and integrations` 页面已经把生态分得很清楚。

可以把本章对象粗分成三类：

### 平台表面
- Desktop
- JetBrains

### 集成入口
- Slack
- Chrome

### 能力扩展
- Computer use

这三类别混在一起。它们不只是“入口不同”，而是能力模型也不同。

## 二、Desktop：不是换壳终端，而是更强的可视化操作面
官方当前对 Desktop 的定位很清楚：
- visual review
- parallel sessions
- managed setup
- 以及 computer use、Dispatch 等图形化能力

所以 Desktop 的核心价值不是：
- “不用命令行了”

而是：
- 图形化 diff review
- 任务列表和会话管理
- schedule 管理
- 更强的并行和可视操作体验

### 什么时候值得用 Desktop
- 你想少和终端纠缠
- 你更在意可视化 review
- 你要用 Desktop scheduled tasks
- 你要用 Desktop 上的 computer use / Dispatch

## 三、Desktop 和 CLI 的关系
最推荐的理解仍然是：
- CLI 是功能最完整的原生执行面
- Desktop 是可视化和管理体验更强的表面

也就是说：

**Desktop 不是“替代 CLI”，而是为另一类工作方式做优化。**

## 四、Slack：不是聊天机器人，而是团队对 Claude Code 的入口
官方当前把 `Claude Code in Slack` 定义为：

**当你在 Slack 里 `@Claude` 一个 coding task 时，系统会自动把它路由到 Claude Code on the web。**

这很关键，因为它说明：
- Slack 本身不是执行环境
- 真正干活的仍是 Claude Code session
- Slack 只是团队协作入口和通知面

## 五、Slack 最适合什么
官方当前列出的典型场景包括：
- bug 调查和修复
- 快速代码修改
- 团队讨论中的协作调试
- 并行委派任务

它的高价值点是：

**很多编码任务最早并不是在 IDE 或终端里出现的，而是在团队沟通里冒出来的。**

Slack 就是把“群里提一句”直接变成 Claude Code session。

## 六、Slack 当前的关键约束
官方当前明确说明：
- 需要 Claude app in Slack
- 需要 Claude Code on the web 已配置
- 需要 GitHub 已连接
- 主要在 channel / thread 中工作，不是 DM 主场景

还支持 routing mode：
- Code only
- Code + Chat

这意味着 Slack 更多是：
- 团队级分发入口
- 而不是个人深度编码主界面

## 七、Chrome：这是浏览器自动化能力，不是普通插件联动
官方当前对 `Use Claude Code with Chrome (beta)` 的定位非常明确：

**把 Claude Code 接到你的浏览器，让它能直接测试 Web app、看 console、自动填表、抽取页面数据。**

这不是简单“打开网页”，而是：
- 浏览器操作
- 登录态复用
- 页面读取
- 自动化测试和调试

所以 Chrome 的真正价值是：

**把 Web 应用调试和浏览器交互纳入 Claude Code 工作流。**

## 八、Chrome 最适合什么
### 1. 测试 web app

### 2. 看浏览器控制台错误

### 3. 自动化表单和页面流程

### 4. 从网页抽数据

### 5. 与本地编码任务串起来
例如：
- 写代码
- 启服务
- 打开页面
- 点流程
- 看错误
- 修代码

这是非常有价值的闭环。

## 九、Chrome 的关键限制
截至 **2026-04-25**，官方当前说明：
- Chrome 集成仍是 beta
- 需要 `Claude in Chrome` 扩展
- 需要 Claude Code `v2.0.73+`
- 需要 direct Anthropic plan
- 不支持 WSL
- 当前支持 Google Chrome 和 Microsoft Edge
- Brave、Arc 等并不在支持范围内

所以如果浏览器自动化没起来，先看环境前提，不要先怀疑任务描述。

## 十、JetBrains：和 VS Code 很像，但面向另一批 IDE 用户
官方当前的 JetBrains 集成能力与 VS Code 非常接近，核心包括：
- quick launch
- diff viewing
- selection context
- file reference shortcuts
- diagnostic sharing

支持的 IDE 包括：
- IntelliJ IDEA
- PyCharm
- Android Studio
- WebStorm
- PhpStorm
- GoLand

所以 JetBrains 这一块的重点不是“新能力”，而是：

**让 JetBrains 用户也获得和 VS Code 类似的深度集成体验。**

## 十一、JetBrains 的价值点
### 1. 选区共享

### 2. IDE diff viewer

### 3. 诊断共享

### 4. 快捷启动

### 5. 与编辑器环境一致的上下文

也就是说，它解决的仍然是：
- 减少信息搬运
- 提升局部协作效率

## 十二、Computer Use：这是真正新增的一类能力
和 Desktop、JetBrains、Slack 不同，Computer Use 不是“换个平台”，而是让 Claude 获得一类新的执行方式：

**直接看到你的屏幕，并操作 GUI。**

官方当前把它描述为：
- 打开 app
- 点击
- 输入
- 滚动
- 截图

这不是传统 CLI 能力的延伸，而是全新的交互层。

## 十三、Computer Use 最适合什么
官方当前列出的场景很典型：
- 测试 native apps
- 调试视觉 / 布局问题
- 驱动没有 CLI 或 API 的工具
- 操作模拟器或 GUI-only 应用

这类任务以前 Claude 很难完整闭环，因为：
- 终端看不到界面
- 浏览器自动化也覆盖不了原生 GUI

Computer Use 就是补这个缺口。

## 十四、Computer Use 当前的关键限制
截至 **2026-04-25**，CLI 里的 Computer Use 官方当前说明：
- 是 research preview
- 仅限 macOS
- 需要 Pro 或 Max
- 不支持 Team / Enterprise
- 需要交互 session，不能用 `-p`
- 通过 built-in MCP server `computer-use` 启用

而 Desktop 里的 computer use 另有自己的支持范围与设置方式。

这意味着你在知识上要把它区分成：
- CLI computer use
- Desktop computer use

虽然都是同一能力族，但支持范围并不完全一样。

## 十五、为什么 Computer Use 不是默认首选
官方明确说了：
- MCP 更精确时先用 MCP
- shell 能做时先用 Bash
- 浏览器任务有 Chrome 时先用 Chrome
- 都不适合时才轮到 computer use

这很合理，因为 Computer Use：
- 最广
- 也最慢
- 信任边界最特殊

所以正确理解不是“终于有最强工具”，而是：

**当其他更精确的接口都不适用时，computer use 兜底。**

## 十六、如何给这些平台能力做选型
推荐用这张快速判断表。

### 想要更强的图形化会话管理
看 Desktop

### 团队希望从聊天里直接派发 coding task
看 Slack

### 需要浏览器自动化和 web app 调试
看 Chrome

### 团队主力 IDE 是 IntelliJ / PyCharm / WebStorm
看 JetBrains

### 需要操作 GUI-only 应用或看真实屏幕
看 Computer Use

## 十七、这一章真正要建立的视角
学到最后一章，你应该已经能把 Claude Code 看成：
- 一个 CLI 工具
- 一个 IDE 协作层
- 一个云端执行面
- 一组自动化机制
- 一套平台与集成生态

而不是“终端里那个会改代码的聊天助手”。

这也是整套知识库最后一章最想完成的认知升级。

## 十八、课程收束：整套能力怎么串起来
如果把 1 到 22 章压缩成一句话：

Claude Code 的真正价值，不是会写几行代码，而是你能否把它放进：
- 正确的上下文
- 正确的权限边界
- 正确的工作流
- 正确的自动化机制
- 正确的平台入口

当这些都配对了，它才会真正稳定、高效、可扩展。

## 学完标准
- 你知道 Desktop、Slack、Chrome、JetBrains、Computer Use 各自解决什么问题
- 你不会再把“平台表面”“集成入口”“能力扩展”混为一谈
- 你知道哪些是主线高频能力，哪些是按需扩展项
- 你已经对 Claude Code 整体生态有完整地图
