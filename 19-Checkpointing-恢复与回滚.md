# 19. Checkpointing、恢复与回滚

## 重要程度
**A 级** - 强烈建议掌握。Checkpointing 是你敢不敢让 Claude 做大改动的心理底座。

## 学习目标
- 理解 checkpoint 到底追踪什么、不追踪什么
- 学会把 `/rewind`、session 恢复、Git 回滚分开理解
- 建立“大改动前有安全网”的操作习惯

## 学什么
- Checkpointing 官方文档
- `/rewind`
- `Esc Esc`
- session persistence
- checkpoint cleanup
- Bash / external changes 的边界

## 你需要掌握
- 截至 **2026-04-25**，Claude Code 会自动跟踪 **编辑工具** 做出的文件改动
- 每个用户 prompt 都会创建一个新 checkpoint
- checkpoints 会跨 session 保留，并在默认 **30 天** 后清理
- checkpoint 不是版本控制，不会替代 Git

## 一、先纠正一个误解：checkpoint 不是 Git
很多人一看到“恢复与回滚”，会自然联想到：
- branch
- commit
- reset
- revert

但 checkpoint 不是这套体系。

官方当前对 checkpoint 的定位很明确：

**它是 session 级的快速恢复安全网。**

所以最推荐的理解是：
- checkpoint 像 local undo
- Git 像 permanent history

一句话：

**checkpoint 保你这轮会话别失控，Git 保你长期历史可追溯。**

## 二、checkpoint 是什么时候创建的
官方当前说明：
- 每个用户 prompt 都会生成一个新 checkpoint
- Claude 在每次编辑前，会捕获代码状态

这意味着你不需要手动说：
- “现在帮我打个 checkpoint”

它默认就在工作。

这是为什么你能：
- 较大胆地让 Claude 做多文件修改
- 因为随时可以 rewind 回之前的状态

## 三、checkpoint 到底追踪什么
这个问题非常关键，因为很多人会高估它。

官方当前明确说明：

**它追踪的是 Claude 的文件编辑工具造成的改动。**

也就是：
- `Edit`
- `Write`
- 其他文件编辑型工具

这些改动能被 rewind 撤回。

## 四、它不追踪什么
这同样重要。

### 1. Bash 改的文件
官方明确写到：
- `rm`
- `mv`
- `cp`

这类通过 Bash 改的文件，不在 checkpoint 追踪范围内。

### 2. 会话外的人工改动
你自己手改的内容，不一定被纳入当前 session 的 checkpoint。

### 3. 其他并发 session 的改动
通常也不在当前 session 的 checkpoint 里。

一句话：

**checkpoint 不是“全文件系统快照”，而是“Claude 编辑工具的会话级变更历史”。**

## 五、为什么这个边界必须记住
因为它直接决定你的风险判断。

### 如果主要是 Edit / Write
checkpoint 非常有用。

### 如果主要是 Bash 批处理
checkpoint 的保护力就会明显下降。

所以当 Claude 要做：
- 大量 shell 脚本式重命名
- 删除文件
- 搬运目录

你就不能误以为 `/rewind` 一定能救回来。

这时更稳的做法是：
- 先 commit
- 或先新 branch
- 或先手工备份

## 六、怎么进入 rewind
官方当前支持两种主要入口。

### 1. 键盘快捷方式
- `Esc` 然后再按一次 `Esc`

### 2. 命令
- `/rewind`

进入后，你会看到：
- 一串 session 中的用户 prompt
- 可滚动列表
- 选中后回到当时状态

## 七、rewind 回退的到底是什么
这也是容易混淆的点。

rewind 会回退：
- 会话状态
- Claude 做出的被追踪编辑

但它不是：
- “把整个系统回到过去”
- “把所有终端命令副作用都抹掉”

所以不要把 rewind 理解成全能时光机。

## 八、session 恢复和代码恢复不是一回事
这是本章最重要的区分之一。

### `--resume` / `--continue`
恢复的是：
- 对话
- session 上下文
- 相关状态

### `/rewind`
恢复的是：
- 到某个 checkpoint 前的会话和编辑状态

### Git 回滚
恢复的是：
- 版本库状态

三者作用面完全不同。

如果这三者混成一团，就很容易做出错误操作。

## 九、checkpoint 跨 session 保留，意味着什么
官方当前说明：
- checkpoints 会在 resumed conversations 中继续可用

这有两个直接好处：

### 1. 长任务更安全
不是一关终端就失去恢复能力。

### 2. 续做任务时仍能回到 earlier state
尤其适合：
- 长重构
- 多轮修复
- 几天内持续推进的任务

## 十、但它会清理，不是永久保留
官方当前说明：
- 默认 30 天清理
- 可配置

这再次强调了：

**checkpoint 不是长期历史系统。**

要保留真正重要的状态，仍然应该：
- git commit
- 分支
- PR

## 十一、什么时候最该主动依赖 checkpoint
### 1. 大范围代码修改前
尤其是你想让 Claude 放手做时。

### 2. 结构性重构
容易走偏，rewind 很有价值。

### 3. 探索性实现
你不确定方案是不是对。

### 4. 你打算先试再说
checkpoint 很适合支持这种探索式工作。

## 十二、什么时候不能只靠 checkpoint
### 1. Bash 会做大量文件副作用

### 2. 你准备做不可逆外部动作
例如：
- 推远程
- 改数据库
- 触发部署

### 3. 多人或多会话并发改同一套文件

### 4. 你需要长期、正式、可共享的回滚点

这些场景应依赖：
- Git
- branch
- commit
- PR

## 十三、checkpoint 和 permissions / automation 一起用时尤其重要
前面章节已经讲过：
- auto mode
- hooks
- subagents
- routines

这些都会让 Claude 更主动。

Claude 越主动，checkpoint 的价值越高。

因为它给你一个最关键的心理保障：

**我可以放手让它做，但一旦走偏，还有 session 级回退手段。**

## 十四、一个很稳的实战策略
推荐把恢复策略分三层。

### 第一层：checkpoint
用于快速试错。

### 第二层：Git branch / commit
用于阶段性稳定点。

### 第三层：PR / remote history
用于团队协作和长期追溯。

这样你就不会把所有恢复期望都压在一个机制上。

## 十五、这章真正要建立的“风险分级”
最推荐的判断方式是：

### 低风险探索
主要靠 checkpoint。

### 中风险多文件改动
checkpoint + Git branch。

### 高风险外部副作用
checkpoint 只是辅助，主恢复手段必须是版本控制或系统级回滚。

## 十六、这章和前几章其实是闭环
从第 8 章权限体系开始，到第 18 章自动化，你已经学到：
- 怎么让 Claude 更主动
- 怎么让它跑更久
- 怎么让它自己做更多事

而第 19 章回答的是：

**那如果它做偏了，你怎么安全撤回来。**

没有这一章，前面的“放手自动化”会变得心理上很难落地。

## 十七、和前后章节的关系
- 第 18 章讲的是自动触发和长期运行
- 第 19 章讲的是恢复与回滚安全网
- 第 20 章会继续讲：如果配置没生效、行为不对，应该怎么调试和排障

## 学完标准
- 你知道 checkpoint 只追踪 Claude 编辑工具，不追踪 Bash 文件副作用
- 你知道 `/rewind`、session 恢复、Git 回滚三者不是一回事
- 你知道什么时候可以主要依赖 checkpoint，什么时候必须上 Git
- 你已经形成“大改动前心里有恢复策略”的习惯
