# 集成测试规范

> 本规范定义集成测试的编写规范，确保系统各组件协作正确性。

---

## 📋 目录

- [测试框架](#测试框架)
- [测试环境配置](#测试环境配置)
- [测试类结构](#测试类结构)
- [测试场景](#测试场景)
- [Mock外部依赖](#mock外部依赖)
- [事务管理](#事务管理)

---

## 测试框架

### 核心框架

| 框架 | 版本 | 用途 |
|------|------|------|
| **Spring Boot Test** | 3.2+ | 集成测试框架 |
| **JUnit 5** | 5.x | 测试框架 |
| **Mockito** | 最新 | Mock框架 |
| **AssertJ** | 最新 | 断言库 |
| **Testcontainers** | 最新 | 容器化测试 |

### 依赖配置

```xml
<dependencies>
    <!-- Spring Boot Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Testcontainers（可选） -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 测试环境配置

### application-test.properties

```properties
# 测试数据库配置
spring.datasource.url=jdbc:postgresql://localhost:5432/dataengine_test
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA配置
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=false

# MyBatis配置
mybatis-plus.configuration.map-underscore-to-camel-case=true
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.slf4j.Slf4jImpl

# Flyway配置（测试环境不执行迁移）
spring.flyway.enabled=false

# 日志配置
logging.level.root=WARN
logging.level.com.dp.dataengine=DEBUG
```

### @TestPropertySource

```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
class UserControllerIntegrationTest {
    // ...
}
```

### 测试容器配置

```java
@Testcontainers
@SpringBootTest
class UserControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:14")
        .withDatabaseName("dataengine_test")
        .withUsername("postgres")
        .withPassword("postgres")
        .withExposedPorts(5432);

    @DynamicPropertySource
    static void registerPgProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    // ...
}
```

---

## 测试类结构

### @SpringBootTest

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("创建用户 - 成功")
    void createUser_Success() throws Exception {
        // Given
        UserCreateRequest request = new UserCreateRequest();
        request.setUsername("admin");
        request.setEmail("admin@example.com");

        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data.username").value("admin"));
    }
}
```

### @WebMvcTest

```java
@WebMvcTest(UserController.class)
@Import(UserAppService.class)
class UserControllerWebMvcTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserAppService userAppService;

    @Test
    @DisplayName("查询用户 - 成功")
    void getUserById_Success() throws Exception {
        // Given
        when(userAppService.getUserById(1L))
            .thenReturn(createUserVO(1L, "admin"));

        // When & Then
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.data.id").value(1))
            .andExpect(jsonPath("$.data.username").value("admin"));
    }
}
```

### @DataJpaTest

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("保存用户 - 成功")
    void saveUser_Success() {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("admin");
        user.setEmail("admin@example.com");

        // When
        UserEntity savedUser = userRepository.insert(user);

        // Then
        assertThat(savedUser).isNotNull();
        assertThat(savedUser.getId()).isNotNull();
        assertThat(savedUser.getUsername()).isEqualTo("admin");
    }
}
```

---

## 测试场景

### 1. REST API测试

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserApiControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("GET /api/users/{id} - 查询用户")
    void getUserById_Success() throws Exception {
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(200))
            .andExpect(jsonPath("$.data.id").value(1));
    }

    @Test
    @DisplayName("POST /api/users - 创建用户")
    void createUser_Success() throws Exception {
        UserCreateRequest request = new UserCreateRequest();
        request.setUsername("admin");
        request.setEmail("admin@example.com");

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(200))
            .andExpect(jsonPath("$.data.username").value("admin"));
    }

    @Test
    @DisplayName("PUT /api/users/{id} - 更新用户")
    void updateUser_Success() throws Exception {
        UserUpdateRequest request = new UserUpdateRequest();
        request.setUsername("newadmin");

        mockMvc.perform(put("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.data.username").value("newadmin"));
    }

    @Test
    @DisplayName("DELETE /api/users/{id} - 删除用户")
    void deleteUser_Success() throws Exception {
        mockMvc.perform(delete("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value(200));
    }
}
```

### 2. Service层集成测试

```java
@SpringBootTest
class UserAppServiceIntegrationTest {

    @Autowired
    private UserAppService userAppService;

    @Test
    @DisplayName("创建用户 - 成功")
    void createUser_Success() {
        // Given
        UserDTO dto = new UserDTO();
        dto.setUsername("admin");
        dto.setEmail("admin@example.com");

        // When
        UserDTO result = userAppService.createUser(dto);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isNotNull();
        assertThat(result.getUsername()).isEqualTo("admin");
    }

    @Test
    @DisplayName("查询用户 - 成功")
    void getUserById_Success() {
        // Given
        Long userId = 1L;

        // When
        UserDTO result = userAppService.getUserById(userId);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(userId);
    }
}
```

### 3. Repository层集成测试

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("插入用户 - 成功")
    void insert_Success() {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("admin");
        user.setEmail("admin@example.com");

        // When
        int count = userRepository.insert(user);

        // Then
        assertThat(count).isEqualTo(1);
        assertThat(user.getId()).isNotNull();
    }

    @Test
    @DisplayName("查询用户 - 成功")
    void selectById_Success() {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("admin");
        userRepository.insert(user);

        // When
        UserEntity result = userRepository.selectById(user.getId());

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getUsername()).isEqualTo("admin");
    }

    @Test
    @DisplayName("更新用户 - 成功")
    void updateById_Success() {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("admin");
        userRepository.insert(user);

        user.setUsername("newadmin");

        // When
        int count = userRepository.updateById(user);

        // Then
        assertThat(count).isEqualTo(1);

        UserEntity updatedUser = userRepository.selectById(user.getId());
        assertThat(updatedUser.getUsername()).isEqualTo("newadmin");
    }

    @Test
    @DisplayName("删除用户 - 成功")
    void deleteById_Success() {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("admin");
        userRepository.insert(user);
        Long userId = user.getId();

        // When
        int count = userRepository.deleteById(userId);

        // Then
        assertThat(count).isEqualTo(1);

        UserEntity deletedUser = userRepository.selectById(userId);
        assertThat(deletedUser).isNull();
    }
}
```

---

## Mock外部依赖

### @MockBean

```java
@SpringBootTest
class UserControllerIntegrationTest {

    @Autowired
    private UserController userController;

    @MockBean
    private UserService userService;

    @Test
    @DisplayName("查询用户 - Mock成功")
    void getUserById_MockSuccess() {
        // Given
        UserVO mockUser = new UserVO();
        mockUser.setId(1L);
        mockUser.setUsername("admin");

        when(userService.getUserById(1L)).thenReturn(mockUser);

        // When
        Result<UserVO> result = userController.getUserById(1L);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getData().getUsername()).isEqualTo("admin");

        verify(userService, times(1)).getUserById(1L);
    }
}
```

### Mock RestTemplate

```java
@SpringBootTest
class ExternalServiceIntegrationTest {

    @Autowired
    private ExternalService externalService;

    @MockBean
    private RestTemplate restTemplate;

    @Test
    @DisplayName("调用外部API - Mock成功")
    void callExternalApi_MockSuccess() {
        // Given
        String mockResponse = "{\"code\":200,\"data\":\"success\"}";
        when(restTemplate.getForObject(anyString(), eq(String.class)))
            .thenReturn(mockResponse);

        // When
        String result = externalService.callExternalApi("https://api.example.com/data");

        // Then
        assertThat(result).isNotNull();
        assertThat(result).isEqualTo(mockResponse);

        verify(restTemplate, times(1)).getForObject(anyString(), eq(String.class));
    }
}
```

---

## 事务管理

### @Transactional

```java
@SpringBootTest
class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("创建用户 - 事务回滚")
    @Transactional
    void createUser_TransactionRollback() {
        // Given
        UserDTO dto = new UserDTO();
        dto.setUsername("admin");

        // When
        UserDTO result = userService.createUser(dto);

        // Then
        assertThat(result).isNotNull();

        // 事务回滚，数据库中不应有数据
        UserEntity user = userRepository.selectById(result.getId());
        assertThat(user).isNull();
    }
}
```

### @Rollback

```java
@SpringBootTest
class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("插入用户 - 回滚")
    @Rollback
    void insert_Rollback() {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("admin");

        // When
        userRepository.insert(user);

        // Then
        // 测试结束后回滚，不影响数据库
    }
}
```

---

## L3 API 集成测试规范（v2.1，2026-04-11 新增）

> 使用 Testcontainers + 真实 PostgreSQL 的 L3 API 集成测试，替代 H2 内存数据库。
> 验证 Controller→AppService→Repository→Database 的完整链路。

### 配置步骤

1. 依赖已配置在 pom.xml（testcontainers 1.19.8 + postgresql + junit-jupiter）
2. 使用 `@ActiveProfiles("integration")` 激活 Testcontainers 配置
3. 继承 `BaseApiIntegrationTest` 基类

### 测试类模板

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("integration")
@DisplayName("L3 - {模块名} API 集成测试")
class {Module}ApiIntegrationTest extends BaseApiIntegrationTest {

    private static final String BASE_URL = "/api/v1/{module}";

    @Nested
    @DisplayName("POST - 创建")
    class Create {
        @Test
        @DisplayName("创建成功")
        void shouldCreate() throws Exception {
            String body = jsonBody("name", "测试数据", "type", "standard");
            var result = mockMvc.perform(post(BASE_URL)
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(body))
                .andExpect(status().isOk())
                .andReturn();
            assertSuccess(result);
            assertDataNotNull(result);
        }
    }
}
```

### 与 H2 集成测试的区别

| 维度 | H2 (test profile) | Testcontainers (integration profile) |
|------|-------------------|-------------------------------------|
| 数据库 | H2 内存 (PostgreSQL 兼容模式) | 真实 PostgreSQL 14 Docker 容器 |
| SQL 兼容性 | 部分（无 JSONB、无枚举类型） | 完全兼容 |
| Flyway 迁移 | 不执行（用 schema.sql） | 完整执行所有 V*.sql 迁移 |
| 启动速度 | 快（~2s） | 慢（~15s，首次拉取镜像更久） |
| 数据清理 | schema.sql 的 CREATE-DROP | cleanup-test-data.sql 的 TRUNCATE CASCADE |
| 适用场景 | Swagger 测试、简单冒烟 | Controller CRUD 全链路验证 |

### 测试数据清理

每个测试方法执行前自动运行 `cleanup-test-data.sql`，按外键依赖顺序 TRUNCATE 所有业务表。
测试方法间完全隔离，无共享状态。

### 相关文件

| 文件 | 位置 | 用途 |
|------|------|------|
| TestcontainersConfiguration | `src/test/java/.../integration/TestcontainersConfiguration.java` | PG 容器配置 |
| BaseApiIntegrationTest | `src/test/java/.../integration/BaseApiIntegrationTest.java` | L3 测试基类 |
| application-integration.yml | `src/test/resources/application-integration.yml` | L3 profile 配置 |
| cleanup-test-data.sql | `src/test/resources/cleanup-test-data.sql` | 测试数据清理 |

### 运行 L3 集成测试

```bash
# 运行所有 L3 集成测试（使用 integration profile）
mvn test -pl backend/dataengine-api -Dspring.profiles.active=integration

# 运行指定模块的 L3 测试
mvn test -pl backend/dataengine-api -Dtest="*ApiIntegrationTest"

# 运行单个测试类
mvn test -pl backend/dataengine-api -Dtest="QualityRuleApiIntegrationTest"

# 使用 Failsafe 插件（推荐用于 CI）
mvn verify -pl backend/dataengine-api -Dit.test="*ApiIntegrationTest"
```

> **注意**: Testcontainers 需要 Docker 环境。首次运行会拉取 `postgres:14` 镜像（约 150MB）。

---

## 🔗 相关文档

- [单元测试规范](./unit-testing.md) - 单元测试详细规范
- [覆盖率标准](./coverage-standards.md) - 覆盖率标准详细规范
- [测试数据管理](./test-data.md) - 测试数据管理详细规范

---

**最后更新**：2026-03-10