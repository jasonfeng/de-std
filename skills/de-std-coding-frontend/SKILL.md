---
name: coding-frontend
description: 前端 Vue 3/TypeScript/Element Plus 编码规范。写 Vue/TS 代码前必须调用。
---

# 前端编码规范

## 触发时机

- "写一个 Vue 组件/页面"
- "添加前端功能"
- "创建表单/表格/列表"
- "实现 UI 交互"

---

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

### UI 开发强制规则

| 禁止 | 原因 |
|------|------|
| 渐变背景 | 过于花哨 |
| 超过 4px 的圆角 | 不符合设计规范 |
| 花哨阴影 | 视觉干扰 |
| 硬编码颜色 | 无法统一主题 |
| 超大字体 | 不协调 |

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

## data-testid 规范

### 命名格式

| 元素类型 | 前缀 | 示例 |
|---------|------|------|
| 页面容器 | `{模块}-{页面}-page` | `quality-tasks-page` |
| 按钮 | `btn-{动作}` | `btn-create-task` |
| 输入框 | `input-{名称}` | `input-task-name` |
| 下拉选择 | `select-{名称}` | `select-target-table` |
| 表格 | `{页面}-table` | `task-table` |
| 弹窗 | `{功能}-modal` | `report-detail-modal` |

### 示例

```vue
<template>
  <div data-testid="quality-tasks-page">
    <button data-testid="btn-create-task">新建</button>
    <input data-testid="input-search" placeholder="搜索" />
    <table data-testid="task-table">...</table>
  </div>
</template>
```

---

## 服务端排序/分页

### useServerTable Composable

```typescript
const {
  data,
  loading,
  pagination,
  sortBy,
  sortOrder,
  refresh,
  handlePageChange,
  handleSortChange
} = useServerTable({
  fetchApi: userApi.getList,
  defaultSortBy: 'updatedAt',
  defaultSortOrder: 'desc'
})
```

### ID 类型处理

- VO 中 ID 使用 String 类型（避免前端精度丢失）
- API 类型定义中 ID 使用 string
- 统计字段（count/amount）保持 number 类型

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| 硬编码 Mock 数据 | 功能视为未完成 |
| 使用 `any` 类型 | 丢失类型安全 |
| 内联样式 | 维护困难 |
| 重建已有公共组件 | 代码审查失败 |
| `index` 作为 v-for key | 性能问题 |
| 依赖 CSS class 定位 | UI 变化导致测试失败 |

---

## 详细规范

- **[vue-coding.md](references/vue-coding.md)** — Vue 3 组件定义、Composition API、生命周期、指令使用
- **[typescript.md](references/typescript.md)** — TypeScript 严格模式、类型定义、泛型、工具类型
- **[component-design.md](references/component-design.md)** — 组件命名、Props/Emits/Slots、组件通信、data-testid 规范
- **[api-calling.md](references/api-calling.md)** — Axios 配置、请求拦截、错误处理、Token 刷新
- **[state-management.md](references/state-management.md)** — Pinia Store 定义、模块化、持久化
- **[routing.md](references/routing.md)** — 路由配置、命名规范、权限控制、动态路由
- **[styling.md](references/styling.md)** — UI 开发规则、BEM 命名、主题系统、CSS 变量
- **[element-plus.md](references/element-plus.md)** — Element Plus 组件使用、主题定制、表单验证
- **[ui-checklist.md](references/ui-checklist.md)** — 开发前/中/后检查清单
- **[testing.md](references/testing.md)** — 前端测试规范、Vitest、组件测试
- **[performance.md](references/performance.md)** — 代码分割、懒加载、渲染优化、内存优化
- **[file-organization.md](references/file-organization.md)** — 文件行数阈值、组件拆分、Composable 提取
- **[environment.md](references/environment.md)** — 环境变量、Vite 配置、部署配置
- **[error-monitoring.md](references/error-monitoring.md)** — 错误监控、Sentry 集成、错误处理
