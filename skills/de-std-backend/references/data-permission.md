# 行级数据权限使用指南

**版本**: v1.0
**更新日期**: 2026-03-14
**适用范围**: 数擎数据中台

---

## 📋 目录

1. [概述](#概述)
2. [架构设计](#架构设计)
3. [权限模型](#权限模型)
4. [实现方式](#实现方式)
5. [使用示例](#使用示例)
6. [最佳实践](#最佳实践)

---

## 概述

### 什么是行级数据权限

行级数据权限（Row-Level Security, RLS）是一种数据库访问控制机制，允许系统根据用户属性（如部门、角色）过滤查询结果，实现"不同人看到不同数据"的效果。

### 应用场景

| 场景 | 说明 | 示例 |
|------|------|------|
| **部门隔离** | 用户只能查看本部门数据 | 销售部只能看销售部的客户 |
| **数据所有者** | 用户只能查看自己创建的数据 | 用户只能看自己创建的任务 |
| **层级权限** | 上级可以查看下级数据 | 总经理可以看所有部门数据 |
| **项目隔离** | 项目成员只能看项目内数据 | 项目A成员看不到项目B数据 |

### 设计目标

| 目标 | 说明 |
|------|------|
| **透明性** | 业务代码无需关心权限控制 |
| **灵活性** | 支持多种权限规则组合 |
| **性能** | 权限检查不影响查询性能 |
| **可维护** | 权限规则集中管理 |

---

## 架构设计

### 系统架构

```
┌─────────────────────────────────────────────────────────┐
│                     业务层                               │
│  UserService.queryUsers() → 无需关心权限过滤            │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  MyBatis Plus                            │
│  UserMapper.selectList() → 自动添加权限条件             │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              数据权限拦截器                               │
│  1. 解析 SQL                                            │
│  2. 识别表名                                            │
│  3. 构建权限条件                                        │
│  4. 注入 WHERE 条件                                     │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              权限服务                                     │
│  获取用户权限范围（部门、数据范围等）                    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              数据库查询                                   │
│  SELECT * FROM t_user WHERE ... AND 权限条件             │
└─────────────────────────────────────────────────────────┘
```

### 组件交互

```
UserRequest
    │
    ▼
Controller → Service → Mapper
                           │
                           ▼
                      MyBatis Plus
                           │
                           ▼
                   DataPermissionInterceptor
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
           SqlParser           PermissionService
                │                     │
                ▼                     ▼
          识别需要控制的表      获取用户权限范围
                │                     │
                └──────────┬──────────┘
                           ▼
                   构建权限条件SQL
                           │
                           ▼
                   注入到原SQL
                           │
                           ▼
                    执行查询并返回
```

---

## 权限模型

### 数据权限类型

```java
package com.dp.dataengine.domain.enums.permission;

import lombok.Getter;

/**
 * 数据权限类型
 */
@Getter
public enum DataScopeType {

    /**
     * 全部数据权限
     */
    ALL(1, "全部数据"),

    /**
     * 本部门及以下数据权限
     */
    DEPT_AND_CHILD(2, "本部门及以下"),

    /**
     * 本部门数据权限
     */
    DEPT_ONLY(3, "本部门"),

    /**
     * 仅本人数据权限
     */
    SELF_ONLY(4, "本人"),

    /**
     * 自定义数据权限
     */
    CUSTOM(5, "自定义");

    private final Integer code;
    private final String description;

    DataScopeType(Integer code, String description) {
        this.code = code;
        this.description = description;
    }

    public static DataScopeType fromCode(Integer code) {
        for (DataScopeType type : values()) {
            if (type.code.equals(code)) {
                return type;
            }
        }
        return SELF_ONLY; // 默认仅本人
    }
}
```

### 用户权限关系

```sql
-- 用户表
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50),
    dept_id BIGINT,           -- 所属部门
    ...
);

-- 角色表
CREATE TABLE t_role (
    id BIGSERIAL PRIMARY KEY,
    role_name VARCHAR(50),
    data_scope INT,           -- 数据权限范围
    ...
);

-- 用户角色关联表
CREATE TABLE t_user_role (
    user_id BIGINT,
    role_id BIGINT,
    PRIMARY KEY (user_id, role_id)
);

-- 部门表
CREATE TABLE t_dept (
    id BIGSERIAL PRIMARY KEY,
    dept_name VARCHAR(50),
    parent_id BIGINT,         -- 父部门ID
    ...
);
```

---

## 实现方式

### 1. 数据权限注解

```java
package com.dp.dataengine.infrastructure.annotation;

import java.lang.annotation.*;

/**
 * 数据权限注解
 * <p>
 * 标记在 Mapper 方法上，指定需要控制的表和字段
 * </p>
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataPermission {

    /**
     * 需要控制权限的表名
     */
    String value() default "";

    /**
     * 权限字段名（默认：created_by）
     */
    String field() default "created_by";

    /**
     * 部门字段名（默认：dept_id）
     */
    String deptField() default "dept_id";
}
```

### 2. 数据权限拦截器

```java
package com.dp.dataengine.infrastructure.interceptor;

import com.baomidou.mybatisplus.extension.plugins.handler.DataPermissionHandler;
import com.dp.dataengine.application.service.permission.DataPermissionService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import net.sf.jsqlparser.expression.*;
import net.sf.jsqlparser.expression.operators.conditional.AndExpression;
import net.sf.jsqlparser.expression.operators.conditional.OrExpression;
import net.sf.jsqlparser.expression.operators.relational.EqualsTo;
import net.sf.jsqlparser.expression.operators.relational.InExpression;
import net.sf.jsqlparser.schema.Column;
import net.sf.jsqlparser.schema.Table;
import net.sf.jsqlparser.statement.select.PlainSelect;
import net.sf.jsqlparser.statement.select.Select;
import net.sf.jsqlparser.statement.select.SelectBody;
import org.springframework.stereotype.Component;

import java.util.Set;

/**
 * 数据权限拦截器
 * <p>
 * 基于 JSQLParser 实现 SQL 解析和权限条件注入
 * </p>
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class RowDataPermissionInterceptor implements DataPermissionHandler {

    private final DataPermissionService dataPermissionService;

    /**
     * 需要进行数据权限控制的表
     */
    private static final Set<String> PERMISSION_TABLES = Set.of(
        "t_user",
        "t_datasource",
        "t_quality_task",
        "t_sync_task"
    );

    @Override
    public Expression getSqlSegment(Select select, int index,
                                    String sqlSegment, Sequence sequence) {
        try {
            SelectBody selectBody = select.getSelectBody();
            if (!(selectBody instanceof PlainSelect)) {
                return null;
            }

            PlainSelect plainSelect = (PlainSelect) selectBody;
            Table table = (Table) plainSelect.getFrom();

            // 检查是否需要数据权限控制
            if (!needDataPermission(table.getName())) {
                return null;
            }

            // 构建权限条件
            Expression permissionExpression = buildPermissionExpression(table);

            // 与现有条件合并
            Expression where = plainSelect.getWhere();
            if (where == null) {
                return permissionExpression;
            }

            return new AndExpression(where, permissionExpression);

        } catch (Exception e) {
            log.error("数据权限处理失败", e);
            return null; // 失败时不影响原SQL执行
        }
    }

    /**
     * 判断是否需要数据权限控制
     */
    private boolean needDataPermission(String tableName) {
        return PERMISSION_TABLES.contains(tableName);
    }

    /**
     * 构建权限表达式
     */
    private Expression buildPermissionExpression(Table table) {
        DataScopeInfo scopeInfo = dataPermissionService.getCurrentUserDataScope();

        return switch (scopeInfo.getDataScopeType()) {
            case ALL -> null; // 全部数据，不添加条件

            case DEPT_AND_CHILD -> buildDeptAndChildExpression(table, scopeInfo);

            case DEPT_ONLY -> buildDeptOnlyExpression(table, scopeInfo);

            case SELF_ONLY -> buildSelfOnlyExpression(table, scopeInfo);

            case CUSTOM -> buildCustomExpression(table, scopeInfo);
        };
    }

    /**
     * 构建本部门及以下权限条件
     */
    private Expression buildDeptAndChildExpression(Table table, DataScopeInfo scopeInfo) {
        // 方案1: dept_id IN (本部门及子部门)
        Column deptColumn = new Column(table, "dept_id");
        InExpression inExpression = new InExpression();
        inExpression.setLeftExpression(deptColumn);

        // 构建子查询或值列表
        Set<Long> deptIds = scopeInfo.getDeptIds();
        if (deptIds.size() == 1) {
            return new EqualsTo(deptColumn, new LongValue(deptIds.iterator().next()));
        }

        // TODO: 构建值列表或子查询
        return inExpression;
    }

    /**
     * 构建本部门权限条件
     */
    private Expression buildDeptOnlyExpression(Table table, DataScopeInfo scopeInfo) {
        Column deptColumn = new Column(table, "dept_id");
        Long deptId = scopeInfo.getDeptId();

        return new EqualsTo(deptColumn, new LongValue(deptId));
    }

    /**
     * 构建本人权限条件
     */
    private Expression buildSelfOnlyExpression(Table table, DataScopeInfo scopeInfo) {
        Column userColumn = new Column(table, "created_by");
        Long userId = scopeInfo.getUserId();

        return new EqualsTo(userColumn, new LongValue(userId));
    }

    /**
     * 构建自定义权限条件
     */
    private Expression buildCustomExpression(Table table, DataScopeInfo scopeInfo) {
        // 根据自定义规则构建条件
        // 例如：指定部门 + 本人
        Expression deptExpr = buildDeptOnlyExpression(table, scopeInfo);
        Expression selfExpr = buildSelfOnlyExpression(table, scopeInfo);

        return new OrExpression(deptExpr, selfExpr);
    }
}
```

### 3. 数据权限服务

```java
package com.dp.dataengine.application.service.permission;

import com.dp.dataengine.domain.entity.iam.user.UserEntity;
import com.dp.dataengine.domain.enums.permission.DataScopeType;
import lombok.Data;
import org.springframework.stereotype.Service;

import java.util.Set;

/**
 * 数据权限服务
 */
@Service
public class DataPermissionService {

    /**
     * 获取当前用户的数据权限范围
     */
    public DataScopeInfo getCurrentUserDataScope() {
        // 1. 获取当前用户
        UserEntity currentUser = getCurrentUser();

        // 2. 获取用户的最大数据权限范围
        DataScopeType maxScopeType = getUserMaxDataScope(currentUser.getId());

        // 3. 根据权限类型构建权限信息
        DataScopeInfo scopeInfo = new DataScopeInfo();
        scopeInfo.setUserId(currentUser.getId());
        scopeInfo.setDeptId(currentUser.getDeptId());
        scopeInfo.setDataScopeType(maxScopeType);

        // 4. 获取部门范围（如果是部门级别权限）
        if (maxScopeType == DataScopeType.DEPT_AND_CHILD ||
            maxScopeType == DataScopeType.DEPT_ONLY) {

            Set<Long> deptIds = getDeptIds(currentUser.getId(), maxScopeType);
            scopeInfo.setDeptIds(deptIds);
        }

        return scopeInfo;
    }

    /**
     * 获取用户的最大数据权限范围
     */
    private DataScopeType getUserMaxDataScope(Long userId) {
        // 查询用户所有角色，取最大权限
        // TODO: 实现角色权限查询
        return DataScopeType.SELF_ONLY; // 默认仅本人
    }

    /**
     * 获取部门ID集合
     */
    private Set<Long> getDeptIds(Long userId, DataScopeType scopeType) {
        // 根据权限类型获取部门范围
        // TODO: 实现部门树查询
        return Set.of(1L);
    }

    private UserEntity getCurrentUser() {
        // TODO: 从 SecurityContext 获取
        return new UserEntity();
    }
}

/**
 * 数据权限范围信息
 */
@Data
class DataScopeInfo {
    /**
     * 用户ID
     */
    private Long userId;

    /**
     * 部门ID
     */
    private Long deptId;

    /**
     * 数据权限类型
     */
    private DataScopeType dataScopeType;

    /**
     * 可访问的部门ID集合
     */
    private Set<Long> deptIds;
}
```

---

## 使用示例

### Mapper 接口

```java
/**
 * 用户Mapper
 * <p>
 * 查询时会自动应用数据权限过滤
 * </p>
 */
public interface UserMapper extends BaseMapper<UserEntity> {

    /**
     * 查询用户列表（自动应用数据权限）
     */
    default List<UserEntity> selectUserList() {
        return selectList(null); // 自动添加权限条件
    }

    /**
     * 分页查询用户（自动应用数据权限）
     */
    default IPage<UserEntity> selectUserPage(Page<?> page) {
        return selectPage(page, null); // 自动添加权限条件
    }
}
```

### Service 层

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserMapper userMapper;

    /**
     * 查询用户列表
     * <p>
     * 无需关心权限过滤，拦截器自动处理
     * </p>
     */
    public List<UserVO> listUsers() {
        // 直接查询，数据权限由拦截器自动处理
        List<UserEntity> entities = userMapper.selectList(null);
        return entities.stream()
            .map(e -> BeanUtil.copyProperties(e, UserVO.class))
            .toList();
    }

    /**
     * 查询指定用户（需要额外校验）
     */
    public UserVO getUser(Long userId) {
        UserEntity entity = userMapper.selectById(userId);

        // 如果用户有"全部数据"权限，可以查看任何人
        // 如果用户只有"本人数据"权限，只能查看自己
        if (!hasPermission(entity)) {
            throw new BusinessException("无权访问该用户数据");
        }

        return BeanUtil.copyProperties(entity, UserVO.class);
    }

    private boolean hasPermission(UserEntity entity) {
        // 特殊校验逻辑（如查看详情时）
        return true;
    }
}
```

---

## 最佳实践

### 1. 权限表设计

```sql
-- 角色表（包含数据权限字段）
CREATE TABLE t_role (
    id BIGSERIAL PRIMARY KEY,
    role_name VARCHAR(50),
    role_key VARCHAR(50),
    data_scope INT NOT NULL DEFAULT 4, -- 数据权限范围
    data_scope_custom VARCHAR(500),    -- 自定义权限范围（JSON）
    ...
);

-- 数据权限示例数据
INSERT INTO t_role (role_name, role_key, data_scope) VALUES
('超级管理员', 'admin', 1),           -- 全部数据
('部门经理', 'dept_manager', 2),     -- 本部门及以下
('部门主管', 'dept_leader', 3),      -- 本部门
('普通用户', 'user', 4);             -- 本人
```

### 2. 缓存策略

```java
/**
 * 数据权限缓存
 */
@Component
public class DataPermissionCache {

    private final Cache<String, DataScopeInfo> cache =
        Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .build();

    public DataScopeInfo getDataScope(Long userId) {
        return cache.get(userId.toString(),
            key -> loadFromDatabase(userId));
    }

    public void evict(Long userId) {
        cache.invalidate(userId.toString());
    }
}
```

### 3. 测试建议

```java
/**
 * 数据权限测试
 */
class DataPermissionTest {

    @Test
    @WithMockUser(username = "user1", userId = 1, deptId = 10)
    void testSelfOnlyPermission() {
        // user1 只能查看自己创建的数据
        List<UserEntity> users = userMapper.selectList(null);
        assertThat(users).allMatch(u -> u.getCreatedBy().equals(1L));
    }

    @Test
    @WithMockUser(username = "manager", userId = 2, deptId = 10, dataScope = "DEPT_ONLY")
    void testDeptOnlyPermission() {
        // manager 可以查看部门10的所有数据
        List<UserEntity> users = userMapper.selectList(null);
        assertThat(users).allMatch(u -> u.getDeptId().equals(10L));
    }
}
```

### 4. 注意事项

| 注意事项 | 说明 |
|---------|------|
| **子查询** | 复杂子查询可能无法正确注入权限条件 |
| **批量操作** | 批量更新/删除需要额外校验权限 |
| **缓存** | 权限变更后需清理缓存 |
| **性能** | 复杂权限条件可能影响查询性能 |
| **日志** | 记录权限拦截日志便于问题排查 |

---

## 附录

### 相关文档

- [MyBatis Plus 配置模板](/CODING_STANDARDS/01-backend/mybatis-plus-config.md)
- [数据库命名规范](/CODING_STANDARDS/03-database/naming-conventions.md)

### 示例代码

完整示例代码请参考：
- `RowDataPermissionInterceptor.java`
- `DataPermissionService.java`
- `DataScopeType.java`

---

**维护者**: 开发团队
**更新周期**: 每季度或重大变更时
