# API契约规范

> 本文档定义前后端API接口契约，确保接口一致性。

---

## 📋 RESTful API命名规范

### URL命名规则

| 规则 | 说明 | 示例 |
|------|------|------|
| **资源命名** | 使用名词复数形式 | `/api/users`, `/api/orders` |
| **层级关系** | 使用路径表示层级 | `/api/users/{id}/orders` |
| **小写+连字符** | URL使用小写和连字符 | `/api/data-quality-rules` |
| **版本控制** | 通过请求头而非URL | `X-API-Version: 1.0` |

### HTTP方法语义

| 方法 | 用途 | 幂等性 | 示例 |
|------|------|--------|------|
| **GET** | 查询资源 | ✅ 是 | `GET /api/users` |
| **POST** | 创建资源 | ❌ 否 | `POST /api/users` |
| **PUT** | 全量更新 | ✅ 是 | `PUT /api/users/{id}` |
| **PATCH** | 部分更新 | ✅ 是 | `PATCH /api/users/{id}` |
| **DELETE** | 删除资源 | ✅ 是 | `DELETE /api/users/{id}` |

### URL路径示例

```bash
# 资源操作
GET    /api/users                    # 获取用户列表
GET    /api/users/{id}               # 获取单个用户
POST   /api/users                    # 创建用户
PUT    /api/users/{id}               # 全量更新用户
PATCH  /api/users/{id}               # 部分更新用户
DELETE /api/users/{id}               # 删除用户

# 关联资源
GET    /api/users/{id}/orders        # 获取用户的订单
POST   /api/users/{id}/orders        # 为用户创建订单

# 操作类接口（动作）
POST   /api/users/{id}/activate      # 激活用户
POST   /api/users/{id}/deactivate    # 停用用户
```

---

## 📤 请求格式规范

### 请求头

| 请求头 | 值 | 说明 |
|--------|-----|------|
| **Content-Type** | `application/json` | 请求体格式 |
| **Accept** | `application/json` | 响应体格式 |
| **X-API-Version** | `1.0` | API版本（预留） |

### 请求参数

#### Path参数（路径参数）
```bash
GET /api/users/123
```

#### Query参数（查询参数）
```bash
GET /api/users?page=1&size=10&keyword=test
```

#### Request Body（请求体）
```json
POST /api/users
{
  "userName": "testuser",
  "email": "test@example.com"
}
```

### 分页参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| **page** | Integer | 1 | 页码（从1开始） |
| **size** | Integer | 10 | 每页大小 |
| **sort** | String | - | 排序字段（如：`createTime,desc`） |

```bash
GET /api/users?page=1&size=10&sort=createTime,desc
```

---

## 📥 响应格式规范

### 统一响应结构

```json
{
  "code": 200,
  "message": "success",
  "data": {},
  "timestamp": 1734567890123
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| **code** | Integer | 业务状态码（非HTTP状态码） |
| **message** | String | 响应消息 |
| **data** | Object/Array | 响应数据 |
| **timestamp** | Long | 响应时间戳 |

### 分页响应结构

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "records": [],
    "total": 100,
    "page": 1,
    "size": 10
  },
  "timestamp": 1734567890123
}
```

### 错误响应结构

```json
{
  "code": 400,
  "message": "参数验证失败",
  "data": {
    "fieldName": "错误详情"
  },
  "timestamp": 1734567890123
}
```

---

## 🔢 状态码规范

### 业务状态码（code）

| code | message | 说明 |
|------|---------|------|
| **200** | success | 操作成功 |
| **400** | 参数验证失败 | 请求参数错误 |
| **401** | 未授权 | 需要登录 |
| **403** | 禁止访问 | 无权限 |
| **404** | 资源不存在 | |
| **409** | 资源冲突 | 如：用户名已存在 |
| **500** | 服务器错误 | 系统错误 |
| **501** | 功能未实现 | |

### HTTP状态码使用

| HTTP状态 | 使用场景 | code值 |
|----------|----------|--------|
| **200** | 成功 | 200 |
| **201** | 创建成功 | 200 |
| **400** | 参数错误 | 400 |
| **401** | 未登录 | 401 |
| **403** | 无权限 | 403 |
| **404** | 资源不存在 | 404 |
| **500** | 系统错误 | 500 |

---

## 📐 字段命名规范

### 后端→前端字段映射

| 规则 | 说明 | 示例 |
|------|------|------|
| **驼峰转短横线** | 后端驼峰 → 前端短横线 | `userName` → `user-name` |
| **保持语义** | 不改变字段语义 | `createTime` → `create-time` |
| **保留缩写** | 技术缩写保持不变 | `userId` → `user-id`, `URL` → `url` |

### 示例

```json
// 后端响应（驼峰）
{
  "userId": 123,
  "userName": "test",
  "createTime": "2026-03-14T10:00:00"
}

// 前端接收（短横线）
{
  "user-id": 123,
  "user-name": "test",
  "create-time": "2026-03-14T10:00:00"
}
```

> **注**：前端使用axios拦截器自动转换字段命名风格

---

## 🔍 前端API调用规范

### API定义位置

```
frontend/src/api/
├── user.ts          # 用户相关API
├── order.ts         # 订单相关API
└── common.ts        # 通用API
```

### API定义模板

```typescript
// src/api/user.ts
import request from '@/utils/request';

/**
 * 用户API
 */
export const userApi = {
  /**
   * 获取用户列表
   */
  getList: (params: UserQuery) => {
    return request.get<UserPageResult>('/api/users', { params });
  },

  /**
   * 获取用户详情
   */
  getDetail: (id: number) => {
    return request.get<UserVO>(`/api/users/${id}`);
  },

  /**
   * 创建用户
   */
  create: (data: UserForm) => {
    return request.post<UserVO>('/api/users', data);
  },

  /**
   * 更新用户
   */
  update: (id: number, data: UserForm) => {
    return request.put<UserVO>(`/api/users/${id}`, data);
  },

  /**
   * 删除用户
   */
  delete: (id: number) => {
    return request.delete(`/api/users/${id}`);
  }
};
```

### 请求拦截器配置

```typescript
// src/utils/request.ts
import axios from 'axios';

const request = axios.create({
  baseURL: '/api',
  timeout: 10000
});

// 请求拦截器
request.interceptors.request.use(
  (config) => {
    // 添加Token
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器
request.interceptors.response.use(
  (response) => {
    const { code, data, message } = response.data;
    if (code === 200) {
      return data;
    }
    // 处理业务错误
    ElMessage.error(message);
    return Promise.reject(new Error(message));
  },
  (error) => {
    // 处理HTTP错误
    if (error.response?.status === 401) {
      // 跳转登录
    }
    return Promise.reject(error);
  }
);

export default request;
```

---

## 📝 后端接口定义规范

### Controller模板

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserAppService userAppService;

    /**
     * 获取用户列表
     */
    @GetMapping
    public Result<PageResult<UserVO>> getList(UserQuery query) {
        PageResult<UserVO> page = userAppService.getList(query);
        return Result.success(page);
    }

    /**
     * 获取用户详情
     */
    @GetMapping("/{id}")
    public Result<UserVO> getDetail(@PathVariable Long id) {
        UserVO vo = userAppService.getDetail(id);
        return Result.success(vo);
    }

    /**
     * 创建用户
     */
    @PostMapping
    public Result<UserVO> create(@Valid @RequestBody UserParam param) {
        UserVO vo = userAppService.create(param);
        return Result.success(vo);
    }

    /**
     * 更新用户
     */
    @PutMapping("/{id}")
    public Result<UserVO> update(
        @PathVariable Long id,
        @Valid @RequestBody UserParam param
    ) {
        UserVO vo = userAppService.update(id, param);
        return Result.success(vo);
    }

    /**
     * 删除用户
     */
    @DeleteMapping("/{id}")
    public Result<Void> delete(@PathVariable Long id) {
        userAppService.delete(id);
        return Result.success();
    }
}
```

---

## ✅ 接口变更流程

### 新增接口
1. 后端定义Controller接口
2. 更新API文档
3. 前端创建API定义文件
4. 前端实现页面调用

### 修改接口
1. 评估影响范围
2. 后端修改Controller（保持兼容性）
3. 更新API文档
4. 前端同步修改API调用

### 废弃接口
1. 标记为废弃（@Deprecated）
2. 设置废弃期限
3. 通知前端
4. 到期后删除

---

## 🔗 相关文档

- [前端API调用](../02-frontend/api-calling.md)
- [后端DTO规范](../01-backend/dto-convention.md)
- [核心规则](./essential-rules.md)
- [项目术语表](./glossary.md)

---

**文档元数据**：
- **维护者**: 技术架构团队
- **目标读者**: 前后端开发者
- **最后更新**: 2026-03-14
- **下次审查**: 每季度
