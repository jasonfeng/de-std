---
name: coding-backend
description: 后端 Java/Spring Boot/MyBatis Plus 编码规范。写 Java 代码前必须调用。
---

# 后端编码规范

## 触发时机

- "写一个 Service/Mapper/Controller"
- "添加后端 API"
- "创建 Entity/DTO/VO"
- "实现业务逻辑"

---

## DDD分层架构

### 包结构

```
com.dp.dataengine/
├── interfaces/
│   ├── controller/    # REST API
│   ├── param/         # 请求参数
│   └── vo/            # 响应视图
├── application/
│   ├── service/       # AppService（事务编排）
│   ├── dto/           # DTO
│   └── assembly/      # 跨层转换
├── domain/
│   ├── entity/        # Entity（必须带Entity后缀）
│   ├── service/       # DomainService（业务逻辑）
│   └── repository/    # Repository
├── infrastructure/
│   └── config/
└── common/
```

### 调用关系

```
Controller → AppService → DomainService → Repository
```

**禁止**：DomainService 直接调用其他 DomainService

### 双Assembly设计

- **DomainAssembly**：Entity ↔ DomainDTO（领域层内部）
- **ApplicationAssembly**：Param/VO/DTO ↔ Entity/DomainDTO（跨层转换）

---

## Spring Boot 规范

### 依赖注入（强制）

```java
// ❌ 禁止
@Autowired
private UserMapper userMapper;

// ✅ 正确
@Service
@RequiredArgsConstructor
public class UserAppService {
    private final UserMapper userMapper;
    private final UserDomainService userDomainService;
}
```

### 事务管理

```java
// AppService：事务边界
@Service
@RequiredArgsConstructor
public class OrderAppService {
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(OrderCreateParam param) { }
}

// DomainService：无事务注解
@Component
public class OrderDomainService {
    // ❌ 禁止 @Transactional
    public void validateOrder(OrderEntity order) { }
}
```

### 异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusiness(BusinessException e) {
        return Result.fail(e.getCode(), e.getMessage());
    }
}
```

### Jackson序列化

- VO 中 ID 使用 String 类型（避免前端精度丢失）
- Assembly 手动转换 Long → String
- 禁止全局 Long→String 序列化
- 统计字段（count/amount）保持 Long 类型

---

## MyBatis Plus 规范

### SQL必须在XML文件中

```java
// ❌ 禁止
@Select("SELECT * FROM t_user")
List<UserEntity> findAll();

// ✅ 正确：XML方式
public interface UserMapper extends BaseMapper<UserEntity> {
    List<UserEntity> findAll();
}
```

### 主键策略

```java
@TableId(value = "id", type = IdType.ASSIGN_ID)  // 雪花算法
private Long id;
```

### 逻辑删除

```java
@TableLogic
private Integer deleted;  // 0=未删除, 1=已删除
```

### 字段自动填充

createdAt、updatedAt、createdBy、updatedBy 通过 MetaObjectHandler 自动填充。

---

## DTO/VO/Param 规范

| 类型 | 位置 | 用途 | 示例 |
|------|------|------|------|
| Param | interfaces/param | 接收请求 | `UserCreateParam` |
| VO | interfaces/vo | 返回响应 | `UserVO` |
| DTO | application/dto | 跨层传递 | `UserDTO` |
| Entity | domain/entity | 数据库映射 | `UserEntity` |

### 转换规则

所有跨层转换必须通过 Assembly 层：

```java
// ✅ 正确
UserEntity entity = UserAssembly.toEntity(param);
UserVO vo = UserAssembly.toVO(entity);

// ❌ 禁止在 Service 中直接 BeanUtils.copyProperties
```

---

## Java编码规范

### Lombok使用（强制）

```java
@Data                    // 替代手写getter/setter
@RequiredArgsConstructor // 构造器注入
@Slf4j                   // 日志
@Builder                  // 构建器模式
```

### 命名规范

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| Entity | XxxEntity | `UserEntity` |
| DTO | XxxDTO | `UserDTO` |
| VO | XxxVO | `UserVO` |
| Param | XxxParam/CreateXxxParam | `UserCreateParam` |
| AppService | XxxAppService | `UserAppService` |
| DomainService | XxxDomainService | `UserDomainService` |
| Mapper | XxxMapper | `UserMapper` |
| Controller | XxxController | `UserController` |

### 工具类优先级

Apache Commons > Hutool > Spring Framework > 自定义工具类

---

## 服务端排序

- Entity：统计字段使用 `@TableField(exist=false)`，ID 为 Long 类型
- VO：ID 使用 String 类型，统计字段保持 Long
- DomainService：支持 sortBy/sortOrder，默认 updatedAt DESC
- 复杂排序逻辑在 MyBatis XML 中使用 `<choose>/<when>` 实现
- Controller：/page 端点接收 pageNum、pageSize、sortBy、sortOrder

---

## 行级数据权限

- 使用 `@DataPermission` 注解标记 Mapper 方法
- DataScopeType 枚举：ALL、DEPT_AND_CHILD、DEPT_ONLY、SELF_ONLY、CUSTOM
- 通过 `DataPermissionInterceptor` 自动注入 WHERE 条件

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| `@Autowired` 字段注入 | 无法使用 final，测试困难 |
| `@Select/@Insert` 注解 | SQL难以维护和审查 |
| `IdType.AUTO` 自增 | 分布式不友好 |
| DomainService 加 `@Transactional` | 事务边界混乱 |
| 留空 `// TODO` 注释 | 所有问题必须解决 |
| Service 中直接 BeanUtils.copyProperties | 必须通过 Assembly |
| 禁止吞异常 | 隐藏错误导致问题难以定位 |

---

## 详细规范

- **[ddd-architecture.md](references/ddd-architecture.md)** — DDD 5层架构详解、调用关系、双Assembly设计、完整开发示例
- **[spring-boot.md](references/spring-boot.md)** — 依赖注入、事务管理、异常处理、参数验证、Jackson序列化
- **[mybatis-plus.md](references/mybatis-plus.md)** — 实体定义、主键策略、SQL标准、兼容性说明、常用方法
- **[mybatis-plus-config.md](references/mybatis-plus-config.md)** — 拦截器链、分页、字段填充、逻辑删除、乐观锁、行级权限、SQL审计
- **[java-coding.md](references/java-coding.md)** — Lombok、JavaDoc、命名规范、代码简洁、工具类、异常处理
- **[package-structure.md](references/package-structure.md)** — 模块划分、目录结构、文件命名、迁移规则
- **[dto-convention.md](references/dto-convention.md)** — DTO转换规范、Assembly职责、自查清单
- **[data-permission.md](references/data-permission.md)** — 行级数据权限设计、实现、最佳实践
- **[server-side-sorting-checklist.md](references/server-side-sorting-checklist.md)** — 前后端排序检查清单、ID处理
- **[testing.md](references/testing.md)** — 后端测试框架、Mock使用、覆盖率、集成测试
- **[transaction-refactor-guide.md](references/transaction-refactor-guide.md)** — 事务从 DomainService 迁移到 AppService 的重构指南
