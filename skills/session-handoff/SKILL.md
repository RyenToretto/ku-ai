---
name: session-handoff
description: Take a context-snapshot handoff so the user can open a fresh session and resume with minimal context loss. Writes structured markdown to ~/.smart_ai/{project}/handoff-{ISO_ts}.md. Three trigger paths in priority order — (1) MANUAL COMMANDS the user types like `/handoff`, `/snapshot`, `/save-context` (also the trailing-only form `;handoff` / `;snapshot` for quick chained use), or plain phrases "handoff now / snapshot now / save my progress / 交接 / 快照 / 保存进度 / 存档 / 新开会话 / 换个会话" — execute IMMEDIATELY, no confirmation, no heuristics, even if the conversation is short; (2) IMPLICIT phrases like "快爆了 / context full / I want to start a fresh session" — honor as if explicit; (3) HEURISTIC self-check at end of turns that did real work (tool-call count ≥80, modified-files ≥15, "Previous conversation summary" injection, user-perceived slowness). MUST run as the LAST step of the current turn — never before the user's actual ask in the same message is finished (e.g. "do X then /handoff" → do X first). If the manual command carries free-text after it (e.g. `/handoff 想用别的方案重做X`), include that text verbatim in the handoff file's §6 as `💡 用户重定向`. After writing, give the user a one-line "open new session with: <prompt template>" guide and STOP — do not unilaterally end the conversation.
---

# Session Handoff Skill

> **Purpose**: When this conversation is about to exhaust its context window, snapshot the in-flight work to disk so the user can spin up a fresh session and resume from a structured digest, losing as little state as possible.
>
> **Hard rule**: This skill MUST NOT interrupt the user's current task. Always finish the current ask first, then append the handoff block at the end of the same turn.

## When to trigger

Three priority levels. Higher level短路 — once a higher level matches, **do not** also evaluate lower levels.

### Level 1 — Manual commands (HIGHEST PRIORITY, 0 heuristics, no confirmation)

If the user's message contains ANY of these, trigger handoff IMMEDIATELY. Skip the 7-signal check — they explicitly asked, that's all you need.

#### Slash commands
- `/handoff`
- `/snapshot`
- `/save-context`
- `/handoff <free-text reason>` — append the reason verbatim to handoff §6 as `💡 用户重定向: ...`

#### Trailing semicolon form (chain-friendly, English/Chinese semicolons)
- `;handoff`  / `；handoff`
- `;snapshot` / `；snapshot`
- Use case: "do X then ;handoff" — finish X, then handoff

#### Plain command phrases (case-insensitive, exact match preferred)
- `handoff now`
- `snapshot now`
- `save my progress`
- `do a handoff`

#### 中文命令
- `交接` / `做交接` / `执行交接`
- `快照` / `保存快照` / `做个快照`
- `保存进度` / `存档`
- `新开会话` / `换个会话` / `开新对话` / `新会话继续`

#### Manual-trigger behavior

- **0 启发式判断** — 命中即执行, 不评估 7 信号
- **不询问 / 不解释为何触发** — 用户主动打的命令本身就是确认
- **仍要先完成同一条消息里的其他任务** — 例如 `加完 toast 再 /handoff` → 先加 toast + lint, 再 handoff
- **空消息单独命令立即执行** — 例如用户单独发 `/handoff`, 没有别的内容, 0 延迟动手
- **有自由文本 reason 必须保留** — 写到 handoff §6 `💡 用户重定向: <原话>`, 这是下一会话最重要的"航向修正"信号
- **短会话也要执行** — Level 1 不受"会话太短跳过"规则约束 (那条仅约束 Level 3 启发式)

### Level 2 — Implicit triggers (always honor, treat as explicit)

These are not commands but clear intent signals. Honor them as if Level 1, but you MAY add a one-line clarification (e.g. "ok, 我理解为做 handoff, 现在写").

- `快爆了` / `要爆了` / `上下文满了` / `context full` / `running out of context`
- `想新开一个会话` / `开个干净的会话` / `想换个 chat 继续`
- `save my progress and let me restart`
- `I want to start a fresh session`

### Level 3 — Heuristic self-check (END of every turn that did real work)

Only evaluated if **no Level 1 / Level 2 match**. Trigger if ANY signal is true:

| Signal | Threshold | Why |
|--------|-----------|-----|
| Cumulative tool calls in this conversation | ≥ 80 | Each call eats context |
| Files modified (Write/StrReplace/EditNotebook) | ≥ 15 | Diff-heavy sessions blow up context |
| Distinct files Read | ≥ 40 | Reads accumulate in window |
| "Previous conversation summary" was injected at the start of this turn | yes | The system has already started compacting; one more compaction cycle and you're at the edge |
| The user's last 2 messages re-stated something you already answered earlier | yes | Strong signal you're losing context coherence |
| You needed to re-read a file you already had in context | yes | Same as above |
| Conversation turn count | ≥ 30 | Empirical knee of the curve |

If NONE of the above is true, **do not** create a handoff — wasteful noise. Level 3 is a safety net, not a per-turn ritual.

### Examples

```
[User]: /handoff
→ Agent: writes handoff, gives next-prompt path, STOPS. (Level 1, 0 ceremony)

[User]: 加完最后一个 toast 之后 /handoff
→ Agent: adds toast, runs lint, then writes handoff. (Level 1 + 同消息任务先做)

[User]: /handoff 我想换 zustand 替代 pinia 重做 store 那一块
→ Agent: writes handoff with §6 含
   "💡 用户重定向: 想换 zustand 替代 pinia 重做 store 那一块". (Level 1 + reason)

[User]: 上下文好像快满了, 帮我保存下进度
→ Agent: "OK, 这就做 handoff" (一句话确认, Level 2) → 写文件 → 给 prompt → STOP.

[User]: (本 turn 改了 18 个文件后) 帮我 commit 一下
→ Agent: 先做 commit (用户的 ask), 然后 turn 末尾 Level 3 命中 → 加 handoff. (Level 3)
```

## What to do (don't break the current task)

1. **Finish the user's actual ask first**. Reply normally — implement the change, run lint, commit if asked. The handoff is the LAST block.
2. **Compute the project slug** (see `docs/heuristics.md` § Project Slug):
   - First try: `git remote get-url origin` → take the last URL segment, strip `.git`
   - Fallback: `basename "$PWD"` of the workspace root
3. **Compute the timestamp**: `date -u +%Y-%m-%dT%H-%M-%SZ` (ISO8601, dashes, UTC)
4. **Ensure target dir exists**: `mkdir -p ~/.smart_ai/{slug}`
5. **Write the handoff file**: `~/.smart_ai/{slug}/handoff-{ts}.md` using the template in `docs/handoff-template.md`. Fill in EVERY section honestly — this is the single source of truth the next session will read.
6. **Tell the user, in ONE short paragraph**:
   - That you snapshotted the context (one sentence why)
   - The exact next-session prompt to paste (a code block with the path)
   - That you will keep going if they want, OR they can open a fresh session now
7. **STOP**. Do not declare the conversation over. Do not start a new task. Wait for user direction.

## What MUST be in the handoff file

See `docs/handoff-template.md` for the full template. The 8 mandatory sections:

1. **Task** — the user's original one-sentence request (verbatim if possible)
2. **Status** — bullet list with ✅ done / 🔄 in-progress / ⏳ pending
3. **Key Decisions** — 3-7 bullets of design choices and *why* (lossy compression of the conversation)
4. **Modified Files** — `path` + one-line diff summary, grouped by intent
5. **Verification** — what was lint/tsc/test/Playwright-verified, with command + result
6. **Open Questions** — blockers, unconfirmed assumptions, things the next agent needs the user to answer
7. **Next Prompt** — the exact text the user should send as their first message in the new session
8. **Source** — original transcript path (if known) + parent-task title for cross-reference

## New-session best practices (encoded in the "Next Prompt")

The Next Prompt section MUST include this template (filled in):

```
读取 ~/.smart_ai/{slug}/handoff-{ts}.md, 先用 2 句话复述任务和当前进度,
然后从 ⏳ 待办 清单第一项开始。

不要重新跑已经在 ✅ Verification 里标记 verified 的步骤。
不要重读已经在 ✅ Modified Files 里列出的文件 (除非真的需要再次确认)。
若上下文还有不清楚的地方, 直接问我, 不要自己脑补。
```

The new agent should:
1. Read the handoff file FIRST (single Read call)
2. Echo back task + status in 2 sentences (alignment check)
3. Skip steps already marked ✅
4. Resume from the first ⏳ item

## Non-negotiables

- **Never** truncate the handoff to "save tokens" — the whole point is to NOT lose state. Be exhaustive on Status / Modified Files / Decisions.
- **Never** delete or overwrite an older handoff. New file each time, ISO timestamp guarantees uniqueness.
- **Never** ask the user "should I do a handoff?" — if heuristics or explicit trigger fired, just do it. (Asking burns more context than acting.)
- **Never** end the current conversation. The user decides whether to open a new one.
- **Never** put secrets / credentials / tokens into the handoff file. Redact with `<REDACTED>`.

## When NOT to trigger

- The conversation is short (< 10 turns and < 30 tool calls)
- The current turn is a one-line Q&A or a quick fix
- You are in the middle of a streaming/long-running command (wait until it returns)
- The user is in `/clear` or `/compact` mode (the host already handles context)

## Project Slug & Path Reference

```bash
# macOS / Linux 兼容写法 (BSD sed 不支持 +? 非贪婪量词, 改用 basename + 后缀剥除)
url=$(git remote get-url origin 2>/dev/null)
slug=$(basename "${url%.git}")
[ -z "$slug" ] && slug=$(basename "$PWD")

ts=$(date -u +%Y-%m-%dT%H-%M-%SZ)
mkdir -p ~/.smart_ai/"$slug"
target=~/.smart_ai/"$slug"/handoff-"$ts".md
```

Example resolved path:
```
~/.smart_ai/fe-picpopop/handoff-2026-05-14T07-58-12Z.md
```

## Detailed References

- Trigger heuristics: `docs/heuristics.md`
- File template: `docs/handoff-template.md`
- Worked example: `examples/basic.md`
