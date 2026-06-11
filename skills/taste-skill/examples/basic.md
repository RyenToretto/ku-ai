# Taste Skill — 使用示例

## 安装示例

```bash
# 一次性安装全部子 Skill（推荐首次使用）
npx skills add Leonxlnx/taste-skill

# 只安装主 Skill（v2 默认）
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"

# 安装极简风格子 Skill
npx skills add https://github.com/Leonxlnx/taste-skill --skill "minimalist-ui"

# 安装高端视觉风格
npx skills add https://github.com/Leonxlnx/taste-skill --skill "high-end-visual-design"

# 安装粗犷实验风格
npx skills add https://github.com/Leonxlnx/taste-skill --skill "industrial-brutalist-ui"
```

安装后，SKILL.md 会出现在 `~/.claude/skills/design-taste-frontend/SKILL.md`（或项目级 `.claude/skills/`）。

## 自然语言调用示例

安装后，正常向 Agent 发送请求即可，无需手动调用：

```
帮我为这个 SaaS 产品构建一个定价页面，使用清晰的层级对比
```

```
重构这个 Dashboard 组件，让它看起来更有质感、不像模板
```

```
为这个移动端登录页面添加入场动效
```

## 图片参考流程（image-to-code）

```bash
# 先安装 image-to-code 子 Skill
npx skills add https://github.com/Leonxlnx/taste-skill --skill "image-to-code"
```

然后向 Agent 提供截图：

```
参考这张截图（@截图.png），用 React + Tailwind 实现这个 UI，保留原始的设计语言
```

## 重设计已有项目（redesign-skill）

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "redesign-existing-projects"
```

```
审计当前项目的主页，找出视觉设计问题并提出重设计方案
```

## 与 DESIGN.md 联动

```
# 先让 Agent 生成 DESIGN.md（使用 design-md skill）
# 再基于 DESIGN.md 用 taste-skill 生成组件

请参考项目根目录的 DESIGN.md，为产品页面生成 Hero 区域
```

## 典型工作流

```
用户：为一个 AI 写作工具产品构建落地页
Agent：（读取简报 → 推断设计语言：简洁/深色/智能感） 
       → 应用 VARIANCE=中 MOTION=轻柔 DENSITY=低
       → 生成带品牌感的 Landing Page，非通用模板
```
