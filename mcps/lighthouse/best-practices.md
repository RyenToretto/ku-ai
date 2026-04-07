# Lighthouse MCP 最佳实践

## 推荐做法

- 每次重大变更后运行一次审计，防止性能回退
- 分别用 mobile 和 desktop 模式测试
- 关注核心 Web Vitals：LCP、FID/INP、CLS
- 将审计结果中的具体建议直接应用到代码

## 审计策略

1. **开发阶段**：用 `throttling: none` 快速检查
2. **提交前**：用 `throttling: simulated` + `device: mobile` 严格测试
3. **上线后**：对生产 URL 做完整审计

## 优化优先级

| 指标 | 目标 | 影响 |
|------|------|------|
| Performance | >= 90 | 用户体验和 SEO |
| Accessibility | >= 90 | 合规和可用性 |
| Best Practices | >= 90 | 代码质量 |
| SEO | >= 90 | 搜索排名 |

## 与其他工具配合

- **Chrome DevTools MCP**：Lighthouse 发现问题后，用 DevTools 定位具体元素
- **Playwright MCP**：先用 Playwright 导航到需要审计的页面状态（如登录后）
- **Performance Cursor Rule**：将 Lighthouse 建议转化为项目规范
