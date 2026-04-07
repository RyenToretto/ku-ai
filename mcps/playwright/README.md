# MCP: Playwright

> 浏览器自动化测试和交互，通过结构化的无障碍快照（而非截图）与网页交互。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `@playwright/mcp` |
| 官方仓库 | https://github.com/microsoft/playwright-mcp |
| 官方文档 | https://playwright.dev/docs/getting-started-mcp |
| Node.js 要求 | >= 18 |

## 安装与配置

### Cursor MCP 配置

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--cdp-endpoint", "http://127.0.0.1:9222"]
    }
  }
}
```

### 前置条件

- Chrome/Chromium 浏览器（使用 CDP 模式时需开启调试端口）
- 或让 Playwright 自行启动浏览器实例

## 提供的 Tools

| Tool 名称 | 描述 |
|-----------|------|
| `browser_navigate` | 导航到 URL |
| `browser_snapshot` | 获取页面无障碍快照（YAML） |
| `browser_click` | 点击元素 |
| `browser_type` | 输入文本 |
| `browser_fill` | 填充表单字段 |
| `browser_select` | 选择下拉选项 |
| `browser_hover` | 悬停元素 |
| `browser_scroll` | 滚动页面 |
| `browser_take_screenshot` | 截取屏幕截图 |
| `browser_handle_dialog` | 处理对话框 |
| `browser_console_messages` | 获取控制台消息 |
| `browser_network_requests` | 获取网络请求 |
| `browser_tabs` | 管理浏览器标签页 |
| `browser_search` | 页面内搜索文本 |

## 典型场景

- 前端功能测试（点击、填表、验证）
- UI 自动化验证
- 页面截图对比
- 网络请求监控
- 表单自动填充测试
