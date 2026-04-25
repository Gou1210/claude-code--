# 14. Hooks：生命周期自动化

## 重要程度
**A 级** - 强烈建议掌握。Hooks 是 Claude Code 真正进入“可控自动化”阶段的分水岭。

## 学习目标
- 理解 hooks 的本质是 deterministic control，而不是提示词建议
- 掌握 Claude Code 生命周期里有哪些 hook 事件、能拦什么、能加什么上下文
- 学会用 hooks 固化你真正希望“每次都发生”的行为

## 学什么
- Hooks guide
- Hooks reference
- hook 配置结构
- event lifecycle
- exit code 与 JSON 输出
- async hooks、HTTP hooks、prompt hooks、MCP tool hooks
- `/hooks` 调试和常见模式

## 你需要掌握
- Hooks 是在 Claude Code 生命周期特定节点自动执行的 shell 命令或相关扩展机制
- 它的核心价值不是“自动化一点事情”，而是 **保证某些事情一定发生**
- hooks 可以拦截用户提示、工具调用、停止时机、session 启停、compact、子代理结束等关键节点
- hook 比 skill 更强制，比 `CLAUDE.md` 更确定，比 permissions 更灵活

## 一、先抓住 hooks 的本质
官方对 hooks 的定义很清楚：

**hooks 是用户定义的命令，会在 Claude Code 生命周期的特定点自动执行。**

它最重要的关键词不是“automation”，而是：

**deterministic control**

也就是：
- 不是希望 Claude 想起来
- 不是寄希望于模型记住规则
- 而是系统级地把某个动作插进流程里

这和前面几个能力的差别非常大。

## 二、为什么 hooks 比提示词更可靠
你可以把规则写进：
- `CLAUDE.md`
- skill
- system prompt

但这些本质上都还是“让模型理解并自觉遵守”。

而 hook 的逻辑是：
- 到了某个时机
- 系统就执行你定义的命令
- 然后根据命令结果决定是否继续

所以：

**`CLAUDE.md` 是建议，hook 是硬插入流程。**

这也是为什么：
- 自动格式化
- 阻止改敏感文件
- 强制附加上下文
- 完成后通知

这类需求更适合用 hooks，而不是长篇提示词。

## 三、hooks 配在哪
截至 **2026-04-25**，官方文档说明 hooks 仍然配置在 settings 文件中：
- `~/.claude/settings.json`
- `.claude/settings.json`
- `.claude/settings.local.json`
- enterprise managed settings

基本结构大致是：

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

对于不需要 matcher 的事件，可以省略 matcher。

## 四、最核心的几个 hook 事件
官方事件很多，但你第一轮最该先掌握这些。

### `PreToolUse`
在 Claude 生成好工具参数之后、真正执行之前触发。

这是最强的一类 hook 之一，因为它能：
- 放行
- 拒绝
- 要求用户确认

适合：
- 拦 Bash
- 拦写文件
- 拦危险目录
- 做自定义权限逻辑

### `PostToolUse`
工具成功执行后触发。

适合：
- 编辑后格式化
- 写文件后 lint
- 执行后记录日志
- 给 Claude 补额外上下文

### `UserPromptSubmit`
用户提交 prompt 后、Claude 处理前触发。

适合：
- 做 prompt 校验
- 自动附加项目状态
- 阻止某类危险请求

### `Stop`
主 Claude 将要结束响应时触发。

适合：
- 做“结束前质量门”
- 发现没验证完时强制 Claude 继续

### `SubagentStop`
subagent 完成响应时触发。

适合：
- 子代理完成后做统一检查
- 防止它在未满足条件时直接停下

### `SessionStart`
新 session 或 resume 时触发。

适合：
- 自动加载最近 issue
- 自动附加 git 状态
- 自动带入开发上下文

### `SessionEnd`
session 结束时触发。

适合：
- 清理
- 记录
- 写统计

### `PreCompact`
compact 前触发。

适合：
- 先抽取关键结论
- 先写摘要

### `Notification`
Claude 需要权限或等待你输入时触发。

适合：
- 桌面提醒
- 声音提醒
- IM 提醒

## 五、matcher 是怎么工作的
官方说明：
- `matcher` 只适用于 `PreToolUse` 和 `PostToolUse`
- 它按工具名匹配，区分大小写
- 可用简单字符串、正则、`*`

例如：
- `Write`
- `Edit|Write`
- `Notebook.*`
- `*`

所以最常见的写法会是：
- 对所有 Bash 做一类检查
- 对 `Write|Edit` 做另一类处理

## 六、最常见、也最有收益的 hook 用法
### 1. 编辑后自动格式化
例如：
- 写完 `.ts` 自动跑 prettier
- 改完 `.go` 自动跑 gofmt

### 2. 修改敏感路径时阻止
例如：
- 阻止写 `infra/prod/`
- 阻止改 `secrets/`

### 3. 自动日志记录
例如：
- 所有 Bash 都记一份
- 所有写操作都留审计

### 4. 结束前质量门
例如：
- 如果还没跑测试，不许 stop
- 如果 lint 没过，不许 stop

### 5. 通知与提醒
例如：
- Claude 等你授权时弹桌面通知
- 长任务结束时提醒

## 七、最关键的返回机制：exit code
官方 hooks 有两层返回方式：简单版和高级版。

### 简单版：exit code
- `0`：成功
- `2`：阻断错误
- 其他非零：非阻断错误

这里最重要的是：

**exit code 2 是真正能参与流程控制的关键。**

不同事件对 `2` 的处理不同，但核心上就是：
- 某些事件会阻止继续
- 并把错误反馈给 Claude 或用户

## 八、高级版：JSON 输出
如果你只想做简单脚本，exit code 就够了。

但如果你要更精细控制，官方支持通过 `stdout` 返回结构化 JSON。

最常见的能力有：
- `continue: false`
- `stopReason`
- `systemMessage`
- 针对不同 hook 的 `hookSpecificOutput`

这意味着 hook 不只是“跑个脚本”，而是能真正参与控制 Claude 的流程。

## 九、`PreToolUse` 是最像“自定义权限系统”的钩子
官方为 `PreToolUse` 提供了专门控制字段：
- `allow`
- `deny`
- `ask`

也就是：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "..."
  }
}
```

所以你应该把它理解成：

**permissions 是内建权限系统，PreToolUse hook 是可编程权限层。**

它非常适合实现“组织自己的细粒度安全策略”。

## 十、`PostToolUse` 最适合做反馈型自动化
和 `PreToolUse` 不同，`PostToolUse` 发生时工具已经执行完了。

所以它更适合做：
- 补上下文
- 补提示
- 发现问题后让 Claude 再修

官方支持：
- `decision: "block"`
- `reason`
- `hookSpecificOutput.additionalContext`

这等于给了你一个能力：

**Claude 做完一件事后，你可以让外部脚本再给它一次机器反馈。**

这就是“自动 reviewer”式工作流的基础。

## 十一、`UserPromptSubmit` 是很多人低估的高价值点
这个事件很值钱，因为它发生在最前面。

适合做：
- prompt lint
- 敏感词拦截
- 自动附加任务上下文
- 自动插入当前分支、最近 diff、issue 信息

如果你希望：
- 用户一提某类需求
- 系统就自动补充相关背景

那这个 hook 很合适。

## 十二、`Stop` / `SubagentStop` 能做质量门
官方说明这两个事件都可以阻止 Claude 停止。

这意味着你可以实现：
- “如果没跑测试，就别停”
- “如果还有 TODO，就继续”
- “如果 reviewer hook 说有问题，就继续修”

这和很多工程系统里的 quality gate 很像。

所以 hooks 的定位不只是自动化脚本，更是：

**把工程质量门嵌进 Claude 的生命周期。**

## 十三、`SessionStart` 是注入上下文的好地方
官方支持在 `SessionStart` 输出额外上下文。

这很适合：
- 把最近 git 变更带入
- 把待处理 issue 列表带入
- 把项目当前状态带入

和 `CLAUDE.md` 的区别是：
- `CLAUDE.md` 放稳定规则
- `SessionStart` 注入动态上下文

这两者组合起来，效果非常强。

## 十四、执行细节里有三个很重要的事实
官方 reference 里有几条很容易被忽略，但实战很重要。

### 1. 默认超时
默认 60 秒，可配置。

### 2. 所有匹配 hook 并行执行
这意味着：
- 不要写互相依赖、靠顺序才能工作的 hook

### 3. 相同 hook 命令会自动去重
这有助于避免重复执行同一条命令。

## 十五、调试 hooks 时先做这几步
官方给的基础排查思路很实用：
- 用 `/hooks` 看是否已注册
- 检查 settings JSON 是否合法
- 确认 matcher 是否写对
- 确认脚本能独立运行

另外一个实用原则是：

**先写最小 hook，再慢慢加复杂逻辑。**

不要一上来就做一个既拦截又改写又发通知又写日志的超级钩子。

## 十六、什么时候 hook 比 skill 更适合
这是高频判断题。

### 用 skill 的情况
- 你希望 Claude 学会一个套路
- 只在相关任务时用
- 仍然由模型决定如何执行

### 用 hook 的情况
- 你希望某动作一定发生
- 你需要流程控制
- 你要拦截、放行、阻断、补上下文

一句话记忆：

**skill 是“给模型能力”，hook 是“给系统约束与自动化”。**

## 十七、什么时候 hook 比 permissions 更适合
permissions 适合静态规则，例如：
- 某命令 allow
- 某目录 deny

hook 更适合动态规则，例如：
- 只有改动某类文件时才额外校验
- 命令参数符合某模式才允许
- 根据当前分支决定可不可以继续

所以：

**permissions 是静态边界，hooks 是动态决策。**

## 十八、这一章真正的升级点
学到这里，你对 Claude Code 的理解应该从：
- “模型会不会记得做这件事”

升级成：
- “这个动作是靠模型自觉，还是靠系统保证”

真正成熟的工程做法，一般是：
- 规范写进 `CLAUDE.md`
- 高价值流程封成 skill
- 高噪音任务给 subagent
- 必须发生的事情写成 hook

## 十九、和前后章节的关系
- 第 13 章讲的是多代理协作
- 第 14 章讲的是生命周期自动化与确定性控制
- 第 15 章会进一步梳理扩展体系：MCP、plugins 与扩展边界

## 学完标准
- 你知道 hooks 的核心不是“脚本自动化”，而是 deterministic control
- 你知道 `PreToolUse`、`PostToolUse`、`UserPromptSubmit`、`Stop`、`SessionStart` 这些关键事件各适合做什么
- 你知道什么时候该用 hook，而不是继续往 `CLAUDE.md` 里塞规则
- 你至少能设计出一个 edit 后校验、一个通知、一个质量门型 hook
