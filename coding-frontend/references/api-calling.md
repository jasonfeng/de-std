# API调用规范

> 本规范定义API调用的使用规范，基于 `src/utils/request.ts` 实现，包括axios配置、拦截器、错误处理和API模块化组织。

---

## 📋 目录

- [axios基础配置](#axios基础配置)
- [请求/响应拦截器](#请求响应拦截器)
- [错误处理机制](#错误处理机制)
- [API模块化组织](#api模块化组织)
- [请求类型定义](#请求类型定义)
- [请求取消与重试](#请求取消与重试)
- [文件上传下载](#文件上传下载)
- [常见场景示例](#常见场景示例)

---

## axios基础配置

### request.ts配置

```typescript
// src/utils/request.ts
import axios from 'axios'
import type { AxiosRequestConfig, AxiosResponse, InternalAxiosRequestConfig } from 'axios'
import { ElMessage } from 'element-plus'

// 创建 axios 实例
const instance = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8080',
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json'
  }
})

export default instance
```

### 环境变量配置

```typescript
// .env.development
VITE_API_URL=http://localhost:8080

// .env.production
VITE_API_URL=https://api.example.com

// .env.staging
VITE_API_URL=https://staging-api.example.com
```

---

## 请求/响应拦截器

### 请求拦截器

```typescript
// 请求拦截器
instance.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    // 添加 token 到请求头
    const token = localStorage.getItem('token')
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`
    }

    // 添加请求时间戳
    config.params = {
      ...config.params,
      _t: Date.now()
    }

    // 请求开始时间
    config.metadata = { startTime: Date.now() }

    return config
  },
  (error) => {
    console.error('请求错误:', error)
    return Promise.reject(error)
  }
)
```

### 响应拦截器

```typescript
// 响应拦截器
instance.interceptors.response.use(
  (response: AxiosResponse) => {
    const { code, message, data } = response.data

    // 类型守卫：验证响应结构
    if (typeof code !== 'number' || typeof message !== 'string') {
      console.error('Invalid response structure:', response.data)
      ElMessage.error('服务器响应格式错误')
      return Promise.reject(new Error('服务器响应格式错误'))
    }

    // 根据业务状态码处理
    if (code === 200) {
      return response.data
    } else {
      ElMessage.error(message || '请求失败')
      return Promise.reject(new Error(message || '请求失败'))
    }
  },
  async (error) => {
    // Token 过期处理
    if (error.response?.status === 401) {
      // 处理token刷新逻辑
      await handleTokenExpired(error)
    }

    // 错误处理
    const errorMessage = getErrorMessage(error)
    ElMessage.error(errorMessage)

    return Promise.reject(error)
  }
)
```

---

## 错误处理机制

### Token刷新机制

```typescript
// Token 刷新状态
let isRefreshing = false
let refreshSubscribers: ((token: string) => void)[] = []

/**
 * 添加刷新订阅
 */
function subscribeTokenRefresh(callback: (token: string) => void) {
  refreshSubscribers.push(callback)
}

/**
 * 通知所有订阅者
 */
function onRefreshed(token: string) {
  refreshSubscribers.forEach(callback => callback(token))
  refreshSubscribers = []
}

/**
 * 刷新 Token
 */
async function refreshToken(): Promise<string | null> {
  try {
    const refreshToken = localStorage.getItem('refreshToken')
    if (!refreshToken) {
      return null
    }

    const response = await axios.post(`${import.meta.env.VITE_API_URL}/api/v1/auth/refresh`, {
      refreshToken
    })

    const { token, refreshToken: newRefreshToken } = response.data.data

    localStorage.setItem('token', token)
    if (newRefreshToken) {
      localStorage.setItem('refreshToken', newRefreshToken)
    }

    return token
  } catch (error) {
    console.error('刷新Token失败:', error)
    localStorage.removeItem('token')
    localStorage.removeItem('refreshToken')
    window.location.href = '/login'
    return null
  }
}

/**
 * 处理Token过期
 */
async function handleTokenExpired(error: any) {
  const originalRequest = error.config

  if (!originalRequest._retry) {
    if (isRefreshing) {
      return new Promise((resolve) => {
        subscribeTokenRefresh((token: string) => {
          originalRequest.headers.Authorization = `Bearer ${token}`
          resolve(instance(originalRequest))
        })
      })
    }

    originalRequest._retry = true
    isRefreshing = true

    try {
      const newToken = await refreshToken()
      if (newToken) {
        onRefreshed(newToken)
        originalRequest.headers.Authorization = `Bearer ${newToken}`
        return instance(originalRequest)
      }
    } catch (refreshError) {
      return Promise.reject(refreshError)
    } finally {
      isRefreshing = false
    }
  }
}
```

### 错误消息提取

```typescript
/**
 * 获取错误消息
 */
function getErrorMessage(error: any): string {
  // 响应错误
  if (error.response) {
    const { status, data } = error.response

    // 自定义错误消息
    if (data?.message) {
      return data.message
    }

    // HTTP状态码映射
    const statusMessages: Record<number, string> = {
      400: '请求参数错误',
      401: '未授权，请登录',
      403: '拒绝访问',
      404: '请求资源不存在',
      409: '数据冲突',
      422: '验证失败',
      429: '请求过于频繁，请稍后再试',
      500: '服务器内部错误',
      502: '网关错误',
      503: '服务暂时不可用',
      504: '请求超时'
    }

    return statusMessages[status] || `请求失败 (${status})`
  }

  // 请求无响应
  if (error.request) {
    return '网络连接异常，请检查网络'
  }

  // 请求配置错误
  return error.message || '请求失败，请重试'
}
```

---

## API模块化组织

### 目录结构

```
src/api/
├── index.ts           # API入口
├── user.ts            # 用户API
├── department.ts      # 部门API
├── role.ts            # 角色API
├── permission.ts      # 权限API
└── types/             # API类型定义
    ├── user.ts
    ├── department.ts
    └── role.ts
```

### API模块示例

```typescript
// src/api/user.ts
import request from '@/utils/request'
import type {
  User,
  CreateUserParams,
  UpdateUserParams,
  PageResponse,
  UserQueryParams
} from '@/types/user'

const USER_API = '/api/v1/users'

/**
 * 用户 API
 */
export const userApi = {
  /**
   * 获取用户列表（分页）
   */
  getList: (params: UserQueryParams) => {
    return request.get<PageResponse<User>>(USER_API, { params })
  },

  /**
   * 根据ID获取用户详情
   */
  getById: (id: number) => {
    return request.get<User>(`${USER_API}/${id}`)
  },

  /**
   * 根据部门ID获取用户列表
   */
  getByDepartment: (departmentId: number) => {
    return request.get<User[]>(`${USER_API}/by-department/${departmentId}`)
  },

  /**
   * 检查用户名是否存在
   */
  checkExists: (username: string, departmentId: number) => {
    return request.get<boolean>(`${USER_API}/check-exists`, {
      params: { username, departmentId }
    })
  },

  /**
   * 创建用户
   */
  create: (data: CreateUserParams) => {
    return request.post<User>(USER_API, data)
  },

  /**
   * 更新用户
   */
  update: (id: number, data: UpdateUserParams) => {
    return request.put<User>(`${USER_API}/${id}`, data)
  },

  /**
   * 删除用户（软删除）
   */
  delete: (id: number) => {
    return request.delete<void>(`${USER_API}/${id}`)
  },

  /**
   * 批量删除用户
   */
  batchDelete: (ids: number[]) => {
    return request.delete<void>(USER_API, { data: { ids } })
  }
}

export default userApi
```

### API入口文件

```typescript
// src/api/index.ts
export { userApi } from './user'
export { departmentApi } from './department'
export { roleApi } from './role'
export { permissionApi } from './permission'

// 统一导出
export const api = {
  user: import('./user').then(m => m.userApi),
  department: import('./department').then(m => m.departmentApi),
  role: import('./role').then(m => m.roleApi),
  permission: import('./permission').then(m => m.permissionApi)
}
```

---

## 请求类型定义

### 通用类型

```typescript
// src/types/common.ts

/**
 * 分页响应
 */
export interface PageResponse<T> {
  /** 当前页 */
  current: number
  /** 每页数量 */
  size: number
  /** 总数 */
  total: number
  /** 数据列表 */
  records: T[]
  /** 总页数 */
  pages: number
}

/**
 * 分页查询参数
 */
export interface PageQueryParams {
  /** 当前页 */
  current: number
  /** 每页数量 */
  size: number
}

/**
 * API响应
 */
export interface ApiResponse<T = any> {
  /** 状态码 */
  code: number
  /** 消息 */
  message: string
  /** 数据 */
  data: T
}

/**
 * 排序参数
 */
export interface SortParams {
  /** 排序字段 */
  field: string
  /** 排序方式 */
  order: 'asc' | 'desc'
}

/**
 * 查询参数
 */
export interface QueryParams extends PageQueryParams {
  /** 搜索关键词 */
  keyword?: string
  /** 排序 */
  sort?: SortParams
  /** 过滤条件 */
  filters?: Record<string, any>
}
```

### 响应数据类型

```typescript
// src/types/response.ts

/**
 * 成功响应
 */
export interface SuccessResponse<T = any> {
  code: 200
  message: string
  data: T
}

/**
 * 错误响应
 */
export interface ErrorResponse {
  code: number
  message: string
  data?: any
  errors?: ValidationError[]
}

/**
 * 验证错误
 */
export interface ValidationError {
  field: string
  message: string
  value?: any
}
```

---

## 请求取消与重试

### 请求取消

```typescript
// src/utils/request.ts

// 取消令牌存储
const pendingRequests = new Map<string, AbortController>()

/**
 * 生成请求key
 */
function getRequestKey(config: AxiosRequestConfig): string {
  const { method, url, params, data } = config
  return `${method}:${url}:${JSON.stringify(params)}:${JSON.stringify(data)}`
}

/**
 * 取消重复请求
 */
function cancelDuplicateRequest(config: AxiosRequestConfig) {
  const key = getRequestKey(config)
  if (pendingRequests.has(key)) {
    const controller = pendingRequests.get(key)!
    controller.abort()
    pendingRequests.delete(key)
  }

  // 添加新的取消控制器
  const controller = new AbortController()
  config.signal = controller.signal
  pendingRequests.set(key, controller)
}

/**
 * 清理请求
 */
function clearRequest(config: AxiosRequestConfig) {
  const key = getRequestKey(config)
  pendingRequests.delete(key)
}

// 在拦截器中使用
instance.interceptors.request.use(
  (config) => {
    cancelDuplicateRequest(config)
    return config
  }
)

instance.interceptors.response.use(
  (response) => {
    clearRequest(response.config)
    return response
  },
  (error) => {
    if (!axios.isCancel(error)) {
      clearRequest(error.config)
    }
    return Promise.reject(error)
  }
)
```

### 请求重试

```typescript
// src/utils/retry.ts
import type { AxiosRequestConfig, AxiosError } from 'axios'

interface RetryConfig {
  /** 重试次数 */
  retries: number
  /** 重试延迟（毫秒） */
  retryDelay: number
  /** 是否重试的条件 */
  shouldRetry?: (error: AxiosError) => boolean
}

/**
 * 请求重试包装器
 */
export async function requestWithRetry<T>(
  requestFn: () => Promise<T>,
  config: RetryConfig = { retries: 3, retryDelay: 1000 }
): Promise<T> {
  const { retries, retryDelay, shouldRetry } = config

  let lastError: any

  for (let i = 0; i < retries; i++) {
    try {
      return await requestFn()
    } catch (error) {
      lastError = error

      // 检查是否应该重试
      if (shouldRetry && !shouldRetry(error)) {
        throw error
      }

      // 最后一次重试不再延迟
      if (i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, retryDelay))
      }
    }
  }

  throw lastError
}

// 使用示例
const response = await requestWithRetry(
  () => request.get('/api/data'),
  {
    retries: 3,
    retryDelay: 1000,
    shouldRetry: (error) => {
      // 只重试网络错误和5xx错误
      return !error.response || error.response.status >= 500
    }
  }
)
```

---

## 文件上传下载

### 文件上传

```typescript
// src/api/upload.ts
import request from '@/utils/request'

const UPLOAD_API = '/api/v1/upload'

export const uploadApi = {
  /**
   * 上传文件
   */
  upload: (file: File, onProgress?: (percent: number) => void) => {
    const formData = new FormData()
    formData.append('file', file)

    return request.post<{ url: string; filename: string }>(UPLOAD_API, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      },
      onUploadProgress: (progressEvent) => {
        if (onProgress && progressEvent.total) {
          const percent = Math.round((progressEvent.loaded / progressEvent.total) * 100)
          onProgress(percent)
        }
      }
    })
  },

  /**
   * 批量上传文件
   */
  batchUpload: (files: File[], onProgress?: (percent: number) => void) => {
    const formData = new FormData()
    files.forEach(file => {
      formData.append('files', file)
    })

    return request.post<{ urls: string[]; filenames: string[] }>(
      `${UPLOAD_API}/batch`,
      formData,
      {
        headers: {
          'Content-Type': 'multipart/form-data'
        },
        onUploadProgress: (progressEvent) => {
          if (onProgress && progressEvent.total) {
            const percent = Math.round((progressEvent.loaded / progressEvent.total) * 100)
            onProgress(percent)
          }
        }
      }
    )
  }
}
```

### 文件下载

```typescript
// src/api/download.ts
import request from '@/utils/request'

export const downloadApi = {
  /**
   * 下载文件
   */
  download: (url: string, filename?: string) => {
    return request.get(url, {
      responseType: 'blob'
    }).then((response: any) => {
      // 创建下载链接
      const blob = new Blob([response.data])
      const downloadUrl = window.URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href = downloadUrl
      link.download = filename || 'download'
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      window.URL.revokeObjectURL(downloadUrl)
    })
  },

  /**
   * 导出数据
   */
  export: (params: any, filename = 'export.xlsx') => {
    return request.get('/api/v1/export', {
      params,
      responseType: 'blob'
    }).then((response: any) => {
      const blob = new Blob([response.data])
      const downloadUrl = window.URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href = downloadUrl
      link.download = filename
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      window.URL.revokeObjectURL(downloadUrl)
    })
  }
}
```

---

## 常见场景示例

### 在组件中使用API

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { userApi } from '@/api/user'

const loading = ref(false)
const users = ref<User[]>([])

// 获取用户列表
const fetchUsers = async () => {
  loading.value = true
  try {
    const response = await userApi.getList({
      current: 1,
      size: 10
    })
    users.value = response.records
  } catch (error) {
    console.error('获取用户列表失败:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchUsers()
})
</script>
```

### 使用requestWithRetry

```typescript
import { requestWithRetry } from '@/utils/retry'

const fetchData = async () => {
  try {
    const response = await requestWithRetry(
      () => request.get('/api/unstable-endpoint'),
      {
        retries: 3,
        retryDelay: 1000,
        shouldRetry: (error) => {
          // 只重试5xx错误
          return error.response?.status >= 500
        }
      }
    )
    return response
  } catch (error) {
    console.error('请求失败:', error)
    throw error
  }
}
```

### 取消请求

```typescript
// 在组件中使用
import { onBeforeUnmount } from 'vue'

let controller: AbortController | null = null

const fetchData = async () => {
  controller = new AbortController()
  try {
    const response = await request.get('/api/data', {
      signal: controller.signal
    })
    return response
  } catch (error) {
    if (axios.isCancel(error)) {
      console.log('请求已取消')
    }
  }
}

onBeforeUnmount(() => {
  // 组件卸载时取消请求
  if (controller) {
    controller.abort()
  }
})
```

---

## ⚠️ Mock数据规范（功能完成标准）

### 核心规则

| 规则 | 说明 | 违反后果 |
|------|------|---------|
| **禁止计划外Mock数据** | 所有页面数据必须从后端API获取，禁止在组件中硬编码Mock数据 | 功能视为未完成 |
| **Mock数据需明确标注** | 开发阶段如需Mock数据，必须在Story计划中明确标注原因和期限 | 代码审查失败 |
| **上线前必须移除Mock** | 所有Mock数据必须在功能上线前替换为真实API调用 | 发布阻塞 |

### ❌ 错误示例（禁止）

```vue
<script setup lang="ts">
// ❌ 错误：硬编码Mock数据
const users = ref([
  { id: 1, name: '张三', email: 'zhangsan@example.com' },
  { id: 2, name: '李四', email: 'lisi@example.com' }
])
</script>
```

### ✅ 正确示例

```vue
<script setup lang="ts">
import { userApi } from '@/api/user'

const loading = ref(false)
const users = ref<User[]>([])

// ✅ 正确：从后端API获取数据
const fetchUsers = async () => {
  loading.value = true
  try {
    const response = await userApi.getList({ current: 1, size: 10 })
    users.value = response.records
  } catch (error) {
    console.error('获取用户列表失败:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchUsers()
})
</script>
```

### Mock数据的使用场景（需明确标注）

以下情况**允许**使用Mock数据，但必须在Story计划中明确标注：

| 场景 | 标注要求 | 示例 |
|------|---------|------|
| 后端API未就绪 | 标注API名称、预计就绪时间 | `// TODO: Mock数据，等待 /api/v1/users 接口，预计 2026-03-25` |
| 复杂场景演示 | 标注演示目的、演示期限 | `// Mock: 用于演示批量导入流程，演示后移除` |
| 前端独立开发 | 标注开发阶段、替换计划 | `// DEV: 前端独立开发阶段，集成测试时替换为真实API` |

### 功能完成检查清单

每个前端功能完成时，必须确认：

- [ ] 所有数据均从后端API获取（或Mock数据已在计划中标注）
- [ ] 无计划外的硬编码Mock数据
- [ ] Mock数据已添加TODO注释，标注替换计划
- [ ] API调用遵循统一规范（使用 `src/api/*.ts` 模块）

---

## 🔗 相关文档

- [路由规范](./routing.md) - 路由详细规范
- [TypeScript规范](./typescript.md) - TypeScript详细规范
- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范

---

**最后更新**：2026-03-22