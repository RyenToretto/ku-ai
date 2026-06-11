# CodeGraph — 最佳实践

## 何时使用

- 大型或中等规模代码库（5K+ 行），Agent 频繁"迷路"
- 需要精确的跨文件符号引用和影响分析（重构、删除、重命名）
- 多文件联动修改前，需要了解依赖链和调用关系
- 团队代码库，新成员借助 Agent 快速理解结构

## 何时不用

- 极小型项目（< 500 行），索引成本高于收益
- 生成全新代码（codegraph 用于理解现有代码，而非凭空生成）
- 不希望在项目本地保留 SQLite 索引文件（隐私敏感度极高的场景）

## 与其他 MCP 组合使用

| 组合 | 效果 |
|------|------|
| CodeGraph + context7 | 同时理解项目内部结构（codegraph）和外部依赖文档（context7），适合大多数开发任务 |
| CodeGraph + playwright | 先用 codegraph 理解测试结构，再用 playwright 运行 E2E 测试 |
| CodeGraph + sequential-thinking | 复杂重构前，sequential-thinking 规划步骤，codegraph 提供影响分析数据 |

## 索引维护

- 首次初始化后，codegraph 通过文件监听自动增量更新索引
- 大规模代码迁移（如整体重命名、目录重组）后，建议手动重建索引：`codegraph rebuild`
- 将索引文件（`.codegraph/` 或 `codegraph.db`）加入 `.gitignore`，避免提交本地索引

## 常见风险

| 风险 | 应对方式 |
|------|------|
| 索引文件占用磁盘空间 | 大型 monorepo 只索引核心子包，避免全量索引 |
| 生成代码的 Agent 工具调用数不降反升 | 确认 MCP server 正确注册并在 Agent 的 `initialize` 握手中被读取 |
| 多版本 Node/运行时冲突 | CodeGraph 内嵌运行时，通常不受影响；如异常检查 PATH 中 `codegraph` 是否正确 |
| 隐私敏感代码库 | 确认 SQLite 文件不被上传（加入 .gitignore），codegraph 不发送代码到外部 |

## Cursor 专项注意

- MCP 配置写入后，需重启 Cursor 生效
- 若 Agent 仍未使用 codegraph 上下文，在 Cursor 设置中检查 MCP server 是否已启用
- 建议每个项目单独初始化，避免不同代码库的索引混淆
