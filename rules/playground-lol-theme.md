# playground-lol-theme

## 设计意图

在 `xh-report` 这类 **SDK 仓库** 中，`playground` 用于本地联调与演示 UI。该目录允许使用更重的视觉风格，但必须与 SDK 发布代码严格隔离，避免：

- SDK 体积/职责被 UI 主题污染
- 业务方误把演示皮肤当成 SDK 能力
- 主题 token 在仓库内扩散导致维护成本上升

## 适用场景

- 编辑 `playground/**/*.{vue,ts,scss}` 时自动提示风格与目录边界
- AI 生成 playground 页面/组件时，优先复用 `playground/src/styles/theme/` 与 `playground-*` 组件体系

## 关键约束（摘要）

- **仅 playground**：LOL 客户端风格；SDK `src/` 禁止承载主题资源
- **SCSS**：禁止 `&__` / `&--` / `&-` 的类名拼接；允许 `&.success` 这类状态连写
- **主题 partial**：只能放在 `playground/src/styles/theme/`，并由 `playground/src/styles/index.scss` 统一注入

## 同步说明

本规则源文件位于 `xh-report` 仓库：`.cursor/rules/playground-lol-theme.mdc`。
