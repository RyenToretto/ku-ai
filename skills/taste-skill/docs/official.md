# Taste Skill — 官方资源

## 核心链接

| 资源 | 链接 |
|------|------|
| GitHub 仓库 | https://github.com/Leonxlnx/taste-skill |
| 官方网站 | https://tasteskill.dev |
| 更新日志 | https://github.com/Leonxlnx/taste-skill/blob/main/CHANGELOG.md |
| Skills CLI | https://github.com/vercel-labs/agent-skills |

## 项目元信息

| 字段 | 值 |
|------|-----|
| 创建时间 | 2026-02-19 |
| 最后更新 | 2026-05-26 |
| 主语言 | Shell（安装脚本 + SKILL.md 规则文件） |
| 许可证 | MIT |
| Stars | 39K+ |
| Forks | 2.7K+ |
| Contributors | 2（Leonxlnx, Blueemi） |

## v2 的主要变化（2026 重写）

- Agent 在开始任何实现前先读取简报，自动推断设计方向（不再使用通用模板）
- 三个可调节旋钮：
  - **VARIANCE**：布局和视觉变化幅度
  - **MOTION**：动效类型和强度（默认 GSAP 规范）
  - **DENSITY**：内容密度和间距
- 内置重设计审计流程（旧项目重构时先执行 audit）
- 严格的 pre-flight 检查清单，执行前验证设计系统映射
- 硬性禁止 em-dash（避免 AI 特征文案）
- 内置 GSAP 代码骨架（规范动效实现方式）

## 稳定性说明

v2 处于 `experimental` 状态，仍在迭代。安装名 `design-taste-frontend` 保持稳定，规则措辞在 v2.0.0 正式版前可能变化。旧项目如不想跟随迭代，可固定 `design-taste-frontend-v1`。

## 相关社区资源

- DEV Community 介绍文章：https://dev.to/wonderlab/open-source-project-of-the-day-89-taste-skill-give-your-ai-agent-good-design-taste-10l0
- 技术解读：https://runany.dev/blog/taste-skill-anti-slop-frontend/
