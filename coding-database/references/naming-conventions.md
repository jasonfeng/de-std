# 数据库命名规范

> 本文定义数据库对象的命名规范，确保命名的一致性和可维护性。

---

## 📋 命名原则

| 原则 | 说明 |
|------|------|
| **小写** | 所有名称使用小写 |
| **蛇形命名** | 使用下划线分隔单词 |
| **表名前缀** | 表名使用`t_`前缀 |
| **简短明确** | 名称应简短且能表达含义 |
| **避免保留字** | 不使用SQL保留字 |

---

## 表名规范

### 基本规范

| 规范 | 示例 |
|------|------|
| 使用`t_`前缀 | `t_user`、`t_order`、`t_product` |
| 蛇形命名 | `t_user_profile`、`t_order_item` |
| 名词单数 | `t_user`（不是`t_users`） |

### 表名与Entity映射

| 数据库表 | 实体类 | 说明 |
|---------|--------|------|
| `t_user` | `UserEntity` | ✅ 正确 |
| `t_user_profile` | `UserProfileEntity` | ✅ 正确 |
| `user` | `UserEntity` | ❌ 缺少t_前缀 |
| `t_user` | `User` | ❌ 缺少Entity后缀 |

---

## 列名规范

### 基本规范

| 规范 | 示例 |
|------|------|
| 蛇形命名 | `user_name`、`created_at`、`is_deleted` |
| 布尔列使用`is_`前缀 | `is_active`、`is_deleted` |
| 时间列使用`_at`后缀 | `created_at`、`updated_at` |
| 外键使用`_id`后缀 | `user_id`、`department_id` |

### 通用列规范

| 列名 | 类型 | 说明 |
|------|------|------|
| `id` | BIGSERIAL | 主键ID |
| `deleted` | INT | 软删除标记（0=未删除，1=已删除） |
| `created_at` | TIMESTAMP | 创建时间 |
| `updated_at` | TIMESTAMP | 更新时间 |
| `created_by` | BIGINT | 创建人ID |
| `updated_by` | BIGINT | 更新人ID |
| `sort_order` | INTEGER | 排序序号 |

---

## 约束命名规范

### 主键约束

**命名格式**：`pk_{表}_id`

| 表名 | 主键约束名 | 示例 |
|------|-----------|------|
| `t_user` | `pk_t_user_id` | `CONSTRAINT pk_t_user_id PRIMARY KEY (id)` |

### 唯一性约束

**命名格式**：`uk_{表}_{字段}`

| 表名 | 约束名 | 示例 |
|------|--------|------|
| `t_user` | `uk_t_user_username` | `CONSTRAINT uk_t_user_username UNIQUE (username)` |
| `t_user_role` | `uk_t_user_role_user_role` | `CONSTRAINT uk_t_user_role_user_role UNIQUE (user_id, role_id)` |

### 外键约束

**命名格式**：`fk_{子表}_{父表}_{字段}`

| 子表 | 父表 | 外键约束名 | 示例 |
|------|------|-----------|------|
| `t_order` | `t_user` | `fk_t_order_user_user_id` | `CONSTRAINT fk_t_order_user_user_id FOREIGN KEY (user_id) REFERENCES t_user(id)` |
| `t_order_item` | `t_order` | `fk_t_order_item_order_order_id` | `CONSTRAINT fk_t_order_item_order_order_id FOREIGN KEY (order_id) REFERENCES t_order(id)` |

**外键删除策略**：
| 策略 | 说明 | 使用场景 |
|------|------|---------|
| `ON DELETE RESTRICT` | 禁止删除（默认） | 有子记录时禁止删除父记录 |
| `ON DELETE CASCADE` | 级联删除 | 删除父记录时同时删除子记录 |
| `ON DELETE SET NULL` | 设为NULL | 删除父记录时子记录外键设为NULL |

### CHECK约束

**命名格式**：`ck_{表}_{条件}`

| 表名 | 约束名 | 示例 |
|------|--------|------|
| `t_user` | `ck_t_user_age_positive` | `CONSTRAINT ck_t_user_age_positive CHECK (age >= 0)` |

---

## 索引命名规范

### 基本命名格式

**命名格式**：`idx_{表}_{字段}`

| 表名 | 索引名 | 示例 |
|------|--------|------|
| `t_user` | `idx_t_user_email` | `CREATE INDEX idx_t_user_email ON t_user(email)` |
| `t_user` | `idx_t_user_name_email` | `CREATE INDEX idx_t_user_name_email ON t_user(name, email)` |

### 使用表名前缀避免冲突

**❌ 不推荐**（可能冲突）：
```sql
CREATE INDEX idx_name ON t_user(name);
CREATE INDEX idx_name ON t_order(name);  -- 冲突
```

**✅ 推荐**（使用表名前缀）：
```sql
CREATE INDEX idx_t_user_name ON t_user(name);
CREATE INDEX idx_t_order_name ON t_order(name);  -- 无冲突
```

### 部分索引

**命名格式**：`idx_{表}_{字段}_partial` 或 `idx_{表}_{字段}_active`

```sql
-- 软删除场景的部分索引
CREATE INDEX idx_t_user_email_active
ON t_user(email)
WHERE deleted = 0;

-- 复合部分索引
CREATE INDEX idx_t_order_status_created_active
ON t_order(status, created_at)
WHERE deleted = 0;
```

---

## 触发器命名规范

### 基本命名格式

**命名格式**：`trg_{表}_{操作}`

| 操作 | 后缀 | 示例 |
|------|------|------|
| 更新 | `_updated` | `trg_t_user_updated` |
| 插入 | `_inserted` | `trg_t_log_inserted` |
| 删除 | `_deleted` | `trg_t_archive_deleted` |
| 所有 | `_all` | `trg_t_user_all` |

### 触发器示例

```sql
-- 自动更新 updated_at
CREATE TRIGGER trg_t_user_updated
  BEFORE UPDATE ON t_user
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- 删除前归档
CREATE TRIGGER trg_t_user_deleted
  BEFORE DELETE ON t_user
  FOR EACH ROW
  EXECUTE FUNCTION archive_user_function();
```

---

## 函数命名规范

### 基本命名格式

**命名格式**：`{动词}_{名词}` 或 `{功能}_function`

| 函数名 | 说明 | 示例 |
|--------|------|------|
| `update_updated_at_column` | 更新updated_at字段 | 通用触发器函数 |
| `archive_user_function` | 归档用户数据 | 业务函数 |
| `calculate_discount` | 计算折扣 | 业务函数 |

### 函数示例

```sql
-- 通用触发器函数
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 业务函数
CREATE OR REPLACE FUNCTION calculate_discount(total_amount NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
    IF total_amount > 1000 THEN
        RETURN total_amount * 0.9;  -- 10%折扣
    ELSE
        RETURN total_amount;
    END IF;
END;
$$ LANGUAGE plpgsql;
```

---

## 视图命名规范

### 基本命名格式

**命名格式**：`v_{表}` 或 `v_{功能}`

| 类型 | 格式 | 示例 |
|------|------|------|
| 单表视图 | `v_{表}` | `v_user_active`（活跃用户视图） |
| 多表视图 | `v_{功能}` | `v_order_summary`（订单汇总视图） |

### 视图示例

```sql
-- 活跃用户视图
CREATE OR REPLACE VIEW v_user_active AS
SELECT id, username, email, created_at
FROM t_user
WHERE deleted = 0 AND is_active = 1;

-- 订单汇总视图
CREATE OR REPLACE VIEW v_order_summary AS
SELECT
    o.id AS order_id,
    u.username,
    COUNT(oi.id) AS item_count,
    SUM(oi.amount) AS total_amount
FROM t_order o
JOIN t_user u ON o.user_id = u.id
JOIN t_order_item oi ON o.id = oi.order_id
WHERE o.deleted = 0
GROUP BY o.id, u.username;
```

---

## 序列命名规范

**注意**：本项目使用BIGSERIAL自动创建序列，一般不需要手动创建序列。

### 基本命名格式

**命名格式**：`seq_{表}_{字段}`

| 表名 | 序列名 | 说明 |
|------|--------|------|
| `t_order` | `seq_t_order_order_no` | 订单号序列 |

### 序列示例

```sql
-- 订单号序列（如果需要自定义序列）
CREATE SEQUENCE seq_t_order_order_no
    START WITH 100000
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;

-- 使用序列
INSERT INTO t_order (id, order_no, ...)
VALUES (nextval('seq_t_order_order_no'), ...);
```

---

## ⭐ 主键ID策略

### 必须使用雪花算法

**MyBatis Plus配置**：
```java
@TableId(value = "id", type = IdType.ASSIGN_ID)
private Long id;
```

**PostgreSQL配合**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    ...
);
```

**为什么使用雪花算法**？

| 优势 | 说明 |
|------|------|
| 分布式友好 | 支持多实例部署，不会ID冲突 |
| 性能更好 | 应用层生成ID，无数据库开销 |
| ID可提前赋值 | 插入前就知道ID，可用于外键关联 |
| 趋势递增 | ID趋势递增，索引友好 |
| 时序有序 | ID包含时间信息，天然有序 |

> 详见：[MyBatis Plus主键策略](../01-backend/mybatis-plus.md#主键id策略)

---

## 🔗 相关文档

- [PostgreSQL语法规范](./postgresql.md) - PostgreSQL详细语法规范
- [Flyway迁移规范](./flyway.md) - 数据库迁移流程
- [MyBatis映射规范](./mybatis-mapping.md) - MyBatis XML SQL规范

---

**最后更新**：2026-03-10