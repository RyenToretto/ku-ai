# sediment-judgement.mdc 设计说明

## 设计意图

把「开发过程中产生的有意义内容」从「随手扔在某个 commit message 里」升级为「主动判断 → 系统化沉淀」。避免：

1. 同一个隐性约束被新代码反复违反（因为没沉淀进 rule）
2. 同一个 5+ 步骤的工作流每次都从头摸索（因为没沉淀进 skill）
3. 踩坑后没复盘，下次重复一遍（因为没沉淀进 dev-scenarios）

## 适用场景

- 所有项目，自动应用（`alwaysApply: true`）
- AI Agent 工作流是核心受益方：每次完成阶段性任务都触发一次"该不该沉淀"的判定
- 团队协作场景同样适用：把个人经验上升为团队资产

## 三种沉淀目标的判定边界

| 类型 | 触发条件 | 落地位置 | 同步目的地 |
|---|---|---|---|
| **rule** | 跨文件 / 框架硬约束 / 业务铁则 / 不沉淀必再犯 | `.cursor/rules/*.mdc` | `~/Desktop/ku-ai/rules/` |
| **skill** | 多步骤工作流 / MCP-SDK 调用链 / 5+ tool calls | `.agents/skills/*` 或 `~/.agents/skills/*` | `~/Desktop/ku-ai/skills/<name>/` |
| **dev-scenarios** | 首次操作 / 踩坑 / 多步流程 / 环境配置 / 破坏性变更 / 团队复用 | `docs/dev-scenarios/*.md` | 留在仓库内（不进 ku-ai） |

## 与 `ai-toolkit-docs.mdc` 的关系

`ai-toolkit-docs.mdc` 说"新增 rule / skill 必须同步到 ku-ai"，本规则说"什么时候要新增"。两者是「时机」与「动作」的搭配。

## 自检触发时机

强制在 4 个时机做一次判定，最低开销：30 秒思考 + 列在 PR / dev-scenarios 末尾「沉淀清单」小节。

## 与其他 rule 的关系

- `track-event-naming.mdc` 是埋点维度的沉淀子规则
- `git-commit-zh.mdc` 是 commit 文案沉淀
- `dev-scenarios.mdc` 是日志沉淀格式
