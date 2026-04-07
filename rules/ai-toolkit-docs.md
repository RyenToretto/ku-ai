# ai-toolkit-docs.mdc 设计说明

## 设计意图

确保每次新增或更新 AI 工具（MCP、Skill、Rule 等）时，都会同步在 ku-ai 知识库中维护对应的文档。防止知识库与实际安装的工具脱节。

## 适用场景

- 安装新的 MCP 服务器后
- 安装新的 Agent Skill 后
- 创建新的全局 Cursor Rule 后
- 更新上述任何工具的配置或版本后
- 自动应用（`alwaysApply: true`）

## 期望行为

AI Agent 在完成工具安装/配置后，应主动提醒并协助在 ku-ai 仓库创建对应文档。
