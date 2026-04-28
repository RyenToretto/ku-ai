# 全局 Cursor Rules

位于 `~/.cursor/rules/`，适用于所有项目。

## 已有规则

| 规则文件 | 描述 | alwaysApply |
|---------|------|-------------|
| `code-quality.mdc` | 通用代码质量规范（DRY、KISS、命名、错误处理） | true |
| `git-workflow.mdc` | Git 工作流规范（原子提交、分支命名、Code Review） | true |
| `css-naming.mdc` | CSS class / DOM id 命名约束（根与子元素互不为子串） | true |

## 与项目级规则的关系

- 全局规则定义通用最佳实践，所有项目共享
- 项目级规则（`.cursor/rules/`）定义项目特有的约定
- 项目规则优先级高于全局规则
- 全局规则不应包含项目特有的配置

## 新增规则

新增全局规则时：
1. 在 `~/.cursor/rules/` 创建 `.mdc` 文件
2. 在本仓库 `rules/` 下同步一份
3. 创建对应的 `.md` 说明文件（设计意图和适用场景）
