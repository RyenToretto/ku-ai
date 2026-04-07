# Sequential Thinking 官方文档

## 文档链接

- GitHub: https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking
- npm: https://www.npmjs.com/package/@modelcontextprotocol/server-sequential-thinking
- MCP 协议: https://modelcontextprotocol.io

## 核心概念

Sequential Thinking 是 MCP 官方提供的思维链服务器，设计用于需要多步推理的复杂任务。

### Tool 参数

`sequentialthinking` tool 接受以下参数：

- `thought`：当前思考步骤的内容
- `nextThoughtNeeded`：是否需要更多思考步骤
- `thoughtNumber`：当前步骤编号
- `totalThoughts`：预估总步骤数（可动态调整）
- `isRevision`：是否是对之前步骤的修正
- `revisesThought`：修正的目标步骤编号
- `branchFromThought`：从哪个步骤创建分支
- `branchId`：分支标识符

### 工作流

1. 发起第一个思考步骤，设定预估总步骤数
2. 逐步推理，每步聚焦一个子问题
3. 如需修正，设置 `isRevision: true` 和 `revisesThought: N`
4. 如需分支探索，设置 `branchFromThought` 和 `branchId`
5. 最后一步设置 `nextThoughtNeeded: false` 结束
