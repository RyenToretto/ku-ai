# Skill: design-md

> 分析 Stitch 项目并将设计系统综合为 DESIGN.md 文件。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `design-md` |
| 来源 | `google-labs-code/stitch-skills` |
| 安装位置 | `~/.agents/skills/design-md/` |
| 触发条件 | 需要生成或更新项目设计系统文档时 |
| 依赖 | Stitch MCP Server |

## 安装

```bash
npx skills add google-labs-code/stitch-skills --skill design-md -g -y
```

## 功能

- 分析 Stitch 项目的所有屏幕，提取设计令牌
- 生成标准化的 DESIGN.md 文件
- 包含色彩、排版、间距、组件模式等完整设计系统
- 输出格式兼容 AI Agent 消费

## 输出内容

DESIGN.md 文件包含：
- 色彩令牌（palette、语义角色、主色/辅色/强调色）
- 排版设置（字体族、字号、字重、行高映射）
- 间距和布局规则
- 组件变体和状态
- 组件模式和设计语言

## 相关链接

- [Google Stitch Skills GitHub](https://github.com/google-labs-code/stitch-skills)
- [Skills 页面](https://skills.sh/google-labs-code/stitch-skills)
- [DESIGN.md 官方规范（google-labs-code/design.md）](https://github.com/google-labs-code/design.md)
- [Stitch DESIGN.md 文档](https://stitch.withgoogle.com/docs/design-md/overview/)

## 社区资源

DESIGN.md 格式已形成活跃的社区生态，详见 [docs/awesome-design-md.md](docs/awesome-design-md.md)：

| 资源 | 描述 |
|------|------|
| [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | 89K+ ⭐ — 55+ 个真实品牌 DESIGN.md 文件，可直接复制到项目使用 |
| [google-labs-code/design.md](https://github.com/google-labs-code/design.md) | 官方规范 + CLI 工具（lint、export、spec） |
| [dimabraven/design-md](https://github.com/dimabraven/design-md) | 带 CLI 的 DESIGN.md 文件目录，可按 AI 工具过滤 |
