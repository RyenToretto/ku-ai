# git-workflow.mdc 设计说明

## 设计意图

规范 Git 操作习惯，确保提交历史干净、分支管理有序。

## 适用场景

- 所有使用 Git 的项目
- 自动应用（`alwaysApply: true`）

## 核心要点

1. **原子提交** — 一次提交只做一件事
2. **清洁提交** — 不提交调试代码、敏感文件
3. **分支命名** — feat/、fix/、refactor/、hotfix/ 前缀
4. **PR 规范** — 提交前自检 diff、描述需说明 what/why/how
