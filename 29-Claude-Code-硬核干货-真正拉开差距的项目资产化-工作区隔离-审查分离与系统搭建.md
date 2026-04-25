# 29. Claude Code 硬核干货：真正拉开差距的项目资产化、工作区隔离、审查分离与系统搭建

## 重要程度
**S 级**。这一章专门排除“用一个月就会知道的基础卫生”。

这里不讲：

- 会不会 `clear`
- 会不会新开 session
- 会不会引用文件再提问
- 会不会让 Claude “先分析再改”

这些都不配叫硬核知识。

这一章只讲真正能拉开使用质量的东西：**怎么把 Claude Code 用成一个稳定系统，而不是一次性聊天工具。**

## 这章解决什么问题
如果你已经用了一段时间 Claude Code，但还是会反复遇到这些问题：

- 任务做完了，却没有沉淀，下一次还要重新解释
- 规则、记忆、skills、hooks 放得一团乱
- 大任务越做越乱，不知道该换 session 还是换工作区
- review 和 implementation 混在一起，质量不稳定
- 看过很多教程，但大部分只是常识

那这章就是写给你的。

## 一、真正的差距，不在“会不会用”，而在有没有项目级系统
很多人以为高手更强，是因为：

- 会更多命令
- 会更多 prompt
- 装了更多 skill / MCP / 插件

这些都不是核心。

真正的差距通常来自这四层是否分清：

1. **事实层**：项目长期事实，进 `CLAUDE.md`
2. **记忆层**：纠偏经验，进 memory
3. **流程层**：可复用流程，进 skills
4. **强制层**：不能靠自觉的动作，进 hooks

如果这四层没分开，Claude Code 用久了几乎一定会变乱。

## 二、最重要的硬核知识：任务结束后，不是结束，而是“资产分类”
这是中文互联网几乎不讲，但英文高质量实践里非常值钱的东西。

大多数人每次都只做当前任务。  
真正稳定的用法，是任务结束后再做一步：

**让 Claude 帮你判断，这次任务产生了哪些长期资产。**

直接用这段：

```text
基于刚才这次任务，请把值得沉淀的内容分成四类：
1. 应写入项目 CLAUDE.md 的长期事实
2. 应写入 memory 的纠偏经验
3. 应做成 skill 的可复用流程
4. 应做成 hook 的强制检查
请分别给出建议、理由和示例。
```

这一步的价值非常高，因为它把 Claude 从“当前任务执行器”变成“项目系统整理器”。

### 什么时候进 `CLAUDE.md`
只放下面这类东西：

- 长期稳定
- 项目级
- 几乎每次都值得知道

例如：

- 项目用 `pnpm`
- API 测试依赖本地 Redis
- `apps/admin` 不能直接改共享组件
- 提交前至少跑哪几个命令

### 什么时候进 memory
放这种：

- 不一定值得永久写死进项目说明
- 但以后少错一次就值
- 更像纠偏经验，而不是正式规则

例如：

- 这个项目排查认证问题时先看哪条日志
- 这个仓库 review 更关注兼容性，不鼓励顺手重构
- 这个团队更偏好先写回归测试再修 bug

### 什么时候做成 skill
只在它已经变成稳定流程时再做。

标准很简单：

- 你已经重复做过至少 3 次
- 它是“怎么做”
- 不是“记住什么”

例如：

- `bugfix`
- `review`
- `catchup`
- `trace-flow`

### 什么时候做成 hook
只在“不能靠自觉”的地方用。

例如：

- commit 前校验
- 改敏感目录时提醒
- 任务完成后通知

## 三、真正的 session 进阶，不是会不会 `clear`，而是 session 拓扑设计
这是一个很少被明说的点。

真正要学的不是“脏了就清”，而是：

**不同类型的工作，应该使用不同的上下文容器。**

### 1. 当前 session 继续做
适合：

- 还是同一个小任务
- 文件范围稳定
- 你需要延续刚才已经读过的上下文

### 2. 新开 session
适合：

- 从实现切换到 review
- 从写代码切换到补测试
- 从修 bug 切换到重新设计方案
- 你想获得“重新判断”的视角

### 3. 直接换 worktree
这是很多用了一阵子的人都还没真正意识到的点。

适合：

- 一个分支做功能，另一个分支做 hotfix
- 你想并行比较两个方案
- 你想让一个 Claude 实现，另一个 Claude 做 review
- 你不想让新任务污染当前改动现场

对单兵开发者来说，`worktree` 往往比 subagent 更有杠杆。  
因为你更常需要的是**隔离分支、隔离上下文、隔离实验**。

## 四、很多人真正不知道的：review 最好从 implementation 里拆出去
这不是形式主义，而是质量控制。

原因很现实：

- 刚写完代码的上下文，会天然维护刚才的思路
- 同一个上下文里 review，更容易继续顺着实现走
- 写代码和挑错，本来就是两种心智模式

所以更稳的方式是：

### 中小改动
同一 session 里切成 reviewer 模式：

```text
停止实现。
现在只做 review，不提供新实现。
只检查：
1. 行为回归
2. 错误处理
3. 测试缺口
4. 过度改动
5. 潜在副作用
```

### 中大改动
新开一个 session，只喂：

- 任务目标
- 关键 diff
- 关键文件

不要把整个实现过程一起喂进去。  
review 要的是新鲜视角，不是延续旧思路。

## 五、memory 的真正关键，不是“会不会记住”，而是它的加载机制
很多人把 memory 想成“自动永久大脑”，这是错的。

官方文档里真正重要、但很多人没意识到的点有三个：

### 1. auto memory 是机器本地的
它不是云端全局脑子。  
同一个仓库的 worktree 和子目录共享一套 auto memory，但换机器、换云环境，不会自动同步。

### 2. 启动时不是全部都加载
官方说明是：

- `MEMORY.md` 启动时只加载前 **200 行** 或前 **25KB**
- 超出部分不会在 session 开始时自动加载
- 详细内容会被拆去 topic files，需要按需读取

这意味着：

**你不能把 memory 写成垃圾场。**

### 3. `CLAUDE.md` 会整份加载，但越短越稳
官方文档明确说 `CLAUDE.md` 会完整加载，但更短通常更容易遵循。

这背后的实战结论是：

- `CLAUDE.md` 适合放短而硬的长期事实
- 详细流程不要全塞进去
- 长流程更适合 skill

## 六、真正值钱的 skill，不是“功能插件”，而是稳定流程模板
很多初学者一看到 skills，就想装很多“能力”。

但英文世界高质量实践更接近这个方向：

- skill 不是堆能力名词
- skill 是让 Claude 在某类任务里少走弯路的流程模板

更实用的 skill 通常不是：

- `frontend-expert`
- `backend-master`
- `ultimate-architect`

而是：

- `bugfix`
- `review`
- `catchup`
- `trace-flow`

因为这些技能名背后对应的是**动作顺序**，不是虚假的身份设定。

## 七、hooks 的高级价值，不是“自动化很酷”，而是它能把项目变成可约束系统
大多数人只知道 hook 能自动跑点东西。  
这只是表层。

真正值钱的是：

- 把关键节点标准化
- 把不能忘的动作变成系统动作
- 让 Claude 的工作流更像工程流程，而不是随缘发挥

官方 hooks 里真正值得你关注的事件，不是全部，而是这几类：

### 1. `InstructionsLoaded`
当 `CLAUDE.md` 或 `.claude/rules/*.md` 被加载时触发。

它的价值不是花哨，而是让你知道：

- 哪个规则真的被加载了
- 是 session 开始加载的，还是后面懒加载的
- 某条路径规则到底有没有命中

如果你以后 rules 多了，这个事件非常值钱。

### 2. `WorktreeCreate`
这个事件的意义很大，因为它把 worktree 从“手工 Git 操作”提升成“Agent 工作流的一部分”。

你可以在 worktree 创建时做：

- 复制 `.env`
- 准备本地依赖
- 设定特定目录结构

这比单纯记得去手工准备稳定得多。

### 3. `PreCompact` / `PostCompact`
这两个事件不是给新手看的，但很适合做可观测性。

例如：

- compact 前记录当前任务状态
- compact 后提醒你补写 handoff

### 4. `CwdChanged`
如果项目依赖目录切换时的环境变化，这个事件很实用。

例如：

- 自动切 Node 版本
- 自动刷新环境变量

### 5. `Notification`
对单兵开发者很实用。

因为你经常会把 Claude 放后台跑。  
任务结束、等待你确认、权限被拒时，及时通知比很多 fancy 自动化更值钱。

## 八、MCP 的真正角色，不是“插件市场”，而是安全边界
这是 Simon Willison 和 Shrivu 都强调得很清楚的观点。

你如果把 MCP 理解成“我再多装几个插件，Claude 就更强”，很容易走偏。

更稳的理解是：

**MCP 的价值主要在受控访问，而不是能力炫技。**

最合理的 MCP 通常做这几类事：

- 访问受控资源
- 执行高风险但被严格包装的动作
- 提供少量高层能力，而不是一堆底层 API

例如：

- 安全读取内部数据
- 受控执行部署动作
- 在特定环境里运行代码并回传结果

如果一个 MCP 只是把一大堆 API 原样暴露给模型，通常不是好设计。

## 九、单兵开发者真正应该先学的，不是 agent teams，而是 worktree + 原子提交
很多人一看英文世界在讲 multi-agent、agent teams，就觉得自己也该上。

不是。

对你这种单兵开发者，优先级通常是：

1. worktree 隔离并行任务
2. 原子提交
3. review 分离
4. 任务结束做资产分类
5. 再考虑更复杂的代理编排

### 为什么原子提交很重要
GSD 这类系统里反复强调的一点是：

- 每个任务一个小提交
- 提交要可回滚
- 提交要可审查
- 提交要能成为以后 Claude 的可读历史

这不是 Git 教条，而是 agent 时代的新价值：

**清晰提交历史，本身就是未来上下文。**

## 十、英文世界里一个真正有用的观察：不要盲目扩展工具面
很多人误以为“更多工具 = 更强能力”。

但实践里经常相反：

- 工具越多，Claude 越容易走偏
- 权限越宽，风险越高
- 工具面越大，行为越不稳定

真正成熟的做法反而常常是：

- 先把默认文件工具、shell、测试流程用稳
- 再补少量高价值 skill
- MCP 只接真正需要的
- hook 只上高杠杆节点

不是越多越好，而是越收敛越稳。

## 十一、很多人没意识到的硬核点：规则也有“加载成本”和“命中问题”
你不是把规则写了，它就一定有效。

真正的问题有两个：

### 1. 规则有没有被加载
特别是路径规则、嵌套 `CLAUDE.md`、`.claude/rules/*.md`，它们可能是按需加载的。

### 2. 规则写法是不是太长、太抽象、太像空话
例如这种规则就很差：

- 保持高质量
- 注意最佳实践
- 写清晰代码

这种话几乎没有执行价值。

真正有用的规则应该像这样：

- 这个项目统一用 `pnpm`
- `apps/admin` 不要直接改共享组件
- 任何认证相关修改都必须补回归测试
- 提交前至少运行 `pnpm test --filter auth`

规则如果不具体，就只是情绪，不是系统。

## 十二、如果你要让 Claude 越用越像搭档，最该养成的是“收尾固定动作”
这不是基础知识，但它特别值钱。

每次任务结束，你都固定让 Claude 做 4 件事：

1. 总结这次改动
2. 列出已验证结果
3. 列出剩余风险
4. 做一次资产分类

直接用这段：

```text
先做收尾，不要继续实现。
1. 总结这次改动
2. 列出我已经完成的验证
3. 列出剩余风险和未覆盖点
4. 判断哪些内容应进入项目 CLAUDE.md、memory、skill、hook
```

你连续做一个月，Claude 在这个项目里的表现会稳定很多。  
因为你不是每次都把上下文用完就扔，而是在持续建设项目系统。

## 十三、真正有区分度的 12 条硬核结论
1. `CLAUDE.md` 适合放长期事实，不适合放长流程。
2. memory 不是永久脑子，而是有加载限制的记忆层。
3. 任务结束后的资产分类，价值高于很多所谓“技巧”。
4. 对单兵开发者，worktree 往往比 subagent 更值钱。
5. review 最好从 implementation 上下文中拆出去。
6. skill 应该代表稳定流程，不该代表空泛身份。
7. hook 的价值在强制节点，不在炫技自动化。
8. MCP 的最佳角色是安全网关，不是插件大卖场。
9. 更短、更硬的规则，比更长、更全的规则更有效。
10. 清晰的小提交，是未来 Claude 可读的上下文资产。
11. 工具面不是越大越好，收敛常常更稳。
12. 真正的进阶不是会更多命令，而是知道项目系统怎么搭。

## 十四、你现在最该先做的 5 件事
1. 重写项目 `CLAUDE.md`，只保留长期事实，把流程删出去。
2. 固定每个任务结束都做一次资产分类。
3. 把 review 从实现上下文里拆出去。
4. 遇到并行任务时优先考虑 worktree。
5. 先只保留少量高价值 skill / hook，不要继续堆工具。

## 十五、这章和前面章节的区别
前面的很多内容更像“会用 Claude Code 的人应该知道什么”。  
这一章只回答：

**哪些知识不是自然用久了就一定会意识到，而是会显著影响你项目质量的硬核做法。**

如果一条知识“用一个月自然就知道”，那它就不该出现在这章。

## 学完标准
- 你知道任务结束后怎样把经验沉淀成项目资产
- 你知道 session、新 session、worktree 分别该在什么场景用
- 你知道为什么 review 要和 implementation 分离
- 你知道 `CLAUDE.md`、memory、skill、hook、MCP 的真实分工
- 你知道真正的差距来自系统搭建，而不是会几个命令

## 关键英文来源（截至 2026-04-25）
### 官方文档
- How Claude remembers your project  
  https://code.claude.com/docs/en/memory
- Hooks reference  
  https://code.claude.com/docs/en/hooks
- Commands  
  https://code.claude.com/docs/en/commands
- Best practices for Claude Code  
  https://code.claude.com/docs/en/best-practices

### Anthropic 工程博客
- Effective context engineering for AI agents  
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Harness design for long-running application development  
  https://www.anthropic.com/engineering/harness-design-long-running-apps

### 高质量实践者经验
- Shrivu Shankar, *How I Use Every Claude Code Feature*  
  https://blog.sshh.io/p/how-i-use-every-claude-code-feature
- Simon Willison, *How I Use Every Claude Code Feature* 引述与长期评论  
  https://simonwillison.net/2025/Nov/2/how-i-use-every-claude-code-feature/
- Simon Willison, Claude Code 相关长期观察  
  https://simonwillison.net/tags/claude-code/

### 开源工作流系统
- GSD / get-shit-done  
  https://github.com/gsd-build/get-shit-done

## 哪些是官方规则，哪些是实践共识
- memory 的机器本地性、启动加载限制、`CLAUDE.md` 完整加载、hook 事件名称和能力边界：来自官方文档
- 把长流程移出 `CLAUDE.md`、任务结束做资产分类、review 分离、worktree 对单兵开发更有杠杆、工具面收敛：主要来自实践者共识和工程经验
- GSD 的原子提交、规划文档、并行执行思路：属于实验性但很有参考价值的工作流系统，不是 Claude Code 官方标准
