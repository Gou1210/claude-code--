# 21. GitHub、CI/CD 与企业集成

## 重要程度
**B 级** - 有场景时重点学。如果你只做个人开发，到这里已经够用；但如果你想让 Claude Code 真正进入团队软件交付链路，这一章就是关键。

## 学习目标
- 理解 Claude Code 在 GitHub、CI/CD、Code Review、GHES 场景里的角色
- 学会区分“Anthropic 托管能力”和“你自己 CI 里跑 Claude”
- 知道哪些团队能力值得接入，哪些要按成本和治理谨慎评估

## 学什么
- Code Review
- GitHub Actions
- GitHub Enterprise Server
- `@claude review`
- `@claude review once`
- `/install-github-app`
- review customization with `CLAUDE.md` / `REVIEW.md`

## 你需要掌握
- 截至 **2026-04-25**，Code Review 处于 **research preview**，面向 Team / Enterprise
- Code Review 在 Anthropic 基础设施上运行，会用多代理并行分析 PR
- GitHub Actions 让你在 **自己的 GitHub runner** 里运行 Claude Code
- GitHub Enterprise Server 支持 Team / Enterprise 计划，覆盖 web sessions、Code Review、teleport、plugin marketplaces 等能力

## 一、这章最重要的区分：托管审查 vs 自己 CI 里跑 Claude
很多人会把所有 GitHub 集成都统称为“Claude 帮我审 PR”。

但官方当前其实有两条很不同的路线。

### 路线 1：Code Review
- Anthropic 托管
- 自动分析 PR
- 直接在 PR 上发 inline comments

### 路线 2：GitHub Actions
- 在你的 GitHub Actions workflow 里运行 Claude
- 你自己定义触发条件和行为
- 更灵活，也更工程化

一句话：

**Code Review 更像托管审查服务，GitHub Actions 更像你自己编排的 CI automation。**

## 二、Code Review 到底是什么
官方当前把 Code Review 定义为：

**用一组专门 agent 对 PR diff 和相关代码上下文做并行分析，然后把发现的问题直接评论到 PR 对应行上。**

其核心特点有四个：
- 多代理并行分析
- 结合整个代码库上下文
- 结果按严重性排序
- 以内联评论形式返回

这不是简单的：
- 看 diff
- 然后生成一段总评

而是更接近一个自动 reviewer。

## 三、Code Review 适合解决什么问题
官方当前强调的重点是：
- logic errors
- security vulnerabilities
- broken edge cases
- subtle regressions

也就是说，它的目标不是替代 lint，而是抓：
- 更语义化
- 更上下文相关
- 更难被静态规则直接命中的问题

## 四、Code Review 怎么触发
官方当前有几种模式。

### 1. PR 创建后自动跑一次

### 2. 每次 push 都跑

### 3. Manual
只在你显式触发时运行。

此外还支持两个评论命令：

### `@claude review`
启动 review，并让 PR 后续 push 继续被 review。

### `@claude review once`
只跑这一次，不订阅后续 push。

这两个命令的差别非常实用，尤其在高频 push 的长 PR 上。

## 五、Code Review 为什么值得配 `REVIEW.md`
官方当前明确说明：
- `CLAUDE.md` 适合全局项目规则
- `REVIEW.md` 适合 review-only 规则

所以：

### 用 `CLAUDE.md`
当规则也适用于日常 Claude Code 会话。

### 用 `REVIEW.md`
当规则只用于 code review，不希望污染通用会话。

例如：
- 新 API 必须有 integration test
- 不评论生成代码格式
- 某些 lock 文件只忽略格式问题

这会明显提升 review 结果的实用性和噪音控制。

## 六、Code Review 的成本和治理要一起考虑
官方当前写得很明确：
- Code Review 按 token usage 计费
- 每次 review 平均大约 `$15-25`
- review 触发频率会直接放大成本

这对团队来说意味着：

### 1. 不是所有仓库都该开最高频模式

### 2. 高频 push 仓库要谨慎选“每次 push 都审”

### 3. Manual 模式在高流量仓库里很有价值

所以这类功能不是“越多越好”，而是要：
- 看 PR 规模
- 看仓库价值
- 看成本承受能力

## 七、GitHub Actions 的定位完全不同
官方当前的 `Claude Code GitHub Actions` 文档定位很清晰：

**在 GitHub workflow 里运行 Claude Code，构建你自己的自动化流程。**

这意味着它可以用来：
- 根据 issue 生成 PR
- 在 PR 里根据 `@claude` 提及实施修改
- 定时维护
- 自定义代码处理流程

它的关键价值不是“自动 review”，而是：

**把 Claude 变成你 CI/CD 中可编排的一步。**

## 八、GitHub Actions 最适合什么
### 1. 自定义流程
你想要自己的触发条件和逻辑。

### 2. 在 GitHub runner 上执行
官方明确说：
- 代码会留在 GitHub 的 runner 上

### 3. 结合 issue / PR / workflow 事件

### 4. 团队已经很依赖 GitHub Actions

如果你的团队本来就把很多自动化都放在 GitHub Actions，那么把 Claude 接进去会更自然。

## 九、GitHub Actions 的典型能力
官方当前强调的典型场景包括：
- 通过 `@claude` mention 创建 PR
- 实现 feature
- 修 bug
- 分析代码

并且会遵循：
- `CLAUDE.md`
- 项目现有模式

所以 GitHub Actions 不是简单“调一个模型”，而是可以跑完整 Claude Code 工作流。

## 十、Code Review 和 GitHub Actions 怎么选
推荐用这个判断：

### 如果你要
- 自动给 PR 提发现
- 评论到具体行
- 减少人工初筛

优先看 Code Review。

### 如果你要
- 自定义 workflow
- 让 Claude 在 CI 里执行任务
- 和你现有自动化深度结合

优先看 GitHub Actions。

两者也不是互斥，完全可以同时存在。

## 十一、GitHub Enterprise Server 为什么单独成章
因为对企业团队来说，GHES 不是“小兼容项”，而是整个能不能接入的前提。

截至 **2026-04-25**，官方说明：
- GHES 支持 Team / Enterprise
- 支持 web sessions、Code Review、Teleport、plugin marketplaces、contribution metrics

这意味着：

**Claude Code 已经不只适配 github.com，也开始覆盖自托管企业 GitHub 场景。**

## 十二、GHES 当前有哪些关键差异
官方当前特别指出几项差别。

### 支持
- Web
- Code Review
- Teleport
- Plugin marketplaces
- GitHub Actions

### 不支持
- GitHub MCP server

官方建议对 GHES 场景改用：
- `gh` CLI，并指定对应 hostname

这点非常重要，因为很多企业用户会天然假设 GitHub MCP 一样可用。

## 十三、企业场景里，权限和治理比“会不会用”更重要
一旦进入团队或企业环境，你要考虑的就不只是：
- Claude 能不能跑

而是：
- 谁能触发
- 哪些仓库能用
- 审查频率怎么配
- 成本怎么控
- 哪些 GitHub App 权限可接受
- 哪些 marketplace 能安装

所以企业集成的重点不是 prompt，而是治理。

## 十四、和前面章节其实是怎么串起来的
前面你已经学了：
- permissions
- hooks
- skills
- subagents
- plugins
- web
- routines

到了 GitHub / CI / Enterprise 这里，本质上是在问：

**这些能力如何进入团队协作和正式交付链路。**

也就是说，个人效率工具开始变成团队工程系统的一部分。

## 十五、一个很实用的接入顺序
如果团队想逐步尝试，最稳的顺序通常是：

1. 先在本地个人开发里稳定使用 Claude Code
2. 再试 GitHub Actions 上的小自动化
3. 再试 Code Review 在部分仓库上线
4. 最后再进入 GHES / 企业级治理和 marketplace 分发

这样阻力最小，也最容易积累信任。

## 十六、这一章真正要建立的认知
学完这章后，你不该再把 Claude Code 只看成一个本地编码助手。

而应该理解成：
- 它可以是 PR reviewer
- 可以是 CI worker
- 可以是 issue-to-PR 自动化节点
- 可以是企业 GitHub 生态的一部分

这就是“进入软件交付系统”的那一步。

## 十七、和前后章节的关系
- 第 20 章讲的是配置调试与排障
- 第 21 章讲的是 GitHub、CI/CD 与企业链路
- 第 22 章会收束到外围平台扩展：Desktop、Slack、Chrome、JetBrains、Computer Use

## 学完标准
- 你知道 Code Review 和 GitHub Actions 的根本差别
- 你知道 `@claude review` 与 `@claude review once` 的差别
- 你知道 `CLAUDE.md` 和 `REVIEW.md` 在 review 场景里的分工
- 你知道 GHES 当前支持哪些能力、限制哪些能力
