# PostgreSQL语法规范

> 本项目使用PostgreSQL 14+数据库，所有SQL必须遵循本规范。

---

## ⚠️ 禁止MySQL语法

### 禁止使用的MySQL特有语法

| MySQL语法 | 原因 | PostgreSQL替代方案 |
|-----------|------|-------------------|
| `id BIGINT PRIMARY KEY` | BIGINT不自增 | `id BIGSERIAL PRIMARY KEY` |
| `column_name VARCHAR(50) COMMENT '说明'` | 行内COMMENT不支持 | 使用`COMMENT ON COLUMN`单独语句 |
| `IF NOT EXISTS` 在CREATE TABLE中 | 语法不同 | Flyway自动处理版本控制 |
| 反引号\`column\` | MySQL语法 | 使用双引号"column"或不使用引号 |
| ENGINE=InnoDB | MySQL特有 | PostgreSQL不需要 |
| AUTO_INCREMENT | MySQL语法 | 使用SERIAL/BIGSERIAL |
| ``ENUM('a','b','c')`` | 语法不同 | 使用TEXT+CHECK约束或自定义类型 |
| `TINYINT` | MySQL特有 | 使用SMALLINT或INTEGER |
| `UNSIGNED` | MySQL特有 | 不使用，使用更大类型 |
| `` \` \` `` | MySQL转义 | 使用双引号或标准SQL转义 |

---

## ✅ 正确的PostgreSQL语法

### 主键定义

**❌ 错误（MySQL）**：
```sql
CREATE TABLE t_user (
    id BIGINT PRIMARY KEY,
    ...
);
```

**✅ 正确（PostgreSQL）**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    ...
);
```

---

### 添加注释

**❌ 错误（MySQL）**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL COMMENT '用户名'
);
```

**✅ 正确（PostgreSQL）**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL
);

-- 添加注释使用单独语句
COMMENT ON TABLE t_user IS '用户表';
COMMENT ON COLUMN t_user.id IS '主键ID';
COMMENT ON COLUMN t_user.user_name IS '用户名';
```

---

### 引号使用

**❌ 错误（MySQL）**：
```sql
CREATE TABLE t_user (
    `id` BIGSERIAL PRIMARY KEY,
    `user_name` VARCHAR(50) NOT NULL
);

SELECT * FROM `t_user` WHERE `user_name` = 'test';
```

**✅ 正确（PostgreSQL）**：
```sql
-- 不使用引号（推荐）
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    user_name VARCHAR(50) NOT NULL
);

SELECT * FROM t_user WHERE user_name = 'test';

-- 或使用双引号（用于特殊字符或保留字）
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    "user_name" VARCHAR(50) NOT NULL  -- 特殊情况使用双引号
);
```

---

### 自增字段

**❌ 错误（MySQL）**：
```sql
CREATE TABLE t_user (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    ...
);
```

**✅ 正确（PostgreSQL）**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,  -- 等价于 BIGINT + SERIAL
    ...
);

-- 或使用SMALLINT
CREATE TABLE t_config (
    id SMALLSERIAL PRIMARY KEY,
    ...
);
```

**SERIAL类型对照表**：
| 类型 | 说明 | 范围 |
|------|------|------|
| `SMALLSERIAL` | SMALLINT自增 | 1 to 32767 |
| `SERIAL` | INTEGER自增 | 1 to 2147483647 |
| `BIGSERIAL` | BIGINT自增 | 1 to 9223372036854775807 |

---

### 枚举类型

**❌ 错误（MySQL）**：
```sql
CREATE TABLE t_user (
    status ENUM('active', 'inactive', 'deleted')
);
```

**✅ 正确（PostgreSQL）**：

**方案1：使用TEXT+CHECK约束（推荐）**：
```sql
CREATE TABLE t_user (
    status TEXT NOT NULL,
    CONSTRAINT ck_user_status CHECK (status IN ('active', 'inactive', 'deleted'))
);
```

**方案2：使用自定义ENUM类型**：
```sql
-- 创建自定义类型
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'deleted');

-- 使用自定义类型
CREATE TABLE t_user (
    status user_status NOT NULL
);
```

---

### 布尔类型

**PostgreSQL支持原生BOOLEAN类型**：
```sql
CREATE TABLE t_user (
    is_active BOOLEAN DEFAULT TRUE,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- 查询时可以使用BOOLEAN字面值
SELECT * FROM t_user WHERE is_active = TRUE;
SELECT * FROM t_user WHERE is_deleted = FALSE;
```

**注意**：项目中使用`deleted INT`作为软删除标记，不使用BOOLEAN类型。

---

## 📅 时间类型

### TIMESTAMP vs TIMESTAMPTZ

| 类型 | 说明 | 使用场景 |
|------|------|---------|
| `TIMESTAMP` | 不带时区 | 项目推荐使用 |
| `TIMESTAMPTZ` | 带时区 | 需要时区感知的场景 |

**示例**：
```sql
CREATE TABLE t_user (
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**默认值**：
| 默认值 | 说明 |
|--------|------|
| `CURRENT_TIMESTAMP` | 当前时间戳 |
| `CURRENT_DATE` | 当前日期 |
| `CURRENT_TIME` | 当前时间 |

---

## 📝 字符串类型

| 类型 | 说明 | 最大长度 | 使用场景 |
|------|------|---------|---------|
| `CHAR(n)` | 定长字符串 | 1GB | 固定长度数据 |
| `VARCHAR(n)` | 变长字符串 | 1GB | 通用字符串 |
| `TEXT` | 无限制文本 | 1GB | 长文本内容 |

**注意**：PostgreSQL的VARCHAR和CHAR最大都是1GB，不是MySQL的65535。

**示例**：
```sql
CREATE TABLE t_article (
    title VARCHAR(200) NOT NULL,     -- 标题
    summary VARCHAR(500),            -- 摘要
    content TEXT                     -- 内容（无长度限制）
);
```

---

## 🔢 数值类型

| 类型 | 说明 | 范围 | 使用场景 |
|------|------|------|---------|
| `SMALLINT` | 小整数 | -32768 to 32767 | 小范围数值 |
| `INTEGER` | 整数 | -2147483648 to 2147483647 | 通用整数 |
| `BIGINT` | 大整数 | -9223372036854775808 to 9223372036854775807 | 主键、大数值 |
| `NUMERIC(p,s)` | 精确小数 | 自定义 | 金额、精确计算 |
| `REAL` | 浮点数 | 6位精度 | 科学计算 |
| `DOUBLE PRECISION` | 双精度浮点 | 15位精度 | 一般计算 |

**示例**：
```sql
CREATE TABLE t_product (
    id BIGSERIAL PRIMARY KEY,
    stock INTEGER DEFAULT 0,
    price NUMERIC(10, 2),      -- 10位数字，2位小数
    discount REAL,
    rating DOUBLE PRECISION
);
```

---

## 🎯 部分索引

PostgreSQL支持部分索引（带WHERE条件的索引），非常适合软删除场景。

### 基本语法

```sql
CREATE INDEX idx_{表}_{字段}
ON {表}({字段})
WHERE {条件};
```

### 示例

```sql
-- 只索引未删除的用户
CREATE INDEX idx_t_user_email_active
ON t_user(email)
WHERE deleted = 0;

-- 复合部分索引
CREATE INDEX idx_t_order_status_created_active
ON t_order(status, created_at)
WHERE deleted = 0;

-- 包含函数的部分索引
CREATE INDEX idx_t_user_name_lower
ON t_user(LOWER(username))
WHERE deleted = 0;
```

### 部分索引的优势

| 优势 | 说明 |
|------|------|
| 索引更小 | 只索引符合条件的行 |
| 查询更快 | 减少索引扫描范围 |
| 写入更快 | 减少索引维护开销 |

---

## ⚡ 触发器

### 触发器函数

PostgreSQL使用PL/pgSQL编写触发器函数。

```sql
-- 自动更新updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 删除前归档
CREATE OR REPLACE FUNCTION archive_user_function()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO t_user_archive (id, username, email, archived_at)
    VALUES (OLD.id, OLD.username, OLD.email, CURRENT_TIMESTAMP);
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

### 创建触发器

```sql
-- 更新触发器
DROP TRIGGER IF EXISTS trg_t_user_updated ON t_user;
CREATE TRIGGER trg_t_user_updated
  BEFORE UPDATE ON t_user
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- 删除触发器
DROP TRIGGER IF EXISTS trg_t_user_deleted ON t_user;
CREATE TRIGGER trg_t_user_deleted
  BEFORE DELETE ON t_user
  FOR EACH ROW
  EXECUTE FUNCTION archive_user_function();
```

### 触发器时机

| 时机 | 说明 | 使用场景 |
|------|------|---------|
| `BEFORE INSERT` | 插入前 | 自动填充字段、数据校验 |
| `AFTER INSERT` | 插入后 | 记录日志、触发后续操作 |
| `BEFORE UPDATE` | 更新前 | 自动更新字段、数据校验 |
| `AFTER UPDATE` | 更新后 | 记录日志、同步数据 |
| `BEFORE DELETE` | 删除前 | 软删除、归档数据 |
| `AFTER DELETE` | 删除后 | 清理关联数据、记录日志 |

---

## 🔄 事务与锁

### 事务隔离级别

PostgreSQL默认使用READ COMMITTED隔离级别。

```sql
-- 设置事务隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### 行锁

```sql
-- SELECT FOR UPDATE（排他锁）
SELECT * FROM t_user WHERE id = 1 FOR UPDATE;

-- SELECT FOR SHARE（共享锁）
SELECT * FROM t_user WHERE id = 1 FOR SHARE;
```

### 表锁

```sql
-- LOCK TABLE（表锁）
LOCK TABLE t_user IN ACCESS EXCLUSIVE MODE;
LOCK TABLE t_user IN SHARE MODE;
```

---

## 🔍 JSON类型

PostgreSQL支持原生JSON和JSONB类型。

| 类型 | 说明 | 推荐使用 |
|------|------|---------|
| `JSON` | 保留原始格式 | 需要保留格式的场景 |
| `JSONB` | 二进制存储，支持索引 | **推荐**，性能更好 |

**示例**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    preferences JSONB NOT NULL DEFAULT '{}'::jsonb
);

-- JSONB查询
SELECT * FROM t_user WHERE preferences->>'theme' = 'dark';

-- JSONB索引
CREATE INDEX idx_t_user_preferences ON t_user USING GIN (preferences);
```

---

## 📚 实用函数

### 字符串函数

```sql
-- 转小写
SELECT LOWER(username) FROM t_user;

-- 转大写
SELECT UPPER(username) FROM t_user;

-- 字符串拼接
SELECT first_name || ' ' || last_name AS full_name FROM t_user;

-- 子字符串
SELECT SUBSTRING(email FROM 1 FOR POSITION('@' IN email) - 1) FROM t_user;

-- 去除空格
SELECT TRIM(username) FROM t_user;
```

### 时间函数

```sql
-- 当前时间
SELECT CURRENT_TIMESTAMP;
SELECT CURRENT_DATE;
SELECT CURRENT_TIME;

-- 时间加法
SELECT created_at + INTERVAL '1 day' FROM t_user;
SELECT updated_at + INTERVAL '1 month' FROM t_user;

-- 时间比较
SELECT * FROM t_user WHERE created_at > CURRENT_TIMESTAMP - INTERVAL '7 days';

-- 时间格式化
SELECT TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI:SS') FROM t_user;
```

### 聚合函数

```sql
-- COUNT
SELECT COUNT(*) FROM t_user WHERE deleted = 0;

-- SUM
SELECT SUM(amount) FROM t_order WHERE deleted = 0;

-- AVG
SELECT AVG(price) FROM t_product WHERE deleted = 0;

-- MAX/MIN
SELECT MAX(created_at), MIN(created_at) FROM t_user;

-- GROUP BY
SELECT status, COUNT(*) FROM t_user GROUP BY status;

-- HAVING
SELECT status, COUNT(*) AS cnt FROM t_user GROUP BY status HAVING cnt > 10;
```

---

## 🔗 相关文档

- [数据库命名规范](./naming-conventions.md) - 完整的命名规范
- [Flyway迁移规范](./flyway.md) - 迁移流程
- [MyBatis映射规范](./mybatis-mapping.md) - MyBatis XML SQL规范

---

**最后更新**：2026-03-10