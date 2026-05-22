---
name: vue2-v-model
description: Vue 2 项目中编写「值包装型」自定义组件（例如 Selector、Input 包装、单位转换控件）时必须遵守的双向绑定实现模式。当编写、审查、重构 .vue 单文件组件并涉及 v-model / props 双向同步 / 表单校验联动时应用本规范。
---

# Vue 2 双向绑定实现规范

## 何时应用

编写或修改 **Vue 2** 单文件组件，且组件本身是对底层控件（input / select / radio-group / 自定义复合控件）的**包装**，需要：
- 暴露 `v-model` 给父组件
- 内部对值做格式转换（例如 `0.2 ↔ 20%`、`Number ↔ String`、单选 ↔ 多选数组等）
- 与外层 `el-form-item` 的校验规则联动

→ 必须严格遵守本规范的「九条 MUST + 七条 MUST NOT」。

如果只是无格式转换、无校验需求的单纯 props 透传（例如 `<el-button v-bind="$attrs" v-on="$listeners" />`），可以直接透传，不适用本规范。

---

## 必须遵守（MUST）

### M1. 显式覆盖 v-model 契约

```js
model: {
  prop: 'modelValue',
  event: 'change'
}
```

理由：
- 避开 Vue 2 默认的 `value` 关键字（容易跟 `:value` 属性混淆，且与 Vue 3 的 `modelValue` 命名前向兼容）
- 用 `change` 而不是 `input` 作为 v-model 同步事件，保证只有「真实变更」才会回流给父组件，避免 `input` 事件的高频触发

### M2. props 必须声明 `modelValue` 类型

```js
props: {
  modelValue: [Number, String]  // 按业务允许的类型枚举
}
```

不要省略类型 —— 否则 Vue devtools 推断不出类型，且没有运行期校验。

### M3. 如果是element-ui必须 `mixins: [Emitter]`

```js
import Emitter from 'element-ui/lib/mixins/emitter'

export default {
  mixins: [Emitter],
  // ...
}
```

理由：包装组件夹在 `el-form-item` 和真实控件之间时，`el-form-item` 无法直接监听到内部控件的 change 事件。必须借助 Emitter 的 `dispatch` 主动派发 `el.form.change` 事件，否则 `<el-form-item :rules="...">` 校验规则**完全不生效**。

### M4. 内部必须有镜像状态 `currentValue`

```js
data() {
  return {
    currentValue: this.transformIn(this.modelValue)  // 可含格式转换
  }
}
```

不允许直接把 `modelValue` 作为 template 的双向源（`:value` + `@input` 直接 emit），原因见 [MUST NOT N1](#n1-禁止裸双向)。

### M5. 必须双向 watch + judgeChange 防回环

```js
watch: {
  // 外部 → 内部：仅同步值，不主动触发校验
  modelValue: {
    deep: true,
    handler(e) {
      const { hasChange, newValue } = this.judgeChange(e, this.currentValue)
      if (hasChange) {
        this.currentValue = this.transformIn(newValue)
      }
    }
  },
  // 内部 → 外部：emit + dispatch 都由 doEmit 统一处理
  currentValue: {
    deep: true,
    handler(e) {
      this.$nextTick(() => {
        const { hasChange, newValue } = this.judgeChange(e, this.modelValue)
        if (hasChange) {
          this.doEmit(newValue)
        }
      })
    }
  }
}
```

`judgeChange(a, b)` 返回 `{ hasChange, newValue }`，由组件自行实现等值比较逻辑（数字、对象、数组各有不同判等方式）。

**为什么 `watch.modelValue` 不主动 dispatch 校验？**

外部 `modelValue` 变化的两种典型场景：
- **业务直接赋值**（如 `this.form.allocationMethod = 1`）：这与原生 el-input 被外部赋值后**不会自动触发校验**保持一致 —— element-ui 设计上认为「程序化赋值 ≠ 用户交互」，不应触发 form change 校验。
- **`form.resetFields()`**：element-ui 的 ElForm 内部会自己派发 `el.form.change` 并 clearValidate，包装组件无需重复 dispatch。

只有"用户主动交互产生的真实变更"才应该触发校验 —— 这一职责放在 `doEmit` 里（见 M7）。

### M6. 必须用 `judgeChange` 防止重复 emit

```js
judgeChange(a, b) {
  // 关键：先归一化（trim / 类型转换 / 单位转换），再比较
  const newValue = this.normalize(a)
  const compareValue = this.normalize(b)
  return {
    hasChange: !this.isAllSame(newValue, compareValue),
    newValue
  }
}
```

不做这一步，父更新 → 子 watch 触发 emit → 父再更新 → 死循环。

### M7. `doEmit` 必须同时 emit `input` 和 `change`，并 dispatch 表单校验

```js
doEmit(e) {
  const finalValue = this.transformOut(e)  // 格式转回外部约定
  this.$emit('input', finalValue)          // 给监听 @input 的消费者
  const { hasChange, newValue } = this.judgeChange(finalValue, this.modelValue)
  if (hasChange) {
    this.$emit('change', newValue)         // 给 v-model 和 el-form-item 校验
    this.$nextTick(() => {
      this.dispatch('ElFormItem', 'el.form.change', newValue)
    })
  }
}
```

- `input`：兼容主动监听 `@input` 的调用方（无变化也 emit，等同「用户做了交互」语义）
- `change`：v-model 同步事件（仅在真实变化时 emit）
- `dispatch('el.form.change')`：包装组件夹在 el-form-item 与底层控件之间，外层 ElFormItem 监听不到内部 emit('change')，必须主动 dispatch 才能触发 `:rules` 校验（见 M3 / N4）

> dispatch 必须在 `judgeChange.hasChange === true` 分支内 + `$nextTick` 内 —— 见下一条 M8。

### M8. 任何「watch 内部 → 对端 set / 外发」都必须包 `$nextTick`

```js
// ✅ doEmit 里 dispatch 校验
this.$nextTick(() => {
  this.dispatch('ElFormItem', 'el.form.change', newValue) // 如 element-ui 的表单校验示例
})

// ✅ watch.currentValue 内调 doEmit
currentValue(e) {
  this.$nextTick(() => {
    if (hasChange) this.doEmit(newValue)
  })
}
```

理由：watch 回调内同 tick 触发对端 set（emit / dispatch / 修改对端值）会让 Vue 的 watcher 嵌套执行，触发顺序不可控；尤其 dispatch `el.form.change` 会唤起 async-validator，可能拿到尚未稳定的 `currentValue`。`$nextTick` 把对端响应推到下一帧，保证状态先稳定再传播。

### M9. 模板优先用 `v-model` 直绑 `currentValue`

```vue
<!-- ✅ 推荐：v-model 直绑 -->
<el-radio-group v-bind="$attrs" v-model="currentValue">
  ...
</el-radio-group>
```

```vue
<!-- ⚠️ 次选：手工 :value + @input -->
<el-radio-group v-bind="$attrs" :value="currentValue" @input="onInputDirect" />
```

```vue
<!-- ❌ 禁止：用 @change 同步 currentValue（见 N7） -->
<el-radio-group v-bind="$attrs" :value="currentValue" @change="onChangeDirect" />
```

为什么优先 v-model 直绑：
- `v-model="currentValue"` 编译为 `:value="currentValue" + @input="currentValue = $event"`，element-ui 控件 `emit('input', newValue)` 时会**立刻把新值写回 `currentValue`**，无需手写 `onInputDirect` / `onChangeDirect`
- 完全绕开 element-ui 部分控件 `@change` 事件**回传 prop 旧值**的坑（见 N7）
- 模板更简洁，业务交互链路只剩「`v-model` 写入 currentValue → `watch.currentValue` → `doEmit`」一条线

只有当底层控件**没有提供 v-model 兼容的 input event**（极少见），才退回到手工 `:value + @input` 模式。

---

## 严格禁止（MUST NOT）

### N1. 禁止裸双向

```vue
<!-- ❌ 错误 -->
<template>
  <el-input :value="modelValue" @input="$emit('input', $event)" />
</template>
```

为什么错：
- 无法做格式转换
- 无法防止重复 emit
- 无法跟 el-form-item 校验联动
- 不支持 `model: { event: 'change' }` 契约

### N2. 禁止直接 mutate prop

```js
// ❌ 错误
methods: {
  onClick() {
    this.modelValue = 'new'  // Vue 警告: Avoid mutating a prop directly
  }
}
```

正确做法：通过 `doEmit(newValue)` 让父组件自己更新。

### N3. 禁止省略 judgeChange 直接 emit

```js
// ❌ 错误：会陷入 emit ↔ watch 死循环
watch: {
  currentValue(e) {
    this.$emit('change', e)
  }
}
```

### N4. 禁止省略 Emitter mixin

只要组件被 `el-form-item` 包裹，且业务方写了 `:rules`，就**必须** mixin Emitter 并在合适时机 `dispatch('ElFormItem', 'el.form.change', value)`。否则校验**静默失效**，业务方排查极困难。

### N5. 禁止在同一 tick 内反向 set

```js
// ❌ 错误
watch: {
  modelValue(e) {
    this.currentValue = e  // 同步赋值，触发 currentValue watch，再次 emit
  }
}
```

必须 `$nextTick` 包裹任何「watch 内部 → 对端 set」的赋值。

### N6. 禁止把 v-model 事件名写错

```js
model: {
  prop: 'modelValue',
  event: 'change'  // ← 业务约定
}

// ❌ 错误：父组件 v-model 完全不工作
methods: {
  onChange(e) {
    this.$emit('input', e)   // 只 emit input，但 v-model 在听 change
  }
}
```

只 emit `input` 不 emit `change`，等同于禁用 v-model。必须两个都 emit（见 M7）。

### N7. 禁止用 `:value + @change` 包装 element-ui 的 `el-radio-group` / `el-checkbox-group`

```vue
<!-- ❌ 错误：el-radio-group 的 @change 会回传 prop 旧值，currentValue 永远写不进去 -->
<el-radio-group :value="currentValue" @change="onChangeDirect" />
```

```js
// ❌ 拿到的 e 是 prop 旧值（如 ''），不是用户点击的新值（如 0）
onChangeDirect(e) {
  this.currentValue = e   // 把 currentValue 又复位回旧值
}
```

**为什么错**：element-ui 2.15.x 的 `ElRadioGroup` 实现里：

```js
// element-ui packages/radio/src/radio-group.vue
created() {
  this.$on('handleChange', value => {
    this.$emit('change', this.value)   // 注意：this.value 是 prop value，不是参数 value
  })
}
```

子 radio-button 的 click 触发链路：
1. input v-model setter → emit `'input', 0`（**新值**）
2. dispatch `'handleChange'` → group emit `'change', this.value`（**prop 旧值** `''`）

如果你只监听 `@change`，会拿到 prop 旧值，把 `currentValue` 反向复位回去，视觉上看起来"radio 点了又取消"。`el-checkbox-group` 有相同实现。

**正确做法**：用 M9 的 `v-model` 直绑（首选）或 `:value + @input`（次选）。

---

## 推荐做法（SHOULD）

### S1. 提供 `last-blur` 事件（可选）

包装 input 类组件时，可对外暴露 `@last-blur` 事件，把当前值连同原生 blur 事件一起回传，方便业务方做「失焦时调接口」之类的扩展：

```js
handleBlur(evt) {
  this.$emit('last-blur', [this.currentValue, evt])
  this.doEmit(this.currentValue)
}
```

### S2. `inheritAttrs: false` + `v-bind="$attrs"`

```js
inheritAttrs: false  // 防止 attrs 同时落到根元素和透传目标上
```

```vue
<el-radio-group v-bind="$attrs" v-model="currentValue">
```

### S3. props 类型用数组写法 `[Number, String]`，不要写联合类型字符串

---

## 标准模板（拷贝即用）

### 模板 A：`v-model` 直绑（首选）

适用于绝大多数 element-ui 输入控件（el-input / el-select / el-radio-group / el-checkbox-group / el-switch / el-input-number 等）。

```vue
<template>
  <el-radio-group v-bind="$attrs" v-model="currentValue">
    <!-- ... options ... -->
  </el-radio-group>
</template>

<script>
import Emitter from 'element-ui/lib/mixins/emitter'

export default {
  name: 'XxxSelector',
  mixins: [Emitter],
  inheritAttrs: false,
  model: {
    prop: 'modelValue',
    event: 'change'
  },
  props: {
    modelValue: [Number, String]
  },
  data() {
    return {
      currentValue: this.transformIn(this.modelValue)
    }
  },
  watch: {
    modelValue: {
      deep: true,
      handler(e) {
        const { hasChange, newValue } = this.judgeChange(e, this.currentValue)
        if (hasChange) {
          this.currentValue = this.transformIn(newValue)
        }
      }
    },
    currentValue: {
      deep: true,
      handler(e) {
        this.$nextTick(() => {
          const { hasChange, newValue } = this.judgeChange(e, this.modelValue)
          if (hasChange) {
            this.doEmit(newValue)
          }
        })
      }
    }
  },
  methods: {
    transformIn(e) {
      // 外部值 → 内部展示值；无转换需求直接 return e
      return e
    },
    transformOut(e) {
      // 内部值 → 外部 modelValue；无转换需求直接 return e
      return e
    },
    judgeChange(a, b) {
      const newValue = this.transformOut(a)
      const compareValue = this.transformOut(b)
      return {
        hasChange: newValue !== compareValue,
        newValue
      }
    },
    doEmit(e) {
      const finalValue = this.transformOut(e)
      this.$emit('input', finalValue)
      const { hasChange, newValue } = this.judgeChange(finalValue, this.modelValue)
      if (hasChange) {
        this.$emit('change', newValue)
        this.$nextTick(() => {
          this.dispatch('ElFormItem', 'el.form.change', newValue)
        })
      }
    }
  }
}
</script>
```

### 模板 B：`:value + @input/@change` 手工同步（次选，需要监听 blur 等额外事件时使用）

```vue
<template>
  <el-input
    v-bind="$attrs"
    :value="currentValue"
    @input="onInputDirect"
    @change="onChangeDirect"
    @blur="handleBlur"
  />
</template>
```

```js
methods: {
  // 仅在 v-model 直绑无法满足时才用（例如需要 blur 拿到值）
  onInputDirect(e) {
    this.currentValue = e
  },
  onChangeDirect(e) {
    this.currentValue = e
  },
  handleBlur(evt) {
    this.$emit('last-blur', [this.currentValue, evt])
    this.doEmit(this.currentValue)
  }
  // ... 其余 transformIn / transformOut / judgeChange / doEmit 同模板 A
}
```

> ⚠️ 模板 B 用于 `el-input` 这种 `@change` 事件本身就传递新值的控件。**不要**用模板 B 包装 `el-radio-group` / `el-checkbox-group`（见 N7）。

---

## 自检清单（提交前对照）

提交一个新写的「值包装型」组件前，逐条对照：

- [ ] M1: 写了 `model: { prop: 'modelValue', event: 'change' }`
- [ ] M2: props 里声明了 `modelValue` 及类型
- [ ] M3: `mixins: [Emitter]`
- [ ] M4: 有内部镜像 `currentValue`
- [ ] M5: 双向 watch（modelValue 仅同步值，currentValue → doEmit 在 `$nextTick` 内）
- [ ] M6: 有 `judgeChange` 防回环
- [ ] M7: `doEmit` 同时 emit `input` + `change` + dispatch `el.form.change`
- [ ] M8: watch 内对端 set / dispatch 都包了 `$nextTick`
- [ ] M9: 模板优先用 `v-model="currentValue"` 直绑（element-ui 控件场景）
- [ ] N1-N7: 全部没踩

---

新组件写完后建议跟以上范例对照一遍 watch/method 结构。
