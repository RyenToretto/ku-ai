# playwright-chrome 规则说明

## 设计意图

Playwright MCP 通过 Chrome DevTools Protocol 连接浏览器进行自动化操作。默认情况下，AI Agent 可能会用 `--user-data-dir=/tmp/xxx` 启动全新的 Chrome 实例，导致：

- 丢失所有已登录的 Google / GitHub / 公司 SSO 账号会话
- localStorage、Cookie、IndexedDB 全部为空
- 需要重新登录所有网站
- 可能与用户已有 9222 实例端口冲突，CDP 连接行为未定义

此规则强制 Agent 复用用户日常使用的 Chrome 实例，并在每次调用 Playwright MCP 工具前**先探测、再复用**：已有 9222 实例时严禁再启动 `chrome-debug` 或新 user-data-dir，已有目标 tab 时优先 `browser_tabs` 切换 / `browser_navigate` 复用，永远不要因「想要的 tab 不在」而重启浏览器。

## 适用场景

- 使用 Playwright MCP 进行页面验证、截图、自动化测试
- 需要已登录状态的页面调试（后台管理、OAuth 回调、SSO 登录页）
- 任何通过 CDP 连接 Chrome 的场景

## 关键约束

1. **禁止**临时 `--user-data-dir`、kill Chrome 进程、自行启动新 Chrome
2. **每次调用前**先 `curl -s http://127.0.0.1:9222/json/version` 探测端口
3. 端口已就绪 → 进 `browser_tabs` 列出现有 tab，优先复用 / 切换；不存在则用 `browser_tabs new` 新开
4. 端口未就绪 → 提示用户手动 `~/bin/chrome-debug`，**不替代用户**执行
5. **禁止**在端口已就绪时仍输出"请重启 Chrome"提示（白让用户操作一次还会丢 tab）
6. **禁止**为了让 dev server 与 9222 不冲突，把 dev server URL 写成 `127.0.0.1:<port>`；dev server URL 必须用局域网 IP（详见 `dev-server-host.mdc`）

## 用户侧配置

macOS 用户可在 `~/bin/chrome-debug` 或 Chrome 启动配置中永久添加 `--remote-debugging-port=9222`，避免每次手动操作。

## 与 dev-server-host 规则的协作

`dev-server-host.mdc` 要求 dev server 给用户的访问 URL 用 `http://<lan-ip>:<port>` 而非 `localhost` / `127.0.0.1`，这同时也避免和本规则的 9222 调试端口在主机名上混淆。两条规则配合使用，保证「业务 URL 走 LAN IP，调试端口走 127.0.0.1:9222」的清晰分工。
