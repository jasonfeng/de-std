# 测试数据管理

> 本规范定义测试数据的管理规范，确保测试数据的隔离性和可维护性。

---

## 📋 目录

- [测试数据来源](#测试数据来源)
- [测试数据隔离](#测试数据隔离)
- [测试数据清理](#测试数据清理)
- [测试数据工厂](#测试数据工厂)
- [测试数据存储](#测试数据存储)

---

## 测试数据来源

### @MethodSource

```java
@ParameterizedTest
@MethodSource("provideUserData")
@DisplayName("创建用户 - 多个用户")
void createUser_MultipleUsers(UserDTO dto) {
    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo(dto.getUsername());
}

private static Stream<UserDTO> provideUserData() {
    return Stream.of(
        createUserDTO("admin", "admin@example.com"),
        createUserDTO("user", "user@example.com"),
        createUserDTO("test", "test@example.com")
    );
}

private static UserDTO createUserDTO(String username, String email) {
    UserDTO dto = new UserDTO();
    dto.setUsername(username);
    dto.setEmail(email);
    return dto;
}
```

### @CsvSource

```java
@ParameterizedTest
@CsvSource({
    "admin, admin@example.com, 30",
    "user, user@example.com, 25",
    "test, test@example.com, 35"
})
@DisplayName("创建用户 - CSV数据源")
void createUser_CsvSource(String username, String email, int age) {
    UserDTO dto = new UserDTO();
    dto.setUsername(username);
    dto.setEmail(email);
    dto.setAge(age);

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo(username);
}
```

### @CsvFileSource

```java
@ParameterizedTest
@CsvFileSource(resources = "/test-data/users.csv")
@DisplayName("创建用户 - CSV文件数据源")
void createUser_CsvFileSource(String username, String email, int age) {
    UserDTO dto = new UserDTO();
    dto.setUsername(username);
    dto.setEmail(email);
    dto.setAge(age);

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo(username);
}
```

**users.csv**:
```csv
username,email,age
admin,admin@example.com,30
user,user@example.com,25
test,test@example.com,35
```

### @ArgumentsSource

```java
@ParameterizedTest
@ArgumentsSource(UserArgumentsProvider.class)
@DisplayName("创建用户 - 自定义参数提供器")
void createUser_CustomProvider(UserDTO dto) {
    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo(dto.getUsername());
}

static class UserArgumentsProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of(
            Arguments.of(createUserDTO("admin", "admin@example.com")),
            Arguments.of(createUserDTO("user", "user@example.com")),
            Arguments.of(createUserDTO("test", "test@example.com"))
        );
    }

    private UserDTO createUserDTO(String username, String email) {
        UserDTO dto = new UserDTO();
        dto.setUsername(username);
        dto.setEmail(email);
        return dto;
    }
}
```

---

## 测试数据隔离

### @BeforeEach

```java
@SpringBootTest
class UserDomainServiceTest {

    @Autowired
    private UserDomainService userDomainService;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        // 每个测试前清理数据
        userRepository.deleteAll();
    }

    @Test
    void createUser_Success() {
        UserDTO dto = new UserDTO();
        dto.setUsername("admin");

        UserDTO result = userDomainService.createUser(dto);

        assertThat(result).isNotNull();
    }
}
```

### @AfterEach

```java
@SpringBootTest
class UserDomainServiceTest {

    @Autowired
    private UserDomainService userDomainService;

    @Autowired
    private UserRepository userRepository;

    @AfterEach
    void tearDown() {
        // 每个测试后清理数据
        userRepository.deleteAll();
    }

    @Test
    void createUser_Success() {
        UserDTO dto = new UserDTO();
        dto.setUsername("admin");

        UserDTO result = userDomainService.createUser(dto);

        assertThat(result).isNotNull();
    }
}
```

### @Transactional

```java
@SpringBootTest
class UserDomainServiceTest {

    @Autowired
    private UserDomainService userDomainService;

    @Test
    @DisplayName("创建用户 - 事务回滚")
    @Transactional
    void createUser_TransactionRollback() {
        UserDTO dto = new UserDTO();
        dto.setUsername("admin");

        UserDTO result = userDomainService.createUser(dto);

        assertThat(result).isNotNull();

        // 测试结束后事务回滚，不影响数据库
    }
}
```

### @Rollback

```java
@SpringBootTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("插入用户 - 回滚")
    @Rollback
    void insert_Rollback() {
        UserEntity user = new UserEntity();
        user.setUsername("admin");

        userRepository.insert(user);

        // 测试结束后回滚，不影响数据库
    }
}
```

---

## 测试数据清理

### 清理策略

```java
@SpringBootTest
class UserDataCleanupTest {

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        // 策略1：清空所有数据
        userRepository.deleteAll();
    }

    @BeforeEach
    void setUp2() {
        // 策略2：按条件删除
        userRepository.deleteByUsername("admin");
    }

    @BeforeEach
    void setUp3() {
        // 策略3：删除测试数据（带前缀）
        userRepository.deleteByUsernameStartingWith("test_");
    }

    @AfterEach
    void tearDown() {
        // 策略4：每个测试后清理
        userRepository.deleteAll();
    }
}
```

### @DirtiesContext

```java
@SpringBootTest
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)
class UserDomainServiceTest {

    // 每个测试后重建Spring上下文
    // 适用于测试后需要完全重置状态的场景
}
```

### TestExecutionListener

```java
@TestExecutionListeners(
    listeners = {TestDataCleanupListener.class},
    mergeMode = TestExecutionListeners.MergeMode.MERGE_WITH_DEFAULTS
)
@SpringBootTest
class UserDomainServiceTest {

    // 使用自定义监听器清理测试数据
}
```

---

## 测试数据工厂

### 基本工厂

```java
public class TestDataFactory {

    /**
     * 创建用户DTO
     */
    public static UserDTO createUserDTO(String username) {
        UserDTO dto = new UserDTO();
        dto.setUsername(username);
        dto.setEmail(username + "@example.com");
        dto.setAge(30);
        return dto;
    }

    /**
     * 创建用户实体
     */
    public static UserEntity createUserEntity(String username) {
        UserEntity entity = new UserEntity();
        entity.setUsername(username);
        entity.setEmail(username + "@example.com");
        entity.setAge(30);
        return entity;
    }

    /**
     * 创建用户VO
     */
    public static UserVO createUserVO(Long id, String username) {
        UserVO vo = new UserVO();
        vo.setId(id);
        vo.setUsername(username);
        vo.setEmail(username + "@example.com");
        vo.setAge(30);
        return vo;
    }
}
```

### 使用示例

```java
@SpringBootTest
class UserDomainServiceTest {

    @Autowired
    private UserDomainService userDomainService;

    @Test
    @DisplayName("创建用户 - 成功")
    void createUser_Success() {
        // Given
        UserDTO dto = TestDataFactory.createUserDTO("admin");

        // When
        UserDTO result = userDomainService.createUser(dto);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getUsername()).isEqualTo("admin");
    }
}
```

### 链式工厂

```java
public class UserBuilder {

    private final UserDTO user;

    private UserBuilder() {
        this.user = new UserDTO();
    }

    public static UserBuilder builder() {
        return new UserBuilder();
    }

    public UserBuilder username(String username) {
        user.setUsername(username);
        return this;
    }

    public UserBuilder email(String email) {
        user.setEmail(email);
        return this;
    }

    public UserBuilder age(int age) {
        user.setAge(age);
        return this;
    }

    public UserDTO build() {
        return user;
    }
}
```

### 使用链式工厂

```java
@Test
void createUser_Builder_Success() {
    // Given
    UserDTO dto = UserBuilder.builder()
        .username("admin")
        .email("admin@example.com")
        .age(30)
        .build();

    // When
    UserDTO result = userDomainService.createUser(dto);

    // Then
    assertThat(result).isNotNull();
}
```

---

## 测试数据存储

### SQL文件存储

```sql
-- src/test/resources/test-data/users.sql
INSERT INTO t_user (id, user_name, email, age, created_at)
VALUES (1, 'admin', 'admin@example.com', 30, NOW());

INSERT INTO t_user (id, user_name, email, age, created_at)
VALUES (2, 'user', 'user@example.com', 25, NOW());
```

### 加载SQL文件

```java
@SpringBootTest
@Sql(scripts = "/test-data/users.sql")
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void selectById_Success() {
        UserEntity user = userRepository.selectById(1L);

        assertThat(user).isNotNull();
        assertThat(user.getUsername()).isEqualTo("admin");
    }
}
```

### @Sql配置

```java
@SpringBootTest
@Sql(
    scripts = {
        "/test-data/cleanup.sql",
        "/test-data/users.sql"
    },
    executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD
)
class UserRepositoryTest {

    @Test
    void selectById_Success() {
        UserEntity user = userRepository.selectById(1L);

        assertThat(user).isNotNull();
    }

    @Test
    @Sql(scripts = "/test-data/cleanup.sql")
    void deleteUser_Success() {
        userRepository.deleteById(1L);

        UserEntity user = userRepository.selectById(1L);
        assertThat(user).isNull();
    }
}
```

### @SqlConfig

```java
@SpringBootTest
@SqlConfig(
    dataSource = "dataSource",
    transactionMode = SqlConfig.TransactionMode.ISOLATED,
    transactionManager = "transactionManager"
)
@Sql(scripts = "/test-data/users.sql")
class UserRepositoryTest {

    // 使用自定义SQL配置
}
```

---

## 🔗 相关文档

- [单元测试规范](./unit-testing.md) - 单元测试详细规范
- [集成测试规范](./integration-testing.md) - 集成测试详细规范
- [覆盖率标准](./coverage-standards.md) - 覆盖率标准详细规范

---

**最后更新**：2026-03-10