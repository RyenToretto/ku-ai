# Skill: stitch-design

> Stitch 设计工作的统一入口，整合 prompt 增强、设计系统合成和屏幕生成/编辑。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `stitch-design` |
| 来源 | `google-labs-code/stitch-skills` |
| 安装位置 | `~/.agents/skills/stitch-design/` |
| 触发条件 | 进行 Stitch 相关的设计工作时 |
| 依赖 | Stitch MCP Server |

## 安装

```bash
npx skills add google-labs-code/stitch-skills --skill stitch-design -g -y
```

## 功能

- **Prompt 增强**：自动优化 UI 描述（注入 UI/UX 关键词和氛围）
- **设计系统合成**：生成 `.stitch/DESIGN.md`
- **屏幕生成**：通过 Stitch MCP 生成高保真界面
- **屏幕编辑**：修改已有的 Stitch 屏幕

## 包含的参考文档

- `references/design-mappings.md` — 设计映射表
- `references/prompt-keywords.md` — Prompt 关键词库
- `references/tool-schemas.md` — Tool 参数 Schema

## 工作流

1. 用户描述 UI 需求
2. Skill 自动增强 prompt（添加关键词和上下文）
3. 调用 Stitch MCP 生成屏幕
4. 可选：合成 DESIGN.md

## 相关链接

- [GitHub](https://github.com/google-labs-code/stitch-skills)
