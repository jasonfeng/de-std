# 组件设计规范

> 本规范定义Vue组件的设计规范，确保组件的可维护性和一致性。

---

## 📋 目录

- [组件命名规范](#组件命名规范)
- [组件结构规范](#组件结构规范)
- [Props规范](#props规范)
- [Emits规范](#emits规范)
- [Slots规范](#slots规范)
- [组件通信](#组件通信)
- [组件生命周期](#组件生命周期)

---

## 组件命名规范

### 文件命名

**格式**：`PascalCase.vue`

| 组件类型 | 命名格式 | 示例 |
|---------|---------|------|
| 单文件组件 | PascalCase | `UserComponent.vue`、`UserProfile.vue`、`DataTable.vue` |
| 父组件 | PascalCase（复合名称） | `UserProfilePanel.vue`、`DataTableToolbar.vue` |
| 子组件 | PascalCase | `UserAvatar.vue`、`Button.vue`、`Input.vue` |

**❌ 错误**：
```vue
<!-- user-component.vue -->
<!-- user-profile.vue (已存在，容易混淆) -->
<!-- Button.vue -->
<!-- button.vue -->
```

**✅ 正确**：
```vue
<!-- UserComponent.vue -->
<!-- UserProfile.vue (如有必要可以加User前缀区分) -->
<!-- PrimaryButton.vue / SecondaryButton.vue (功能区分) -->
```

### 组件注册

```typescript
// 注册组件
import UserComponent from './UserComponent.vue'

app.component('UserComponent', UserComponent)
```

---

## 组件结构规范

### 基本结构

```vue
<template>
  <!-- 模板 -->
</template>

<script setup lang="ts">
// 脚本
</script>

<style scoped>
/* 样式 */
</style>
```

### 模板顺序

```
1. 最外层容器
2. 条件渲染（v-if/v-show）
3. 列表渲染（v-for）
4. 动态组件
5. 静态内容
6. 空状态/加载状态
7. 其他
```

### 示例

```vue
<template>
  <div class="user-list">
    <!-- 条件渲染 -->
    <div v-if="loading" class="loading">Loading...</div>

    <!-- 列表渲染 -->
    <div v-for="user in users" :key="user.id" class="user-item">
      <UserAvatar :user="user" />
      <span>{{ user.name }}</span>
    </div>

    <!-- 空状态 -->
    <div v-if="!loading && users.length === 0" class="empty">
      No users found
    </div>
  </div>
</template>
```

---

## data-testid 可测试性规范

> **规范来源**：Story 7.5-10/7.5-11 E2E 测试实战踩坑总结。缺少 `data-testid` 是 E2E 测试不稳定的首要原因。
>
> **配套文档**：[E2E 测试规范 v3.0](../../04-testing/e2e-testing.md) — Section 4 元素定位规范

### 强制规则

| # | 规则 | 说明 | 违反后果 |
|---|------|------|---------|
| 1 | **页面容器必须加 `data-testid`** | 页面根元素加 `{模块}-{页面}-page` | E2E 冒烟测试无法定位页面 |
| 2 | **所有按钮必须加 `data-testid`** | 包括新建、删除、编辑、提交、取消、刷新、重置 | E2E 无法点击操作 |
| 3 | **所有表单输入必须加 `data-testid`** | 包括 input、select、textarea、radio、checkbox | E2E 无法填写表单 |
| 4 | **表格行操作链接必须加 `data-testid`** | 如执行、编辑、删除、查看详情 | E2E 无法操作行数据 |
| 5 | **表格名称列的链接/文本必须加 `data-testid`** | 用于行匹配和详情跳转 | E2E 无法定位特定行 |
| 6 | **筛选/搜索组件必须加 `data-testid`** | 包括搜索框、筛选下拉、日期选择 | E2E 无法筛选数据 |
| 7 | **弹窗/对话框必须加 `data-testid`** | 模态框容器 + 确认/取消按钮 | E2E 无法操作弹窗 |

### 命名规范

**格式**：`{类型前缀}-{语义名称}`

| 元素类型 | 前缀 | 示例 | 说明 |
|---------|------|------|------|
| 页面容器 | `{模块}-{页面}-page` | `quality-tasks-page`、`sync-tasks-page` | 每个页面唯一 |
| 按钮 | `btn-{动作}` | `btn-create-task`、`btn-delete`、`btn-refresh` | |
| 输入框 | `input-{名称}` | `input-task-name`、`input-search` | |
| 下拉选择 | `select-{名称}` | `select-target-table`、`select-data-source` | |
| 筛选器 | `filter-{名称}` | `filter-status`、`filter-schedule` | |
| 链接 | `link-{名称}` | `link-task-name`、`view-detail-link` | |
| 表格 | `{页面}-table` | `task-table` | |
| 单元格 | `cell-{名称}` | `cell-status`、`cell-type` | |
| 弹窗 | `{功能}-modal` / `{功能}-dialog` | `report-detail-modal`、`dialog-task` | |
| 图表 | `{类型}-chart` | `dimension-radar-chart`、`trend-line-chart` | |
| 分页 | `pagination` | `pagination` | |
| 复选框 | `checkbox-{名称}` | `checkbox-all`、`checkbox-row` | |
| 导出/下载 | `export-{名称}` / `download-{名称}` | `export-report-btn`、`download-report-link` | |

### 示例

```vue
<template>
  <!-- 页面容器 -->
  <div class="page-content" data-testid="quality-tasks-page">
    <!-- 操作按钮 -->
    <button class="btn btn-primary" data-testid="btn-create-task">新建任务</button>
    <button class="btn" data-testid="btn-refresh">刷新</button>
    <button class="btn" data-testid="btn-reset">重置</button>

    <!-- 搜索 -->
    <input data-testid="input-search" placeholder="搜索任务名" />

    <!-- 筛选 -->
    <select data-testid="filter-status">...</select>

    <!-- 表格 -->
    <table class="table" data-testid="task-table">
      <tr v-for="item in list" :key="item.id">
        <!-- 名称链接（truncateText 截断后用 title 保留完整值） -->
        <td>
          <a :title="item.taskName" data-testid="link-task-name"
             @click="handleDetail(item)">
            {{ truncateText(item.taskName, 20) }}
          </a>
        </td>
        <!-- 状态 -->
        <td><span data-testid="cell-status">{{ item.status }}</span></td>
        <!-- 操作 -->
        <td>
          <a data-testid="btn-execute" @click="handleExecute(item)">执行</a>
          <a data-testid="btn-edit" @click="handleEdit(item)">编辑</a>
          <a data-testid="btn-delete" @click="handleDelete(item)">删除</a>
        </td>
      </tr>
    </table>

    <!-- 分页 -->
    <PaginationBar data-testid="pagination" />
  </div>
</template>
```

### 与 truncateText 配合

使用 `truncateText` 截断的列，**必须同时添加 `:title` 属性和 `data-testid`**：

```vue
<!-- 截断文本 + title 保留完整值 + data-testid 供 E2E 定位 -->
<a :title="item.taskName" data-testid="link-task-name">
  {{ truncateText(item.taskName, 20) }}
</a>
```

E2E 测试通过 `title` 属性匹配完整文本，而非 `textContent()` 返回的截断文本。

### truncateText 复用规范

**禁止重复定义 `truncateText` 函数**。项目已有公共导出：

```typescript
// src/api/transformers.ts（已有）
export function truncateText(text: string | null | undefined, maxLength: number = 20): string
```

```vue
<!-- 在组件中使用 -->
<script setup lang="ts">
import { truncateText } from '@/api/transformers'
</script>
```

**禁止**：在组件内部重新定义 `truncateText`。如果需要不同的截断长度，调用时传参即可：`truncateText(text, 30)`。

---

## Props规范

### Props定义

```typescript
// 基本类型
interface Props {
  title: string
  count?: number
  disabled?: boolean
}

const props = defineProps<Props>()
```

### Props默认值

```typescript
interface Props {
  title: string
  count?: number
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  count: 10,
  disabled: false
})
```

### Props验证

```typescript
interface Props {
  title: string
  count: number
  status: 'active' | 'inactive' | 'deleted'
}

const props = defineProps<Props>()

// 使用验证
watch(() => props.status, (newStatus) => {
  if (!['active', 'inactive', 'deleted'].includes(newStatus)) {
    throw new Error('Invalid status')
  }
})
```

### Props命名

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| 字符串 | `name`、`title` | `username`、`email` |
| 数字 | `count`、`index` | `maxCount`、`startIndex` |
| 布尔 | `disabled`、`loading` | `isVisible`、`isLoading` |
| 数组 | `items`、`list` | `users`、`options` |
| 对象 | `config`、`user` | `userConfig`、`currentUser` |
| 函数 | `onXxx` | `onSubmit`、`onCancel` |
| 事件 | `@update:xxx` | `@update:title` |

---

## Emits规范

### Emits定义

```typescript
const emit = defineEmits<{
  'update:modelValue': [value: string]
  'submit': [payload: SubmitData]
  'cancel': []
}>()
```

### 触发事件

```typescript
// 触发更新事件
const updateValue = (value: string) => {
  emit('update:modelValue', value)
}

// 触发自定义事件
const handleSubmit = () => {
  emit('submit', { data: 'test' })
}

// 触发无参数事件
const handleCancel = () => {
  emit('cancel')
}
```

### 命名规范

| 事件类型 | 命名格式 | 示例 |
|---------|---------|------|
| 更新v-model | `update:xxx` | `update:modelValue`、`update:title` |
| 动作事件 | `xxx` | `submit`、`cancel`、`delete` |
| 通知事件 | `xxx` | `success`、`error`、`warning` |

---

## Slots规范

### 具名插槽

```vue
<template>
  <div class="modal">
    <div class="modal-header">
      <slot name="header">
        <h2>Default Header</h2>
      </slot>
    </div>

    <div class="modal-body">
      <slot>Default content</slot>
    </div>

    <div class="modal-footer">
      <slot name="footer">
        <button>Default Button</button>
      </slot>
    </div>
  </div>
</template>

<style scoped>
.modal-header,
.modal-body,
.modal-footer {
  /* ... */
}
</style>
```

### 使用插槽

```vue
<Modal>
  <template #header>
    <h2>Custom Header</h2>
  </template>

  <template #default>
    <p>Custom content</p>
  </template>

  <template #footer>
    <button @click="handleConfirm">Confirm</button>
    <button @click="handleCancel">Cancel</button>
  </template>
</Modal>
```

### 作用域插槽

```vue
<template>
  <div class="list">
    <slot name="item" v-for="item in items" :item="item" :index="index">
      {{ item.name }}
    </slot>
  </div>
</template>

<script setup lang="ts">
interface Props {
  items: Array<{ id: number; name: string }>
}

const props = defineProps<Props>()
</script>
```

```vue
<List :items="users">
  <template #item="{ item, index }">
    {{ index }}: {{ item.name }}
  </template>
</List>
```

---

## 组件通信

### Props Down / Events Up

```vue
<!-- Parent.vue -->
<template>
  <Child
    :message="message"
    @update:message="handleUpdateMessage"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'

const message = ref('Hello')

const handleUpdateMessage = (newMessage: string) => {
  message.value = newMessage
}
</script>

<!-- Child.vue -->
<template>
  <input :value="message" @input="$emit('update:message', $event.target.value)">
</template>

<script setup lang="ts">
const props = defineProps<{
  message: string
}>()

const emit = defineEmits<{
  'update:message': [value: string]
}>()
</script>
```

### provide/inject

```vue
<!-- Parent.vue -->
<template>
  <Child />
</template>

<script setup lang="ts">
import { provide, ref } from 'vue'

const message = ref('Hello from parent')

provide('message', message)
</script>

<!-- Child.vue -->
<template>
  <div>{{ message }}</div>
</template>

<script setup lang="ts">
import { inject } from 'vue'

const message = inject('message', ref('default message'))
</script>
```

### Pinia Store

```vue
<script setup lang="ts">
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// 读取状态
const username = computed(() => userStore.username)

// 调用action
const logout = () => {
  userStore.logout()
}
</script>
```

---

## 组件生命周期

### setup()中的初始化

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const users = ref<User[]>([])

// 使用onMounted
onMounted(() => {
  fetchUsers()
})

const fetchUsers = async () => {
  const response = await api.getUsers()
  users.value = response.data
}
</script>
```

### watch监听Props变化

```vue
<script setup lang="ts">
const props = defineProps<{
  userId: number
}>()

const user = ref<User | null>(null)

// 监听props变化
watch(
  () => props.userId,
  async (newUserId) => {
    if (newUserId) {
      user.value = await api.getUser(newUserId)
    }
  },
  { immediate: true }
)
</script>
```

### 清理副作用

```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

let timer: number | null = null

onMounted(() => {
  timer = setInterval(() => {
    console.log('Timer tick')
  }, 1000)
})

onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
  }
})
</script>
```

---

## 通用组件清单

### SqlCodeBlock（SQL代码展示）

用于展示SQL语句，支持语法高亮和复制功能。

**位置**：`src/components/common/SqlCodeBlock.vue`

**Props**：
| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `code` | `string` | 必填 | SQL代码内容 |
| `language` | `string` | `'sql'` | 语法高亮语言 |

**使用示例**：
```vue
<template>
  <SqlCodeBlock :code="ddlContent" />
</template>

<script setup lang="ts">
import { SqlCodeBlock } from '@/components/common'
</script>
```

**依赖**：
- `prismjs` - 语法高亮库
- `prismjs/themes/prism.css` - 亮色主题

**特性**：
- 白色背景，等宽字体
- 内置复制按钮
- 最大高度500px，超出自动滚动

---

## 🔗 相关文档

- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [TypeScript规范](./typescript.md) - TypeScript详细规范
- [状态管理规范](./state-management.md) - Pinia详细规范

---

**最后更新**：2026-03-29