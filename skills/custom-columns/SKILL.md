---
name: custom-columns
description: 在本项目中为 table 页面实现自定义列能力。当用户要给一个新页面接入自定义列、新增/修改列 schema、迁移旧 tableControl 页面、或排查自定义列问题时使用。涵盖 schema 定义、useSchemaColumnConfig mixin 接入、模板写法（扁平/嵌套）、版本管理、cellComponent、以及与 DoTableHeader/DoConfigColumnDialog 的兼容方式。
---

# 自定义列实现 Skill

本项目自定义列的目标架构是 **schema + v-for**，通过 `useSchemaColumnConfig` mixin 驱动。
每次实现前请先读取 `docs/custom-columns/ADR.md` 了解架构决策背景。

## 使用场景

- 新 table 页面需要接入自定义列
- 已有页面需要新增/修改可配置列
- 从 `tableControl` 迁移到 `useSchemaColumnConfig`
- 排查列配置、版本兼容、嵌套表头相关问题

---

## 实现流程

### Step 1：创建列 schema 文件

在页面模块目录下新建 `*ColumnSchemas.js`，与页面组件同级。

```js
// src/modules/xxx/_module/xxxColumnSchemas.js

export const XXX_COLUMN_SCHEMAS = [
  // ── 扁平叶子列 ──────────────────────────────────────────
  {
    prop: 'cost',           // 必须：唯一稳定标识符，用于配置存储
    label: '成本',
    minWidth: 110,
    align: 'right',
    renderType: 'float',
    renderArgs: [2, true],  // [小数位数, 零值显示为'-']
    group: '效果指标'        // 弹窗分组展示
  },
  {
    prop: 'rate',
    label: '转化率',
    minWidth: 100,
    align: 'right',
    renderType: 'percent',
    renderArgs: [1, true],
    group: '效果指标'
  },
  // ── 嵌套表头（父节点不设 prop，只设 label + children）───
  {
    label: '转化层',
    group: '转化指标',
    children: [
      { prop: 'install', label: '激活', renderType: 'integer', group: '转化指标' },
      { prop: 'activate', label: '注册', renderType: 'integer', group: '转化指标' }
    ]
  }
]
```

**schema 字段速查：**

| 字段 | 必须 | 说明 |
|---|---|---|
| `prop` | 叶子必须 | 唯一稳定标识，用于配置存储和 v-for key |
| `label` | 必须 | 列头文字 |
| `group` | 推荐 | 弹窗左侧分组名 |
| `renderType` | 推荐 | `text`/`integer`/`float`/`percent`/`roi-link` |
| `renderArgs` | 可选 | float `[digits, noZero]`，percent `[digits, noZero]` |
| `minWidth` | 推荐 | px 数字 |
| `align` | 可选 | `left`/`right`/`center` |
| `sortable` | 可选 | `true` → `sortable="custom"` |
| `fixed` | 可选 | `true`/`'left'`/`'right'` |
| `elAttrs` | 可选 | 透传任意 el-table-column 原生属性 |
| `cellComponent` | 可选 | 自定义单元格 Vue 组件（详见下方） |
| `renderHeader` | 可选 | 自定义列头 render 函数 `(h, {column}) => VNode` |
| `showOverflowTooltip` | 可选 | boolean |
| `children` | 嵌套父节点 | 嵌套子列数组，有此字段则为父节点 |
| `alwaysVisible` | 可选 | true 则不纳入可配置范围（应直接写在模板外） |

---

### Step 2：页面组件接入 mixin

```js
import useSchemaColumnConfig from '@/mixins/useSchemaColumnConfig'
import { XXX_COLUMN_SCHEMAS } from './ xxxColumnSchemas'

export default {
  name: 'XxxPage',
  mixins: [useSchemaColumnConfig],
  // 不能再 mixins: [tableControl] — 两者不兼容
  data() {
    return {
      loading: false,
      tableData: [],

      // ── useSchemaColumnConfig 必须提供的三个字段 ──
      columnSchemas: XXX_COLUMN_SCHEMAS,
      schemaStorageKey: 'xxx_module_col',   // 全局唯一，不得与其他页面重复
      schemaVersion: 'sv1',                 // prop rename/delete 时递增为 sv2…
    }
  }
}
```

---

### Step 3：模板写法

#### 3a. 扁平列（无嵌套）

```html
<table-wrap v-loading="loading" enable-do-header :disabled-column-config="false">
  <el-table
    :key="tableRenderKey"
    class="do-inner-scroller page-table"
    :data="tableData"
    border
    stripe
  >
    <!-- 固定列：不受配置控制，写在 v-for 外 -->
    <el-table-column prop="id"   label="ID"   fixed="left" width="60" align="center" />
    <el-table-column prop="name" label="名称" fixed="left" min-width="120" />

    <!-- 可配置列：由 visibleSchemas 驱动，刷新时只 mount 配置中的列 -->
    <el-table-column
      v-for="schema in visibleSchemas"
      :key="schema.prop"
      :prop="schema.prop"
      :label="schema.label"
      :min-width="schema.minWidth || undefined"
      :align="schema.align || 'left'"
      :sortable="schema.sortable ? 'custom' : false"
      :render-header="schema.renderHeader || undefined"
      v-bind="schema.elAttrs || {}"
    >
      <template #default="{ row, column, $index }">
        <!-- 自定义组件（cellComponent 方式） -->
        <component
          :is="schema.cellComponent"
          v-if="schema.cellComponent"
          :row="row"
          :column="column"
          :index="$index"
          :schema="schema"
        />
        <!-- 内建 renderType 渲染 -->
        <div
          v-else
          :class="['float','percent','integer','roi-link'].includes(schema.renderType) ? 'mono' : 'name'"
        >
          {{ formatSchemaCell(row[column.property], schema) }}
        </div>
      </template>
    </el-table-column>
  </el-table>
</table-wrap>
```

#### 3b. 嵌套表头（schema 有 children）

```html
<!-- 额外引入递归组件 -->
<!-- import SchemaColumn from '@/modules/_example/customColumns/_components/SchemaColumn.vue' -->

<table-wrap v-loading="loading" enable-do-header :disabled-column-config="false">
  <el-table :key="tableRenderKey" :data="tableData" border stripe>
    <el-table-column prop="name" label="名称" fixed="left" min-width="120" />

    <!-- 用 SchemaColumn 替换 el-table-column v-for 块 -->
    <schema-column
      v-for="schema in visibleSchemas"
      :key="schema.prop || schema.label"
      :schema="schema"
      :format-cell="formatSchemaCell"
    />
  </el-table>
</table-wrap>
```

> `SchemaColumn` 自动递归处理 children，父节点消失逻辑由 `_pruneVisibleTree` 处理（取消所有子列后父列自动不渲染）。

---

### Step 4：cellComponent — 自定义单元格

三种写法，按推荐度排序：

**方式 A：外部 .vue SFC（推荐，有 scoped style 和 IDE 跳转）**

```js
// cells/MyCellComponent.vue
export default {
  name: 'MyCellComponent',
  props: ['row', 'column', 'index', 'schema'],
}
// schema 中设置：
{ prop: 'cost', cellComponent: MyCellComponent, ... }
```

**方式 C：render 函数（无运行时编译，推荐次选）**

```js
const MyCell = {
  props: ['row', 'column', 'index', 'schema'],
  render(h) {
    const val = this.row[this.column.property]
    return h('div', { class: 'mono' }, val != null ? val.toFixed(2) : '-')
  }
}
```

**方式 B：模板字符串（需完整 Vue 构建，仅 demo 用）**

```js
const MyCell = {
  props: ['row', 'column', 'schema'],
  template: `<span>{{ row[column.property] || '-' }}</span>`
}
```

---

### Step 5：版本管理

```
字段新增  → 不需要递增 schemaVersion（新 prop 静默忽略）
字段删除  → 必须递增 schemaVersion（sv1 → sv2）
字段重命名 → 必须递增 schemaVersion
```

版本递增后，旧的 localStorage key 自动失效，用户配置以默认配置重新初始化。
无需手动清理旧数据。

---

## 常见问题排查

| 现象 | 原因 | 解法 |
|---|---|---|
| 列顺序变了但表格不更新 | `el-table` 缺少 `:key="tableRenderKey"` | 补上绑定 |
| 两个页面配置互相覆盖 | `schemaStorageKey` 重复 | 改为唯一 key |
| 弹窗「已选列」为空 | 默认配置 columns 为空 | `getDefaultConfig()` 已兜底，确认 `_initSchemaStorage` 已执行 |
| 嵌套父节点不消失 | schema 未用 `_pruneVisibleTree` 路径 | 检查 `visibleSchemas` 计算 —— 有 children 时自动走嵌套路径 |
| 旧缓存废弃字段报错 | 不应该报错 | `_isSchemaVisible` 容忍未知 prop，不崩溃 |
| 刷新后先渲染全量列 | 混用了 `tableControl`（旧方案） | 迁移到 `useSchemaColumnConfig` |

---

## 参考文件

- Mixin 实现：`src/mixins/useSchemaColumnConfig.js`
- 递归组件：`src/modules/_example/customColumns/_components/SchemaColumn.vue`
- Demo 场景（01~07）：`src/modules/_example/customColumns/`
- 真实接入案例：`src/modules/oceanAdv/promotion/_module/AccountList.vue`
- 真实接入案例：`src/modules/oceanAdv/monitor/roi/hour/_module/HourReportData.vue`
- 架构决策：`docs/custom-columns/ADR.md`
- 任务追踪：`docs/custom-columns/TODO.md`
