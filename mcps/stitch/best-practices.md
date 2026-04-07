# Google Stitch MCP 最佳实践

## 推荐做法

- 使用 DESIGN.md 维护一致的设计系统，跨项目共享品牌规范
- 用 Vibe Design（情感化描述）代替线框图，获得更好的生成效果
- 先用 `enhance-prompt` skill 优化 prompt，再调用 Stitch 生成
- 生成后立即导出代码，在本地进一步调整

## Prompt 技巧

- 描述氛围和情感：「现代、暗色、科技感、渐变紫色」
- 指定具体组件：「顶部导航栏 + 英雄区域 + 三列特性卡片 + CTA 按钮」
- 引用 DESIGN.md 中的 token：「使用 primary-500 作为 CTA 背景色」
- 指定交互状态：「悬停时按钮放大 1.05x，带 0.2s 过渡」

## 常见陷阱

- 认证过期会导致连接失败，运行 `npx @_davideast/stitch-mcp doctor` 排查
- Stitch 目前不原生支持 Vue 组件导出，但支持 HTML/Tailwind，可手工转换
- 免费额度有限，复杂项目建议用 Flash 模型快速迭代

## 与其他工具配合

- **design-md Skill**：将 Stitch 项目的设计系统导出为 DESIGN.md
- **enhance-prompt Skill**：优化 UI 描述后再提交给 Stitch
- **stitch-design Skill**：统一的设计工作流入口
- **Impeccable Skill**：用 `/audit` 审计 Stitch 生成的设计
