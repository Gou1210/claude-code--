# 13. Agent Teams：多代理协作

## 重要程度
**B 级** - 有场景时重点学。Agent Teams 不是日常必开功能，但一旦任务复杂到需要多路并行、相互讨论和独立协作，它能明显抬高 Claude Code 的上限。

## 学习目标
- 理解 agent team 和 subagent 的架构差异
- 掌握 agent teams 的启动方式、显示模式、协作机制和适用场景
- 知道什么时候值得从“单会话 + subagents”升级到“多会话团队”

## 学什么
- Agent teams 官方文档
- lead / teammates 模型
- shared task list
- peer-to-peer messaging
- display mode
- teammate lifecycle
- 与 subagents、worktree、permissions 的关系

## 你需要掌握
- 截至 **2026-04-25**，Agent Teams 已有独立官方文档，但仍是 **experimental**，默认关闭
- Agent team 不是单会话内部的任务分发，而是 **多个独立 Claude Code session 协同**
- 一个 lead 负责任务协调、分工和结果综合，teammates 彼此也可以直接沟通
- 它比 subagents 更重，但也更适合需要讨论、互相依赖和长期协作的复杂任务

## 一、先纠正一个最重要的误解
很多人看到“多代理协作”，第一反应是：
- 就是多开几个 Claude
- 或者就是更强版 subagent

这都不准确。

官方当前的定义更接近：

**一个 agent team 是多个独立 Claude Code 实例组成的协作团队。**

其中：
- 一个 session 作为 `lead`
- 多个 session 作为 `teammates`
- 共享任务列表
- 可以互相发消息
- 由系统管理依赖和协同

所以 agent team 的本质不是“在一个会话里多分几支线程”，而是：

**让多个独立工作中的 Claude，会像一个真实小团队一样协作。**

## 二、为什么它和 subagent 不是一回事
这是第 12 章之后最应该立刻建立的区分。

### Subagent
- 运行在你的主 session 之内
- 有独立上下文，但结果最终回到主会话
- 适合一次性、聚焦型、只要摘要的任务

### Agent Team
- 是多个独立 session
- teammates 可以彼此发消息，不只向主代理汇报
- 共享任务列表，可以自协调
- 更适合复杂协作而不是单点执行

官方的对比也非常明确：
- subagents：token 成本更低、适合 focused work
- agent teams：token 成本更高、适合需要 discussion 和 collaboration 的任务

一句话记忆：

**subagent 是“隔离 worker”，agent team 是“独立 teammate”。**

## 三、Agent Teams 当前的官方状态
截至 **2026-04-25**，官方文档明确说明：
- Agent Teams 是实验功能
- 默认关闭
- 在 session 恢复、任务协调、关闭行为上仍有已知限制

启用方式是设置：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

也可以通过环境变量方式开启。

这意味着你在知识结构上要把它归为：
- 已可学
- 已可用
- 但不应当默认作为所有项目的日常标准流程

## 四、Agent Team 的核心运行模型
官方对运行模型的描述可以压缩成四个角色关系。

### 1. Lead
主会话。

负责：
- 读取你的目标
- 组建团队
- 分配任务
- 汇总结果
- 对外代表整个 team

### 2. Teammates
独立的 Claude Code 会话。

特点：
- 各自有自己的 context window
- 可被分配不同任务
- 可互相发消息
- 可独立推进工作

### 3. Shared task list
团队共享任务列表。

作用：
- 追踪任务状态
- 管理依赖关系
- 让阻塞任务在依赖完成后自动解锁

### 4. Mailbox / messaging
代理之间的消息系统。

作用：
- 互相同步发现
- 请求澄清
- 传递结论
- 让 lead 不必手动中转所有信息

## 五、什么时候最值得用 Agent Teams
官方建议的最佳场景，本质上有三个共同点：
- 任务复杂
- 可并行
- 参与者之间需要互相交流

### 高价值场景 1：研究型探索
例如：
- 从架构、产品、性能三个角度并行研究一个新功能
- 对同一个 bug 建立多个假设并同时验证

### 高价值场景 2：并行审查
例如：
- 一个 teammate 看安全
- 一个 teammate 看性能
- 一个 teammate 看可维护性

### 高价值场景 3：新功能开发
例如：
- 一个 teammate 负责后端
- 一个 teammate 负责前端
- 一个 teammate 负责测试策略

### 高价值场景 4：需要持续交互的复杂任务
不是“做完告诉我”，而是：
- 中间要交换信息
- 互相补充和纠正
- 任务依赖会动态变化

## 六、什么时候不要上 Agent Teams
这是更重要的判断。

更不适合 agent teams 的情况：
- 小修小补
- 局部重构
- 单点调查
- 只需要一个结果摘要
- 你还没把问题定义清楚

这些通常更适合：
- 单会话
- 或 subagents

一句话判断：

**如果你连团队应该怎么分工都说不清，通常还没到该开 agent team 的时候。**

## 七、最推荐的升级路径：单会话 -> subagent -> agent team
从学习路径上，最稳的顺序是：

1. 先会单会话工作流
2. 再会 subagent 委派
3. 最后再用 agent teams

因为 agent teams 会引入更多复杂度：
- 更多 token 成本
- 更多权限打断
- 更多运行状态
- 更多协调成本

所以它不是“更高级就该默认用”，而是：

**只有在复杂度真的需要多人协作时才值得用。**

## 八、如何启动一个 Agent Team
官方当前推荐的方式不是写一堆配置，而是直接用自然语言描述：
- 你要做什么
- 希望 team 怎么分工
- 各自从什么角度探索

例如可以这样理解：

```text
我要设计一个 CLI 工具来追踪代码库里的 TODO。
请创建一个 agent team：
一个人研究用户体验和命令设计，
一个人研究索引与性能，
一个人研究输出格式和集成方式。
最后汇总建议。
```

这是因为：
- lead 会根据任务动态建队
- system 会管理 teammates 生命周期
- 你不需要手写一整套 orchestration DSL

## 九、显示模式：你是“切着看”，还是“同时看”
官方当前支持两种主要 display mode。

### `in-process`
所有 teammates 运行在主终端里。

特点：
- 通用
- 不额外依赖环境
- 可用快捷键切换 teammates

### `split panes`
每个 teammate 单独一个 pane。

特点：
- 能同时观察多个代理
- 更像真的在看团队协作
- 需要 `tmux` 或 iTerm2 支持

官方还提供 `teammateMode` 设置项，支持：
- `auto`
- `in-process`
- `tmux`

## 十、如何和 teammates 互动
Agent team 和 subagent 的差异之一，就是你不仅能跟 lead 说话，还能直接和 teammate 交互。

官方文档提到：
- 在 `in-process` 模式里，可通过快捷键轮换 teammates
- 在 split panes 模式里，可直接点进某个 pane 交互

这意味着：

**你不是只能看结果，你可以在协作中途介入、纠偏、追问。**

这也是它更像真实团队而不是后台 worker 的地方。

## 十一、团队状态和数据存放在哪里
官方当前说明，teams 和 tasks 都保存在本地：
- team config：`~/.claude/teams/{team-name}/config.json`
- task list：`~/.claude/tasks/{team-name}/`

但官方也明确提醒：
- 这些主要是运行时状态
- Claude 会自动生成和更新
- 不建议手工编辑

你真正应该手工维护的，不是 team runtime state，而是：
- subagent 定义
- skill
- hooks
- permissions

也就是那些“团队成员的能力模板”，而不是“当前这一轮协作的临时状态”。

## 十二、和 subagent 的最佳配合方式
一个很容易忽略的点是：

**agent team 和 subagent 不是替代关系，而是可以叠加。**

例如：
- lead 组织一个 team
- 某个 teammate 内部再调用 subagent
- 用于做更小粒度的隔离调查

所以更准确的理解是：
- subagent 是单 session 内部的分工机制
- agent team 是多 session 之间的协作机制

## 十三、为什么权限会变得更烦
官方 troubleshooting 专门提到：

**teammate 的权限请求会冒泡到 lead。**

这会带来一个现实问题：
- 人一多
- 权限弹窗也多

所以如果你准备认真用 agent teams，应该提前：
- 放行常用低风险操作
- 配好 permissions
- 减少重复审批

否则多代理协作会被权限确认打断得很厉害。

## 十四、和 worktree 的关系
Agent Teams 很适合并行，但并行修改文件天然有冲突风险。

因此在实操上，你应当天然联想到：
- worktree
- 文件所有权分工
- teammate 角色边界

虽然 team 的核心是“协作”，但真正落地时，仍要控制：
- 谁改哪里
- 哪些工作只读
- 哪些工作只审查不实施

否则就会出现典型问题：
- 改动互相踩
- 结论互相覆盖
- lead 最后很难整合

## 十五、当前已知限制意味着什么
官方说明 agent teams 在这些方面仍有限制：
- resume
- task coordination
- shutdown behavior

这对你的实际含义是：

### 1. 别把它当长期常驻基础设施
它更适合一段复杂任务，而不是一直开着不关。

### 2. 先从 research / review 开始
研究型任务容错更高，最适合试 agent teams。

### 3. 文件改动类任务要更保守
尤其多人并行写代码时，要更重视边界和验证。

## 十六、这一章最该形成的判断标准
推荐你用下面四个问题判断值不值得开 team：

1. 任务是否天然可拆成几个平行方向？
2. 这些方向之间是否需要互相交流？
3. 我是否希望直接和某个角色单独对话？
4. subagents 已经不够，因为它们彼此不能自由协作？

如果这四个问题里有两个以上是“是”，agent teams 就很可能值得试。

## 十七、和前后章节的关系
- 第 12 章讲的是 subagents：单会话内的委派
- 第 13 章讲的是 agent teams：多会话之间的协作
- 第 14 章会把视角转到 hooks：如何把确定性自动化嵌进整个生命周期

## 学完标准
- 你知道 agent team 和 subagent 的根本差别
- 你知道 agent teams 当前仍是实验功能，默认关闭
- 你知道什么时候适合开 team，什么时候该继续用 subagent
- 你知道 lead、teammates、shared task list、display mode 各自解决什么问题
