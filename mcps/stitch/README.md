# MCP: Google Stitch

> Google Labs 推出的 AI 原生 UI 设计工具，通过自然语言生成生产级 UI 设计和代码。

## 基本信息

| 项目 | 值 |
|------|-----|
| npm 包 | `@_davideast/stitch-mcp` |
| 版本 | 0.5.3 |
| 官方仓库 | https://github.com/davideast/stitch-mcp |
| 官方网站 | https://stitch.withgoogle.com |
| 官方文档 | https://stitch.withgoogle.com/docs/learn/overview/ |
| Node.js 要求 | >= 18 |

## 安装与配置

### 初始化认证

```bash
npx @_davideast/stitch-mcp init
```

### Cursor MCP 配置

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "@_davideast/stitch-mcp@latest", "proxy"]
    }
  }
}
```

### 前置条件

- Google Cloud 项目（需开启计费）
- stitch.withgoogle.com 上至少有一个项目
- gcloud CLI 已安装并认证

### 验证安装

```bash
npx @_davideast/stitch-mcp doctor --verbose
```

## 提供的 Tools

| Tool 名称 | 描述 |
|-----------|------|
| `build_site` | 将屏幕映射为路由，获取每个页面的 HTML |
| `get_screen_code` | 获取特定屏幕的原始 HTML/CSS |
| `get_screen_image` | 获取屏幕截图（base64） |
| `generate_screen_from_text` | 从文本描述生成屏幕 |
| `list_projects` | 列出所有 Stitch 项目 |
| `extract_design_context` | 从 URL 提取设计上下文 |

## 代码导出格式

支持 7 种框架：HTML、CSS、Tailwind CSS、Vue.js、Angular、Flutter、SwiftUI

## 核心功能

- **多屏幕生成**：一次描述生成最多 5 个关联屏幕
- **DESIGN.md**：导出/导入机器可读的设计系统
- **AI 无限画布**：智能布局和设计迭代
- **模型选择**：Gemini 2.5 Pro（高质量）/ Flash（快速迭代）
