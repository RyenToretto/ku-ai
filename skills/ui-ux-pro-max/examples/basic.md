# UI/UX Pro Max — 使用示例

## 安装

```bash
# 安装 CLI
npm install -g uipro-cli

# 安装到 Cursor
uipro init --ai cursor

# 安装到 Claude Code
uipro init --ai claude
```

## 自然语言调用（自动触发）

```
构建一个 SaaS 定价页面，有三档套餐
```

```
设计一个医疗健康应用的 Dashboard，展示患者健康数据
```

```
为金融科技产品创建一个登录页，需要体现专业感和安全感
```

## 指定设计风格

```
用玻璃态（glassmorphism）风格创建一个产品展示卡片组件
```

```
用极简主义风格（类似 Linear）构建任务管理 App 的主界面
```

```
参考暗色模式 + Bento Grid 布局设计一个 AI 产品落地页
```

## 使用推理规则生成设计系统

```
我在构建一个医疗预约平台，帮我生成一套完整的设计系统
```

Agent 会自动：
1. 识别产品类型（医疗）
2. 应用对应的 161 条推理规则
3. 推荐色板：医疗产品通常使用蓝色/绿色系，传达信任和健康
4. 推荐字体：易读性优先
5. 生成完整设计令牌

## Python 脚本高级查询

```bash
# 查询 SaaS 类产品的设计系统建议
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "SaaS project management" --design-system -p "TaskFlow"

# 查询电商产品的色板
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "ecommerce fashion" --domain color
```

## 斜杠命令调用（Kiro、GitHub Copilot 等）

```
/ui-ux-pro-max 为旅游预订平台构建搜索结果页面
```

## UX 审查

```
检查这个表单设计是否符合无障碍和 UX 最佳实践
```

```
这个 Dashboard 有哪些 UX 反模式？
```
