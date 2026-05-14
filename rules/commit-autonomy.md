# commit-autonomy.mdc 设计说明

## 设计意图

用户的工作偏好：**改动完成后 agent 自己判断要不要 commit，不要反复问**。
反复问"是否要直接 commit"是一种典型的"AI 礼貌过度"现象——用户已经把 agent 当工程师在用，
工程师不会每改一行就问主管"我可以保存吗？"。错了 `git reset` / `git revert` 能回滚，
但反复问会浪费用户的注意力和会话上下文。

本规则把"直接 commit / 给 message 让用户审"两条路径写清楚边界条件，让 agent 有底气
自主决策，同时给安全护栏（敏感文件、历史重写、破坏契约等场景必须先问）。

## 适用场景

- 自动应用 (`alwaysApply: true`)
- 所有 agent + git 协同的工作流
- Cursor IDE / Cursor CLI / Cursor Cloud Agents 等都生效

## 核心边界

### ✅ 直接 commit（agent 自主）

满足全部：
1. 不含敏感信息（`.env` / credentials / 私钥）
2. lint / tsc / 测试都过
3. commit message 符合项目规范（中文/英文 type 前缀）
4. 单一主题或主动拆 commit 后的单一主题
5. 不涉及历史重写

### ❌ 给 message 让用户审

任一即停手：
- 架构级调整（删核心目录、改 baseUrl、改全局插件加载顺序）
- 破坏现有契约（API 签名 / 协议字段重命名）
- 多个不相关主题混合且拆分边界模糊
- 含大文件 / 二进制 / 截图等可能不该入库的内容
- lint / tsc / 测试有报错或新警告未修
- 涉及 secrets / .env / credentials 类文件
- 用户在同一会话明确说过"先别 commit"

### 模糊地带 → 偏向直接 commit

< 5 文件、单一主题、lint/tsc 过、message 清楚 → 默认直接 commit。

## 输出格式

### 直接 commit 后必做

- 1 句话说明 commit 内容
- 贴 commit 短 hash + message 第一行（用户可 spot check）
- 列出需要用户后续决策的事项（push / 关联 issue / 跟同事 sync）

### 不 commit 时必做

- 列「为什么不直接 commit」（命中哪条 ❌ 边界）
- 给完整 commit message 草稿
- 等用户决策

## 例外

- 用户明确说「先别 commit」/「我先 review」/「拆成 N 个我自己定」→ 立即停手
- 用户明确说「commit message 改一下」→ 等用户给改后版本，不擅自重写

## 与其他规则联动

- 配合各项目内的 commit message 规范（如 `git-commit-zh.mdc` 中文规范、conventional commits 等）
- 与 `dev-scenarios.mdc`：完成"典型场景"操作 → 写 dev-scenarios → commit 包含文档
- 与 `playwright-artifacts.mdc`：截图等调试产物在 `.playwright-mcp/logs/`（已 gitignore），不会误入 commit
- 与 Cursor Git Safety Protocol：`--amend` / force push / 历史重写仍然需要用户明确同意

## 历史教训

- **2026-05-14**: fe-picpopop 项目同一会话连续 3 次问"是否要直接 commit"，用户最终明确说
  「以后不用问，觉着合适就直接 commit，不合适就给 commit msg」→ 立此规则沉淀为全局工作习惯
