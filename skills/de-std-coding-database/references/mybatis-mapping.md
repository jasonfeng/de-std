# MyBatis映射规范

> 本规范定义MyBatis Plus的使用规范，包括Mapper接口和XML SQL文件的编写规范。

---

## 📂 文件位置

### Mapper接口位置

```
backend/dataengine-api/src/main/java/com/dp/dataengine/domain/repository/{module}/
├── UserMapper.java
├── RoleMapper.java
└── DepartmentMapper.java
```

### XML映射文件位置

```
backend/dataengine-api/src/main/resources/mapper/{module}/
├── UserMapper.xml
├── RoleMapper.xml
└── DepartmentMapper.xml
```

---

## ⚠️ 核心规则

### 1. SQL必须在XML文件中

禁止使用 `@Select/@Insert/@Update/@Delete` 注解，所有 SQL 必须写在 XML 中。

### 2. 复杂SQL逻辑必须在XML中使用条件方式实现

CASE 表达式、多字段组合排序等复杂 SQL 逻辑，**禁止在 Java 代码中拼接 SQL 片段**，必须使用 MyBatis XML 的 `<choose>/<when>` 条件方式实现。DomainService 仅负责排序字段白名单校验和命名转换。

```xml
<!-- ✅ 正确：XML中使用choose条件处理复杂排序 -->
<if test="sortBy != null and sortOrder != null">
    <choose>
        <when test="sortBy != null and sortBy == 'status'.toString()">
            ORDER BY
                CASE lm.status WHEN 'draft' THEN 1 WHEN 'published' THEN 2 ELSE 0 END ${sortOrder},
                lm.updated_at DESC
        </when>
        <otherwise>
            ORDER BY ${sortBy} ${sortOrder}
        </otherwise>
    </choose>
</if>
```

```java
// ❌ 错误：在DomainService中拼接CASE表达式
if ("status".equals(sortBy)) {
    dbSortBy = "CASE WHEN status = 'draft' THEN 1 WHEN 'published' THEN 2 ELSE 0 END, updated_at DESC";
}
```

> OGNL 字符串比较需使用 `sortBy == 'value'.toString()` 确保类型匹配。

**❌ 错误（禁止使用注解）**：
```java
@Mapper
public interface UserMapper extends BaseMapper<UserEntity> {
    @Select("SELECT * FROM t_user WHERE id = #{id}")
    UserEntity findById(Long id);

    @Insert("INSERT INTO t_user (username, email) VALUES (#{username}, #{email})")
    int insertUser(UserEntity user);

    @Update("UPDATE t_user SET username = #{username} WHERE id = #{id}")
    int updateUser(UserEntity user);

    @Delete("DELETE FROM t_user WHERE id = #{id}")
    int deleteUser(Long id);
}
```

**✅ 正确（XML方式）**：
```java
// Mapper接口
@Mapper
public interface UserMapper extends BaseMapper<UserEntity> {
    UserEntity findById(Long id);
    int insertUser(UserEntity user);
    int updateUser(UserEntity user);
    int deleteUser(Long id);
}
```

```xml
<!-- XML文件位置：resources/mapper/user/UserMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dp.dataengine.domain.repository.user.UserMapper">

    <!-- 查询 -->
    <select id="findById" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
        SELECT * FROM t_user
        WHERE id = #{id}
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

---

## 📝 Mapper接口规范

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
     * 根据ID查询用户。
     *
     * @param id 用户ID
     * @return 用户信息
     */
    UserEntity findById(@Param("id") Long id);

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

    /**
     * 统计用户数量。
     *
     * @return 用户数量
     */
    Long countUsers();
}
```

### 参数传递

| 场景 | 使用方式 | 示例 |
|------|---------|------|
| 单个参数 | 直接使用参数 | `findById(Long id)` |
| 多个参数 | 使用`@Param` | `findByStatusAndDeleted(@Param("status") int status, @Param("deleted") int deleted)` |
| 对象参数 | 直接使用对象 | `insertUser(UserEntity user)` |
| 集合参数 | 使用`@Param` + `List/Set` | `findByIds(@Param("ids") List<Long> ids)` |

---

## 📄 XML映射文件规范

### 文件头

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dp.dataengine.domain.repository.user.UserMapper">
    <!-- SQL语句 -->
</mapper>
```

### resultType vs resultMap

**resultType（简单场景）**：
```xml
<!-- 简单查询，字段名与Entity属性名一致 -->
<select id="findById" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
    SELECT * FROM t_user WHERE id = #{id} AND deleted = 0
</select>
```

**resultMap（复杂场景）**：
```xml
<!-- 复杂查询，需要字段映射 -->
<resultMap id="userDetailMap" type="com.dp.dataengine.domain.entity.user.UserEntity">
    <id property="id" column="id"/>
    <result property="username" column="user_name"/>
    <result property="email" column="email"/>
    <result property="departmentId" column="department_id"/>
    <!-- 关联查询 -->
    <association property="department" javaType="com.dp.dataengine.domain.entity.department.DepartmentEntity">
        <id property="id" column="dept_id"/>
        <result property="name" column="dept_name"/>
    </association>
</resultMap>

<select id="findUserDetail" resultMap="userDetailMap">
    SELECT u.id, u.user_name, u.email, u.department_id,
           d.id AS dept_id, d.name AS dept_name
    FROM t_user u
    LEFT JOIN t_department d ON u.department_id = d.id
    WHERE u.id = #{id} AND u.deleted = 0
</select>
```

---

## 🎯 SQL编写规范

### SELECT语句

```xml
<!-- 基本查询 -->
<select id="findById" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
    SELECT
        id,
        username,
        email,
        department_id,
        deleted,
        created_at,
        updated_at
    FROM t_user
    WHERE id = #{id}
      AND deleted = 0
</select>

<!-- 动态查询 -->
<select id="findByCondition" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
    SELECT * FROM t_user
    WHERE deleted = 0
    <if test="username != null and username != ''">
        AND username LIKE CONCAT('%', #{username}, '%')
    </if>
    <if test="departmentId != null">
        AND department_id = #{departmentId}
    </if>
    ORDER BY created_at DESC
</select>

<!-- IN查询 -->
<select id="findByIds" resultType="com.dp.dataengine.domain.entity.user.UserEntity">
    SELECT * FROM t_user
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
      AND deleted = 0
</select>
```

### INSERT语句

```xml
<!-- 基本插入 -->
<insert id="insertUser">
    INSERT INTO t_user (id, username, email, department_id, deleted, created_at, updated_at)
    VALUES (#{id}, #{username}, #{email}, #{departmentId}, #{deleted}, #{createdAt}, #{updatedAt})
</insert>

<!-- 动态插入 -->
<insert id="insertUserDynamic">
    INSERT INTO t_user
    <trim prefix="(" suffix=")" suffixOverrides=",">
        <if test="id != null">id,</if>
        <if test="username != null">username,</if>
        <if test="email != null">email,</if>
        <if test="departmentId != null">department_id,</if>
        deleted,
        created_at,
        updated_at
    </trim>
    <trim prefix="VALUES (" suffix=")" suffixOverrides=",">
        <if test="id != null">#{id},</if>
        <if test="username != null">#{username},</if>
        <if test="email != null">#{email},</if>
        <if test="departmentId != null">#{departmentId},</if>
        #{deleted},
        #{createdAt},
        #{updatedAt}
    </trim>
</insert>
```

### UPDATE语句

```xml
<!-- 基本更新 -->
<update id="updateUser">
    UPDATE t_user
    SET username = #{username},
        email = #{email},
        department_id = #{departmentId},
        updated_at = CURRENT_TIMESTAMP
    WHERE id = #{id}
      AND deleted = 0
</update>

<!-- 动态更新 -->
<update id="updateUserDynamic">
    UPDATE t_user
    <set>
        <if test="username != null">username = #{username},</if>
        <if test="email != null">email = #{email},</if>
        <if test="departmentId != null">department_id = #{departmentId},</if>
        updated_at = CURRENT_TIMESTAMP,
    </set>
    WHERE id = #{id}
      AND deleted = 0
</update>
```

### DELETE语句

**注意**：本项目使用软删除，DELETE语句应更新deleted字段。

```xml
<!-- 软删除 -->
<update id="deleteUser">
    UPDATE t_user
    SET deleted = 1,
        updated_at = CURRENT_TIMESTAMP
    WHERE id = #{id}
</update>

<!-- 批量软删除 -->
<update id="deleteUsers">
    UPDATE t_user
    SET deleted = 1,
        updated_at = CURRENT_TIMESTAMP
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</update>
```

---

## ⭐ MyBatis Plus兼容性注意事项

### 主键ID策略

**必须使用雪花算法**，不依赖数据库自增。

```java
// Entity
@TableName("t_user")
public class UserEntity {
    @TableId(type = IdType.ASSIGN_ID)  // 雪花算法
    private Long id;
    // ...
}
```

**PostgreSQL配合**：
```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,  -- 配合雪花算法使用
    ...
);
```

> 详见：[主键ID策略](./naming-conventions.md#主键id策略)

### 软删除字段映射

**PostgreSQL使用INT作为软删除标记**：

```java
@TableName("t_user")
public class UserEntity {
    // ⚠️ 使用Integer，不是Boolean
    private Integer deleted;  // 0=未删除，1=已删除
}
```

**查询时自动过滤**：

```xml
<!-- MyBatis Plus自动添加deleted条件 -->
<select id="findById" resultType="UserEntity">
    SELECT * FROM t_user
    WHERE id = #{id}
      AND deleted = 0  <!-- 手动添加过滤 -->
</select>
```

### TEXT类型映射

```java
@TableName("t_user")
public class UserEntity {
    // TEXT类型映射为String
    private String description;
}
```

### 唯一性约束冲突处理

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

### 批量操作

**PostgreSQL支持批量插入**：

```java
// 使用MyBatis Plus批量插入
List<UserEntity> users = ...;
userService.saveBatch(users, 100);  // 每批100条
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
```

---

## 🔗 相关文档

- [PostgreSQL语法规范](./postgresql.md) - PostgreSQL详细语法
- [数据库命名规范](./naming-conventions.md) - 完整命名规范
- [MyBatis Plus规范](../01-backend/mybatis-plus.md) - MyBatis Plus详细规范

---

**最后更新**：2026-03-10