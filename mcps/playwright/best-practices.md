# Playwright MCP 最佳实践

## 核心工作流

1. 先用 `browser_tabs` 检查已有标签页
2. 用 `browser_snapshot` 获取页面结构（主要信息来源）
3. 通过 snapshot 中的 ref 执行交互操作
4. 操作后重新 `browser_snapshot` 验证结果

## 推荐做法

- 每次交互前先 `browser_snapshot` 获取最新的元素 ref
- 使用 `browser_scroll` + `scrollIntoView: true` 确保元素可见后再点击
- 使用 `browser_hover` 触发下拉菜单/tooltip 后再操作内部元素
- 表单填充用 `browser_fill`（替换内容），追加文本用 `browser_type`

## 常见陷阱

- ref 是基于最新 snapshot 的，页面变化后 ref 失效，需重新 snapshot
- iframe 内的元素不可直接访问
- 对话框（alert/confirm/prompt）不会阻塞自动化
- 不要对同一个失败操作重试超过 4 次
- 截图坐标只对当前 viewport 有效，执行其他操作后需重新截图

## 与其他工具配合

- **Chrome DevTools MCP**：DevTools 适合调试样式和性能，Playwright 适合功能测试
- **Lighthouse MCP**：先用 Playwright 导航到页面，再用 Lighthouse 做审计
