# Claude Code AI 摘抄：单兵作战核心

这篇用来替代“自己一篇篇读完再摘抄”的步骤。它只保留需要反复复习、真正会影响使用水平的核心判断、模板和反模式。完整细节仍回到原 31 篇查。

## 1. 最重要的一句话

Claude Code 的高手差距，不在会多少命令，而在能不能把任务变成稳定的执行系统：

```text
清楚的上下文 + 明确的边界 + 可验证的结果 + 可沉淀的流程
```

## 2. 四个执行单元

所有任务都先归类到这四种之一。

| 单元 | 本质 | 你要说的话 |
| --- | --- | --- |
| Explore | 调查事实 | 先不要改代码，找证据 |
| Plan | 收敛方案 | 先不要实现，比较方案、风险和验证 |
| Implement | 执行改动 | 按方案实现，保持边界，跑验证 |
| Review | 独立审查 | 不要复述改动，只找问题 |

最常用的链路：

```text
Explore -> Plan -> Implement -> Review
```

小任务可以：

```text
Implement -> Review
```

Bug 任务最好：

```text
Explore -> Implement -> Review
```

长任务必须：

```text
Plan -> 分阶段 Implement -> 阶段 Review -> Handoff
```

## 3. 单兵能力分层

不要把所有东西都塞进 memory、skill 或 `CLAUDE.md`。

| 层级 | 解决的问题 | 记忆句 |
| --- | --- | --- |
| `CLAUDE.md` | 长期稳定事实和规则 | 每次都该知道 |
| memory | 反复纠偏后的偏好或事实 | 需要记住但要检查 |
| skill | 按场景触发的流程 | 重复流程按需加载 |
| hook | 必须发生或必须阻止的动作 | 不靠模型自觉 |
| permissions | 工具和命令边界 | 危险动作先控制 |
| settings | 默认模式和配置 | 系统行为配置 |
| MCP | 外部工具和数据 | Claude 能连什么 |
| plugin | 打包安装扩展组件 | 能力集合和分发 |

核心判断：

```text
事实放 CLAUDE.md。
流程放 skill。
强制动作放 hook。
外部连接放 MCP。
组合分发放 plugin。
```

## 4. `CLAUDE.md` 的摘抄

`CLAUDE.md` 不是越长越好。它应该像项目常识卡，不像操作手册。

### 应该放

- 项目长期稳定事实
- 架构约束
- 常用命令
- 最低验证要求
- 风险目录
- 禁止事项
- 代码风格中不容易从文件看出的规则

### 不应该放

- 临时任务
- 长流程
- 大段教程
- 偶发经验
- 未来可能变化的猜测
- 所有插件和 skill 的说明

摘抄句：

```text
CLAUDE.md 是稳定制度层，不是杂物间。
```

## 5. Skill 的摘抄

Skill 的价值不是“多一个命令”，而是把重复流程变成按需加载的能力。

### 最值得做的个人 skills

| Skill | 用途 |
| --- | --- |
| `catchup` | 接手仓库、分支、PR、陌生模块 |
| `bugfix` | 复现、定位、修复、验证 |
| `review` | 独立审查风险和测试缺口 |
| `handoff` | 长任务交接 |
| `release` | 发布前检查 |

### 判断题

如果一段说明满足这三个条件，就适合做 skill：

- 以后会重复用
- 不是每次会话都需要
- 内容比一句偏好更长，且有步骤

如果只是“这个项目用 pnpm”，放 `CLAUDE.md` 或 memory，不要做 skill。

摘抄句：

```text
Skill 是按需流程，不是项目事实仓库。
```

## 6. Hook 的摘抄

Hook 的本质是确定性控制。

适合 hook 的事情：

- 提交前必须跑检查
- 改敏感文件必须提醒或阻断
- 任务完成后通知
- 工具调用前做额外权限判断
- 停止前追加固定检查

不适合只写在 prompt 里的事情：

- 禁止危险命令
- 提交前必须测试
- 不能改某些目录
- 关键文件变更必须说明理由

摘抄句：

```text
必须发生或必须阻止的事，不要靠 Claude 记住，交给 hook 或 permissions。
```

## 7. MCP / Plugin 的摘抄

第一阶段不要研究生态大全，只记边界。

### MCP

MCP 让 Claude 连接外部工具和数据。

适合：

- GitHub issue / PR / Actions
- Sentry / 日志 / 监控
- 数据库
- Figma / Notion / Slack / Google Drive
- 内部系统 API

不适合：

- 写行为规范
- 替代 skill
- 替代 hook
- 替代本地测试

### Plugin

Plugin 是扩展能力的打包方式，可以包含 skills、agents、hooks、MCP 等。

第一阶段只需要会：

- 装官方插件
- 查可用命令
- 判断是否值得保留
- 不乱装

优先记住：

```text
Context7 降低 API 幻觉。
GitHub 打通 issue / PR / Actions。
Playwright 做前端流程验证。
Sentry 查线上错误。
Code Review 做独立审查。
```

## 8. 上下文摘抄

上下文是预算，不是垃圾桶。

### 好输入

- 明确文件范围
- 明确任务目标
- 明确不做什么
- 明确验证方式
- 给出现象、日志、错误、diff

### 坏输入

- 大段背景故事
- 模糊说“你看看”
- 让 Claude 猜入口
- 一次塞太多互斥目标
- 会话已经混乱还继续硬拖

摘抄句：

```text
少给背景故事，多给可消费材料。
```

## 9. `/compact` 和重启摘抄

`/compact` 是压缩历史，不是清空污染。

### 用 compact

- 任务仍然清楚
- 只是上下文太长
- 需要保留连续性

### 重开 session

- 连续纠偏还失败
- 引用过期信息
- 开始假装完成
- 目标已经变了
- 无关历史太多

### 重启前要生成 handoff

handoff 至少包含：

- 当前目标
- 已完成内容
- 关键决策
- 已验证内容
- 失败尝试
- 待办
- 下一步先读哪些文件

摘抄句：

```text
长任务靠交接物保持连续，不靠一个 session 硬撑到底。
```

## 10. 起手模板摘抄

### 探索仓库

```text
先不要改代码。
请给我这个仓库的工作地图：
1. 目录结构和关键入口
2. 主流程和核心模块
3. 构建、运行、测试方式
4. 风险最高的目录或文件
5. 我要改 <功能> 时应该先读哪些文件

每个判断都给出文件依据。
```

### 需求计划

```text
先不要实现。
基于当前仓库，给出这个需求的 2 到 3 种方案：<需求>

每个方案写清：
1. 改哪些模块
2. 风险是什么
3. 怎么验证
4. 哪个方案最稳，为什么
```

### 小需求实现

```text
实现这个需求：<需求>

要求：
1. 先读相关文件
2. 保持现有接口兼容
3. 只做必要改动
4. 跑相关测试或给出最小验证
5. 最后汇报改动、验证、风险
```

### Bug 排查

```text
先不要修。
排查这个 bug：<现象>

请输出：
1. 相关入口和调用链
2. 最可能的 3 个根因
3. 每个根因的证据
4. 最小复现路径
5. 推荐修复和验证方式
```

### Code Review

```text
不要复述改动。
把自己当 reviewer，只找问题：
1. 行为回归风险
2. 测试缺口
3. 接口、权限、数据一致性问题
4. 无关改动或过度设计
5. 会卡 PR 的意见

按严重程度排序，给出证据。
```

### Handoff

```text
请生成 handoff 文档：
1. 当前目标
2. 已完成内容
3. 关键决策
4. 已验证内容
5. 失败尝试
6. 待办
7. 新 session 下一步先读哪些文件
```

## 11. 单兵作战最小系统

先搭 5 样东西：

1. 全局 `~/.claude/CLAUDE.md`
2. 项目 `CLAUDE.md`
3. `catchup` / `bugfix` / `review` 三个 skills
4. `Notification` 和提交边界校验两个 hooks
5. 一套固定 prompt 模板

不要一开始就追求：

- 大量插件
- 自建 MCP server
- 多代理团队
- 企业治理
- 自动化全流程

摘抄句：

```text
先把主工作流打磨稳，再扩展能力面。
```

## 12. 插件优先级摘抄

只按真实开发价值排序：

1. Context7：查当前版本文档，减少 API 幻觉
2. GitHub：issue、PR、Actions、仓库信息
3. Playwright：浏览器流程验证和截图
4. Code Review：独立 reviewer
5. Commit Commands：commit / PR 描述
6. Sentry：线上错误
7. Vercel：部署排障
8. Linear：任务系统

装插件前先问：

```text
它是否服务我每周都会发生的工作流？
```

如果答案是否，就先别装。

## 13. 高频反模式摘抄

| 反模式 | 代价 | 替代动作 |
| --- | --- | --- |
| 上来就实现 | 容易改错方向 | 先 Explore / Plan |
| 没有验证路径 | 结果不可控 | prompt 里写明测试 |
| 长会话硬拖 | 上下文污染 | 阶段复盘 / compact / handoff |
| `CLAUDE.md` 过长 | 每次加载噪音 | 流程拆 skill |
| skill 过大 | 调用后污染上下文 | 主文件短，细节放 supporting files |
| 靠 memory 管规则 | 不稳定 | 稳定事实进 `CLAUDE.md` |
| 靠 prompt 管危险动作 | 会忘 | permissions / hooks |
| 插件乱装 | 工具选择混乱 | 只保留高频插件 |
| 实现和审查混合 | 漏风险 | 单独 Review |
| 没有交接物 | 长任务断档 | 写 handoff |

## 14. 阶段学习路线

### 第一阶段：单兵稳定交付

必须熟练：

- Explore / Plan / Implement / Review
- `CLAUDE.md`
- memory
- permissions
- hooks 基础
- bugfix / review / handoff
- 最小测试闭环

### 第二阶段：单兵扩展能力

开始掌握：

- Skills 进阶
- Context7 / GitHub / Playwright / Sentry
- MCP 的使用边界
- Plugins 的安装和筛选
- Subagents 做高噪音任务

### 第三阶段：团队专家

以后再学：

- 团队共享 `CLAUDE.md`
- 项目级 `.mcp.json`
- CI/CD 集成
- 权限治理
- Agent Teams
- 企业插件分发

## 15. 最小复习清单

每次用 Claude Code 前问：

- 这次是 Explore、Plan、Implement 还是 Review
- 我给的上下文是否最小但足够
- 我有没有写清不做什么
- 验证方式是否明确
- 这个经验要不要沉淀

每次任务结束问：

- 哪些事实应该进 `CLAUDE.md`
- 哪些偏好应该进 memory
- 哪些流程应该做成 skill
- 哪些强制动作应该交给 hook
- 哪些外部数据源值得接 MCP 或插件

