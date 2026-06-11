# Skill: UI/UX Pro Max

> 设计智能 Skill——为 AI Agent 提供 67 种 UI 风格、161 套色板、57 种字体搭配、99 条 UX 指南的可搜索数据库，帮助构建专业级 UI。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `ui-ux-pro-max` |
| 来源仓库 | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) |
| 官方网站 | https://ui-ux-pro-max-skill.com |
| 当前版本 | v2.0 |
| 许可证 | **CC-BY-NC-4.0（非商业授权）** |
| Stars | 88K+ |

> ⚠️ **许可证注意**：CC-BY-NC-4.0 禁止商业使用。个人学习和非商业项目可以使用，商业项目请联系作者或使用其他方案。

## 安装

### 方法一：官方 CLI（推荐）

```bash
# 安装 CLI 工具
npm install -g uipro-cli

# 为 Cursor 安装
uipro init --ai cursor

# 为 Claude Code 安装
uipro init --ai claude

# 同时安装到所有支持的工具
uipro init --ai all

# 离线模式（无网络时使用 CLI 内置资产）
uipro init --offline
```

### 方法二：Claude Code 插件市场

```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

### 安装位置

| AI 工具 | 安装路径 |
|---------|---------|
| Claude Code | `.claude/skills/ui-ux-pro-max/` |
| Cursor | `.cursor/commands/ui-ux-pro-max.md` + `.shared/ui-ux-pro-max/` |
| Windsurf | `.windsurf/workflows/ui-ux-pro-max.md` + `.shared/ui-ux-pro-max/` |

## 数据库规模（v2.0）

| 类别 | 数量 |
|------|------|
| UI 风格 | 67 种（玻璃态、新态设计、极简、粗野主义、Bento Grid、暗色模式等） |
| 色板 | 161 套（按产品类型分类：SaaS、电商、医疗、金融科技等） |
| 字体搭配 | 57 组 |
| UX 指南 | 99 条（动效、无障碍、z-index、加载状态等最佳实践） |
| 图表类型 | 25 种（含库推荐：Chart.js、Recharts、D3.js） |
| 推理规则 | 161 条（行业特定设计系统生成逻辑，v2.0 新增） |
| 支持技术栈 | 16 种（React、Next.js、Vue、Svelte、Tailwind、SwiftUI 等） |

## 触发条件

安装后，Agent 在以下情况自动激活：
- 请求构建、设计、创建、实现、审查、修复、改善任何 UI/UX 任务

也可以通过斜杠命令调用（适用于 Kiro、GitHub Copilot、Roo Code 等）：

```
/ui-ux-pro-max 为 SaaS 产品构建定价页面
```

## v2.0 核心新特性：设计系统生成器

v2.0 的旗舰功能是**设计系统生成器**——基于项目需求自动生成完整的、定制化的设计系统：

1. 用户描述需求（产品类型、目标用户）
2. Agent 使用 161 条推理规则匹配最适合的风格/色板/字体
3. 生成完整设计系统（色彩令牌、排版比例、间距规则）
4. 交付前检查常见 UI/UX 反模式

## 与仓库现有 Skills 的关系

| Skills | 关系 |
|--------|------|
| [skills/taste-skill](../taste-skill/) | 互补。ui-ux-pro-max 提供设计数据库检索；taste-skill 提供行为约束，防止 AI 生成泛化 UI |
| [skills/impeccable](../impeccable/) | 互补。impeccable 专注 7 大设计领域的专家建议；ui-ux-pro-max 专注可搜索的样式数据库 |
| [skills/design-md](../design-md/) | 互补。DESIGN.md 固化已有设计系统；ui-ux-pro-max 在项目初期帮助发现和选择设计方向 |

## 支持的 AI 工具

Claude Code、Cursor、Windsurf、Antigravity、Codex CLI、Continue、Gemini CLI、OpenCode、Kiro、GitHub Copilot 等。
