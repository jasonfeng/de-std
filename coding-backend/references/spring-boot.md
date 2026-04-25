# Spring Boot规范

> 本规范定义Spring Boot框架的使用规范，确保代码质量和一致性。

---

## 📋 目录

- [依赖注入规范](#依赖注入规范-⚠️强制)
- [事务管理规范](#事务管理规范)
- [配置管理规范](#配置管理规范)
- [异常处理规范](#异常处理规范)
- [验证规范](#验证规范)

---

## 依赖注入规范（⚠️强制）

### 必须使用构造器注入

**❌ 错误（禁止字段注入）**：
```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
}
```

**❌ 错误（禁止Setter注入）**：
```java
@Service
public class UserService {
    private UserMapper userMapper;

    @Autowired
    public void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }
}
```

**✅ 正确（构造器注入）**：
```java
@Service
@RequiredArgsConstructor  // Lombok生成构造器
public class UserService {
    private final UserMapper userMapper;
}
```

### 为什么使用构造器注入？

| 对比项 | 构造器注入 | 字段注入 |
|--------|-----------|---------|
| **对象不可变** | ✅ 可使用final修饰依赖 | ❌ 依赖必须是可变的 |
| **测试简易性** | ✅ 直接new时传入Mock对象 | ❌ 需要反射或额外框架 |
| **依赖清晰** | ✅ 构造器参数一眼看出所有依赖 | ❌ 无法一眼看出依赖关系 |
| **依赖非空保证** | ✅ 构造器确保依赖不为空 | ❌ 可能产生null依赖 |
| **Spring推荐** | ✅ Spring官方推荐方式 | ❌ 不推荐 |

### @RequiredArgsConstructor使用

**基本用法**：
```java
@Service
@RequiredArgsConstructor
public class UserAppService {
    // final字段自动包含在构造器中
    private final UserDomainService userDomainService;
    private final DepartmentDomainService departmentDomainService;

    // 非final字段不会包含在构造器中
    private String someField;

    public UserDTO createUser(UserDTO dto) {
        return userDomainService.createUser(dto);
    }
}
```

**等价于**：
```java
@Service
public class UserAppService {
    private final UserDomainService userDomainService;
    private final DepartmentDomainService departmentDomainService;
    private String someField;

    // Lombok自动生成
    public UserAppService(
        UserDomainService userDomainService,
        DepartmentDomainService departmentDomainService
    ) {
        this.userDomainService = userDomainService;
        this.departmentDomainService = departmentDomainService;
    }
}
```

### 循环依赖处理

**问题**：如果A依赖B，B依赖A，会报错。

**解决方案**：
1. **重构设计**（推荐）- 消除循环依赖
2. 使用`@Lazy`注解

```java
@Service
@RequiredArgsConstructor
public class ServiceA {
    private final ServiceB serviceB;
}

@Service
@RequiredArgsConstructor
public class ServiceB {
    @Lazy  // 延迟加载，解决循环依赖
    private final ServiceA serviceA;
}
```

### 测试对比示例

**构造器注入测试简单**：
```java
@ExtendWith(MockitoExtension.class)
class UserAppServiceTest {

    @Mock
    private UserDomainService userDomainService;

    @InjectMocks
    private UserAppService userAppService;

    @Test
    void testCreateUser() {
        // 直接构造，无需额外步骤
        UserDTO result = userAppService.createUser(userDto);
        // ...
    }
}
```

**字段注入需要额外步骤**：
```java
@ExtendWith(MockitoExtension.class)
class UserAppServiceTest {

    @InjectMocks
    private UserAppService userAppService;

    @Mock
    private UserDomainService userDomainService;

    @BeforeEach
    void setUp() {
        // 需要额外步骤注入Mock
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void testCreateUser() {
        // ...
    }
}
```

---

## 事务管理规范

### @Transactional使用

**基本用法**：
```java
@Service
@RequiredArgsConstructor
public class UserAppService {
    private final UserDomainService userDomainService;
    private final RoleDomainService roleDomainService;

    @Transactional  // 开启事务
    public void createUserWithRole(UserDTO userDto, RoleDTO roleDto) {
        // 执行多个数据库操作
        userDomainService.createUser(userDto);
        roleDomainService.createRole(roleDto);
        // 如果中间出现异常，会自动回滚
    }
}
```

### 事务传播行为

```java
@Service
@RequiredArgsConstructor
public class OrderAppService {
    private final OrderDomainService orderDomainService;
    private final InventoryDomainService inventoryDomainService;

    @Transactional
    public void createOrder(OrderDTO orderDto) {
        // 默认：REQUIRED（如果存在事务则加入，否则创建新事务）
        orderDomainService.createOrder(orderDto);
        inventoryDomainService.reduceStock(orderDto);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)  // 创建新事务
    public void logOrder(OrderDTO orderDto) {
        // 总是创建新事务，独立于外部事务
    }

    @Transactional(propagation = Propagation.NEVER)  // 以非事务方式运行
    public void sendNotification(OrderDTO orderDto) {
        // 以非事务方式运行，如果存在事务则抛出异常
    }
}
```

### 事务隔离级别

```java
@Service
@RequiredArgsConstructor
public class OrderAppService {

    @Transactional(isolation = Isolation.READ_COMMITTED)  // 读已提交
    public void createOrder(OrderDTO orderDto) {
        // 使用读已提交隔离级别
    }

    @Transactional(isolation = Isolation.REPEATABLE_READ)  // 可重复读
    public OrderDTO getOrder(Long id) {
        // 使用可重复读隔离级别
    }

    @Transactional(isolation = Isolation.SERIALIZABLE)  // 串行化
    public void updateOrder(OrderDTO orderDto) {
        // 使用串行化隔离级别（最高隔离级别）
    }
}
```

### 事务回滚

**默认行为**：`RuntimeException`和`Error`会回滚，`checked exception`不会回滚。

```java
@Service
@RequiredArgsConstructor
public class UserAppService {

    @Transactional(rollbackFor = BusinessException.class)  // 指定回滚异常
    public void createUser(UserDTO userDto) throws BusinessException {
        // ...
    }

    @Transactional(noRollbackFor = BusinessException.class)  // 指定不回滚异常
    public void logOperation(UserDTO userDto) throws BusinessException {
        // ...
    }

    @Transactional(rollbackForClassName = {"java.lang.Exception"})  // 指定回滚所有异常
    public void createUserWithRollback(UserDTO userDto) throws Exception {
        // ...
    }
}
```

### 只读事务

```java
@Service
@RequiredArgsConstructor
public class UserAppService {

    @Transactional(readOnly = true)  // 只读事务
    public UserDTO getUser(Long id) {
        // 只读事务，可以优化性能
    }
}
```

---

## 配置管理规范

### @ConfigurationProperties

**使用类型安全的配置**：

```java
// application.yml
app:
  name: DataEngine
  version: 1.0.0
  features:
    user-management: true
    role-management: true
```

```java
@Configuration
@ConfigurationProperties(prefix = "app")
@Data
public class AppProperties {
    private String name;
    private String version;
    private Features features = new Features();

    @Data
    public static class Features {
        private boolean userManagement;
        private boolean roleManagement;
    }
}
```

**使用配置**：
```java
@Service
@RequiredArgsConstructor
public class SomeService {
    private final AppProperties appProperties;

    public void doSomething() {
        if (appProperties.getFeatures().isUserManagement()) {
            // ...
        }
    }
}
```

### @Value使用

**简单配置使用@Value**：

```java
@Component
public class SomeComponent {
    @Value("${app.name}")
    private String appName;

    @Value("${app.max.connections:10}")
    private int maxConnections;  // 默认值10

    @Value("#{systemProperties['user.home']}")
    private String userHome;  // SpEL表达式
}
```

---

## 异常处理规范

### @ControllerAdvice

**全局异常处理**：

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 业务异常处理。
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResult> handleBusinessException(BusinessException ex) {
        ErrorResult error = new ErrorResult(ex.getCode(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    /**
     * 参数校验异常处理。
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResult> handleValidationException(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.toList());
        ErrorResult error = new ErrorResult("VALIDATION_ERROR", "参数校验失败", errors);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    /**
     * 系统异常处理。
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResult> handleException(Exception ex) {
        log.error("系统异常", ex);
        ErrorResult error = new ErrorResult("SYSTEM_ERROR", "系统内部错误");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

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

---

## 验证规范

### @Valid使用

**Controller层验证**：

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    @PostMapping
    public ResponseEntity<Void> createUser(@Valid @RequestBody CreateUserParam param) {
        // @Valid触发参数校验
        userService.createUser(param);
        return ResponseEntity.ok().build();
    }
}
```

**Param定义校验规则**：

```java
@Data
public class CreateUserParam {

    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 50, message = "用户名长度必须在3-50之间")
    private String username;

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;

    @Min(value = 1, message = "部门ID必须大于0")
    private Long departmentId;
}
```

### 自定义校验注解

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UsernameValidator.class)
public @interface ValidUsername {
    String message() default "用户名格式不正确";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

```java
public class UsernameValidator implements ConstraintValidator<ValidUsername, String> {

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        // 校验逻辑
        return value.matches("^[a-zA-Z0-9_]{3,50}$");
    }
}
```

---

## Jackson序列化规范（⚠️强制）

### 雪花算法ID处理规范

**问题**：雪花算法生成的ID是19位数字，超过JavaScript的安全整数范围（2^53），导致前端精度丢失。

**解决方案**：VO层ID字段使用`String`类型，在Assembly层手动转换。

#### 各层ID字段类型规范

| 层级 | ID字段类型 | 说明 |
|------|-----------|------|
| **Entity** | `Long` | 数据库主键，使用雪花算法 |
| **DTO** | `Long` | 内部传输对象，保持Long |
| **VO** | `String` | 响应对象，避免前端精度丢失 |
| **前端TypeScript** | `string` | 接口定义使用string |

#### VO层实现示例

```java
@Data
@Schema(description = "字典类型响应对象")
public class DictTypeVO implements Serializable {
    private static final long serialVersionUID = 1L;

    // 使用String避免前端精度丢失
    private String id;

    private String typeCode;
    private String typeName;
    // ...
}
```

#### Assembly层手动转换

```java
public static DictTypeVO dtoToVO(DictTypeDTO dto) {
    if (dto == null) {
        return null;
    }
    DictTypeVO vo = new DictTypeVO();
    BeanUtils.copyProperties(dto, vo);
    // 手动转换ID为String，避免前端精度丢失
    if (dto.getId() != null) {
        vo.setId(dto.getId().toString());
    }
    return vo;
}
```

#### Jackson配置

**❌ 错误（全局Long序列化）**：
```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer jacksonCustomizer() {
    return builder -> {
        SimpleModule longModule = new SimpleModule();
        // 这会导致所有Long字段（包括分页数字）都被序列化为string
        longModule.addSerializer(Long.class, ToStringSerializer.instance);
        builder.modules(new JavaTimeModule(), longModule);
    };
}
```

**✅ 正确（只配置时间模块）**：
```java
@Configuration
public class JacksonConfig {

    /**
     * 配置Jackson序列化行为
     *
     * 只注册JavaTimeModule，移除Long全局序列化配置
     * ID字段在VO层使用String类型避免精度问题
     */
    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jacksonCustomizer() {
        return builder -> {
            // 只注册JavaTimeModule，不序列化Long为字符串
            builder.modules(new JavaTimeModule());
        };
    }
}
```

#### 统计字段类型规范

| 字段类型 | Entity | DTO | VO | 前端 |
|---------|--------|-----|-----|------|
| **数量统计** (itemCount, totalCount) | `Long` | `Long` | `Long` | `number` |
| **金额统计** | `Long` (分) | `Long` | `Long` | `number` |
| **分页数字** (total, pageNum) | `Long` | `Long` | `Long` | `number` |

**原则**：数量/金额/分页等在安全范围内的数字使用`Long`/`number`

---

## 分页与排序规范

### 服务端排序实现

#### 后端职责

1. 支持`sortBy`和`sortOrder`参数
2. 在DomainService层实现排序逻辑
3. 默认按`updatedAt`降序

```java
public IPage<DictTypeEntity> getDictTypesPage(
    Page<DictTypeEntity> page,
    String sortBy,
    String sortOrder
) {
    LambdaQueryWrapper<DictTypeEntity> wrapper = new LambdaQueryWrapper<>();

    // 处理排序
    if (StringUtils.hasText(sortBy)) {
        boolean isAsc = "asc".equalsIgnoreCase(sortOrder);
        switch (sortBy) {
            case "createdAt":
                wrapper.orderBy(true, isAsc, DictTypeEntity::getCreatedAt);
                break;
            case "updatedAt":
            default:
                wrapper.orderBy(true, isAsc, DictTypeEntity::getUpdatedAt);
                break;
        }
    } else {
        // 默认按更新时间降序
        wrapper.orderByDesc(DictTypeEntity::getUpdatedAt);
    }

    return dictTypeRepository.selectPage(page, wrapper);
}
```

#### 统计字段实现

在分页查询时统计关联数据：

```java
public IPage<DictTypeEntity> getDictTypesPage(
    Page<DictTypeEntity> page,
    String sortBy,
    String sortOrder
) {
    // ... 排序逻辑

    IPage<DictTypeEntity> resultPage = dictTypeRepository.selectPage(page, wrapper);

    // 为每个字典类型统计字典项数量
    for (DictTypeEntity entity : resultPage.getRecords()) {
        Integer itemCount = dictItemRepository.countByTypeId(entity.getId());
        entity.setItemCount(itemCount != null ? itemCount.longValue() : 0L);
    }

    return resultPage;
}
```

#### Entity添加统计字段

```java
@Data
@TableName("t_dict_type")
public class DictTypeEntity implements Serializable {
    // ... 其他字段

    /**
     * 字典项数量（非数据库字段，用于查询时统计）
     */
    @TableField(exist = false)
    private Long itemCount;
}
```

#### Controller分页接口

```java
@GetMapping("/page")
@Operation(summary = "分页查询字典类型", description = "分页查询字典类型列表，支持排序")
public ResponseEntity<PageResponseVO<DictTypeVO>> getDictTypesPage(
    @RequestParam(defaultValue = "1") Integer pageNum,
    @RequestParam(defaultValue = "10") Integer pageSize,
    @RequestParam(defaultValue = "updatedAt") String sortBy,
    @RequestParam(defaultValue = "desc") String sortOrder
) {
    IPage<DictTypeDTO> dtoPage = dictTypeAppService.getDictTypesPage(
        pageNum, pageSize, sortBy, sortOrder
    );

    // 构造响应
    PageResponseVO<DictTypeVO> response = new PageResponseVO<>();
    response.setRecords(dtoPage.getRecords().stream()
            .map(StandardAppAssembly::dtoToVO)
            .collect(Collectors.toList()));
    response.setTotal(dtoPage.getTotal());
    response.setPageNum((long) dtoPage.getCurrent());
    response.setPageSize((long) dtoPage.getSize());
    response.setPages((long) dtoPage.getPages());

    return ResponseEntity.ok(response);
}
```

#### 分页响应VO

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Schema(description = "分页响应")
public class PageResponseVO<T> {
    private List<T> records;
    private Long total;
    private Long pageNum;
    private Long pageSize;
    private Long pages;
}
```

---

## 🔗 相关文档

- [MyBatis Plus规范](./mybatis-plus.md) - MyBatis Plus详细规范
- [DDD架构规范](./ddd-architecture.md) - DDD架构详细规范
- [Java编码规范](./java-coding.md) - Java编码详细规范

---

**最后更新**：2026-03-10