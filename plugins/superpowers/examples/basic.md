# Superpowers — 使用示例

## 安装

```
# 在 Cursor Agent 对话框中
/add-plugin superpowers
```

或在 Cursor 插件市场搜索 `superpowers` 并点击安装。

## 验证安装

安装后，在 Cursor 中打开新对话，描述一个任务。Agent **不应该**立即开始写代码，而是先检查相关 Skills：

```
用户：帮我实现一个用户登录功能

Agent（正确行为）：
检查可用 Skills...
发现相关 Skills: brainstorming, writing-plans, test-driven-development
先通过 brainstorming 理解需求...
```

## 典型工作流示例

### 新功能开发

```
1. [brainstorming]
   用户："我想在 App 里加一个通知系统"
   Agent：进行苏格拉底式追问——哪些事件触发通知？支持哪些渠道？
   
2. [writing-plans]
   Agent：基于讨论结果，编写带验证步骤的详细计划
   
3. [test-driven-development]
   Agent：先写测试（RED），再实现（GREEN），再重构（REFACTOR）
   
4. [requesting-code-review]
   Agent：自动执行预检清单后，发起代码审查
   
5. [finishing-a-development-branch]
   Agent：处理 PR/合并决策流程
```

### Bug 修复

```
1. [systematic-debugging]
   Agent：4 阶段根因分析（复现 → 假设 → 验证 → 修复）
   
2. [verification-before-completion]
   Agent：修复后验证问题是否真正解决（而非只是表面症状消失）
```

### 复杂任务并行处理

```
用户："我需要同时重构 API 层和更新前端组件"

[dispatching-parallel-agents]
Agent：识别这两个任务相互独立，
       分派两个 Subagent 并行处理，
       保留自身上下文以合并结果
```

## 手动触发 Skill（在 Cursor 中）

```
告诉我你的 Skills，然后使用 brainstorming Skill 讨论这个功能
```

```
使用 systematic-debugging Skill 帮我定位这个 bug
```
