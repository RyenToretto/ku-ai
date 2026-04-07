# Playwright MCP 官方文档

## 文档链接

- 官方文档: https://playwright.dev/docs/getting-started-mcp
- GitHub: https://github.com/microsoft/playwright-mcp
- npm: https://www.npmjs.com/package/@playwright/mcp

## 核心概念

Playwright MCP 通过无障碍快照（Accessibility Snapshots）与页面交互，而不是依赖截图和视觉模型。快照返回 YAML 格式的页面结构，包含 ref（引用标识符）用于定位元素。

## 运行模式

1. **CDP 模式**：连接已有的 Chrome 实例（`--cdp-endpoint`）
2. **独立模式**：Playwright 自行启动浏览器

## 关键注意事项

- Ref 绑定于最新的 snapshot，页面变化后需重新获取
- 原生对话框默认自动处理（confirm 返回 true，prompt 返回默认值）
- 可通过 `browser_handle_dialog` 在触发前设置自定义响应
- 支持 `take_screenshot_afterwards: true` 在操作后附带截图
