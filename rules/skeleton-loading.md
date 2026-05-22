# skeleton-loading.mdc 设计说明

## 设计意图

`fe-picpopop` 项目里历史上骨架屏 (loading 占位) 各自为政:

- 首页 `ExploreWaterfallFlow` / `EffectWaterfallFlow` / `DoMediaSkeleton`: 各自定义灰底
  + 横向 shimmer + 私有 `@keyframes shimmer / skeletonLoading`, 视觉相近但代码重复 4 处
- `/start` `StartTemplateRow`: 粉紫霓虹光晕 + 135° 斜向高对比扫光 (`skel-neon-shimmer`
  / `skel-neon-pulse`), 跟首页视觉完全不一致, 用户在不同页面会看到两种"loading"样式

PM 2026-05-18 决议: 全项目骨架屏视觉必须统一到首页配方 (灰底 + 横向 shimmer), 通过
SCSS mixin + 工具 class 双通道, 一处定义全局复用, 杜绝再有人自己拼骨架样式。

本规则把"唯一允许的使用方式" + "禁止反模式" + "自检命令" + "改造历史"全部写清, 让
未来加新骨架场景的人**没机会**走野路子。

## 适用场景

- 自动应用 (`alwaysApply: true`)
- 项目所有 Vue 单文件组件 (`.vue`) + SCSS module
- 任何 loading / 占位 / 数据未到位的 DOM 区块

## 核心规则

### 唯一允许的使用方式

| 方式 | 何时用 | 示例 |
|---|---|---|
| `@include doSkeleton($radius)` mixin | 需要自定义圆角 / 嵌入复杂 layout | `@include doSkeleton(11px)` |
| `.do-skeleton` 工具 class | DOM 直接套, 不写 SCSS | `<div class="do-skeleton do-skeleton--md"></div>` |

mixin 位置: `app/assets/css/common/mixins.scss` (通过 vite `additionalData` 全局自动 `@use`)

工具 class 位置: `app/assets/css/modules/skeleton.scss` (通过 `assets/css/index.scss`
全局 forward)

### 三条禁止

1. **不允许自己拼 background + ::after + 私有 @keyframes** (本地重复定义灰底/渐变/动画)
2. **不允许用 brand 色 / 霓虹光晕 / 斜向高对比扫光做骨架** (那是装饰, 不是骨架)
3. **不允许在 .vue 内重复定义 @keyframes shimmer / skeletonLoading** (全局已有, 复用即可)

### 自检命令

```bash
rg -n '@keyframes (shimmer|skeletonLoading|skel-)' app/ -g '!app/assets/css/common/animation.scss'
rg -n 'skel-neon|neon-shimmer|neon-pulse' app/
rg -n 'rgba\(255,\s*255,\s*255,\s*0\.1\)' app/ -g '!app/assets/css/**'
```

输出应为空 (除了已迁移的合规 .vue)。

## 例外

- 激光扫描动效 (`LaserRevealCanvas`): 是 brand 互动动画, 不属"骨架屏"
- 数据加载完成后的过渡动画 (悬浮 / focus / 入场): 各组件自由实现
- 全屏 loading mask (`PixLoading`): 走 `pix-loading` 视觉, 不是占位
- 按钮内 spinner (`do-rotate`): 走 `i-custom-loading` 旋转, 不是占位

## 历史教训 (2026-05-18 改造)

| 文件 | 改造前 | 改造后 |
|---|---|---|
| `ExploreWaterfallFlow.vue` | 本地 18 行骨架 | `@include doSkeleton(16px)` |
| `EffectWaterfallFlow.vue` | 本地 18 行骨架 | `@include doSkeleton(11px)` |
| `DoMediaSkeleton.vue` | 双层 div + 本地 `@keyframes shimmer` | `@include doSkeleton(inherit)` |
| `StartTemplateRow.vue` | 粉紫霓虹 + 斜扫光 + 2 个本地 keyframes | `@include doSkeleton(10.386px)` |

合计删除 ~80 行重复 SCSS, 替换为统一的 1 行 `@include`。

## 与其他规则联动

- `css-naming.mdc`: 骨架 class 命名仍遵循 BEM (`block__skeleton`); `.do-skeleton` 是
  全局工具类例外
- `css-calc-interpolation.mdc`: 骨架 width/height 100% 不涉及 calc 插值
- `accessibility`: 骨架 div 应配 `aria-hidden="true"` 或父容器 `aria-busy="true"`,
  避免屏幕阅读器朗读空白占位
