# remind-action-card Hook 说明

创建时间: 2026-05-25
类型: 用户级 stop hook

## 设计意图

`next-steps-summary.mdc` 规则在 system prompt 中被注入，但模型（尤其是 Composer 2.5 等轻量模型）在长对话、换模型后经常漏触发「下一步行动卡」。

此 hook 通过 `stop` 事件（每次 agent 完成时）强制检查：
- 若最后一条回复是任务性回复（包含代码块 ``` 或完成/修改等关键词）
- 且回复末尾缺少「下一步行动卡」
- 则注入 `followup_message` 让 agent 补充输出

## 文件位置

- `~/.cursor/hooks.json` — 用户级 hooks 配置
- `~/.cursor/hooks/remind-action-card.sh` — 检查脚本

## hooks.json 关键配置

```json
{
  "version": 1,
  "hooks": {
    "stop": [
      {
        "command": "./hooks/remind-action-card.sh",
        "failClosed": false
      }
    ]
  }
}
```

`failClosed: false` — hook 崩溃时放行，不阻断 agent。

## 适用场景

- 所有工作区（用户级 hook）
- 任何模型（Sonnet / Opus / GPT / Composer）
- 代替单纯依赖 system prompt rule 的软约束

## 触发条件（脚本逻辑）

1. 解析 stdin JSON 中的 `last_agent_message`
2. 如果含「下一步行动卡」→ 放行（`{}`）
3. 如果不含代码块或完成关键词 → 视为纯问答，放行
4. 否则 → 返回 `followup_message` 提醒 agent 补充
