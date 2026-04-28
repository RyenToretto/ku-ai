# playwright-artifacts 规则说明

## 设计意图

Playwright MCP 在调试 / 自检过程中会大量产出截图（`browser_take_screenshot`）、HTML 快照、trace、HAR 等文件。这些资源**只服务于当前 Agent 会话的验证过程**，不属于项目交付物，但默认会被写到 `browser_take_screenshot` 调用时的工作目录——通常就是项目根。

历史教训：在「短剧自动分配 9.21-9.24」实现验证过程中，Agent 直接把 22 张验证截图（`drawer-opened.png` / `drawer-status-failed.png` / `feishu-9-21-section.png` …）丢到项目根，污染了 IDE 文件树，严重时还可能被误 `git add .` 提交进仓库。

此规则强制把所有 Playwright 临时产物收纳到 `.playwright-mcp/logs/<topic>/` 子文件夹，与已经 gitignored 的 `.playwright-mcp/` 目录共生。

## 适用场景

- 用 Playwright MCP 做页面验证、截图复盘、回归检查
- 用 Playwright MCP 抓 HAR / 网络请求 / trace 排查问题
- 用 Stitch / 飞书 等其它 MCP 落地大量调试图片（同样建议走 `.<mcp-name>/logs/<topic>/` 模式）

## 关键约束

1. **路径模板固定**：`.playwright-mcp/logs/<topic-kebab-case>/<seq>-<desc>.png`
2. **每次任务一个 topic 子目录**，便于事后清理
3. **`browser_take_screenshot.filename` 必须显式传完整路径**，不能只传文件名
4. **`.playwright-mcp/` 必须在项目 `.gitignore` 中**，规则强制要求
5. Playwright MCP 自动产出的 `console-*.log` / `page-*.yml` 保持原位，不手动搬动

## 自检

任务完成前必跑（基于 git untracked，避免误伤项目合法资源）：

```bash
git status --porcelain | grep -E '\.(png|jpe?g|webp|gif|har|zip|trace)$' || echo 'CLEAN'
```

判定逻辑：

- `.playwright-mcp/` 已 gitignore，里面的产物**永远不会**出现在 git status，所以输出里的图片/HAR 一定是 Agent 漏洒到业务目录的违规品
- 已被 git 追踪的合法资源（`logo.png` / `cover.png` / README 截图）也不会出现在 untracked 列表
- 输出 `CLEAN` 即合规；否则按命中文件移动到 `.playwright-mcp/logs/<topic>/` 或删除

> ⚠️ **不要再用 `ls *.png` 这类裸通配符做自检** —— 会把项目根下的 `logo.png` / `cover.png` / README 引用的截图等合法资源也命中，导致误判。

## 与既有规则的关系

- **playwright-chrome.mdc**：管 Playwright 怎么连接 Chrome（CDP 复用）
- **playwright-artifacts.mdc（本规则）**：管 Playwright **产物**怎么存放
- 两条规则正交，应同时启用 `alwaysApply: true`
