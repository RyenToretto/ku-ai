# 调研：Karpathy CLAUDE.md 行为原则

> 本文件是调研记录，不是已激活的 Cursor Rule（不含 `.mdc` 配置）。  
> 目的是整理这套 4 条行为原则的来源、内容和在本知识库中的应用参考。

## 来源

- **原始推文**：Andrej Karpathy，[@karpathy](https://x.com/karpathy/status/2015883857489522876)
- **社区 Skill 实现**：[forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpthy-skills)（167K+ ⭐）
- **另一个实现**：[ddesmond/andrej-karpathy-skills](https://github.com/ddesmond/andrej-karpathy-skills)
- **许可证**：MIT

## 四条核心原则

### 1. Think Before Coding（编码前先思考）

解决问题：错误假设、隐藏困惑、权衡不清晰

**行为要求**：
- 明确陈述假设，发现假设可能有误时立即提出
- 遇到困惑时提问，而不是猜测
- 执行前主动阐述方案的权衡取舍

### 2. Simplicity First（简洁优先）

解决问题：过度工程化、投机性抽象

**行为要求**：
- 只实现请求的最小必要代码
- 不添加未被请求的功能
- 避免超前的抽象（YAGNI 原则）

### 3. Surgical Changes（精准修改）

解决问题：无关改动、代码风格蔓延、"顺手"重构

**行为要求**：
- 编辑范围严格限定在请求的任务
- 不重构任务范围外的邻近代码
- 不在修改功能的同时做代码风格统一

### 4. Goal-Driven Execution（目标驱动执行）

解决问题：只解决表面症状、缺乏可验证的完成标准

**行为要求**：
- 将任务转化为可验证的成功标准
- 优先考虑先写测试，循环执行直到测试通过
- 验证真正的问题被解决，而非只是症状消失

## 与 Cursor Rules 的关系

这套原则可以通过三种方式在本工具链中应用：

| 方式 | 适用场景 | 操作 |
|------|------|------|
| **Cursor Rule（.mdc）** | 希望对所有 Cursor 对话生效 | 将原则内容写入 `rules/karpathy-coding-principles.mdc`，配置 `alwaysApply: true` |
| **CLAUDE.md（项目级）** | 只对单个项目的 Claude Code 生效 | 在项目根目录创建 `CLAUDE.md` 并引入原则 |
| **AGENTS.md** | 跨 Agent 工具通用 | 在项目根目录创建 `AGENTS.md`（`CLAUDE.md` 的 symlink 或独立文件） |

### 如果要将其转为 Cursor Rule

在 `rules/` 目录创建 `.mdc` 文件：

```markdown
---
description: Karpathy Coding Principles — Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution
alwaysApply: true
---

## Think Before Coding
State assumptions explicitly. Ask when confused. Surface tradeoffs before implementing.

## Simplicity First
Implement only the minimum code necessary. No speculative abstractions, no unrequested features.

## Surgical Changes
Limit edits strictly to the requested task. Do not refactor neighboring code or drift in style.

## Goal-Driven Execution
Transform tasks into verifiable success criteria. Write tests first when possible. Loop until they pass.
```

### 如果要添加到项目 CLAUDE.md

```bash
# 直接下载并追加到 CLAUDE.md
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md >> CLAUDE.md
```

## 安装为 Agent Skill（Claude Code）

```
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install andrej-karpathy-skills@karpathy-skills
```

## 关联的本仓库 Rules

这四条原则与本仓库现有规则有高度相关性：

| 规则文件 | 相关原则 |
|----------|------|
| `rules/code-quality.md/.mdc` | Simplicity First + Surgical Changes |
| `rules/form-validation.mdc` | Goal-Driven Execution（可验证的完成标准） |

## 评估与应用建议

这套原则是**行为护栏**，而非严格合同：

- 适合作为通用编码行为基线，与项目特定指令合并使用
- "Surgical Changes" 对 AI Agent 的作用最明显——避免 Agent 在修改一个函数时"顺手"改了整个模块
- "Think Before Coding" 在复杂任务中防止 Agent 基于错误假设沉默执行
- 与 Superpowers 的 `brainstorming` Skill 高度互补（一个约束行为，一个提供流程框架）

## 已知局限

- 这些原则是 Karpathy 对 LLM 编码行为的观察，主要面向 Claude Code 场景
- 在 Cursor 中以 Rule 形式使用时，效果取决于 Rule 触发机制（alwaysApply vs. 按场景触发）
- 4 条原则比较宽泛，不替代具体的技术规范（测试框架、代码风格等）
