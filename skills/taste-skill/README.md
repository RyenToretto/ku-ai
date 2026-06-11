# Skill: taste-skill

> Anti-slop 前端设计框架——让 AI Agent 生成有品味的 UI，而非千篇一律的模板风格。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `design-taste-frontend` |
| 来源仓库 | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) |
| 官方网站 | https://tasteskill.dev |
| 安装位置 | `~/.claude/skills/design-taste-frontend/` 或 `.claude/skills/design-taste-frontend/`（项目级） |
| 当前版本 | v2（experimental，2026 大幅重写，默认安装） |
| 许可证 | MIT |
| Stars | 39K+ |

## 安装

```bash
# 安装全部子 Skill（推荐）
npx skills add Leonxlnx/taste-skill

# 只安装默认主 Skill（v2）
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"

# 固定使用 v1（兼容旧项目）
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend-v1"
```

## 子 Skill 目录

| 子 Skill（文件夹） | 安装名 | 描述 |
|---|---|---|
| `taste-skill` | `design-taste-frontend` | **默认主 Skill（v2 experimental）** — 读取简报、推断设计语言、调节 VARIANCE/MOTION/DENSITY 三个旋钮 |
| `taste-skill-v1` | `design-taste-frontend-v1` | 原始 v1，为依赖其精确行为的项目保留 |
| `gpt-tasteskill` | `gpt-taste` | 针对 GPT/Codex 的加强变体，更高 layout variance 和 GSAP 动效 |
| `image-to-code-skill` | `image-to-code` | 图片优先流程：生成→分析→实现 |
| `redesign-skill` | `redesign-existing-projects` | 审计并重设计已有项目 UI |
| `soft-skill` | `high-end-visual-design` | 精致、沉稳、高端视觉风格，带弹簧动效 |
| `minimalist-skill` | `minimalist-ui` | Editorial Notion/Linear 极简风格 |
| `brutalist-skill` | `industrial-brutalist-ui` | 瑞士字体、强对比、实验性布局 |
| `stitch-skill` | `stitch-design-taste` | 与 Google Stitch 兼容的设计规则 |
| `output-skill` | `full-output-enforcement` | 防止 AI 输出被截断 |

## 触发条件

安装后，Agent 在以下场景会自动激活 Skill：

- 任何 UI 组件/页面/应用构建请求
- 前端改版或视觉重构
- 生成落地页、Dashboard、产品页面

## 与仓库现有 Skills 的关系

| 本仓库 Skill | 关系 |
|---|---|
| [skills/frontend-design](../frontend-design/) | taste-skill 聚焦"设计语言推断与反泛化"；frontend-design (impeccable) 聚焦"设计领域专业指导和命令集"，两者互补 |
| [skills/impeccable](../impeccable/) | 同上，impeccable 适合需要精细审计命令（`/audit`、`/polish`）的场景 |
| [skills/ui-ux-pro-max](../ui-ux-pro-max/) | ui-ux-pro-max 提供可搜索的样式/色板/字体数据库；taste-skill 侧重行为约束，防止 AI 输出"AI 味" |
| [skills/design-md](../design-md/) | taste-skill 可与 DESIGN.md 联动，先用 DESIGN.md 定义设计令牌，再用 taste-skill 约束生成行为 |

## 支持的 AI 工具

Cursor、Claude Code、Codex CLI、Gemini CLI、v0、Lovable、OpenCode 等所有支持 `SKILL.md` 格式的工具。
