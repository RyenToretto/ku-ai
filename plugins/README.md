# Cursor 插件 Skills

通过 Cursor 插件市场安装的 Skills，位于 `~/.cursor/plugins/cache/cursor-public/`。

## Figma 插件

7 个 Figma 相关 Skills：

| Skill | 描述 |
|-------|------|
| `figma-use` | **必须前置** — 每次 `use_figma` 调用前必须先加载 |
| `figma-implement-design` | 将 Figma 设计翻译为生产级代码 |
| `figma-generate-design` | 从代码/描述创建 Figma 页面 |
| `figma-generate-library` | 构建/更新 Figma 设计系统库 |
| `figma-code-connect-components` | 将 Figma 组件连接到代码组件 |
| `figma-create-design-system-rules` | 生成项目设计系统规则 |
| `figma-create-new-file` | 创建新 Figma 文件 |

## Superpowers 插件

完整的软件开发方法论框架。安装方式：在 Cursor Agent 对话中运行 `/add-plugin superpowers`。

详细调研文档：[superpowers/README.md](superpowers/README.md)

14 个工作流 Skills：

| Skill | 描述 |
|-------|------|
| `using-superpowers` | 会话入口 — 建立 Skills 发现和使用方式 |
| `brainstorming` | 创意工作前的苏格拉底式需求探索 |
| `writing-plans` | 编写带验证步骤的详细实施计划 |
| `executing-plans` | 在独立会话中批量执行计划（含人工检查点） |
| `dispatching-parallel-agents` | 并行分发独立任务给多个 Subagent |
| `subagent-driven-development` | 基于 Subagent 的高速迭代 + 两阶段代码审查 |
| `using-git-worktrees` | Git Worktree 隔离开发分支 |
| `systematic-debugging` | 4 阶段系统化根因分析 |
| `test-driven-development` | 严格 RED-GREEN-REFACTOR TDD 循环 |
| `verification-before-completion` | 完成前验证结果真正解决了问题 |
| `requesting-code-review` | 提交前预检清单 |
| `receiving-code-review` | 系统性处理代码审查反馈 |
| `finishing-a-development-branch` | 完成开发分支的合并/PR 决策流程 |
| `writing-skills` | 按最佳实践和测试方法编写新 Skill |
