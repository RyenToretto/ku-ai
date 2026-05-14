# Best Practices: session-handoff

## 给 Agent 的实施铁律

### 1. 触发判断只在"做了实事"的 turn 末尾跑

```
turn 干了什么?
  ├─ 纯 Q&A (单条回答, 没动文件没跑命令) → 跳过自检
  ├─ 单步小改 (一个 StrReplace, 没跑 lint) → 跳过自检
  └─ 多步任务 (跑命令 / 改文件 / lint / commit) → 跑 7 信号自检
```

**不要每 turn 都自检** — 自检本身也消耗注意力, 应该限定在"高强度 turn"。

### 2. 命中信号 → 立刻动手, 不要先问用户

```
✗ 错误: "我注意到上下文比较多了, 要不要做个 handoff?"
✓ 正确: 直接写 handoff 文件, 在 turn 末尾告诉用户已写, 给 next-prompt 模板
```

理由: 询问本身就是一次完整的 turn 来回, 浪费宝贵的剩余上下文; 而且用户多半不知道
"是否应该 handoff" 该怎么判断, 把这个判断推给用户反而拖累。

### 3. handoff 的失真要"主动有损"

handoff 不是 transcript 的压缩, 而是 **state 的快照**。两者的区别:

| Transcript 视角 | State 视角 |
|----------------|----------|
| "用户说了 X, 我回了 Y, 然后用户又说 Z" | "我们已经决定用 X 方案, 因为 Y" |
| 完整对话流 (按时间) | 关键事实 (按主题分组) |
| 大量探索过程 | 只有最终结论 |

写 §3 (Key Decisions) 时, 想象自己在写 **postmortem 的"经验教训"** 那一节,
而不是"事件时间线"。

### 4. §5 Verification 严禁虚标

```
✗ 错误:
- [x] Playwright tested all error toasts (实际只测了一个 mime_invalid)

✓ 正确:
- [x] Playwright: mime_invalid → toast "Unsupported file type..." PASS
- [ ] Playwright: rawTooLarge — 没实测 (需要造 30MB+ 大文件, 跳过)
- [ ] Playwright: compressFailed — 没实测 (需要 mock canvas 异常)
```

虚标会让下一会话 agent 跳过实际有问题的步骤, 比"没标"破坏性更大。

### 5. §7 Next Prompt 必须 copy-paste 安全

- 路径要写 **完整绝对路径** (替换好所有 `{slug}` `{ts}` 占位符)
- 不能引用"上文"/"刚才"/"之前我们" — 新会话没有"上文"
- 不能用项目内变量名 (如 `$startStore`) 假设新会话能直接 resolve — 要给完整描述

### 6. 写完 handoff 之后必须 STOP

```
✗ 错误:
- 写 handoff
- 然后顺势 commit
- 然后顺势 push
- 然后顺势开 PR
(用户根本没说要 commit, agent 单方面把上下文耗光了)

✓ 正确:
- 写 handoff
- 一段引导文字: "写好了, 路径 X. 想继续在这个会话推进 Y? 还是开新会话粘 Z?"
- 不动作, 等用户回复
```

## 给用户的使用建议

### 看到 handoff 提示后, 怎么决定?

| 场景 | 推荐 |
|------|-----|
| 当前 task 已 100% 完成, 只剩 commit/push | 留在当前会话做完, 下次再开新会话时再读 handoff (作为参考) |
| 当前 task 还有明确的 ⏳ 待办, 上下文确实糊涂 | 立刻开新会话粘 next-prompt |
| 不确定 | 开新会话, 损失最小 |

### 新会话第一句的"加料"

handoff 里给的 next-prompt 是默认模板, 你可以在末尾加一句修正:

```
读取 ~/.smart_ai/fe-picpopop/handoff-2026-05-14T07-58-12Z.md, ...

(我修正一下: §3 里的"决定 A"我想改成 B, 重新评估这个改动对其他环节的影响)
```

新 agent 会按修正后的方向走, 而不是死板跟着原 handoff。

### 多次 handoff 形成的"链"

长任务可能需要多次 handoff:
```
~/.smart_ai/fe-picpopop/
├── handoff-2026-05-14T07-58-12Z.md  ← 第一次
├── handoff-2026-05-14T11-22-04Z.md  ← 第二次, §8 "Related handoffs" 引用第一次
└── handoff-2026-05-15T03-41-38Z.md  ← 第三次, 引用第二次
```

按 ISO 时间排序, 永远新的覆盖旧的语义, 但 **不删旧文件** — 旧 handoff 是历史档案,
便于回查"这个决定是哪一轮做的"。

### 清理策略

- ≤ 30 天: 保留 (短期 grep 历史)
- 31-180 天: 按需保留 (用 `cd ~/.smart_ai && find . -mtime +30` 列出)
- > 180 天: 可清理或归档到 `~/.smart_ai-archive/`

不要自动清理 — handoff 是低成本档案, 价值远大于占用的几 KB。

## 反模式速查

| 反模式 | 后果 |
|--------|------|
| 每个 turn 末尾都自检并询问 | 自己耗掉大量上下文 |
| 只在用户说"快爆了"时才动 | 到时 agent 自己已经糊涂, 写出来的 handoff 也丢失关键决策 |
| handoff 里写"详见 chat history" | 新会话没有 chat history, 等于没写 |
| handoff 后还顺势做未授权的事 (commit/push/PR) | 用户失控, 旧会话突然"自己结束了" |
| handoff 路径写相对路径 (./handoff.md) | 新会话从别的目录启动, 文件找不到 |
| 多个 handoff 都用同一个文件名 (handoff.md) | 后写的覆盖前写的, 历史丢失 |

## 与 host 的 /clear · /compact 的关系

| 工具 | 何时 | 谁来 | 透明度 |
|------|------|------|--------|
| Cursor `/clear` | 主动清空对话 | 用户 | 全透明, 但也全损失 |
| Cursor `/compact` | 主动让系统压缩 | 用户 | 黑盒压缩, 不可控 |
| 系统自动注入 summary | 上下文接近极限时 | 系统自动 | 半黑盒, agent 看得到 |
| **session-handoff** | 启发式自检命中 | agent | **全透明 + 用户可读可改的 markdown** |

session-handoff 是这些机制的 **互补**, 不是替代:
- `/clear`: 全清, 适合"换个完全无关的话题"
- `/compact`: 部分压缩, 适合"我想继续, 但请精简上下文"
- handoff: **跨会话** 的状态转移, 适合"我想从干净的窗口重启同一个任务"
