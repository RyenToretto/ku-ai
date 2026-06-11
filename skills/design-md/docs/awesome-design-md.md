# Awesome DESIGN.md 生态调研

## 什么是 DESIGN.md

DESIGN.md 是由 **Google Stitch** 引入并开源的 AI Agent 设计系统协议文件。它是一个放在项目根目录的 Markdown 文件，让 AI 编码 Agent（Cursor、Claude Code、Codex、Windsurf 等）在生成 UI 时有一份机器可读的设计规范，而非每次凭空猜测样式。

### DESIGN.md 的两层结构

```markdown
---
# 第一层：YAML front matter（机器可读设计令牌）
version: alpha
name: my-design-system

colors:
  primary: "#0077FF"
  surface: "#F8FAFC"
  border: "#E2E8F0"

typography:
  heading: "Inter, sans-serif"
  body: "Inter, sans-serif"
  mono: "JetBrains Mono, monospace"
---

## 第二层：Markdown 正文（人类可读设计理念）

### Color Philosophy
Primary blue conveys trust and clarity. Use only for CTAs and key interactive elements...

### Typography
Use Inter for all text. Heading weights: 600-700. Body weight: 400...
```

令牌提供精确数值，正文解释"为什么"以及如何应用。

## 官方规范来源

### Google Labs（官方规范）

| 资源 | 链接 |
|------|------|
| 官方规范仓库 | https://github.com/google-labs-code/design.md |
| npm 包 | `npm install @google/design.md` |
| Google 博客公告 | https://blog.google/innovation-and-ai/models-and-research/google-labs/stitch-design-md/ |
| Stitch 文档 | https://stitch.withgoogle.com/docs/design-md/overview/ |
| 规范详情 | https://stitch.withgoogle.com/docs/design-md/specification/ |

规范状态：`alpha`，仍在迭代中。

#### 官方 CLI 工具

```bash
# 安装
npm install @google/design.md

# 校验 DESIGN.md 文件
design.md lint

# 导出为其他格式（CSS 变量、JSON 等）
design.md export

# 查看规范（向 Agent 注入格式说明）
design.md spec
```

#### Linting 规则

官方 linter 对 DESIGN.md 执行 9 条规则检查，包括令牌完整性、颜色格式、组件属性合法性等。

## 社区资源仓库

### 1. VoltAgent/awesome-design-md（推荐）

| 字段 | 值 |
|------|-----|
| GitHub | https://github.com/VoltAgent/awesome-design-md |
| Stars | 89K+ |
| 许可证 | MIT |
| 文件数量 | 55+ 个品牌 DESIGN.md |

从真实网站提取的品牌 DESIGN.md 文件集合。每个文件包含：
- `DESIGN.md` — 设计系统（Agent 读取文件）
- `preview.html` — 可视化色板、字型比例、按钮、卡片
- `preview-dark.html` — 暗色模式可视化

**用法**：
```bash
# 从仓库复制某个品牌的 DESIGN.md 到项目根目录
cp stripe/DESIGN.md ./DESIGN.md

# 告诉 Agent
"严格按照 DESIGN.md 中的设计系统构建这个组件"
```

**收录的品牌示例**：Stripe（金融）、Linear（极简/深色）、Vercel（单色/Geist）、Notion（文档风格）、GitHub（Primer 系统）、Supabase（开源/绿色）、Resend（极简开发者）、VoltAgent（AI/深色/翠绿）等。

### 2. dimabraven/design-md

| 字段 | 值 |
|------|-----|
| GitHub | https://github.com/dimabraven/design-md |
| 配套网站 | https://designmd.directory |

带 CLI 工具的 DESIGN.md 文件目录，可按工具类型过滤：
- [DESIGN.md for Cursor](https://designmd.directory/for/cursor)
- [DESIGN.md for Claude Code](https://designmd.directory/for/claude-code)
- [DESIGN.md for Codex](https://designmd.directory/for/codex)
- [DESIGN.md for Windsurf](https://designmd.directory/for/windsurf)

## 在 Cursor 中使用 DESIGN.md

### 方式一：项目级上下文（推荐）

将 `DESIGN.md` 放在项目根目录，Cursor Agent 会自动将其作为上下文。

```
项目根目录/
├── DESIGN.md      ← Agent 自动读取
├── src/
└── ...
```

### 方式二：配合 CLAUDE.md

在 `CLAUDE.md`（或 `AGENTS.md`）中引用：

```markdown
# 设计规范
本项目的设计系统定义在 DESIGN.md 中。生成任何 UI 代码前，必须先读取并严格遵循该文件中的设计令牌和规则。
```

### 方式三：在请求中显式引用

```
@DESIGN.md 基于这份设计系统，帮我构建 Hero 区域组件
```

## 与本仓库 design-md Skill 的关系

| 资源 | 定位 |
|------|------|
| 本仓库 `skills/design-md/SKILL.md` | **生成** DESIGN.md 的 Agent Skill（从 Stitch 项目中提取并生成 DESIGN.md） |
| `VoltAgent/awesome-design-md` | **现成的** 品牌 DESIGN.md 文件库（直接复制使用） |
| `google-labs-code/design.md` | **规范** — DESIGN.md 格式的官方标准和 CLI 工具 |

**推荐工作流**：
1. 新项目：从 `awesome-design-md` 找一个接近品牌的文件作为起点
2. 已有 Stitch 项目：用 `design-md` Skill 生成精准的 DESIGN.md
3. 自定义：参考官方规范手写/调整令牌
4. 使用：放入项目根目录，告知 Agent 遵循

## 注意事项

- DESIGN.md 规范仍处于 `alpha` 状态，格式可能变化
- 各工具对 DESIGN.md 的解读深度不同，Stitch > Claude Code > Cursor（按集成程度）
- 从 `awesome-design-md` 借用品牌文件时，注意只取设计语言，不要直接复制品牌 logo/颜色声称是自己品牌
