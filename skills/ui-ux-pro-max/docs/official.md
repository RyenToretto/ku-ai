# UI/UX Pro Max — 官方资源

## 核心链接

| 资源 | 链接 |
|------|------|
| GitHub 仓库 | https://github.com/nextlevelbuilder/ui-ux-pro-max-skill |
| 官方网站 | https://ui-ux-pro-max-skill.com |
| CLI 文档 | https://ui-ux-pro-max-skill.com/docs/cli-reference/ |
| npm（CLI） | https://www.npmjs.com/package/uipro-cli |

## 项目元信息

| 字段 | 值 |
|------|-----|
| 主语言 | Python |
| 许可证 | **CC-BY-NC-4.0（非商业授权）** |
| Stars | 88K+ |
| 版本 | v2.0 |

## 许可证详细说明

CC-BY-NC-4.0 = Creative Commons 署名-非商业性使用 4.0：

- ✅ 允许：个人学习、学术研究、非营利项目、开源项目展示
- ❌ 禁止：任何以营利为目的的商业使用（包括商业产品的 UI 生成）
- 必须：保留作者署名

商业使用需联系作者获得额外授权。

## 搜索脚本（高级用法）

v2.0 提供 Python 脚本用于生成设计系统：

```bash
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "<查询词>" --design-system -p "<项目名>"
```

例如：
```bash
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "SaaS dashboard" --design-system -p "MyApp"
```

## 版本管理

```bash
# 查看可用版本
uipro versions

# 更新到最新版
uipro update

# 固定版本（离线/团队一致性）
uipro init --offline
```

## 搜索域（v2 数据库）

| 域 | 用途 | 示例关键词 |
|------|------|------|
| `product` | 产品类型推荐 | SaaS, e-commerce, portfolio, healthcare |
| `style` | UI 风格、色彩、效果 | glassmorphism, minimalism, dark mode, brutalism |
| `typography` | 字体搭配 | elegant, playful, professional, modern |
| `color` | 按产品类型的色板 | saas, ecommerce, healthcare, beauty, fintech |
| `landing` | 页面结构、CTA 策略 | hero, testimonial, pricing, social-proof |
| `chart` | 图表类型和库推荐 | trend, comparison, timeline, funnel, pie |
| `ux` | UX 最佳实践和反模式 | animation, accessibility, z-index, loading |
