# 错误监控规范

> 本规范定义前端错误监控的使用规范，包括Sentry集成、错误边界、性能监控和用户反馈收集。

---

## 📋 目录

- [错误监控原则](#错误监控原则)
- [Sentry集成](#sentry集成)
- [错误边界](#错误边界)
- [错误处理](#错误处理)
- [性能监控](#性能监控)
- [用户反馈](#用户反馈)
- [最佳实践](#最佳实践)

---

## 错误监控原则

### 及时捕获

```typescript
// ✅ 推荐：及时捕获错误
try {
  await riskyOperation()
} catch (error) {
  captureError(error)
}
```

### 完整上下文

```typescript
// ✅ 推荐：提供完整上下文
captureException(error, {
  user: userStore.userInfo,
  extra: {
    url: window.location.href,
    userAgent: navigator.userAgent
  }
})
```

### 用户友好

```typescript
// ✅ 推荐：显示用户友好的错误消息
handleError(error) {
  showUserFriendlyMessage(error)
  captureError(error)
}
```

---

## Sentry集成

### 安装Sentry

```bash
npm install @sentry/vue @sentry/tracing
```

### 基础配置

```typescript
// src/main.ts
import * as Sentry from '@sentry/vue'
import { BrowserTracing } from '@sentry/tracing'

if (import.meta.env.PROD) {
  Sentry.init({
    app,
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,
    release: import.meta.env.VITE_APP_VERSION,

    // 性能监控
    integrations: [
      new BrowserTracing({
        tracePropagationTargets: [
          'localhost',
          import.meta.env.VITE_API_URL
        ]
      })
    ],

    // 采样率
    tracesSampleRate: 0.1, // 性能采样率10%
    replaysSessionSampleRate: 0.1, // 回放采样率10%
    replaysOnErrorSampleRate: 1.0, // 错误回放采样率100%,

    // 过滤敏感信息
    beforeSend(event, hint) {
      return filterSensitiveData(event)
    }
  })
}
```

### 敏感数据过滤

```typescript
// src/utils/sentry.ts
/**
 * 过滤敏感数据
 */
export function filterSensitiveData(event: any): any {
  if (!event.request) return event

  // 过滤请求headers中的敏感信息
  if (event.request.headers) {
    const sensitiveHeaders = ['authorization', 'cookie', 'set-cookie']
    for (const header of sensitiveHeaders) {
      delete event.request.headers[header]
    }
  }

  // 过滤用户数据中的敏感信息
  if (event.user) {
    const sensitiveFields = ['password', 'token', 'apiKey', 'secret']
    for (const field of sensitiveFields) {
      delete event.user[field]
    }
  }

  return event
}

/**
 * 自定义用户信息
 */
export function setUserContext(user: any) {
  Sentry.setUser({
    id: user.id,
    username: user.username,
    email: user.email
  })
}

/**
 * 清除用户信息
 */
export function clearUserContext() {
  Sentry.setUser(null)
}
```

### 错误上报

```typescript
// src/utils/sentry.ts
/**
 * 捕获异常
 */
export function captureException(
  error: Error,
  context?: {
    level?: 'fatal' | 'error' | 'warning' | 'info' | 'debug'
    tags?: Record<string, string>
    extra?: Record<string, any>
  }
) {
  if (import.meta.env.DEV) {
    console.error('Error:', error)
    return
  }

  Sentry.withScope((scope) => {
    if (context?.level) {
      scope.setLevel(context.level)
    }
    if (context?.tags) {
      scope.setTags(context.tags)
    }
    if (context?.extra) {
      scope.setExtras(context.extra)
    }

    Sentry.captureException(error)
  })
}

/**
 * 捕获消息
 */
export function captureMessage(
  message: string,
  context?: {
    level?: 'fatal' | 'error' | 'warning' | 'info' | 'debug'
    tags?: Record<string, string>
    extra?: Record<string, any>
  }
) {
  if (import.meta.env.DEV) {
    console.log('Message:', message)
    return
  }

  Sentry.withScope((scope) => {
    if (context?.level) {
      scope.setLevel(context.level)
    }
    if (context?.tags) {
      scope.setTags(context.tags)
    }
    if (context?.extra) {
      scope.setExtras(context.extra)
    }

    Sentry.captureMessage(message)
  })
}

/**
 * 添加面包屑
 */
export function addBreadcrumb(breadcrumb: {
  category: string
  message: string
  level?: 'fatal' | 'error' | 'warning' | 'info' | 'debug'
  data?: Record<string, any>
}) {
  Sentry.addBreadcrumb({
    ...breadcrumb,
    timestamp: Date.now()
  })
}
```

---

## 错误边界

### Vue错误边界组件

```vue
<!-- src/components/ErrorBoundary.vue -->
<template>
  <slot v-if="!error" />
  <div v-else class="error-boundary">
    <div class="error-icon">😢</div>
    <h2>出错了</h2>
    <p>抱歉，遇到了一些问题</p>
    <el-button type="primary" @click="handleReload">
      重新加载
    </el-button>
    <el-button @click="handleReport">
      反馈问题
    </el-button>
  </div>
</template>

<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'
import { captureException } from '@/utils/sentry'

const error = ref<Error | null>(null)

onErrorCaptured((err: Error) => {
  error.value = err
  captureException(err, {
    tags: { component: 'ErrorBoundary' }
  })
  // 返回false阻止错误继续向上传播
  return false
})

const handleReload = () => {
  window.location.reload()
}

const handleReport = () => {
  // 打开反馈表单
  window.open('https://github.com/xxx/issues/new')
}
</script>

<style scoped>
.error-boundary {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  text-align: center;
}

.error-icon {
  font-size: 64px;
  margin-bottom: 20px;
}

h2 {
  font-size: 24px;
  margin-bottom: 10px;
}

p {
  color: #666;
  margin-bottom: 20px;
}
</style>
```

### 全局错误处理

```typescript
// src/main.ts
app.config.errorHandler = (err, instance, info) => {
  console.error('Global error:', err, info)

  // 上报到Sentry
  captureException(err, {
    tags: {
      vueError: true,
      errorInfo: info
    },
    extra: {
      componentName: instance?.$options.name || 'Unknown'
    }
  })
}

// 捕获未处理的Promise错误
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason)

  captureException(event.reason, {
    tags: { unhandledRejection: true }
  })
})

// 捕获全局错误
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error)

  captureException(event.error, {
    tags: { globalError: true }
  })
})
```

---

## 错误处理

### API错误处理

```typescript
// src/utils/errorHandler.ts
import { ElMessage } from 'element-plus'
import type { HumanizedErrorResponse } from '@/types/error'
import { captureException } from '@/utils/sentry'

/**
 * 处理API错误
 */
export function handleApiError(error: any): HumanizedErrorResponse | null {
  // 添加面包屑
  addBreadcrumb({
    category: 'api',
    message: 'API Error',
    data: {
      url: error.config?.url,
      method: error.config?.method,
      status: error.response?.status
    }
  })

  // 上报到Sentry
  if (error.response?.status >= 500) {
    captureException(error, {
      tags: { errorType: 'api-error' }
    })
  }

  // 返回人性化错误
  return ErrorHandler.handleApiError(error)
}

/**
 * 处理网络错误
 */
export function handleNetworkError(error: any) {
  addBreadcrumb({
    category: 'network',
    message: 'Network Error',
    data: {
      message: error.message
    }
  })

  ElMessage.error('网络连接异常，请检查网络')

  if (import.meta.env.PROD) {
    captureException(error, {
      tags: { errorType: 'network-error' }
    })
  }
}
```

### 表单错误处理

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { captureException } from '@/utils/sentry'

const error = ref<string | null>(null)

const handleSubmit = async () => {
  try {
    error.value = null
    await submitForm()
  } catch (err) {
    error.value = '提交失败，请重试'
    captureException(err, {
      tags: { operation: 'form-submit' }
    })
  }
}
</script>
```

---

## 性能监控

### Web Vitals监控

```typescript
// src/utils/performance.ts
import { onCLS, onFID, onLCP, onTTFB, onFCP } from 'web-vitals'
import * as Sentry from '@sentry/vue'

/**
 * 监控Web Vitals
 */
export function monitorWebVitals() {
  // 累积布局偏移（CLS）
  onCLS((metric) => {
    sendMetricToSentry('CLS', metric)
  })

  // 首次输入延迟（FID）
  onFID((metric) => {
    sendMetricToSentry('FID', metric)
  })

  // 最大内容绘制（LCP）
  onLCP((metric) => {
    sendMetricToSentry('LCP', metric)
  })

  // 首字节时间（TTFB）
  onTTFB((metric) => {
    sendMetricToSentry('TTFB', metric)
  })

  // 首次内容绘制（FCP）
  onFCP((metric) => {
    sendMetricToSentry('FCP', metric)
  })
}

/**
 * 发送指标到Sentry
 */
function sendMetricToSentry(name: string, metric: any) {
  if (import.meta.env.DEV) {
    console.log(`${name}:`, metric.value)
    return
  }

  Sentry.metrics.gauge(name, metric.value, {
    tags: {
      id: metric.id,
      rating: metric.rating
    }
  })
}
```

### 自定义性能监控

```typescript
// src/utils/performance.ts
/**
 * 监控函数执行时间
 */
export function monitorFunction<T>(
  name: string,
  fn: () => Promise<T>
): Promise<T> {
  const startTime = performance.now()

  return fn()
    .then((result) => {
      const duration = performance.now() - startTime
      sendMetricToSentry(`function.${name}`, { value: duration })
      return result
    })
    .catch((error) => {
      const duration = performance.now() - startTime
      captureException(error, {
        tags: { function: name, duration: duration.toString() }
      })
      throw error
    })
}

/**
 * 监控API请求性能
 */
export function monitorApiRequest(url: string, fn: () => Promise<any>) {
  const startTime = performance.now()

  return fn()
    .then((result) => {
      const duration = performance.now() - startTime
      addBreadcrumb({
        category: 'performance',
        message: `API Request: ${url}`,
        data: { duration: duration.toFixed(2) + 'ms' }
      })
      return result
    })
    .catch((error) => {
      const duration = performance.now() - startTime
      addBreadcrumb({
        category: 'performance',
        message: `API Error: ${url}`,
        level: 'error',
        data: { duration: duration.toFixed(2) + 'ms' }
      })
      throw error
    })
}
```

---

## 用户反馈

### 反馈组件

```vue
<!-- src/components/FeedbackDialog.vue -->
<template>
  <el-dialog
    v-model="visible"
    title="问题反馈"
    width="600px"
  >
    <el-form :model="form" label-width="80px">
      <el-form-item label="问题类型">
        <el-select v-model="form.type">
          <el-option label="功能问题" value="feature" />
          <el-option label="Bug" value="bug" />
          <el-option label="性能问题" value="performance" />
          <el-option label="其他" value="other" />
        </el-select>
      </el-form-item>

      <el-form-item label="问题描述">
        <el-input
          v-model="form.description"
          type="textarea"
          :rows="4"
          placeholder="请描述您遇到的问题"
        />
      </el-form-item>

      <el-form-item label="页面地址">
        <el-input v-model="form.url" disabled />
      </el-form-item>
    </el-form>

    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="handleSubmit" :loading="submitting">
        提交
      </el-button>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { ElMessage } from 'element-plus'
import { captureException, setUserContext } from '@/utils/sentry'

const visible = defineModel<boolean>()
const submitting = ref(false)

const form = reactive({
  type: 'bug',
  description: '',
  url: window.location.href
})

const handleSubmit = async () => {
  submitting.value = true

  try {
    // 发送反馈到服务器
    await api.post('/api/v1/feedback', form)

    // 上报到Sentry
    captureMessage('用户反馈', {
      level: 'info',
      extra: {
        type: form.type,
        description: form.description,
        url: form.url
      }
    })

    ElMessage.success('反馈已提交')
    visible.value = false
  } catch (error) {
    ElMessage.error('提交失败')
    captureException(error)
  } finally {
    submitting.value = false
  }
}
</script>
```

### 反馈按钮

```vue
<template>
  <el-button
    :icon="Message"
    circle
    @click="showFeedback"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Message } from '@element-plus/icons-vue'

const feedbackVisible = ref(false)

const showFeedback = () => {
  feedbackVisible.value = true
}
</script>
```

---

## 最佳实践

### 1. 区分开发和生产环境

```typescript
if (import.meta.env.DEV) {
  // 开发环境：输出到控制台
  console.error(error)
} else {
  // 生产环境：上报到Sentry
  captureException(error)
}
```

### 2. 提供完整的上下文

```typescript
captureException(error, {
  tags: {
    feature: 'user-management',
    action: 'create-user'
  },
  extra: {
    userId: currentUser.id,
    formData: sanitizeFormData(formData)
  }
})
```

### 3. 用户友好的错误提示

```typescript
try {
  await riskyOperation()
} catch (error) {
  // 显示用户友好的错误
  ElMessage.error('操作失败，请重试')
  // 上报详细错误
  captureException(error)
}
```

### 4. 错误恢复机制

```typescript
const retryCount = ref(0)
const maxRetries = 3

const fetchData = async () => {
  try {
    const data = await api.get('/api/v1/data')
    return data
  } catch (error) {
    if (retryCount.value < maxRetries) {
      retryCount.value++
      return fetchData()
    }
    throw error
  }
}
```

### 5. 性能监控与错误监控结合

```typescript
export async function safeExecute<T>(
  name: string,
  fn: () => Promise<T>
): Promise<T> {
  const startTime = performance.now()

  try {
    return await fn()
  } catch (error) {
    const duration = performance.now() - startTime
    captureException(error, {
      extra: {
        function: name,
        duration: duration.toFixed(2) + 'ms'
      }
    })
    throw error
  }
}
```

---

## 🔗 相关文档

- [API调用规范](./api-calling.md) - API调用详细规范
- [性能优化规范](./performance.md) - 性能优化详细规范
- [安全规范](./security.md) - 安全详细规范

---

**最后更新**：2026-03-12