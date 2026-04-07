# ku-ai — AI 技能知识库

个人 AI 开发工具链的统一知识库，包含所有已安装和使用的 MCP 服务器、Agent Skills、Cursor Rules、IDE 插件的文档、参考资料、示例和最佳实践。

## 目录结构

```
ku-ai/
├── mcps/                    # MCP (Model Context Protocol) 服务器
│   ├── chrome-devtools/     # Chrome DevTools 调试
│   ├── context7/            # 库文档实时查询
│   ├── playwright/          # 浏览器自动化测试
│   ├── sequential-thinking/ # 结构化思维链
│   ├── stitch/              # Google Stitch UI 设计生成
│   └── lighthouse/          # 性能/无障碍/SEO 审计
│
├── skills/                  # 全局 Agent Skills (~/.agents/skills/)
│   ├── design-md/           # Stitch DESIGN.md 生成
│   ├── enhance-prompt/      # UI Prompt 优化
│   ├── find-skills/         # Skills 发现与安装
│   ├── impeccable/          # 7 大设计领域专业指导
│   ├── stitch-design/       # Stitch 统一设计工作流
│   ├── systematic-debugging/# 系统化调试流程
│   ├── test-driven-development/ # TDD 测试驱动开发
│   └── verification-before-completion/ # 完成前验证
│
├── skills-cursor/           # Cursor 内置 Skills (~/.cursor/skills-cursor/)
│   ├── babysit/             # PR 维护与 CI 修复
│   ├── create-rule/         # 创建 Cursor Rules
│   ├── create-skill/        # 创建 Agent Skills
│   ├── create-subagent/     # 创建自定义 Subagent
│   ├── migrate-to-skills/   # 迁移 Rules 到 Skills
│   ├── shell/               # Shell 命令执行
│   └── update-cursor-settings/ # 修改编辑器设置
│
├── plugins/                 # Cursor 插件 Skills
│   ├── figma/               # Figma 设计集成 (7 skills)
│   └── superpowers/         # Superpowers 工作流 (14 skills)
│
├── rules/                   # 全局 Cursor Rules (~/.cursor/rules/)
│   ├── code-quality.mdc     # 代码质量规范
│   └── git-workflow.mdc     # Git 工作流规范
│
├── templates/               # 新建 MCP/Skill 的模板
│   ├── mcp-template/        # MCP 文档模板
│   └── skill-template/      # Skill 文档模板
│
└── CONVENTIONS.md           # 知识库维护规范
```

## 安装位置映射

| 本仓库路径 | 实际安装位置 | 说明 |
|-----------|-------------|------|
| `mcps/*` | `~/.cursor/mcp.json` | 全局 MCP 配置 |
| `skills/*` | `~/.agents/skills/` | 全局 Agent Skills |
| `skills-cursor/*` | `~/.cursor/skills-cursor/` | Cursor 内置 Skills |
| `plugins/*` | `~/.cursor/plugins/cache/cursor-public/` | Cursor 插件 |
| `rules/*` | `~/.cursor/rules/` | 全局 Cursor Rules |

## 使用方式

本仓库仅作为文档和知识管理用途，不需要安装或运行。每个子目录包含：

- `README.md` — 概述、安装命令、配置方式
- `docs/` — 官方文档摘要或链接
- `examples/` — 使用示例
- `best-practices.md` — 最佳实践总结

## 维护

新增任何 MCP/Skill/Rule 时，必须同步在本仓库创建对应文件夹和文档，详见 [CONVENTIONS.md](CONVENTIONS.md)。
