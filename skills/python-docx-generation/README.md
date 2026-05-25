# python-docx 文档生成 Skill

## 概述

用 python-docx 批量生成或修改 Word 文档时的字体、行距、页眉页脚、封面页最佳实践，专门解决中文排版问题。

## 安装

```bash
pip install python-docx
```

## 触发条件

- 用 Python 从 Markdown / 结构化数据生成 .docx
- 批量修改 docx 字体/样式
- 遇到字扁、字上字下、页码不正确等问题

## 核心文档

见 [SKILL.md](./SKILL.md)

## 关键 API

| 问题 | 解决方案 |
|---|---|
| 中文字扁/fallback | 设 `w:eastAsia` + `w:cs` |
| 字上字下 | 同步 `w:sz` 和 `w:szCs` |
| 封面隔离 | `nextPage` 分节符 |
| 页码 | `PAGE` / `NUMPAGES` 域代码 |

## 沉淀来源

2026-05-25 李井法仲裁项目，批量生成 26 份法律辅助材料 docx 过程中总结。
