# Skill: session-handoff

> 上下文窗口接近极限时, 自动 / 手动写一份结构化交接快照到 `~/.smart_ai/{project}/`,
> 让用户可以无痛开新会话从断点继续, 而不是等被系统压缩到丢关键决策。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `session-handoff` |
| 来源 | 本地原创 (Cursor 用户场景驱动) |
| 安装位置 | `~/.agents/skills/session-handoff/` |
| 触发条件 | 启发式自检 (工具调用 ≥80 / 修改文件 ≥15 / 系统注入摘要 / 用户明示) |
| 依赖 | 无 (纯 Markdown + Shell) |
| 输出位置 | `~/.smart_ai/{project-slug}/handoff-{ISO-UTC-ts}.md` |

## 设计动机

Cursor agent 没有直接读 token 用量的 API, 但长会话有几个明确的"信号":

1. **系统已经开始压缩**: 当前 turn 看到 `[Previous conversation summary]` 注入
2. **工具调用累计过多**: 80+ 次 read/write/grep, 每次都吃上下文
3. **修改文件过多**: 15+ 文件的 diff 都在窗口里
4. **用户察觉到糊涂感**: 重复同一个问题、要 agent 复述早先决定

这些信号触发时, agent 应该 **立刻**(在完成当前 turn 任务之后) 写一份精炼的进度快照,
而不是被动等系统强制截断丢上下文。

## 核心原则

- **不打断当前任务**: 用户的 ask 必须先完成, handoff 是 turn 末尾的最后一步
- **写完即停**: 写完 handoff 给一段引导文字 + STOP, 不要单方面结束会话, 让用户决定下一步
- **失真要主动**: 在 handoff 里"有损压缩"——保留关键决策与原因, 抛弃推导过程
- **不重复 verified 步骤**: 新会话明确告知"已 verified 的别再跑"
- **Project slug 稳定**: 优先 git remote URL, fallback cwd basename, 跨机器一致

## 触发自检清单 (任意一条命中即触发)

| 信号 | 阈值 |
|------|------|
| 累计工具调用 | ≥ 80 |
| 修改文件数 | ≥ 15 |
| 已 Read 文件数 | ≥ 40 |
| 系统注入 `[Previous conversation summary]` | yes (最强信号) |
| 用户重复了已答过的问题 | yes |
| 重读了已读过的文件 | yes |
| 会话 turn 数 | ≥ 30 |

详见 `docs/heuristics.md`。

## 输出文件 (8 大区块)

每份 handoff 必须填齐:

1. **Task** — 用户原话需求
2. **Status** — ✅ 已完成 / 🔄 进行中 / ⏳ 待办 / ❌ 阻塞
3. **Key Decisions** — 3-7 条关键设计选择 + 为什么
4. **Modified Files** — 改动文件清单 (按意图分组)
5. **Verification** — lint / tsc / test / Playwright 已 verified 的命令 + 结果
6. **Open Questions** — 阻塞点 / 待用户确认的疑问
7. **Next Prompt** — 新会话第一条 user message 模板 (代码块, 即粘即用)
8. **Source** — 原 transcript path / 父任务标题 / 相关 rules

详见 `docs/handoff-template.md`。

## 新会话恢复 Prompt 模板

handoff 文件的 §7 必须包含 (填好占位符):

```
读取 ~/.smart_ai/{project-slug}/handoff-{ISO-timestamp}.md, 先用 2 句话复述任务和当前进度,
然后从 ⏳ 待办 清单第一项开始。

不要重新跑已经在 ✅ Verification 里标记 verified 的步骤。
不要重读已经在 ✅ Modified Files 里列出的文件 (除非真的需要再次确认其当前状态)。
若上下文还有不清楚的地方, 直接问我, 不要自己脑补。
```

## 安装

skill 已放在 `~/.agents/skills/session-handoff/`, Cursor agent 自动加载, 无需配置。

如需在新设备上安装:

```bash
mkdir -p ~/.agents/skills/session-handoff/docs ~/.agents/skills/session-handoff/examples
# 把本仓库 ku-ai/skills/session-handoff/ 下所有文件 cp 过去
```

## 使用示例

详见 `examples/basic.md` (含真实长会话场景的完整 handoff 示例)。

## 与其他规则 / Skill 的关系

- **dev-scenarios.mdc**: 典型场景日志记录是"事后归档", session-handoff 是"会话中实时快照"——两者互补
- **verification-before-completion**: handoff 的 §5 Verification 区块直接复用其"证据先于断言"原则
- **/clear / /compact**: 这是 Cursor host 的内置压缩, session-handoff 是它的"显式安全网"——host 压缩是有损黑盒, handoff 是用户可读可改

## 例外 (不要触发)

- 短会话 (< 10 turn 且 < 30 tool call)
- 当前 turn 是单行 Q&A
- 正在 await 长任务 (等它完成)
- 用户刚要求 commit/push, 应该完成那一步再说
