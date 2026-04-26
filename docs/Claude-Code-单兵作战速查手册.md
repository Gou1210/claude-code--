# Claude Code 单兵作战实操速查手册

这篇的用法不是阅读，而是照着做。每一节都尽量给你：什么时候用、复制什么、验收什么、沉淀到哪里。

主线：先把 Claude Code 用成稳定的个人开发系统，再接入 Skills、Hooks、MCP、Plugins。团队协作、企业治理、CI/CD 深度集成先不展开。

## 0. 先照做：30 分钟把一个仓库接入 Claude Code

适用：你刚打开一个新项目，或者之前一直随便用 Claude Code，没有系统配置。

### 第 1 步：让 Claude 建仓库地图

```text
先不要改代码。
请给我这个仓库的工作地图：
1. 目录结构和关键入口
2. 主业务流程和核心模块
3. install / dev / test / lint / build 命令
4. 哪些目录或文件改动风险最高
5. 如果我要修 bug、加功能、补测试，分别应该先读哪些文件

要求：
- 每个判断都给出文件证据
- 不要猜测不存在的命令
- 不要修改任何文件
```

验收：你应该拿到一份能回答“这个项目怎么跑、怎么测、哪里危险”的地图。

### 第 2 步：生成项目 `CLAUDE.md`

```text
基于刚才的仓库地图，起草项目 CLAUDE.md。
只写长期稳定、几乎每次会话都应该知道的事实。

必须包含：
1. 项目概览
2. 关键目录
3. 常用命令
4. 验证要求
5. 风险区域
6. 禁止事项

不要写：
- 长流程
- 临时任务
- 今天这个需求
- 大段教程
```

可直接用这个骨架：

```md
# Project Guide

## Overview
- This project ...

## Key Directories
- `src/`: ...
- `tests/`: ...

## Commands
- install: `<command>`
- dev: `<command>`
- test: `<command>`
- lint: `<command>`
- build: `<command>`

## Validation
- For code changes, run `<command>`.
- For UI changes, verify `<flow>`.

## Risk Areas
- `<path>`: ...

## Rules
- Follow existing patterns.
- Do not modify generated files.
- Do not introduce new dependencies unless asked.
```

验收：`CLAUDE.md` 应该短、硬、稳定。超过 150 行通常就开始变成杂物间。

### 第 3 步：检查 memory

在 Claude Code 里执行：

```text
/memory
```

该删的：

- 过时命令
- 错误项目事实
- 只适用于某一次任务的临时信息

该记的：

```text
remember that this repo uses <package-manager>, not <wrong-one>
remember that tests require <service> before running
remember that <directory> is generated and should not be edited manually
```

验收：memory 里只保留跨任务仍然有效的事实或偏好。

### 第 4 步：设置最小权限

先在 Claude Code 里执行：

```text
/permissions
```

单兵开发的实用思路：

```text
allow:
- git status
- git diff
- rg / rg --files
- package-manager test/lint/build 的安全命令

ask:
- git commit
- git push
- 安装依赖
- 启动长期服务

deny:
- 读取 .env、secrets、私钥
- 删除大量文件
- 强制 reset / checkout 覆盖改动
```

可参考规则形态：

```text
Bash(git status)
Bash(git diff:*)
Bash(rg:*)
Bash(pnpm test:*)
Bash(pnpm lint:*)
Bash(pnpm build:*)
```

验收：常规搜索和测试不反复打断，高风险动作仍要确认。

### 第 5 步：装最小插件组

在 Claude Code 里执行：

```text
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
/plugin install playwright@claude-plugins-official
/plugin install code-review@claude-plugins-official
/plugin install commit-commands@claude-plugins-official
/reload-plugins
```

验证 5 句：

```text
给我按当前文档解释这个项目使用的主要框架，use context7
```

```text
看看这个仓库最近失败的 GitHub Actions，并指出最可能的修复入口
```

```text
打开 localhost:3000，跑一遍核心页面流程并截图
```

```text
/code-review
```

```text
基于当前改动生成 commit message 和 PR 描述草稿
```

验收：每个插件都至少跑过一次真实动作。没用起来的插件先卸载或禁用。

## 1. 任务入口判断表

| 你现在的情况 | 起手动作 | 不要做什么 |
| --- | --- | --- |
| 不熟悉仓库 | Explore | 不要直接实现 |
| 需求不清 | Plan | 不要让 Claude 猜验收标准 |
| 小需求明确 | Implement | 不要顺手重构 |
| Bug 根因不清 | Explore bug | 不要直接“修一下” |
| 代码太乱 | Refactor plan | 不要混入新需求 |
| 改完了 | Review | 不要让实现者自己糊弄过关 |
| 会话很长 | Handoff / compact | 不要硬拖 |
| 想试一条不确定路线 | Checkpoint / `/rewind` | 不要靠手工复制文件回退 |
| 搜索、审查、日志噪音太大 | Subagent | 不要把主会话塞脏 |
| 想并行试方案或拆审查 | Worktree / Writer-Reviewer | 不要在同一工作区混改 |
| 总是重复同一流程 | Skill | 不要每次复制长 prompt |
| 必须阻止某事 | Hook / permission | 不要只写提示词 |
| 要查外部系统 | MCP / plugin | 不要把外部事实靠猜 |

## 2. 小需求：从指令到验收

适用：改动 1 到 3 个文件，目标明确。

### 复制这个

```text
实现这个需求：<需求>

边界：
1. 只改和该需求直接相关的文件
2. 保持现有公开接口和用户行为兼容
3. 不引入新依赖
4. 不顺手重构无关代码

执行：
1. 先读相关文件并简述改动点
2. 再实现
3. 跑相关测试或给出最小手动验证步骤
4. 最后只汇报：改动、验证、风险
```

### 你要盯的点

- Claude 有没有先读文件
- 有没有扩大改动范围
- 有没有给验证结果，而不是只说“应该可以”
- 有没有改到生成文件、锁文件、配置文件

### 结束后沉淀

- 项目长期事实：进 `CLAUDE.md`
- 个人偏好：进 memory
- 重复流程：做 skill
- 必须检查：做 hook

## 3. 大需求：先锁方案再开工

适用：多模块、多文件、涉及状态、接口、权限、数据结构。

### 复制这个

```text
先不要实现。
需求：<需求>

请给出 2 到 3 个方案，每个方案必须写：
1. 需要改哪些模块或文件
2. 调用链或数据流怎么变
3. 风险点
4. 验证方式
5. 回退方式

最后推荐一个最稳方案，并说明为什么。
不要写代码。
```

### 方案通过后复制这个

```text
按方案 <A/B/C> 实现。

执行约束：
1. 分小步改，每步保持可验证
2. 不改变未声明的行为
3. 相关测试失败时先停下来解释，不要盲修
4. 完成后跑验证并做 self-review
```

### 验收标准

- 方案里有明确“不做什么”
- 知道哪些文件会改
- 知道怎么证明做完了
- 知道失败后怎么回退

## 4. Bug：排查剧本

适用：现象明确，但根因不清。

### 第 1 轮只查不修

```text
先不要修。
Bug 现象：<现象>
已知信息：<日志/截图/复现步骤/报错>

请排查：
1. 找入口、调用链、状态变化
2. 给出最多 3 个根因假设
3. 每个假设必须有文件证据、日志证据或复现证据
4. 给出最小复现路径
5. 推荐最小修复方案
```

### 第 2 轮再修

```text
按推荐方案修复。

要求：
1. 优先补复现测试；如果没有测试框架，给最小手动验证步骤
2. 只修根因，不做无关重构
3. 修完跑相关测试
4. 汇报根因、修复点、验证结果、剩余风险
```

### 线上 bug 加插件

Sentry：

```text
/seer 最近 24 小时最严重的 5 个错误，按影响用户数和发生频率排序
```

GitHub：

```text
看看最近一次失败的 Actions，找出和这个 bug 相关的失败日志和代码入口
```

Playwright：

```text
打开 localhost:3000，按这个复现步骤操作：<步骤>。截图并说明实际结果。
```

## 5. Review：把 Claude 当第二双眼睛

适用：你或 Claude 已经改完。

### 当前工作区 review

```text
不要总结优点。
请审查当前工作区 diff，只找问题：
1. 行为回归风险
2. 测试缺口
3. 接口、权限、数据一致性问题
4. 无关改动
5. 过度设计
6. 如果这是 PR，你会卡住的意见

按严重程度排序，每条给文件证据和建议修法。
```

### PR review

```text
审查这个 PR：<PR 链接或编号>

要求：
1. 先理解需求和改动范围
2. 再查 diff
3. 只输出需要作者处理的问题
4. 不要输出泛泛建议
5. 每条问题都要说明为什么可能造成 bug 或维护风险
```

### Review 输出格式

```text
Severity: high / medium / low
File:
Problem:
Why it matters:
Suggested fix:
Test gap:
```

## 6. 重构：不改变行为的操作法

适用：代码能跑，但复杂、重复、难维护。

### 先做保护

```text
先不要重构。
请分析 <范围> 的现有行为，并找出重构前最应该补的保护测试。

输出：
1. 必须保持不变的行为
2. 当前已有测试
3. 缺失的高价值测试
4. 最小测试补齐计划
```

### 再重构

```text
执行最小重构。

要求：
1. 不改变外部行为
2. 不引入新依赖
3. 不改 public API
4. 每次只做一种重构动作
5. 跑相关测试
6. 最后列出行为不变的证据
```

### 适合用 `/simplify`

```text
/simplify 只整理最近改过的文件，不改变行为，不扩大改动面。先给计划，再改。
```

## 7. 长任务：阶段推进模板

适用：超过 1 小时、多阶段、多次验证。

### 开始前

```text
这是一个长任务：<目标>

先不要实现。
请拆成 3 到 6 个阶段，每个阶段写：
1. 目标
2. 改动范围
3. 验证方式
4. 完成标志
5. 失败时怎么止损

同时指出哪些阶段适合单独 review。
```

### 每阶段结束

```text
做阶段复盘：
1. 完成了什么
2. 改了哪些文件
3. 跑了哪些验证
4. 还有哪些风险
5. 下一阶段第一步
6. 是否需要 compact 或 handoff
```

### Handoff

```text
生成 handoff 文档，供新 session 继续：
1. 当前目标
2. 已完成内容
3. 关键决策
4. 文件改动概览
5. 已验证内容
6. 失败尝试
7. 未完成事项
8. 下一步先读哪些文件
```

重开判断：

- 连续两次纠偏失败
- Claude 引用过期信息
- 开始假装完成
- 目标已经变了
- 上下文里混入太多无关内容

## 8. Checkpoints 与 `/rewind`：基础安全网

适用：你要让 Claude 尝试一条不确定路线，或者已经改乱了，想回到之前状态。

### 你要记住的边界

- checkpoint 是 Claude Code 的试错安全网，不是 git 的替代品。
- git 负责长期版本历史；checkpoint 负责会话中的短周期回退。
- `/rewind` 适合撤回一次错误方向、恢复对话和代码状态、重新选择方案。
- 高风险改动前仍然要看 `git status` 和 `git diff`。

### 试错前 prompt

```text
这条方案不确定。
请先明确当前状态和回退点：
1. 当前目标
2. 计划改哪些文件
3. 如果失败，应该回退到哪里
4. 成功的验证标准

确认后再开始改。
```

### 改乱后 prompt

```text
当前实现方向不对。
请停止继续修改，先总结：
1. 从哪个步骤开始偏离
2. 哪些改动应该保留
3. 哪些改动应该回退
4. 是否适合用 /rewind 回到上一个稳定点
```

### 什么时候用 `/rewind`

- Claude 选错方案，继续修会越修越歪
- 改动还没形成有价值的 diff
- 你想恢复代码和对话到某个较早节点
- 你要重新走 Plan，而不是在错误实现上补丁

### 什么时候用 git

- 已经产生一组有价值的文件改动
- 需要精确保留部分文件、丢弃部分文件
- 已经 commit 或准备 commit
- 需要和远端或其他分支对齐

## 9. Subagents：单兵上下文隔离工具

Subagents 不是团队专属。单兵最常用的是把“高噪音任务”从主会话分出去，主会话只拿结论。

### 适合派 subagent

| 场景 | 目标 |
| --- | --- |
| 大范围代码搜索 | 找入口、调用链、影响面 |
| 独立 review | 不受实现过程污染，只看风险 |
| 长日志分析 | 从大量输出里提取根因 |
| 多方案比较 | 并行调查不同方案 |
| 测试失败排查 | 专门追失败链路 |

### 不适合派 subagent

- 你下一步立刻依赖它的结果，等它会阻塞主线
- 任务边界模糊，不知道让它查什么
- 改动高度耦合，容易和主会话冲突
- 只是为了“显得高级”

### 调查型 prompt

```text
请作为独立调查员，只调查不要改代码。
目标：<问题>

输出：
1. 相关入口和调用链
2. 关键文件
3. 最可能的根因或改动点
4. 证据
5. 主会话下一步应该读哪些文件
```

### Review 型 prompt

```text
请作为独立 reviewer 审查当前 diff。
不要参考实现过程，只看最终改动。

重点：
1. 行为回归
2. 测试缺口
3. 权限、安全、数据一致性
4. 无关改动
5. 需要阻断的 PR 意见
```

### 使用原则

- 主会话负责决策和整合。
- Subagent 负责调查、审查、提取证据。
- 不要让多个 agent 改同一批文件。
- subagent 返回后，先看证据，再决定是否采纳。

## 10. Worktree 并行与 Writer/Reviewer 分离

Worktree 并行是单兵提效工具，不是团队专属。它适合同时试两个方案，或者把实现和审查彻底隔离。

### 适合用 worktree

- 两个方案都可能成立，想并行试
- 一个修 bug，一个补测试，互不影响
- 想让 reviewer 在干净上下文里审查
- 当前分支太乱，需要隔离一个实验分支

### 不适合用 worktree

- 你不熟 git worktree
- 改动都集中在同一批文件
- 任务很小，开 worktree 成本大于收益
- 没有清晰命名和收尾习惯

### 基本命令

```bash
git worktree add ../project-auth-fix -b auth-fix
git worktree list
git worktree remove ../project-auth-fix
```

如果 Claude Code 支持对应入口，也可以用：

```bash
claude --worktree feature-auth
```

具体命令以当前 `claude --help` 和本机版本为准。

### Writer / Reviewer 分离

Writer 会话：

```text
你是 writer。
按方案实现，保持最小改动。
不要做最终审查，只在结束时给 diff 概览和验证结果。
```

Reviewer 会话：

```text
你是 reviewer。
不要参考 writer 的解释，先读取当前 diff。
只找问题：回归、测试缺口、接口破坏、权限和数据风险。
```

### 收尾检查

```text
请帮我收尾这个 worktree：
1. 当前分支和主分支差异
2. 是否还有未提交改动
3. 哪些改动值得保留
4. 是否可以合并回主工作区
5. 合并前必须跑哪些验证
```

## 11. 三个最小 Skills，直接照抄

放置位置：

```text
~/.claude/skills/<skill-name>/SKILL.md
```

或当前项目：

```text
.claude/skills/<skill-name>/SKILL.md
```

### `catchup`

```md
---
name: catchup
description: Build a concise working map for an unfamiliar repo, branch, PR, or module before making changes.
---

When catching up:
1. Do not edit files.
2. Identify entry points, core flow, tests, commands, and risk areas.
3. Prefer `rg` and existing docs before broad reading.
4. Summarize only facts with file evidence.
5. End with the first 3 files to read for the requested task.

Output:
- What this area does
- Key files
- Main flow
- How to run or test it
- Risks
- Next files to read
```

### `bugfix`

```md
---
name: bugfix
description: Diagnose and fix a bug with evidence, reproduction, minimal change, and validation.
---

Bugfix process:
1. Do not edit before identifying the likely root cause.
2. Collect symptoms, logs, reproduction steps, and related code paths.
3. Produce up to 3 root-cause hypotheses with evidence.
4. Prefer a failing test or minimal reproduction before fixing.
5. Apply the smallest root-cause fix.
6. Run related validation.

Final output:
- Root cause
- Files changed
- Validation
- Remaining risk
```

### `review`

```md
---
name: review
description: Review code changes as a strict reviewer. Focus on regressions, test gaps, boundaries, and maintainability.
---

Review rules:
1. Do not summarize the change first.
2. Find only actionable issues.
3. Prioritize behavior regressions, data loss, security, permissions, and test gaps.
4. Cite file evidence.
5. If no blocking issues exist, say so and list residual risk.

Output format:
- Severity:
- File:
- Problem:
- Why it matters:
- Suggested fix:
- Test gap:
```

验收：你应该能用 `/catchup`、`/bugfix`、`/review` 触发稳定流程，而不是每次复制长 prompt。

## 12. 最小 Hooks：先做两个

Hooks 不要一上来搞复杂。单兵先做通知和边界提醒。

### Notification hook

用途：长任务完成、需要你确认、等待输入时提醒。

配置位置通常在：

```text
~/.claude/settings.json
.claude/settings.json
.claude/settings.local.json
```

示意配置：

```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell -NoProfile -Command \"[console]::beep(800,200)\""
          }
        ]
      }
    ]
  }
}
```

### 敏感文件提醒

用途：改配置、密钥、迁移、锁文件时提高警惕。

示意配置：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "powershell -NoProfile -File .claude/hooks/check-sensitive.ps1"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/check-sensitive.ps1` 的思路：

```powershell
$inputJson = [Console]::In.ReadToEnd()
$patterns = @(".env", "secrets", "migration", "package-lock.json", "pnpm-lock.yaml")
foreach ($p in $patterns) {
  if ($inputJson -like "*$p*") {
    Write-Error "Sensitive path matched: $p. Please confirm this change is intentional."
    exit 2
  }
}
exit 0
```

注意：hook 脚本要先在你自己的项目里小范围试，不要一上来全局启用复杂拦截。

## 13. MCP / Plugins 实操选择

### 最小插件组合

普通开发：

```text
Context7 + GitHub + Playwright + Code Review + Commit Commands
```

生产排障：

```text
GitHub + Sentry + Playwright + Context7
```

前端 / SaaS：

```text
Context7 + GitHub + Playwright + Vercel + Frontend Design
```

### 什么时候用 Context7

```text
给我按 Next.js 15 当前文档实现 middleware 鉴权，use context7。
要求指出用到的 API 来自哪个文档结论。
```

### 什么时候用 GitHub

```text
查看当前仓库最近失败的 GitHub Actions。
请定位失败 job、关键日志、最可能代码入口，并给修复计划。
```

### 什么时候用 Playwright

```text
打开 localhost:3000。
按这个流程验证：<流程>。
要求截图关键页面，并说明实际结果和预期是否一致。
```

### 什么时候用 Sentry

```text
查看最近 24 小时影响最大的错误。
按用户影响、频率、最新发生时间排序，并关联可能代码入口。
```

### MCP 判断

用 MCP 的场景：

- 外部系统里有真实数据
- 靠手动复制信息容易漏
- 需要 Claude 反复查询同一系统
- 查询结果会改变实现方案

不用 MCP 的场景：

- 一次性查个网页就够
- 本地文件已经包含事实
- 外部系统权限风险很高
- 你还没形成稳定工作流

## 14. 权限配置实操

### 个人开发推荐策略

| 动作 | 策略 |
| --- | --- |
| 读项目文件 | allow |
| `rg`、`git status`、`git diff` | allow |
| 跑 test / lint / build | allow 或 ask |
| 安装依赖 | ask |
| 启动 dev server | ask |
| commit | ask |
| push | ask |
| 删除文件 | ask 或 deny |
| 读 `.env` / secrets | deny |
| reset / force push | deny |

### 常见 allowlist

```text
Bash(git status)
Bash(git diff:*)
Bash(rg:*)
Bash(rg --files:*)
Bash(pnpm test:*)
Bash(pnpm lint:*)
Bash(pnpm build:*)
Bash(npm test:*)
Bash(npm run lint:*)
Bash(npm run build:*)
```

### 常见 denylist

```text
Read(.env)
Read(.env.*)
Read(*secret*)
Bash(git reset --hard:*)
Bash(git push --force:*)
Bash(rm -rf:*)
```

规则以当前版本 `/permissions` 显示为准。遇到不确定命令，宁可 ask。

## 15. 你的日常工作流

### 每天开始

```text
请基于当前 git 状态和最近改动，告诉我：
1. 当前分支在做什么
2. 工作区有哪些未提交改动
3. 哪些改动可能有风险
4. 下一步最小行动是什么
不要改文件。
```

### 开始写代码前

```text
先确认任务边界：
1. 目标是什么
2. 不做什么
3. 会改哪些文件
4. 怎么验证
5. 最大风险是什么
```

### 写完后

```text
请做收尾检查：
1. 当前 diff 是否只包含本任务需要的改动
2. 是否有调试代码、无关格式化、临时文件
3. 是否跑过验证
4. 是否需要补测试
5. commit message 应该怎么写
```

### 提交前

```text
请基于当前 diff 生成：
1. commit message
2. PR 描述
3. 测试说明
4. 风险说明
5. reviewer 应重点看的文件
```

## 16. 反模式和立即修正

| 反模式 | 立即修正 |
| --- | --- |
| “帮我实现一下” | 改成“先读文件，列改动点和验证方式” |
| Claude 猜架构 | 要求“每个判断给文件证据” |
| 改动越来越大 | 要求“停止实现，列出已扩大范围的改动” |
| 测试没跑 | 要求“给出已跑命令和结果；没跑就说明原因” |
| 会话变糊 | 生成 handoff，新开 session |
| 每次复制同一段 prompt | 做成 skill |
| 反复提醒不要做某事 | 写 permission 或 hook |
| 插件装完不用 | 卸载或禁用，减少工具面 |

## 17. 后续学习路线

第一阶段：单兵稳定交付

- 熟练 Explore / Plan / Implement / Review
- 写好全局和项目 `CLAUDE.md`
- 会管理 memory
- 会配置基础 permissions
- 会用 checkpoints / `/rewind` 控制试错成本
- 会用 subagents 隔离调查和 review
- 会用 `catchup` / `bugfix` / `review`

第二阶段：单兵扩展能力

- Skills supporting files 和 scripts
- Hooks 生命周期
- Context7 / GitHub / Playwright / Sentry
- MCP scope 和 `.mcp.json`
- Worktree 并行和 Writer/Reviewer 分离

第三阶段：团队专家

- 团队共享规范
- CI/CD 集成
- 权限治理
- 项目级 MCP
- Agent Teams
- 插件分发

## 18. 英文资料补充候选

这一节先不展开，只保留后续用 AI 深挖时的线索。学完本手册后，可以让 AI 基于这些主题和来源补成第二版。

### 1. Verification-first 工作流

关键信息：不要只让 Claude “写完”，要让它带着测试、lint、build、截图、CLI 输出、预期结果一起自检。可以补成不同场景的验证菜单：后端、前端、CLI、文档、数据库迁移。

关键词：

```text
Claude Code verification loop
Claude Code best practices test lint screenshot build
```

来源：

- <https://code.claude.com/docs/en/best-practices>
- <https://support.claude.com/en/articles/14554000-claude-code-power-user-tips>

### 2. 让 Claude 先 interview 你

关键信息：大需求不要一次性写超长 prompt。让 Claude 先追问需求、边界、edge cases、tradeoffs，再生成 `SPEC.md`，之后新 session 按 spec 实现。

关键词：

```text
Claude Code let Claude interview you SPEC.md
Claude Code planning mode requirements interview
```

来源：

- <https://code.claude.com/docs/en/best-practices>

### 3. `CLAUDE.md` 高级组织法

关键信息：`CLAUDE.md` 可以分层组织；根目录、home、子目录、本地文件承担不同职责。官方还支持用 `@path` 导入其他文件，适合把长内容拆开。

关键词：

```text
Claude Code CLAUDE.md import @path
Claude Code memory files CLAUDE.local.md
```

来源：

- <https://code.claude.com/docs/en/best-practices>

### 4. Hooks 进阶模式

关键信息：除了通知和敏感文件提醒，还可以研究 `PostToolUse` 自动格式化、`Stop` 阻止测试失败时结束、`PreCompact` 或 session hooks 注入上下文。注意 hooks 会自动执行命令，安全风险比普通 prompt 高。

关键词：

```text
Claude Code hooks PostToolUse Stop PreCompact
Claude Code deterministic control hooks security
```

来源：

- <https://code.claude.com/docs/en/hooks-guide>
- <https://docs.anthropic.com/en/docs/claude-code/hooks>

### 5. 不可信仓库、Hooks、MCP、Plugins 安全

关键信息：不要随便信任陌生仓库里的 `.claude/settings.json`、hooks、`.mcp.json`、第三方插件和 marketplace。重点关注自动执行命令、环境变量、API key、MCP server、插件权限。

关键词：

```text
Claude Code security hooks MCP plugins untrusted repositories
Claude Code CVE project files API token exfiltration
```

来源：

- <https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/>
- <https://github.com/trailofbits/skills>

### 6. 官方与社区 Skills 仓库

关键信息：后续学习 skill 时，不要只看文章，要看真实 skill 仓库怎么组织 `SKILL.md`、supporting files、scripts、description、allowed tools。

关键词：

```text
Claude Code skills examples GitHub
Claude Code SKILL.md supporting files scripts
```

来源：

- <https://github.com/anthropics/skills>
- <https://github.com/trailofbits/skills>
- <https://github.com/obra/superpowers>
