# css-naming.mdc 设计说明

## 设计意图

阻止"根 block 名同时作为子元素 class 前缀"陷阱，同时强制根 class 与组件文件名一一对应，让 DevTools 元素树能直接反映组件归属。

## 适用场景

- 所有 Web 前端项目（Vue / React / Svelte / 原生）
- 自动应用（`alwaysApply: true`）

## 三条核心规则

### 规则 1：根 class 强制等于组件名 kebab-case

`AaaaBbb.vue` → 根 class 必须是 `aaaa-bbb`，**不允许加任何后缀**（不允许 `-page` / `-host` / `-root` / `-screen`）。

### 规则 2：同一组件内 class / id 互不为子串

根 class 不能被子元素 class 完整包含；反之亦然。子串关系按字符串子串判定，不依赖 `-` / `_` / `__` 分隔符。

### 规则 3：子元素 BEM block 用独立命名空间

由于规则 1 + 规则 2，子元素**不能**继续用根 class 作 BEM block。必须换成其他不与项目其他 class 冲突的命名空间。

命名空间优先级：
1. **业务语义** - `drama-ep__list`、`oss-pick__input`
2. **功能简称** - `creator-form__panel`、`tag-picker__chip`
3. **避免缩写** - `elf__xxx`、`ouv__xxx`（语义模糊）

## 典型违规与修正

| 违规 | 修正 |
|------|------|
| `OssUploader.vue` 根 `.oss-uploader-host` + 子 `.oss-uploader__input` | 根 `.oss-uploader` + 子 `.oss-pick__input` |
| `EpisodeListField.vue` 根 `.episode-list-field` + 子 `.episode-list-field__item` | 根 `.episode-list-field` + 子 `.drama-ep__item` |
| `id="user"` + 子 `id="user-row"` | `id="user"` + 子 `id="user-row-0"`（语义化命名空间） |

## 联动既有 LESS 规则

- ✅ `&.is-active` / `&:hover` / `&::before`（伪类与状态类）
- ❌ `&__item` / `&-success` / `&--large`（类名字符串拼接）
- ❌ 根 class 与子元素 class 互为子串（本规则）

## 自检命令

```bash
# 子串关系检测
rg --pcre2 'class="[^"]*\b<root>[\-_a-zA-Z0-9]+' src/
rg '\.<root>[\-_]' src/

# 命名空间冲突检测
rg '\.<new-namespace>[\-_\s\{,:]' src/
```

## 例外

- 公共工具类（`.is-active` / `.with-affix-header`）不参与
- 三方组件库类（element-ui / ant-design）不参与
- 规则约束源代码可读层级，不影响 Vue scoped style 编译产物
