---
name: feishu-policy-sync
description: >-
  Synchronizes the seven legal / Payermax-review pages (Terms of Service,
  Privacy Policy, Subscription Agreement, Recurring Payment Authorization
  Terms, Subscription Cancellation, Subscription Confirmation Email Sample,
  Payment Receipt Email Sample) from the canonical Feishu wiki sources to the
  Nuxt Vue pages, including variable-dictionary constant mapping, dual-email
  convention (CONTACT_EMAIL vs SUPPORT_EMAIL), and Playwright verification.
  Use when the user asks to update / re-sync / re-check the policy or
  Payermax pages, or mentions any of the Feishu wiki URLs
  rqqmslsz9y.feishu.cn/wiki/W1bNwUlTIi1z0PkSTuAces2Qn4f or
  rqqmslsz9y.feishu.cn/wiki/OjBIwobyBiCAWTkUmymc4o72ngg, or modifies any of
  app/pages/{terms-of-service,privacy-policy,subscription-agreement,
  payment-terms,merchant-agreement-termination,email-confirmation,
  billing-statement}.vue, or mentions Payermax / 代扣.
---

# Feishu Policy Sync

唯一真源（飞书云文档）：

- **协议总集**（terms / privacy / subscription）：<https://rqqmslsz9y.feishu.cn/wiki/W1bNwUlTIi1z0PkSTuAces2Qn4f>
- **Payermax 订阅申请**（payment-terms / merchant-agreement-termination / email-confirmation / billing-statement）：<https://rqqmslsz9y.feishu.cn/wiki/OjBIwobyBiCAWTkUmymc4o72ngg>

## 受影响文件

### A. 三个对外协议页（用户可在 FooterSection 直链访问）

| 文件 | 作用 | 飞书源 |
|------|------|--------|
| `app/pages/terms-of-service.vue` | 服务条款（约 19 章节） | W1bNwUlTIi1z0PkSTuAces2Qn4f |
| `app/pages/privacy-policy.vue` | 隐私政策（约 15 章节） | W1bNwUlTIi1z0PkSTuAces2Qn4f |
| `app/pages/subscription-agreement.vue` | 订阅协议（章节数会变化，当前 9） | W1bNwUlTIi1z0PkSTuAces2Qn4f |

### B. 四个 Payermax 审核页（普通用户无入口，**不进 FooterSection**）

形态总则（v2）：**参照 cubetv.cc 审核样板的真页面形态 + 已登录接真接口 / 未登录 demo 兜底**。
- 飞书 §2 / §3 是邮件设计稿截图，**不再以"邮件文本预览"落地**，而是按 cubetv 样板做成功能页。
- 已登录展真用户邮箱 / 真订阅 / 真账单；未登录展 cubetv 风格 demo（方便审核访问）。
- SSR-safe：登录态依赖客户端 store，所有"按登录态切换"的内容必须包在 `<ClientOnly>` 里。

| 文件 | 路由 | 审核样板 | 飞书源章节 / 形态 |
|------|------|----------|-------------------|
| `app/pages/payment-terms.vue` | `/payment-terms` | <https://cubetv.cc/payment-terms> | §1 代扣协议（6 ARTICLE）；纯条款长文，正文以飞书为准；ARTICLE 5 §Cancellation Path 末尾追加跳转链接到 `/merchant-agreement-termination` |
| `app/pages/merchant-agreement-termination.vue` | `/merchant-agreement-termination` | <https://cubetv.cc/merchant-agreement-termination> | §所需资料 #3；**自助退订表单**（email / userId / orderId 三 input + Unsubscribe 按钮）；已登录 prefill 走 `useSubscriptionOrderStore.confirmCancelSubscription`，未登录点击触发 `userStore.showLogin()` 兜底（仓库无匿名退订接口） |
| `app/pages/email-confirmation.vue` | `/email-confirmation` | <https://cubetv.cc/email-confirmation> | §2 代扣签约邮件样例；**订阅确认邮件卡片页**；已登录 Email 取自 `userStore.currentAccountInfo.email`、Subscription Status 取自 `userStore.isVip` / `membershipStatus`，未登录 fallback `user@example.com` + `Premium Active` |
| `app/pages/billing-statement.vue` | `/billing-statement` | <https://cubetv.cc/billing-statement> | §3 代扣扣款 receipt 邮件；**账单详情页**（v3 per-card example 重构）；4 张内容卡片：Order Summary / Payment Method / Subscription History / Credits Purchase History；顶部「当前账单」三级降级取数：① 订阅历史非空→取最后一笔订阅；② 订阅空+积分非空→取最后一笔积分；③ 两者都空→展 demo + 右上 EXAMPLE 印章 + amber 虚线边框；Subscription History / Credits Purchase History 两张卡片**各自独立判定**（真数据 / 空状态文案 / demo+example）；Customer Email 卡片永远不挂 example（未登录展 `your@mail.com`，登录展真邮箱） |

### C. 共用样式

| 文件 | 作用 |
|------|------|
| `app/assets/css/modules/article.scss` | 7 个页面统一排版（`.deep-article` / `ol` / `ul`） |

> 内容审核政策（旧 W1bNwUlTIi1z0PkSTuAces2Qn4f cell-66/67）当前**不需要**前端落地，跳过。

## 硬性约定

1. **禁止接入 i18n**。所有 7 个页面正文/标题/SEO 均用硬编码英文 + 文件内常量，`useI18n()` 不要出现。
2. **变量字典必须用常量映射**。飞书「变量字典」表格里的 `{{XXX}}` 占位符在 `<script setup>` 顶部统一定义为常量，模板中通过 `{{ }}` 插值，**禁止散点硬编码**。Payermax 文档目前没有独立的「变量字典」表，但其正文里出现的 `LUNARETH LIMITED` / 公司地址 / `contact@asterlyn.com` / `Effective Date` 同样必须以常量形式维护。
3. **正文以飞书为准**。每次同步直接整体覆盖 `<section>` 内容，但变量值仍走常量插值——即使飞书源已直接展示替换后的真实值，本地也仍然走常量。
4. **`BRAND_NAME` 与 `COMPANY_NAME` 严格区分**：
   - `BRAND_NAME = 'PixPop'`：产品名，用于 SEO title / description / 用户面向文案
   - `COMPANY_NAME = 'LUNARETH LIMITED'`：法律实体，用于协议正文里的「the Company / we」主体
5. **双邮箱约定**：本仓库存在两个不同语义的对外邮箱，**严禁串用**：
   - `CONTACT_EMAIL = 'business@asterlyn.com'`：商务 / 法律咨询邮箱，仅用于 A 组（`terms-of-service` / `privacy-policy` / `subscription-agreement`）的 Contact Information 章节
   - `SUPPORT_EMAIL = 'contact@asterlyn.com'`：客服 / 代扣支持邮箱，仅用于 B 组（4 个 Payermax 审核页）
   - 验收脚本里有显式 cross-check：A 组页面不得出现 `contact@asterlyn.com`，B 组页面不得出现 `business@asterlyn.com`
6. **每次同步后必须更新 A 组的 `LAST_UPDATED`** 为飞书 `{{DATE}}` 当前值；B 组的 `EFFECTIVE_DATE` 在飞书 §1 代扣协议正文里描述（"Effective immediately upon..."），跟着正文同步即可。
7. **每个页面文件头中文注释里的更新规则不要删**，新增源文档结构变化时同步追加说明。

## 变量字典 ↔ Vue 常量映射

### A 组（terms / privacy / subscription）— 飞书 W1bNwUlTIi… 文档「变量字典」

| 飞书占位符 | Vue 常量 | 当前值 | 用在哪 |
|------------|----------|--------|--------|
| `{{COMPANY_NAME}}` | `COMPANY_NAME` | `LUNARETH LIMITED` | A 组三页面正文 |
| `{{ADDRESS}}` | `ADDRESS` | `Room B53, 2/F, Phase 1, Kwai Shing Industrial Building, 36-40 Tai Lin Pai Road, Kwai Chung, New Territories, Hong Kong` | A 组三页面 Contact Information |
| `{{EMAIL}}` | `CONTACT_EMAIL` | `business@asterlyn.com` | 同上 |
| `{{DATE}}` | `LAST_UPDATED` | `April 3, 2026` | A 组三页面顶部 "Last updated" |
| `{{RETENTION_PERIOD}}` | `RETENTION_PERIOD` | `30 days` | privacy §4 Data Retention |
| `{{JURISDICTION}}` | `JURISDICTION` | `Hong Kong SAR` | terms §18 Governing Law |
| —（产品品牌） | `BRAND_NAME` | `PixPop` | 仅 SEO，不进正文 |

### B 组（payment-terms / merchant-agreement-termination / email-confirmation / billing-statement）— 飞书 OjBIwoby… Payermax 文档

> Payermax 文档目前**没有独立的「变量字典」表格**，但正文里出现的可变值仍按以下常量映射维护：

| 来源 | Vue 常量 | 当前值 | 用在哪 |
|------|----------|--------|--------|
| 正文 "Service Provider" | `COMPANY_NAME` | `LUNARETH LIMITED` | 4 个 Payermax 页正文（共享 A 组同名常量） |
| 正文 "Company Address" | `ADDRESS` | `Room B53, 2/F, Phase 1, Kwai Shing Industrial Building, 36-40 Tai Lin Pai Road, Kwai Chung, New Territories, Hong Kong` | `payment-terms` meta、`merchant-agreement-termination` 客服区 |
| 正文 "Customer Support Email" | `SUPPORT_EMAIL` | `contact@asterlyn.com` | 4 个 Payermax 页（**与 A 组的 CONTACT_EMAIL 严格区分**） |
| 正文 "Effective Date" | `EFFECTIVE_DATE` | `Effective immediately upon the User selecting "I Agree" and successfully completing the initial payment.` | `payment-terms` meta |
| 邮件样例 From 字段 | `SENDER_EMAIL` | `noreply@asterlyn.com` | `email-confirmation` 邮件信封 From |
| 未登录态 demo 邮箱（个人） | 字面量 `'user@example.com'` | — | `email-confirmation` 未登录 fallback |
| 未登录态 demo 邮箱（账单） | 字面量 `'your@mail.com'` | — | `billing-statement` example 模式 fallback |
| 未登录态 demo 订单号 | 字面量 `'ORD-202501120001'` | — | `billing-statement` example 模式 fallback |

### B 组「Example 视觉规范」（v3 per-card example）

B 组 4 页里，**只要某张卡片正在展示 demo / fallback 数据**就必须给这张卡片本身打 EXAMPLE 标记（避免 Payermax / 用户误以为是真实数据）。**v3 起取消"全局 example mode"** —— 每张可挂 example 的卡片各自独立判定。

#### 触发条件（即「example」何时挂在某张卡片上）

| 页面 | example 触发条件 |
|------|------------------|
| `payment-terms` | **永不触发**（纯条款长文，无数据展示，不上 EXAMPLE） |
| `merchant-agreement-termination` | 整页：`!isLoggedIn`（顶部 amber 提示条 + 主区虚线 dashed border） |
| `email-confirmation` | 整页：`!isLoggedIn`（顶部 amber 提示条 + 主区虚线 dashed border） |
| `billing-statement` | **per-card** 模式：`isExampleMode = realInvoiceRows.length === 0 && realCreditRows.length === 0`，4 张卡片**同步**挂 / 不挂 example。Customer Email 卡片**永不**挂 example。 |

#### `merchant-agreement-termination` / `email-confirmation` 视觉规范（沿用 v2.2 二件套）

1. **顶部提示条** `.<page>__example_notice`：
   - amber 背景（`rgba(255, 193, 7, 0.08~0.12)`）+ amber 边框
   - 左侧 `.<page>__example_badge`（实心 amber 矩形 `EXAMPLE` 文本）
   - 右侧文案：未登录态出 `Sign in` 按钮触发 `userStore.showLogin(LoginSource.Unknown)`
2. **主区虚线描边** `.<page>__main.is-example`：
   - 给所有内容卡片 `border-style: dashed; border-color: rgba(255, 193, 7, 0.4);`

#### `billing-statement` 视觉规范（v3 per-card）

**取消项**（v3 起不再使用，SKILL 旧版有提及，新版本搞错就是 bug）：

- ❌ 顶部 `.billing-statement__example_notice` 提示条
- ❌ Customer Email 卡片右上角 `.billing-statement__example_stamp` 印章
- ❌ `<main>.is-example` 全局描边模式 + `isClientMounted` 闸门（v3 因为没有全局 binding，自然水合安全，不再需要）

**新规范**：4 张可能挂 example 的卡片各自独立挂 `is-example`：

| 卡片 | 选择器 | 示例触发 |
|------|--------|---------|
| Order Summary | `.billing-statement__section` 第 1 个 | `isExampleMode === true` |
| Payment Method | `.billing-statement__section` 第 2 个 | `isExampleMode === true` |
| Subscription History | `.billing-statement__section` 第 3 个 | `isExampleMode === true` |
| Credits Purchase History | `.billing-statement__section` 第 4 个 | `isExampleMode === true` |

每张挂 example 的卡片：

1. `class="billing-statement__section is-example"` → amber 虚线 dashed 描边
2. 内嵌右上角 `<span class="billing-statement__card_stamp">EXAMPLE</span>` → 旋转 -6deg amber 矩形印章

Subscription History / Credits Purchase History 两张卡片**还有第三态**：当 `isExampleMode === false` 但**自身**为空（即另一类有真数据这张没有），渲染 `<p class="billing-statement__empty_inline">No subscription transactions yet.</p>` / `No credit purchases yet.`，**不挂** example。

#### 数据优先级规则（v3 仅 `billing-statement`）

```
顶部 currentInvoice (Order Summary + Payment Method)：
  realInvoiceRows.length > 0   →  realInvoiceRows[0]    (subscription)
  realCreditRows.length > 0    →  realCreditRows[0]     (credit)
  都为空                        →  DEMO_INVOICE          (demo + example)

Subscription History 卡片：
  realInvoiceRows.length > 0   →  realInvoiceRows.slice(0, 5)   (真数据，无 example)
  realCreditRows.length > 0     →  []  + empty_inline 文案       (空状态，无 example)
  都为空                        →  DEMO_HISTORY                  (demo + example)

Credits Purchase History 卡片：
  realCreditRows.length > 0    →  realCreditRows.slice(0, 5)    (真数据，无 example)
  realInvoiceRows.length > 0    →  []  + empty_inline 文案       (空状态，无 example)
  都为空                        →  DEMO_CREDIT_HISTORY           (demo + example)
```

`buildInvoiceRow` 同时识别订阅 (`transactionType !== 'PURCHASE'`) 和积分 (`transactionType === 'PURCHASE'`) 订单：

- 订阅：`serviceLabel = '{COMPANY_NAME} {Yearly|Monthly|...} Premium'`、`paymentTypeLabel = 'Auto Debit (Airwallex)'`
- 积分：`serviceLabel = 'Credits Purchase ({pointAmount} Credits)'`、`paymentTypeLabel = 'Credit Card (Airwallex)'`、`creditsLabel = '{pointAmount} Credits'`（用于积分卡片表格 Credits 列）

#### SSR 安全

- 所有「按 example 切换」的内容（提示条 / 印章 / 主卡片）**都必须包在 `<ClientOnly>` 里**，避免 hydration mismatch（SSR 拿不到 user store 状态）。
- `<ClientOnly fallback>` 必须给一个占位（即便是 `<span aria-hidden="true" />`），不然 fallback slot 不展开会掉一个 `<!---->` 注释占位，影响首屏布局。
- **v3 关键改动**：`billing-statement` 顶层 `<main>` **不挂任何 :class binding**，4 张 section 的 `:class="{ 'is-example': isExampleMode }"` 都在 `<ClientOnly>` 默认 slot 内，SSR 不评估它们 → **天然水合安全，不再需要 `isClientMounted` 闸门**。
- **历史坑（v2.3，已通过 v3 重构规避）**：旧版本 `<main :class="{ 'is-example': isExampleMode }">` 会被 SSR 评估，但 `app.vue` 的 `onBeforeMount` 会在 `BillingStatement` 子组件 setup 之前同步把 `subscriptionOrderStore.loading = true`，导致 SSR 阶段 `isExampleMode === true`、客户端首屏 `=== false`，触发：

  ```text
  [Vue warn]: Hydration class mismatch
    - rendered on server: class="is-example billing-statement__main"
    - expected on client: class="billing-statement__main"
  ```

  v3 起改成 per-card example，所有 binding 都进 `<ClientOnly>`，hydration 安全是直接保证的。
- **SCSS 规则也要清干净**：v3 起没有 `.<page>__main.is-example`、`.__example_notice`、`.__example_badge`、`.__example_stamp`、`.__example_signin`、`.__example_link` 这些类（`billing-statement` 一律删除）；保留 `.billing-statement__card_stamp`（per-card 印章）和 `.billing-statement__empty_inline`（空状态文案）。

#### 验收要点

- 未登录态 7 页 / 已登录有数据态 7 页 / 已登录无数据态（仅 `billing-statement` 有意义）三套快照都要跑过。
- `payment-terms` 任何状态下都**不应**出现 `EXAMPLE` 文本 / `__example_notice` 节点 / `__card_stamp` 节点（否则就是误标了）。
- A 组 3 页同上，永远不应有 example 标记。
- `billing-statement` v3 起：
  - example 态：`document.querySelectorAll('.billing-statement__card_stamp').length === 4`（4 张卡片各 1 个）
  - 真数据态（订阅或积分有任意一个）：`__card_stamp` 数量 === 0
  - 任何态都**不应**出现 `.billing-statement__example_notice`、`.billing-statement__example_stamp`、`<main>.is-example`（v3 删除）

### B 组「已登录数据来源」（v3 含积分订单）

| 用途 | 文件 / 符号 |
|------|-------------|
| 当前用户邮箱 / VIP 状态 / `showLogin` | `app/stores/useUserStore.ts` — `currentAccountInfo` / `isLoggedIn` / `isVip` / `showLogin()` |
| 业务用户 ID（merchant-agreement-termination userId 字段） | `app/composables/useBusinessUserId.ts` — `currentBusinessUserId` |
| 订阅历史列表 / 取消订阅流程 | `app/stores/useSubscriptionOrderStore.ts` — `loadSubscriptionHistory()` / `subscriptionOrders` / `showCancelSubscriptionConfirm(record)` / `confirmCancelSubscription()` |
| 积分购买历史（v3 新增，仅 `/billing-statement` 用） | `app/stores/useCreditOrderStore.ts` — `loadCreditHistory()` / `creditOrders` |
| 取消订阅二次确认弹窗（已在 app.vue 全局挂载） | `app/components/dialog/DialogConfirmCancelSubscription.vue` + `app/components/ConfirmCancelSubscriptionView.vue` |
| 渠道 / 金额 / 会员类型 / 时间格式化 | `useMembership(skuType)` / `usePayChannel(code)` / `useFenMoney(num)` / `formatDate(ts)`（du-utils） |

> A 组检查变量值是否变更：飞书「变量字典」表格 cell 编号会随文档结构调整漂移（历史曾从 cell-15~35 → cell-23~49）。**不要靠 cell 编号定位**，按表头中文名（"类目 / 变量 / 替换值 / 上线替换"）和单元格相对位置匹配。

### 「上线替换」列优先级（A 组）

A 组飞书变量字典从某次起新增了「上线替换」列（出现在「替换值」右侧）。该列存在的目的是：上线版本的值与初稿/法律审核临时值不一致，需要在上线时切换。

- **该列有值** → Vue 常量取该列值（覆盖「替换值」列）
- **该列为空** → 沿用「替换值」列

例：2026-04-28 飞书表头 = `类目 / 变量 / 替换值 / 上线替换`，其中 `COMPANY_NAME` / `EMAIL` / `ADDRESS` 三行的「上线替换」列分别填 `LUNARETH LIMITED` / `business@asterlyn.com` / `Room B53, 2/F, Phase 1, Kwai Shing Industrial Building, 36-40 Tai Lin Pai Road, Kwai Chung, New Territories, Hong Kong`，其余行该列为空，故只有这三个常量需要切换。

## 同步工作流

```
- [ ] 1. 用 Playwright 抓取飞书源（A 组 / B 组分别抓，见下方脚本）
- [ ] 2. A 组：对比变量字典「上线替换 ?? 替换值」是否变更，更新对应常量
       B 组：对比正文里 LUNARETH / 地址 / contact@ / EFFECTIVE_DATE 是否变更，更新对应常量
- [ ] 3. 对比正文章节数和正文，整体覆盖 <section>
- [ ] 4. 同步更新 LAST_UPDATED（A 组）+ 文件头中文注释里的章节数说明
- [ ] 5. ESLint + Playwright 实访验收（见下方验收清单，**7 个页面循环**）
```

### Step 1：Playwright 抓取

通过 MCP `user-playwright` server 调用：

1. `browser_navigate` 到对应飞书 URL
2. 等 2-3 秒页面渲染完
3. `browser_evaluate` 跑下面这段脚本拉全文（飞书是虚拟滚动，必须两个方向都滚一遍才能拿全 block）：

```js
async () => {
  const sleep = ms => new Promise(r => setTimeout(r, ms));
  const container = document.querySelector('.bear-web-x-container');
  const blockMap = new Map();
  const H = container.scrollHeight;
  const ys = [
    ...Array.from({length: Math.ceil(H/150)+2}, (_,i)=>i*150),
    ...Array.from({length: Math.ceil(H/150)+2}, (_,i)=>H - i*150)
  ];
  for (const y of ys) {
    container.scrollTo({ top: Math.max(0, y), behavior: 'instant' });
    await sleep(120);
    container.querySelectorAll('[data-block-id]').forEach(b => {
      const id = b.getAttribute('data-block-id');
      const text = (b.innerText || '').replace(/\u200b/g, '').trim();
      if (!text) return;
      const cell = b.closest('.docx-table_cell-block');
      const cls = cell ? Array.from(cell.classList).find(c => c.startsWith('cell-')) : null;
      const rect = b.getBoundingClientRect();
      const top = rect.top + container.scrollTop - container.getBoundingClientRect().top;
      const prev = blockMap.get(id);
      if (!prev || prev.text.length < text.length) blockMap.set(id, { text, cellCls: cls, top });
    });
  }
  const byCell = {};
  for (const [, info] of blockMap) {
    const k = info.cellCls || '__nocell__';
    (byCell[k] ||= []).push(info);
  }
  for (const k of Object.keys(byCell)) byCell[k].sort((a,b) => a.top - b.top);
  window.__scrapeData = byCell;
  return Object.keys(byCell).sort((a,b) => {
    const na=parseInt((a.match(/cell-(\d+)/)||[])[1]||'0');
    const nb=parseInt((b.match(/cell-(\d+)/)||[])[1]||'0');
    return na-nb;
  }).map(k => ({ cell:k, blocks:byCell[k].length, len:byCell[k].reduce((s,b)=>s+b.text.length,0), first:byCell[k][0]?.text.slice(0,80) }));
}
```

拿到 overview 后，再单 cell 抓正文（cell 名按内容定位，不要硬记编号）。

### Step 2：常量值变更检查

不要硬记 cell 编号——飞书每次结构调整 cell 编号都会漂移。正确做法：

**A 组**：

1. 在抓取 overview 中按内容定位「变量字典」起始 cell（包含「类目」「变量」「替换值」表头）
2. 遍历变量名 cell（包含 `{{COMPANY_NAME}}` / `{{ADDRESS}}` 等）
3. 同一行向右取「替换值」cell 与（如存在）「上线替换」cell
4. 若「上线替换」非空则采用该值，否则采用「替换值」
5. 与本 SKILL 表格中的「当前值」对比，差异即需要更新的常量

**B 组**（Payermax 文档没有变量字典表）：

1. 用正则在抓取结果里搜 `Service Provider` / `Company Address` / `Customer Support Email` / `Effective Date`，分别拿后面紧邻一行的真实值
2. 与本 SKILL 表格 B 组「当前值」对比

### Step 3：正文覆盖

通用规则：

- 章节小标题用 `<h2 class="article-deep__subtitle">N. Title</h2>`
- 子标题（payment-terms 的 Authorization / Service Coverage 等）用 `<h3 class="article-deep__subhead">Title</h3>`
- 普通段落用 `<p class="article-deep__txt">...</p>`
- A 组顶部「Last updated」用 `<p class="article-deep__tips">Last updated: {{ LAST_UPDATED }}</p>`
- bullet 列表用 `<ul class="article-deep__list"><li>...</li></ul>`，圆点样式由 `article.scss` 的 `.deep-article ul` 自动处理，**不要**额外内联 `style`
- A 组 Contact Information 章节里的 Email / Address 用 `{{ CONTACT_EMAIL }}` / `{{ ADDRESS }}` 插值
- B 组 Contact Us / Need Help 章节里的 Email 用 `{{ SUPPORT_EMAIL }}` 插值（**不要写 CONTACT_EMAIL**）

A 组模板骨架（带 FooterSection）：

```vue
<template>
  <div class="page-xxx">
    <div class="do-wrapper">
      <div class="deep-article">
        <h1 class="article-deep__title">XXX</h1>
        <section>
          <p class="article-deep__tips">Last updated: {{ LAST_UPDATED }}</p>
          <section class="article-deep__para">
            <p class="article-deep__txt">序言段落... {{ COMPANY_NAME }} ...</p>
          </section>
          <!-- 更多章节 -->
        </section>
      </div>
    </div>
    <FooterSection />
  </div>
</template>
```

B 组模板骨架（**不挂 FooterSection**）：

```vue
<template>
  <div class="page-xxx">
    <div class="do-wrapper">
      <div class="deep-article">
        <h1 class="article-deep__title">XXX</h1>
        <section>...</section>
      </div>
    </div>
  </div>
</template>
```

### Step 4：文件头中文注释

每个 vue 文件头部的中文 `<!-- ... -->` 注释里包含：

- 文档维护地址
- 定位（A 组 / B 组）说明
- 更新规则（i18n 禁用 / 占位符常量映射 / BRAND_NAME vs COMPANY_NAME / **CONTACT_EMAIL vs SUPPORT_EMAIL 双邮箱约定** / LAST_UPDATED 同步）
- 当前飞书源章节数（**变化时更新这里的描述**，不要删）

### Step 5：验收

```bash
npx eslint \
  app/pages/terms-of-service.vue \
  app/pages/privacy-policy.vue \
  app/pages/subscription-agreement.vue \
  app/pages/payment-terms.vue \
  app/pages/merchant-agreement-termination.vue \
  app/pages/email-confirmation.vue \
  app/pages/billing-statement.vue
```

Playwright 实访 7 个 URL（dev server 默认 `http://localhost:3000`）。
B 组 4 页要求**双态实访**：未登录（先 `localStorage.clear()` + reload）+ 已登录（手动登录后）。

通用断言脚本（A / B 共用，B 组双态都跑一遍）：

```js
() => {
  const subtitles = Array.from(document.querySelectorAll('.article-deep__subtitle')).map(el => el.textContent.trim());
  const txt = document.body.innerText;
  const path = location.pathname;
  const isGroupB = ['/payment-terms','/merchant-agreement-termination','/email-confirmation','/billing-statement'].includes(path);
  return {
    title: document.title,
    path,
    subtitleCount: subtitles.length,
    hasCompany: txt.includes('LUNARETH LIMITED'),                     // 7 页都必须 true
    hasAddress: txt.includes('Kwai Shing Industrial Building'),       // A 组三页 + payment-terms + merchant-agreement-termination：true；email-confirmation / billing-statement：false
    // 双邮箱 cross-check：
    hasContactEmail: txt.includes('business@asterlyn.com'),           // A 组：true；B 组：必须 false
    hasSupportEmail: txt.includes('contact@asterlyn.com'),            // A 组：必须 false；B 组：true
    expectedContactEmail: !isGroupB,
    expectedSupportEmail: isGroupB,
    // 旧值黑名单：
    hasOldCompany: txt.includes('Noctera Limited'),                   // 必须 false
    hasOldEmail: txt.includes('yuyouwei@do-global.com'),              // 必须 false
    hasOldAddress: txt.includes('Lippo Centre'),                      // 必须 false
    // 占位符黑名单：
    hasUnreplaced: /\{\{/.test(txt)                                   // 必须 false
  };
}
```

通用期望：

- 所有 7 页 `hasUnreplaced === false`、`hasOldXxx === false`、`hasCompany === true`
- A 组：`hasContactEmail === true`、`hasSupportEmail === false`
- B 组：`hasContactEmail === false`、`hasSupportEmail === true`
- payment-terms：6 ARTICLE + meta；A 组章节数随飞书源同步

B 组「双态」额外断言（在通用脚本基础上再跑这段）：

```js
() => {
  const txt = document.body.innerText;
  const path = location.pathname;
  return {
    path,
    // 老式 example 视觉（v2.2 旧三件套，仅 /merchant-agreement-termination + /email-confirmation 沿用）
    hasExampleNotice: !!document.querySelector('[class$="__example_notice"]'),
    hasExampleBadge: !!document.querySelector('[class$="__example_badge"]'),
    hasIsExampleMain: !!document.querySelector('main.is-example, [class$="__main"].is-example'),
    // v3 per-card example（仅 /billing-statement）
    cardStampCount: document.querySelectorAll('.billing-statement__card_stamp').length,
    cardStampTexts: Array.from(document.querySelectorAll('.billing-statement__card_stamp')).map(s => s.textContent.trim()),
    isExampleSectionCount: document.querySelectorAll('.billing-statement__section.is-example').length,
    // v3 起 /billing-statement **不应**再出现 __example_notice / 旧 __example_stamp / <main>.is-example
    hasLegacyBillingNotice: !!document.querySelector('.billing-statement__example_notice'),
    hasLegacyBillingStamp: !!document.querySelector('.billing-statement__example_stamp'),
    hasLegacyBillingMainExample: !!document.querySelector('main.billing-statement__main.is-example'),
    // v3 /billing-statement 表格头：4 张卡片
    hasOrderSummary: txt.includes('Order Summary'),
    hasPaymentMethod: txt.includes('Payment Method'),
    hasSubscriptionHistory: txt.includes('Subscription History'),
    hasCreditsPurchaseHistory: txt.includes('Credits Purchase History'),
    // v3 空状态文案（仅一类有真数据另一类为空时出现）
    hasNoSubscriptionEmpty: txt.includes('No subscription transactions yet'),
    hasNoCreditEmpty: txt.includes('No credit purchases yet'),
    // /merchant-agreement-termination：未登录 input 应为空、已登录 input 应预填且 readonly
    hasUnsubscribeBtn: !!document.querySelector('button[type="submit"]') &&
      /Unsubscribe/i.test(document.body.innerText),
    inputCount: document.querySelectorAll('input[type="text"]').length, // 仅 /merchant-agreement-termination：3
    inputValues: Array.from(document.querySelectorAll('input[type="text"]')).map(i => i.value),
    inputReadonly: Array.from(document.querySelectorAll('input[type="text"]')).map(i => i.readOnly),
    // /email-confirmation：未登录显示 user@example.com；已登录显示真用户 email
    hasDemoUserEmail: txt.includes('user@example.com'),
    // /billing-statement：example 态（订阅+积分都空）展示 your@mail.com + ORD-202501120001
    hasDemoCustomerEmail: txt.includes('your@mail.com'),
    hasDemoOrderId: txt.includes('ORD-202501120001')
  };
}
```

B 组四态期望（**v3 增加"已登录有订阅有积分" / "已登录有订阅无积分" / "已登录无订阅有积分" / "已登录都为空"**）：

| 页面 | 未登录态 | 已登录有订阅 + 有积分 | 已登录仅订阅 / 仅积分 | 已登录都为空 |
|------|---------|---------------------|---------------------|--------------|
| `/payment-terms` | `hasExampleNotice===false`、`cardStampCount===0` | 同左 | 同左 | 同左 |
| `/merchant-agreement-termination` | `hasExampleNotice===true`；`inputCount===3`、`inputReadonly===[false,false,false]`、`inputValues===['','','']`；`hasUnsubscribeBtn===true`（disabled） | `hasExampleNotice===false`；`inputReadonly===[true,true,true]`、`inputValues` 三项均非空；点击 Unsubscribe 弹 `DialogConfirmCancelSubscription` | 同已登录列 | 退化为"无订阅可退"态（按当前实现：`merchant-agreement-termination` 不在登录空态切 example，按真实订阅状态展示） |
| `/email-confirmation` | `hasExampleNotice===true`、`hasDemoUserEmail===true`；badge 文案 `Premium Member` | `hasExampleNotice===false`、`hasDemoUserEmail===false`；展示真用户 email；isVip 时 badge `Premium Member`，否则 `Free Account` | 同已登录列 | 同已登录列 |
| `/billing-statement` | `cardStampCount===4`、`isExampleSectionCount===4`、`hasDemoCustomerEmail===true`、`hasDemoOrderId===true`、`hasLegacyBillingNotice===false`、`hasLegacyBillingStamp===false`、`hasLegacyBillingMainExample===false` | `cardStampCount===0`、`isExampleSectionCount===0`、真订单号 / 真邮箱、`hasNoSubscriptionEmpty===false`、`hasNoCreditEmpty===false` | `cardStampCount===0`、`isExampleSectionCount===0`、空那一类卡片显示 `hasNoSubscriptionEmpty===true` 或 `hasNoCreditEmpty===true` 之一 | `cardStampCount===4`、`isExampleSectionCount===4`、`hasDemoCustomerEmail===true`（真用户 email 为空时降级 demo）、`hasDemoOrderId===true` |
| A 组三页（回归） | `hasExampleNotice===false`、`cardStampCount===0` | 同左 | 同左 | 同左 |

## 已知坑

1. **飞书源会直接展示替换值**：飞书源会把 `{{COMPANY_NAME}}` 直接渲染成真实值，**抓取结果里看不到 `{{}}` 占位符不代表本地不需要常量**——本地仍要走常量，方便变更时一处改全局。
2. **「替换值」与「上线替换」可能并存**（A 组）：飞书正文里使用的多半还是「替换值」列的旧值（如 `Noctera Limited`），但变量字典「上线替换」列指出实际上线值（如 `LUNARETH LIMITED`）。前端**始终以「上线替换」优先**，与正文中的旧值不一致是预期的。
3. **cell 编号会漂移**：每次飞书结构调整 cell 编号都会变（历史出现过 cell-15→cell-23、cell-44→cell-59、cell-60→cell-491 等大跨度跳跃）。脚本必须按内容定位，不要硬编码 cell id。
4. **订阅协议章节数会变化**（A 组）：历史上从 3 章节扩到 9 章节，每次同步必看实际章节数，不要假设跟历史一致。
5. **i18n 残留**：如果在 PR 中误引入了 `useI18n()` / `i18n/locales/en.js` 的 SEO key，必须删除。所有 7 个页面均不接入 i18n。
6. **Footer 行为**：
   - A 组（`/terms-of-service`、`/privacy-policy`、`/subscription-agreement`）页面**自身渲染** `<FooterSection />`，但**不要把它们当作 nav 入口加进 FooterSection 组件内部的菜单**。
   - B 组（`/payment-terms`、`/merchant-agreement-termination`、`/email-confirmation`、`/billing-statement`）页面**不渲染** `<FooterSection />`（Payermax 审核专用，普通用户无入口）。
7. **bullet 列表样式**：圆点由 `.deep-article ul li::before` 全局提供，**不要在 li 文本中重复写 `•` 字符**，否则会出现「圆点 + •」双 bullet。
8. **双邮箱不要串**（关键！）：
   - `business@asterlyn.com`（CONTACT_EMAIL）只能出现在 A 组
   - `contact@asterlyn.com`（SUPPORT_EMAIL）只能出现在 B 组
   - PR review 时必须人肉看一眼模板里 email 出现的位置和插值名是否对得上；验收脚本里也有 cross-check
9. **B 组未登录 / 空数据 mock 值**：email-confirmation 里的 `user@example.com` 仅在未登录态展示；billing-statement 里的 `your@mail.com` / `ORD-202501120001` / `US$29.90` / `CREDIT-202501120002` / `US$9.99` 等在「未登录」**或**「已登录但订阅+积分历史都为空」时展示。同步飞书时如果设计稿改了字段名 / 顺序需要跟着改，但 mock 值本身可以保持稳定。
10. **merchant-agreement-termination 不仅是合规页，也是真实功能页**：已登录用户点击 Unsubscribe 会真的进入 `useSubscriptionOrderStore.confirmCancelSubscription` → `requestUnsubscribeAgreement(agreementId)` 流程，**会真退订**。改这页时务必跑双态实访，不能只走未登录态肉眼看。
11. **未登录态退订没有匿名后端接口**：cubetv 三因素自助退订（邮箱+userID+订单号反查）我们仓库没实现，未登录态点击 Unsubscribe **统一走 `userStore.showLogin()` 兜底**。如审核方坚持需要"不依赖登录的退订能力"需另开后端需求。
12. **B 组 SSR hydration**：登录态依赖 client-side `pix_bid` cookie + Pinia store，必须把"按登录态切换"的内容包在 `<ClientOnly>` 里（含 fallback slot 给 SSR / 首屏未注水期），否则会出现 hydration mismatch（"server rendered text didn't match client"）。
13. **`subscriptionOrderStore.loadSubscriptionHistory()` 已被 `app.vue` 全局触发**：登录用户进入站点时已自动加载，B 组页面的 `onMounted` 仍要做防御性调用——例如审核员在私密浏览模式直接点 `/billing-statement` 链接、`subscriptionOrders` 为空时不要假设数据已就绪。
14. **`DialogConfirmCancelSubscription` 已在 `app.vue` 全局挂载**（`LazyDialogConfirmCancelSubscription`），merchant-agreement-termination 不需要自己挂这个组件，只要调 `subscriptionOrderStore.showCancelSubscriptionConfirm(record)` 即可触发。
