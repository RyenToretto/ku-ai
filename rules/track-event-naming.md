# track-event-naming.mdc 设计说明

## 设计意图

让所有埋点上报（悬壶 / FB Pixel / GA / xh-report）保持统一可机读的命名规范，避免：

1. PM 文档里 `PageView` / `ViewContent` 是 PascalCase，代码里随手用 `event_time` 是 snake_case，结果同一业务事件在不同平台名称对不上、漏斗拼不起来
2. 散落字符串字面量调用 `$attrTracker('xxx')`，未来重命名 / 类型推导 / 自动联想全部失效
3. 开发只埋 PM 列出的事件，线上出问题无法诊断（资源加载失败 / 弹窗主动关闭 / 漏斗损耗节点都看不见）

## 适用场景

- 所有需要前端埋点上报的项目（fe-picpopop / fe-pixpop 等）
- 自动应用（`alwaysApply: true`）
- 涉及悬壶 / FB Pixel / GA / xh-report / Adjust 任一上报通道

## 三类核心约束

### 1. snake_case 命名

事件名 + 参数 key 全部 `snake_case`（小写 + 下划线分隔）。唯一例外是 FB Pixel **平台标准事件**（`PageView` / `ViewContent` / `InitiateCheckout` / `Purchase` 等保留名），保持 PascalCase 原样。

自定义事件名跨平台**复用同一字符串**：例如 `upload_success` 既是悬壶事件名，也是 `fbq('trackCustom', 'upload_success', ...)` 的自定义事件名，方便漏斗对齐。

### 2. 集中管理

所有事件名必须通过 `shared/constants/tracker.ts` 的 enum / 常量取值，**禁止**业务代码里散落字符串字面量。enum 注释里写清触发时机 + 必需参数。

### 3. 开发期主动补点

不依赖 PM 文档完整性。开发过程中遇到「失败可能 / 漏斗损耗 / 拦截命中」三类场景，必须主动补点。判定标准：

- 一旦出问题，没埋点没法定位
- 业务上是关键漏斗节点
- 涉及外部依赖（SDK / CDN / 网关）

## 配套检测

`ripgrep` 自检命令在规则末尾给出，CI 可以串到 lint 流程里跑。

## 与其他 rule 的关系

- 与 `sediment-judgement.mdc`：本 rule 是「埋点维度的沉淀规范」，sediment-judgement 是「全维度的沉淀判定」
- 与 `i18n.mdc`：都遵循「严格复用已有 key、禁止字面量散落」的同源思想
