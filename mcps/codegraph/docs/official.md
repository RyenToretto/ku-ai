# CodeGraph — 官方资源

## 核心链接

| 资源 | 链接 |
|------|------|
| GitHub 仓库 | https://github.com/colbymchenry/codegraph |
| 官方文档网站 | https://colbymchenry.github.io/codegraph/ |
| npm 包 | https://www.npmjs.com/package/@colbymchenry/codegraph |
| Release 历史 | https://github.com/colbymchenry/codegraph/releases |

## 项目元信息

| 字段 | 值 |
|------|-----|
| 创建时间 | 2026-01-18 |
| 最后更新 | 2026-06-08 |
| 最新版本 | v0.9.9（2026-06-02） |
| 主语言 | TypeScript（91.8%），JavaScript（5%），Shell（2.3%） |
| 许可证 | MIT |
| Stars | 44K+ |
| Forks | 2.7K+ |
| Contributors | 40+ |
| Releases | 15 |

## 架构说明

CodeGraph 使用 Tree-sitter 将代码解析为 AST（抽象语法树），然后提取符号和依赖边，构建 SQLite 知识图谱。Agent 通过 MCP server 连接到该图谱，在 `initialize` 握手时接收使用指南，后续无需额外工具调用即可获得代码上下文。

```
代码文件 → Tree-sitter → AST → 符号/边提取 → SQLite
                                              ↑ 增量更新（文件监听）
Agent ← MCP Server ← 图谱查询 ←————————————┘
```

## 隐私与安全说明

- 所有索引数据存储在本地 SQLite 文件中
- 不调用任何外部 API，无数据上传
- 适用于包含敏感商业代码的私有仓库

## 自带运行时说明

CodeGraph 内嵌了独立运行时，无需额外配置 Node.js 环境，在所有平台上行为一致。

## 相关讨论

- GitHub Issues（包含使用问题和功能请求）：https://github.com/colbymchenry/codegraph/issues
