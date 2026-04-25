# MyBatis Plus 配置模板

**版本**: v1.0
**更新日期**: 2026-03-14
**MyBatis Plus 版本**: 3.5+

---

## 📋 目录

1. [核心配置](#核心配置)
2. [分页配置](#分页配置)
3. [字段自动填充](#字段自动填充)
4. [逻辑删除](#逻辑删除)
5. [乐观锁](#乐观锁)
6. [行级数据权限](#行级数据权限)
7. [SQL 审计](#sql-审计)
8. [性能分析](#性能分析)

---

## 核心配置

### 基础配置类

```java
package com.dp.dataengine.infrastructure.config.mybatis;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.*;
import org.apache.ibatis.reflection.MetaObject;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.LocalDateTime;

/**
 * MyBatis Plus 完整配置
 * <p>
 * 包含分页、字段填充、逻辑删除、乐观锁、数据权限、性能分析等功能
 * </p>
 */
@Configuration
@MapperScan("com.dp.dataengine.domain.repository")
public class MybatisPlusConfig {

    // ==================== 拦截器配置 ====================

    /**
     * MyBatis Plus 拦截器链
     * <p>
     * 注意：拦截器顺序很重要，从内到外执行
     * </p>
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(
            PaginationInnerInterceptor paginationInterceptor,
            OptimisticLockerInnerInterceptor optimisticLockerInterceptor,
            BlockAttackInnerInterceptor blockAttackInterceptor,
            DataPermissionInterceptor dataPermissionInterceptor) {

        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();

        // 1. 分页插件（最内层）
        interceptor.addInnerInterceptor(paginationInterceptor);

        // 2. 乐观锁插件
        interceptor.addInnerInterceptor(optimisticLockerInterceptor);

        // 3. 防止全表更新和删除插件
        interceptor.addInnerInterceptor(blockAttackInterceptor);

        // 4. 数据权限插件（最外层，最后执行）
        interceptor.addInnerInterceptor(dataPermissionInterceptor);

        return interceptor;
    }

    // ==================== 分页配置 ====================

    /**
     * 分页插件配置
     */
    @Bean
    public PaginationInnerInterceptor paginationInnerInterceptor() {
        PaginationInnerInterceptor interceptor = new PaginationInnerInterceptor(DbType.POSTGRE_SQL);

        // 单页分页条数限制（默认无限制）
        interceptor.setMaxLimit(1000L);

        // 溢出总页数后是否进行处理（默认不处理）
        // true: 返回首页; false: 继续请求
        interceptor.setOverflow(false);

        // 单页分页条数小于0时是否执行（默认不执行）
        interceptor.setLimit(Long.MAX_VALUE);

        return interceptor;
    }

    // ==================== 乐观锁配置 ====================

    /**
     * 乐观锁插件
     * <p>
     * 需要实体类包含 @Version 注解的字段
     * </p>
     */
    @Bean
    public OptimisticLockerInnerInterceptor optimisticLockerInnerInterceptor() {
        return new OptimisticLockerInnerInterceptor();
    }

    // ==================== 防全表操作配置 ====================

    /**
     * 防止全表更新和删除插件
     */
    @Bean
    public BlockAttackInnerInterceptor blockAttackInnerInterceptor() {
        return new BlockAttackInnerInterceptor();
    }

    // ==================== 数据权限配置 ====================

    /**
     * 数据权限插件
     * <p>
     * 结合自定义的数据权限处理器实现行级权限控制
     * </p>
     */
    @Bean
    public DataPermissionInterceptor dataPermissionInterceptor() {
        return new DataPermissionInterceptor(new DataEngineDataPermissionHandler());
    }

    // ==================== 字段填充配置 ====================

    /**
     * 字段自动填充处理器
     */
    @Bean
    public MetaObjectHandler metaObjectHandler() {
        return new DataEngineMetaObjectHandler();
    }
}
```

---

## 分页配置

### 使用示例

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserMapper userMapper;

    /**
     * 分页查询用户
     */
    public Page<UserVO> pageUsers(UserQuery query) {
        // 创建分页对象
        Page<UserEntity> page = new Page<>(query.getPageNum(), query.getPageSize());

        // 执行分页查询
        Page<UserEntity> result = userMapper.selectPage(page,
            Wrappers.<UserEntity>lambdaQuery()
                .like(StringUtils.isNotBlank(query.getUsername()),
                    UserEntity::getUsername, query.getUsername())
                .eq(query.getStatus() != null,
                    UserEntity::getStatus, query.getStatus())
                .orderByDesc(UserEntity::getCreatedAt)
        );

        // 转换为 VO
        return result.convert(user -> BeanUtil.copyProperties(user, UserVO.class));
    }
}
```

### Mapper 方法

```java
/**
 * 自定义分页查询（XML 实现）
 */
IPage<UserEntity> selectUserPage(Page<?> page,
                                 @Param("query") UserQuery query);
```

### XML 配置

```xml
<select id="selectUserPage" resultType="UserEntity">
    SELECT * FROM t_user
    <where>
        <if test="query.username != null and query.username != ''">
            AND username LIKE CONCAT('%', #{query.username}, '%')
        </if>
        <if test="query.status != null">
            AND status = #{query.status}
        </if>
    </where>
    ORDER BY created_at DESC
</select>
```

---

## 字段自动填充

### 填充处理器

```java
package com.dp.dataengine.infrastructure.config.mybatis;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * 字段自动填充处理器
 * <p>
 * 自动填充：createdAt、updatedAt、createdBy、updatedBy
 * </p>
 */
@Slf4j
@Component
public class DataEngineMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        log.debug("开始插入填充...");

        // 自动填充创建时间
        this.strictInsertFill(metaObject, "createdAt", LocalDateTime.class, LocalDateTime.now());

        // 自动填充更新时间
        this.strictInsertFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());

        // 自动填充创建人
        Long currentUserId = getCurrentUserId();
        if (currentUserId != null) {
            this.strictInsertFill(metaObject, "createdBy", Long.class, currentUserId);
            this.strictInsertFill(metaObject, "updatedBy", Long.class, currentUserId);
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.debug("开始更新填充...");

        // 自动填充更新时间
        this.strictUpdateFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());

        // 自动填充更新人
        Long currentUserId = getCurrentUserId();
        if (currentUserId != null) {
            this.strictUpdateFill(metaObject, "updatedBy", Long.class, currentUserId);
        }
    }

    /**
     * 获取当前用户ID
     * <p>
     * 从 SecurityContext 或 ThreadLocal 获取
     * </p>
     */
    private Long getCurrentUserId() {
        // TODO: 实现从上下文获取当前用户ID
        // return SecurityContextHolder.getUserId();
        return 1L; // 测试用
    }
}
```

### 实体类配置

```java
@Data
@TableName("t_user")
public class UserEntity {

    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    private String username;

    /**
     * 创建时间 - 插入时自动填充
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;

    /**
     * 更新时间 - 插入和更新时自动填充
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;

    /**
     * 创建人ID - 插入时自动填充
     */
    @TableField(fill = FieldFill.INSERT)
    private Long createdBy;

    /**
     * 更新人ID - 插入和更新时自动填充
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Long updatedBy;
}
```

---

## 逻辑删除

### 全局配置

```yaml
mybatis-plus:
  global-config:
    db-config:
      # 逻辑删除字段名
      logic-delete-field: deleted
      # 逻辑删除值（默认 1）
      logic-delete-value: 1
      # 逻辑未删除值（默认 0）
      logic-not-delete-value: 0
```

### 实体类配置

```java
@Data
@TableName("t_user")
public class UserEntity {

    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    /**
     * 逻辑删除标记
     * 0: 未删除, 1: 已删除
     */
    @TableLogic
    private Integer deleted;
}
```

### 使用示例

```java
// 删除操作会自动转换为 UPDATE
userMapper.deleteById(1L);
// 实际执行: UPDATE t_user SET deleted = 1 WHERE id = 1 AND deleted = 0

// 查询操作会自动过滤已删除数据
userMapper.selectList(null);
// 实际执行: SELECT * FROM t_user WHERE deleted = 0
```

---

## 乐观锁

### 实体类配置

```java
@Data
@TableName("t_user")
public class UserEntity {

    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    private String username;

    /**
     * 乐观锁版本号
     */
    @Version
    private Integer version;
}
```

### 数据库表设计

```sql
CREATE TABLE t_user (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50),
    version INTEGER DEFAULT 0  -- 乐观锁版本号
);
```

### 使用示例

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserMapper userMapper;

    /**
     * 更新用户（带乐观锁）
     */
    public void updateUser(UserEntity user) {
        // 查询当前版本
        UserEntity existing = userMapper.selectById(user.getId());

        // 修改属性
        existing.setUsername(user.getUsername());

        // 执行更新（自动带版本号检查）
        int rows = userMapper.updateById(existing);
        if (rows == 0) {
            throw new OptimisticLockException("数据已被其他用户修改，请刷新后重试");
        }
    }
}
```

---

## 行级数据权限

### 数据权限处理器

```java
package com.dp.dataengine.infrastructure.config.mybatis;

import com.baomidou.mybatisplus.extension.plugins.handler.DataPermissionHandler;
import net.sf.jsqlparser.expression.Expression;
import net.sf.jsqlparser.expression.LongValue;
import net.sf.jsqlparser.expression.operators.conditional.AndExpression;
import net.sf.jsqlparser.expression.operators.relational.EqualsTo;
import net.sf.jsqlparser.expression.operators.relational.InExpression;
import net.sf.jsqlparser.schema.Column;
import net.sf.jsqlparser.schema.Table;
import net.sf.jsqlparser.statement.select.PlainSelect;
import net.sf.jsqlparser.statement.select.Select;
import net.sf.jsqlparser.statement.select.SelectBody;

/**
 * 数据权限处理器
 * <p>
 * 基于 SQL 解析实现行级数据权限控制
 * </p>
 */
@Slf4j
public class DataEngineDataPermissionHandler implements DataPermissionHandler {

    @Override
    public Expression getSqlSegment(Select select, int index, String sqlSegment, Sequence sequence) {
        SelectBody selectBody = select.getSelectBody();
        if (!(selectBody instanceof PlainSelect)) {
            return null;
        }

        PlainSelect plainSelect = (PlainSelect) selectBody;
        Table table = (Table) plainSelect.getFrom();

        // 只处理特定表的权限控制
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
    }

    /**
     * 判断是否需要数据权限控制
     */
    private boolean needDataPermission(String tableName) {
        // 定义需要数据权限控制的表
        return Set.of(
            "t_user",
            "t_datasource",
            "t_quality_task"
        ).contains(tableName);
    }

    /**
     * 构建权限表达式
     */
    private Expression buildPermissionExpression(Table table) {
        Long currentUserId = getCurrentUserId();
        List<Long> dataScope = getDataScope(currentUserId);

        // 方案1: 本人数据
        // created_by = #{currentUserId}

        // 方案2: 部门数据
        // dept_id IN (deptIds)

        Column column = new Column(table, "created_by");
        return new EqualsTo(column, new LongValue(currentUserId));
    }

    private Long getCurrentUserId() {
        // TODO: 从 SecurityContext 获取
        return 1L;
    }

    private List<Long> getDataScope(Long userId) {
        // TODO: 查询用户的数据权限范围
        return List.of(1L, 2L, 3L);
    }
}
```

---

## SQL 审计

### SQL 审计拦截器

```java
package com.dp.dataengine.infrastructure.interceptor;

import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;
import org.springframework.stereotype.Component;

import java.util.Properties;

/**
 * SQL 审计拦截器
 * <p>
 * 记录所有执行的 SQL 语句及执行时间
 * </p>
 */
@Slf4j
@Intercepts({
    @Signature(type = Executor.class,
            method = "update",
            args = {MappedStatement.class, Object.class}),
    @Signature(type = Executor.class,
            method = "query",
            args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})
})
@Component
public class SqlAuditInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
        Object parameter = invocation.getArgs()[1];

        BoundSql boundSql = mappedStatement.getBoundSql(parameter);
        String sql = boundSql.getSql();

        long startTime = System.currentTimeMillis();
        Object result;
        try {
            result = invocation.proceed();
            return result;
        } finally {
            long duration = System.currentTimeMillis() - startTime;

            // 记录慢 SQL（超过 1 秒）
            if (duration > 1000) {
                log.warn("慢 SQL 检测: {}ms, SQL: {}", duration, formatSql(sql));
            }

            // 记录审计日志
            auditLog(mappedStatement.getId(), sql, duration);
        }
    }

    private String formatSql(String sql) {
        // 格式化 SQL 便于查看
        return sql.replaceAll("\\s+", " ").trim();
    }

    private void auditLog(String statementId, String sql, long duration) {
        // TODO: 将审计日志写入数据库或文件
        log.info("SQL审计: statement={}, duration={}ms, sql={}",
            statementId, duration, formatSql(sql));
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        // 配置属性
    }
}
```

---

## 性能分析

### 性能分析拦截器

```java
package com.dp.dataengine.infrastructure.config.mybatis;

import com.baomidou.mybatisplus.extension.plugins.inner.InnerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.BlockAttackInnerInterceptor;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

/**
 * 性能分析配置
 * <p>
 * 仅在开发环境启用
 * </p>
 */
@Slf4j
@Configuration
@Profile("dev")
public class PerformanceAnalysisConfig {

    /**
     * 性能分析拦截器
     */
    @Bean
    public PerformanceInterceptor performanceInterceptor() {
        PerformanceInterceptor interceptor = new PerformanceInterceptor();

        // SQL 执行最大时长（超过则停止运行）
        interceptor.setMaxTime(1000);

        // 是否格式化 SQL
        interceptor.setFormat(true);

        return interceptor;
    }

    /**
     * 自定义性能分析拦截器
     */
    @Slf4j
    public static class PerformanceInterceptor implements InnerInterceptor {

        private long maxTime = 1000;
        private boolean format = true;

        public void setMaxTime(long maxTime) {
            this.maxTime = maxTime;
        }

        public void setFormat(boolean format) {
            this.format = format;
        }

        @Override
        public void beforeQuery(Executor executor, MappedStatement ms,
                               Object parameter, RowBounds rowBounds,
                               ResultHandler resultHandler, BoundSql boundSql) {
            // 记录查询开始时间
            long startTime = System.currentTimeMillis();
            SqlPerformanceContext.setStartTime(startTime);
        }

        @Override
        public void afterQuery(Executor executor, MappedStatement ms,
                              Object parameter, RowBounds rowBounds,
                              ResultHandler resultHandler, BoundSql boundSql) {
            Long startTime = SqlPerformanceContext.getStartTime();
            if (startTime != null) {
                long duration = System.currentTimeMillis() - startTime;
                String sql = boundSql.getSql();

                if (duration > maxTime) {
                    log.error("SQL性能警告: 执行时间 {}ms, 超过阈值 {}ms, SQL: {}",
                        duration, maxTime, format ? formatSql(sql) : sql);
                }

                SqlPerformanceContext.remove();
            }
        }

        private String formatSql(String sql) {
            return sql.replaceAll("\\s+", " ").trim();
        }
    }

    /**
     * SQL 性能上下文
     */
    public static class SqlPerformanceContext {
        private static final ThreadLocal<Long> START_TIME = new ThreadLocal<>();

        public static void setStartTime(long startTime) {
            START_TIME.set(startTime);
        }

        public static Long getStartTime() {
            return START_TIME.get();
        }

        public static void remove() {
            START_TIME.remove();
        }
    }
}
```

---

## 附录

### 常用注解

| 注解 | 说明 | 示例 |
|------|------|------|
| `@TableName` | 表名映射 | `@TableName("t_user")` |
| `@TableId` | 主键映射 | `@TableId(type = IdType.ASSIGN_ID)` |
| `@TableField` | 字段映射 | `@TableField("user_name")` |
| `@TableLogic` | 逻辑删除 | `@TableLogic` |
| `@Version` | 乐观锁 | `@Version` |
| `@TableField(fill=...)` | 自动填充 | `@TableField(fill = FieldFill.INSERT)` |

### 配置参数

```yaml
mybatis-plus:
  # Mapper XML 扫描路径
  mapper-locations: classpath*:/mapper/**/*.xml
  # 实体类扫描路径
  type-aliases-package: com.dp.dataengine.domain.entity
  # 配置文件路径
  config-location: classpath:mybatis-config.xml

  global-config:
    db-config:
      # 主键类型（AUTO 数据库自增, ASSIGN_ID 雪花算法）
      id-type: ASSIGN_ID
      # 表名下划线命名
      table-underline: true
      # 逻辑删除配置
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0

  configuration:
    # 驼峰转下划线
    map-underscore-to-camel-case: true
    # 缓存开关
    cache-enabled: true
    # 日志输出
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl
```

---

**维护者**: 开发团队
**相关文档**:
- [数据库命名规范](/CODING_STANDARDS/03-database/naming-conventions.md)
- [行级数据权限指南](/CODING_STANDARDS/01-backend/data-permission.md)
