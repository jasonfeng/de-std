---
name: coding-essentials
description: 写任何代码前必须调用的核心规范。包含10条不可违反的强制规则、API契约、术语表。
---

# 编码核心规范

⚠️ **强制执行**：违反以下规则的代码将被拒绝。

## 触发时机

用户说以下内容时**必须**先调用此 Skill：

- "帮我添加/修改/重构 X 功能"
- "写一个 Service/Controller/组件"
- "创建一个新的模块"
- **任何涉及写代码的请求**

---

## 10条核心规则

### 规则1：实体类必须带Entity后缀

```java
// ❌ 错误
public class User { }
public class TUser { }

// ✅ 正确
@Data
@TableName("t_user")
public class UserEntity { }
```

### 规则2：PostgreSQL语法禁止MySQL语法

```sql
-- ❌ 错误（MySQL）
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL COMMENT '用户名'
) ENGINE=InnoDB;

-- ✅ 正确（PostgreSQL）
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL
);
COMMENT ON TABLE t_user IS '用户表';
COMMENT ON COLUMN t_user.user_name IS '用户名';
```

### 规则3：SQL必须在XML文件中

```java
// ❌ 禁止
@Select("SELECT * FROM t_user WHERE id = #{id}")
UserEntity findById(Long id);

// ✅ 正确：XML方式
public interface UserMapper {
    UserEntity findById(@Param("id") Long id);
}
```

### 规则4：依赖注入使用构造器注入

```java
// ❌ 禁止
@Autowired
private UserMapper userMapper;

// ✅ 正确
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserMapper userMapper;
}
```

### 规则5：主键使用雪花算法

```java
@TableId(value = "id", type = IdType.ASSIGN_ID)  // ✅
private Long id;
```

### 规则6：表名使用t_前缀

| 数据库表 | 实体类 | 状态 |
|---------|--------|------|
| `t_user` | `UserEntity` | ✅ |
| `user` | `UserEntity` | ❌ 缺少t_前缀 |
| `t_user` | `User` | ❌ 缺少Entity后缀 |

### 规则7：DDD分层架构

```
Controller → AppService → DomainService → Repository → Database
```

### 规则8：测试覆盖率必须达标

| 层级 | 指令覆盖率 | 分支覆盖率 |
|------|-----------|-----------|
| DomainService | ≥80% | ≥65% |
| AppService | ≥65% | ≥55% |
| Controller | ≥50% | ≥40% |

### 规则9：所有问题必须解决

禁止以"后续修复"、"临时方案"遗留问题。

### 规则10：数据库变更必须通过Flyway

```bash
src/main/resources/db/migration/V24__create_table.sql
mvn flyway:migrate
```

---

## API契约规范

### 统一响应结构

```json
{
  "code": 200,
  "message": "success",
  "data": {},
  "timestamp": 1734567890123
}
```

### 分页响应

```json
{
  "code": 200,
  "data": {
    "records": [],
    "total": 100,
    "page": 1,
    "size": 10
  }
}
```

### RESTful命名

| 方法 | 用途 | 示例 |
|------|------|------|
| GET | 查询 | `GET /api/users` |
| POST | 创建 | `POST /api/users` |
| PUT | 全量更新 | `PUT /api/users/{id}` |
| DELETE | 删除 | `DELETE /api/users/{id}` |

### 业务状态码

| code | 说明 |
|------|------|
| 200 | 成功 |
| 400 | 参数验证失败 |
| 401 | 未授权 |
| 403 | 禁止访问 |
| 500 | 服务器错误 |

---

## 数据对象层次

```
Param (请求) → DTO (传输) → Entity (领域) → VO (响应)
```

| 类型 | 位置 | 用途 | 示例 |
|------|------|------|------|
| Param | interfaces/param | 接收请求 | `UserCreateParam` |
| VO | interfaces/vo | 返回响应 | `UserVO` |
| DTO | application/dto | 跨层传递 | `UserDTO` |
| Entity | domain/entity | 数据库映射 | `UserEntity` |

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| 先写实现后补测试 | 违反 TDD |
| @Autowired 字段注入 | 无法使用 final，测试困难 |
| @Select/@Insert 注解 | SQL难以维护和审查 |
| MySQL 语法 | PostgreSQL 不兼容 |
| IdType.AUTO 自增 | 分布式不友好 |
| 手动修改数据库 | Flyway 版本失控 |
| 留空 TODO 注释 | 所有问题必须解决 |
| 硬编码 Mock 数据 | 功能视为未完成 |

---

## 详细规范

需要深入某个主题时，阅读对应的 references 文档：

- **[essential-rules.md](references/essential-rules.md)** — 10条核心规则完整详解
- **[api-contract.md](references/api-contract.md)** — API契约规范（请求/响应格式、状态码、前后端字段映射）
- **[glossary.md](references/glossary.md)** — 项目术语表（DDD术语、前端术语、数据库术语）
- **[newcomer-guide.md](references/newcomer-guide.md)** — 新人入门指南（环境搭建、学习路径）
