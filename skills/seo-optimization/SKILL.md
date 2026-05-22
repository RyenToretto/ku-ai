---
name: seo-optimization
description: >-
  SEO optimization best practices for Nuxt SSR applications. Use when
  working with page metadata, structured data, sitemap, or any
  SEO-related implementation.
---

# SEO Optimization for Nuxt SSR

## Core API — usePageSeo

项目统一使用 `usePageSeo` composable 管理页面 SEO，禁止在页面中直接调用 `useHead` / `useSeoMeta` 设置 title/description。

```typescript
const { t } = useI18n()
usePageSeo({
  title: computed(() => t('seo.page.title')),
  description: computed(() => t('seo.page.description')),
  keywords: computed(() => t('seo.page.keywords')),  // 可选
  image: 'https://cdn.pixpop.ai/og-image.png',       // 可选
  type: 'website'                                      // 可选
})
```

`usePageSeo` 自动处理：canonical URL、og:title/description/image/url、twitter card、keywords meta。

## Rules

1. **禁止硬编码 SEO 文本**：所有 title / description / keywords 必须使用 i18n key
2. **参数必须响应式**：使用 `computed(() => t(...))` 包裹，确保语言切换后更新
3. **禁止用 definePageMeta 设置 title/description**：与 usePageSeo 冗余，无消费者
4. **每个页面必须有唯一的 title + description**：禁止多个页面共用同一组 i18n key，每个路由对应独立的 `seo.<page>.*` key
5. 图片必须有语义化的 `alt` 属性（通过 i18n）
6. 语义化 HTML：每页一个 `<h1>`，正确的标题层级

## 结构化数据（JSON-LD）

### 全局身份 JSON-LD（Organization / WebSite）

面向搜索引擎爬虫的**全局身份**数据（Organization、WebSite）应使用**纯静态英文**内容，禁止使用 `t()` 动态调用——爬虫只看 SSR 输出，多语言切换无意义：

```typescript
// ✅ 纯静态，不依赖 i18n
useHead({
  script: [{ type: 'application/ld+json', innerHTML: JSON.stringify({
    '@context': 'https://schema.org', '@type': 'Organization', name: 'PixPop'
  }) }]
})
```

### 页面级 JSON-LD（Product / AggregateOffer 等）

包含**运行时动态数据**（API 返回的价格、标题等）的 JSON-LD 必须用 `computed` 包裹：

```typescript
const jsonLd = computed(() => JSON.stringify({
  '@context': 'https://schema.org',
  '@type': 'SoftwareApplication',
  name: 'PixPop',
  offers: hasData.value ? { '@type': 'AggregateOffer', ... } : undefined
}))

useHead({
  script: computed(() => [{ type: 'application/ld+json', innerHTML: jsonLd.value }])
})
```

即使动态数据为空，也应输出基础结构（不含可选字段），避免搜索引擎完全无法识别页面类型。

## iframe / 内嵌页面

嵌入在 iframe 中的页面（如支付结果页 `pay/success`、`pay/cancel`）搜索引擎不应索引：

```typescript
useSeoMeta({ robots: 'noindex, nofollow' })
```

在 `usePageSeo` 之后追加即可，不要在 `usePageSeo` 中增加通用 robots 参数。

## Nuxt Modules in Use

- `@nuxtjs/sitemap` — 自动生成 sitemap.xml
- `@nuxtjs/i18n` — 自动处理 hreflang 标签

## 图片性能（影响 SEO）

- 首屏图片：`fetchpriority="high" loading="eager"`
- 非首屏图片：`loading="lazy"`
- 设置明确的 width/height 减少 CLS
- 优先使用 `<NuxtImg>` / `<NuxtPicture>`
