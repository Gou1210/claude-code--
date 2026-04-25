# 11. Skills：可复用能力封装

## 重要程度
**A 级** - 强烈建议掌握。Skills 是 Claude Code 从“每次都重新讲一遍”升级到“沉淀成团队资产”的核心机制。

## 学习目标
- 理解 skill 和 `CLAUDE.md`、hooks、subagents 的边界
- 掌握 `SKILL.md` 的基本结构、frontmatter 和加载机制
- 能把高频流程、项目知识、团队规范封装成可复用 skill

## 学什么
- Skills 官方文档
- bundled skills
- `SKILL.md`
- `.claude/skills/` 目录结构
- frontmatter 字段
- supporting files、arguments、allowed-tools、hooks、fork context
- custom commands 已并入 skills 的最新变化

## 你需要掌握
- 截至 **2026-04-25**，**custom commands 已并入 skills 体系**
- skill 的最大价值不是“定义一个命令”，而是 **按需加载，节省主上下文**
- `CLAUDE.md` 解决长期稳定规则，skill 解决按场景触发的知识和流程
- skill 既可以手动 `/name` 调，也可以让 Claude 自动在相关上下文里调用

## 一、先理解：为什么 skill 比长提示词更高级
官方对 skills 的定义很直接：

**把一段你经常重复使用的知识、流程或操作套路，放进 `SKILL.md`，让 Claude 在需要时自动或手动加载。**

它解决的不是“Claude 会不会”，而是：
- 你是不是每次都得重新讲
- 这些说明是不是应该常驻上下文
- 这些说明会不会把主会话越聊越肿

skill 的关键优势在于：

**它默认不是一开始就把全文塞进上下文，而是在 relevant 时再加载。**

这就是它和 `CLAUDE.md` 最大的差别。

## 二、skills 到底适合装什么
官方把 skill 内容大体分成两类。

### 1. Reference 型
适合放：
- 项目 API 约定
- 代码风格补充
- 领域知识
- 某个子系统的非显然规则

这类 skill 更像“按需知识包”。

### 2. Task 型
适合放：
- 部署流程
- 提交流程
- 修 issue 流程
- 代码审查流程
- 发布清单

这类 skill 更像“按需工作流模板”。

## 三、什么时候用 skill，而不是写进 `CLAUDE.md`
这是最重要的判断题之一。

### 用 `CLAUDE.md` 的情况
- 每次会话都应生效
- 属于项目级长期规则
- 需要在所有任务里持续影响行为

### 用 skill 的情况
- 只在部分任务里有用
- 内容较长，不适合常驻
- 是一个流程，而不是一条规则
- 你已经开始重复粘贴相同说明

一句话区分：

**长期稳定且普适的，放 `CLAUDE.md`；按场景触发的，放 skill。**

## 四、skills 的最新体系变化：custom commands 已并入
官方文档已经明确说明：
- `.claude/commands/deploy.md`
- `.claude/skills/deploy/SKILL.md`

这两种方式都能创建 `/deploy`，而且现在工作机制已经统一。

这意味着你应该建立的新认知是：

**以后优先把“命令”理解成一种带 frontmatter、带 supporting files、可自动调用的 skill。**

旧的 `.claude/commands/` 仍可工作，但 skills 更完整。

## 五、先看基本结构：一个 skill 至少长什么样
每个 skill 本质上是一个目录，入口是 `SKILL.md`。

典型结构：

```text
.claude/skills/fix-issue/
├── SKILL.md
├── reference.md
├── examples.md
└── scripts/
```

其中：
- `SKILL.md` 必需
- 其他文件可选

官方建议把核心说明留在 `SKILL.md`，把细节放 supporting files，避免主文件过长。

## 六、skill 放在哪，决定谁能用
官方当前支持这些层级：

### Personal
- `~/.claude/skills/<skill-name>/SKILL.md`
- 你所有项目都能用

### Project
- `.claude/skills/<skill-name>/SKILL.md`
- 当前项目内可用

### Enterprise
- 组织级分发

### Plugin
- 由插件提供，带命名空间

### 优先级
官方说明同名 skill 优先级是：

`enterprise > personal > project`

plugin skill 用 `plugin-name:skill-name` 命名空间，不直接冲突。

## 七、skill 的最小可用写法
```md
---
name: explain-code
description: 用类比和 ASCII 图解释代码。适合讲解代码结构、执行流程和关键坑点。
---

解释代码时总是：
1. 先给一个生活类比
2. 再画一个 ASCII 结构图
3. 再按执行顺序解释
4. 最后指出一个容易误解的点
```

这里最重要的两个字段是：
- `name`
- `description`

其中真正影响自动触发效果的，往往是 `description`。

## 八、frontmatter 里最值得先掌握的字段
官方字段很多，但你第一轮最该掌握的是这些。

### `name`
skill 名称。

### `description`
做什么、什么时候用。Claude 会用它判断是否自动加载。

### `when_to_use`
补充触发语境、示例请求、边界条件。

### `disable-model-invocation: true`
禁止 Claude 自动调用，只能你手动 `/skill-name` 触发。

适合：
- deploy
- commit
- 发消息
- 有副作用的流程

### `user-invocable: false`
用户不能手动调，Claude 可在相关上下文自动用。

适合：
- 背景知识
- 系统历史
- 老模块说明

### `allowed-tools`
skill 激活时预先允许某些工具，无需逐次审批。

### `arguments`
声明命名参数，便于在 skill 里用 `$name`。

### `context: fork`
让 skill 在 fork / subagent 上下文里运行，而不是在主会话里内联执行。

### `agent`
当 `context: fork` 时，指定使用哪个 subagent 类型。

### `paths`
限制只有在处理特定路径时，Claude 才自动加载这个 skill。

### `hooks`
为这个 skill 生命周期配置 hooks。

### `shell`
控制 `!` 命令注入时用 `bash` 还是 `powershell`。

## 九、自动调用与手动调用的差别
官方把 skill 的调用权限讲得很清楚。

默认情况下：
- 你可以 `/skill-name`
- Claude 也可以在相关上下文自动调用

### 如果你不想让 Claude 自动调用
用：

```yaml
disable-model-invocation: true
```

### 如果你不想让用户手动调
用：

```yaml
user-invocable: false
```

这是一个非常实用的设计：
- 有副作用的流程，通常只允许手动触发
- 背景知识型 skill，通常只让 Claude 自动用

## 十、skill 内容什么时候加载，什么时候不加载
这也是 skill 最值钱的机制之一。

### 默认情况
- skill 的描述会进入上下文，告诉 Claude 有这个东西
- skill 的全文不会常驻
- 真正被调用时，全文才进入对话

### 调用之后会怎样
官方说明：
- 渲染后的 `SKILL.md` 会作为一条消息进入会话
- 会在本次 session 中持续保留
- compact 时会带着走，但受 token budget 约束

这意味着：

**skill 不是“一次性提示词”，而是被调用后会持续影响后续任务。**

## 十一、arguments：让 skill 真正可参数化
你可以把 skill 做成真正的模板，而不只是固定文案。

### 最简单方式
用 `$ARGUMENTS`

```md
---
name: fix-issue
description: 修复一个 GitHub issue
disable-model-invocation: true
---

修复 GitHub issue $ARGUMENTS，并按团队规范补测试、提交变更。
```

执行：
```text
/fix-issue 1234
```

### 更精细方式
用：
- `$ARGUMENTS[0]`
- `$0`
- `$issue`

如果声明了：

```yaml
arguments: [issue, branch]
```

就可以在正文里写：
- `$issue`
- `$branch`

## 十二、supporting files：别把所有东西塞进 `SKILL.md`
官方建议：
- `SKILL.md` 控制在 500 行以内
- 详细资料拆出去

适合拆出去的内容：
- API 规范
- 示例输出
- 长参考文档
- 脚本
- 模板

在 `SKILL.md` 里明确告诉 Claude：
- 哪个文件放的是什么
- 什么时候该去读

这样既节省上下文，又提升可维护性。

## 十三、`allowed-tools`：给 skill 预授权，而不是让它变成万能钥匙
官方明确说明：

`allowed-tools` 的作用是：
- 在 skill 激活时，给指定工具预先放行

它 **不是**：
- 限制 skill 只能用这些工具
- 覆盖全局 deny

例如一个 `commit` skill，可以允许：

```yaml
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
```

但如果某工具被全局 deny，skill 也不能越过。

## 十四、动态上下文注入：让 skill 先拉实时信息，再交给 Claude
官方支持 `!` 命令预处理。

它的含义不是“让 Claude 去执行一条命令”，而是：

**skill 在真正送进模型之前，先把命令结果嵌进 prompt。**

这很适合：
- 先取 `gh pr diff`
- 先取 `git status`
- 先取环境信息
- 再让 Claude 基于实时数据工作

这类 skill 很适合做：
- PR 总结
- 发布前检查
- 环境诊断

## 十五、skill 也可以跑在 fork / subagent 上下文
官方已经把这条链路打通：
- `context: fork`
- `agent: Explore` 或自定义 subagent

这意味着 skill 不只是“主会话内的一段说明”，还可以变成：

**把某个流程模板，放到隔离上下文里执行。**

适合场景：
- 调研型 skill
- 输出很多的 skill
- 想和主会话隔离的批处理流程

## 十六、bundled skills：Claude 自带的一批可用能力
官方当前明确列出的 bundled skills 包括：
- `/simplify`
- `/batch`
- `/debug`
- `/loop`
- `/claude-api`

这类 skill 和普通内建命令不同：
- 不是固定逻辑直接执行
- 而是 Anthropic 已经写好的 prompt-based playbook

所以你要理解：

**bundled skills 本质上也是“官方已经替你封装好的 skill”。**

## 十七、什么时候该用 skill，什么时候不该
### skill 很适合
- 修 issue 的固定流程
- 代码解释风格
- API 约定
- 发布 checklist
- PR 总结
- 数据分析套路

### skill 不太适合
- 必须每次强制执行的校验
- 需要真正系统级拦截的约束
- 需要大量独立上下文调查的任务

这些通常更适合：
- hooks
- permissions
- subagents

## 十八、和 hooks、subagents、CLAUDE.md 的边界
### `CLAUDE.md`
长期规则。

### Skill
按需知识 / 按需流程。

### Hook
确定性自动动作。

### Subagent
隔离上下文的角色化执行。

这是最推荐记忆的四分法。

## 十九、团队里怎么组织 skill 才不乱
### 建议做法
- 项目共用 skill 放 `.claude/skills/`
- 个人偏好 skill 放 `~/.claude/skills/`
- 一个 skill 只解决一类问题
- `description` 写清楚适用场景
- 参考资料拆 supporting files
- 有副作用的 skill 默认手动触发

### 命名建议
- `fix-issue`
- `api-conventions`
- `review-pr`
- `release-checklist`
- `explain-legacy-module`

## 二十、这一章最重要的升级
从这一章开始，你不该再把 Claude Code 只理解成：
- 每次重新打一段 prompt

而应该理解成：
- 我可以把组织经验、项目知识、工作套路，封装成一个会随着项目积累的能力层

这也是 skill 最本质的价值：

**它让“个人提示词技巧”变成“团队可复用资产”。**

## 二十一、和前后章节的关系
- 第 10 章讲的是高频工作流模板
- 第 11 章讲的是如何把这些模板沉淀为 skills
- 第 12 章会继续讲：如果任务本身会污染上下文，就不只是 skill 化，而要 subagent 化

## 学完标准
- 你知道为什么 skill 比长提示词更适合沉淀可复用能力
- 你能区分 skill、`CLAUDE.md`、hooks、subagents 的职责边界
- 你至少能设计出 3 个值得做成 skill 的高频流程
- 你会写基础 `SKILL.md`，并理解 `description`、`disable-model-invocation`、`allowed-tools`、`arguments`、supporting files 的作用
