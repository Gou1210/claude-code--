# 6. Memory：CLAUDE.md、auto memory 与 `.claude/` 目录

## 重要程度
**S 级** - 必须掌握。Memory 决定 Claude Code 每次进入项目时“先知道什么”。写得好，可以少解释很多背景；写得差，会把上下文塞满，还会不断误导它。

## 学习目标
- 知道什么内容值得长期记住，什么不该常驻
- 分清 `CLAUDE.md`、auto memory、rules、settings、skills 的边界
- 能为一个真实项目写出简洁、可维护的 memory

## 先建立一个场景
你接手一个项目，每次让 Claude Code 改代码，都要重复说明：
- 项目怎么启动
- 测试该跑哪条命令
- 哪些目录不能乱动
- 这个项目不用某个库或某种写法

如果这些话每次都靠聊天补充，问题有三个：
- 容易漏
- 占上下文
- 换一个 session 又要重说

Memory 要解决的不是“让 Claude 永久记住一切”，而是把稳定、重复、对当前项目重要的信息，在 session 开始时自动交给它。

## 一、Memory 不是强制配置
这是最容易误解的地方。

`CLAUDE.md`、auto memory、rules 本质上都是上下文输入。它们能指导 Claude，但不能像权限、hooks、sandbox 那样强制拦截行为。

所以判断标准很简单：
- 想让 Claude 知道：写 memory
- 想让 Claude 必须遵守：写 settings、permissions 或 hooks
- 想让 Claude 按需执行一套流程：写 skill

例如：
- “本项目使用 pnpm，不要用 npm”可以写进 `CLAUDE.md`
- “禁止读取 `.env`”不应该只写进 `CLAUDE.md`，应该放进权限规则
- “发布前按 8 步检查”更适合做成 skill，而不是每次启动都加载

## 二、`CLAUDE.md` 适合放什么
`CLAUDE.md` 是你写给 Claude 的项目说明书。它适合放稳定、长期、每次都可能用到的信息。

最常见的内容是：
- 项目是什么，核心目录在哪里
- 常用开发、测试、构建命令
- 编码风格和命名约定
- 关键业务边界
- 改动后的验证要求
- 明确禁止的低级错误

一个好的 `CLAUDE.md` 不追求全面，而追求“开局有用”。

## 三、真实场景：新项目第一版 memory 怎么写
假设你刚把 Claude Code 接入一个前端项目，不要一上来写长篇背景。先写这几块就够：

```md
# Project Overview
- This is a React admin dashboard.
- Main app code lives in `src/`.
- API client lives in `src/api/`.

# Commands
- Install: `pnpm install`
- Dev: `pnpm dev`
- Typecheck: `pnpm typecheck`
- Test: `pnpm test`

# Coding Rules
- Use existing component patterns in `src/components/`.
- Do not introduce new UI libraries without asking.
- Keep behavior changes covered by tests when possible.

# Verification
- For logic changes, run related tests.
- For UI changes, run typecheck and inspect the affected page.
```

这类 memory 的价值在于：Claude 不需要每次重新问“怎么跑项目”“组件放哪”“改完怎么验”。

## 四、`CLAUDE.md` 不要写成百科
很多人一开始觉得“多写一点更稳”，结果正好相反。

不适合常驻的内容包括：
- 大段 API 文档
- 一次性排障记录
- 低频发布流程
- 只有某个目录才需要的规则
- Claude 读代码就能看出来的事实

原因很直接：`CLAUDE.md` 会进入上下文。内容越长，每个 session 开局越重，真正重要的规则越容易被淹没。

经验上，第一版尽量控制在短文档范围内。先保证它准确、简洁，再逐步补充高频规则。

## 五、auto memory 适合记什么
auto memory 更像 Claude 给自己写的经验笔记。

它适合沉淀这些信息：
- 你反复纠正过的个人偏好
- 某个仓库里常见的排查入口
- 多次任务里都出现过的局部经验
- “下次遇到类似问题先看哪里”的线索

它不适合承担团队规范。因为 auto memory 更偏个人和会话演化，不应该成为项目事实的唯一来源。

一句话区分：

**从第一天就必须知道的，写进 `CLAUDE.md`；做项目过程中逐渐学到的，交给 auto memory。**

## 六、`.claude/` 目录不是只有 memory
`.claude/` 更像 Claude Code 的项目控制中心。

常见结构是：

```text
.claude/
├── CLAUDE.md
├── rules/
├── skills/
├── agents/
├── settings.json
└── settings.local.json
```

它们的职责不同：
- `CLAUDE.md` 和 `rules/`：告诉 Claude 应该知道什么
- `skills/`：封装可复用流程
- `agents/`：封装子代理角色
- `settings*.json`：控制权限、hooks、环境变量、默认模式

不要把所有问题都塞进 memory。memory 是说明层，不是配置层，也不是自动化层。

## 七、大项目应该用 rules 拆分
当 `CLAUDE.md` 开始变长，通常说明你需要拆。

适合拆到 `.claude/rules/` 的内容：
- 只对前端目录生效的组件规则
- 只对后端目录生效的 API 规则
- 只对测试文件生效的断言风格
- 安全、性能、迁移等主题规则

这样做的好处是：
- 根级 `CLAUDE.md` 保持短
- 规则按主题维护
- 路径相关规则可以在需要时再加载，减少上下文负担

场景判断很简单：如果一条规则只对一部分文件有意义，不要放进总纲。

## 八、`CLAUDE.local.md` 放个人私有信息
`CLAUDE.local.md` 适合本机、本项目、不可共享的补充说明。

例如：
- 本地测试账号说明
- 私有调试地址
- 个人偏好的临时命令
- 不适合提交到仓库的路径

不适合放：
- 团队共同规范
- 构建必须依赖的规则
- 权限和安全边界

团队应该依赖可提交的项目 memory 和 settings，而不是某个人机器上的 local 文件。

## 九、什么时候该停下来，不再加 memory
遇到下面情况，就不要继续往 `CLAUDE.md` 里塞：
- 内容只在少数任务里用到
- 内容更像一套步骤，而不是长期规则
- 内容需要强制执行
- 内容已经可以从代码或 README 直接读到
- 内容只影响某个目录

对应处理方式：
- 少数任务才用：做成 skill
- 需要强制执行：用 permissions 或 hooks
- 目录相关：放进 rules
- 已有文档很完整：用 `@path` 导入或让 Claude 按需读取

## 十、最小可用写法
如果你只想先写一版够用的 `CLAUDE.md`，用这个结构：

```md
# Project Overview
- 这个项目解决什么问题
- 核心代码目录在哪里

# Commands
- 安装命令
- 启动命令
- 测试命令
- 构建或检查命令

# Conventions
- 命名和目录规则
- 重要技术选择
- 不要使用的方案

# Verification
- 改逻辑后跑什么
- 改 UI 后怎么检查
- 改配置后怎么确认
```

写完后问自己一句：如果一个新同事只读这份文件，能不能少踩最常见的坑？如果能，就够了。

## 和后续章节的关系
- 第 7 章讲 settings：哪些东西应该变成硬配置
- 第 8 章讲 permissions：哪些行为应该被允许、询问或禁止
- 第 9 章讲实践习惯：如何把 memory、权限和验证组合成稳定工作流

## 学完标准
- 你知道 memory 是上下文输入，不是强制配置
- 你能判断什么该写进 `CLAUDE.md`，什么该拆成 rules、skills 或 settings
- 你能写出一份短而有用的项目 memory
- 你不会再把 `.claude/` 目录简单理解成“记忆文件夹”
