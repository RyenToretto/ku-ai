# 知识库维护规范

本文档定义了 ku-ai 知识库的维护规则。每次新增/更新 MCP、Skill、Rule 或其他 AI 工具时，必须遵循此规范。

## 新增 MCP 服务器

1. 在 `mcps/` 下创建以 MCP 名称命名的文件夹
2. 必须包含以下文件：

```
mcps/<name>/
├── README.md           # 概述、功能、安装命令、Cursor 配置 JSON
├── docs/               # 官方文档摘要或重要页面截取
│   └── official.md     # 官方文档链接和关键内容
├── examples/           # 使用示例（prompt 示例、调用示例）
│   └── basic.md
└── best-practices.md   # 最佳实践、注意事项、常见问题
```

3. `README.md` 必须包含：
   - 一句话描述
   - npm 包名和版本
   - `mcp.json` 配置片段
   - 前置条件（Node.js 版本、认证等）
   - 提供的 tools 列表
   - 官方仓库链接

## 新增 Agent Skill

1. 在 `skills/` 下创建以 Skill 名称命名的文件夹
2. 必须包含以下文件：

```
skills/<name>/
├── README.md           # 概述、功能、安装命令
├── SKILL.md            # 完整的 SKILL.md 内容副本
├── docs/               # 参考文档
│   └── official.md
├── examples/           # 使用示例
│   └── basic.md
└── best-practices.md   # 最佳实践
```

3. `README.md` 必须包含：
   - Skill 名称和描述
   - 安装命令（`npx skills add ...`）
   - 安装位置
   - 触发条件（什么时候会被激活）
   - 依赖关系（需要哪些 MCP 或其他 Skill）

## 新增 Cursor Rule

1. 在 `rules/` 下直接放置 `.mdc` 文件
2. 同时创建同名的 `.md` 说明文件

```
rules/
├── <name>.mdc          # 规则文件本体
└── <name>.md           # 规则的设计意图、适用场景、示例
```

## 新增 Cursor 插件 Skill

1. 在 `plugins/<plugin-name>/` 下按 Skill 名创建子目录
2. 结构同 Agent Skill

## 提交规范

- 每次新增/更新一个工具做一次提交
- Commit message 使用中文
- 格式：`feat: 新增 <类型> <名称> 文档`
- 示例：`feat: 新增 MCP stitch 文档`

## 同步检查清单

新增任何 AI 工具后，对照此清单：

- [ ] 创建了独立文件夹
- [ ] 编写了 README.md（含安装命令和配置）
- [ ] 记录了官方文档链接
- [ ] 添加了使用示例
- [ ] 总结了最佳实践
- [ ] 更新了根 README.md 的目录结构
- [ ] 提交并 push 到 master
