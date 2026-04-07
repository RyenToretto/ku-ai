# MCP: Context7

> 实时查询任意库/框架/SDK 的最新官方文档，确保 AI Agent 使用的 API 信息是最新的。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `@upstash/context7-mcp` |
| 官方仓库 | https://github.com/upstash/context7 |
| 官方网站 | https://context7.com |
| Node.js 要求 | >= 18 |

## 安装与配置

### Cursor MCP 配置

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### 前置条件

- 无需额外认证，开箱即用

## 提供的 Tools

| Tool 名称 | 描述 |
|-----------|------|
| `resolve-library-id` | 根据库名搜索并返回 Context7 兼容的库 ID |
| `get-library-docs` | 获取指定库的最新文档内容 |

## 典型场景

- 查询 Nuxt/Vue/React 等框架的最新 API 文档
- 确认某个库的配置参数和用法
- 版本迁移时查看新版本的 breaking changes
- 使用不熟悉的第三方库时获取文档
