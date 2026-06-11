# Cursor 插件: Superpowers

> 完整的软件开发方法论框架——通过 14 个可组合的工作流 Skills，让 AI Agent 在编码前先规划、先验证、先审查，而非直接开始写代码。

## 基本信息

| 项目 | 值 |
|------|-----|
| 插件名称 | `superpowers` |
| 来源仓库 | [obra/superpowers](https://github.com/obra/superpowers) |
| 插件市场 | Cursor Plugin Marketplace |
| 许可证 | MIT |
| 当前版本 | v4.3.1+（2026-02） |

## 安装

### Cursor（推荐）

在 Cursor Agent 对话框中：

```
/add-plugin superpowers
```

或在 Cursor 插件市场搜索 `superpowers`。

### Claude Code

```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### 其他平台（Codex CLI、OpenCode、Gemini CLI）

```bash
# 从 GitHub 获取安装指南
curl https://raw.githubusercontent.com/obra/superpowers/main/README.md
```

## 核心机制

### Skill 自动触发

Skills 的关键特性是**自动触发**，而不是手动调用命令：
- Agent 在每次任务开始前检查是否有相关 Skill
- 相关 Skill 作为**强制要求**而非建议
- 会话启动时，Hook 系统注入 "You have Superpowers" 提示

### Hooks 系统

- `hooks-cursor.json` — Cursor 会话生命周期事件钩子
- Session 启动时自动触发 `using-superpowers` Skill，建立使用框架

## 14 个工作流 Skills

| Skill | 触发场景 |
|------|------|
| `using-superpowers` | **会话入口**——每次会话开始，建立 Skill 发现和使用方式 |
| `brainstorming` | 任何创意/设计类任务前，进行苏格拉底式需求探索 |
| `writing-plans` | 明确需求后，编写带验证步骤的详细实施计划 |
| `executing-plans` | 有完整计划后，在独立会话中批量执行（含人工检查点） |
| `dispatching-parallel-agents` | 2+ 个独立任务可以并行处理时 |
| `subagent-driven-development` | 高速迭代：Subagent 执行 + 两阶段代码审查 |
| `using-git-worktrees` | 需要隔离开发分支时，管理 Git Worktree |
| `systematic-debugging` | 遇到 Bug 时，执行 4 阶段根因分析 |
| `test-driven-development` | 编写新功能前，严格遵守 RED-GREEN-REFACTOR 循环 |
| `verification-before-completion` | 宣称完成前，验证结果是否真正解决了问题 |
| `requesting-code-review` | 提交 PR 或代码审查前，执行预检清单 |
| `receiving-code-review` | 收到代码审查反馈后，系统性处理 |
| `finishing-a-development-branch` | 功能开发完成，处理合并/PR 决策流程 |
| `writing-skills` | 创建新 Skill 时，遵循最佳实践和测试方法 |

## 与本仓库现有 Skills 的关系

本仓库 `skills/` 目录中的部分 Skills 与 Superpowers 同源或高度相关：

| 本仓库 Skills 目录 | 对应 Superpowers Skill | 说明 |
|---|---|---|
| [skills/systematic-debugging](../../skills/systematic-debugging/) | `systematic-debugging` | 同一 Skill 的本地文档副本 |
| [skills/test-driven-development](../../skills/test-driven-development/) | `test-driven-development` | 同上 |
| [skills/verification-before-completion](../../skills/verification-before-completion/) | `verification-before-completion` | 同上 |
| [skills/session-handoff](../../skills/session-handoff/) | 无直接对应 | 独立 Skill |

本目录（`plugins/superpowers/`）记录的是 Superpowers 作为 **Cursor 插件**的整体研究文档，不重复单个 Skill 的详细说明。
