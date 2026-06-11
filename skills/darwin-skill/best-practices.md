# Darwin Skill — 最佳实践

## 何时使用

- 已积累一批 Skills，但感觉 Agent 输出质量停滞不前
- 某个 Skill 在特定场景触发不精准或输出不符合预期
- 想对 Skill 进行有依据的迭代，而非凭直觉修改
- 定期维护 Skill 库（类似 "Skill 健康检查"）

## 何时不用

- 某个 Skill 刚安装，还没有足够的实际使用经验作为基线
- 完全凭直觉写的 Skill，测试集难以覆盖（先积累使用反馈再优化）
- 生产关键 Skill，不能在未经完整回归的情况下修改

## 安全操作原则

- Darwin 在**独立 git 分支**上操作，所有变更均可回滚
- 每次优化循环都有人工确认节点——不要因为分数提升就盲目批准，要同时阅读 diff
- 批准前检查：优化后的 Skill 是否改变了触发条件（可能影响其他场景）
- 优化完成后，用实际任务对 Skill 进行手动验证

## 组合使用建议

| 组合 | 场景 |
|------|------|
| Nuwa Skill → Darwin Skill | 用 Nuwa 创建新 Skill，用 Darwin 持续进化 |
| Darwin + Git Worktree | 在隔离的 worktree 中运行 Darwin 实验，不影响主线工作 |
| Darwin + 本仓库文档记录 | 将每次重要优化的前后分数、改动摘要记录在本仓库的 `examples/` 或 `best-practices.md` 中 |

## 常见风险

| 风险 | 应对方式 |
|------|------|
| 测试提示集覆盖不全，分数虚高 | 优化完成后用真实任务手动验证；必要时手动扩充 `test-prompts.json` |
| 循环次数过多，分支历史混乱 | 每次优化后及时清理 Darwin 分支，合并到主分支 |
| 过度优化导致 Skill 对单一场景过拟合 | 保持测试集的多样性，覆盖 Skill 的所有预期触发场景 |
| 不小心批准了降低其他 Skill 的协作效果的变更 | 优化完成后进行"Skill 协作测试"，验证与依赖 Skill 的联动 |

## 注意事项

- Darwin 面向 Claude Code 和其他支持 SKILL.md 格式的工具，与 Cursor Rules（.mdc）不同
- 本仓库记录的是 darwin-skill 的调研文档，实际优化操作发生在本机 Skills 目录，不在此仓库运行
