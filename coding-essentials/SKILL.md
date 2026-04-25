---
name: coding-essentials
description: 写任何代码前必须调用的核心规范。包含10条不可违反的强制规则。
---

# 编码核心规范

⚠️ **强制执行**：违反以下规则的代码将被拒绝。

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

---

### 规则2：PostgreSQL语法禁止MySQL语法

```sql
-- ❌ 错误（MySQL）
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL COMMENT '用户名',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- ✅ 正确（PostgreSQL）
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
COMMENT ON TABLE t_user IS '用户表';
COMMENT ON COLUMN t_user.user_name IS '用户名';
```

---

### 规则3：SQL必须在XML文件中

```java
// ❌ 错误（禁止）
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM t_user WHERE id = #{id}")
    UserEntity findById(Long id);
}

// ✅ 正确
@Mapper
public interface UserMapper {
    UserEntity findById(Long id);
}
```

```xml
<!-- resources/mapper/user/UserMapper.xml -->
<select id="findById" resultType="...UserEntity">
    SELECT * FROM t_user WHERE id = #{id} AND deleted = 0
</select>
```

---

### 规则4：依赖注入使用构造器注入

```java
// ❌ 错误（禁止）
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
}

// ✅ 正确
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserMapper userMapper;
}
```

---

### 规则5：主键使用雪花算法

```java
// ❌ 错误
@TableId(value = "id", type = IdType.AUTO)
private Long id;

// ✅ 正确
@TableId(value = "id", type = IdType.ASSIGN_ID)
private Long id;
```

---

### 规则6：表名使用t_前缀

| 数据库表 | 实体类 | 状态 |
|---------|--------|------|
| `t_user` | `UserEntity` | ✅ 正确 |
| `user` | `UserEntity` | ❌ 缺少t_前缀 |
| `t_user` | `User` | ❌ 缺少Entity后缀 |

---

### 规则7：DDD分层架构

```
com.dp.dataengine/
├── interfaces/          # Controller、Param、VO
├── application/         # AppService、DTO
├── domain/              # Entity、DomainService、Repository ⭐核心
├── infrastructure/      # Config、Security
└── common/              # Result、Enums
```

**调用关系**：
```
Controller → AppService → DomainService → Repository → Database
```

---

### 规则8：测试覆盖率必须达标

| 层级 | 指令覆盖率 | 分支覆盖率 |
|------|-----------|-----------|
| DomainService | ≥80% | ≥65% |
| AppService | ≥65% | ≥55% |
| Controller | ≥50% | ≥40% |

---

### 规则9：所有问题必须解决

**禁止行为**：
- ❌ 以"后续修复"、"临时方案"遗留问题
- ❌ 编译错误未修复就提交
- ❌ 测试失败未修复就提交

---

### 规则10：数据库变更必须通过Flyway

**禁止**：手动修改数据库结构

**正确流程**：
```bash
# 1. 创建迁移脚本
src/main/resources/db/migration/V24__create_table.sql

# 2. 执行迁移
mvn flyway:migrate
```

---

## 调用时机

- 开始写任何代码之前
- 开始新功能开发时
- 代码审查前自查