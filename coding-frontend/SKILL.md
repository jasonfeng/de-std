---
name: coding-frontend
description: 前端 Vue 3/TypeScript/Element Plus 编码规范。写前端代码前必须调用。
---

# 前端编码规范

## 调用时机

- 写 Vue 组件之前
- 写 TypeScript 文件之前
- 写 API 调用代码之前
- 写样式之前

## Vue 3 规范

### 组件命名

```
components/
├── common/           # 公共组件（必须复用）
│   ├── DataTable.vue
│   ├── StandardTable.vue
│   └── EmptyState.vue
└── features/         # 功能组件
    ├── UserForm.vue
    └── OrderCard.vue
```

**强制规则**：开发前检查 `components/common/` 是否已有可复用组件

### 组件结构

```vue
<template>
  <!-- 模板 -->
</template>

<script setup lang="ts">
// 导入
import { ref, computed, onMounted } from 'vue'

// 类型定义
interface Props { }
interface Emits { }

// Props/Emits
const props = defineProps<Props>()
const emit = defineEmits<Emits>()

// 状态
const loading = ref(false)

// 方法
function handleClick() { }

// 生命周期
onMounted(() => { })
</script>

<style scoped>
/* 样式 */
</style>
```

---

## TypeScript 规范

### 类型定义

```typescript
// ✅ 使用 interface 定义对象
interface User {
  id: string
  name: string
}

// ✅ 使用 type 定义联合类型
type Status = 'active' | 'inactive'

// ❌ 禁止 any
function process(data: any) { }  // ❌

// ✅ 使用 unknown
function process(data: unknown) {
  if (typeof data === 'string') {
    // ...
  }
}
```

---

## API 调用规范

### 禁止硬编码 Mock 数据

```typescript
// ❌ 禁止
const mockUsers = [
  { id: 1, name: '张三' },
  { id: 2, name: '李四' }
]

// ✅ 正确：从 API 获取
const users = ref<User[]>([])
onMounted(async () => {
  users.value = await userApi.list()
})
```

### API 文件结构

```
api/
├── user/
│   ├── index.ts      # API 方法
│   └── types.ts      # 类型定义
└── request.ts        # Axios 配置
```

---

## 样式规范

### 禁止内联样式

```vue
<!-- ❌ 禁止 -->
<div style="color: red; padding: 10px">

<!-- ✅ 正确 -->
<div class="error-text">

<style scoped>
.error-text {
  color: #ef4444;
  padding: 10px;
}
</style>
```

### 全局样式优先

```
styles/
├── button.css        # 按钮样式（32px 高度）
├── table-common.css  # 表格样式
└── variables.css     # CSS 变量
```

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| 硬编码 Mock 数据 | 功能视为未完成 |
| 使用 `any` 类型 | 丢失类型安全 |
| 内联样式 | 维护困难 |
| 重建已有公共组件 | 代码审查失败 |
| `index` 作为 v-for key | 性能问题 |