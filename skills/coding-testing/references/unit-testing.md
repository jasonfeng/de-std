# 单元测试规范

> 本规范定义单元测试的编写规范，确保测试质量和覆盖率。

---

## 📋 目录

- [测试框架](#测试框架)
- [测试类结构](#测试类结构)
- [测试方法规范](#测试方法规范)
- [Mock使用](#mock使用)
- [断言规范](#断言规范)
- [测试场景](#测试场景)

---

## 测试框架

### 核心框架

| 框架 | 版本 | 用途 |
|------|------|------|
| **JUnit 5** | 5.x | 测试框架 |
| **Mockito** | 最新 | Mock框架 |
| **AssertJ** | 最新 | 断言库 |

### 依赖配置

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

    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 测试类结构

### 基本结构

```java
@ExtendWith(MockitoExtension.class)
class UserDomainServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserDomainService userDomainService;

    @Test
    @DisplayName("测试方法")
    void testMethod() {
        // 测试逻辑
    }
}
```

### 注解说明

| 注解 | 说明 | 使用场景 |
|------|------|---------|
| `@ExtendWith` | 指定测试运行器 | 必填，使用MockitoExtension |
| `@Mock` | 创建Mock对象 | 模拟依赖 |
| `@InjectMocks` | 自动注入Mock对象 | 被测类 |
| `@Test` | 标记测试方法 | 标记测试方法 |
| `@DisplayName` | 测试描述 | 必填，描述测试场景 |
| `@BeforeEach` | 每个测试前执行 | 初始化测试数据 |
| `@AfterEach` | 每个测试后执行 | 清理测试数据 |
| `@ParameterizedTest` | 参数化测试 | 多数据源测试 |
| `@TestInstance` | 测试实例生命周期 | PER_CLASS/PER_METHOD |

---

## 测试方法规范

### Given-When-Then模式

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
    assertThat(result.getUsername()).isEqualTo("admin");
}
```

### 测试方法命名

**格式**：`{方法名}_{场景}_{预期结果}`

| 方法 | 测试方法名 | 说明 |
|------|-----------|------|
| `createUser` | `createUser_Success` | 创建用户成功 |
| `createUser` | `createUser_DuplicateUsername_Exception` | 用户名重复抛异常 |
| `getUserById` | `getUserById_NotFound_Null` | 用户不存在返回null |
| `getUserById` | `getUserById_Deleted_Exception` | 用户已删除抛异常 |

### @DisplayName

**必填**，使用中文描述测试场景：

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

```java
@ExtendWith(MockitoExtension.class)
class UserDomainServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserDomainService userDomainService;

    @Test
    void test() {
        // userRepository是Mock对象
        when(userRepository.selectById(1L))
            .thenReturn(mockUser);

        UserEntity user = userRepository.selectById(1L);
        assertThat(user.getUsername()).isEqualTo("admin");
    }
}
```

### when().thenReturn()

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

```java
// 抛出异常
when(userRepository.existsByUsername("admin"))
    .thenReturn(true);

when(userRepository.insert(any(UserEntity.class)))
    .thenThrow(new DataIntegrityViolationException("Duplicate key"));
```

### ArgumentMatchers

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
when(userRepository.selectList(argThat(list -> list != null && list.isEmpty())))
    .thenReturn(Collections.emptyList());
```

### verify验证

```java
// 验证方法被调用一次
verify(userRepository, times(1)).insert(any(UserEntity.class));

// 验证方法从未被调用
verify(userRepository, never()).delete(anyLong());

// 验证方法被调用多次
verify(userRepository, atLeast(2)).selectById(anyLong());

// 验证具体参数
verify(userRepository).selectById(1L);
```

---

## 断言规范

### AssertJ断言

```java
import static org.assertj.core.api.Assertions.assertThat;

// 基本断言
assertThat(actual).isEqualTo(expected);
assertThat(actual).isNotNull();
assertThat(actual).isNull();
assertThat(actual).isTrue();
assertThat(actual).isFalse();

// 字符串断言
assertThat(actual).isEqualTo("Hello");
assertThat(actual).startsWith("H");
assertThat(actual).endsWith("o");
assertThat(actual).hasSize(5);

// 数字断言
assertThat(actual).isGreaterThan(10);
assertThat(actual).isLessThan(100);
assertThat(actual).isBetween(10, 100);

// 集合断言
assertThat(users).hasSize(2);
assertThat(users).extracting("username").contains("admin");
assertThat(users).extracting("id").containsExactly(1L, 2L);

// 异常断言
assertThatThrownBy(() -> userDomainService.createUser(null))
    .isInstanceOf(BusinessException.class)
    .hasMessage("用户名不能为空");
```

### JSON断言

```java
import org.json.JSONObject;
import static org.skyscreamer.jsonassertions.JsonAssertions.assertThatJson;

String json = "{ \"id\": 1, \"name\": \"John\" }";

assertThatJson(json).isEqualTo("{ \"id\": 1, \"name\": \"John\" }");
assertThatJson(json).isNumber("id", 1);
assertThatJson(json).isString("name", "John");
```

---

## 测试场景

### 1. 正常流程

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

### 2. 异常场景

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

### 3. 边界条件

```java
@Test
@DisplayName("创建用户 - 用户名最小长度 - 成功")
void createUser_UsernameMinLength_Success() {
    // Given
    UserDTO dto = new UserDTO();
    dto.setUsername("ab");  // 最小长度3

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
    assertThat(result.getUsername()).hasSize(2);
}
```

### 4. 列表操作

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

### 5. 组合操作

```java
@Test
@DisplayName("批量创建用户 - 成功")
void createUsers_Success() {
    // Given
    List<UserDTO> dtos = Arrays.asList(
        createDTO("user1"),
        createDTO("user2")
    );

    when(userRepository.insertBatch(any()))
        .thenReturn(2);

    // When
    int count = userDomainService.createUsers(dtos);

    // Then
    assertThat(count).isEqualTo(2);
    verify(userRepository, times(1)).insertBatch(any());
}
```

---

## 🔗 相关文档

- [TDD 开发工作流](./tdd-workflow.md) - **什么时候写测试**（开发流程约束）
- [集成测试规范](./integration-testing.md) - 集成测试详细规范
- [覆盖率标准](./coverage-standards.md) - 覆盖率标准详细规范
- [测试数据管理](./test-data.md) - 测试数据管理详细规范

---

**最后更新**：2026-03-10