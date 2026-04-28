# css-naming.mdc 设计说明

## 设计意图

阻止 BEM 风格中常见的"根 block 名同时作为子元素 class 前缀"陷阱：当 `.foo` 是根、`.foo__bar` 是子元素时，`foo` 字符串子串关系会引起多处工程问题（搜索误伤、属性选择器歧义、CSS 重置插件按 prefix 误匹配、重构级联）。

本规则要求根元素的 class（或 id）与子元素的 class（或 id）**互不为子串**。

## 适用场景

- 所有 Web 前端项目（Vue / React / 原生）
- 自动应用（`alwaysApply: true`）

## 核心要点

1. **互不为子串约束**：同一组件内根元素的 class/id 不允许被任何子元素 class/id 完整包含；反之亦然。
2. **命名后缀模式**：
   - 页面级容器：`-page` / `-screen` / `-view`
   - 通用组件 / 表单字段：`-host` / `-root`
   - 弹层容器：`-dialog-host`
3. **子元素继续 BEM**：`block` / `block__element` / `block--modifier` / 状态类 `is-xxx` 不变
4. **联动既有 LESS 规则**：禁止 `&__item` / `&-success` / `&--large` 类名拼接；允许 `&.is-xxx` / `&:hover` / `&::before`
5. **公共类与三方组件不受约束**：`.is-active` / `.with-affix-header` / element-ui 输出类等

## 典型违规与修正

| 违规 | 修正 |
|------|------|
| 根 `.foo` + 子 `.foo__bar` | 根 `.foo-host` + 子 `.foo__bar` |
| 根 `.user-list` + 子 `.user-list-item` | 根 `.user-list-page` + 子 `.user-list__item` |
| `id="user"` + 子 `id="user-row"` | `id="user-section"` + 子 `id="user-row"` |

## 自检命令

```bash
# 替换 <root> 为根 class 名
rg --pcre2 'class="[^"]*\b<root>[\-_a-zA-Z0-9]+' src/
rg '\.<root>[\-_]' src/
```
