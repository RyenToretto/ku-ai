# Skill: verification-before-completion

> 声称工作完成前必须运行验证命令并确认输出，用证据支持而非空口断言。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `verification-before-completion` |
| 来源 | `superpowers` 插件 / 全局预装 |
| 安装位置 | `~/.agents/skills/verification-before-completion/` |
| 触发条件 | 即将声称工作完成/修复/通过时 |
| 依赖 | 无 |

## 核心原则

- **证据先于断言**：先运行验证，再说 "搞定了"
- **不跳过验证**：即使 "看起来应该能工作"
- **记录证据**：将验证命令的输出作为完成的依据

## 验证清单

- [ ] 代码编译通过（无类型错误）
- [ ] Lint 检查通过
- [ ] 相关测试通过
- [ ] 功能手动验证（如适用）
- [ ] 无意外的副作用

## 典型场景

```
✗ 错误做法：
"我已经修复了这个 bug，问题出在 null check。"

✓ 正确做法：
"运行 npm run test -- --filter login 的结果如下：
[粘贴测试输出]
所有 12 个测试通过，bug 已修复。"
```
