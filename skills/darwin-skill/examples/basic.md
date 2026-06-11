# Darwin Skill — 使用示例

## 安装

```bash
npx skills add alchaincyf/darwin-skill
```

## 优化单个 Skill

```
优化 design-taste-frontend 这个 Skill
```

或英文：

```
optimize design-taste-frontend skill
```

Agent 会执行以下步骤：
1. 分析 `design-taste-frontend/SKILL.md` 当前版本
2. 生成测试提示集 `test-prompts.json`
3. 评分基线（如：73/100）
4. 修改 SKILL.md，重新评分（如：81/100）
5. 展示 diff，等待用户确认：
   ```
   ┌─────────────────────────────────────────┐
   │ Skill: design-taste-frontend            │
   │ 基线分: 73/100  →  优化后: 81/100       │
   │ 主要改动: 强化了 VARIANCE 旋钮说明       │
   │ 是否保留此次优化？ [Y/n]                 │
   └─────────────────────────────────────────┘
   ```

## 全量优化所有 Skills

```
优化所有 Skills
```

Darwin 会按顺序处理每个 Skill，每个循环后都会暂停等待用户确认。

## 回滚到上一版本

若对某次优化不满意，告知 Agent：

```
回滚 design-taste-frontend 的最后一次 Darwin 优化
```

Darwin 在独立 git 分支上工作，直接 `git revert` 即可。

## 典型工作流

```
# 1. 安装并使用某个 Skill 一段时间，积累使用经验
npx skills add Leonxlnx/taste-skill

# 2. 发现 Skill 在某些场景表现不理想
# 3. 运行 Darwin 优化
优化 design-taste-frontend，重点改善它在 SaaS Dashboard 场景的输出

# 4. 查看报告，批准或拒绝
# 5. 将优化后的 SKILL.md 合并到主分支
```

## 结合本仓库使用

由于本仓库只记录调研文档而不同步 SKILL.md 实体，Darwin 的实际操作应在本机 `~/.claude/skills/` 或项目的 `.claude/skills/` 目录进行，不在本仓库执行。
