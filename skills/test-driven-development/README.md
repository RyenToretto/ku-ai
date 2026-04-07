# Skill: test-driven-development

> TDD 测试驱动开发流程，实现功能或修复 bug 前先写测试。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `test-driven-development` |
| 来源 | `superpowers` 插件 / 全局预装 |
| 安装位置 | `~/.agents/skills/test-driven-development/` |
| 触发条件 | 实现任何功能或修复 bug 前 |
| 依赖 | 项目需有测试框架 |

## TDD 循环

1. **Red** — 写一个失败的测试
2. **Green** — 用最简单的方式让测试通过
3. **Refactor** — 重构代码，保持测试通过

## 核心原则

- 测试先于实现代码
- 每次只写一个失败测试
- 实现代码只够让当前测试通过
- 重构时不修改测试（除非测试本身有问题）
- 所有测试通过后才算完成

## 适用场景

- 新功能开发
- Bug 修复（先写复现 bug 的测试）
- API 设计（通过测试驱动接口设计）
- 重构（测试作为安全网）
