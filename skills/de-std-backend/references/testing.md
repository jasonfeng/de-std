# 后端测试规范

> 本规范定义后端测试的编写规范，确保测试质量和覆盖率。

---

## 📋 目录

- [测试框架](#测试框架)
- [测试结构](#测试结构)
- [测试编写规范](#测试编写规范)
- [Mock使用](#mock使用)
- [测试覆盖率](#测试覆盖率)
- [测试数据管理](#测试数据管理)
- [常见测试场景](#常见测试场景)

---

## 测试框架

### 核心框架

| 框架 | 版本 | 用途 |
|------|------|------|
| **JUnit 5** | 5.x | 测试框架 |
| **Mockito** | 最新 | Mock框架 |
| **AssertJ** | 最新 | 断言库 |
| **Spring Boot Test** | 3.x | Spring Boot测试支持 |

### Maven依赖

```xml
<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Spring Boot Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- H2 数据库（测试用） -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 测试结构

### 测试类位置

```
src/test/java/com/dp/dataengine/
├── domain/service/user/
│   └── UserDomainServiceTest.java
├── application/service/user/
│   └── UserAppServiceTest.java
└── interfaces/controller/user/
    └── UserControllerTest.java
```

### 测试类命名

| 类名 | 测试类名 | 示例 |
|------|---------|------|
| `UserDomainService` | `UserDomainServiceTest` | ✅ 正确 |
| `UserDomainService` | `UserDomainServiceTestNG` | ❌ 错误 |
| `UserAppService` | `UserAppServiceTest` | ✅ 正确 |

### 测试方法命名

**格式**：`{方法名}_{场景}_{预期结果}`

| 方法 | 测试方法名 | 说明 |
|------|-----------|------|
| `createUser` | `createUser_Success` | 创建用户成功 |
| `createUser` | `createUser_DuplicateUsername_Exception` | 用户名重复抛异常 |
| `getUserById` | `getUserById_NotFound_Null` | 用户不存在返回null |
| `getUserById` | `getUserById_Deleted_Null` | 用户已删除返回null |

---

## 测试编写规范

### 基本结构

```java
@ExtendWith(MockitoExtension.class)
class UserDomainServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserDomainService userDomainService;

    @Test
    @DisplayName("创建用户 - 成功")
    void createUser_Success() {
        // Given
        UserDTO dto = new UserDTO();
        dto.setUsername("admin");
        dto.setEmail("admin@example.com");

        when(userRepository.insert(any(UserEntity.class)))
            .thenAnswer(invocation -> {
                UserEntity entity = invocation.getArgument(0);
                entity.setId(1L);
                return 1;
            });

        // When
        UserDTO result = userDomainService.createUser(dto);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getUsername()).isEqualTo("admin");
        verify(userRepository, times(1)).insert(any(UserEntity.class));
    }
}
```

### Given-When-Then模式

**使用注释标注**：

```java
@Test
@DisplayName("创建用户 - 成功")
void createUser_Success() {
    // Given - 准备测试数据
    UserDTO dto = new UserDTO();
    dto.setUsername("admin");
    dto.setEmail("admin@example.com");

    when(userRepository.insert(any(UserEntity.class)))
        .thenAnswer(invocation -> {
            UserEntity entity = invocation.getArgument(0);
            entity.setId(1L);
            return 1;
        });

    // When - 执行测试
    UserDTO result = userDomainService.createUser(dto);

    // Then - 验证结果
    assertThat(result).isNotNull();
    assertThat(result.getId()).isEqualTo(1L);
}
```

### @DisplayName

**使用@DisplayName描述测试用例**：

```java
@Test
@DisplayName("创建用户 - 成功")
void createUser_Success() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名为空 - 抛异常")
void createUser_UsernameBlank_Exception() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名已存在 - 抛异常")
void createUser_DuplicateUsername_Exception() { /* ... */ }
```

---

## Mock使用

### @Mock

**创建Mock对象**：

```java
@ExtendWith(MockitoExtension.class)
class UserDomainServiceTest {

    @Mock
    private UserRepository userRepository;  // Mock对象

    @InjectMocks
    private UserDomainService userDomainService;  // 自动注入Mock对象

    @Test
    void test() {
        // userRepository是Mock对象，可以定义行为
        when(userRepository.selectById(1L))
            .thenReturn(createMockUser());

        UserEntity user = userRepository.selectById(1L);
        assertThat(user.getUsername()).isEqualTo("admin");
    }
}
```

### when().thenReturn()

**定义Mock方法返回值**：

```java
// 返回固定值
when(userRepository.selectById(1L))
    .thenReturn(mockUser);

// 返回null
when(userRepository.selectById(999L))
    .thenReturn(null);

// 返回列表
when(userRepository.selectList(any()))
    .thenReturn(Arrays.asList(mockUser1, mockUser2));
```

### when().thenThrow()

**定义Mock方法抛出异常**：

```java
// 抛出业务异常
when(userRepository.existsByUsername("admin"))
    .thenReturn(true);

// 抛出数据库异常
when(userRepository.insert(any(UserEntity.class)))
    .thenThrow(new DataIntegrityViolationException("Duplicate key"));
```

### thenReturn().thenThrow()

**链式调用**：

```java
// 第一次调用返回正常值，第二次调用抛出异常
when(userRepository.selectById(1L))
    .thenReturn(mockUser)
    .thenThrow(new RuntimeException("Database error"));
```

### ArgumentMatchers

**参数匹配**：

```java
// 匹配任意参数
when(userRepository.selectById(anyLong()))
    .thenReturn(mockUser);

// 匹配特定参数
when(userRepository.selectById(1L))
    .thenReturn(mockUser);

// 匹配字符串包含
when(userRepository.findByUsername(contains("admin")))
    .thenReturn(mockUser);

// 匹配自定义条件
when(userRepository.selectList(argThat(list -> list != null && !list.isEmpty())))
    .thenReturn(mockUsers);
```

---

## 测试覆盖率

### 分层标准

| 层级 | 指令覆盖率 | 分支覆盖率 | 方法覆盖率 | 说明 |
|------|-----------|-----------|-----------|------|
| **DomainService** | ≥80% | ≥65% | ≥85% | 核心业务逻辑 |
| **AppService** | ≥65% | ≥55% | ≥75% | 应用服务编排 |
| **Controller** | ≥50% | ≥40% | ≥60% | 接口层（集成测试） |
| **Common/Infrastructure** | ≥70% | ≥60% | ≥75% | 工具类/基础设施 |
| **Entity/VO/Param/DTO** | 不强制 | 不强制 | 不强制 | 数据载体类 |

### JaCoCo配置

**排除类**（不统计覆盖率）：
- `**/entity/**` - 实体类
- `**/vo/**` - 视图对象
- `**/param/**` - 请求参数
- `**/dto/**` - 数据传输对象
- `**/enums/**` - 枚举类

### 查看覆盖率

```bash
# 运行测试并生成覆盖率报告
mvn clean test

# 查看HTML报告
open target/site/jacoco/index.html

# 查看CSV数据
cat target/site/jacoco/jacoco.csv
```

---

## 测试数据管理

### @BeforeEach

**每个测试前执行**：

```java
@ExtendWith(MockitoExtension.class)
class UserDomainServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserDomainService userDomainService;

    @BeforeEach
    void setUp() {
        // 每个测试前执行
        // 重置Mock对象
        reset(userRepository);
    }
}
```

### @MethodSource

**参数化测试**：

```java
@ParameterizedTest
@MethodSource("provideUsernames")
@DisplayName("创建用户 - 用户名{0} - {1}")
void createUser_Username(String username, String expectedStatus) {
    // Given
    UserDTO dto = new UserDTO();
    dto.setUsername(username);

    // When & Then
    if ("SUCCESS".equals(expectedStatus)) {
        UserDTO result = userDomainService.createUser(dto);
        assertThat(result).isNotNull();
    } else {
        assertThatThrownBy(() -> userDomainService.createUser(dto))
            .isInstanceOf(BusinessException.class);
    }
}

static Stream<Arguments> provideUsernames() {
    return Stream.of(
        Arguments.of("admin", "SUCCESS"),
        Arguments.of("", "INVALID"),
        Arguments.of("ab", "INVALID")
    );
}
```

### @TempDir

**临时文件**：

```java
@TempDir
Path tempDir;

@Test
void testWithTempFile() throws IOException {
    // 创建临时文件
    Path file = tempDir.resolve("test.txt");
    Files.write(file, "test content".getBytes());

    // 测试逻辑
    assertThat(Files.exists(file)).isTrue();
    assertThat(Files.readString(file)).isEqualTo("test content");
}
```

---

## 常见测试场景

### 1. 测试正常流程

```java
@Test
@DisplayName("创建用户 - 成功")
void createUser_Success() {
    // Given
    UserDTO dto = new UserDTO();
    dto.setUsername("admin");
    dto.setEmail("admin@example.com");

    when(userRepository.insert(any(UserEntity.class)))
        .thenAnswer(invocation -> {
            UserEntity entity = invocation.getArgument(0);
            entity.setId(1L);
            return 1;
        });

    // When
    UserDTO result = userDomainService.createUser(dto);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getId()).isEqualTo(1L);
    assertThat(result.getUsername()).isEqualTo("admin");
}
```

### 2. 测试异常场景

```java
@Test
@DisplayName("创建用户 - 用户名为空 - 抛异常")
void createUser_UsernameBlank_Exception() {
    // Given
    UserDTO dto = new UserDTO();
    dto.setUsername("");

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(dto))
        .isInstanceOf(BusinessException.class)
        .hasMessage("用户名不能为空");

    verify(userRepository, never()).insert(any(UserEntity.class));
}
```

### 3. 测试条件分支

```java
@Test
@DisplayName("查询用户 - 用户不存在 - 返回null")
void getUserById_NotFound_Null() {
    // Given
    Long userId = 999L;
    when(userRepository.selectById(userId))
        .thenReturn(null);

    // When
    UserDTO result = userDomainService.getUserById(userId);

    // Then
    assertThat(result).isNull();
}

@Test
@DisplayName("查询用户 - 用户已删除 - 抛异常")
void getUserById_Deleted_Exception() {
    // Given
    Long userId = 1L;
    UserEntity entity = new UserEntity();
    entity.setId(userId);
    entity.setDeleted(1);

    when(userRepository.selectById(userId))
        .thenReturn(entity);

    // When & Then
    assertThatThrownBy(() -> userDomainService.getUserById(userId))
        .isInstanceOf(BusinessException.class)
        .hasMessage("用户已被删除");
}
```

### 4. 测试列表操作

```java
@Test
@DisplayName("查询所有用户 - 成功")
void findAllUsers_Success() {
    // Given
    when(userRepository.selectList(any()))
        .thenReturn(Arrays.asList(mockUser1, mockUser2));

    // When
    List<UserDTO> result = userDomainService.findAllUsers();

    // Then
    assertThat(result).hasSize(2);
    assertThat(result.get(0).getUsername()).isEqualTo("admin");
    assertThat(result.get(1).getUsername()).isEqualTo("user");
}
```

### 5. 测试事务

```java
@SpringBootTest
@Transactional
class UserAppServiceIntegrationTest {

    @Autowired
    private UserAppService userAppService;

    @Test
    @DisplayName("创建用户 - 事务回滚测试")
    void createUser_TransactionRollback() {
        // Given
        UserDTO dto = new UserDTO();
        dto.setUsername("test");

        // When & Then - 事务会回滚
        assertThatThrownBy(() -> userAppService.createUserWithInvalidData(dto))
            .isInstanceOf(Exception.class);

        // 验证数据库中没有插入
        // ...
    }
}
```

---

## 集成测试

### @SpringBootTest

**使用H2内存数据库**：

```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
class UserControllerIntegrationTest {

    @Autowired
    private UserController userController;

    @MockBean
    private UserAppService userAppService;

    @Test
    @DisplayName("创建用户 - 集成测试")
    void createUser_Integration() {
        // Given
        CreateUserParam param = new CreateUserParam();
        param.setUsername("admin");

        when(userAppService.createUser(any()))
            .thenReturn(createMockDTO());

        // When
        ResponseEntity<UserVO> response = userController.createUser(param);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
    }
}
```

### application-test.properties

```properties
# 使用H2数据库
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA配置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

## 🔗 相关文档

- [测试覆盖率标准](../04-testing/coverage-standards.md) - 覆盖率标准详细规范
- [Java编码规范](./java-coding.md) - Java编码详细规范
- [DDD架构规范](./ddd-architecture.md) - DDD架构详细规范

---

**最后更新**：2026-03-10