# Vue 3编码规范

> 本规范定义Vue 3的编码规范，确保前端代码质量和一致性。

---

## 📋 目录

- [组件定义规范](#组件定义规范)
- [Composition API使用](#composition-api使用)
- [Script Setup](#script-setup)
- [生命周期](#生命周期)
- [指令使用](#指令使用)
- [事件处理](#事件处理)
- [样式规范](#样式规范)

---

## 组件定义规范

### 基本结构

**使用Vue 3 + TypeScript + Script Setup**：

```vue
<template>
  <div class="user-component">
    <h2>{{ title }}</h2>
    <p>{{ description }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

// Props定义
interface Props {
  title: string
  description?: string
}

const props = defineProps<Props>()

// 响应式数据
const count = ref(0)

// 方法
const increment = () => {
  count.value++
}
</script>

<style scoped>
.user-component {
  /* scoped样式 */
}
</style>
```

### 组件命名

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| 单文件组件 | PascalCase | `UserComponent.vue`、`UserProfile.vue` |
| 组件注册 | PascalCase | `UserComponent`、`UserProfile` |

**❌ 错误**：
```vue
<!-- user-component.vue -->  <!-- kebab-case -->
```

**✅ 正确**：
```vue
<!-- UserComponent.vue -->  <!-- PascalCase -->
```

---

## Composition API使用

### ref vs reactive

**ref**：用于基本类型和需要替换整个对象的情况

```vue
<script setup lang="ts">
import { ref } from 'vue'

// 基本类型
const count = ref(0)
const message = ref('Hello')

// 对象
const user = ref<User>({
  id: 1,
  username: 'admin'
})

// 修改
count.value++
user.value.username = 'newname'
</script>
```

**reactive**：用于对象类型，不需要替换整个对象的情况

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// 对象
const state = reactive({
  count: 0,
  user: {
    id: 1,
    username: 'admin'
  }
})

// 修改
state.count++
state.user.username = 'newname'
</script>
```

### computed

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const count = ref(0)
const doubled = computed(() => count.value * 2)

// 只读computed
const countText = computed(() => `Count: ${count.value}`)
</script>
```

### watch

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const count = ref(0)
const message = ref('Hello')

// 简单watch
watch(count, (newValue, oldValue) => {
  console.log(`count changed from ${oldValue} to ${newValue}`)
})

// 立即执行
watch(message, (newValue) => {
  console.log(`message changed to ${newValue}`)
}, { immediate: true })

// 监听多个源
watch(
  [count, message],
  ([newCount, newMessage], [oldCount, oldMessage]) => {
    console.log(`count: ${oldCount} → ${newCount}, message: ${oldMessage} → ${newMessage}`)
  }
)

// 深度监听
const user = ref({ name: 'John', address: { city: 'NYC' } })

watch(
  user,
  (newValue) => {
    console.log(`address.city changed to ${newValue.address.city}`)
  },
  { deep: true }
)
</script>
```

---

## Script Setup

### 基本用法

```vue
<script setup lang="ts">
// 导入
import { ref, computed, onMounted } from 'vue'

// 定义响应式数据
const message = ref('Hello')

// 定义计算属性
const reversed = computed(() => message.value.split('').reverse().join(''))

// 定义方法
const changeMessage = (newMessage: string) => {
  message.value = newMessage
}

// 生命周期
onMounted(() => {
  console.log('Component mounted')
})

// 暴露给模板
defineExpose({
  message,
  changeMessage
})
</script>
```

### Props定义

```vue
<script setup lang="ts">
// 基本类型
interface Props {
  title: string
  count?: number
}

const props = defineProps<Props>()

// 使用
console.log(props.title)
</script>
```

### Emits定义

```vue
<script setup lang="ts">
// 定义emits
const emit = defineEmits<{
  'update:title': [value: string]
  'submit': [payload: SubmitPayload]
}>()

// 触发事件
const handleSubmit = () => {
  emit('update:title', 'new title')
  emit('submit', { data: 'test' })
}
</script>
```

---

## 生命周期

### Composition API生命周期

| Vue 2 | Vue 3 (Composition API) | 说明 |
|-------|----------------------|------|
| `beforeCreate` | ❌ 不需要 | Composition API没有beforeCreate |
| `created` | ❌ 不需要 | 在setup()中初始化 |
| `beforeMount` | ❌ 不需要 | 使用onBeforeMount |
| `mounted` | `onMounted` | 组件挂载后 |
| `beforeUpdate` | `onBeforeUpdate` | 组件更新前 |
| `updated` | `onUpdated` | 组件更新后 |
| `beforeUnmount` | `onBeforeUnmount` | 组件卸载前 |
| `unmounted` | `onUnmounted` | 组件卸载后 |

### 示例

```vue
<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

let timer: number | null = null

onMounted(() => {
  console.log('Component mounted')
  timer = setInterval(() => {
    console.log('Timer tick')
  }, 1000)
})

onUnmounted(() => {
  console.log('Component unmounted')
  if (timer) {
    clearInterval(timer)
  }
})
</script>
```

---

## 指令使用

### v-if vs v-show

```vue
<template>
  <!-- v-if：条件渲染，完全移除/添加DOM -->
  <div v-if="isVisible">Visible content</div>
  <div v-else>Hidden content</div>

  <!-- v-show：CSS display切换，保留DOM -->
  <div v-show="isVisible">Toggle visible</div>
</template>
```

| 指令 | 适用场景 |
|------|---------|
| `v-if` | 条件渲染，初始不渲染 |
| `v-show` | 频繁切换显示/隐藏 |

### v-for

```vue
<template>
  <!-- 数组遍历 -->
  <div v-for="user in users" :key="user.id">
    {{ user.name }}
  </div>

  <!-- 对象遍历 -->
  <div v-for="(value, key) in user" :key="key">
    {{ key }}: {{ value }}
  </div>

  <!-- 带索引 -->
  <div v-for="(user, index) in users" :key="user.id">
    {{ index }}: {{ user.name }}
  </div>
</template>
```

**⚠️ 必须使用key**：
```vue
<!-- ❌ 错误：没有key -->
<div v-for="user in users">
  {{ user.name }}
</div>

<!-- ✅ 正确：使用唯一key -->
<div v-for="user in users" :key="user.id">
  {{ user.name }}
</div>
```

### v-bind和v-on缩写

```vue
<template>
  <!-- v-bind缩写 -->
  <input :value="message" @input="message = $event.target.value">

  <!-- v-on缩写 -->
  <button @click="handleClick">Click me</button>

  <!-- 修饰符 -->
  <form @submit.prevent="handleSubmit">
    <input @keyup.enter="handleEnter">
  </form>
</template>
```

### 常用修饰符

| 修饰符 | 说明 |
|--------|------|
| `.prevent` | 阻止默认事件 |
| `.stop` | 阻止事件冒泡 |
| `.capture` | 使用事件捕获模式 |
| `.self` | 只在自身触发时触发 |
| `.once` | 只触发一次 |
| `.passive` | 使用passive事件监听器 |
| `.enter` | 回车键 |
| `.esc` | Esc键 |

---

## 事件处理

### 事件参数

```vue
<script setup lang="ts">
const handleClick = (event: MouseEvent) => {
  console.log('Button clicked', event)
  console.log('Target:', event.target)
}

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  console.log('Input value:', target.value)
}
</script>

<template>
  <button @click="handleClick($event)">Click me</button>
  <input @input="handleInput($event)">
</template>
```

### 自定义事件

```vue
<script setup lang="ts">
const emit = defineEmits<{
  'update:modelValue': [value: string]
  'submit': [payload: SubmitData]
}>()

const updateValue = (value: string) => {
  emit('update:modelValue', value)
}

const handleSubmit = () => {
  emit('submit', { data: 'test' })
}
</script>

<template>
  <input
    :value="modelValue"
    @input="updateValue($event.target.value)"
  >
  <button @click="handleSubmit">Submit</button>
</template>
```

---

## 样式规范

### ⚠️ 模板与CSS类名必须匹配

**重要**：CSS 类名必须与模板中实际使用的类名完全匹配。这是最常见的布局问题根源。

```vue
<!-- ❌ 错误：模板使用新结构，CSS保留旧样式 -->
<template>
  <div class="card-header">
    <div class="card-actions-row">...</div>
  </div>
</template>

<style scoped>
/* 旧样式，不会生效！ */
.action-bar { ... }
.bar-left { ... }
.bar-right { ... }
</style>

<!-- ✅ 正确：CSS与模板匹配 -->
<style scoped>
.card-header { ... }
.card-actions-row { ... }
.card-actions { ... }
</style>
```

**检查清单**：
1. 修改模板结构后，必须同步更新 CSS
2. 响应式媒体查询中的类名也需同步更新
3. 删除不再使用的旧 CSS 样式，避免混淆

### scoped样式

```vue
<style scoped>
/* 只作用于当前组件 */
.user-component {
  /* ... */
}

.user-component h2 {
  /* ... */
}
</style>
```

### CSS模块

```vue
<template>
  <div :class="$style.container">
    <h2 :class="$style.title">{{ title }}</h2>
  </div>
</template>

<style module>
.container {
  /* ... */
}

.title {
  /* ... */
}
</style>
```

### 动态class

```vue
<template>
  <div :class="{ active: isActive, disabled: isDisabled }">
    Dynamic class
  </div>

  <div :class="classObject">
    Dynamic class object
  </div>

  <div :class="[activeClass, errorClass]">
    Dynamic class array
  </div>
</template>

<script setup lang="ts">
const isActive = ref(true)
const isDisabled = ref(false)
const classObject = reactive({
  active: true,
  disabled: false
})
const activeClass = 'active'
const errorClass = 'error'
</script>
```

---

## 服务端排序与分页规范（⚠️强制）

### 使用 useServerTable Composable

**创建服务端排序页面时，必须使用 `useServerTable` composable**：

```typescript
// src/composables/useServerTable.ts
export interface ServerTableQuery {
  pageNum: number
  pageSize: number
  sortBy?: string
  sortOrder?: 'asc' | 'desc'
}

export function useServerTable(options: ServerTableOptions = {}) {
  const {
    defaultPageSize = 10,
    pageSizeOptions = [10, 20, 50, 100],
    defaultSortField = 'updatedAt',
    defaultSortOrder = 'desc'
  } = options

  const currentPage = ref(1)
  const pageSize = ref(defaultPageSize)
  const totalCount = ref(0)
  const sortField = ref<string>(defaultSortField)
  const sortOrder = ref<SortOrder>(defaultSortOrder)

  // 查询参数
  const query = computed<ServerTableQuery>(() => ({
    pageNum: currentPage.value,
    pageSize: pageSize.value,
    sortBy: sortField.value,
    sortOrder: sortOrder.value
  }))

  // ... 方法实现

  return {
    currentPage,
    pageSize,
    totalCount,
    sortField,
    sortOrder,
    query,
    handlePageChange,
    handlePageSizeChange,
    handleSort,
    reset
  }
}
```

### 页面组件实现

```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { dictTypeApi, type DictType } from '@/api/dict-type'
import { useServerTable } from '@/composables'

// 使用服务端表格 composable
const {
  query,
  sortField,
  sortOrder,
  totalCount,
  handlePageChange,
  handlePageSizeChange,
  handleSort,
  setTotal
} = useServerTable({
  defaultPageSize: 10,
  defaultSortField: 'updatedAt',
  defaultSortOrder: 'desc'
})

const dictTypes = ref<DictType[]>([])
const loading = ref(false)

// 加载数据
const loadDictTypes = async () => {
  loading.value = true
  try {
    const response = await dictTypeApi.getTypesPage(query.value)
    dictTypes.value = response.records
    setTotal(response.total)
  } finally {
    loading.value = false
  }
}

// 监听查询变化，重新加载数据
watch(() => query.value, () => {
  loadDictTypes()
}, { deep: true })

onMounted(() => {
  loadDictTypes()
})
</script>

<template>
  <table class="table">
    <thead>
      <tr>
        <th @click="handleSort('updatedAt')">
          更新时间
          <SortIcon :is-active="sortField === 'updatedAt'" :order="sortOrder" />
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in dictTypes" :key="item.id">
        <td>{{ formatDateTime(item.updatedAt) }}</td>
      </tr>
    </tbody>
  </table>
</template>
```

### CRUD操作后刷新

**⚠️ 强制**：添加、编辑、删除操作成功后，必须刷新列表。

```typescript
// ❌ 错误：只更新本地数据
const handleDelete = async (row: DictType) => {
  await dictTypeApi.deleteType(row.id)
  dictTypes.value = dictTypes.value.filter(t => t.id !== row.id)
  // 没有刷新，统计数据不会更新！
}

// ✅ 正确：刷新列表
const handleDelete = async (row: DictType) => {
  try {
    await dictTypeApi.deleteType(row.id)
    ElMessage.success('删除成功')
    await loadDictTypes() // 刷新列表，更新统计数据
  } catch (error) {
    ElMessage.error('删除失败')
  }
}
```

---

## API 类型定义规范

### 雪花算法ID处理

**⚠️ 强制**：所有使用雪花算法ID的接口，ID字段必须使用`string`类型。

```typescript
// ✅ 正确：ID使用string类型
export interface DictType {
  id: string  // 雪花算法ID使用string类型避免精度丢失
  typeCode: string
  typeName: string
  itemCount?: number
  createdAt: string
  updatedAt: string
}

// ❌ 错误：ID使用number类型
export interface DictType {
  id: number  // 精度会丢失！
  typeCode: string
  // ...
}
```

### API参数类型兼容

```typescript
// ✅ 正确：支持string | number，兼容性好
export const dictTypeApi = {
  deleteType(id: string | number): Promise<void> {
    return http.delete(`${BASE_URL}/${id}`)
  },

  getTypeById(id: string | number): Promise<DictType> {
    return http.get(`${BASE_URL}/${id}`)
  }
}
```

### 分页接口定义

```typescript
// 分页查询参数
export interface PageQueryParams {
  pageNum?: number
  pageSize?: number
  sortBy?: string
  sortOrder?: 'asc' | 'desc'
}

// 分页响应
export interface PageResponse<T> {
  records: T[]
  total: number
  pageNum: number
  pageSize: number
  pages: number
}
```

---

## 🔗 相关文档

- [组件设计规范](./component-design.md) - 组件设计详细规范
- [TypeScript规范](./typescript.md) - TypeScript详细规范
- [状态管理规范](./state-management.md) - Pinia详细规范

---

**最后更新**：2026-03-21