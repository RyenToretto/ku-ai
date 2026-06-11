# CodeGraph — 使用示例

## 初始化一个新项目

```bash
# 进入项目目录
cd my-project

# 运行交互式安装器（推荐）
npx @colbymchenry/codegraph

# 安装器提示：
# ? Which agents to configure? (auto-detected: Cursor, Claude Code)
# > [x] Cursor
# > [x] Claude Code
# ? Install codegraph globally on PATH? (Y/n)
# ? Apply to this project only or all projects? (This project)
```

## 全局安装后手动配置

```bash
# 全局安装
npm i -g @colbymchenry/codegraph

# 在项目目录初始化并配置 Agent
codegraph install
```

## Cursor 手动 MCP 配置

在 `~/.cursor/mcp.json` 中添加：

```json
{
  "mcpServers": {
    "codegraph": {
      "command": "codegraph",
      "args": ["mcp"]
    }
  }
}
```

## 典型工作流

### 场景 1：理解陌生代码库

```
# 直接向 Agent 提问，CodeGraph 自动提供上下文
"这个项目的 UserAuthService 在哪里被调用？列出所有调用方"
"auth.ts 依赖哪些模块，如果修改它会影响哪些文件？"
```

### 场景 2：安全重构

```
"我要重命名 processPayment 函数，帮我找出所有需要同步修改的地方"
"将 UserRepository 从 MySQL 迁移到 PostgreSQL，分析影响范围"
```

### 场景 3：新功能开发

```
"我要新增一个 WebSocket 消息处理器，参考项目中现有的处理器模式"
"基于当前代码结构，建议在哪里添加缓存层最合适？"
```

## 以代码方式集成

```typescript
import CodeGraph from '@colbymchenry/codegraph';

const graph = new CodeGraph({ projectPath: './src' });
await graph.initialize();

// 查询某个符号的所有引用
const refs = await graph.findReferences('processPayment');
```
