---
name: session-handoff
description: Take a context-snapshot handoff when the agent self-detects the conversation is approaching token/context limits, WITHOUT interrupting the current task. Writes a structured handoff markdown to ~/.smart_ai/{project}/handoff-{ISO_ts}.md so the user can open a fresh session and resume with minimal context loss. Trigger heuristically (tool-call count, modified-file count, "Previous conversation summary" injection, user-perceived slowness) OR explicitly when the user says "handoff / 交接 / 快爆了 / 新开会话 / 保存进度". MUST run as the LAST step of the current turn — never before the user's actual ask is finished. After writing the handoff, give the user a one-line "open new session with: <prompt template>" guide and STOP. Do not unilaterally end the conversation.
---

# Session Handoff Skill

> **Purpose**: When this conversation is about to exhaust its context window, snapshot the in-flight work to disk so the user can spin up a fresh session and resume from a structured digest, losing as little state as possible.
>
> **Hard rule**: This skill MUST NOT interrupt the user's current task. Always finish the current ask first, then append the handoff block at the end of the same turn.

## When to trigger

### Heuristic self-check (run at the END of every turn that did real work)

If ANY of the following is true, perform a handoff:

| Signal | Threshold | Why |
|--------|-----------|-----|
| Cumulative tool calls in this conversation | ≥ 80 | Each call eats context |
| Files modified (Write/StrReplace/EditNotebook) | ≥ 15 | Diff-heavy sessions blow up context |
| Distinct files Read | ≥ 40 | Reads accumulate in window |
| "Previous conversation summary" was injected at the start of this turn | yes | The system has already started compacting; one more compaction cycle and you're at the edge |
| The user's last 2 messages re-stated something you already answered earlier | yes | Strong signal you're losing context coherence |
| You needed to re-read a file you already had in context | yes | Same as above |
| Conversation turn count | ≥ 30 | Empirical knee of the curve |

If NONE of the above is true, **do not** create a handoff — it's wasteful noise. This is not a per-turn ritual; it's a safety net.

### Explicit triggers (always honor)

Always honor any of these user phrases — even if heuristics say otherwise:

- "handoff" / "交接" / "保存进度" / "新开会话" / "快爆了" / "context full"
- "save my progress and let me restart"
- "I want to start a fresh session"

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
slug=$(git remote get-url origin 2>/dev/null \
       | sed -E 's#.*/([^/]+?)(\.git)?$#\1#' \
       || basename "$PWD")
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
