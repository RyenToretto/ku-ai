# Heuristics: When to Trigger Session Handoff

The agent has no direct API to read its own context-window usage, so we rely on **proxy signals** that empirically correlate with "approaching the limit".

## The single self-check (run at end of every meaningful turn)

```
Did this turn do real work (read/write files, run commands)?
  └─ Yes → Run the 7-signal check below
  └─ No (pure Q&A, single-line answer)→ Skip
```

## The 7 signals

Trigger handoff if **ANY ONE** of these is true. Don't wait for multiple.

### 1. Cumulative tool calls ≥ 80

You can self-estimate by counting your own tool invocations (Read / Grep / Shell / StrReplace / Write / etc.) since the conversation began. The threshold is empirical — most agent contexts choke around 100-120 tool calls due to accumulated tool-result text.

### 2. Files modified ≥ 15

Each file edit produces a diff that lives in context. 15 modified files = significant accumulated diff state.

```
Count: distinct file paths you've passed to Write / StrReplace / EditNotebook
```

### 3. Distinct files Read ≥ 40

Even if you only Read a file, the content stays in context. 40 distinct reads = a LOT of accumulated source code.

### 4. "Previous conversation summary" injected

If the current turn's prompt starts with `[Previous conversation summary]`, the host system has already started auto-compacting. **This is the loudest signal**. The next compaction cycle will lose much more.

When you see this marker → handoff IMMEDIATELY, do not wait for other signals.

### 5. User repeated a request you already addressed

If the user's last 1-2 messages re-state something already in the conversation:
- "Why didn't you just X?" (you already explained why earlier)
- "I told you to use Y" (you already used Y two turns ago)

→ Strong signal you've lost coherence. The user can feel it.

### 6. You re-read a file you already had in context

If you find yourself calling Read on a path that was already Read or Written this session — and you can't confidently recall its current shape — your effective context has degraded.

### 7. Conversation turn count ≥ 30

Empirical knee. By turn 30, most multi-step tasks have either finished or accumulated enough context to risk a wall.

## Compounding signals

Two weak signals together = strong. Examples:

| Signal A | Signal B | Action |
|----------|----------|--------|
| 50 tool calls | 8 modified files | OK, keep going |
| 50 tool calls | 8 modified files + user repeated themselves | **Handoff** |
| 60 tool calls | 12 modified files | **Handoff** (close to either threshold + compounding) |

## Project Slug derivation

```bash
# Preferred: derive from git remote (stable across machines)
slug=$(git remote get-url origin 2>/dev/null \
       | sed -E 's#.*/([^/]+?)(\.git)?$#\1#')

# Fallback 1: workspace folder basename
[ -z "$slug" ] && slug=$(basename "$PWD")

# Fallback 2: a hash of the cwd (last resort, non-portable but unique)
[ -z "$slug" ] && slug=$(echo -n "$PWD" | shasum | cut -c1-12)
```

Examples:
- `git@github.com:org/fe-picpopop.git` → `fe-picpopop`
- `/Users/x/code/fe-picpopop` (no git) → `fe-picpopop`

## Anti-patterns (do NOT trigger when…)

- Pure Q&A turn ("what does this function do?")
- One-line config tweak with no follow-up planned
- You're polling a long-running shell job (`AwaitShell`) — wait for it to finish
- The user just asked you to commit & push — finish that, then check signals
- You're in the FIRST 5 turns of the conversation — too early, context is fine

## How to "feel" your own degradation

Honest self-questions to ask at end of a heavy turn:

- *"If the user asked me 'what's still TODO?', could I list it confidently right now?"*
- *"If the user asked me 'which files did we modify?', could I list them without re-reading git status?"*
- *"Did I just hesitate on a decision I made earlier in this conversation?"*

If any answer is "no" or "kinda" → that's the same signal as #5/#6 above. Handoff.
