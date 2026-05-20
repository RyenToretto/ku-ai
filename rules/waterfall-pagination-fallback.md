# waterfall-pagination-fallback.mdc

## 设计意图

服务端分页接口偶尔会出现 `totalSize` 字段不准确（多数因历史原因 / 多租户复用 / 全量统计未走筛选条件等）。如果前端瀑布流的 `hasMore` 完全依赖 `totalSize`，命中不准的接口会陷入"无限翻页 + 重复 append"循环——用户感知为页面出现大量重复卡片、无限加载。

本规则把判定主权收回到「**本次返回 `data.length < pageSize` 即为最后一页**」这条对所有正常分页都成立的不变量上，`totalSize` 仅作为辅助上限。同时强制 append 前按 `id` 去重作为防御性兜底。

## 适用场景

- 任何分页瀑布流 / 滚动加载组件
- 接口契约带 `data[]` + `totalSize` + `curPage` + `pageSize` 的标准分页形态
- 未来引入新分页接口时一并遵循

## 历史背景

2026-05 fe-picpopop 项目 PM 反馈 `/v1/web/homepage/tags/template` 接口 `totalSize` 不准（筛选后实际 2 条但返回 430），前端 `ExploreWaterfallFlow.vue` 完全依赖 totalSize → 无限请求 + 重复卡片。修复后三个瀑布流组件统一使用本规范。

## 关键不变量

1. `isLastPage = waitList.length < PAGE_SIZE` 作为 `hasMore` 的**总开关**（短路终止）
2. append 前按 `id` 去重（防御性兜底，无副作用）
3. `hasMore` 与 `totalSize` 做 AND 关系，**禁止**单独依赖 totalSize

## 对正常接口零负担

- 正常接口最后一页 `length` 本来就 `< pageSize`，新逻辑只比旧逻辑提前一步终止（少一次"空响应"请求），数据完整性等价
- id 去重的成本：每页一次 `Set` 构造 + `filter`，pageSize=20 量级 O(n)，零感知
