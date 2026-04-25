# Element Plus 使用规范

> 本规范定义Element Plus组件库的使用规范，确保UI组件的一致性和可维护性。

---

## 📋 目录

- [引入方式](#引入方式)
- [主题定制](#主题定制)
- [组件使用规范](#组件使用规范)
- [常用组件示例](#常用组件示例)
- [表单验证](#表单验证)
- [表格使用](#表格使用)
- [弹窗与抽屉](#弹窗与抽屉)
- [消息提示](#消息提示)
- [图标使用](#图标使用)
- [响应式布局](#响应式布局)

---

## 引入方式

### 完整引入（不推荐）

```typescript
// src/main.ts
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import zhCn from 'element-plus/es/locale/lang/zh-cn'

app.use(ElementPlus, {
  locale: zhCn
})
```

### 按需引入（推荐）

```typescript
// src/main.ts
import {
  ElButton,
  ElForm,
  ElInput,
  ElTable,
  ElMessage,
  ElMessageBox
} from 'element-plus'

app.use(ElButton)
app.use(ElForm)
app.use(ElInput)
app.use(ElTable)
```

### 自动导入（推荐）

安装插件：

```bash
npm install -D unplugin-vue-components unplugin-auto-import
```

配置 vite.config.ts：

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      resolvers: [ElementPlusResolver()],
      imports: ['vue', 'vue-router', 'pinia']
    }),
    Components({
      resolvers: [ElementPlusResolver()]
    })
  ]
})
```

---

## 主题定制

### SCSS变量覆盖

```scss
// src/styles/element-variables.scss
$--color-primary: #409eff;
$--color-success: #67c23a;
$--color-warning: #e6a23c;
$--color-danger: #f56c6c;

$--border-radius-base: 4px;
$--border-radius-small: 2px;

@forward 'element-plus/theme-chalk/src/common/var.scss' with (
  $colors: (
    'primary': (
      'base': $--color-primary
    )
  )
);
```

### CSS变量覆盖

```css
/* src/styles/element-theme.css */
:root {
  --el-color-primary: #409eff;
  --el-color-success: #67c23a;
  --el-color-warning: #e6a23c;
  --el-color-danger: #f56c6c;

  --el-border-radius-base: 4px;
  --el-font-size-base: 14px;
}
```

### 动态主题切换

```typescript
// src/stores/theme.ts
import { defineStore } from 'pinia'

export const useThemeStore = defineStore('theme', {
  state: () => ({
    isDark: false,
    primaryColor: '#409eff'
  }),

  actions: {
    toggleTheme() {
      this.isDark = !this.isDark
      document.documentElement.classList.toggle('dark')
    },

    setPrimaryColor(color: string) {
      this.primaryColor = color
      document.documentElement.style.setProperty('--el-color-primary', color)
    }
  }
})
```

---

## 组件使用规范

### 组件命名

| 规则 | 说明 | 示例 |
|------|------|------|
| 按钮前缀 | 使用语义化前缀 | `el-button-submit`, `el-button-cancel` |
| 表单前缀 | 使用 `form-` 前缀 | `form-user`, `form-role` |
| 表格前缀 | 使用 `table-` 前缀 | `table-user`, `table-role` |
| 弹窗前缀 | 使用 `dialog-` 前缀 | `dialog-user`, `dialog-role` |

### 组件封装示例

```vue
<!-- src/components/common/DataTable.vue -->
<template>
  <div class="data-table">
    <!-- 表格头部 -->
    <div class="data-table-header">
      <slot name="header" />
      <el-button
        v-if="showRefresh"
        :icon="Refresh"
        @click="handleRefresh"
      >
        刷新
      </el-button>
    </div>

    <!-- 表格主体 -->
    <el-table
      v-loading="loading"
      :data="data"
      :height="height"
      :stripe="stripe"
      :border="border"
      @selection-change="handleSelectionChange"
    >
      <slot />
    </el-table>

    <!-- 分页 -->
    <div v-if="showPagination" class="data-table-pagination">
      <el-pagination
        v-model:current-page="currentPage"
        v-model:page-size="pageSize"
        :total="total"
        :page-sizes="[10, 20, 50, 100]"
        layout="total, sizes, prev, pager, next, jumper"
        @current-change="handlePageChange"
        @size-change="handleSizeChange"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Refresh } from '@element-plus/icons-vue'

interface Props {
  data: any[]
  loading?: boolean
  total?: number
  height?: string | number
  stripe?: boolean
  border?: boolean
  showRefresh?: boolean
  showPagination?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
  total: 0,
  stripe: true,
  border: true,
  showRefresh: true,
  showPagination: true
})

const emit = defineEmits<{
  'refresh': []
  'selection-change': [selection: any[]]
  'page-change': [page: number]
  'size-change': [size: number]
}>()

const currentPage = ref(1)
const pageSize = ref(10)

const handleRefresh = () => {
  emit('refresh')
}

const handleSelectionChange = (selection: any[]) => {
  emit('selection-change', selection)
}

const handlePageChange = (page: number) => {
  currentPage.value = page
  emit('page-change', page)
}

const handleSizeChange = (size: number) => {
  pageSize.value = size
  emit('size-change', size)
}
</script>
```

---

## 常用组件示例

### 按钮（Button）

```vue
<template>
  <div class="button-group">
    <!-- 主要按钮 -->
    <el-button type="primary" @click="handlePrimary">
      主要按钮
    </el-button>

    <!-- 成功按钮 -->
    <el-button type="success" @click="handleSuccess">
      成功按钮
    </el-button>

    <!-- 警告按钮 -->
    <el-button type="warning" @click="handleWarning">
      警告按钮
    </el-button>

    <!-- 危险按钮 -->
    <el-button type="danger" @click="handleDanger">
      危险按钮
    </el-button>

    <!-- 带图标 -->
    <el-button :icon="Plus" type="primary">
      新增
    </el-button>

    <!-- 禁用状态 -->
    <el-button disabled>
      禁用按钮
    </el-button>

    <!-- 加载状态 -->
    <el-button :loading="loading" @click="handleLoading">
      加载中
    </el-button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Plus } from '@element-plus/icons-vue'

const loading = ref(false)

const handleLoading = async () => {
  loading.value = true
  try {
    await fetchData()
  } finally {
    loading.value = false
  }
}
</script>
```

### 输入框（Input）

```vue
<template>
  <div class="input-group">
    <!-- 基础输入框 -->
    <el-input v-model="username" placeholder="请输入用户名" />

    <!-- 带前缀 -->
    <el-input v-model="email" placeholder="请输入邮箱">
      <template #prefix>
        <el-icon><Message /></el-icon>
      </template>
    </el-input>

    <!-- 带后缀 -->
    <el-input v-model="password" type="password" show-password>
      <template #suffix>
        <el-icon><Lock /></el-icon>
      </template>
    </el-input>

    <!-- 文本域 -->
    <el-input
      v-model="description"
      type="textarea"
      :rows="3"
      placeholder="请输入描述"
    />

    <!-- 字数限制 -->
    <el-input
      v-model="remark"
      type="textarea"
      :rows="3"
      placeholder="请输入备注"
      maxlength="200"
      show-word-limit
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Message, Lock } from '@element-plus/icons-vue'

const username = ref('')
const email = ref('')
const password = ref('')
const description = ref('')
const remark = ref('')
</script>
```

### 选择器（Select）

```vue
<template>
  <div class="select-group">
    <!-- 基础选择器 -->
    <el-select v-model="value" placeholder="请选择">
      <el-option
        v-for="item in options"
        :key="item.value"
        :label="item.label"
        :value="item.value"
      />
    </el-select>

    <!-- 可搜索 -->
    <el-select
      v-model="value"
      filterable
      placeholder="请选择"
    >
      <el-option
        v-for="item in options"
        :key="item.value"
        :label="item.label"
        :value="item.value"
      />
    </el-select>

    <!-- 可清空 -->
    <el-select
      v-model="value"
      clearable
      placeholder="请选择"
    >
      <el-option
        v-for="item in options"
        :key="item.value"
        :label="item.label"
        :value="item.value"
      />
    </el-select>

    <!-- 远程搜索 -->
    <el-select
      v-model="value"
      filterable
      remote
      :remote-method="remoteMethod"
      :loading="loading"
      placeholder="请输入关键词搜索"
    >
      <el-option
        v-for="item in remoteOptions"
        :key="item.value"
        :label="item.label"
        :value="item.value"
      />
    </el-select>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const value = ref('')
const loading = ref(false)

const options = [
  { value: 'option1', label: '选项1' },
  { value: 'option2', label: '选项2' },
  { value: 'option3', label: '选项3' }
]

const remoteOptions = ref([])

const remoteMethod = async (query: string) => {
  if (query) {
    loading.value = true
    try {
      const response = await searchOptions(query)
      remoteOptions.value = response
    } finally {
      loading.value = false
    }
  } else {
    remoteOptions.value = []
  }
}
</script>
```

---

## 表单验证

### 表单验证规则

```typescript
// src/utils/validate.ts

/**
 * 验证必填
 */
export const validateRequired = (message: string = '该项不能为空') => {
  return {
    required: true,
    message,
    trigger: 'blur'
  }
}

/**
 * 验证邮箱
 */
export const validateEmail = (message: string = '请输入正确的邮箱地址') => {
  return {
    type: 'email',
    message,
    trigger: 'blur'
  }
}

/**
 * 验证手机号
 */
export const validatePhone = (message: string = '请输入正确的手机号') => {
  return {
    pattern: /^1[3-9]\d{9}$/,
    message,
    trigger: 'blur'
  }
}

/**
 * 验证长度
 */
export const validateLength = (min: number, max: number) => {
  return {
    min,
    max,
    message: `长度在 ${min} 到 ${max} 个字符`,
    trigger: 'blur'
  }
}

/**
 * 自定义验证函数
 */
export const validateUsername = (rule: any, value: string, callback: any) => {
  if (!value) {
    callback(new Error('用户名不能为空'))
  } else if (!/^[a-zA-Z0-9_]{4,20}$/.test(value)) {
    callback(new Error('用户名由4-20位字母、数字或下划线组成'))
  } else {
    callback()
  }
}
```

### 表单组件示例

```vue
<template>
  <el-form
    ref="formRef"
    :model="formData"
    :rules="rules"
    label-width="120px"
    @submit.prevent="handleSubmit"
  >
    <el-form-item label="用户名" prop="username">
      <el-input v-model="formData.username" placeholder="请输入用户名" />
    </el-form-item>

    <el-form-item label="邮箱" prop="email">
      <el-input v-model="formData.email" placeholder="请输入邮箱" />
    </el-form-item>

    <el-form-item label="手机号" prop="phone">
      <el-input v-model="formData.phone" placeholder="请输入手机号" />
    </el-form-item>

    <el-form-item label="密码" prop="password">
      <el-input
        v-model="formData.password"
        type="password"
        show-password
        placeholder="请输入密码"
      />
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="handleSubmit">
        提交
      </el-button>
      <el-button @click="handleReset">
        重置
      </el-button>
    </el-form-item>
  </el-form>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'
import { validateRequired, validateEmail, validatePhone, validateLength } from '@/utils/validate'

const formRef = ref<FormInstance>()

const formData = reactive({
  username: '',
  email: '',
  phone: '',
  password: ''
})

const rules: FormRules = {
  username: [
    validateRequired('用户名不能为空'),
    { validator: validateUsername, trigger: 'blur' }
  ],
  email: [
    validateRequired('邮箱不能为空'),
    validateEmail()
  ],
  phone: [
    validateRequired('手机号不能为空'),
    validatePhone()
  ],
  password: [
    validateRequired('密码不能为空'),
    validateLength(6, 20)
  ]
}

const handleSubmit = async () => {
  if (!formRef.value) return

  try {
    await formRef.value.validate()
    // 提交表单
    await submitForm(formData)
    ElMessage.success('提交成功')
  } catch (error) {
    console.error('表单验证失败:', error)
  }
}

const handleReset = () => {
  formRef.value?.resetFields()
}
</script>
```

---

## 表格使用

### 基础表格

```vue
<template>
  <el-table
    :data="tableData"
    :loading="loading"
    stripe
    border
    @selection-change="handleSelectionChange"
  >
    <!-- 选择列 -->
    <el-table-column type="selection" width="55" />

    <!-- 序号列 -->
    <el-table-column type="index" label="序号" width="80" />

    <!-- 普通列 -->
    <el-table-column prop="name" label="姓名" min-width="120" />

    <!-- 自定义列 -->
    <el-table-column label="状态" width="100">
      <template #default="{ row }">
        <el-tag :type="getStatusType(row.status)">
          {{ getStatusText(row.status) }}
        </el-tag>
      </template>
    </el-table-column>

    <!-- 操作列 -->
    <el-table-column label="操作" width="200" fixed="right">
      <template #default="{ row }">
        <el-button
          type="primary"
          link
          @click="handleEdit(row)"
        >
          编辑
        </el-button>
        <el-button
          type="danger"
          link
          @click="handleDelete(row)"
        >
          删除
        </el-button>
      </template>
    </el-table-column>
  </el-table>

  <!-- 分页 -->
  <el-pagination
    v-model:current-page="currentPage"
    v-model:page-size="pageSize"
    :total="total"
    :page-sizes="[10, 20, 50, 100]"
    layout="total, sizes, prev, pager, next, jumper"
    @current-change="handlePageChange"
    @size-change="handleSizeChange"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

interface User {
  id: number
  name: string
  status: number
}

const loading = ref(false)
const tableData = ref<User[]>([])
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)

const getStatusType = (status: number) => {
  const types: Record<number, any> = {
    1: 'success',
    2: 'warning',
    3: 'danger'
  }
  return types[status] || 'info'
}

const getStatusText = (status: number) => {
  const texts: Record<number, string> = {
    1: '启用',
    2: '禁用',
    3: '锁定'
  }
  return texts[status] || '未知'
}

const handleSelectionChange = (selection: User[]) => {
  console.log('选中:', selection)
}

const handleEdit = (row: User) => {
  console.log('编辑:', row)
}

const handleDelete = async (row: User) => {
  try {
    await ElMessageBox.confirm(
      `确定要删除用户 "${row.name}" 吗？`,
      '提示',
      {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }
    )

    // 执行删除
    await deleteUser(row.id)
    ElMessage.success('删除成功')

    // 刷新列表
    fetchUsers()
  } catch (error) {
    // 用户取消
  }
}
</script>
```

---

## 弹窗与抽屉

### Dialog 弹窗

```vue
<template>
  <el-dialog
    v-model="visible"
    :title="title"
    :width="width"
    :close-on-click-modal="false"
    @close="handleClose"
  >
    <el-form
      ref="formRef"
      :model="formData"
      :rules="rules"
      label-width="120px"
    >
      <!-- 表单内容 -->
    </el-form>

    <template #footer>
      <el-button @click="handleClose">
        取消
      </el-button>
      <el-button type="primary" @click="handleConfirm">
        确定
      </el-button>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import type { FormInstance } from 'element-plus'

interface Props {
  modelValue: boolean
  title?: string
  width?: string | number
}

const props = withDefaults(defineProps<Props>(), {
  title: '弹窗',
  width: '600px'
})

const emit = defineEmits<{
  'update:modelValue': [value: boolean]
  'confirm': []
}>()

const visible = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const formRef = ref<FormInstance>()
const formData = ref({})
const rules = {}

const handleClose = () => {
  visible.value = false
  formRef.value?.resetFields()
}

const handleConfirm = async () => {
  try {
    await formRef.value?.validate()
    emit('confirm')
    handleClose()
  } catch (error) {
    console.error('表单验证失败:', error)
  }
}
</script>
```

### Drawer 抽屉

```vue
<template>
  <el-drawer
    v-model="visible"
    :title="title"
    :size="size"
    @close="handleClose"
  >
    <!-- 内容 -->
  </el-drawer>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  modelValue: boolean
  title?: string
  size?: string | number
}

const props = withDefaults(defineProps<Props>(), {
  title: '抽屉',
  size: '40%'
})

const emit = defineEmits<{
  'update:modelValue': [value: boolean]
}>()

const visible = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const handleClose = () => {
  visible.value = false
}
</script>
```

---

## 消息提示

### Message 提示

```typescript
import { ElMessage } from 'element-plus'

// 成功提示
ElMessage.success('操作成功')

// 警告提示
ElMessage.warning('警告信息')

// 错误提示
ElMessage.error('操作失败')

// 信息提示
ElMessage.info('提示信息')

// 可关闭
ElMessage({
  message: '这是一条消息',
  showClose: true
})

// 自定义时长
ElMessage({
  message: '3秒后关闭',
  duration: 3000
})

// HTML内容
ElMessage({
  message: '<strong>这是 <i>HTML</i> 内容</strong>',
  dangerouslyUseHTMLString: true
})
```

### MessageBox 弹框

```typescript
import { ElMessageBox } from 'element-plus'

// 确认弹框
ElMessageBox.confirm('确定要删除吗？', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  type: 'warning'
}).then(() => {
  // 确定
}).catch(() => {
  // 取消
})

// 提示弹框
ElMessageBox.alert('这是一条消息', '提示', {
  confirmButtonText: '确定'
})

// 输入弹框
ElMessageBox.prompt('请输入用户名', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消'
}).then(({ value }) => {
  console.log('输入:', value)
})

// 自定义内容
ElMessageBox({
  title: '消息',
  message: h('p', null, [
    h('span', null, '消息可以是 '),
    h('i', { style: 'color: teal' }, 'VNode')
  ]),
  showCancelButton: true,
  confirmButtonText: '确定',
  cancelButtonText: '取消'
})
```

---

## 图标使用

### 引入图标

```typescript
import { ref } from 'vue'
import {
  Plus,
  Edit,
  Delete,
  Search,
  Refresh,
  Download,
  Upload,
  Setting,
  User,
  Lock,
  Message
} from '@element-plus/icons-vue'
```

### 使用图标

```vue
<template>
  <!-- 在按钮中使用 -->
  <el-button :icon="Plus">新增</el-button>

  <!-- 独立使用 -->
  <el-icon :size="20" color="#409eff">
    <Search />
  </el-icon>

  <!-- 自定义大小和颜色 -->
  <el-icon :size="30" color="red">
    <Delete />
  </el-icon>
</template>

<script setup lang="ts">
import { Plus, Search, Delete } from '@element-plus/icons-vue'
</script>
```

---

## 响应式布局

### 栅格布局

```vue
<template>
  <el-row :gutter="20">
    <el-col :span="8">
      <div class="grid-content">占8列</div>
    </el-col>
    <el-col :span="8">
      <div class="grid-content">占8列</div>
    </el-col>
    <el-col :span="8">
      <div class="grid-content">占8列</div>
    </el-col>
  </el-row>

  <el-row :gutter="20">
    <el-col :xs="24" :sm="12" :md="8" :lg="6">
      <div class="grid-content">响应式</div>
    </el-col>
  </el-row>
</template>

<style scoped>
.grid-content {
  background: #f0f2f5;
  padding: 20px;
  text-align: center;
  border-radius: 4px;
}
</style>
```

### 响应式表格

```vue
<template>
  <el-table :data="tableData">
    <el-table-column
      prop="name"
      label="姓名"
      min-width="120"
    />
    <el-table-column
      prop="age"
      label="年龄"
      width="100"
      class-name="hidden-xs-only"
    />
    <el-table-column
      prop="email"
      label="邮箱"
      min-width="180"
      class-name="hidden-sm-and-down"
    />
  </el-table>
</template>

<style>
/* Element Plus 响应式断点 */
@media screen and (max-width: 768px) {
  .hidden-xs-only {
    display: none !important;
  }
}

@media screen and (max-width: 992px) {
  .hidden-sm-and-down {
    display: none !important;
  }
}
</style>
```

---

## 🔗 相关文档

- [组件设计规范](./component-design.md) - 组件设计详细规范
- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [API调用规范](./api-calling.md) - API调用详细规范

---

**最后更新**：2026-03-12