---
name: feishu-api-sync
description: >-
  Synchronizes all REST API call sites in app/pixpopService/* with the
  canonical Feishu API document
  (https://rqqmslsz9y.feishu.cn/wiki/PFmwwDtHciNJJCki5cEctZRynxh).
  Scrapes the full Feishu doc via Playwright, extracts the `[GET]/[POST]
  /api/v1/...` → `/api/airepair/...` pairs, cross-references every
  `this.fetcher('...')` call in the service layer, and emits a diff plan
  that strictly mirrors the doc — never infers a "new path" that the doc
  did not explicitly list. Use when the user asks to align / re-sync /
  re-check / 对齐 接口路径 with 飞书接口文档, mentions the wiki URL
  `PFmwwDtHciNJJCki5cEctZRynxh`, edits any `app/pixpopService/*Service.ts`
  in a path-rename context, or migrates the fetcher baseUrl.
---

# Feishu API Sync (魔拍 Web 接口文档同步)

唯一真源：[飞书《魔拍 Web 接口文档（新）》](https://rqqmslsz9y.feishu.cn/wiki/PFmwwDtHciNJJCki5cEctZRynxh)

## 适用场景

只要满足下面任一条件即可调用本 skill：

- 用户说「对齐接口」「同步接口」「接口改造」「检查接口和飞书文档是否一致」
- 用户粘贴或提到 wiki URL `PFmwwDtHciNJJCki5cEctZRynxh`

## 核心约束（必须严格遵守）

1. **文档即真源**。代码以飞书文档为准，**不允许"按规律推断新 path"**。
2. **只有旧 path 的接口**（文档章节只列了 `[POST] /api/v1/...`，没有列对应的 `[POST] /api/airepair/...`）保持文档原貌：
   - 代码层把 `/api/v1` 这一段标准前缀去掉，剩余路径段**原样保留**到 `fetcher` 入参里
   - 例：文档「14. 根据用户图片生成视频效果 AI 提示词（已废弃）」只列 `[POST] /api/v1/ai/i2v/ai_prompt` → 代码用 `this.fetcher('/ai/i2v/ai_prompt', …)`，**不要**自行改成 `/i2v/ai_prompt`
   - 同时在该方法的 JSDoc 上加 `@deprecated 文档只列了旧 path，新接口待后端补充` 注释
   - **历史教训 (2026-05-14)**: 「9. 生视频任务进度查询(根据 batchId 数组返回多条)」当时只列旧 path `/api/v1/i2v/query/batch`，按本规则代码保留为 `/i2v/query/batch`；后端补出新 path 是 `/api/airepair/i2v/queryBatch`（`ai/` 段被 drop），与"按规律推断"的猜测不一致——**这正是禁止推断的原因**，必须等后端确认。
3. **有新 path 的接口**：代码完全使用新 path（即文档列出的 `/api/airepair/...` 去掉 `/api/airepair` 这一段标准前缀）。
4. **fetcher baseUrl 是 `/api/airepair`**（见 [app/plugins/01-pixFetch.ts](app/plugins/01-pixFetch.ts)），因此 `fetcher` 的第一个参数 = 完整 URL 去掉 `/api/airepair` 前缀。
5. **POST/GET 方法、请求体、响应体也都要核对**，不仅仅是 path。
6. **id 类字段一律 `string`**：项目内 `id` / `templateId` / `orderId` / `userId` / `agreementId` / `batchId` / `tagId` 等 id 类字段统一以 `string` 表达。文档标 `Long` / `number` 不予理会，**不算不一致**，不要在审计报告里列、不要在代码里改回 `number`。
7. **每个未决项一份独立 markdown，解决后直接删文件**：
   - `docs/api/changes/README.md` 只放约定 + 流程 + 文件清单，**不再汇总修复历史也不汇总未决列表**。
   - `docs/api/changes/_doc-index.md` 是文档章节 × path 索引。
   - 其余每个**待后端/产品确认或待优化**的差异都开一个独立 `.md`，文件名采用 `<service-kebab>-<topic-kebab>.md` 形态（如 `pix-account-google-login-body.md`）。
   - 已经和 doc 对齐 / 本次 PR 已修复 / 仅 doc 截断 / id 类型差异 —— **不写**任何文件。
   - 决议落地后**直接 `rm` 该 .md 文件**，不要改写成"已解决"——目标是让 `docs/api/changes/` 目录最终只剩 `README.md` + `_doc-index.md`。

## 输出物

每次执行本 skill 只产出 / 更新下列文件：

```
docs/api/changes/
├── README.md                                 # 约定 + 流程 + 当前未决项清单（短链接列表）
├── _doc-index.md                             # 文档章节 × 行号 × path 总索引
├── <service>-<topic>.md                      # 每个未决项一个文件，解决即删
├── <service>-<topic>.md
└── ...
```

`.playwright-mcp/logs/api-sync-audit/feishu-doc-latest.txt` —— Playwright 抓取的文档原文，已纳入 `.gitignore`，**不进仓库**。

## 标准流程（每次执行必须走完）

### Step 1 · 抓取最新文档

通过 `user-playwright` MCP 抓取整个飞书文档：

```js
// 1) 找到飞书浏览器 tab（用户通常已经打开）
browser_tabs({ action: 'list' })
// 2) 选中飞书 tab
browser_tabs({ action: 'select', index: <wiki-tab-index> })
// 3) 翻页 + 收集所有 [data-record-id] 区块
browser_evaluate({ function: `
  async () => {
    const scroller = document.querySelector('.bear-web-x-container');
    if (!scroller) return { error: 'no scroller' };
    const seen = new Map();
    const wait = (ms) => new Promise(r => setTimeout(r, ms));
    scroller.scrollTop = 0;
    await wait(400);
    for (let y = 0; y <= scroller.scrollHeight; y += 500) {
      scroller.scrollTop = y;
      await wait(160);
      document.querySelectorAll('[data-record-id]').forEach(b => {
        const id = b.getAttribute('data-record-id');
        const type = b.getAttribute('data-block-type');
        if (!seen.has(id)) seen.set(id, { id, type, text: (b.innerText || '') });
      });
    }
    return Array.from(seen.values())
      .map(b => '=== [' + b.type + '] ' + b.id + ' ===\\n' + b.text)
      .join('\\n\\n');
  }
` })
```

> 为什么必须翻页：飞书 docx 是虚拟滚动渲染，未滚动到的 block 不会出现在 DOM 里。一次性抓取必然漏内容。

把 `browser_evaluate` 的返回值（JSON 字符串）解码后写入 `.playwright-mcp/logs/api-sync-audit/feishu-doc-latest.txt`。使用项目里的辅助脚本：

```bash
python3 .agents/skills/feishu-api-sync/scripts/decode-tool-result.py \
  /Users/<user>/.cursor/projects/<...>/agent-tools/<uuid>.txt \
  .playwright-mcp/logs/api-sync-audit/feishu-doc-latest.txt
```

### Step 2 · 提取文档 path 对照表

运行：

```bash
python3 .agents/skills/feishu-api-sync/scripts/extract-doc-mapping.py
```

脚本会扫描 `.playwright-mcp/logs/api-sync-audit/feishu-doc-latest.txt`，按章节顺序输出一张表：

```
GET    /api/v1/user/me                                   /api/airepair/v1/web/user/me        ✓
POST    /api/v1/i2v/queryBatch                        — (没有新 path)                       ⚠ only-old
…
```

末尾会单独列出「只有旧 path、没有新 path」的接口，提醒不要推断。

### Step 3 · 对照代码

运行：

```bash
python3 .agents/skills/feishu-api-sync/scripts/audit-services.py
```

脚本扫描 `app/pixpopService/*Service.ts` 里所有 `this.fetcher('...')` 调用，对照 Step 2 的对照表，输出：

```
file                  code path                        full path (after baseUrl)            status
PixAccountService.ts  /v1/web/user/me                  /api/airepair/v1/web/user/me         ✓ matches new doc path
PixFeatureService.ts  /i2v/queryBatch                 /api/airepair/i2v/queryBatch        ⚠ DOC ONLY HAS OLD: /api/v1/i2v/queryBatch
…
```

任何 `⚠`/`?` 行都要在 Step 4 里处理。

### Step 4 · 全维度逐接口审计（path 之外）

仅对 path 进行机械对照后还不够，必须逐接口核对**五个维度**：

1. **path**（已由 Step 3 脚本覆盖）
2. **method** —— doc 行下方的「方法：GET/POST」字样
3. **请求 query**（GET）或 **请求 body**（POST）—— 字段名、类型、必填
4. **响应字段** —— 字段名、类型，对照 TS 类型定义
5. **TS 类型一致性** —— `shared/types/*.d.ts` 是否覆盖了 doc 列出的所有字段

#### 推荐做法：每个未决项一个独立 `.md`，解决即删

**严格遵守**：

- 已与 doc 对齐的接口 → **不写**任何文件
- 本次 PR 已修复并对齐的接口 → **不写**任何文件（git 历史就是修复记录，无需在 docs 里再留一份）
- 仅 doc 截断 / doc 字段表为空导致无法对照 → 不必单独建文件，反馈给后端补 doc 即可
- id 类字段差异（`number` ↔ `string`）→ **不写**（按核心约束 6，全部沿用项目内 `string`）

会被记录的**只有两类**：

1. **🔴 待后端 / 产品确认**：字段名重命名、business 硬编码、必填字段缺失等可能影响业务的差异
2. **🟡 待优化**：调用方便、可读性、错误兜底等代码改进建议

每个 `.md` 文件结构：

```markdown
# <Service>.<method> —— <一句话问题摘要>

> 解决后请直接删除此文件。

## 现状
（代码片段 + 文件路径 + doc 摘录/行号）

## 问题
（与 doc 不一致的具体点，或代码本身的隐患）

## 待确认
（需要谁来决策什么）

## 决策后
（分情况说明 code 侧 / doc 侧分别怎么改）
```

文件命名：`<service-kebab>-<topic-kebab>.md`，例如：

- `pix-account-google-login-body.md`
- `pix-account-pay-order-paid-amount.md`
- `pix-article-detail-content-required.md`

**重要**：在 `README.md` 的「当前未决项」清单段加上短链接（一行一句话），保持 README 是单一入口。决议落地后**同时删除文件与 README 中的链接行**。

> 如果接口非常多（一个 service ≥10 个），可以并行 dispatch 多个 explore subagent，每个负责一段调研；但所有未决项产物都要落到独立的 `<service>-<topic>.md` 文件里。

### Step 5 · 应用修复（分级处理）

按风险等级把修复分两批：

**安全修复（直接应用）**：
- 给 TS 类型添加**可选字段**（doc 列了 code 未声明的字段）
- 给方法返回类型纠错（如 `Promise<EmailCheckResponse>` 误用应改为 `Promise<DoResponse>`）
- 删除明显的多余泛型（如 `this.fetcher<HomeData>(...)` 但实际不需要）
- 给 `catch` 分支补返回值兜底

**id 类字段差异：跳过，不算修复，不写入报告**：
- doc 写 `id: Long` / `templateId: 10001` / `tagId: number`，code 写 `string` —— 项目惯例就是 `string`，不动。
- 此类条目**不要**列入「待确认」也**不要**改 code 类型。

**需要后端 / 产品确认（先 document，不直接改）**：
- 字段名拼写不一致（doc 笔误 vs code 正确）
- body 字段名重命名（`token` vs `code`）
- 类型从 string 改 number / boolean 改 int（影响所有调用方，**不含 id 类**）
- 重命名共享类型字段（如 `isDefault` → `default`）
- 业务硬编码（如 `taskStatus = 'SUCCESS'`）改成可配置

把这类待确认项**集中**写到 `docs/api/changes/README.md` 的「⚠️ 需要后端 / 产品确认后再改的事项」段落里，标注严重等级（🔴 高 / 🟡 中 / 🟢 低），方便人工跟进。

### Step 6 · 自检

```bash
# 1) 没有遗漏的旧 path（除了 only-old 接口和 deprecated 注释里的描述性引用）
grep -rnE "/api/v1/(user|auth|template|homepage|article|ai/i2v)" app/pixpopService/ \
  --include='*.ts' \
  | grep -v '@deprecated' \
  | grep -v '// '

# 2) eslint
npx eslint --fix app/pixpopService/ app/composables/usePixpopSetting.ts shared/types/

# 3) 再跑一次对照脚本，确保所有 ⚠/? 都消失（除已加 @deprecated 注释的 only-old 接口）
python3 .agents/skills/feishu-api-sync/scripts/audit-services.py
```

### Step 7 · 同步周边

修改 path 时，下面这些文件常常需要联动：

- [app/composables/usePixpopSetting.ts](app/composables/usePixpopSetting.ts) `passPath` 数组：所有"鉴权 403 失败时不应触发登出兜底"的接口都要写全 `/api/airepair/...` 完整路径。
- [docs/payment/current/airwallex-api.md](docs/payment/current/airwallex-api.md) / [docs/payment/current/airwallex-integration.md](docs/payment/current/airwallex-integration.md) / [app/stores/usePaymentStore.ts](app/stores/usePaymentStore.ts) 注释：里面出现的支付查询/支付流水 path 要随代码同步。
- [server/utils/proxy.config.ts](server/utils/proxy.config.ts) 的 proxy 表前缀（目前是 `/api/airepair`，不要轻动）。

## 受影响文件清单

| 文件 | 作用 |
|------|------|
| `app/plugins/01-pixFetch.ts` | 定义 fetcher baseUrl = `/api/airepair` |
| `app/pixpopService/PixAccountService.ts` | 用户、订阅、积分、支付查询 |
| `app/pixpopService/PixArticleService.ts` | 文章 tag / 列表 / 详情 |
| `app/pixpopService/PixCreationService.ts` | 我的创作列表 / 删除 / 下载 / 收藏 |
| `app/pixpopService/PixFeatureService.ts` | 图生视频任务提交 / 进度查询 / 模型配置 |
| `app/pixpopService/PixMaterialService.ts` | 模板详情 / 首页 tag / explore / 收藏 |
| `app/composables/usePixpopSetting.ts` | `passPath` 鉴权白名单 |
| `shared/types/*.d.ts` | 请求体 / 响应体 TS 类型（promotion / payment / pixpop / aiModel） |
| `server/utils/proxy.config.ts` | 网关代理表 |

## 常见踩坑

1. **「只有旧 path」≠「该接口废弃」**。文档 §9 query/batch 标题写「生视频任务进度查询」并没有「(已废弃)」字样，但 §14 / §15 明确写了「(已废弃)」。**不能因为没有新 path 就当作废弃**。
2. **deprecated 接口仍有调用方**。`requestGeneratePrompt`（被 `ToolsI2aController.vue` 调用）、`requestAllPixWords`（被 `useI2aStore.ts` 调用）即使加了 `@deprecated`，调用方也仍在跑。本 skill 不主动删除调用方，需要时单独跟用户确认。
3. **死代码顺手清理**。审计时若发现 service 里有「全项目零调用且飞书文档未列」的方法（如曾经的 `PixMaterialService.requestExploreModelList`），直接连同其专属类型一起删掉, 不要保留半成品。
4. **不要拼接 base URL**。fetcher 第一个参数永远以 `/` 开头、永远不含 `/api/airepair`、永远不含 `/api/v1`；如果遇到 `/api/v1/...` 字面量出现在 fetcher 入参里，那就是 bug。
5. **不要改 service 的 method 签名顺序**（参数名 / 类型），只改 path 字符串和必要的 method/body 字段。

## 提交规范（按 `.cursor/rules/git-commit-zh.mdc`）

完成同步后提交，建议消息：

```
fix: 接口 path 与最新飞书文档对齐

- 用户 / 积分 / 支付 path 改为 /v1/web/user/me/* 前缀
- 模板 / 首页 tag 系列 path 补齐 /v1/web 前缀
- 仅有旧 path 的接口（query/batch、ai_prompt、config/video/tag、template/search）保持旧 path 段，并标注 @deprecated
- usePixpopSetting passPath 同步到新 path
- 支付相关文档与 store 注释同步
```

## 进度日志

每次执行本 skill，把扫到的差异表、修复前后的 path、自检命令输出归档到 `.playwright-mcp/logs/api-sync-audit/<date>/`，方便事后回溯。
