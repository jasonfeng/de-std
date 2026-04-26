# 前端文件组织与长度管理规范

> 本规范定义前端文件的拆分策略和长度管理原则，确保代码可维护性。
> **核心原则**：一个文件只做一件事。

---

## 目录

- [行数阈值](#行数阈值)
- [组件拆分](#组件拆分)
- [Composable 提取](#composable-提取)
- [类型与常量外置](#类型与常量外置)
- [样式拆分](#样式拆分)
- [判断标准与决策树](#判断标准与决策树)
- [常见反模式](#常见反模式)

---

## 行数阈值

单文件组件（SFC）各区域建议上限：

| 区域 | 建议上限 | 超出时做法 |
|------|---------|-----------|
| `<template>` | ~150 行 | 提取子组件 |
| `<script setup>` | ~150 行 | 提取 Composable |
| `<style scoped>` | ~100 行 | 提取样式文件 |
| **单文件总体** | **~300 行** | 综合运用以上策略 |

> **注意**：阈值是**预警信号**，不是硬性规则。100 行的文件如果有 3 个不相关的职责，也应该拆分。

---

## 组件拆分

### 拆分原则

按**职责边界**提取子组件，而不是按"行数"机械拆分。

**判断标准**：一段模板 + 对应的逻辑可以被**独立描述和理解**，就该提取。

### 示例

```vue
<!-- ❌ 拆分前：SyncTaskDetail.vue（500+ 行）-->
<template>
  <div class="task-detail">
    <!-- 头部区域 -->
    <div class="header">
      <el-page-header @back="goBack">
        <template #content>{{ task.name }}</template>
      </el-page-header>
      <el-dropdown>...</el-dropdown>
    </div>

    <!-- 基本信息卡片 -->
    <el-card>
      <el-descriptions :column="3">
        <el-descriptions-item label="状态">...</el-descriptions-item>
        <!-- ... 20+ 个字段 ... -->
      </el-descriptions>
    </el-card>

    <!-- 配置面板 -->
    <el-card>
      <el-tabs>
        <el-tab-pane label="数据源配置">...</el-tab-pane>
        <el-tab-pane label="目标配置">...</el-tab-pane>
        <el-tab-pane label="字段映射">...</el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 执行日志 -->
    <el-card>
      <el-table :data="logs">...</el-table>
    </el-card>
  </div>
</template>

<script setup lang="ts">
// 500+ 行的逻辑...
</script>
```

```vue
<!-- ✅ 拆分后：SyncTaskDetail.vue（~80 行）-->
<template>
  <div class="task-detail">
    <TaskHeader :task="task" @back="goBack" @edit="handleEdit" @delete="handleDelete" />
    <TaskBasicInfo :task="task" />
    <TaskConfigPanel :config="task.config" :readonly="!isEditing" />
    <TaskExecutionLog :task-id="task.id" />
  </div>
</template>

<script setup lang="ts">
import { useTaskDetail } from './composables/useTaskDetail'
import { useTaskActions } from './composables/useTaskActions'

const props = defineProps<{ taskId: string }>()
const { task, loading } = useTaskDetail(computed(() => props.taskId))
const { handleEdit, handleDelete } = useTaskActions(task)
</script>
```

### 提取粒度

| 粒度 | 适用场景 | 示例 |
|------|---------|------|
| **页面级区块** | 同一页面内独立的 UI 区域 | `TaskHeader`、`TaskBasicInfo` |
| **功能组件** | 可跨页面复用的功能单元 | `DataTable`、`StatCard` |
| **业务组件** | 特定业务场景的封装 | `DataSourceSelector`、`FieldMapping` |

---

## Composable 提取

### 提取时机

当 `<script setup>` 中出现以下信号时，应提取 Composable：

1. **逻辑超过 ~100 行**
2. **存在独立的数据获取逻辑**（API 调用 + 状态管理）
3. **存在独立的事件处理逻辑**（多个 handler 围绕同一主题）
4. **相同逻辑可能被其他组件复用**

### 命名规范

```
composables/use{Feature}.ts
```

### 示例

```typescript
// composables/useTaskDetail.ts
import { ref, computed, type Ref } from 'vue'
import { taskApi, type SyncTask } from '@/api/sync-task'

export function useTaskDetail(taskId: Ref<string>) {
  const task = ref<SyncTask | null>(null)
  const loading = ref(false)

  const loadTask = async () => {
    loading.value = true
    try {
      task.value = await taskApi.getById(taskId.value)
    } finally {
      loading.value = false
    }
  }

  // 监听 ID 变化自动加载
  watch(taskId, loadTask, { immediate: true })

  return { task, loading, refresh: loadTask }
}
```

```typescript
// composables/useTaskActions.ts
import { taskApi } from '@/api/sync-task'
import { ElMessage, ElMessageBox } from 'element-plus'
import type { Ref } from 'vue'

export function useTaskActions(task: Ref<SyncTask | null>) {
  const handleDelete = async () => {
    await ElMessageBox.confirm('确认删除？', '提示')
    await taskApi.delete(task.value!.id)
    ElMessage.success('删除成功')
  }

  const handleToggleStatus = async () => {
    await taskApi.toggleStatus(task.value!.id)
    ElMessage.success('操作成功')
  }

  return { handleDelete, handleToggleStatus }
}
```

### Composable 组织规则

| 规则 | 说明 |
|------|------|
| 单一职责 | 每个 Composable 只封装一个关注点 |
| 返回对象 | 使用 `return { ... }` 而非导出多个 ref |
| 接受参数 | 通过参数注入依赖，不要在内部硬编码 |
| 副作用清理 | 有定时器/订阅时必须在 `onUnmounted` 中清理 |

---

## 类型与常量外置

### 类型定义

```typescript
// ❌ 错误：类型写在 .vue 文件中
<script setup lang="ts">
interface SyncTask {
  id: string
  name: string
  // ... 20+ 个字段
}
</script>

// ✅ 正确：类型定义在独立文件中
// types/task.ts
export interface SyncTask {
  id: string
  name: string
  // ...
}

// SyncTaskDetail.vue
<script setup lang="ts">
import type { SyncTask } from './types/task'
</script>
```

### 常量配置

```typescript
// ❌ 错误：常量散落在组件中
const statusOptions = [
  { label: '运行中', value: 'running' },
  // ...
]

// ✅ 正确：常量集中管理
// constants/task.ts
export const TASK_STATUS_OPTIONS = [
  { label: '运行中', value: 'running' },
  { label: '已停止', value: 'stopped' },
  { label: '异常', value: 'error' },
] as const

export const DEFAULT_PAGE_SIZE = 10
```

### 组织方式

```
src/views/integration/sync-task/
├── SyncTaskDetail.vue          # 页面主组件（< 300 行）
├── components/                  # 页面级子组件
│   ├── TaskHeader.vue
│   ├── TaskBasicInfo.vue
│   └── TaskConfigPanel.vue
├── composables/                 # 页面级 Composable
│   ├── useTaskDetail.ts
│   └── useTaskActions.ts
├── types/                       # 页面级类型定义
│   └── task.ts
└── constants/                   # 页面级常量
    └── task.ts
```

---

## 样式拆分

当 `<style>` 超过 100 行时，提取到独立文件：

```vue
<!-- ❌ 样式内联 -->
<style scoped>
.task-detail { ... }
.task-detail .header { ... }
.task-detail .body { ... }
/* ... 100+ 行 ... */
</style>

<!-- ✅ 样式外置 -->
<style scoped lang="scss">
@import './styles/task-detail.scss';
</style>
```

---

## 判断标准与决策树

### 文件需要拆分的信号

1. **多个不相关的"段落"**：文件里有明显可以独立讨论的部分
2. **滚动超过一屏**：需要频繁上下滚动才能理解上下文
3. **修改时需要理解整个文件**：改一个功能需要阅读大量无关代码
4. **Git diff 难以定位**：PR 中改动涉及大量上下文

### 决策树

```
文件超过 300 行？
├── 否 → 检查是否有不相关的职责段落
│   ├── 有 → 按职责拆分
│   └── 无 → 保持现状
└── 是 → 分析哪个区域最长
    ├── <template> 最长 → 提取子组件
    ├── <script setup> 最长 → 提取 Composable
    └── <style> 最长 → 提取样式文件
```

---

## 常见反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|---------|
| **复制粘贴组件代码** | 相似 UI 在多处重复 | 提取可配置的子组件 |
| **巨型 Composable** | 一个 use 函数包含所有逻辑 | 按关注点拆分为多个 use 函数 |
| **过早抽象** | 只用一次的组件也提取 | 至少 2 处使用才提取为公共组件 |
| **碎片化** | 每个小组件都独立文件 | 页面级组件放在同目录的 `components/` 下 |
| **全局常量污染** | 所有常量都放全局 | 页面级常量放在页面目录，跨页面常量放全局 |

---

## 链接

- [组件设计规范](./component-design.md) — 组件命名、Props/Emits、通信模式
- [Vue 3 编码规范](./vue-coding.md) — Composition API、生命周期、指令
- [TypeScript 规范](./typescript.md) — 类型定义、严格模式

---

**最后更新**：2026-04-02
