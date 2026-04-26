# 路由规范

> 本规范定义Vue Router的使用规范，包括路由配置、路由守卫、权限控制和路由元信息。

---

## 📋 目录

- [路由配置规范](#路由配置规范)
- [路由命名规范](#路由命名规范)
- [路由元信息（meta）](#路由元信息meta)
- [路由守卫](#路由守卫)
- [权限控制](#权限控制)
- [动态路由](#动态路由)
- [页面标题管理](#页面标题管理)

---

## 路由配置规范

### 基本结构

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'home',
    component: () => import('@/views/Home.vue'),
    meta: {
      title: '首页',
      requiresAuth: false
    }
  }
]

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes
})

export default router
```

### 路由模块化

```
src/router/
├── index.ts           # 路由主文件
├── routes/            # 路由模块
│   ├── system.ts      # 系统管理模块
│   ├── standard.ts    # 数据标准模块
│   ├── quality.ts     # 数据质量模块
│   └── dashboard.ts   # 仪表盘模块
└── guards.ts          # 路由守卫
```

### 模块化路由示例

```typescript
// src/router/routes/system.ts
import type { RouteRecordRaw } from 'vue-router'

const systemRoutes: RouteRecordRaw[] = [
  {
    path: '/system',
    name: 'system',
    redirect: '/system/users',
    meta: {
      title: '系统管理',
      requiresAuth: true,
      permission: 'system:view'
    },
    children: [
      {
        path: 'users',
        name: 'system-users',
        component: () => import('@/views/system/Users.vue'),
        meta: {
          title: '用户管理',
          requiresAuth: true,
          permission: 'system:user:view'
        }
      }
    ]
  }
]

export default systemRoutes
```

### ⚠️ 修改页面前必须确认路由

**重要**：修改页面前，必须先确认路由指向的实际组件文件路径。

**问题案例**：路由配置为 `@/views/user/UserList.vue`，但开发者错误地修改了 `@/views/system/Users.vue`，导致修改无效。

**正确流程**：

```bash
# 1. 先查看路由配置确认组件路径
grep -r "component:" src/router/

# 2. 确认具体路由指向
grep -r "users" src/router/index.ts
# 输出：component: () => import('@/views/user/UserList.vue')

# 3. 修改正确的文件
# 编辑 src/views/user/UserList.vue
```

**注意事项**：
- 同一功能可能存在多个相似命名的文件（如 `Users.vue` 和 `UserList.vue`）
- 始终以路由配置中的 `component` 字段为准
- 目录结构和路由配置可能不一致（如 `/system/users` 路由指向 `@/views/user/` 目录）

---

## 路由命名规范

### 命名规则

| 路由类型 | 命名格式 | 示例 |
|---------|---------|------|
| 模块路由 | `{module}-{page}` | `system-users`, `quality-rules` |
| 功能路由 | `{module}-{action}` | `system-users-create`, `quality-rules-edit` |
| 详情路由 | `{module}-{page}-detail` | `system-users-detail` |
| 嵌套路由 | 使用 `parent-child` | `system-users-role` |

### 示例

```typescript
{
  path: '/system/users',
  name: 'system-users',              // ✅ 正确：模块-页面
  component: UsersList
}

{
  path: '/system/users/create',
  name: 'system-users-create',       // ✅ 正确：模块-动作
  component: UserForm
}

{
  path: '/system/users/:id',
  name: 'system-users-detail',       // ✅ 正确：模块-详情
  component: UserDetail
}

// ❌ 错误示例
{
  path: '/system/users',
  name: 'usersList',                 // ❌ 不推荐：命名不一致
  component: UsersList
}

{
  path: '/system/users',
  name: 'UsersList',                 // ❌ 错误：使用PascalCase
  component: UsersList
}
```

---

## 路由元信息（meta）

### 标准meta字段

```typescript
interface RouteMeta {
  /** 页面标题 */
  title?: string
  /** 是否需要登录 */
  requiresAuth?: boolean
  /** 所需权限（支持数组） */
  permission?: string | string[]
  /** 页面图标 */
  icon?: string
  /** 是否在菜单中显示 */
  hidden?: boolean
  /** 是否缓存页面 */
  keepAlive?: boolean
  /** 面包屑配置 */
  breadcrumb?: boolean
  /** 自定义样式类 */
  pageClass?: string
  /** 页面描述 */
  description?: string
  /** 是否属于框架页 */
  layout?: 'default' | 'simple' | 'auth'
  /** 排序权重 */
  order?: number
}
```

### meta使用示例

```typescript
{
  path: '/system/users',
  name: 'system-users',
  component: () => import('@/views/system/Users.vue'),
  meta: {
    title: '用户管理',                // 页面标题
    requiresAuth: true,              // 需要登录
    permission: ['system:user:view'], // 需要权限
    icon: 'User',                    // 菜单图标
    hidden: false,                   // 显示在菜单中
    keepAlive: true,                 // 缓存页面
    breadcrumb: true,                // 显示面包屑
    layout: 'default',               // 默认布局
    order: 1                         // 排序
  }
}
```

---

## 路由守卫

### 全局前置守卫

```typescript
// src/router/guards.ts
import type { Router } from 'vue-router'
import { useUserStore } from '@/stores/user'

/**
 * 设置全局路由守卫
 */
export function setupRouterGuards(router: Router) {
  // 前置守卫
  router.beforeEach(async (to, from, next) => {
    const userStore = useUserStore()

    // 检查是否需要登录
    if (to.meta.requiresAuth) {
      if (!userStore.isLoggedIn) {
        // 未登录，跳转到登录页
        next({
          name: 'login',
          query: { redirect: to.fullPath }
        })
        return
      }
    }

    // 检查权限
    if (to.meta.permission) {
      const permissions = Array.isArray(to.meta.permission)
        ? to.meta.permission
        : [to.meta.permission]

      const hasPermission = permissions.some(perm =>
        userStore.hasPermission(perm)
      )

      if (!hasPermission) {
        // 无权限，跳转到403页面
        next({ name: 'error-403' })
        return
      }
    }

    next()
  })

  // 后置守卫 - 设置页面标题
  router.afterEach((to) => {
    document.title = to.meta.title
      ? `${to.meta.title} - 数据中台`
      : '数据中台'
  })
}
```

### 组件内守卫

```vue
<script setup lang="ts">
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

// 离开守卫
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges) {
    const answer = confirm('有未保存的更改，确定要离开吗？')
    if (!answer) {
      return false
    }
  }
})

// 更新守卫
onBeforeRouteUpdate(async (to, from) => {
  if (to.params.id !== from.params.id) {
    await fetchData(to.params.id)
  }
})
</script>
```

### 独享守卫

```typescript
{
  path: '/system/users/:id',
  name: 'system-users-detail',
  component: () => import('@/views/system/UserDetail.vue'),
  beforeEnter: (to, from, next) => {
    // 验证参数
    const id = Number(to.params.id)
    if (isNaN(id) || id <= 0) {
      next({ name: 'error-404' })
      return
    }
    next()
  }
}
```

---

## 权限控制

### 权限Store

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

interface UserState {
  permissions: string[]
  roles: string[]
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    permissions: [],
    roles: []
  }),

  getters: {
    // 检查是否有某个权限
    hasPermission: (state) => (permission: string): boolean => {
      return state.permissions.includes(permission)
    },

    // 检查是否有某个角色
    hasRole: (state) => (role: string): boolean => {
      return state.roles.includes(role)
    },

    // 检查是否有任一权限
    hasAnyPermission: (state) => (permissions: string[]): boolean => {
      return permissions.some(perm => state.permissions.includes(perm))
    },

    // 检查是否有所有权限
    hasAllPermissions: (state) => (permissions: string[]): boolean => {
      return permissions.every(perm => state.permissions.includes(perm))
    }
  }
})
```

### 权限指令

```typescript
// directives/permission.ts
import type { Directive, DirectiveBinding } from 'vue'
import { useUserStore } from '@/stores/user'

const permission: Directive = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const { value } = binding
    const userStore = useUserStore()

    if (value && !userStore.hasPermission(value)) {
      el.parentNode?.removeChild(el)
    }
  }
}

export default permission
```

### 权限函数

```typescript
// utils/permission.ts
import { useUserStore } from '@/stores/user'

/**
 * 检查权限
 */
export function checkPermission(permission: string | string[]): boolean {
  const userStore = useUserStore()
  const permissions = Array.isArray(permission) ? permission : [permission]

  return permissions.some(perm => userStore.hasPermission(perm))
}

/**
 * 检查角色
 */
export function checkRole(role: string | string[]): boolean {
  const userStore = useUserStore()
  const roles = Array.isArray(role) ? role : [role]

  return roles.some(r => userStore.hasRole(r))
}
```

### 在组件中使用权限

```vue
<template>
  <!-- 使用权限指令 -->
  <button v-permission="'system:user:create'">
    创建用户
  </button>

  <!-- 使用权限函数 -->
  <button v-if="checkPermission(['system:user:edit', 'system:user:delete'])">
    操作
  </button>
</template>

<script setup lang="ts">
import { checkPermission } from '@/utils/permission'
</script>
```

---

## 动态路由

### 基于权限动态添加路由

```typescript
// src/router/guards.ts
import type { Router } from 'vue-router'
import { useUserStore } from '@/stores/user'

/**
 * 设置动态路由
 */
export function setupDynamicRoutes(router: Router) {
  router.beforeEach(async (to, from, next) => {
    const userStore = useUserStore()

    // 如果已登录且未加载路由，则加载
    if (userStore.isLoggedIn && !userStore.routesLoaded) {
      try {
        // 获取用户菜单和路由
        const routes = await fetchUserRoutes()

        // 动态添加路由
        routes.forEach(route => {
          router.addRoute('dashboard', route)
        })

        userStore.routesLoaded = true

        // 重新进入当前路由
        next({ ...to, replace: true })
        return
      } catch (error) {
        console.error('加载路由失败:', error)
        next({ name: 'error-500' })
        return
      }
    }

    next()
  })
}

/**
 * 从后端获取用户路由
 */
async function fetchUserRoutes() {
  const userStore = useUserStore()
  const response = await api.get('/api/v1/routes')

  // 转换为路由格式
  return response.data.map((item: any) => ({
    path: item.path,
    name: item.name,
    component: () => import(`@/views/${item.component}`),
    meta: {
      title: item.title,
      permission: item.permission,
      icon: item.icon,
      hidden: item.hidden
    }
  }))
}
```

---

## 页面标题管理

### 统一标题管理

```typescript
// src/router/guards.ts
/**
 * 设置页面标题
 */
export function setupPageTitle(router: Router) {
  router.afterEach((to) => {
    // 从meta获取标题
    const title = to.meta.title || '数据中台'

    // 设置文档标题
    document.title = `${title} - 数据中台`

    // 可选：动态修改浏览器标签页图标
    if (to.meta.icon) {
      const link = document.querySelector("link[rel*='icon']") as HTMLLinkElement
      if (link) {
        link.href = `/icons/${to.meta.icon}.png`
      }
    }
  })
}
```

### 动态标题示例

```typescript
{
  path: '/system/users/:id',
  name: 'system-users-detail',
  component: () => import('@/views/system/UserDetail.vue'),
  meta: {
    title: (route) => {
      const id = route.params.id
      return `用户详情 #${id}`
    }
  }
}
```

---

## 🔗 相关文档

- [API调用规范](./api-calling.md) - API调用详细规范
- [组件设计规范](./component-design.md) - 组件设计详细规范
- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范

---

**最后更新**：2026-03-12