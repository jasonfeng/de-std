# 状态管理规范

> 本规范定义Pinia状态管理的使用规范，确保状态的一致性和可维护性。

---

## 📋 目录

- [Store定义规范](#store定义规范)
- [Store结构](#store结构)
- [State、Getters、Actions](#stategettersactions)
- [模块化](#模块化)
- [持久化](#持久化)
- [TypeScript支持](#typescript支持)
- [使用示例](#使用示例)

---

## Store定义规范

### 基本结构

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    userInfo: null as User | null,
    token: null as string | null,
    isLoggedIn: false
  }),

  getters: {
    isLoggedIn: (state) => !!state.token
  },

  actions: {
    setUserInfo(info: User) {
      this.userInfo = info
    },
    clearUserInfo() {
      this.userInfo = null
    }
  }
})
```

### TypeScript类型定义

```typescript
interface User {
  id: number
  username: string
  email: string
}

interface UserState {
  userInfo: User | null
  token: string | null
  isLoggedIn: boolean
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    userInfo: null,
    token: null,
    isLoggedIn: false
  }),

  getters: {
    isLoggedIn: (state): boolean {
      return !!state.token
    }
  },

  actions: {
    setUserInfo(info: User) {
      this.userInfo = info
    },
    clearUserInfo() {
      this.userInfo = null
    }
  }
})
```

---

## Store结构

### 完整示例

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import api from '@/api/user'

interface User {
  id: number
  username: string
  email: string
  avatar?: string
}

interface UserState {
  userInfo: User | null
  token: string | null
  isLoggedIn: boolean
  permissions: string[]
}

export const useUserStore = defineStore('user', {
  id: 'user',  // 唯一ID，可选

  state: (): UserState => ({
    userInfo: null,
    token: null,
    isLoggedIn: false,
    permissions: []
  }),

  getters: {
    // 获取用户显示名称
    displayName: (state): string => {
      if (state.userInfo) {
        return state.userInfo.username
      }
      return 'Guest'
    },

    // 检查权限
    hasPermission: (state) => (permission: string): boolean => {
      return state.permissions.includes(permission)
    }
  },

  actions: {
    // 登录
    async login(credentials: LoginCredentials) {
      const response = await api.login(credentials)
      this.token = response.token
      this.isLoggedIn = true
      this.permissions = response.permissions

      // 设置token到axios
      api.setToken(response.token)
    },

    // 登出
    async logout() {
      await api.logout()
      this.token = null
      this.userInfo = null
      this.isLoggedIn = false
      this.permissions = []

      // 清除token
      api.clearToken()
    },

    // 获取用户信息
    async fetchUserInfo() {
      const response = await api.getUserInfo()
      this.userInfo = response
    },

    // 更新用户信息
    updateUserInfo(userInfo: Partial<User>) {
      if (this.userInfo) {
        this.userInfo = { ...this.userInfo, ...userInfo }
      }
    }
  },

  // 持久化配置
  persist: {
    enabled: true,
    strategies: [
      {
        key: 'user',
        storage: localStorage
      }
    ]
  }
})
```

---

## State、Getters、Actions

### State

**职责**：存储状态数据

```typescript
state: () => ({
  // 原始数据
  count: 0,
  message: 'Hello',

  // 对象
  user: null as User | null,

  // 数组
  items: [] as Item[],

  // 复杂对象
  settings: {
    theme: 'light',
    language: 'zh-CN'
  }
})
```

### Getters

**职责**：计算属性，基于state派生数据

```typescript
getters: {
  // 基本计算
  doubledCount: (state) => state.count * 2,

  // 对象属性
  username: (state): string => {
    return state.user?.username || 'Guest'
  },

  // 数组过滤
  activeItems: (state) => state.items.filter(item => item.active),

  // 多个getter组合
  userDisplayName: (state) => {
    return `${state.user?.username || 'Guest'} (${state.user?.email || 'No email'})`
  }
}
```

### Actions

**职责**：业务逻辑，修改state

```typescript
actions: {
  // 同步action
  increment() {
    this.count++
  },

  // 异步action
  async fetchUsers() {
    const response = await api.getUsers()
    this.items = response.data
  },

  // 组合action
  async loginAndFetchUser(credentials: LoginCredentials) {
    await this.login(credentials)
    await this.fetchUserInfo()
  }
}
```

---

## 模块化

### 按功能划分

```
stores/
├── index.ts                # 入口文件
├── user.ts                 # 用户模块
├── settings.ts             # 设置模块
├── permission.ts           # 权限模块
└── notification.ts         # 通知模块
```

### index.ts

```typescript
// stores/index.ts
import { createPinia } from 'pinia'

const pinia = createPinia()

export default pinia
```

### main.ts

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import pinia from './stores'

const app = createApp(App)
app.use(pinia)
```

---

## 持久化

### pinia-plugin-persistedstate

```bash
npm install pinia-plugin-persistedstate
```

### 配置

```typescript
// stores/user.ts
export const useUserStore = defineStore('user', {
  // ... state, getters, actions ...

  persist: {
    enabled: true,
    strategies: [
      {
        key: 'user',
        storage: localStorage,
        paths: ['token', 'userInfo']
      }
    ]
  }
})
```

### 全局配置

```typescript
// stores/index.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

export default pinia
```

---

## TypeScript支持

### 类型定义

```typescript
// types/user.ts
export interface User {
  id: number
  username: string
  email: string
  avatar?: string
}

export interface LoginCredentials {
  username: string
  password: string
}

export interface LoginResponse {
  token: string
  user: User
  permissions: string[]
}
```

### Store类型定义

```typescript
import type { UserState } from '@/types/user'
import type { User, LoginCredentials } from '@/types/user'

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    userInfo: null,
    token: null,
    isLoggedIn: false
  }),

  actions: {
    async login(credentials: LoginCredentials) {
      // ...
    }
  }
})
```

---

## 使用示例

### 在组件中使用

```vue
<script setup lang="ts">
import { computed } from 'vue'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// 访问state
const username = computed(() => userStore.userInfo?.username)

// 访问getter
const displayName = computed(() => userStore.displayName)

// 调用action
const handleLogin = async () => {
  await userStore.login({ username: 'admin', password: '123456' })
}

const handleLogout = async () => {
  await userStore.logout()
}
</script>

<template>
  <div v-if="userStore.isLoggedIn">
    <p>Welcome, {{ displayName }}!</p>
    <button @click="handleLogout">Logout</button>
  </div>
  <div v-else>
    <button @click="handleLogin">Login</button>
  </div>
</template>
```

### 在其他Store中使用

```typescript
// stores/settings.ts
import { defineStore } from 'pinia'
import { useUserStore } from './user'

export const useSettingsStore = defineStore('settings', {
  state: () => ({
    theme: 'light' as 'light' | 'dark',
    language: 'zh-CN'
  }),

  actions: {
    switchTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light'
    },

    async saveUserPreferences() {
      const userStore = useUserStore()
      await api.savePreferences({
        theme: this.theme,
        language: this.language
      })
    }
  }
})
```

---

## 🔗 相关文档

- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [TypeScript规范](./typescript.md) - TypeScript详细规范
- [组件设计规范](./component-design.md) - 组件设计详细规范

---

**最后更新**：2026-03-10