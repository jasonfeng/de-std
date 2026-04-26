# 前端测试规范

> 本规范定义前端测试的使用规范，包括单元测试、组件测试、E2E测试和测试覆盖率要求。

---

## 📋 目录

- [测试原则](#测试原则)
- [单元测试](#单元测试)
- [组件测试](#组件测试)
- [E2E测试](#e2e测试)
- [Mock数据](#mock数据)
- [测试覆盖率](#测试覆盖率)
- [最佳实践](#最佳实践)

---

## 测试原则

### AAA模式

**Arrange（准备）** - 准备测试数据
**Act（执行）** - 执行被测试的代码
**Assert（断言）** - 验证结果

```typescript
describe('UserService', () => {
  it('should return user by id', () => {
    // Arrange - 准备测试数据
    const userId = 1
    const expectedUser = { id: 1, name: 'John' }

    // Act - 执行被测试的代码
    const actualUser = getUserById(userId)

    // Assert - 验证结果
    expect(actualUser).toEqual(expectedUser)
  })
})
```

### FIRST原则

| 原则 | 说明 |
|------|------|
| **F**ast | 测试应该快速运行 |
| **I**ndependent | 测试之间应该相互独立 |
| **R**epeatable | 测试应该可以重复运行 |
| **S**elf-Validating | 测试应该有明确的通过/失败结果 |
| **T**imely | 测试应该及时编写 |

---

## 单元测试

### Vitest配置

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/mockData',
        'src/e2e/'
      ]
    }
  }
})
```

### 工具函数测试

```typescript
// src/utils/format.test.ts
import { describe, it, expect } from 'vitest'
import { formatDate, formatCurrency, formatNumber } from '@/utils/format'

describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-01')
    expect(formatDate(date)).toBe('2024-01-01')
  })

  it('should handle invalid date', () => {
    expect(formatDate(new Date('invalid'))).toBe('')
  })
})

describe('formatCurrency', () => {
  it('should format currency correctly', () => {
    expect(formatCurrency(1234.56)).toBe('¥1,234.56')
  })

  it('should format zero', () => {
    expect(formatCurrency(0)).toBe('¥0.00')
  })
})

describe('formatNumber', () => {
  it('should format number with thousand separator', () => {
    expect(formatNumber(1234567)).toBe('1,234,567')
  })

  it('should format negative number', () => {
    expect(formatNumber(-1234)).toBe('-1,234')
  })
})
```

### API函数测试

```typescript
// src/api/user.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { userApi } from '@/api/user'
import request from '@/utils/request'

vi.mock('@/utils/request')

describe('userApi', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('getList', () => {
    it('should return user list', async () => {
      const mockResponse = {
        records: [
          { id: 1, name: 'John' },
          { id: 2, name: 'Jane' }
        ],
        total: 2
      }

      vi.mocked(request.get).mockResolvedValue(mockResponse)

      const result = await userApi.getList({ current: 1, size: 10 })

      expect(request.get).toHaveBeenCalledWith('/api/v1/users', {
        params: { current: 1, size: 10 }
      })
      expect(result).toEqual(mockResponse)
    })
  })

  describe('getById', () => {
    it('should return user by id', async () => {
      const mockUser = { id: 1, name: 'John' }
      vi.mocked(request.get).mockResolvedValue(mockUser)

      const result = await userApi.getById(1)

      expect(request.get).toHaveBeenCalledWith('/api/v1/users/1')
      expect(result).toEqual(mockUser)
    })
  })
})
```

### Store测试

```typescript
// src/stores/user.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useUserStore } from '@/stores/user'

describe('useUserStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('should initialize with empty state', () => {
    const store = useUserStore()
    expect(store.userInfo).toBeNull()
    expect(store.isLoggedIn).toBe(false)
  })

  it('should set user info', () => {
    const store = useUserStore()
    const user = { id: 1, name: 'John' }

    store.setUserInfo(user)

    expect(store.userInfo).toEqual(user)
  })

  it('should clear user info', () => {
    const store = useUserStore()
    store.setUserInfo({ id: 1, name: 'John' })

    store.clearUserInfo()

    expect(store.userInfo).toBeNull()
  })

  it('should check permission', () => {
    const store = useUserStore()
    store.permissions = ['user:view', 'user:edit']

    expect(store.hasPermission('user:view')).toBe(true)
    expect(store.hasPermission('user:delete')).toBe(false)
  })
})
```

---

## 组件测试

### 组件挂载测试

```typescript
// src/components/UserCard.test.ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import UserCard from '@/components/UserCard.vue'

describe('UserCard', () => {
  it('should render user name', () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      }
    })

    expect(wrapper.text()).toContain('John')
  })

  it('should emit click event', async () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      }
    })

    await wrapper.find('.user-card').trigger('click')

    expect(wrapper.emitted()).toHaveProperty('click')
    expect(wrapper.emitted('click')?.[0]).toEqual([{ id: 1, name: 'John' }])
  })
})
```

### Props测试

```typescript
describe('UserCard Props', () => {
  it('should accept user prop', () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      }
    })

    expect(wrapper.props('user')).toEqual({ id: 1, name: 'John' })
  })

  it('should validate required prop', () => {
    const wrapper = mount(UserCard, {
      props: {
        // @ts-ignore
        user: null
      }
    })

    expect(wrapper.text()).toContain('User is required')
  })
})
```

### Slots测试

```typescript
describe('UserCard Slots', () => {
  it('should render default slot', () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      },
      slots: {
        default: '<div class="custom-content">Custom Content</div>'
      }
    })

    expect(wrapper.find('.custom-content').exists()).toBe(true)
  })

  it('should render named slot', () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      },
      slots: {
        header: '<h1>Custom Header</h1>'
      }
    })

    expect(wrapper.find('h1').text()).toBe('Custom Header')
  })
})
```

### 事件测试

```typescript
describe('UserCard Events', () => {
  it('should emit click event when card is clicked', async () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      }
    })

    await wrapper.find('.user-card').trigger('click')

    expect(wrapper.emitted('click')).toBeTruthy()
    expect(wrapper.emitted('click')?.[0]).toEqual([{ id: 1, name: 'John' }])
  })

  it('should emit delete event when delete button is clicked', async () => {
    const wrapper = mount(UserCard, {
      props: {
        user: { id: 1, name: 'John' }
      }
    })

    await wrapper.find('.delete-button').trigger('click')

    expect(wrapper.emitted('delete')).toBeTruthy()
  })
})
```

### 表单组件测试

```typescript
describe('UserForm', () => {
  it('should validate required fields', async () => {
    const wrapper = mount(UserForm)

    await wrapper.find('form').trigger('submit')

    expect(wrapper.text()).toContain('Name is required')
  })

  it('should submit valid form', async () => {
    const wrapper = mount(UserForm)

    await wrapper.find('input[name="name"]').setValue('John')
    await wrapper.find('form').trigger('submit')

    expect(wrapper.emitted('submit')).toBeTruthy()
    expect(wrapper.emitted('submit')?.[0]).toEqual([{ name: 'John' }])
  })
})
```

---

## E2E测试

### Playwright配置

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }]
  ],
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure'
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    }
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI
  }
})
```

### Page Object模式

```typescript
// e2e/pages/UserListPage.ts
export class UserListPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/system/users')
  }

  async getUserCount() {
    return await this.page.locator('[data-testid="user-table"] tbody tr').count()
  }

  async searchUser(keyword: string) {
    await this.page.locator('[data-testid="input-search"]').fill(keyword)
    await this.page.locator('[data-testid="input-search"]').press('Enter')
  }

  async clickCreateButton() {
    await this.page.getByTestId('btn-create-user').click()
  }

  async isUserVisible(name: string) {
    return await this.page.getByTestId('link-user-name').filter({ hasText: name }).isVisible()
  }
}
```

> **定位规范**：优先使用 `data-testid`，禁止依赖 CSS class。详见 [E2E 测试规范 v3.0](../../04-testing/e2e-testing.md) Section 4。

### E2E测试用例

```typescript
// e2e/user.spec.ts
import { test, expect } from '@playwright/test'
import { UserListPage } from './pages/UserListPage'

const E2E_PREFIX = 'e2e_test_'
const createdUsers: string[] = []

test.describe('用户管理', () => {
  let userListPage: UserListPage

  test.beforeEach(async ({ page }) => {
    userListPage = new UserListPage(page)
    await userListPage.goto()
  })

  test.afterEach(async ({ page }) => {
    // 清理测试数据
    for (const username of createdUsers) {
      try {
        await page.goto('/system/users')
        const rows = page.locator('[data-testid="user-table"] tbody tr')
        const count = await rows.count()
        for (let i = 0; i < count; i++) {
          const row = rows.nth(i)
          const text = await row.textContent()
          if (text?.includes(username)) {
            await row.getByTestId('btn-delete').click()
            await page.locator('.el-message-box .el-button--primary').click()
            break
          }
        }
      } catch { /* 清理失败不影响测试 */ }
    }
    createdUsers.length = 0
  })

  test('应该显示用户列表', async ({ page }) => {
    await expect(page.getByTestId('users-page')).toBeVisible()
    await expect(page.getByTestId('user-table')).toBeVisible()
  })

  test('应该能够搜索用户', async ({ page }) => {
    await userListPage.searchUser('admin')
    const isVisible = await userListPage.isUserVisible('admin')
    expect(isVisible).toBe(true)
  })

  test('应该能够创建用户', async ({ page }) => {
    const username = `${E2E_PREFIX}user_${Date.now()}`
    createdUsers.push(username)

    await userListPage.clickCreateButton()
    await page.getByTestId('input-username').fill(username)
    await page.getByTestId('input-email').fill('test@example.com')
    await page.getByTestId('input-password').fill('password123')
    await page.getByTestId('btn-save-user').click()

    await expect(page.locator('.el-message--success')).toBeVisible()
  })
})
```

### API 集成测试说明

> **注意**：API 集成测试应在后端测试中完成（JUnit + Vitest），不属于前端 E2E 测试范畴。
> E2E 测试中**禁止调用后端 API**（`fetch()`、`request.post()` 等），必须通过浏览器 UI 操作完成全链路。
> 详见 [E2E 测试规范 v3.0](../../04-testing/e2e-testing.md)。

---

## Mock数据

### Mock数据管理

```typescript
// src/test/mockData/users.ts
export const mockUsers = [
  { id: 1, name: 'John', email: 'john@example.com' },
  { id: 2, name: 'Jane', email: 'jane@example.com' }
]

export const mockUser = { id: 1, name: 'John', email: 'john@example.com' }

export const mockCreateUser = {
  name: 'New User',
  email: 'new@example.com'
}
```

### Mock API

```typescript
// src/test/mocks/api.ts
import { http, HttpResponse } from 'msw'
import { setupServer } from 'msw/node'

const server = setupServer(
  http.get('/api/v1/users', () => {
    return HttpResponse.json({
      code: 200,
      message: 'success',
      data: {
        records: mockUsers,
        total: mockUsers.length
      }
    })
  }),

  http.get('/api/v1/users/:id', ({ params }) => {
    const user = mockUsers.find(u => u.id === Number(params.id))
    return HttpResponse.json({
      code: 200,
      message: 'success',
      data: user
    })
  }),

  http.post('/api/v1/users', async ({ request }) => {
    const body = await request.json()
    const newUser = { id: mockUsers.length + 1, ...body }
    return HttpResponse.json({
      code: 200,
      message: 'success',
      data: newUser
    })
  })
)

export default server
```

### 使用Mock

```typescript
// src/api/user.test.ts
import { beforeAll, afterEach, afterAll } from 'vitest'
import { describe, it, expect } from 'vitest'
import server from '@/test/mocks/api'

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

describe('userApi with mock', () => {
  it('should return mock users', async () => {
    const result = await userApi.getList({ current: 1, size: 10 })
    expect(result.records).toHaveLength(2)
  })
})
```

---

## 测试覆盖率

### 覆盖率要求

| 类型 | 最低覆盖率 | 推荐覆盖率 |
|------|-----------|-----------|
| 工具函数 | 90% | 95% |
| API模块 | 80% | 90% |
| Store | 80% | 90% |
| 组件 | 70% | 80% |
| 页面 | 60% | 70% |

### 运行覆盖率

```bash
# 运行所有测试并生成覆盖率报告
npm run test:coverage

# 只生成覆盖率报告
npm run test:coverage:ui
```

### 覆盖率配置

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      lines: 80,
      functions: 80,
      branches: 80,
      statements: 80,
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/mockData'
      ]
    }
  }
})
```

---

## 最佳实践

### 1. 测试命名规范

```typescript
// ✅ 推荐：清晰的描述
it('should return user by id', () => {})
it('should throw error when user not found', () => {})

// ❌ 不推荐：模糊的描述
it('test user', () => {})
it('it works', () => {})
```

### 2. 使用测试辅助函数

```typescript
// src/test/helpers.ts
export function createMockUser(overrides: Partial<User> = {}): User {
  return {
    id: 1,
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  }
}

// 使用
const mockUser = createMockUser({ name: 'Custom Name' })
```

### 3. 隔离测试环境

```typescript
beforeEach(() => {
  // 清理store状态
  setActivePinia(createPinia())

  // 重置mock
  vi.clearAllMocks()
})
```

### 4. 避免测试实现细节

```typescript
// ✅ 推荐：测试行为
it('should show error message when validation fails', () => {
  expect(wrapper.text()).toContain('Name is required')
})

// ❌ 不推荐：测试实现细节
it('should set error message in state', () => {
  expect(wrapper.vm.errorMessage).toBe('Name is required')
})
```

### 5. 使用描述性断言

```typescript
// ✅ 推荐：描述性断言
expect(user.name).toBe('John')
expect(users).toHaveLength(2)

// ❌ 不推荐：非描述性断言
expect(user.name == 'John').toBeTruthy()
expect(users.length == 2).toBeTruthy()
```

---

## 🔗 相关文档

- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [组件设计规范](./component-design.md) - 组件设计详细规范
- [API调用规范](./api-calling.md) - API调用详细规范

---

**最后更新**：2026-03-12