# fe-picpopop-test-accounts.mdc 设计说明

## 设计意图

为 fe-picpopop 项目提供一个稳定的、跨环境（沙盒 / 线上 / 本地 dev）共用的测试账号凭据来源，避免多个 agent 会话 / 多个开发者各自维护自己的"个人测试账号"，导致 PM 验收脚本、Playwright 自动化、烟雾测试每次都要重新约定凭据。

## 适用场景

- 在 fe-picpopop 项目中跑 Playwright 自动化登录流程
- 对受保护页面（`/ai/creation` / `/tools/video-templates` / `/account` 等）做烟雾测试
- 沙盒 / 线上回归验证「登录后页面渲染正常」
- 本地 dev server 启动后做 E2E 验证

## 召回方式

按需触发（`alwaysApply: false`），description 包含「e2e / Playwright / login / fe-picpopop」等关键词，agent 在准备执行登录流程时通过 description 自动召回。

## 关键约束

1. **三环境账号同密码**：禁止为不同环境维护多套凭据，PM 侧需保证沙盒 / 线上 / 本地都能用同一对 email + password 登录
2. **账号不可用时不自动注册**：服务端报"未注册" / "密码错误"时立即停下来告诉用户，由 PM 处理，不绕开
3. **凭据见 .mdc**：本说明文件**不**重复写明文密码，凭据所在地仅一个，方便统一替换
4. **配套规则**：调用方应同时遵守 `playwright-chrome.mdc`（9222 端口、复用 tab）+ `playwright-artifacts.mdc`（截图归档）+ `dev-server-host.mdc`（本地用局域网 IP）

## 安全考量

凭据明文存在于项目 `.cursor/rules/` 中，受 git 跟踪。设计时已权衡过：

- 接受现状：该账号仅供测试用途，不持有任何敏感数据 / 充值能力，团队内仓库可见的密码符合"团队共享凭据"语义
- 备选：若未来引入外部协作者或仓库要开源，需迁移到 `.cursor/rules/local/` + `.gitignore`，并把脱敏版本签入 git

## 配套自检

`fe-picpopop-test-accounts.mdc` 末尾的 `rg` 命令用于检测密码是否被错误泄露到代码 / 日志 / 其他文档中。CR 时必跑。

## 历史变更

- 2026-05-15：首次创建。沉淀缘由：在做 GemsHistoryRecord DTO 重构后需要 Playwright 实测登录态页面，临时编入对话的凭据每次会话都要重述，沉淀到 rule 后可被未来所有 fe-picpopop 烟雾测试自动召回。
