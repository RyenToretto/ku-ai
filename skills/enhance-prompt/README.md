# Skill: enhance-prompt

> 将模糊的 UI 想法转化为精确的、Stitch 优化的 prompt。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `enhance-prompt` |
| 来源 | `google-labs-code/stitch-skills` |
| 安装位置 | `~/.agents/skills/enhance-prompt/` |
| 触发条件 | 准备向 Stitch 提交 UI 生成请求前 |
| 依赖 | 无（可独立使用） |

## 安装

```bash
npx skills add google-labs-code/stitch-skills --skill enhance-prompt -g -y
```

## 功能

- 增强 UI 描述的具体性和结构化程度
- 注入 UI/UX 关键词和设计系统上下文
- 添加氛围/情感描述（Vibe Design）
- 优化输出结构以获得更好的 Stitch 生成结果

## 使用场景

用户描述：「做一个视频编辑页面」

增强后：「创建一个深色主题的 AI 视频编辑工作台，左侧为素材面板（拖拽式），中间为视频预览区域（16:9 比例），底部为时间轴编辑器。使用 #ff317d 作为主操作色，圆角卡片式布局，毛玻璃效果面板背景。」

## 相关链接

- [GitHub](https://github.com/google-labs-code/stitch-skills)
