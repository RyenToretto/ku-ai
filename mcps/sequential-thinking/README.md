# MCP: Sequential Thinking

> 为 AI Agent 提供结构化的多步骤思维链工具，用于复杂问题的分步推理。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `@modelcontextprotocol/server-sequential-thinking` |
| 官方仓库 | https://github.com/modelcontextprotocol/servers |
| Node.js 要求 | >= 18 |

## 安装与配置

### Cursor MCP 配置

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking@latest"]
    }
  }
}
```

### 前置条件

- 无需额外认证，开箱即用

## 提供的 Tools

| Tool 名称 | 描述 |
|-----------|------|
| `sequentialthinking` | 动态的、可反思的思维链工具 |

## 核心能力

- **逐步推理**：将复杂问题拆解为多个思考步骤
- **修正与反思**：可以修改之前的推理步骤
- **分支思考**：可以创建思维分支探索不同方向
- **动态调整**：根据推理进展动态调整剩余步骤数

## 典型场景

- 复杂架构设计决策
- 多步骤问题排查
- 需要权衡多个方案的技术选型
- 逻辑推导和数学推理
