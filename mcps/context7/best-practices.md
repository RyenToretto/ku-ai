# Context7 MCP 最佳实践

## 推荐做法

- 遇到任何库/框架/SDK/API/CLI 相关问题时优先使用 Context7，而非依赖训练数据
- 即使是熟知的库（React、Vue、Tailwind），也建议查询以获取最新变更
- 先用 `resolve-library-id` 确认库 ID，再用 `get-library-docs` 获取文档

## 适用场景

- API 语法和配置查询
- 版本迁移指导
- 库特定的调试问题
- 安装和配置说明
- CLI 工具用法

## 不适用场景

- 代码重构（不需要查文档）
- 从零编写脚本（除非涉及特定库 API）
- 调试业务逻辑
- 代码审查
- 通用编程概念

## 与其他工具配合

- 比 Web Search 更精准：Context7 直接返回结构化文档内容，无需解析网页
- 配合 Cursor Rules 使用：查到最新用法后可更新项目的 Rules
