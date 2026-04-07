# Chrome DevTools MCP 最佳实践

## 推荐做法

- 截图前先用 `emulate_device` 设置正确的视口尺寸
- 使用 `evaluate_script` 时，检查返回值是否为预期类型
- 配合设计稿还原流程使用：截图 → 对比 → 调整 → 再截图

## 常见陷阱

- Chrome 未启动或未开启调试端口会导致连接失败
- `evaluate_script` 中的异步操作需要 `await`
- 截图时页面可能未完全加载，需要先等待特定元素出现

## 与其他工具配合

- **Playwright MCP**：Playwright 更适合自动化测试流程，Chrome DevTools 更适合实时调试
- **Lighthouse MCP**：用 DevTools 定位具体性能瓶颈，用 Lighthouse 做整体评估
