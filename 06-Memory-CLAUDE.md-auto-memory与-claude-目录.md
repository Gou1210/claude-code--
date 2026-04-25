# 6. Memory：CLAUDE.md、auto memory 与 `.claude/` 目录

## 重要程度
**S 级** - 必须掌握。不会配 memory，就会不停重复解释项目背景；配错了，又会把上下文塞满。

## 学习目标
- 理解 Claude Code 如何跨 session 记住项目
- 分清 `CLAUDE.md`、auto memory、rules、skills、settings 的职责边界
- 知道 `.claude/` 目录里哪些东西属于“记忆层”，哪些属于“配置层”或“扩展层”

## 先纠正一个常见误区
很多人把“Claude 记忆”理解成“它真的永久记住了一切”。

不是。

Claude Code 的 memory 更接近：
- 你写给它的长期说明书
- 它为自己记下的工作笔记

而且它们本质上都是 **上下文输入**，不是硬性配置。

这意味着：
- 写得清楚，Claude 更稳定
- 写得含糊、矛盾、过长，Claude 就更容易跑偏
- 需要强制执行的东西，应该交给 settings、permissions、hooks，而不是只靠 memory

## 学什么
- `CLAUDE.md` 是什么
- auto memory 是什么
- `CLAUDE.local.md` 的用途
- `.claude/` 目录里常见内容
- `.claude/rules/` 的价值
- `#` 快捷写入与 `/memory` 管理

## 你需要掌握
- 每个新 session 都会从 fresh context 开始，但 `CLAUDE.md` 和 auto memory 会再次注入
- `CLAUDE.md` 由你写，适合长期规则、项目事实、构建命令、架构说明
- auto memory 由 Claude 自动积累，适合“你多次纠正后，它自己学到的规律”
- 两者都不是强制执行层
- `.claude/` 不只是放 memory，它还是 Claude Code 的项目级扩展与配置目录

## 一、先把“记忆层”拆开看
截至 **2026-04-25**，Claude Code 的项目记忆至少要分成四类：

### 1. `CLAUDE.md`
你手写的长期说明书。

适合放：
- 项目结构
- 代码规范
- 构建、测试、lint 命令
- 常见工作流
- 重要业务约束
- “不要这样做”的规则

### 2. auto memory
Claude 自动为自己记录的工作笔记。

适合积累：
- 你经常纠正它的偏好
- 调试时发现的关键线索
- 某个 worktree 下常用的命令或约定

### 3. `CLAUDE.local.md`
当前项目、当前用户的私有补充说明。

适合：
- 不想提交到仓库的个人偏好
- 本地测试地址
- 个人调试习惯

### 4. `.claude/rules/`
把大而杂的长期规则拆成多个文件，并且可以按路径触发。

适合：
- 只对某些目录或文件类型生效的规则
- 大项目的模块化说明
- 团队维护多份主题规则，而不是把所有内容都塞进一个超长 `CLAUDE.md`

## 二、`CLAUDE.md` 和 auto memory 到底有什么区别
最简单的理解是：

- `CLAUDE.md`：你写给 Claude 的“主动指令”
- auto memory：Claude 给自己写的“经验笔记”

你可以这样分：

### 更适合写进 `CLAUDE.md` 的内容
- 明确、稳定、需要每次都知道的规则
- 新同事也应该知道的项目信息
- 可验证的流程约束
- 目录结构、命名规范、测试入口

### 更适合让 auto memory 自己积累的内容
- 你的个人偏好被反复纠正后形成的模式
- 某次排障里发现的有效线索
- “这个仓库通常这样排查更快”之类经验型信息

一句话判断：

**你希望它“从第一天就知道”的，写进 `CLAUDE.md`；你希望它“做着做着学会”的，交给 auto memory。**

## 三、`CLAUDE.md` 不是越长越好
这是第一个实际坑。

官方文档给出的倾向非常明确：
- `CLAUDE.md` 最好保持具体、简洁、结构化
- 经验上应尽量控制在 200 行以内

原因不是美观，而是上下文成本。

`CLAUDE.md` 会在 session 启动时进入上下文。如果你把一堆低频参考资料、长篇背景文档、接口细节全塞进去，相当于每个任务开局先背一大包无关信息。

所以：
- 长期事实放 `CLAUDE.md`
- 大段参考材料放 skill 支持文件
- 按目录才需要的规则放 `.claude/rules/`

## 四、`CLAUDE.md` 放在哪
这一点很多人只知道项目根目录，其实不止。

### 常见位置
- `./CLAUDE.md`
- `./.claude/CLAUDE.md`
- `~/.claude/CLAUDE.md`
- `./CLAUDE.local.md`
- 组织级 managed `CLAUDE.md`

### 你可以这样理解它们的作用域
- 用户级：`~/.claude/CLAUDE.md`
  适合你的全局个人偏好
- 项目级：`./CLAUDE.md` 或 `./.claude/CLAUDE.md`
  适合团队共享的项目规则
- 本地项目级：`./CLAUDE.local.md`
  适合你自己在这个项目里的私有偏好
- 组织级 managed `CLAUDE.md`
  适合公司统一规则和合规要求

## 五、Claude 是怎么发现这些 memory 的
不是只读当前目录那一个文件。

Claude Code 会沿着当前工作目录向上查找 `CLAUDE.md` / `CLAUDE.local.md`。  
如果你在子目录启动，它可能同时读到：
- 当前目录的 `CLAUDE.md`
- 上层目录的 `CLAUDE.md`
- 对应位置的 `CLAUDE.local.md`

另外，当前工作目录以下子树里的 `CLAUDE.md` 不是一开始全部加载，而是通常等 Claude 真读到那个子树下的文件时再按需注入。

这也是为什么：
- 大仓库里要小心祖先目录的 `CLAUDE.md`
- 按路径拆 rules 常常比不断膨胀根级 `CLAUDE.md` 更稳

## 六、`.claude/` 目录不要只理解成“放记忆”
这一章标题里专门写 `.claude/`，就是因为它不是一个单一用途目录。

常见结构可以这样记：

```text
.claude/
├── CLAUDE.md
├── rules/
├── skills/
├── agents/
├── settings.json
└── settings.local.json
```

其中：
- `CLAUDE.md` / `rules/`：偏“记忆与说明”
- `skills/` / `agents/`：偏“扩展能力”
- `settings*.json`：偏“硬配置”

所以 `.claude/` 更像 Claude Code 的项目控制中心，而不只是记忆文件夹。

## 七、`.claude/rules/` 是大项目真正该重视的能力
很多初学者只会写一个越来越大的 `CLAUDE.md`，这在小项目里还行，在大项目里很快失控。

更好的方式是：
- 在 `.claude/rules/` 里按主题拆文件
- 对只影响某部分代码的规则加 `paths:` frontmatter

这样带来的好处有两个：

### 1. 结构更清楚
例如拆成：
- `testing.md`
- `security.md`
- `frontend/react.md`
- `backend/api.md`

### 2. 上下文更省
带 `paths:` 的规则不是每次启动都全量常驻，而是在 Claude 真接触到匹配文件时再加载。

这就是为什么 rules 比“超长总纲式 `CLAUDE.md`”更适合复杂项目。

## 八、`@path` 导入是组织 memory 的重要手段
`CLAUDE.md` 可以通过 `@path/to/file` 导入其他文件。

它适合解决两个问题：

### 1. 复用已有文档
如果仓库里已经有：
- `README`
- 架构文档
- `AGENTS.md`

就没必要重复抄写，可以在 `CLAUDE.md` 里导入。

### 2. 跨 worktree 共享你的私有偏好
如果你有多个 git worktree，而某些个人偏好不想每个 worktree 都手动建一份 `CLAUDE.local.md`，可以改成导入家目录下的私有说明文件。

## 九、`CLAUDE.local.md` 怎么用才合理
它不是“默认首选”，而是“本项目、仅自己、可不入库”的补充层。

适合放：
- 本地调试地址
- 私有测试数据说明
- 你个人的输出偏好
- 不适合提交到仓库的备注

不适合放：
- 团队共享规范
- 必须所有人都遵守的约束
- 影响 CI / 构建 / 权限 的硬配置

## 十、如何操作这些 memory
两个高频入口最重要：

### `#` 快捷写入
输入以 `#` 开头时，Claude Code 会把它当作“我要记下来”的内容，并提示写入哪个 memory 文件。

这非常适合：
- 你刚纠正完一个长期偏好
- 你刚发现一个以后还会重复解释的规则

### `/memory`
它是 memory 管理入口。

适合：
- 查看当前加载了哪些 memory
- 编辑 `CLAUDE.md`
- 管理 auto memory
- 开关 auto memory

## 十一、什么时候不该再往 `CLAUDE.md` 里塞东西
下面这些情况，应该停手：

### 1. 内容是流程，不是规则
例如“发布前先做 A，再做 B，再做 C”。  
这更像 skill，而不是常驻 memory。

### 2. 内容只对一小块代码生效
这更适合 path-scoped rule。

### 3. 内容需要强制执行
这更适合 settings / permissions / hooks。

### 4. 内容太长，且不是每次都要看
这更适合 skill 的支持文件。

## 十二、这一章最容易混淆的几个点
### `CLAUDE.md` vs settings
- `CLAUDE.md` 是提示与说明
- settings 是硬配置

### `CLAUDE.md` vs skill
- `CLAUDE.md` 是总在场的长期规则
- skill 是按需调用的流程能力

### `CLAUDE.md` vs auto memory
- 前者你写
- 后者 Claude 写

### `CLAUDE.local.md` vs `~/.claude/CLAUDE.md`
- 前者只影响当前项目
- 后者影响你所有项目

## 十三、建议你这样写第一版项目 memory
最实用的骨架通常就这几块：

```md
# 项目概览
- 这是一个什么系统
- 关键目录在哪

# 开发命令
- 安装
- 启动
- 测试
- lint

# 编码约束
- 命名
- 风格
- 目录放置原则

# 验证要求
- 改完至少跑什么

# 禁止事项
- 哪些文件不要改
- 哪些方案不要用
```

先做到“短、准、能执行”，再逐步演化，不要一开始就写成百科全书。

## 十四、和后续章节的关系
- 第 5 章讲的是上下文负载，这一章讲的是“什么内容值得常驻”
- 第 7 章会讲 settings，解释哪些东西不能只靠 memory
- 第 8 章会讲 permissions，解释如何真正限制 Claude 能做什么
- 第 11 到 15 章会继续展开 skills、subagents、hooks、MCP 等更强的扩展机制

## 学完标准
- 你能清楚区分 `CLAUDE.md`、auto memory、rules、settings、skills
- 你知道 `.claude/` 目录不只是 memory，而是项目级控制中心
- 你能写出一份不过长、但足够实用的项目 `CLAUDE.md`
- 你知道什么时候该继续加 memory，什么时候该改用 rules / skills / hooks
