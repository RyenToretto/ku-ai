# Chrome DevTools MCP 官方文档

## 文档链接

- Chrome DevTools Protocol: https://chromedevtools.github.io/devtools-protocol/
- npm: https://www.npmjs.com/package/chrome-devtools-mcp

## 核心概念

Chrome DevTools MCP 封装了 Chrome DevTools Protocol (CDP)，让 AI Agent 能够：

1. **页面检查** — 获取 DOM 结构、computed styles、布局信息
2. **脚本执行** — 在页面上下文中运行 JavaScript
3. **截图** — 支持全页面、区域、设备模拟截图
4. **网络** — 监控和拦截网络请求
5. **性能** — 获取性能指标和 timeline 数据

## 连接方式

默认连接 `http://127.0.0.1:9222`，需要 Chrome 以调试模式启动：

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# 或通过命令行标志
open -a "Google Chrome" --args --remote-debugging-port=9222
```
