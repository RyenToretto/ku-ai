# Chrome DevTools MCP 使用示例

## 截取移动端页面截图

```
请用 Chrome DevTools 模拟 iPhone 14（390x844）截取当前页面的截图
```

## 检查元素样式

```
用 Chrome DevTools 检查 .page-home 下的 .hero-title 元素的 computed styles，
包括 font-size、color、line-height、margin
```

## 执行脚本获取运行时数据

```
用 Chrome DevTools 在页面中执行以下脚本，获取所有图片的加载状态：
document.querySelectorAll('img').forEach(img => console.log(img.src, img.complete))
```

## 设计稿对比流程

```
1. 用 emulate_device 设置为 375x812（iPhone X）
2. 截取当前页面截图
3. 与 docs/design/preview/home.png 设计稿对比
4. 列出所有差异点（间距、颜色、字号）
```
