# Worked Example

## Scenario

The user is in a long session refactoring a Vue component. Conversation has hit:
- 92 tool calls
- 18 files modified
- "Previous conversation summary" was injected at the start of the current turn
- The user just asked to "也补个 toast 给 cancel 按钮"

The agent should:
1. **Finish the cancel-button toast change** (the actual ask)
2. **Then** trigger the handoff at the end of the turn

## What the agent does

### Step 1 — Do the work (normal)

```
[StrReplace on app/components/Foo.vue ...]
[npx eslint app/components/Foo.vue → exit 0]
```

### Step 2 — Compute slug + timestamp

```bash
$ git remote get-url origin | sed -E 's#.*/([^/]+?)(\.git)?$#\1#'
fe-picpopop
$ date -u +%Y-%m-%dT%H-%M-%SZ
2026-05-14T07-58-12Z
$ mkdir -p ~/.smart_ai/fe-picpopop
```

### Step 3 — Write the handoff file

Path: `~/.smart_ai/fe-picpopop/handoff-2026-05-14T07-58-12Z.md`

Filled-in handoff (abridged):

```markdown
# Handoff: /start 页 error_code 与 i18n key 命名对齐

| Field | Value |
|-------|-------|
| Project | fe-picpopop |
| Workspace | /Users/x/.../fe-picpopop |
| Created | 2026-05-14T07:58:12Z |
| Trigger reason | tool_calls=92, modified_files=18, summary_injected=yes |
| Reason for handoff | 上下文已被系统压缩过一次, 下一轮压缩可能丢关键决策 |

## 1. Task
PM 反馈: 上传按钮选图后 requestImagePromotionCreate 没被调; 需诊断 + 检查整个 /start 流程缺反馈的环节并补全。

## 2. Status
- ✅ 诊断完: 反白嫖 (canTrial=false) 触发跳过生成分支, 不是 bug 而是设计
- ✅ 补全 8 处反馈盲区 (反白嫖 toast / upload error 4 种 / Again error / poll 超时 -1 / 任务级失败 -2)
- ✅ dev `?bypassTrial=1` 开关
- ✅ error_code 与 i18n key 命名对齐重构 (camelCase 单源)
- ✅ Rule .cursor/rules/i18n-error-code-aligned.mdc + ku-ai 同步
- 🔄 cancel 按钮 toast (本轮刚加, 已 lint, 未实测)
- ⏳ Playwright 实测 cancel toast
- ⏳ 用户确认 commit

## 3. Key Decisions
- error_code 全链路 camelCase = i18n key 末段, 父组件一行 t(`x.y.${code}`); tracker 边界用 toReportCase 转 snake_case 兼容历史 BI
- 反白嫖路径不绕过, 加 toast 提示用户; dev 期用 ?bypassTrial=1 开关绕过
- 失败一律 reset() 清空用户上传, 避免半截卡死

## 4. Modified Files
### 服务层归一化
- app/pixpopService/PixAccountService.ts — 加 toServerImagePayloadList 剥 base64 前缀
- app/pixpopService/PixFeatureService.ts — 4 个接口加同样处理

### Page UI / 状态机
- app/pages/start.vue — 重构 onUploadError + 反白嫖 toast + watch generateFailCode + ?bypassTrial=1
- app/stores/useStartStore.ts — bypassTrial ref + reason camelCase
- app/components/start/StartUploadButton.vue — emit camelCase

### 规则沉淀
- .cursor/rules/i18n-error-code-aligned.mdc — 新建 + 同步 ku-ai/rules/

## 5. Verification
- [x] npx eslint app/{...} — exit 0
- [x] npx vue-tsc --noEmit -p tsconfig.json — exit 0
- [x] Playwright: mime_invalid → toast "Unsupported file type..." PASS
- [x] Playwright: 反白嫖 → toast "You've already used your free trial..." PASS
- [x] Playwright: ?bypassTrial=1 → POST /api/airepair/.../landing 接口被调 PASS
- [x] Playwright: 轮询 -1/-2 sentinel → 对应 toast + state reset PASS
- [ ] cancel 按钮 toast — 本轮未实测

## 6. Open Questions
- ❓ tracker 上报 snake_case 转换是否影响 BI 数据? 数据团队需要确认 (本次实测 dataLayer.push 已是 mime_invalid 格式, 应该兼容)
- ❓ 是否提交? 用户最后说"完成", 但还没说 commit

## 7. Next Prompt

```
读取 ~/.smart_ai/fe-picpopop/handoff-2026-05-14T07-58-12Z.md, 先用 2 句话复述任务和当前进度,
然后从 ⏳ 待办 清单第一项开始 (Playwright 实测 cancel toast).

不要重新跑已经在 ✅ Verification 里标记 verified 的步骤。
不要重读已经在 ✅ Modified Files 里列出的文件 (除非真的需要再次确认其当前状态)。
若上下文还有不清楚的地方, 直接问我, 不要自己脑补。
```

## 8. Source
- Transcript: /Users/x/.cursor/projects/.../agent-transcripts/{uuid}.jsonl
- Parent task title: 修 /start 上传不调接口 + 补反馈 + 命名对齐
- Related rules: .cursor/rules/i18n-error-code-aligned.mdc, .cursor/rules/pixpopservice-image-payload.mdc
```

### Step 4 — Tell the user (one short paragraph at end of turn)

> Cancel 按钮 toast 已加完, lint 通过. 当前会话已累计 92 tool calls + 18 modified files,
> 系统也已经压缩过一次上下文 — 我把进度快照写到了:
>
> ```
> ~/.smart_ai/fe-picpopop/handoff-2026-05-14T07-58-12Z.md
> ```
>
> 如果你想继续在当前会话推进 (Playwright 实测 + commit) 也可以直接说继续; 如果想新开个干净会话, 第一句粘这个:
>
> ```
> 读取 ~/.smart_ai/fe-picpopop/handoff-2026-05-14T07-58-12Z.md, 先用 2 句话复述任务和当前进度,
> 然后从 ⏳ 待办 清单第一项开始。
> ```

### Step 5 — STOP and wait

Do not continue. Do not auto-commit. Wait for user direction.

## What this example demonstrates

- ✅ The actual ask (cancel toast) was finished BEFORE handoff
- ✅ The handoff file is exhaustive on §2/§3/§5 — these are the most lossy parts
- ✅ The next-prompt is copy-paste safe
- ✅ The agent did not unilaterally end the conversation
- ✅ Both options (continue here / open fresh) were offered

## Anti-example: what NOT to do

```
> [User]: 也补个 toast 给 cancel 按钮
> [Agent]: I notice we're at 92 tool calls. Should I do a handoff first?
```

❌ Wrong: asks instead of doing. Burns more context. Doesn't address the user's ask.

```
> [User]: 也补个 toast 给 cancel 按钮
> [Agent]: I'm at the context limit, please open a new session and read this file.
```

❌ Wrong: skipped the actual task. User still has no cancel toast.

```
> [User]: 也补个 toast 给 cancel 按钮
> [Agent]: [adds toast, lints, tests, commits, opens PR, summarizes everything]
```

❌ Wrong: did the work but missed the heuristic — the user is now in a session that
will hit the wall on the next ask, with no snapshot. They lose state involuntarily.
