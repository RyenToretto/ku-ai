# dev-server-host 规则说明

## 设计意图

本地开发服务器（Vite / Nuxt / Next / Webpack / Express / Python http.server 等）默认会同时打印 `Local: http://localhost:<port>` 和 `Network: http://<lan-ip>:<port>` 两个入口。AI Agent 在回报 URL、调用 Playwright `browser_navigate`、或为用户复制粘贴访问链接时，**默认**应当使用「局域网 IP + 端口」而不是 `localhost` / `127.0.0.1`。

理由：

1. **真机 / 跨设备调试**：移动端、平板、虚拟机、同事电脑都需要从局域网访问当前 dev server，`localhost` 仅本机可用
2. **避免与 9222 调试端口生态歧义**：Playwright Chrome 调试用 `127.0.0.1:9222`，业务 dev server 也用 `127.0.0.1` 时容易误连
3. **与上线域名行为更一致**：cookie SameSite、CORS、Mixed Content、WebSocket Origin 等问题在 `localhost` 下经常被默认放行，IP 暴露能更早发现
4. **跨容器 / 跨 Profile 行为统一**：MCP 在不同 user profile / Docker 容器里跑时，`localhost` 解析行为可能不一致；用 IP 永远确定指向开发机

## 适用场景

- 任何 `npm run dev` / `pnpm dev` / `yarn dev` / `vite` / `nuxt dev` / `next dev` 启动的本地开发服务器
- Agent 在响应中向用户提供「访问入口」URL
- Agent 调用 Playwright MCP 的 `browser_navigate` 等工具
- 真机扫码访问、跨设备调试、团队成员间分享调试链接

## 关键约束

1. 启动命令必须监听所有网卡（如 `vite --host 0.0.0.0` / `next dev -H 0.0.0.0`）
2. 给用户的 URL 必须是 `http://<lan-ip>:<port>` 形式
3. 探测 IP 优先使用 `route get default` 拿到的活动网卡，再 `ipconfig getifaddr <iface>`，兜底 `en0` / `en1`
4. 全部失败（无网络）才允许退回 `127.0.0.1`，并显式说明
5. dev server 已只绑 `127.0.0.1` 时，先提示用户加 `--host 0.0.0.0` 重启，禁止用 `localhost` 凑合

## 例外

- 进程内自检脚本（`curl http://127.0.0.1:9222/json/version`）属合法回环
- CI 容器内服务互联默认 `localhost`
- OAuth 提供方强制要求 `localhost` 回调时按业务需要使用
