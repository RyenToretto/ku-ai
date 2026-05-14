# i18n-error-code-aligned.mdc 设计说明

## 设计意图

让业务代码里的 `error_code` / `reason` 等枚举字符串值 **= i18n key 末段** (camelCase),
父组件 / 调用方直接用模板字符串 `t(\`x.y.${code}\`)` 取文案, **彻底消除三元 / 查表映射**。

避免:

1. **三套命名同义不同名**: 子组件 emit 一套 (如 `unsupported_format`), tracker 接收一套
   (如 `mime_invalid`), i18n 又一套 (如 `mimeInvalid`), 心智负担成倍叠加
2. **三元链膨胀**: 每加一种新错误就得改 4 处 (子组件 + 父组件 if + tracker enum + i18n)
3. **fallback 错误覆盖真实 case**: 父组件 `allowed` 数组写错命名时, 所有错误被错误兜底
   到默认 case, 真正的差异化反馈失效。本仓库 2026-05 真实 bug 即此。

## 适用场景

- 所有需要把 error_code / reason 字符串值映射到 i18n 文案的项目
- 自动应用 (`alwaysApply: true`)
- 涉及子组件 emit / Pinia store action 返回 / fetch 错误码 → toast / 表单提示 等场景

## 核心约束

### 1. error_code 值 = i18n key 末段 (camelCase)

```js
// i18n
start: { upload: { error: { mimeInvalid: '...', rawTooLarge: '...', compressFailed: '...' } } }

// 子组件
emit('pick-error', { error_code: 'mimeInvalid' }) // 与 i18n key 末段同名

// 父组件
doTips.error(t(`start.upload.error.${payload.error_code}`)) // 一行模板字符串
```

### 2. tracker 边界做 case 转换 (BI 兼容)

业务侧用 camelCase, tracker 上报值历史是 snake_case, **只在 tracker 函数内部一行
`toReportCase` 转**, 业务调用方零感知:

```ts
const toReportCase = (s: string): string =>
  s.replace(/([A-Z])/g, '_$1').toLowerCase()
```

### 3. 数字 sentinel 例外

服务端返回的数字 failCode (1 / 5 / 6 等业务约定): 数字到 camelCase 名的转换可保留单层
ternary 或 const map (例如 `failCode === 1 ? 'imageRejected' : 'generateFail'`), 因为
数字 sentinel 是协议常量, 不参与"命名 = i18n key"的对齐。

## 自检命令

```bash
# 找潜在的 error_code/reason 三元映射 (review 重点)
rg --pcre2 -n "(error_code|reason|code|status)\s*===.*\?\s*['\"][\w.]+['\"]\s*:" app/

# 找 errKey/i18nKey 类的查表/三元变量名 (强嫌疑信号)
rg -n "(errKey|errorKey|i18nKey|messageKey)\s*=" app/
```

理想状态: 业务代码里 error → toast 永远是一行 `doTips.x(t(\`ns.${code}\`))`。

## 与其他规则的关系

- **track-event-naming.mdc**: 事件名 (event_name) 必须 snake_case, 本规则约束的是 payload
  里的 value (camelCase 业务值 → tracker 边界转 snake), 互不冲突
- **i18n.mdc**: i18n key 末段 camelCase, 本规则保证 error_code 与之 1:1 同名

## 历史教训

2026-05 `app/pages/start.vue` `onUploadError`:
- 子组件 emit `unsupported_format` / `too_large` / `compress_fail`
- 父组件 `allowed: ['mime_invalid', 'raw_too_large', 'compress_failed', 'no_file']`
  (写的是 tracker 命名, 与子组件不一致)
- → `(allowed).includes(payload.error_code)` 永远 false → 全部 fallback 到 `compress_failed`
- → 真实 mime/size 错误的 toast 永远不弹

改成"业务侧 camelCase + 与 i18n 末段对齐 + 模板字符串"后:
- 子组件 emit `mimeInvalid` / `rawTooLarge` / `compressFailed`
- 父组件一行 `doTips.error(t(\`start.upload.error.${code}\`))`
- 4 处映射 + 1 个 bug 同时消失
