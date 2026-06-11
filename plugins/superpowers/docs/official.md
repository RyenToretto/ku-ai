# Superpowers — 官方资源

## 核心链接

| 资源 | 链接 |
|------|------|
| GitHub 主仓库 | https://github.com/obra/superpowers |
| Cursor 插件市场页 | https://cursor.directory/plugins/superpowers |
| Claude Code 插件市场 | obra/superpowers-marketplace |
| DeepWiki 文档 | https://deepwiki.com/obra/superpowers |
| Ry Walker 研究分析 | https://rywalker.com/research/superpowers-skills-framework |

## 项目元信息

| 字段 | 值 |
|------|-----|
| 主语言 | Shell（Hooks 和脚本）+ Markdown（SKILL.md 文件） |
| 架构 | Markdown SKILL.md 文件 + Prompt 模板 + Hook 脚本 |
| 许可证 | MIT |
| 版本 | v4.3.1（2026-02，加入 Cursor 支持） |
| Contributors | 10（obra: 245 commits） |
| Open Issues | 144 |

## 技术架构说明

```
.cursor-plugin/plugin.json
  ├── skills/          ← SKILL.md 文件（14 个工作流定义）
  ├── agents/          ← 专用 Agent 定义
  ├── commands/        ← 斜杠命令（已废弃，改用 Skills 系统）
  └── hooks/
      └── hooks-cursor.json  ← 生命周期 Hook 配置
```

### Bootstrap 机制

1. Cursor 安装插件后注册 Skills、Agents、Commands、Hooks
2. 每次新会话，Hook 系统注入 "You have Superpowers" 提示
3. Agent 在任务开始前强制检查可用 Skills
4. 找到相关 Skill 后，按 Skill 定义的工作流执行

### Skill 依赖链示例

`subagent-driven-development` 依赖以下 Skills 协同运行：
- `using-git-worktrees` — 确保隔离工作区
- `writing-plans` — 创建执行计划
- `requesting-code-review` — 代码审查模板
- `finishing-a-development-branch` — 完成开发分支

## Cursor 集成说明

- Cursor v4.3.1 后通过 `/add-plugin superpowers` 安装
- 传统斜杠命令（`/brainstorm`、`/write-plan`）已废弃，改用 Skills 自动发现
- 安装后在新对话中描述任务，Agent 会自动检查相关 Skill，不应立即开始编码

## 版本历史亮点

| 版本 | 变化 |
|------|------|
| v4.3.1 (2026-02) | 新增 Cursor 插件支持 |
| 更早版本 | Claude Code、Codex CLI、OpenCode、Gemini CLI 支持 |
