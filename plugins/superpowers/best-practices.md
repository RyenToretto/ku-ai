# Superpowers — 最佳实践

## 核心理念

Superpowers 的核心洞察是：**不要让 Agent 直接跳入编码**。每次任务前先检查相关 Skills，按工作流执行，而非直接输出代码。

## 何时使用

- 所有非trivial的软件开发任务（2+ 文件的改动）
- 不确定需求的功能开发（先 brainstorming）
- 大型功能或重构（writing-plans + executing-plans）
- 调试顽固 bug（systematic-debugging）
- 需要保证代码质量的关键路径（TDD + code-review）

## 何时不用（或跳过部分 Skill）

- 简单的单行修改（不值得全流程走）
- 明确需求的快速原型（可跳过 brainstorming）
- 紧急热修复（直接 systematic-debugging，跳过规划流程）

## Skills 使用顺序指南

```
需求不清晰 → brainstorming → writing-plans → executing-plans
需求清晰 → writing-plans → executing-plans
遇到 bug → systematic-debugging → verification-before-completion
提交代码 → requesting-code-review → finishing-a-development-branch
并行任务 → dispatching-parallel-agents → subagent-driven-development
```

## 与本仓库其他 Skills 结合

| 场景 | 组合 |
|------|------|
| UI 功能开发 | Superpowers(brainstorming + TDD) + taste-skill(设计约束) |
| API 重构 | Superpowers(systematic-debugging + writing-plans) + codegraph(影响分析) |
| 新 Skill 开发 | Superpowers(writing-skills) + darwin-skill(优化循环) |
| 新功能设计系统 | Superpowers(brainstorming) + ui-ux-pro-max(风格选择) + design-md(令牌固化) |

## 常见误区

| 误区 | 正确做法 |
|------|------|
| 用旧的斜杠命令（`/brainstorm`） | 这些命令已废弃，直接描述任务让 Skills 自动触发 |
| 以为安装后需要手动调用每个 Skill | Skills 是强制触发的，Agent 自动检查和应用 |
| 认为 Superpowers 会让每个任务都很慢 | 规划和验证减少了返工，整体更快 |
| 只在 Claude Code 中使用 | Cursor 从 v4.3.1 开始完整支持，通过插件市场安装 |

## 注意事项

- 安装后需要新建会话才能激活 Hook 机制
- 若 Agent 没有自动检查 Skills，在请求中加入提示："先检查你有哪些 Skills"
- 14 个 Skills 中有依赖关系（见 `docs/official.md`），执行计划类任务时确保依赖 Skill 可用
- 本仓库 `skills/` 目录中已有部分 Superpowers Skill 的本地文档，可参考以了解各 Skill 的详细工作方式
