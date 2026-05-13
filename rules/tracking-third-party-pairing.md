# tracking-third-party-pairing

## 设计意图

PM 强制要求：**除 FB Pixel 标准事件 `PageView` 外，任何第三方上报必须先走 `$xhTracker` 或 `$attrTracker` 业务上报**。这是为了：

1. **漏斗一致性**：业务上报（xh/GA）是公司内部口径，第三方（FB Ads）是投放归因口径，两者事件量必须相等才能算准确转化率
2. **可控开关**：第三方 SDK 加载失败 / AdBlock 拦截时，业务上报仍能跑完，反之亦然
3. **代码审计**：评审看一次业务 hook diff 就能确认所有上报渠道覆盖
4. **未来扩展**：接入 TikTok / Adjust 时只需在业务 hook 内追加一行，调用方零侵入

## 适用场景

- 任何 PM 提出的「FB 标准事件 / 自定义事件」上报需求（除 `PageView`）
- 未来接入任何新第三方分析 / 投放 SDK
- 已有业务上报点想新增第三方配对

## 核心约束

1. **业务上报在前, 第三方在后**：保证业务漏斗不依赖第三方 SDK 加载状态
2. **同 hook 封装**：业务 + 第三方上报必须在同一个 composable 内一次性触发
3. **事件名 snake_case 镜像**：FB `InitiateCheckout` → 业务 `initiate_checkout`
4. **跨平台对齐**：FB 自定义事件名与悬壶事件名应复用同一字符串（如 `upload_success`）

## 实现位置

| 项目 hook | 职责 |
|---|---|
| `app/composables/useFacebookTracker.ts` | ⛔ 内部封装层，业务代码禁止直接 import |
| `app/composables/useStartTracker.ts` | /start 投放承接页业务上报 + fb 配对 |
| `app/composables/usePricingTracker.ts` | 支付订阅业务上报 + fb Purchase 配对 |
| `app/composables/useRegisterTracker.ts` | 注册流程业务上报 + fb CompleteRegistration 配对 |

## 检测命令

```bash
# 业务代码裸 import useFacebookTracker (应为空)
rg -n "useFacebookTracker\(\)|from.*useFacebookTracker" app/ -g '!app/composables/use*Tracker.ts'

# fbq 直接调用 (除 PageView 初始化白名单, 应为空)
rg -n "fbq\(['\"]track" app/ -g '!app/plugins/00_a2_pixel.client.ts'

# 业务事件名混用 PascalCase (应为空)
rg -n "\\\$attrTracker\(['\"][A-Z]|\\\$xhTracker\(['\"][A-Z]" app/
```

## 与现有规则的关系

- 与 `track-event-naming.mdc` 互补
  - 旧规则：覆盖事件名 / 参数 key 命名 (snake_case)
  - 本规则：覆盖配对绑定 + hook 封装 + 第三方上报顺序

## 历史

- 2026-05-13 首次落地：PM 提出"裸 fb 调用不再允许"，新建本规则，同步修复 `usePaymentStore` / `RegisterForm` 两处违规
