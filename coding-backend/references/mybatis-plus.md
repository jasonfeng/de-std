# MyBatis Plus规范

> 本规范定义MyBatis Plus框架的使用规范，确保数据访问层的一致性。

---

## 📋 目录

- [Entity定义规范](#entity定义规范)
- [主键ID策略](#主键id策略-⚠️强制)
- [Mapper接口规范](#mapper接口规范)
- [SQL规范](#sql规范-⚠️强制)
- [兼容性注意事项](#兼容性注意事项-⭐)
- [常用方法](#常用方法)

---

## Entity定义规范

### 基本结构

```java
package com.dp.dataengine.domain.entity.user;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;

/**
 * 用户实体。
 *
 * @author Fengzhongtian
 * @version 1.0
 * @since 2026-03-10
 */
@Data
@TableName("t_user")
public class UserEntity {

    /**
     * 主键ID。
     */
    @TableId(value = "id", type = IdType.ASSIGN_ID)
    private Long id;

    /**
     * 用户名。
     */
    private String username;

    /**
     * 邮箱。
     */
    private String email;

    /**
     * 部门ID。
     */
    private Long departmentId;

    /**
     * 软删除标记（0=未删除，1=已删除）。
     */
    @TableLogic
    private Integer deleted;

    /**
     * 创建时间。
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;

    /**
     * 更新时间。
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;
}
```

### 注解说明

| 注解 | 说明 | 使用场景 |
|------|------|---------|
| `@TableName` | 指定表名 | 所有Entity类 |
| `@TableId` | 指定主键 | 主键字段 |
| `@TableField` | 指定字段映射 | 字段名与属性名不一致时 |
| `@TableLogic` | 逻辑删除字段 | 软删除标记 |
| `@Version` | 乐观锁版本号 | 需要乐观锁的场景 |

---

## 主键ID策略（⚠️强制）

### 必须使用雪花算法

**❌ 错误（禁止数据库自增）**：
```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
```

**❌ 错误（禁止数据库输入）**：
```java
@TableId(value = "id", type = IdType.INPUT)
private Long id;
```

**❌ 错误（禁止UUID）**：
```java
@TableId(value = "id", type = IdType.ASSIGN_UUID)
private String id;
```

**✅ 正确（雪花算法）**：
```java
@TableId(value = "id", type = IdType.ASSIGN_ID)
private Long id;
```

### 为什么使用雪花算法？

| 优势 | 说明 |
|------|------|
| **分布式友好** | 支持多实例部署，不会ID冲突 |
| **性能更好** | 应用层生成ID，无数据库开销 |
| **ID可提前赋值** | 插入前就知道ID，可用于外键关联 |
| **趋势递增** | ID趋势递增，索引友好 |
| **时序有序** | ID包含时间信息，天然有序 |

### PostgreSQL配合使用

```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,  -- 配合雪花算法使用
    ...
);
```

**注意**：
- Entity使用`@TableId(type = IdType.ASSIGN_ID)`生成ID
- PostgreSQL的BIGSERIAL不会自动使用（避免冲突）
- 如果需要PostgreSQL生成ID，使用`@TableId(type = IdType.AUTO)`

### 雪花算法ID格式

| 部分 | 位数 | 说明 |
|------|------|------|
| 时间戳 | 41位 | 毫秒级时间戳 |
| 机器ID | 10位 | 机器标识 |
| 序列号 | 12位 | 同一毫秒内的序列号 |

> 详见：[主键ID策略完整说明](../03-database/naming-conventions.md#主键id策略)

---

## Mapper接口规范

### 基本结构

```java
package com.dp.dataengine.domain.repository.user;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.dp.dataengine.domain.entity.user.UserEntity;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * 用户Mapper。
 *
 * @author Fengzhongtian
 * @version 1.0
 * @since 2026-03-10
 */
@Mapper
public interface UserMapper extends BaseMapper<UserEntity> {

    /**
     * 根据用户名查询用户。
     *
     * @param username 用户名
     * @return 用户信息
     */
    UserEntity findByUsername(@Param("username") String username);

    /**
     * 根据部门ID查询用户列表。
     *
     * @param departmentId 部门ID
     * @return 用户列表
     */
    List<UserEntity> findByDepartmentId(@Param("departmentId") Long departmentId);
}
```

### BaseMapper提供的方法

```java
// 继承BaseMapper后自动获得以下方法

// 插入
int insert(T entity);

// 删除
int deleteById(Serializable id);
int deleteByMap(Map<String, Object> columnMap);
int delete(Wrapper<T> wrapper);

// 更新
int updateById(T entity);
int update(T entity, Wrapper<T> updateWrapper);

// 查询
T selectById(Serializable id);
List<T> selectBatchIds(Collection<? extends Serializable> idList);
List<T> selectByMap(Map<String, Object> columnMap);
T selectOne(Wrapper<T> queryWrapper);
Long selectCount(Wrapper<T> queryWrapper);
List<T> selectList(Wrapper<T> queryWrapper);
List<Map<String, Object>> selectMaps(Wrapper<T> queryWrapper);
Page<T> selectPage(IPage<T> page, Wrapper<T> queryWrapper);
```

---

## SQL规范（⚠️强制）

### 禁止使用注解方式定义SQL

**❌ 错误（禁止）**：
```java
@Mapper
public interface UserMapper extends BaseMapper<UserEntity> {
    @Select("SELECT * FROM t_user WHERE username = #{username}")
    UserEntity findByUsername(String username);

    @Insert("INSERT INTO t_user (username, email) VALUES (#{username}, #{email})")
    int insertUser(UserEntity user);

    @Update("UPDATE t_user SET username = #{username} WHERE id = #{id}")
    int updateUser(UserEntity user);

    @Delete("DELETE FROM t_user WHERE id = #{id}")
    int deleteUser(Long id);
}
```

**✅ 正确（XML方式）**：

**Mapper接口**：
```java
@Mapper
public interface UserMapper extends BaseMapper<UserEntity> {
    UserEntity findByUsername(@Param("username") String username);
    int insertUser(UserEntity user);
    int updateUser(UserEntity user);
    int deleteUser(Long id);
}
```

**XML文件**：
```xml
<!-- resources/mapper/user/UserMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dp.dataengine.domain.repository.user.UserMapper">

    <!-- 查询 -->
    <select id="findByUsername" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
        SELECT * FROM t_user
        WHERE username = #{username}
          AND deleted = 0
    </select>

    <!-- 插入 -->
    <insert id="insertUser">
        INSERT INTO t_user (id, username, email, created_at, updated_at)
        VALUES (#{id}, #{username}, #{email}, #{createdAt}, #{updatedAt})
    </insert>

    <!-- 更新 -->
    <update id="updateUser">
        UPDATE t_user
        SET username = #{username},
            email = #{email},
            updated_at = CURRENT_TIMESTAMP
        WHERE id = #{id}
          AND deleted = 0
    </update>

    <!-- 删除（软删除） -->
    <update id="deleteUser">
        UPDATE t_user
        SET deleted = 1,
            updated_at = CURRENT_TIMESTAMP
        WHERE id = #{id}
    </update>

</mapper>
```

> 详见：[MyBatis映射规范](../03-database/mybatis-mapping.md)

---

## 兼容性注意事项（⭐）

### 1. 软删除字段映射

**PostgreSQL使用INT作为软删除标记**：

```java
@TableName("t_user")
public class UserEntity {
    // ⚠️ 使用Integer，不是Boolean
    @TableLogic
    private Integer deleted;  // 0=未删除，1=已删除
}
```

**MyBatis Plus自动过滤**：

```java
// 查询时自动添加 deleted = 0 条件
userMapper.selectById(id);

// 等价于
SELECT * FROM t_user WHERE id = #{id} AND deleted = 0
```

### 2. TEXT类型映射

```java
@TableName("t_user")
public class UserEntity {
    // TEXT类型映射为String
    private String description;
}
```

### 3. 主键ID生成

```java
@TableName("t_user")
public class UserEntity {
    // ⚠️ 使用雪花算法，不依赖数据库自增
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
}
```

**Entity插入示例**：

```java
UserEntity user = new UserEntity();
user.setUsername("test");
user.setEmail("test@example.com");
// 不需要设置id，MyBatis Plus自动生成

userMapper.insert(user);
// 插入后，user.getId() 可以获取生成的ID
System.out.println("生成的ID: " + user.getId());
```

### 4. 唯一性约束冲突处理

**在DomainService中处理唯一性冲突**：

```java
@Service
@RequiredArgsConstructor
public class UserDomainService {
    private final UserMapper userMapper;

    public Long createUser(UserEntity entity) {
        try {
            userMapper.insert(entity);
            return entity.getId();
        } catch (DuplicateKeyException e) {
            throw new BusinessException("用户名已存在");
        }
    }
}
```

### 5. 批量操作

**PostgreSQL支持批量插入**：

```java
// 使用MyBatis Plus批量插入
List<UserEntity> users = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    UserEntity user = new UserEntity();
    user.setUsername("user" + i);
    user.setEmail("user" + i + "@example.com");
    users.add(user);
}

// 批量插入，每批100条
userService.saveBatch(users, 100);
```

### 6. 时间类型映射

```java
@TableName("t_user")
public class UserEntity {
    // TIMESTAMP映射为LocalDateTime
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

### 7. 枚举类型映射

**PostgreSQL TEXT + CHECK约束**：

```sql
CREATE TABLE t_user (
    status TEXT NOT NULL,
    CONSTRAINT ck_user_status CHECK (status IN ('active', 'inactive', 'deleted'))
);
```

```java
@TableName("t_user")
public class UserEntity {
    // 枚举映射为String
    private UserStatus status;
}

public enum UserStatus {
    ACTIVE,
    INACTIVE,
    DELETED
}
```

---

## 常用方法

### 条件构造器

**QueryWrapper**：

```java
// 等于
QueryWrapper<UserEntity> wrapper = new QueryWrapper<>();
wrapper.eq("username", "admin");

// 不等于
wrapper.ne("deleted", 1);

// 模糊查询
wrapper.like("username", "admin");

// 大于
wrapper.gt("age", 18);

// 小于等于
wrapper.le("score", 100);

// IN查询
wrapper.in("id", Arrays.asList(1L, 2L, 3L));

// BETWEEN
wrapper.between("age", 18, 60);

// 排序
wrapper.orderByDesc("created_at");

// 条件组装
wrapper.eq("username", "admin")
       .eq("deleted", 0)
       .orderByDesc("created_at");

List<UserEntity> users = userMapper.selectList(wrapper);
```

**LambdaQueryWrapper**（类型安全）：

```java
LambdaQueryWrapper<UserEntity> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(UserEntity::getUsername, "admin")
       .eq(UserEntity::getDeleted, 0)
       .orderByDesc(UserEntity::getCreatedAt);

List<UserEntity> users = userMapper.selectList(wrapper);
```

### 分页查询

```java
// 创建分页对象（第1页，每页10条）
Page<UserEntity> page = new Page<>(1, 10);

// 查询条件
LambdaQueryWrapper<UserEntity> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(UserEntity::getDeleted, 0)
       .orderByDesc(UserEntity::getCreatedAt);

// 分页查询
Page<UserEntity> result = userMapper.selectPage(page, wrapper);

// 获取结果
List<UserEntity> records = result.getRecords();
long total = result.getTotal();
long current = result.getCurrent();
long size = result.getSize();
```

### 批量操作

```java
// 批量插入
List<UserEntity> users = ...;
userService.saveBatch(users);

// 批量插入（指定批次大小）
userService.saveBatch(users, 100);

// 批量更新
List<UserEntity> users = ...;
userService.updateBatchById(users);

// 批量删除
List<Long> ids = Arrays.asList(1L, 2L, 3L);
userService.removeByIds(ids);
```

### 字段选择

```java
// 只选择指定字段
QueryWrapper<UserEntity> wrapper = new QueryWrapper<>();
wrapper.select("id", "username", "email")
       .eq("deleted", 0);

List<UserEntity> users = userMapper.selectList(wrapper);
```

---

## 🔧 配置示例

### application.yml

```yaml
# MyBatis Plus配置
mybatis-plus:
  # Mapper XML位置
  mapper-locations: classpath:mapper/**/*.xml
  # 实体类扫描路径
  type-aliases-package: com.dp.dataengine.domain.entity
  # 全局配置
  global-config:
    db-config:
      # 主键类型（ASSIGN_ID=雪花算法）
      id-type: ASSIGN_ID
      # 表名前缀
      table-prefix: t_
      # 逻辑删除字段
      logic-delete-field: deleted
      # 逻辑删除值
      logic-delete-value: 1
      logic-not-delete-value: 0
  # 配置
  configuration:
    # 驼峰命名转下划线
    map-underscore-to-camel-case: true
    # 日志输出
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# PageHelper配置
pagehelper:
  helper-dialect: postgresql
  reasonable: true
  support-methods-arguments: true
```

---

## 🔗 相关文档

- [Spring Boot规范](./spring-boot.md) - Spring Boot详细规范
- [MyBatis映射规范](../03-database/mybatis-mapping.md) - XML SQL详细规范
- [数据库命名规范](../03-database/naming-conventions.md) - 数据库命名规范

---

**最后更新**：2026-03-10