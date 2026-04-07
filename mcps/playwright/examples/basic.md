# Playwright MCP 使用示例

## 登录测试

```
1. browser_navigate 到 https://www.pixpop.ai/login
2. browser_snapshot 获取页面结构
3. browser_fill 填写邮箱输入框
4. browser_fill 填写密码输入框
5. browser_click 点击登录按钮
6. browser_snapshot 验证是否跳转到首页
```

## 页面功能验证

```
1. 导航到首页
2. 截图当前状态
3. 点击 "创建视频" 按钮
4. 等待页面跳转
5. snapshot 验证页面标题和表单元素存在
```

## 响应式测试

```
1. 导航到目标页面
2. 调整 viewport 为 375x812（移动端）
3. 截图保存
4. 调整 viewport 为 1920x1080（桌面端）
5. 截图保存
6. 对比两个截图的布局差异
```
