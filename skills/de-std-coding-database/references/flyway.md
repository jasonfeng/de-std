# Flyway迁移规范

> 本项目使用Flyway进行数据库版本管理，所有数据库变更必须通过Flyway脚本执行。

---

## ⚠️ 核心铁律

**任何表结构的变动（CREATE / ALTER / DROP / RENAME / ADD COLUMN / MODIFY COLUMN 等 DDL 操作）必须且只能通过 Flyway 迁移脚本完成，并由应用程序启动时自动执行。**

| 禁止行为 | 正确做法 |
|----------|----------|
| 手动连接数据库执行 DDL | 编写 Flyway 迁移脚本，随应用启动自动执行 |
| 通过 psql / DBeaver / pgAdmin 等工具直接修改表结构 | 将 SQL 放入 `db/migration/` 目录，通过 `mvn flyway:migrate` 或应用启动触发 |
| 在 Java 代码中拼接执行 DDL（`@Update("ALTER TABLE ...")`） | 所有 DDL 必须是独立的 `.sql` 迁移文件 |
| 在 CI/CD 流水线中绕过 Flyway 直接执行 SQL | 统一由 Flyway 管理版本和执行顺序 |

**违规后果**：数据库版本不一致，迁移历史断裂，其他环境无法部署。

---

## 📁 迁移脚本位置

```
backend/dataengine-api/src/main/resources/db/migration/
├── V1__create_user_tables.sql
├── V2__create_dict_tables.sql
├── V3__create_form_example_tables.sql
├── ...
└── V24__create_example_table.sql
```

---

## 📝 迁移脚本命名规范

### 基本格式

```
V{版本号}__{描述}.sql
```

| 部分 | 说明 | 示例 |
|------|------|------|
| `V` | 版本标记（Versioned） | `V1__`、`V24__` |
| `{版本号}` | 数字序列 | `1`、`2`、`24` |
| `__` | 双下划线分隔符 | `__` |
| `{描述}` | 脚本描述（蛇形命名） | `create_user_tables` |

### 命名示例

| 脚本名 | 说明 |
|--------|------|
| `V1__create_user_tables.sql` | 创建用户表 |
| `V2__create_dict_tables.sql` | 创建字典表 |
| `V3__add_user_index.sql` | 添加用户索引 |
| `V4__modify_user_column.sql` | 修改用户字段 |

---

## ✅ 迁移脚本验证流程

### 1. 创建迁移脚本

在 `src/main/resources/db/migration/` 目录下创建新的迁移脚本。

**示例**：
```sql
-- ============================================================================
-- 示例表创建脚本
-- Story: 2.X - 示例功能
-- Author: Fengzhongtian
-- Date: 2026-03-10
-- ============================================================================

-- 1. 创建表
CREATE TABLE t_example (
    id BIGSERIAL PRIMARY KEY,
    example_code VARCHAR(50) NOT NULL,
    example_name VARCHAR(100) NOT NULL,
    description TEXT,
    sort_order INTEGER DEFAULT 0,
    deleted INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- 唯一性约束
    CONSTRAINT uk_example_code UNIQUE (example_code)
);

-- 2. 创建索引
CREATE INDEX IF NOT EXISTS idx_t_example_code
ON t_example(example_code)
WHERE deleted = 0;

-- 3. 添加注释
COMMENT ON TABLE t_example IS '示例表';
COMMENT ON COLUMN t_example.id IS '主键ID';
COMMENT ON COLUMN t_example.example_code IS '示例代码（唯一）';
COMMENT ON COLUMN t_example.deleted IS '软删除标记（0=未删除，1=已删除）';

-- 4. 创建触发器函数
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 5. 创建触发器
DROP TRIGGER IF EXISTS trg_t_example_updated ON t_example;
CREATE TRIGGER trg_t_example_updated
  BEFORE UPDATE ON t_example
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

### 2. 语法验证

```bash
# 进入项目目录
cd backend/dataengine-api

# 启动PostgreSQL环境（如果未启动）
docker-compose up -d dataengine-postgres

# 验证迁移脚本语法
docker exec -i dataengine-postgres psql -U postgres -d dataengine_dev < src/main/resources/db/migration/V24__create_example_table.sql
```

**验证要点**：
- ✅ 语法无错误
- ✅ 对象创建成功
- ✅ 约束、索引创建成功
- ✅ 触发器创建成功

### 3. 迁移执行验证

```bash
# 执行Flyway迁移
mvn flyway:migrate

# 检查迁移历史
mvn flyway:info
```

**输出示例**：
```
+-----------+---------+----------------------+---------------------+---------+
| Category   | Version | Description          | Installed On        | State   |
+-----------+---------+----------------------+---------------------+---------+
| Versioned  | 1       | create user tables    | 2026-03-10 10:00:00 | Success |
| Versioned  | 2       | create dict tables    | 2026-03-10 10:05:00 | Success |
| ...        | ...     | ...                  | ...                 | ...     |
| Versioned  | 24      | create example table  | 2026-03-10 12:00:00 | Success |
+-----------+---------+----------------------+---------------------+---------+
```

### 4. 功能完成标准

- ✅ 迁移脚本在PostgreSQL环境语法验证通过
- ✅ Flyway迁移执行成功
- ✅ 检查迁移历史确认脚本已执行
- ❌ 禁止跳过验证直接提交代码

---

## ⚠️ 禁止事项

### ❌ 禁止手动修改数据库

**错误做法**：
```bash
# 直接连接数据库执行SQL
psql -U postgres -d dataengine_dev
# 执行DDL语句
CREATE TABLE t_new_table (...);
```

**错误做法（通过工具执行）**：
```bash
# 通过 DBeaver / pgAdmin / Navicat 等可视化工具执行 DDL
# 通过 CI/CD 脚本直接运行 psql 执行 SQL 文件
psql -U postgres -d dataengine_dev -f changes.sql
```

**正确做法**：
```bash
# 1. 编写 Flyway 迁移脚本
# src/main/resources/db/migration/V25__create_new_table.sql

# 2. 本地语法验证（仅验证，不算正式执行）
docker exec -i dataengine-postgres psql -U postgres -d dataengine_dev < V25__create_new_table.sql

# 3. 通过 Flyway 正式执行迁移
mvn flyway:migrate
# 或直接启动应用程序，Flyway 会自动执行未应用的迁移脚本
```

### ❌ 禁止修改已执行的迁移脚本

**Flyway校验机制**：
- 每个迁移脚本执行后会记录校验和（checksum）
- 修改已执行的脚本会导致校验失败
- 解决方法：创建新的迁移脚本

**错误做法**：
```sql
-- 修改V1__create_user_tables.sql（已执行）
-- ALTER TABLE t_user ADD COLUMN new_column TEXT;
```

**正确做法**：
```sql
-- 创建新的迁移脚本
-- V25__add_user_column.sql

ALTER TABLE t_user ADD COLUMN new_column TEXT;
```

### ❌ 禁止跳过版本号

**错误做法**：
```
V1__create_user_tables.sql
V2__create_dict_tables.sql
V10__add_index.sql  -- ❌ 跳过版本号
```

**正确做法**：
```
V1__create_user_tables.sql
V2__create_dict_tables.sql
V3__add_index.sql  -- ✅ 顺序递增
```

---

## 🔄 迁移脚本最佳实践

### 1. 使用事务

```sql
BEGIN;

-- DDL语句
CREATE TABLE t_example (...);
ALTER TABLE t_example ADD COLUMN ...;

-- 提交事务
COMMIT;
```

### 2. 使用IF NOT EXISTS

```sql
-- 创建表
CREATE TABLE IF NOT EXISTS t_example (...);

-- 创建索引
CREATE INDEX IF NOT EXISTS idx_t_example_code ON t_example(code);

-- 创建函数
CREATE OR REPLACE FUNCTION example_function() ...
```

### 3. 添加注释

```sql
-- 脚本头部注释
-- ============================================================================
-- 脚本描述
-- Story: X.X - 用户故事
-- Author: 作者名
-- Date: YYYY-MM-DD
-- ============================================================================

-- 对象注释
COMMENT ON TABLE t_example IS '示例表';
COMMENT ON COLUMN t_example.id IS '主键ID';
```

### 4. 软删除处理

```sql
-- 添加deleted字段
ALTER TABLE t_example ADD COLUMN deleted INT DEFAULT 0;

-- 创建部分索引
CREATE INDEX idx_t_example_deleted
ON t_example(id)
WHERE deleted = 0;

-- 更新现有数据（如果需要）
UPDATE t_example SET deleted = 0 WHERE deleted IS NULL;
```

---

## 🛠️ 常用命令

### 迁移相关

```bash
# 执行迁移
mvn flyway:migrate

# 检查迁移历史
mvn flyway:info

# 检查待执行的迁移
mvn flyway:validate

# 清理数据库（开发环境）
mvn flyway:clean

-- ⚠️ 警告：clean会删除所有表，仅在开发环境使用
```

### 修复迁移

**方案1：回滚到指定版本（开发环境）**

```bash
# 查看迁移历史
mvn flyway:info

# 回滚到指定版本
mvn flyway:undo -target=23

# ⚠️ 需要配置flyway.undo.enabled=true
```

**方案2：创建修复迁移（生产环境推荐）**

```bash
# 如果V24有问题，创建V24.1进行修复
V24__create_example_table.sql  -- 有问题的脚本
V24.1__fix_example_table.sql   -- 修复脚本
```

---

## 📊 迁移脚本模板

### 创建表模板

```sql
-- ============================================================================
-- {表名称}表创建脚本
-- Story: X.X - {用户故事}
-- Author: {作者}
-- Date: {日期}
-- ============================================================================

-- 1. 创建表
CREATE TABLE t_{表名} (
    id BIGSERIAL PRIMARY KEY,
    {字段1} {类型} NOT NULL,
    {字段2} {类型},
    deleted INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- 约束
    CONSTRAINT uk_{表}_{字段} UNIQUE ({字段})
);

-- 2. 创建索引
CREATE INDEX IF NOT EXISTS idx_t_{表}_{字段}
ON t_{表}({字段})
WHERE deleted = 0;

-- 3. 添加注释
COMMENT ON TABLE t_{表} IS '{表说明}';
COMMENT ON COLUMN t_{表}.id IS '主键ID';
COMMENT ON COLUMN t_{表}.deleted IS '软删除标记（0=未删除，1=已删除）';

-- 4. 创建触发器（如需要）
CREATE TRIGGER trg_t_{表}_updated
  BEFORE UPDATE ON t_{表}
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

### 添加字段模板

```sql
-- ============================================================================
-- 添加{字段}字段
-- Story: X.X - {用户故事}
-- Author: {作者}
-- Date: {日期}
-- ============================================================================

-- 添加字段
ALTER TABLE t_{表}
ADD COLUMN {字段} {类型};

-- 添加注释
COMMENT ON COLUMN t_{表}.{字段} IS '{字段说明}';

-- 如果需要，创建索引
CREATE INDEX IF NOT EXISTS idx_t_{表}_{字段}
ON t_{表}({字段})
WHERE deleted = 0;
```

---

## 🔗 相关文档

- [PostgreSQL语法规范](./postgresql.md) - PostgreSQL详细语法
- [数据库命名规范](./naming-conventions.md) - 完整命名规范
- [MyBatis映射规范](./mybatis-mapping.md) - MyBatis XML SQL规范

---

**最后更新**：2026-03-10