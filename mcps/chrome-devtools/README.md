# MCP: Chrome DevTools

> 让 AI Agent 直接操控 Chrome DevTools Protocol，执行页面截图、DOM 查询、性能分析、网络拦截等操作。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `chrome-devtools-mcp` |
| 官方仓库 | https://github.com/anthropics/chrome-devtools-mcp |
| Node.js 要求 | >= 18 |

## 安装与配置

### Cursor MCP 配置

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

### 前置条件

- Chrome/Chromium 浏览器已安装
- Chrome 需开启远程调试端口（`--remote-debugging-port=9222`）

## 提供的 Tools

| Tool 名称 | 描述 |
|-----------|------|
| `take_screenshot` | 截取页面截图（支持设备模拟） |
| `evaluate_script` | 在页面上下文中执行 JavaScript |
| `get_styles` | 获取元素的 computed styles |
| `network_request` | 拦截和查看网络请求 |
| `emulate_device` | 模拟移动设备 |
| `get_console_logs` | 获取控制台日志 |

## 典型场景

- 设计稿还原时截图对比
- 运行时调试 computed style
- 性能分析（FCP、LCP、CLS）
- 网络请求分析
