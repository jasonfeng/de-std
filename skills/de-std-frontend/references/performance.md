# 性能优化规范

> 本规范定义前端性能优化的使用规范，包括代码分割、懒加载、资源优化和渲染优化。

---

## 📋 目录

- [代码分割](#代码分割)
- [路由懒加载](#路由懒加载)
- [资源优化](#资源优化)
- [渲染优化](#渲染优化)
- [网络优化](#网络优化)
- [内存优化](#内存优化)
- [性能监控](#性能监控)
- [最佳实践](#最佳实践)

---

## 代码分割

### Vite自动分割

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // 手动分包
        manualChunks: {
          // Vue核心
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          // UI框架
          'element-plus': ['element-plus'],
          // 图表库
          'echarts': ['echarts'],
          // 工具库
          'utils': ['axios', 'lodash-es']
        }
      }
    }
  }
})
```

### 动态导入

```typescript
// ❌ 静态导入（不推荐）
import { heavyFunction } from './heavy-utils'

// ✅ 动态导入（推荐）
const heavyFunction = () => import('./heavy-utils')

// 使用时才加载
const handleHeavyTask = async () => {
  const module = await heavyFunction()
  module.heavyFunction()
}
```

### 条件导入

```typescript
// 条件导入第三方库
if (import.meta.env.DEV) {
  const DevTools = await import('@vue/devtools')
  DevTools.default.connect()
}

// 功能开关导入
if (import.meta.env.VITE_ENABLE_ANALYTICS === 'true') {
  const Analytics = await import('./analytics')
  Analytics.init()
}
```

---

## 路由懒加载

### 基础懒加载

```typescript
// router/index.ts
const routes = [
  {
    path: '/',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/users',
    component: () => import('@/views/users/UserList.vue')
  }
]
```

### 分组懒加载

```typescript
// router/index.ts
// 按模块分组
const systemRoutes = [
  {
    path: '/system/users',
    component: () =>
      import(
        /* webpackChunkName: "system" */
        '@/views/system/Users.vue'
      )
  },
  {
    path: '/system/roles',
    component: () =>
      import(
        /* webpackChunkName: "system" */
        '@/views/system/Roles.vue'
      )
  }
]

const qualityRoutes = [
  {
    path: '/quality/rules',
    component: () =>
      import(
        /* webpackChunkName: "quality" */
        '@/views/quality/Rules.vue'
      )
  }
]
```

### 预加载策略

```typescript
// router/index.ts
const routes = [
  {
    path: '/',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/dashboard',
    // 预加载高优先级路由
    component: () =>
      import(
        /* webpackPrefetch: true */
        '@/views/Dashboard.vue'
      )
  },
  {
    path: '/settings',
    // 预获取低优先级路由
    component: () =>
      import(
        /* webpackPreload: true */
        '@/views/Settings.vue'
      )
  }
]
```

---

## 资源优化

### 图片优化

```vue
<template>
  <!-- 使用现代图片格式 -->
  <picture>
    <source srcset="@/assets/image.webp" type="image/webp">
    <source srcset="@/assets/image.avif" type="image/avif">
    <img src="@/assets/image.jpg" alt="Image" loading="lazy">
  </picture>

  <!-- 懒加载 -->
  <img v-lazy="imageUrl" alt="Image">

  <!-- 响应式图片 -->
  <img
    srcset="image-320w.jpg 320w,
            image-640w.jpg 640w,
            image-1280w.jpg 1280w"
    sizes="(max-width: 640px) 320px,
           (max-width: 1280px) 640px,
           1280px"
    src="image-1280w.jpg"
    alt="Responsive Image"
  >
</template>
```

### 字体优化

```css
/* 使用字体子集化 */
@font-face {
  font-family: 'CustomFont';
  src: url('custom-font-subset.woff2') format('woff2');
  font-display: swap; /* 字体交换策略 */
}

/* 预加载关键字体 */
<link rel="preload" href="/fonts/custom-font.woff2" as="font" type="font/woff2" crossorigin>
```

### CSS优化

```css
/* 使用CSS变量代替重复值 */
:root {
  --primary-color: #4066E5;
  --spacing-unit: 8px;
}

/* 避免过度嵌套 */
/* ❌ 不推荐 */
.parent .child .grandchild .text {
  color: var(--primary-color);
}

/* ✅ 推荐 */
.text {
  color: var(--primary-color);
}

/* 使用will-change优化动画 */
.animated-element {
  will-change: transform;
}

.animated-element:hover {
  transform: translateY(-4px);
}
```

---

## 渲染优化

### 虚拟滚动

```vue
<template>
  <VirtualList
    :data="largeList"
    :item-height="50"
    :visible-height="500"
  >
    <template #default="{ item }">
      <div class="list-item">{{ item.name }}</div>
    </template>
  </VirtualList>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { VirtualList } from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

const largeList = ref([...]) // 大量数据
</script>
```

### 列表渲染优化

```vue
<template>
  <!-- ❌ 不推荐：使用index作为key -->
  <div v-for="(item, index) in list" :key="index">
    {{ item.name }}
  </div>

  <!-- ✅ 推荐：使用唯一ID作为key -->
  <div v-for="item in list" :key="item.id">
    {{ item.name }}
  </div>

  <!-- ✅ 推荐：对于复杂列表，使用v-memo -->
  <div v-for="item in list" :key="item.id" v-memo="[item.id, item.name]">
    {{ item.name }}
  </div>
</template>
```

### 计算属性优化

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const items = ref<Item[]>([])

// ❌ 不推荐：每次都重新计算
const filteredItems = items.value.filter(item => item.active)

// ✅ 推荐：使用computed缓存
const filteredItems = computed(() =>
  items.value.filter(item => item.active)
)

// ✅ 推荐：复杂计算使用computed
const complexResult = computed(() => {
  let result = 0
  for (let i = 0; i < 10000; i++) {
    result += items.value[i]?.value || 0
  }
  return result
})
</script>
```

### 防抖和节流

```typescript
// utils/debounce.ts
export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timer: number | null = null

  return function (this: any, ...args: Parameters<T>) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => {
      fn.apply(this, args)
    }, delay)
  }
}

// utils/throttle.ts
export function throttle<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let lastTime = 0

  return function (this: any, ...args: Parameters<T>) {
    const now = Date.now()
    if (now - lastTime >= delay) {
      lastTime = now
      fn.apply(this, args)
    }
  }
}
```

### 使用示例

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { debounce } from '@/utils/debounce'

const searchQuery = ref('')

// 防抖搜索
const handleSearch = debounce((query: string) => {
  performSearch(query)
}, 300)
</script>
```

---

## 网络优化

### 请求合并

```typescript
// 批量请求
export async function batchFetch<T>(
  fetcher: (id: number) => Promise<T>,
  ids: number[],
  batchSize = 10
): Promise<T[]> {
  const results: T[] = []

  for (let i = 0; i < ids.length; i += batchSize) {
    const batch = ids.slice(i, i + batchSize)
    const batchResults = await Promise.all(
      batch.map(id => fetcher(id))
    )
    results.push(...batchResults)
  }

  return results
}
```

### 请求缓存

```typescript
// utils/cache.ts
const cache = new Map<string, { data: any; timestamp: number }>()
const CACHE_DURATION = 5 * 60 * 1000 // 5分钟

export function fetchWithCache<T>(
  key: string,
  fetcher: () => Promise<T>
): Promise<T> {
  const cached = cache.get(key)

  if (cached && Date.now() - cached.timestamp < CACHE_DURATION) {
    return Promise.resolve(cached.data)
  }

  return fetcher().then(data => {
    cache.set(key, { data, timestamp: Date.now() })
    return data
  })
}
```

### 数据预取

```vue
<script setup lang="ts">
import { onMounted, onBeforeMount } from 'vue'

// 在组件挂载前预取数据
onBeforeMount(async () => {
  // 预取用户信息
  prefetchUserInfo()
})

onMounted(() => {
  // 组件挂载后预取其他数据
  prefetchNotifications()
})
</script>
```

---

## 内存优化

### 组件销毁

```vue
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'
import { useUserStore } from '@/stores/user'

let timer: number | null = null
let eventListener: ((event: Event) => void) | null = null

onMounted(() => {
  // 设置定时器
  timer = setInterval(() => {
    // 定时任务
  }, 1000)

  // 添加事件监听
  eventListener = (event: Event) => {
    // 事件处理
  }
  window.addEventListener('resize', eventListener)
})

onUnmounted(() => {
  // 清理定时器
  if (timer) {
    clearInterval(timer)
  }

  // 移除事件监听
  if (eventListener) {
    window.removeEventListener('resize', eventListener)
  }

  // 清理store
  const userStore = useUserStore()
  userStore.$dispose()
})
</script>
```

### 大数据优化

```typescript
// 分页处理大数据
export function processLargeData<T>(
  data: T[],
  processor: (chunk: T[]) => void,
  chunkSize = 1000
) {
  for (let i = 0; i < data.length; i += chunkSize) {
    const chunk = data.slice(i, i + chunkSize)
    processor(chunk)
  }
}

// 使用示例
processLargeData(largeArray, (chunk) => {
  // 处理数据块
  console.log(chunk.length)
})
```

---

## 性能监控

### Performance API

```typescript
// utils/performance.ts
export function measurePerformance(
  name: string,
  fn: () => void | Promise<void>
) {
  const start = performance.now()

  const cleanup = () => {
    const end = performance.now()
    const duration = end - start
    console.log(`${name} 耗时: ${duration.toFixed(2)}ms`)
  }

  if (fn.constructor.name === 'AsyncFunction') {
    return fn().finally(cleanup)
  }

  fn()
  cleanup()
}

// 使用示例
measurePerformance('数据加载', async () => {
  await fetchData()
})
```

### 性能指标

```typescript
// utils/metrics.ts
export function collectPerformanceMetrics() {
  if (!window.performance) return

  const navigation = performance.getEntriesByType('navigation')[0] as any

  const metrics = {
    // DNS查询时间
    dns: navigation.domainLookupEnd - navigation.domainLookupStart,
    // TCP连接时间
    tcp: navigation.connectEnd - navigation.connectStart,
    // 请求时间
    request: navigation.responseEnd - navigation.requestStart,
    // DOM解析时间
    dom: navigation.domComplete - navigation.domInteractive,
    // 白屏时间
    whiteScreen: navigation.responseStart - navigation.fetchStart,
    // 首次内容绘制
    fcp: navigation.domContentLoadedEventEnd - navigation.fetchStart,
    // 总加载时间
    load: navigation.loadEventEnd - navigation.fetchStart
  }

  console.log('性能指标:', metrics)
  return metrics
}
```

### Web Vitals

```typescript
// utils/web-vitals.ts
export function reportWebVitals() {
  // 首次内容绘制（FCP）
  new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.log('FCP:', entry.startTime)
    }
  }).observe({ entryTypes: ['paint'] })

  // 最大内容绘制（LCP）
  new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.log('LCP:', entry.startTime)
    }
  }).observe({ entryTypes: ['largest-contentful-paint'] })

  // 首次输入延迟（FID）
  new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.log('FID:', (entry as any).processingStart - entry.startTime)
    }
  }).observe({ entryTypes: ['first-input'] })
}
```

---

## 最佳实践

### 1. 减少重渲染

```vue
<script setup lang="ts">
import { shallowRef, markRaw } from 'vue'

// ❌ 深度响应式（不推荐）
const largeData = ref(largeObject)

// ✅ 浅层响应式（推荐）
const largeData = shallowRef(largeObject)

// ✅ 标记为不需要响应式
const staticData = markRaw(staticObject)
</script>
```

### 2. 使用v-show代替v-if

```vue
<template>
  <!-- ❌ 频繁切换使用v-if（不推荐） -->
  <div v-if="isVisible">内容</div>

  <!-- ✅ 频繁切换使用v-show（推荐） -->
  <div v-show="isVisible">内容</div>
</template>
```

### 3. 合理使用keep-alive

```vue
<template>
  <router-view v-slot="{ Component }">
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </router-view>
</template>
```

### 4. 优化事件监听

```vue
<template>
  <!-- ❌ 内联函数（不推荐） -->
  <button @click="handleClick(item)">
    点击
  </button>

  <!-- ✅ 使用事件参数（推荐） -->
  <button @click="handleClick">
    点击
  </button>
</template>

<script setup lang="ts">
const handleClick = (event: MouseEvent) => {
  // 获取数据
  const item = (event.target as HTMLElement).dataset.item
}
</script>
```

### 5. 避免不必要的计算

```vue
<script setup lang="ts">
import { computed, watchEffect } from 'vue'

const count = ref(0)
const name = ref('')

// ❌ 不必要地追踪所有依赖
const result = computed(() => {
  return count.value * 2
})

// ✅ 只在需要时更新
watchEffect(() => {
  if (count.value > 10) {
    // 只在特定条件下执行
  }
})
</script>
```

---

## 🔗 相关文档

- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [API调用规范](./api-calling.md) - API调用详细规范
- [环境配置规范](./environment.md) - 环境配置详细规范

---

**最后更新**：2026-03-12