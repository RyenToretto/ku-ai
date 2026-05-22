---
name: tailwind-css
description: >-
  Tailwind CSS utility-first styling best practices. Use when writing
  styles, creating responsive layouts, or customizing themes in Vue/Nuxt
  components that use Tailwind CSS via @nuxt/ui.
---

# Tailwind CSS Best Practices for Nuxt UI Projects

## Important: This project uses BEM + SCSS, NOT utility classes in templates

This project enforces `css-bem.mdc` rule: **no Tailwind utility classes in DOM**.
All styling is done via SCSS with BEM naming. Tailwind is used only through
`@nuxt/ui` component props and theme configuration.

## When Tailwind Knowledge Applies

- Configuring `@nuxt/ui` component `ui` prop overrides
- Understanding Tailwind design tokens (colors, spacing, breakpoints)
- Working with `app.config.ts` theme customization
- Understanding responsive breakpoints: `sm:640` `md:768` `lg:1024` `xl:1280` `2xl:1536`

## Design Tokens Reference

```
Spacing: 0 0.5 1 1.5 2 2.5 3 3.5 4 5 6 7 8 9 10 11 12 14 16 20 24 28 32 36 40 44 48 52 56 60 64 72 80 96
Font sizes: xs(12) sm(14) base(16) lg(18) xl(20) 2xl(24) 3xl(30) 4xl(36) 5xl(48)
Border radius: none sm(2) DEFAULT(4) md(6) lg(8) xl(12) 2xl(16) 3xl(24) full(9999)
```

## @nuxt/ui Component Styling Pattern

```vue
<!-- Use ui prop for Tailwind-based overrides, not class attributes -->
<UButton
  :ui="{
    base: 'rounded-full font-semibold',
    padding: { sm: 'px-4 py-2' }
  }"
/>
```

## Responsive Design

- Mobile-first approach: base styles for mobile, then `md:` and `lg:` for larger
- Project uses `@include mobile()` and `@include pc()` SCSS mixins
- Breakpoint reference: mobile < 768px, tablet 768-1024px, desktop > 1024px
