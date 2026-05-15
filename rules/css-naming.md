# css-naming.mdc 设计说明

## 设计意图

阻止"根 block 名同时作为子元素 class 前缀"陷阱, 同时强制根 class 与组件文件名一一对应, 让 DevTools 元素树能直接反映组件归属。

**2026-05-15 更新 (v2)**: 把"子元素之间也应避免互为子串"从软约束升级为铁律, 并明确禁止 BEM element-of-element 命名(`__cta` 与 `__cta-icon` 共存)。

## 适用场景

- 所有 Web 前端项目 (Vue / React / Svelte / 原生)
- 自动应用 (`alwaysApply: true`)

## 五条核心规则

### 规则 1: 根 class 强制等于组件名 kebab-case

`AaaaBbb.vue` → 根 class 必须是 `aaaa-bbb`, **不允许加任何后缀** (不允许 `-page` / `-host` / `-root` / `-screen`)。

### 规则 2: 同一组件内任意两个 class / id 必须互不为字符串子串 (铁律)

三种关系全部禁止:
- 根 class ↔ 任一子元素 class
- **任意两个子元素 class 之间** (旧规则曾用"应避免", 现升级为铁律)
- **同一 BEM block 的多个 element 名之间** (常见: `__cta` 与 `__cta-ico`)

子串判定按字符串子串, 不依赖 `-` / `_` / `__` / `--` 分隔符。

### 规则 3: 子元素 BEM block 用独立命名空间

子元素**不能**用根 class 作 BEM block。

### 规则 4: BEM element 命名扁平化, 禁止 element-of-element

`block__a` 与 `block__a-x` 共存 = 前缀子串违规 = 禁止。

二选一修复:
1. **首选**: 扁平化 element, 多个 element 平级共享 block 前缀, 名字彼此互不为子串
2. **次选**: 用 BEM modifier `block__a--variant`

#### 单复数陷阱

`__feature` (单项) 与 `__features` (容器) 互为前缀子串, 必须改名:
- `__features` → `__feature-list`
- `__feature` → `__feature-item`

### 规则 5: BEM block 名不应单独作 class 使用

`block` 单独作 class + `block__element` 必为前缀子串, 触发规则 2。修复:
1. 用 wrap element 包住: `<button class="topbar-home-btn"><span class="topbar-home-icon" /></button>` (扁平兄弟独立 class)
2. 把 block 单独使用语义改成独立 class, 不再用作 element 前缀

## 典型违规与修正 (2026-05 fe-picpopop 实战记录)

| 违规 | 类型 | 修正 |
|------|------|------|
| `.blur-curtain__cta` + `.blur-curtain__cta-ico` + `.blur-curtain__cta-label` | 规则 4 | `__cta` + `__icon` + `__label` |
| `.result-cta__btn` + `.result-cta__btn-ico` + `.result-cta__btn-label` | 规则 4 | `__btn` + `__icon` + `__label` |
| `.paywall-modal__plan` + `__plans` + `__plan-name` + `__plan-price` + `__plan-strike` + `__plan-badge` + `__plan-right` | 规则 4 (单复数 + element-of-element 双违规) | `__sku-card` + `__sku-list` + `__sku-name` + `__sku-price` + `__sku-strike` + `__sku-badge` + `__sku-right` |
| `.paywall-modal__feature` + `__features` + `__feature-ico` + `__feature-text` | 规则 4 | `__check-item` + `__check-list` + `__check-icon` + `__check-text` |
| `.paywall-modal__trust` + `__trust-ico` + `__trust-text` + `__trust-dot` | 规则 4 | `__trust-row` + `__trust-icon` + `__trust-text` + `__trust-dot` |
| `.paywall-modal__title` + `__subtitle` (title 是 subtitle 子串) | 规则 4 (巧合) | `__title` + `__lead` |
| `.gen-mask__inner` + `__spinner` (inner 是 spinner 子串) | 规则 4 (巧合) | `__body` + `__spinner` |
| `.topbar-home` 单独 + `.topbar-home__ico` | 规则 5 | `.topbar-home-btn` + `.topbar-home-icon` (兄弟独立 class) |
| `.tpl-cell` 单独 + `.tpl-cell__media` | 规则 5 | `.tpl-cell` + `.tpl-media` (兄弟独立 class) |

## 联动既有 LESS 规则

- 允许 `&.is-active` / `&:hover` / `&[disabled]` / `&::before`
- 禁止 `&__item` / `&-success` / `&--large` (类名字符串拼接)
- 禁止根 class 与子元素 class 互为子串 (本规则)
- 禁止同一 block 的两个 element 名互为前缀子串 (本规则规则 4)

## 自检命令

### 同一 BEM block 内 element 间前缀子串扫描

```bash
# 抽取一个文件所有 BEM element 名 (block__element 形式)
rg -o '\.[a-z][a-z0-9-]*__[a-z][a-z0-9-]*' <file> \
  | sort -u \
  | awk -F'__' '{print $2}' \
  | sort -u

# 然后人工/脚本两两比对 case-in-substr
rg -o '__[a-z][a-z0-9-]*' <file> | sort -u | while read a; do
  rg -o '__[a-z][a-z0-9-]*' <file> | sort -u | while read b; do
    [[ "$a" != "$b" ]] && case "$a" in *"$b"*) echo "VIOLATION: $a contains $b in <file>" ;; esac
  done
done
```

### 标准 grep 子串关系自检

```bash
# 检查根 class 与子元素子串关系 (替换 <root>)
rg --pcre2 'class="[^"]*\b<root>[\-_a-zA-Z0-9]+' src/
rg '\.<root>[\-_]' src/

# 跨文件搜索新 namespace 是否冲突
rg '\.<new-namespace>[\-_\s\{,:]' src/
```

## 例外

- 公共工具类 (`.is-active` / `.with-affix-header` / `.do-form-section`) 不受约束
- 三方组件库 (element-ui / ant-design) 输出 class 不受约束
- **BEM modifier**: `__btn` 与 `__btn--again` / `__btn--download` 是 BEM 标准 modifier 关系, **不算违规** (`--` 后是同 element 的形态修饰, 不是新 element)
- scoped style 编译后属性选择器, 本规则约束的是源代码可读层级

## 真实案例: 2026-05 fe-picpopop StartBlurMask.vue 截图问题

PM 反馈截图: IDE 搜索 `.blur-curtain__cta` 同时高亮 5 处 (`__cta` × 2 + `__cta-ico` × 2 + `__cta-label` × 1), 想精确定位某一处时极其干扰; 用 `:global([class*="blur-curtain__cta"])` 选择器会误命中 ico/label。

修复方案 (扁平化 element):

```less
// 旧 (违规)
.blur-curtain__cta { ... }
.blur-curtain__cta-ico { ... }    // ❌ '__cta' 是 '__cta-ico' 前缀
.blur-curtain__cta-label { ... }  // ❌ '__cta' 是 '__cta-label' 前缀

// 新 (合规)
.blur-curtain__cta { ... }
.blur-curtain__icon { ... }       // ✅ 独立 element 名
.blur-curtain__label { ... }      // ✅ 独立 element 名
// cta / icon / label 三者两两互不为子串
```

## 版本历史

- **v1** (2026-04): 初版规则 1/2/3, 规则 2 表述"子元素之间也应避免互为子串" (软约束)
- **v2** (2026-05-15): 升级软约束为铁律, 新增规则 4 (BEM element-of-element 禁止) + 规则 5 (BEM block 不可单独用作 class), 补 fe-picpopop 实战修正案例 + 自检脚本
