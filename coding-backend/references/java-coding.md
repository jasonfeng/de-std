# Java编码规范

> 本规范定义Java语言的编码规范，确保代码质量和一致性。

---

## 📋 目录

- [Lombok使用规范](#lombok使用规范-⚠️强制)
- [JavaDoc规范](#javadoc规范-⚠️强制)
- [命名规范](#命名规范)
- [代码简洁原则](#代码简洁原则)
- [工具类使用](#工具类使用)
- [对象比较与拷贝](#对象比较与拷贝)
- [异常处理规范](#异常处理规范)
- [代码格式化](#代码格式化)

---

## Lombok使用规范（⚠️强制）

### 规则

**所有实体类必须使用Lombok注解**，禁止手写Getter/Setter。

### @Data

**使用场景**：POJO类、DTO类、VO类、Entity类

```java
@Data
@TableName("t_user")
public class UserEntity {
    private Long id;
    private String username;
    private String email;
}

// 等价于手写：
// - Getter/Setter
// - toString()
// - equals()
// - hashCode()
```

### @Getter/@Setter

**使用场景**：只需要Getter或Setter时

```java
@Getter
@Setter
public class Config {
    private String appName;
    private int maxConnections;

    // 只需要Getter的字段
    private final String version = "1.0.0";
}
```

### @RequiredArgsConstructor

**使用场景**：构造器注入（必用）

```java
@Service
@RequiredArgsConstructor  // 生成全参构造器
public class UserDomainService {
    private final UserMapper userMapper;
    private final DepartmentMapper departmentMapper;

    // Lombok自动生成构造器
    // public UserDomainService(UserMapper userMapper, DepartmentMapper departmentMapper) {
    //     this.userMapper = userMapper;
    //     this.departmentMapper = departmentMapper;
    // }
}
```

### @NoArgsConstructor/@AllArgsConstructor

**使用场景**：需要无参构造器或全参构造器时

```java
@Data
@NoArgsConstructor      // 无参构造器
@AllArgsConstructor    // 全参构造器
public class UserEntity {
    private Long id;
    private String username;
    private String email;
}
```

### @Builder

**使用场景**：复杂对象构建

```java
@Builder
@Data
public class UserDTO {
    private Long id;
    private String username;
    private String email;
    private Integer deleted;
}

// 使用
UserDTO dto = UserDTO.builder()
    .id(1L)
    .username("admin")
    .email("admin@example.com")
    .build();
```

### @Slf4j

**使用场景**：日志记录

```java
@Slf4j  // 等价于：private static final Logger log = LoggerFactory.getLogger(UserService.class);
@Service
@RequiredArgsConstructor
public class UserDomainService {
    private final UserMapper userMapper;

    public Long createUser(UserEntity entity) {
        log.info("Creating user: {}", entity.getUsername());
        userMapper.insert(entity);
        log.debug("User created with ID: {}", entity.getId());
        return entity.getId();
    }
}
```

### @Value

**使用场景**：不可变对象

```java
@Value
public class Constants {
    private String APP_NAME;
    private String APP_VERSION;
    private int MAX_RETRY;
}

// 等价于：
// public final String APP_NAME;
// public final String APP_VERSION;
// public final int MAX_RETRY;
// public Constants(String appName, String appVersion, int maxRetry) {
//     this.APP_NAME = appName;
//     this.APP_VERSION = appVersion;
//     this.MAX_RETRY = maxRetry;
// }
```

### 禁止的写法

**❌ 错误（手写Getter/Setter）**：
```java
public class UserEntity {
    private String username;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
    // ... 其他字段
}
```

**✅ 正确（使用Lombok）**：
```java
@Data
public class UserEntity {
    private String username;
}
```

---

## JavaDoc规范（⚠️强制）

### 规则

**所有类、方法、属性必须增加JavaDoc注解**。

### 类JavaDoc

**标准格式**：
```java
/**
 * 用户领域服务。
 * <p>
 * 提供用户相关的业务逻辑操作，包括用户创建、查询、更新等。
 *
 * @author Fengzhongtian
 * @version 1.0
 * @since 2026-03-10
 */
@Service
@RequiredArgsConstructor
public class UserDomainService {
    // ...
}
```

**必填项**：
- `@author` - 作者
- `@version` - 版本
- `@since` - 起始版本
- 类描述（第一行）

### 方法JavaDoc

**标准格式**：
```java
/**
 * 根据用户ID查询用户信息。
 *
 * @param id 用户ID
 * @return 用户信息，如果不存在返回null
 * @throws BusinessException 如果用户已被删除
 */
public UserDTO getUserById(Long id) {
    // ...
}
```

**必填项**：
- 方法描述（第一行）
- `@param` - 参数说明（每个参数必填）
- `@return` - 返回值说明（有返回值时必填）
- `@throws` - 异常说明（有异常时必填）

### 属性JavaDoc

```java
/**
 * 用户ID（雪花算法生成）。
 */
@TableId(value = "id", type = IdType.ASSIGN_ID)
private Long id;

/**
 * 用户名（3-50字符）。
 */
private String username;

/**
 * 软删除标记（0=未删除，1=已删除）。
 */
@TableLogic
private Integer deleted;
```

### 简化JavaDoc

**简单getter/setter可以省略JavaDoc**（使用@Data时自动生成）：

```java
@Data
public class UserDTO {
    private Long id;
    private String username;
    // 不需要为每个字段写JavaDoc
}
```

---

## 命名规范

### 包命名

**格式**：`com.dp.dataengine.{层级}.{子包}`

```
com.dp.dataengine.interfaces.controller.user
com.dp.dataengine.application.service.user
com.dp.dataengine.domain.entity.user
com.dp.dataengine.domain.repository.user
com.dp.dataengine.domain.service.user
```

### 类命名

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| 实体类 | `XxxEntity` | `UserEntity`、`OrderEntity` |
| DTO类 | `XxxDTO` | `UserDTO`、`DepartmentTreeNodeDTO` |
| VO类 | `XxxVO` | `UserVO`、`RolePathVO` |
| Param类 | `XxxParam` | `UserParam`、`CreateOrderParam` |
| 应用服务 | `XxxAppService` | `UserAppService` |
| 领域服务 | `XxxDomainService` | `UserDomainService` |
| Mapper | `XxxMapper` | `UserMapper`、`OrderMapper` |
| Controller | `XxxController` | `UserController` |
| 配置类 | `XxxConfig` | `MyBatisConfig`、`WebConfig` |
| 异常类 | `XxxException` | `BusinessException` |
| 枚举类 | `XxxEnum` | `UserStatusEnum` |
| 工具类 | `XxxUtil` | `DateUtil`、`StringUtil` |

### 方法命名

| 场景 | 命名格式 | 示例 |
|------|---------|------|
| 查询单个对象 | `get`+名词 | `getUser`、`getOrderById` |
| 查询列表 | `find`+名词 | `findUsers`、`findByStatus` |
| 查询数量 | `count`+名词 | `countUsers`、`countByStatus` |
| 检查是否存在 | `exists`+名词 | `existsUser`、`existsByEmail` |
| 创建 | `create`+名词 | `createUser`、`createOrder` |
| 更新 | `update`+名词 | `updateUser`、`updatePassword` |
| 删除 | `delete`+名词 | `deleteUser`、`deleteById` |
| 保存 | `save`+名词 | `saveUser`、`saveOrder` |
| 转换 | `to`+目标类型 | `toDTO`、`toEntity`、`toVO` |

### 变量命名

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| 普通变量 | 驼峰命名 | `userName`、`orderId` |
| 常量 | 大写下划线 | `MAX_RETRY_COUNT`、`DEFAULT_PAGE_SIZE` |
| 布尔变量 | `is`/`has`前缀 | `isActive`、`hasPermission` |
| 集合变量 | 复数形式 | `users`、`orders`、`permissions` |

---

## 代码简洁原则

### 避免冗余代码

**❌ 冗余**：
```java
if (flag == true) {  // ❌ 冗余
    // ...
}

if (user != null && user.getUsername() != null) {  // ❌ 冗余
    return user.getUsername();
}
```

**✅ 简洁**：
```java
if (flag) {  // ✅ 简洁
    // ...
}

// 使用Optional
if (user != null) {
    return Optional.ofNullable(user.getUsername());
}
```

### 提前返回

**❌ 嵌套if**：
```java
public void process(User user) {
    if (user != null) {
        if (user.isActive()) {
            if (user.hasPermission()) {
                // 处理逻辑
            }
        }
    }
}
```

**✅ 提前返回**：
```java
public void process(User user) {
    if (user == null) {
        return;
    }
    if (!user.isActive()) {
        return;
    }
    if (!user.hasPermission()) {
        return;
    }
    // 处理逻辑
}
```

### 使用Optional处理null

**❌ null检查**：
```java
public String getUsername(Long userId) {
    UserEntity user = userMapper.selectById(userId);
    if (user != null) {
        return user.getUsername();
    }
    return null;
}
```

**✅ Optional**：
```java
public Optional<String> getUsername(Long userId) {
    return Optional.ofNullable(userMapper.selectById(userId))
        .map(UserEntity::getUsername);
}
```

### 使用Stream API

**❌ 循环**：
```java
List<String> names = new ArrayList<>();
for (UserEntity user : users) {
    if (user.getDeleted() == 0) {
        names.add(user.getUsername());
    }
}
```

**✅ Stream**：
```java
List<String> names = users.stream()
    .filter(user -> user.getDeleted() == 0)
    .map(UserEntity::getUsername)
    .collect(Collectors.toList());
```

---

## 工具类使用

### 优先使用成熟工具类

| 优先级 | 工具 | 适用场景 |
|--------|------|---------|
| 1st | Apache Commons | 字符串、集合、IO等基础操作 |
| 2nd | Hutool | Apache中没有的工具类 |
| 3rd | Spring Framework | Spring内置的工具类 |
| last | 自实现 | 仅当前三者都不满足时 |

### 字符串工具

```java
// Apache Commons
import org.apache.commons.lang3.StringUtils;

// 判空
if (StringUtils.isNotBlank(username)) {  // 判空（包括空字符串）
    // ...
}

// 判空（包括null、""、" "）
if (StringUtils.isNotEmpty(username)) {
    // ...
}

// 默认值
String result = StringUtils.defaultIfBlank(username, "anonymous");

// 比较
boolean equals = StringUtils.equals(str1, str2);
```

### 集合工具

```java
// Apache Commons
import org.apache.commons.collections4.CollectionUtils;

// 判空
if (CollectionUtils.isNotEmpty(users)) {
    // ...
}

// 空集合
List<UserEntity> emptyList = Collections.emptyList();
```

### 对象工具

```java
// Spring
import org.springframework.util.ObjectUtils;

// 判空
if (ObjectUtils.isEmpty(user)) {
    // ...
}
```

### 日期工具

```java
// Hutool
import cn.hutool.core.date.DateUtil;

// 当前时间
Date now = DateUtil.date();

// 格式化
String formatted = DateUtil.format(new Date(), "yyyy-MM-dd HH:mm:ss");

// 解析
Date date = DateUtil.parse("2026-03-10 12:00:00");
```

---

## 对象比较与拷贝

### 对象比较

**使用Lombok注解**：
```java
@Data
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class UserEntity {
    @EqualsAndHashCode.Include
    private Long id;

    private String username;
    // equals和hashCode只比较id字段
}
```

### 对象拷贝

**使用Apache Commons**：
```java
// Apache Commons
import org.apache.commons.beanutils.BeanUtils;

// 简单拷贝
UserEntity source = new UserEntity();
UserDTO target = new UserDTO();
BeanUtils.copyProperties(source, target);

// 跳过null字段
BeanUtils.copyProperties(source, target, new String[]{"createdAt", "updatedAt"});
```

**Spring的BeanUtils**：
```java
import org.springframework.beans.BeanUtils;

// 拷贝
BeanUtils.copyProperties(source, target);

// 忽略null值
BeanUtils.copyProperties(source, target, "createdAt", "updatedAt");
```

### Map转换

```java
// Entity转Map
UserEntity user = new UserEntity();
Map<String, Object> userMap = BeanUtils.describe(user);

// Map转Entity
Map<String, Object> userMap = new HashMap<>();
UserEntity user = new UserEntity();
BeanUtils.populate(user, userMap);
```

---

## 异常处理规范

### 自定义异常

```java
/**
 * 业务异常。
 */
@Getter
public class BusinessException extends RuntimeException {
    private final String code;

    public BusinessException(String message) {
        super(message);
        this.code = "BUSINESS_ERROR";
    }

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }
}
```

### 异常使用

```java
public UserEntity getUserById(Long id) {
    UserEntity user = userMapper.selectById(id);
    if (user == null) {
        throw new BusinessException("USER_NOT_FOUND", "用户不存在");
    }
    if (user.getDeleted() == 1) {
        throw new BusinessException("USER_DELETED", "用户已被删除");
    }
    return user;
}
```

### 异常捕获

```java
try {
    // 业务逻辑
} catch (BusinessException e) {
    // 业务异常，记录日志后抛出
    log.error("业务异常: {}", e.getMessage(), e);
    throw e;
} catch (Exception e) {
    // 系统异常，记录日志后抛出
    log.error("系统异常", e);
    throw new BusinessException("SYSTEM_ERROR", "系统内部错误");
}
```

### 禁止吞噬异常

**❌ 错误（吞噬异常）**：
```java
try {
    userMapper.insert(user);
} catch (Exception e) {
    log.error("保存失败", e);
    // 吞掉异常，继续执行
}
```

**✅ 正确（抛出异常）**：
```java
try {
    userMapper.insert(user);
} catch (Exception e) {
    log.error("保存失败", e);
    throw new BusinessException("保存失败", e);
}
```

---

## 代码格式化

### 代码风格

使用统一的代码风格：
- **缩进**：4空格
- **行宽**：120字符
- **大括号**：K&R风格（不换行）

### 示例

```java
@Service
@RequiredArgsConstructor
public class UserDomainService {

    private final UserMapper userMapper;

    public Long createUser(UserEntity entity) {
        validateUsername(entity.getUsername());
        userMapper.insert(entity);
        return entity.getId();
    }

    private void validateUsername(String username) {
        if (StringUtils.isBlank(username)) {
            throw new BusinessException("用户名不能为空");
        }
        if (username.length() < 3 || username.length() > 50) {
            throw new BusinessException("用户名长度必须在3-50之间");
        }
    }
}
```

### 导入顺序

```java
// 1. Java标准库
import java.time.LocalDateTime;
import java.util.List;

// 2. 第三方库
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;
import org.springframework.stereotype.Service;

// 3. 项目内部
import com.dp.dataengine.domain.entity.user.UserEntity;
import com.dp.dataengine.domain.repository.user.UserMapper;
```

---

## 🔗 相关文档

- [Spring Boot规范](./spring-boot.md) - Spring Boot详细规范
- [MyBatis Plus规范](./mybatis-plus.md) - MyBatis Plus详细规范
- [DDD架构规范](./ddd-architecture.md) - DDD架构详细规范

---

**最后更新**：2026-03-10