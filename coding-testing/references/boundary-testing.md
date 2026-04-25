# 边界测试规范

> 本规范定义边界测试的编写标准和方法，确保系统在各种边界条件和极端场景下的稳定性。

---

## 📋 目录

- [边界测试定义](#边界测试定义)
- [边界测试分类](#边界测试分类)
- [通用边界测试清单](#通用边界测试清单)
- [边界测试模板](#边界测试模板)
- [边界测试审查标准](#边界测试审查标准)

---

## 边界测试定义

### 什么是边界测试

边界测试是针对系统在**极端条件**、**临界值**、**异常输入**等边界情况下的测试，旨在发现系统在正常测试中难以暴露的问题。

### 边界测试的重要性

| 重要性 | 说明 |
|--------|------|
| **发现隐藏Bug** | 正常测试无法覆盖的极端场景 |
| **提升系统健壮性** | 确保系统在各种输入下稳定运行 |
| **防止生产事故** | 提前发现边界条件导致的问题 |
| **增强用户信心** | 确保系统质量 |

### 边界测试与普通测试的区别

| 维度 | 普通测试 | 边界测试 |
|------|---------|---------|
| **测试目标** | 正常业务流程 | 极端与异常场景 |
| **输入数据** | 合法、有效数据 | 极端、边界、异常数据 |
| **测试范围** | 覆盖主要逻辑 | 覆盖边界条件 |
| **测试方法** | 正向测试 | 正向+逆向+边界测试 |

---

## 边界测试分类

### 1. 空值边界测试

测试参数为null或空字符串的场景。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **参数为null** | 测试null参数处理 | `createUser(null)` |
| **参数为空字符串** | 测试空字符串处理 | `createUser("")` |
| **集合为空** | 测试空集合处理 | `findAll()` 返回 `Collections.emptyList()` |

#### 示例代码

```java
@Test
@DisplayName("创建用户 - 用户名为null应抛出异常")
void testCreateUser_NullUsername_ShouldFail() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername(null); // null
    user.setPassword("Test123456");

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(user))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("用户名不能为空");

    verify(userMapper, never()).insert(any(UserEntity.class));
}

@Test
@DisplayName("创建用户 - 用户名为空字符串应抛出异常")
void testCreateUser_EmptyUsername_ShouldFail() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername(""); // 空字符串
    user.setPassword("Test123456");

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(user))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("用户名不能为空");

    verify(userMapper, never()).insert(any(UserEntity.class));
}
```

---

### 2. 长度边界测试

测试参数的最小长度、最大长度以及超出范围的场景。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **最小长度** | 测试最小有效长度 | `username = "a"` |
| **最大长度** | 测试最大有效长度 | `username = "a".repeat(50)` |
| **超出长度** | 测试超出最大长度 | `username = "a".repeat(51)` |
| **边界长度** | 测试边界值 | `username = "a".repeat(49)` |

#### 示例代码

```java
@Test
@DisplayName("创建用户 - 用户名长度为1个字符")
void testCreateUser_UsernameLength1() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("a"); // 最小长度
    user.setPassword("Test123456");

    when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).hasSize(1);
}

@Test
@DisplayName("创建用户 - 用户名长度为50个字符")
void testCreateUser_UsernameLength50() {
    // Given
    String longUsername = "a".repeat(50); // 最大长度
    UserEntity user = new UserEntity();
    user.setUsername(longUsername);
    user.setPassword("Test123456");

    when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).hasSize(50);
}

@Test
@DisplayName("创建用户 - 用户名长度超过50个字符应抛出异常")
void testCreateUser_UsernameLength51_ShouldFail() {
    // Given
    String longUsername = "a".repeat(51); // 超出最大长度
    UserEntity user = new UserEntity();
    user.setUsername(longUsername);
    user.setPassword("Test123456");

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(user))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("用户名长度不能超过50个字符");

    verify(userMapper, never()).insert(any(UserEntity.class));
}
```

---

### 3. 数值边界测试

测试数值类型的边界值，包括0、负数、极值等。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **0值** | 测试0值处理 | `id = 0` |
| **负数** | 测试负数处理 | `id = -1` |
| **最小值** | 测试Integer.MIN_VALUE | `id = Integer.MIN_VALUE` |
| **最大值** | 测试Integer.MAX_VALUE | `id = Integer.MAX_VALUE` |

#### 示例代码

```java
@Test
@DisplayName("查询用户 - ID为0应抛出异常")
void testGetUserById_IdZero() {
    // Given
    when(userMapper.selectById(0L)).thenReturn(null);

    // When & Then
    assertThatThrownBy(() -> userDomainService.getUserById(0L))
        .isInstanceOf(UserNotFoundException.class)
        .hasMessage("用户不存在");
}

@Test
@DisplayName("查询用户 - ID为负数应抛出异常")
void testGetUserById_NegativeId() {
    // Given
    when(userMapper.selectById(-1L)).thenReturn(null);

    // When & Then
    assertThatThrownBy(() -> userDomainService.getUserById(-1L))
        .isInstanceOf(UserNotFoundException.class)
        .hasMessage("用户不存在");
}

@Test
@DisplayName("查询用户 - ID为Long.MAX_VALUE")
void testGetUserById_IdMaxValue() {
    // Given
    when(userMapper.selectById(Long.MAX_VALUE)).thenReturn(null);

    // When & Then
    assertThatThrownBy(() -> userDomainService.getUserById(Long.MAX_VALUE))
        .isInstanceOf(UserNotFoundException.class)
        .hasMessage("用户不存在");
}
```

---

### 4. ID边界测试

测试ID类型的边界值，包括0、负数、极值等。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **ID为0** | 测试ID为0 | `id = 0` |
| **ID为负数** | 测试ID为负数 | `id = -1` |
| **ID为Long.MIN_VALUE** | 测试最小Long值 | `id = Long.MIN_VALUE` |
| **ID为Long.MAX_VALUE** | 测试最大Long值 | `id = Long.MAX_VALUE` |

#### 示例代码

```java
@Test
@DisplayName("删除用户 - ID为0应抛出异常")
void testDeleteUser_IdZero() {
    // Given
    when(userMapper.selectById(0L)).thenReturn(null);

    // When & Then
    assertThatThrownBy(() -> userDomainService.deleteUser(0L))
        .isInstanceOf(UserNotFoundException.class)
        .hasMessage("用户不存在");

    verify(userMapper, never()).deleteById(anyLong());
}

@Test
@DisplayName("删除用户 - ID为负数应抛出异常")
void testDeleteUser_NegativeId() {
    // Given
    when(userMapper.selectById(-1L)).thenReturn(null);

    // When & Then
    assertThatThrownBy(() -> userDomainService.deleteUser(-1L))
        .isInstanceOf(UserNotFoundException.class)
        .hasMessage("用户不存在");

    verify(userMapper, never()).deleteById(anyLong());
}
```

---

### 5. 状态边界测试

测试枚举类型的所有状态值以及状态切换场景。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **所有枚举值** | 测试所有状态值 | 测试所有 `UserStatusEnum` |
| **状态切换** | 测试状态转换 | `ACTIVE` → `DISABLED` |
| **无效状态** | 测试无效状态 | 切换到无效状态 |

#### 示例代码

```java
@Test
@DisplayName("创建用户 - 测试所有状态值")
void testCreateUser_AllStatusValues() {
    // 测试所有枚举状态值
    UserStatusEnum[] statuses = UserStatusEnum.values();

    for (UserStatusEnum status : statuses) {
        // Given
        UserEntity user = new UserEntity();
        user.setUsername("testuser_" + status.name());
        user.setPassword("Test123456");
        user.setStatus(status);

        when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

        // When
        UserEntity result = userDomainService.createUser(user);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getStatus()).isEqualTo(status);
    }
}

@Test
@DisplayName("修改用户状态 - 循环切换状态")
void testChangeUserStatus_CycleThroughAllStatuses() {
    // Given
    UserEntity user = new UserEntity();
    user.setId(1L);
    user.setUsername("testuser");
    user.setStatus(UserStatusEnum.ACTIVE);

    when(userMapper.selectById(1L)).thenReturn(user);
    when(userMapper.updateById(any(UserEntity.class))).thenReturn(1);

    // When & Then - 循环切换状态
    UserStatusEnum[] statuses = UserStatusEnum.values();
    for (int i = 0; i < statuses.length; i++) {
        UserStatusEnum newStatus = statuses[(i + 1) % statuses.length];
        userDomainService.changeUserStatus(1L, newStatus);
        user.setStatus(newStatus);
    }

    verify(userMapper, times(statuses.length)).updateById(any(UserEntity.class));
}
```

---

### 6. 特殊字符测试

测试参数包含特殊字符、Unicode字符的场景。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **特殊字符** | 测试特殊字符 | `username = "test_user-123@example.com"` |
| **Unicode字符** | 测试Unicode支持 | `username = "测试用户"` |
| **SQL注入** | 测试SQL注入防御 | `username = "' OR '1'='1"` |
| **XSS攻击** | 测试XSS防御 | `username = "<script>alert(1)</script>"` |

#### 示例代码

```java
@Test
@DisplayName("创建用户 - 用户名包含特殊字符")
void testCreateUser_UsernameWithSpecialChars() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("test_user-123@example.com");
    user.setPassword("Test123456");

    when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo("test_user-123@example.com");
}

@Test
@DisplayName("创建用户 - 用户名包含Unicode字符")
void testCreateUser_UsernameWithUnicode() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("测试用户");
    user.setPassword("Test123456");

    when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo("测试用户");
}

@Test
@DisplayName("创建用户 - 密码包含特殊字符")
void testCreateUser_PasswordWithSpecialChars() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("testuser");
    user.setPassword("P@ssw0rd!#$%");

    when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getPassword()).isNotEqualTo("P@ssw0rd!#$%"); // 密码已加密
}
```

---

### 7. 并发边界测试

测试并发场景下的边界条件，如并发创建、并发更新等。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **并发创建** | 测试并发创建相同数据 | 多线程创建相同用户名 |
| **并发更新** | 测试并发更新同一数据 | 多线程更新同一用户 |
| **并发删除** | 测试并发删除同一数据 | 多线程删除同一用户 |

#### 示例代码

```java
@Test
@DisplayName("创建用户 - 并发创建相同用户名")
void testCreateUser_ConcurrentDuplicateUsername() throws InterruptedException {
    // Given
    String username = "concurrent_user";
    when(userMapper.existsByUsername(username)).thenReturn(false, true); // 第一次不存在，第二次存在
    when(userMapper.insert(any(UserEntity.class))).thenReturn(1L);

    // When - 并发创建
    CountDownLatch latch = new CountDownLatch(2);
    AtomicReference<Exception> exception = new AtomicReference<>();

    new Thread(() -> {
        try {
            UserEntity user = new UserEntity();
            user.setUsername(username);
            user.setPassword("Test123456");
            userDomainService.createUser(user);
        } catch (Exception e) {
            exception.set(e);
        } finally {
            latch.countDown();
        }
    }).start();

    new Thread(() -> {
        try {
            UserEntity user = new UserEntity();
            user.setUsername(username);
            user.setPassword("Test123456");
            userDomainService.createUser(user);
        } catch (Exception e) {
            exception.set(e);
        } finally {
            latch.countDown();
        }
    }).start();

    latch.await();

    // Then - 第二次创建应该失败
    assertThat(exception.get()).isNotNull();
    assertThat(exception.get()).isInstanceOf(UserAlreadyExistsException.class);
}
```

---

### 8. 性能边界测试

测试大数据量、批量操作等性能边界场景。

#### 测试要点

| 要点 | 说明 | 示例 |
|------|------|------|
| **大数据量** | 测试大数据量处理 | 查询10000条数据 |
| **批量操作** | 测试批量操作性能 | 批量创建1000个用户 |
| **超时场景** | 测试超时处理 | 操作超过超时时间 |

#### 示例代码

```java
@Test
@DisplayName("批量创建用户 - 大批量数据")
void testBatchCreateUsers_LargeBatch() {
    // Given
    int batchSize = 1000;
    List<UserEntity> users = new ArrayList<>();
    for (int i = 0; i < batchSize; i++) {
        UserEntity user = new UserEntity();
        user.setUsername("user_" + i);
        user.setPassword("Test123456");
        users.add(user);
    }

    when(userMapper.insertBatch(anyList())).thenReturn(batchSize);

    // When
    long startTime = System.currentTimeMillis();
    int result = userDomainService.batchCreateUsers(users);
    long endTime = System.currentTimeMillis();

    // Then
    assertThat(result).isEqualTo(batchSize);
    assertThat(endTime - startTime).isLessThan(5000); // 应该在5秒内完成
}
```

---

## 通用边界测试清单（v2.1 风险驱动）

> 边界测试数量不再固定为 10 个，而是根据方法的风险等级动态确定。
> 详见 [项目测试策略 - 边界测试强制性要求](./project-testing-strategy.md)

### 风险等级与边界测试要求

| 风险等级 | 判定条件 | 边界测试数量 | 必测边界点 |
|---------|---------|------------|-----------|
| **P0-关键** | 涉及数据安全/权限/资金 | 8+ | null, 空字符串, 长度, 特殊字符, 唯一性, 状态, 并发, SQL注入 |
| **P1-高** | 多分支业务逻辑、跨服务调用 | 5-8 | null, 空字符串, 长度, 唯一性, 状态 |
| **P2-中** | 单一职责 CRUD | 3-5 | null, 空字符串, ID边界 |
| **P3-低** | 纯查询、无状态转换 | 1-3 | null, 空字符串 |

### 风险评级决策树

```
方法是否涉及数据安全或权限？
  ├─ 是 → P0-关键
  └─ 否 → 方法是否有3个以上分支？
              ├─ 是 → P1-高
              └─ 否 → 方法是否修改数据？
                          ├─ 是 → P2-中
                          └─ 否 → P3-低
```

### 边界点按风险等级选择

| # | 边界类型 | P0 | P1 | P2 | P3 |
|---|---------|----|----|----|----|
| 1 | **参数为null** | 必须 | 必须 | 必须 | 必须 |
| 2 | **参数为空字符串** | 必须 | 必须 | 必须 | 必须 |
| 3 | **参数为最小/最大长度** | 必须 | 必须 | 推荐 | - |
| 4 | **参数为0或负数** | 必须 | 必须 | 必须 | 推荐 |
| 5 | **参数包含特殊字符** | 必须 | 推荐 | - | - |
| 6 | **参数包含Unicode字符** | 推荐 | 推荐 | - | - |
| 7 | **参数为Long极值** | 必须 | 必须 | 推荐 | - |
| 8 | **参数为重复值（唯一性）** | 必须 | 必须 | 推荐 | - |
| 9 | **状态/枚举转换** | 必须 | 必须 | 推荐 | - |
| 10 | **并发/竞态** | 必须 | 推荐 | - | - |
| 11 | **SQL注入/XSS防御** | 必须 | 推荐 | - | - |

---

## 边界测试模板

### 边界测试类模板

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("{Service}领域服务 - 边界测试")
class {Service}BoundaryTest {

    @Mock
    private {Repository} {repository};

    @InjectMocks
    private {Service} {service};

    // ========== 空值/Null 边界测试 ==========

    @Test
    @DisplayName("{Method} - {参数}为null应抛出异常")
    void test{Method}_{Parameter}Null_ShouldFail() {
        // Given
        {Entity} entity = new {Entity}();
        entity.set{Field}(null); // null

        // When & Then
        assertThatThrownBy(() -> {service}.{Method}(entity))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("{错误消息}");

        verify({repository}, never()).{RepositoryMethod}(any());
    }

    @Test
    @DisplayName("{Method} - {参数}为空字符串应抛出异常")
    void test{Method}_{Parameter}Empty_ShouldFail() {
        // Given
        {Entity} entity = new {Entity}();
        entity.set{Field}(""); // 空字符串

        // When & Then
        assertThatThrownBy(() -> {service}.{Method}(entity))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("{错误消息}");

        verify({repository}, never()).{RepositoryMethod}(any());
    }

    // ========== 长度边界测试 ==========

    @Test
    @DisplayName("{Method} - {参数}长度为1个字符")
    void test{Method}_{Parameter}Length1() {
        // Given
        {Entity} entity = new {Entity}();
        entity.set{Field}("a"); // 最小长度

        when({repository}.{RepositoryMethod}(any())).thenReturn(1L);

        // When
        {Entity} result = {service}.{Method}(entity);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.get{Field}()).hasSize(1);
    }

    @Test
    @DisplayName("{Method} - {参数}长度为{最大长度}个字符")
    void test{Method}_{Parameter}Length{Max}() {
        // Given
        String long{Field} = "a".repeat({Max}); // 最大长度
        {Entity} entity = new {Entity}();
        entity.set{Field}(long{Field});

        when({repository}.{RepositoryMethod}(any())).thenReturn(1L);

        // When
        {Entity} result = {service}.{Method}(entity);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.get{Field}()).hasSize({Max});
    }

    // ========== 特殊字符测试 ==========

    @Test
    @DisplayName("{Method} - {参数}包含特殊字符")
    void test{Method}_{Parameter}WithSpecialChars() {
        // Given
        {Entity} entity = new {Entity}();
        entity.set{Field}("test_{field}-123@example.com");

        when({repository}.{RepositoryMethod}(any())).thenReturn(1L);

        // When
        {Entity} result = {service}.{Method}(entity);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.get{Field}()).isEqualTo("test_{field}-123@example.com");
    }

    @Test
    @DisplayName("{Method} - {参数}包含Unicode字符")
    void test{Method}_{Parameter}WithUnicode() {
        // Given
        {Entity} entity = new {Entity}();
        entity.set{Field}("测试{Field}");

        when({repository}.{RepositoryMethod}(any())).thenReturn(1L);

        // When
        {Entity} result = {service}.{Method}(entity);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.get{Field}()).isEqualTo("测试{Field}");
    }

    // ========== ID 边界测试 ==========

    @Test
    @DisplayName("{Method} - ID为0应抛出异常")
    void test{Method}_IdZero() {
        // Given
        when({repository}.selectById(0L)).thenReturn(null);

        // When & Then
        assertThatThrownBy(() -> {service}.{Method}(0L))
            .isInstanceOf({Exception}.class)
            .hasMessage("{错误消息}");
    }

    @Test
    @DisplayName("{Method} - ID为负数应抛出异常")
    void test{Method}_NegativeId() {
        // Given
        when({repository}.selectById(-1L)).thenReturn(null);

        // When & Then
        assertThatThrownBy(() -> {service}.{Method}(-1L))
            .isInstanceOf({Exception}.class)
            .hasMessage("{错误消息}");
    }

    @Test
    @DisplayName("{Method} - ID为Long.MAX_VALUE")
    void test{Method}_IdMaxValue() {
        // Given
        when({repository}.selectById(Long.MAX_VALUE)).thenReturn(null);

        // When & Then
        assertThatThrownBy(() -> {service}.{Method}(Long.MAX_VALUE))
            .isInstanceOf({Exception}.class)
            .hasMessage("{错误消息}");
    }

    // ========== 状态边界测试 ==========

    @Test
    @DisplayName("{Method} - 测试所有状态值")
    void test{Method}_AllStatusValues() {
        // 测试所有枚举状态值
        {StatusEnum}[] statuses = {StatusEnum}.values();

        for ({StatusEnum} status : statuses) {
            // Given
            {Entity} entity = new {Entity}();
            entity.set{Field}("test_{field}_" + status.name());
            entity.setStatus(status);

            when({repository}.{RepositoryMethod}(any())).thenReturn(1L);

            // When
            {Entity} result = {service}.{Method}(entity);

            // Then
            assertThat(result).isNotNull();
            assertThat(result.getStatus()).isEqualTo(status);
        }
    }
}
```

---

## 边界测试审查标准（v2.1）

### PR审查检查清单

| 检查项 | 说明 | 结果 |
|--------|------|------|
| **1. 是否有边界测试** | 每个DomainService必须有边界测试（可独立类或在主测试类的 @Nested 分组） | ☐ |
| **2. 风险等级是否合理** | 方法风险评级是否符合决策树 | ☐ |
| **3. 边界测试数量与风险匹配** | P0 ≥8, P1 5-8, P2 3-5, P3 1-3 | ☐ |
| **4. 必测边界点是否覆盖** | 根据风险等级的必测清单逐一核对 | ☐ |
| **5. 测试命名规范** | `@DisplayName` 中文描述，方法名语义清晰 | ☐ |

### 代码提交前检查

| 检查项 | 说明 |
|--------|------|
| **1. 方法风险评级** | 按决策树确定每个公共方法的风险等级 |
| **2. 边界测试数量达标** | P0 ≥8, P1 ≥5, P2 ≥3, P3 ≥1 |
| **3. 测试全部通过** | `mvn test` 无失败 |
| **4. 覆盖率达标** | DomainService ≥80% |

---

## 🔗 相关文档

- [项目测试策略](./project-testing-strategy.md) - 项目整体测试策略
- [测试用例数量指南](./test-quantity-guide.md) - 测试用例数量计算指南
- [测试检查清单](./test-checklist.md) - 测试质量检查清单
- [单元测试规范](./unit-testing.md) - 单元测试编写规范
- [覆盖率标准](./coverage-standards.md) - 覆盖率验证标准

---

**最后更新**：2026-03-12