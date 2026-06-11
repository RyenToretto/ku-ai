# Taste Skill — 最佳实践

## 何时使用

- 启动全新前端项目，希望 AI 生成有品牌感而非通用外观的 UI
- 对已有项目进行视觉重构，消除"AI 味"（蓝色 blue-500、圆角卡片堆砌）
- 需要 AI 自主决定设计方向而非完全依赖人工指定样式
- 多工具协作场景（Cursor + Claude Code + Codex），需要统一设计约束

## 何时不用

- 项目已有严格设计系统或 Figma 规范，此时用 DESIGN.md 更精准
- 只需要功能代码、不在乎 UI 样式时，引入 Skill 增加上下文成本
- 固定在 v1 行为的遗留项目，升级前先在隔离分支测试 v2

## 子 Skill 选择指南

| 场景 | 推荐子 Skill |
|------|------|
| 通用新项目 | `design-taste-frontend`（v2 默认） |
| 高端 SaaS / 品牌页 | `high-end-visual-design` |
| 文档工具、生产力应用 | `minimalist-ui` |
| 实验性/艺术类项目 | `industrial-brutalist-ui` |
| GPT/Codex 工作流 | `gpt-taste` |
| 参考截图还原 | `image-to-code` |
| 现有项目重构 | `redesign-existing-projects` |
| Google Stitch 项目 | `stitch-design-taste` |

## 组合使用建议

- **taste-skill + DESIGN.md**：最强组合。DESIGN.md 提供精确令牌（色彩、字体、间距），taste-skill 提供行为约束（防泛化、动效规范）。
- **taste-skill + impeccable**：impeccable 用于具体领域审计和命令（`/audit`、`/animate`），taste-skill 作为基础行为层。
- **taste-skill + ui-ux-pro-max**：ui-ux-pro-max 提供可搜索的样式/色板数据库，taste-skill 负责约束 AI 实际输出行为，两者不冲突。

## 常见风险

| 风险 | 应对方式 |
|------|------|
| v2 experimental 规则措辞在迭代，输出可能变化 | 对需要稳定输出的项目固定 `design-taste-frontend-v1` |
| 项目已有设计系统，Skill 推断出不一致方向 | 在请求中明确指定设计语言或配合 DESIGN.md |
| 上下文过长导致性能问题 | 按需只安装用到的子 Skill，不全量安装 |
| 在非 Skill 格式工具（如纯 API）使用 | 可手动复制 SKILL.md 内容到 system prompt |

## 注意事项

- 此 Skill 影响 UI 美感而非代码正确性，不替代测试和代码审查。
- SKILL.md 内容由上游维护，建议定期重新安装以获取最新规则。
- 许可证为 MIT，可商业使用，请保留版权声明。
