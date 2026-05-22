# css-calc-interpolation.mdc 设计说明

## 设计意图

避免在 CSS `calc()` 中直接写 `doRem(...)` 等预处理器函数，导致函数被原样输出、编译期函数没有执行，或被 CSS `calc()` 解析逻辑错误处理。

## 适用场景

- Vue SFC 的 `<style>` 样式块
- `.scss` / `.sass` / `.less` 样式文件
- 任意需要在 `calc()` 中混合百分比、视口单位、rem 工具函数的布局样式

## 核心规则

`calc()` 内调用编译期函数时，必须通过 `#{...}` 插值注入结果。

```scss
/* ❌ 错误 */
width: calc(100% - doRem(50));

/* ✅ 正确 */
width: calc(100% - #{doRem(50)});
```

## 允许直接写入的内容

- CSS 原生函数：`min()` / `max()` / `clamp()`
- CSS 变量：`var(--gap)`
- 浏览器环境变量：`env(safe-area-inset-bottom)`

## 自检命令

```bash
rg -n "calc\\([^;{}]*(doRem|pxToRem|vw)\\(" app/ shared/ -g '*.{vue,scss,sass,less}'
```
