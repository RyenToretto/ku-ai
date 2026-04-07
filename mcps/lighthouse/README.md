# MCP: Lighthouse

> 让 AI Agent 直接运行 Google Lighthouse 审计，获取性能、无障碍性、SEO、最佳实践评分。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `lighthouse-mcp` |
| 版本 | 0.1.14 |
| 官方仓库 | https://github.com/nicholasoxford/lighthouse-mcp |
| Node.js 要求 | >= 16 |

## 安装与配置

### Cursor MCP 配置

```json
{
  "mcpServers": {
    "lighthouse": {
      "command": "npx",
      "args": ["-y", "lighthouse-mcp@latest"]
    }
  }
}
```

### 前置条件

- Chrome/Chromium 浏览器已安装
- 无需额外认证

## 提供的 Tools

| Tool 名称 | 描述 |
|-----------|------|
| `lighthouse_audit` | 对指定 URL 运行完整的 Lighthouse 审计 |

### 审计参数

| 参数 | 说明 |
|------|------|
| `url` | 目标 URL |
| `categories` | 审计类别（performance, accessibility, best-practices, seo, pwa） |
| `device` | 设备类型（mobile / desktop） |
| `throttling` | 网络限速（simulated / devtools / none） |

## 典型场景

- 发布前性能检查
- 无障碍性合规审计
- SEO 问题诊断
- PWA 配置验证
- 优化前后对比
