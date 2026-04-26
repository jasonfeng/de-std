# TypeScript规范

> 本规范定义TypeScript的使用规范，确保类型安全和代码质量。

---

## 📋 目录

- [严格模式](#严格模式-⚠️强制)
- [类型定义](#类型定义)
- [接口vs类型](#接口vs类型)
- [泛型使用](#泛型使用)
- [类型断言](#类型断言)
- [工具类型](#工具类型)
- [常用类型](#常用类型)

---

## 严格模式（⚠️强制）

### tsconfig.json配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "exclude": ["node_modules"]
}
```

### 严格模式规则

| 规则 | 说明 |
|------|------|
| `strict` | 启用所有严格类型检查 |
| `noImplicitAny` | 禁止隐式any类型 |
| `noImplicitReturns` | 函数必须有返回值类型 |
| `noImplicitThis` | this必须有明确类型 |
| `noUnusedLocals` | 禁止未使用的局部变量 |
| `noUnusedParameters` | 禁止未使用的参数 |

---

## 类型定义

### 基本类型

```typescript
// 原始类型
const str: string = 'hello'
const num: number = 42
const bool: boolean = true
const arr: string[] = ['a', 'b', 'c']
const obj: object = { key: 'value' }
const nul: null = null
const undef: undefined = undefined

// 数组类型
const numbers: number[] = [1, 2, 3]
const strings: Array<string> = ['a', 'b', 'c']

// 对象类型
const user: {
  id: number
  username: string
  email?: string
} = {
  id: 1,
  username: 'admin'
}
```

### 函数类型

```typescript
// 函数声明
function add(a: number, b: number): number {
  return a + b
}

// 箭头函数
const multiply = (a: number, b: number): number => {
  return a * b
}

// 可选参数
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Hello'}, ${name}!`
}

// 剩余参数
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0)
}

// 函数类型别名
type MathOperation = (a: number, b: number) => number
const subtract: MathOperation = (a, b) => a - b
```

---

## 接口vs类型

### 接口（Interface）

**使用场景**：定义对象结构、类实现

```typescript
interface User {
  id: number
  username: string
  email?: string
}

interface UserService {
  getUserById(id: number): User
  createUser(user: Omit<User, 'id'>): User
}

// 类实现
class UserServiceImpl implements UserService {
  getUserById(id: number): User {
    return { id, username: 'admin' }
  }

  createUser(user: Omit<User, 'id'>): User {
    return { id: 1, ...user }
  }
}
```

### 类型（Type）

**使用场景**：联合类型、元组类型、工具类型

```typescript
// 联合类型
type Status = 'active' | 'inactive' | 'deleted'
type ID = number | string

// 元组类型
type Point = [number, number]
const point: Point = [10, 20]

// 交叉类型
type Name = { name: string }
type Age = { age: number }
type Person = Name & Age

const person: Person = {
  name: 'John',
  age: 30
}
```

### 选择原则

| 场景 | 使用接口 | 使用类型 |
|------|---------|---------|
| 定义对象结构 | ✅ 推荐 | ❌ 不推荐 |
| 类实现 | ✅ 推荐 | ❌ 不推荐 |
| 函数签名 | ✅ 推荐 | ✅ 推荐 |
| 联合类型 | ❌ 不推荐 | ✅ 推荐 |
| 元组类型 | ❌ 不推荐 | ✅ 推荐 |
| 工具类型 | ❌ 不推荐 | ✅ 推荐 |

---

## 泛型使用

### 泛型函数

```typescript
function identity<T>(arg: T): T {
  return arg
}

const num = identity<number>(42)
const str = identity<string>('hello')

// 类型推断
const autoNum = identity(42)
const autoStr = identity('hello')
```

### 泛型接口

```typescript
interface Repository<T> {
  findById(id: number): T | null
  findAll(): T[]
  create(entity: Omit<T, 'id'>): T
  update(id: number, entity: Partial<T>): boolean
}

interface UserRepository extends Repository<User> {
  findByUsername(username: string): User | null
}
```

### 泛型约束

```typescript
interface Lengthwise {
  length: number
}

function logLength<T extends Lengthwise>(item: T): void {
  console.log(item.length)
}

// 泛型约束多个类型
interface NameAndAge {
  name: string
  age: number
}

function showNameAndAge<T extends NameAndAge>(item: T): void {
  console.log(`${item.name} is ${item.age} years old`)
}
```

---

## 类型断言

### as语法

```typescript
// 类型断言
const canvas = document.getElementById('canvas') as HTMLCanvasElement
canvas.getContext('2d')
```

### 非空断言

```typescript
function processValue(value: string | null) {
  // 非空断言：如果value为null，运行时错误
  const len = value!.length
}
```

### 类型守卫

```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string'
}

function processValue(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase())
  }
}
```

---

## 工具类型

### Partial

```typescript
interface User {
  id: number
  username: string
  email: string
}

type PartialUser = Partial<User>
// 等价于
// interface PartialUser {
//   id?: number
//   username?: string
//   email?: string
// }
```

### Required

```typescript
type RequiredUser = Required<User>
// 所有可选属性变为必需
```

### Readonly

```typescript
type ReadonlyUser = Readonly<User>
// 所有属性变为只读
```

### Pick

```typescript
type UserSummary = Pick<User, 'id' | 'username'>
// 等价于
// interface UserSummary {
//   id: number
//   username: string
// }
```

### Omit

```typescript
type CreateUserDTO = Omit<User, 'id' | 'createdAt' | 'updatedAt'>
// 排除指定属性
```

### Record

```typescript
type UserMap = Record<string, User>
// 等价于
// interface UserMap {
//   [key: string]: User
// }
```

---

## 常用类型

### Vue 3类型

```typescript
import { Ref, ComputedRef } from 'vue'

// Ref
const count: Ref<number> = ref(0)

// ComputedRef
const doubled: ComputedRef<number> = computed(() => count.value * 2)
```

### Props类型

```typescript
interface Props {
  title: string
  count?: number
}

const props = defineProps<Props>()
```

### Emits类型

```typescript
const emit = defineEmits<{
  'update:title': [value: string]
  'submit': [payload: SubmitData]
}>()
```

### Component类型

```typescript
import { ComponentCustomProperties } from 'vue'

// 扩展全局属性
declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    $http: HttpClient
    $router: Router
  }
}
```

---

## 类型导出

### 导出类型

```typescript
// types/user.ts
export interface User {
  id: number
  username: string
}

export type UserId = number
export type Status = 'active' | 'inactive' | 'deleted'
```

### 导入类型

```typescript
import type { User, UserId, Status } from '@/types/user'
```

### 类型声明文件

```typescript
// 声明文件：*.d.ts
declare global {
  interface Window {
    appConfig: AppConfig
  }
}

export {}
```

---

## 🔗 相关文档

- [组件设计规范](./component-design.md) - 组件设计详细规范
- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [状态管理规范](./state-management.md) - Pinia详细规范

---

**最后更新**：2026-03-10