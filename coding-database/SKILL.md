---
name: coding-database
description: 数据库 PostgreSQL/Flyway/MyBatis 映射规范。写 SQL 或迁移脚本前必须调用。
---

# 数据库编码规范

## 触发时机

- "创建数据库表"
- "写 SQL 查询"
- "添加 Flyway 迁移脚本"
- "写 MyBatis Mapper XML"
- "修改表结构/添加字段"

---

## PostgreSQL 规范

### 禁止 MySQL 语法

```sql
-- ❌ 禁止（MySQL）
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) COMMENT '用户名'
) ENGINE=InnoDB;

-- ✅ 正确（PostgreSQL）
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);
COMMENT ON TABLE t_user IS '用户表';
COMMENT ON COLUMN t_user.name IS '用户名';
```

### 禁止的 MySQL 特性

| MySQL 语法 | PostgreSQL 替代 |
|------------|----------------|
| `AUTO_INCREMENT` | `BIGSERIAL` 或 `SERIAL` |
| `COMMENT '...'` | `COMMENT ON COLUMN` |
| `ENGINE=InnoDB` | 无需指定 |
| `IFNULL()` | `COALESCE()` |
| `LIMIT ?, ?` | `LIMIT offset, count` |
| `REPLACE INTO` | `INSERT ... ON CONFLICT` |

---

## 表命名规范

### 表名：t_前缀

| 表名 | 状态 |
|------|------|
| `t_user` | ✅ 正确 |
| `t_order` | ✅ 正确 |
| `user` | ❌ 缺少 t_ 前缀 |

### 字段命名

```sql
-- ✅ 正确命名
id              -- 主键
created_at      -- 创建时间
updated_at      -- 更新时间
deleted         -- 逻辑删除标记（Integer）
user_name       -- 业务字段（snake_case）
```

### 主键策略

```sql
-- ✅ 雪花算法（推荐）
id BIGINT PRIMARY KEY

-- ❌ 自增（禁止分布式环境使用）
id BIGSERIAL PRIMARY KEY
```

---

## Flyway 规范

### 迁移脚本命名

```
V{version}__{description}.sql

示例：
V1__create_user_table.sql
V2__add_user_email_column.sql
V24__create_order_table.sql
```

### 迁移脚本示例

```sql
-- V24__create_order_table.sql
CREATE TABLE t_order (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted INTEGER NOT NULL DEFAULT 0
);

COMMENT ON TABLE t_order IS '订单表';
COMMENT ON COLUMN t_order.status IS '订单状态：pending/paid/completed/cancelled';

-- 索引
CREATE INDEX idx_order_user_id ON t_order(user_id);
CREATE INDEX idx_order_status ON t_order(status);
```

### 迁移流程

```bash
# 1. 创建迁移脚本
src/main/resources/db/migration/V24__create_table.sql

# 2. 语法验证
docker exec -i dataengine-postgres psql -U postgres -d dataengine_dev < V24__create_table.sql

# 3. 执行迁移
mvn flyway:migrate

# 4. 检查历史
mvn flyway:info
```

---

## MyBatis 映射规范

### 禁止注解定义 SQL

```java
// ❌ 禁止
@Select("SELECT * FROM t_user WHERE id = #{id}")
UserEntity findById(Long id);

// ✅ 正确：XML方式
public interface UserMapper {
    UserEntity findById(@Param("id") Long id);
}
```

### XML 文件位置

```
resources/mapper/
├── user/
│   └── UserMapper.xml
└── order/
    └── OrderMapper.xml
```

### 禁止 Java 拼接 SQL

```java
// ❌ 禁止
String sql = "SELECT * FROM t_user WHERE status = '" + status + "'";

// ✅ 正确：使用 MyBatis 参数
// XML:
// <select id="findByStatus">
//     SELECT * FROM t_user WHERE status = #{status}
// </select>
```

### 复杂排序逻辑

```xml
<!-- 在 XML 中使用 choose/when 实现复杂排序 -->
<select id="findWithSort" resultType="...Entity">
    SELECT * FROM t_user
    WHERE deleted = 0
    ORDER BY
    <choose>
        <when test="sortBy == 'name' and sortOrder == 'asc'">user_name ASC</when>
        <when test="sortBy == 'name' and sortOrder == 'desc'">user_name DESC</when>
        <when test="sortBy == 'createTime' and sortOrder == 'asc'">created_at ASC</when>
        <otherwise>updated_at DESC</otherwise>
    </choose>
</select>
```

---

## 实体类规范

### 标准实体结构

```java
@Data
@Accessors(chain = true)
@TableName("t_user")
public class UserEntity {
    @TableId(value = "id", type = IdType.ASSIGN_ID)
    private Long id;

    @TableField("user_name")
    private String userName;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;

    @TableLogic
    private Integer deleted;
}
```

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| MySQL 语法 | PostgreSQL 不兼容 |
| `@Select` 注解 | SQL 无法审查 |
| Java 拼接 SQL | SQL 注入风险 |
| 手动修改数据库 | Flyway 版本失控 |
| 缺少 t_ 前缀 | 命名规范违规 |
| 自增主键（分布式） | ID 冲突风险 |

---

## 详细规范

- **[postgresql.md](references/postgresql.md)** — PostgreSQL 语法规范、禁止的 MySQL 特性、数据类型映射
- **[naming-conventions.md](references/naming-conventions.md)** — 表命名、字段命名、主键策略、索引规范
- **[flyway.md](references/flyway.md)** — 迁移脚本命名、版本管理、回滚策略
- **[mybatis-mapping.md](references/mybatis-mapping.md)** — XML映射规范、复杂SQL编写、动态SQL
