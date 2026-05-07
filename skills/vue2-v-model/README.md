# Skill: vue2-v-model

> Vue 2「值包装型」自定义组件双向绑定实现规范。涵盖 v-model 契约、props 镜像、双向 watch + 防回环、与 element-ui `el-form-item` 的校验联动。

## 基本信息

| 项目 | 值 |
|------|-----|
| Skill 名称 | `vue2-v-model` |
| 来源 | 项目内沉淀 |
| 安装位置 | `~/.agents/skills/vue2-v-model/` |
| 触发条件 | 编写或修改 Vue 2 单文件组件，且组件本身是对底层控件（input / select / radio-group / 复合控件）的包装，需要暴露 v-model |
| 依赖 | 项目使用 Vue 2 + element-ui（如未使用 element-ui，N4/M3 等 form 校验联动条款可豁免） |

## 核心要求

九条 MUST + 七条 MUST NOT，关键点：

| 编号 | 内容 |
|------|------|
| **M1** | `model: { prop: 'modelValue', event: 'change' }` 显式契约 |
| **M3** | `mixins: [Emitter]`（element-ui 项目必须） |
| **M4** | 内部镜像状态 `currentValue` |
| **M5** | 双向 watch（modelValue 仅同步值；currentValue → doEmit 在 `$nextTick` 内） |
| **M7** | `doEmit` 同时 emit `input` + `change`，并 dispatch `el.form.change`（仅在 `judgeChange.hasChange === true` 分支） |
| **M9** | 模板优先 `v-model="currentValue"` 直绑（绕开 element-ui `@change` 旧值陷阱） |
| **N7** | 禁止用 `:value + @change` 包装 `el-radio-group` / `el-checkbox-group` |

## 适用场景

- 写新的 Selector / Input 包装 / 单位转换控件
- 重构旧的「值包装型」组件
- 排查 v-model 同步异常 / form 校验静默失效
- 排查 element-ui radio-group / checkbox-group 的"点击不切换"问题

## 不适用场景

- 无格式转换、无校验联动的纯 props 透传（如 `<el-button v-bind="$attrs" v-on="$listeners" />`）
- Vue 3 / Composition API（请参考 Vue 3 v-model 规范，本 skill 不覆盖）

## 变更历史

- **2026-05-07**：根据 `PlayletWaySelector.vue` 实战经验调整：
  - M9 重定义为「模板优先用 v-model 直绑 currentValue」
  - dispatch `el.form.change` 从 `watch.modelValue` 移到 `doEmit`（M5 / M7 联动调整）
  - 新增 N7：禁止用 `:value + @change` 包装 el-radio-group / el-checkbox-group（element-ui 实现细节坑）

## 引用

完整规范见同目录 [`SKILL.md`](./SKILL.md)。
