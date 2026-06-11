# Skill: darwin-skill

> 达尔文 Skill——让你的 SKILL.md 自主进化的系统：评估 → 改进 → 测试 → 保留或回滚。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `darwin-skill` |
| 来源仓库 | [alchaincyf/darwin-skill](https://github.com/alchaincyf/darwin-skill) |
| 安装位置 | `~/.claude/skills/darwin-skill/` |
| 创作者 | 花叔 Huashu（@alchaincyf） |
| 灵感来源 | [Andrej Karpathy 的 autoresearch](https://github.com/karpathy/autoresearch) |
| 许可证 | MIT |
| Stars | 3.5K+ |
| 微软认可 | 已被 [SkillOpt 仓库](https://github.com/microsoft/SkillOpt) 列入官方集成名单 |

## 安装

```bash
npx skills add alchaincyf/darwin-skill
```

无法访问 GitHub 时，可下载离线包：
```bash
# 下载 zip，解压后手动放置
# https://pub-161ae4b5ed0644c4a43b5c6412287e03.r2.dev/skills/darwin-skill.zip
mkdir -p ~/.claude/skills/darwin-skill/
# 将 SKILL.md 放入该目录
```

## 核心概念：Skill 优化循环

Darwin Skill 将 Karpathy 的 autoresearch 循环应用到 Skill 优化场景：

| autoresearch 概念 | darwin-skill 对应 | 说明 |
|---|---|---|
| `program.md` | `SKILL.md`（本 Skill） | 定义评估标准和约束 |
| `train.py` | 每个目标 SKILL.md | 每次实验中被修改的资产 |
| `val_bpb` | 9 维加权评分（满分 100） | 可量化的优化目标 |
| `git ratchet` | keep/revert 机制 | 只保留有改进的提交 |
| `test set` | `test-prompts.json` | 验证改进是否真实有效 |

**关键差异**：autoresearch 是全自动的（loss 是客观的），darwin-skill 保留人工判断节点——每次优化循环后 Agent 暂停并展示 diff 和分数变化，等待用户确认后再继续。这体现了"Skill 质量比 loss 更主观"的设计哲学。

## 触发条件

对 Agent 说出以下指令：
- `"优化所有 Skills"` — 全量扫描并优化
- `"优化 [skill-name]"` — 优化指定 Skill
- `"optimize all skills"` / `"optimize [skill-name]"` — 英文指令同样有效

## 工作流程

1. **扫描** — 列出所有可优化的目标 SKILL.md
2. **建立 git 分支** — 在新分支上进行实验，保证可回滚
3. **设计测试提示** — 为目标 Skill 生成 `test-prompts.json`
4. **基线评分** — 对当前 Skill 进行 9 维评分（满分 100）
5. **优化循环** — Agent 修改 SKILL.md，在测试集上评分
6. **人工确认** — 展示 diff 和分数变化，等待用户批准
7. **保留或回滚** — 分数提升则 `git commit`，否则 `git revert`
8. **汇总报告** — 输出前后分数对比表

## 配套工具

Darwin 与另一个 Skill 形成互补：

| Skill | 功能 |
|------|------|
| [Nuwa Skill](https://github.com/alchaincyf/nuwa-skill) | 创建新 Skill（女娲造物） |
| **Darwin Skill** | 让已有 Skill 进化（达尔文演化） |
