# playwright-chrome 规则说明

## 设计意图

Playwright MCP 通过 Chrome DevTools Protocol 连接浏览器进行自动化操作。默认情况下，AI Agent 可能会用 `--user-data-dir=/tmp/xxx` 启动全新的 Chrome 实例，导致：

- 丢失所有已登录的 Google/GitHub 等账号会话
- localStorage、Cookie、IndexedDB 全部为空
- 需要重新登录所有网站

此规则强制 Agent 复用用户日常使用的 Chrome 实例，保留完整的浏览器状态。

## 适用场景

- 使用 Playwright MCP 进行页面验证、截图、自动化测试
- 需要已登录状态的页面调试（如后台管理、OAuth 回调页面）
- 任何通过 CDP 连接 Chrome 的场景

## 关键约束

1. 禁止使用临时 `--user-data-dir`
2. 禁止 kill Chrome 进程
3. 禁止自行启动新 Chrome
4. 必须先检测 9222 端口，未就绪则提示用户手动重启 Chrome

## 用户侧配置

macOS 用户可在 Chrome 启动命令中永久添加 `--remote-debugging-port=9222`，避免每次手动操作。
