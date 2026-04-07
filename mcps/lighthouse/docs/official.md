# Lighthouse MCP 官方文档

## 文档链接

- Lighthouse 官方: https://developer.chrome.com/docs/lighthouse
- lighthouse-mcp GitHub: https://github.com/nicholasoxford/lighthouse-mcp
- npm: https://www.npmjs.com/package/lighthouse-mcp

## 核心概念

### 审计类别

| 类别 | 检测内容 |
|------|---------|
| Performance | FCP, LCP, TBT, CLS, Speed Index |
| Accessibility | WCAG 合规、ARIA、色彩对比度、键盘导航 |
| Best Practices | HTTPS、JS 错误、过时 API、安全头 |
| SEO | meta 标签、robots.txt、移动端适配、结构化数据 |
| PWA | manifest、service worker、离线支持 |

### 核心 Web Vitals

| 指标 | 好 | 需要改进 | 差 |
|------|---|---------|---|
| LCP (最大内容绘制) | <= 2.5s | <= 4.0s | > 4.0s |
| INP (交互延迟) | <= 200ms | <= 500ms | > 500ms |
| CLS (累积布局偏移) | <= 0.1 | <= 0.25 | > 0.25 |

### 限速模式

- **Simulated**：基于数学模型模拟慢速网络（默认、推荐）
- **DevTools**：通过 Chrome DevTools Protocol 实际限速
- **None**：无限速，适合本地开发快速检查
