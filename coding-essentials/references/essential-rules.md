# 核心规则 - 10条必须遵守的规则

> 这是所有开发者必须遵守的核心规则，违反将导致代码审查失败或功能问题。

**新成员从这里开始！** 其他详细规范请查看各领域文档。

---

## 📋 10条核心规则清单

| # | 规则 | 详细文档 |
|---|------|---------|
| 1 | **实体类必须带Entity后缀** | [命名规范](../01-backend/java-coding.md#命名规范) |
| 2 | **PostgreSQL语法禁止MySQL语法** | [PostgreSQL语法](../03-database/postgresql.md) |
| 3 | **SQL必须在XML文件中** | [MyBatis映射](../03-database/mybatis-mapping.md) |
| 4 | **依赖注入使用构造器注入** | [依赖注入](../01-backend/spring-boot.md) |
| 5 | **主键使用雪花算法** | [主键策略](../03-database/naming-conventions.md) |
| 6 | **表名使用t_前缀** | [表名规范](../03-database/naming-conventions.md) |
| 7 | **DDD分层架构** | [DDD架构](../01-backend/ddd-architecture.md) |
| 8 | **测试覆盖率必须达标** | [覆盖率标准](../04-testing/coverage-standards.md) |
| 9 | **所有问题必须解决** | [问题解决原则](../05-workflow/README.md) |
| 10 | **数据库变更必须通过Flyway** | [Flyway规范](../03-database/flyway.md) |

---

## ⚠️ 前3条规则（最容易违反）

### 规则1：实体类必须带Entity后缀

**❌ 错误**：
```java
public class User { }  // ❌ 缺少Entity后缀
public class TUser { } // ❌ 带t前缀是表名，不是类名
```

**✅ 正确**：
```java
@Data
@TableName("t_user")
public class UserEntity { }  // ✅ 正确
```

---

### 规则2：PostgreSQL语法禁止MySQL语法

**❌ 错误（MySQL语法）**：
```sql
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY,              -- ❌ BIGINT不自增
    user_name VARCHAR(50) NOT NULL COMMENT '用户名',  -- ❌ 行内COMMENT
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;                        -- ❌ MySQL特有
```

**✅ 正确（PostgreSQL语法）**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,           -- ✅ BIGSERIAL自增
    user_name VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ✅ 添加注释使用单独语句
COMMENT ON TABLE t_user IS '用户表';
COMMENT ON COLUMN t_user.user_name IS '用户名';
```

---

### 规则3：SQL必须在XML文件中

**❌ 错误**：
```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM t_user WHERE id = #{id}")
    UserEntity findById(Long id);
}
```

**✅ 正确**：
```java
// Mapper接口
@Mapper
public interface UserMapper {
    UserEntity findById(Long id);
}
```

```xml
<!-- resources/mapper/user/UserMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dp.dataengine.domain.repository.user.UserMapper">
    <select id="findById" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
        SELECT * FROM t_user WHERE id = #{id} AND deleted = 0
    </select>
</mapper>
```

---

## 📖 其他7条规则详解

### 规则4：依赖注入使用构造器注入

**❌ 错误**：
```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
}
```

**✅ 正确**：
```java
@Service
@RequiredArgsConstructor  // Lombok生成构造器
public class UserService {
    private final UserMapper userMapper;
}
```

---

### 规则5：主键使用雪花算法

**❌ 错误**：
```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
```

**✅ 正确**：
```java
@TableId(value = "id", type = IdType.ASSIGN_ID)
private Long id;
```

**理由**：
- 分布式友好（支持多实例部署）
- 性能更好（应用层生成，无数据库开销）
- ID可提前赋值（插入前就知道ID）

---

### 规则6：表名使用t_前缀

| 数据库表 | 实体类 | 说明 |
|---------|--------|------|
| `t_user` | `UserEntity` | ✅ 正确 |
| `t_order` | `OrderEntity` | ✅ 正确 |
| `user` | `UserEntity` | ❌ 缺少t_前缀 |
| `t_user` | `User` | ❌ 缺少Entity后缀 |

---

### 规则7：DDD分层架构

**正确的包结构**：
```
com.dp.dataengine/
├── interfaces/          # 接口层
│   └── controller/
├── application/         # 应用层
│   └── service/         # AppService
├── domain/              # 领域层 ⭐核心
│   ├── entity/
│   ├── service/         # DomainService
│   └── repository/
├── infrastructure/      # 基础设施层
└── common/              # 通用组件
```

**调用关系**：
```
Controller → AppService → DomainService → Repository → Database
```

---

### 规则8：测试覆盖率必须达标

| 层级 | 指令覆盖率 | 分支覆盖率 | 方法覆盖率 |
|------|-----------|-----------|-----------|
| DomainService | ≥80% | ≥65% | ≥85% |
| AppService | ≥65% | ≥55% | ≥75% |
| Controller | ≥50% | ≥40% | ≥60% |

**验证命令**：
```bash
mvn clean test
open target/site/jacoco/index.html
```

---

### 规则9：所有问题必须解决

**禁止行为**：
- ❌ 以"后续修复"、"临时方案"等理由遗留问题
- ❌ 在任务报告中标注"遗留问题"而不解决
- ❌ 编译错误未修复就提交
- ❌ 测试失败未修复就提交

**正确做法**：
- ✅ 所有问题必须解决（编译错误、测试失败、代码规范问题等）
- ✅ 除非能证明该问题无法解决（如框架限制、第三方库bug等）

---

### 规则10：数据库变更必须通过Flyway

**✅ 正确流程**：
```bash
# 1. 创建迁移脚本
backend/dataengine-backend/src/main/resources/db/migration/V24__create_table.sql

# 2. 语法验证
docker exec -i dataengine-postgres psql -U postgres -d dataengine_dev < V24__create_table.sql

# 3. 执行迁移
mvn flyway:migrate

# 4. 检查历史
mvn flyway:info
```

**❌ 禁止**：
- ❌ 手动修改数据库结构
- ❌ 跳过验证直接提交代码

---

## 🔗 相关文档

- [完整后端规范](../01-backend/)
- [完整前端规范](../02-frontend/)
- [完整数据库规范](../03-database/)
- [完整测试规范](../04-testing/)
- [完整工作流规范](../05-workflow/)

---

**文档元数据**：
- **维护者**: 技术架构团队
- **最后更新**: 2026-03-14
- **版本**: 2.0.0（派对模式审查优化版）
- **状态**: active
