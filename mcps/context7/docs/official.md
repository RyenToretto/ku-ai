# Context7 官方文档

## 文档链接

- 官方网站: https://context7.com
- GitHub: https://github.com/upstash/context7
- npm: https://www.npmjs.com/package/@upstash/context7-mcp

## 核心概念

Context7 维护了一个实时更新的文档索引，覆盖数千个流行的库和框架。通过 MCP 接口，AI Agent 可以直接查询最新文档内容，无需访问互联网。

## 工作原理

1. `resolve-library-id` — 将人类可读的库名（如 "nuxt"）解析为 Context7 内部 ID
2. `get-library-docs` — 根据 ID 返回文档内容，支持按话题/功能过滤

## 支持的库

覆盖主流前端/后端框架、数据库、云服务 SDK 等，包括但不限于：
Vue, Nuxt, React, Next.js, Tailwind CSS, Prisma, Express, Django, Spring Boot, AWS SDK 等。
