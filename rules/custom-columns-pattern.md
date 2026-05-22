# custom-columns-pattern 规则说明

## 设计意图

在 `xuanhu-ai` 项目中，自定义列经历了一次完整的架构治理：从直接操作 Element UI Table 内部 store（`tableControl` mixin）迁移到 schema + v-for 驱动的 `useSchemaColumnConfig` mixin。

本规则（`custom-columns-pattern.mdc`）的目标是：
- 防止新页面重蹈旧路，继续使用 `tableControl` mixin 接入自定义列
- 在 agent 工作于相关文件时自动注入开发模式约束和接入示例
- 提供 schema 字段、renderType、cellComponent、版本管理的快速参考

## 适用范围

- `src/modules/**/*.vue`：table 页面
- `src/**/*ColumnSchemas.js`：列 schema 定义文件
- `src/mixins/useSchemaColumnConfig.js`：mixin 本体

## 关联文档

- ADR：`docs/custom-columns/ADR.md`（架构决策）
- ADAPTER 设计：`docs/custom-columns/ADAPTER-DESIGN.md`
- TODO：`docs/custom-columns/TODO.md`
- Demo 页面：`src/modules/_example/customColumns/`
- 真实案例：`AccountList.vue`、`HourReportData.vue`

## 关联 Skill

`custom-columns` skill（见 `skills/custom-columns/SKILL.md`）是本规则的完整实现指南，包含逐步接入流程、嵌套表头写法、cellComponent 三种方式、版本管理规则和常见问题排查。

## 历史背景

2026-05 完成架构治理：
1. `ElementTableColumnAdapter`（`src/mixins/elementTableColumnAdapter.js`）作为过渡层，收敛所有 Element UI Table 内部 store 访问
2. `useSchemaColumnConfig`（`src/mixins/useSchemaColumnConfig.js`）作为目标架构 mixin
3. `AccountList.vue`、`HourReportData.vue` 完成迁移，通过全量回归验证
4. 7 个 Demo 场景页面（`src/modules/_example/customColumns/`）提供参考
