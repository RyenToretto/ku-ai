# MCP: CodeGraph

> 本地优先的代码知识图谱，通过 MCP 协议为 AI Agent 提供语义化代码智能——更少 token 消耗、更少工具调用。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `@colbymchenry/codegraph` |
| 官方仓库 | https://github.com/colbymchenry/codegraph |
| 官方网站 | https://colbymchenry.github.io/codegraph/ |
| 当前版本 | v0.9.9（2026-06-02） |
| 许可证 | MIT |
| Node.js 要求 | 自带运行时，无需额外安装 Node |
| Stars | 44K+ |

## 核心优势

| 指标 | 数值 |
|------|------|
| Token 成本降低 | ~16% |
| Tool calls 减少 | ~58% |
| 数据存储 | 100% 本地（SQLite），无任何外部 API 调用 |

## 安装

```bash
# 零安装模式（推荐首次使用）
npx @colbymchenry/codegraph

# 全局安装
npm i -g @colbymchenry/codegraph

# 初始化并配置 Agent（交互式安装器）
codegraph install
```

交互式安装器会自动检测本机安装的 Agent 工具（Claude Code、Cursor、Codex 等），并自动写入 MCP server 配置，无需手动配置。

## Cursor MCP 配置（手动方式）

```json
{
  "mcpServers": {
    "codegraph": {
      "command": "codegraph",
      "args": ["mcp"]
    }
  }
}
```

> 推荐使用 `codegraph install` 自动配置，它会处理各平台的配置文件格式差异。

## 项目初始化

在项目根目录运行：

```bash
npx @colbymchenry/codegraph
```

安装器会：
1. 检测已安装的 Agent 工具
2. 为每个工具写入 MCP server 配置
3. 构建初始代码索引（SQLite 数据库）
4. 之后自动监听文件变化并增量更新索引

## 提供的 MCP Tools

CodeGraph 通过 MCP 协议自动向 Agent 提供以下能力（无需手动调用工具名，Agent 在 `initialize` 时接收到指南）：

| 能力 | 描述 |
|------|------|
| 符号语义检索 | 基于 Tree-sitter AST 的精确符号索引（函数、类、变量） |
| 影响分析 | 变更一个符号前追踪所有调用方、被调用方和依赖链 |
| 代码图谱查询 | 以图关系查询跨文件依赖 |
| 增量更新 | 文件变化时自动更新索引，无需重建 |

## 支持的 Agent 工具

Claude Code、Cursor、Codex CLI、opencode、Hermes Agent、Gemini CLI、Antigravity IDE、Kiro

## 支持的语言

基于 Tree-sitter 解析，支持 20+ 种语言（TypeScript/JavaScript、Python、Go、Rust、Java 等）。

## 与现有 MCP 的关系

| MCP | 定位 |
|-----|------|
| [context7](../context7/) | 查询外部库/框架的最新官方文档 |
| **codegraph** | 索引并理解你自己的代码库内部结构 |

两者互补：context7 解决"这个库怎么用"，codegraph 解决"我的代码库里这个函数在哪里被调用"。
