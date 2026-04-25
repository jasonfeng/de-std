# 测试用例数量指南

> 本规范定义测试用例数量的计算方法和按复杂度的用例要求，确保测试覆盖率达标。

---

## 📋 目录

- [测试用例数量计算方法](#测试用例数量计算方法)
- [按复杂度的用例要求](#按复杂度的用例要求)
- [测试用例类型分布](#测试用例类型分布)
- [测试用例数量示例](#测试用例数量示例)
- [测试用例数量验证](#测试用例数量验证)

---

## 测试用例数量计算方法

### 基本公式

```
测试用例数 = 方法数 × 3 + 分支数 × 2 + 边界数 × 1
```

**说明**：

- **方法数 × 3**：每个方法至少3个测试用例（正常流程、异常流程、边界条件）
- **分支数 × 2**：每个分支至少2个测试用例（分支true、分支false）
- **边界数 × 1**：每个边界点至少1个测试用例

### 详细计算步骤

#### 步骤1：统计方法数

```java
public class UserDomainService {

    public Long createUser(UserEntity entity) { /* ... */ }     // 方法1
    public UserEntity getUserById(Long id) { /* ... */ }        // 方法2
    public void updateUser(UserEntity entity) { /* ... */ }      // 方法3
    public void deleteUser(Long id) { /* ... */ }               // 方法4
    public List<UserEntity> findAllUsers() { /* ... */ }        // 方法5
}
```

方法数 = 5

#### 步骤2：统计分支数

```java
public Long createUser(UserEntity entity) {
    if (entity == null) {                                        // 分支1
        throw new IllegalArgumentException("用户不能为空");
    }
    if (entity.getUsername() == null || entity.getUsername().isEmpty()) { // 分支2
        throw new IllegalArgumentException("用户名不能为空");
    }
    if (userRepository.existsByUsername(entity.getUsername())) {          // 分支3
        throw new UserAlreadyExistsException("用户名已存在");
    }
    // ... 正常流程
}
```

分支数 = 3

#### 步骤3：统计边界数

| 边界类型 | 数量 |
|---------|------|
| null参数 | 1 |
| 空字符串 | 1 |
| 最小长度 | 1 |
| 最大长度 | 1 |
| 特殊字符 | 1 |
| Unicode字符 | 1 |
| 0值 | 1 |
| 负数 | 1 |
| 极值 | 1 |
| 重复值 | 1 |

边界数 = 10

#### 步骤4：计算测试用例数

```
测试用例数 = 5 × 3 + 3 × 2 + 10 × 1 = 15 + 6 + 10 = 31个测试用例
```

---

## 按复杂度的用例要求

### 复杂度分级

| 复杂度 | 说明 | 圈复杂度 | 示例 |
|--------|------|---------|------|
| **简单方法** | 无分支、无边界 | 1-2 | 简单CRUD、getter/setter |
| **中等方法** | 有分支、有边界 | 3-5 | 带条件判断的CRUD |
| **复杂方法** | 多分支、多边界 | 6-10 | 复杂业务逻辑 |
| **核心方法** | 核心业务逻辑 | >10 | 关键业务方法 |

### 按复杂度的最少用例数

| 复杂度 | 最少用例数 | 正常流程 | 异常流程 | 边界条件 |
|--------|-----------|---------|---------|---------|
| **简单方法** | 5个 | 2个 | 1个 | 2个 |
| **中等方法** | 8个 | 2个 | 2个 | 4个 |
| **复杂方法** | 12个 | 3个 | 3个 | 6个 |
| **核心方法** | 15个 | 4个 | 4个 | 7个 |

### 简单方法示例

```java
// 简单方法 - 无分支、无边界
public UserEntity getUserById(Long id) {
    return userRepository.selectById(id);
}
```

**测试用例**（最少5个）：

```java
@Test
@DisplayName("查询用户 - 成功")
void getUserById_Success() { /* ... */ }

@Test
@DisplayName("查询用户 - 用户不存在")
void getUserById_NotFound() { /* ... */ }

@Test
@DisplayName("查询用户 - ID为null")
void getUserById_NullId() { /* ... */ }

@Test
@DisplayName("查询用户 - ID为0")
void getUserById_IdZero() { /* ... */ }

@Test
@DisplayName("查询用户 - ID为负数")
void getUserById_NegativeId() { /* ... */ }
```

### 中等方法示例

```java
// 中等方法 - 有分支、有边界
public Long createUser(UserEntity entity) {
    if (entity == null) {
        throw new IllegalArgumentException("用户不能为空");
    }
    if (entity.getUsername() == null || entity.getUsername().isEmpty()) {
        throw new IllegalArgumentException("用户名不能为空");
    }
    if (userRepository.existsByUsername(entity.getUsername())) {
        throw new UserAlreadyExistsException("用户名已存在");
    }
    entity.setId(snowflakeIdGenerator.nextId());
    return userRepository.insert(entity);
}
```

**测试用例**（最少8个）：

```java
// 正常流程 (2个)
@Test
@DisplayName("创建用户 - 成功")
void createUser_Success() { /* ... */ }

// 异常流程 (2个)
@Test
@DisplayName("创建用户 - 用户为null应抛出异常")
void createUser_NullUser_Exception() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名为空应抛出异常")
void createUser_EmptyUsername_Exception() { /* ... */ }

// 边界条件 (4个)
@Test
@DisplayName("创建用户 - 用户名长度为1个字符")
void createUser_UsernameLength1() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名长度为50个字符")
void createUser_UsernameLength50() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名包含特殊字符")
void createUser_UsernameWithSpecialChars() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名包含Unicode字符")
void createUser_UsernameWithUnicode() { /* ... */ }
```

### 复杂方法示例

```java
// 复杂方法 - 多分支、多边界
public void assignRolesToUser(Long userId, List<Long> roleIds) {
    if (userId == null) {
        throw new IllegalArgumentException("用户ID不能为空");
    }
    if (roleIds == null || roleIds.isEmpty()) {
        throw new IllegalArgumentException("角色ID列表不能为空");
    }

    UserEntity user = userRepository.selectById(userId);
    if (user == null) {
        throw new UserNotFoundException("用户不存在");
    }

    List<Long> existingRoleIds = userRoleRepository.selectRoleIdsByUserId(userId);
    List<Long> newRoleIds = roleIds.stream()
        .filter(roleId -> !existingRoleIds.contains(roleId))
        .collect(Collectors.toList());

    if (newRoleIds.isEmpty()) {
        throw new BusinessException("所有角色已分配");
    }

    for (Long roleId : newRoleIds) {
        RoleEntity role = roleRepository.selectById(roleId);
        if (role == null) {
            throw new RoleNotFoundException("角色不存在: " + roleId);
        }
        UserRoleEntity userRole = new UserRoleEntity();
        userRole.setUserId(userId);
        userRole.setRoleId(roleId);
        userRoleRepository.insert(userRole);
    }
}
```

**测试用例**（最少12个）：

```java
// 正常流程 (3个)
@Test
@DisplayName("分配角色 - 成功")
void assignRolesToUser_Success() { /* ... */ }

@Test
@DisplayName("分配角色 - 部分角色已存在")
void assignRolesToUser_SomeRolesExist() { /* ... */ }

// 异常流程 (3个)
@Test
@DisplayName("分配角色 - 用户ID为null应抛出异常")
void assignRolesToUser_NullUserId_Exception() { /* ... */ }

@Test
@DisplayName("分配角色 - 角色ID列表为空应抛出异常")
void assignRolesToUser_EmptyRoleIds_Exception() { /* ... */ }

@Test
@DisplayName("分配角色 - 用户不存在应抛出异常")
void assignRolesToUser_UserNotFound_Exception() { /* ... */ }

// 边界条件 (6个)
@Test
@DisplayName("分配角色 - 分配1个角色")
void assignRolesToUser_SingleRole() { /* ... */ }

@Test
@DisplayName("分配角色 - 分配100个角色")
void assignRolesToUser_ManyRoles() { /* ... */ }

@Test
@DisplayName("分配角色 - 所有角色已存在")
void assignRolesToUser_AllRolesExist() { /* ... */ }

@Test
@DisplayName("分配角色 - 角色不存在应抛出异常")
void assignRolesToUser_RoleNotFound_Exception() { /* ... */ }

@Test
@DisplayName("分配角色 - 角色ID为0")
void assignRolesToUser_RoleIdZero() { /* ... */ }

@Test
@DisplayName("分配角色 - 角色ID为负数")
void assignRolesToUser_RoleIdNegative() { /* ... */ }
```

---

## 测试用例类型分布

### 标准分布

| 类型 | 占比 | 说明 |
|------|------|------|
| **正常流程** | 30% | 正常业务场景 |
| **异常流程** | 30% | 异常处理场景 |
| **边界条件** | 40% | 边界与极端场景 |

### 正常流程测试

**目的**：验证系统在正常业务场景下的行为。

**示例**：

```java
@Test
@DisplayName("创建用户 - 成功")
void createUser_Success() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("admin");
    user.setPassword("Test123456");

    when(userRepository.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo("admin");
}
```

### 异常流程测试

**目的**：验证系统在异常情况下的处理。

**示例**：

```java
@Test
@DisplayName("创建用户 - 用户名为空应抛出异常")
void createUser_EmptyUsername_Exception() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("");
    user.setPassword("Test123456");

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(user))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("用户名不能为空");

    verify(userRepository, never()).insert(any(UserEntity.class));
}

@Test
@DisplayName("创建用户 - 用户名已存在应抛出异常")
void createUser_DuplicateUsername_Exception() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("admin");
    user.setPassword("Test123456");

    when(userRepository.existsByUsername("admin")).thenReturn(true);

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(user))
        .isInstanceOf(UserAlreadyExistsException.class)
        .hasMessage("用户名已存在");

    verify(userRepository, never()).insert(any(UserEntity.class));
}
```

### 边界条件测试

**目的**：验证系统在边界条件下的行为。

**示例**：

```java
@Test
@DisplayName("创建用户 - 用户名长度为1个字符")
void createUser_UsernameLength1() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("a");
    user.setPassword("Test123456");

    when(userRepository.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).hasSize(1);
}

@Test
@DisplayName("创建用户 - 用户名长度为50个字符")
void createUser_UsernameLength50() {
    // Given
    String longUsername = "a".repeat(50);
    UserEntity user = new UserEntity();
    user.setUsername(longUsername);
    user.setPassword("Test123456");

    when(userRepository.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).hasSize(50);
}

@Test
@DisplayName("创建用户 - 用户名包含特殊字符")
void createUser_UsernameWithSpecialChars() {
    // Given
    UserEntity user = new UserEntity();
    user.setUsername("test_user-123@example.com");
    user.setPassword("Test123456");

    when(userRepository.insert(any(UserEntity.class))).thenReturn(1L);

    // When
    UserEntity result = userDomainService.createUser(user);

    // Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo("test_user-123@example.com");
}
```

---

## 测试用例数量示例

### UserDomainService

| 方法 | 复杂度 | 最少用例数 | 正常 | 异常 | 边界 |
|------|--------|-----------|------|------|------|
| `createUser` | 中等 | 8个 | 2个 | 2个 | 4个 |
| `getUserById` | 简单 | 5个 | 2个 | 1个 | 2个 |
| `updateUser` | 中等 | 8个 | 2个 | 2个 | 4个 |
| `deleteUser` | 简单 | 5个 | 2个 | 1个 | 2个 |
| `findAllUsers` | 简单 | 5个 | 2个 | 1个 | 2个 |
| `assignRoles` | 复杂 | 12个 | 3个 | 3个 | 6个 |
| `changePassword` | 中等 | 8个 | 2个 | 2个 | 4个 |
| `disableUser` | 简单 | 5个 | 2个 | 1个 | 2个 |
| **总计** | - | **56个** | **17个** | **13个** | **26个** |

### QualityRuleDomainService

| 方法 | 复杂度 | 最少用例数 | 正常 | 异常 | 边界 |
|------|--------|-----------|------|------|------|
| `createRule` | 复杂 | 12个 | 3个 | 3个 | 6个 |
| `getRuleById` | 简单 | 5个 | 2个 | 1个 | 2个 |
| `updateRule` | 复杂 | 12个 | 3个 | 3个 | 6个 |
| `deleteRule` | 中等 | 8个 | 2个 | 2个 | 4个 |
| `executeRule` | 核心 | 15个 | 4个 | 4个 | 7个 |
| `findAllRules` | 简单 | 5个 | 2个 | 1个 | 2个 |
| `batchExecuteRules` | 核心 | 15个 | 4个 | 4个 | 7个 |
| `disableRule` | 中等 | 8个 | 2个 | 2个 | 4个 |
| **总计** | - | **80个** | **22个** | **20个** | **38个** |

---

## 测试用例数量验证

### 验证方法

#### 方法1：使用Maven命令

```bash
# 运行测试并统计测试用例数
mvn test -Dtest=UserDomainServiceTest

# 查看测试报告
cat target/surefire-reports/com.dp.dataengine.domain.service.user.UserDomainServiceTest.txt
```

#### 方法2：使用IDE

在IDEA中，右键点击测试类 → "Run {Test}" → 查看运行结果。

#### 方法3：使用代码覆盖率工具

```bash
# 运行测试并生成覆盖率报告
mvn clean test jacoco:report

# 查看HTML报告
open target/site/jacoco/index.html
```

### 验证标准

| 标准 | 要求 |
|------|------|
| **测试用例数量** | 达到最少用例数 |
| **测试类型分布** | 正常30%、异常30%、边界40% |
| **测试覆盖率** | DomainService ≥80% |
| **边界测试覆盖率** | ≥90% |

### 验证检查清单

| 检查项 | 说明 | 结果 |
|--------|------|------|
| **1. 测试用例数量达标** | 达到最少用例数 | ☐ 是 ☐ 否 |
| **2. 正常流程测试充足** | 正常流程≥30% | ☐ 是 ☐ 否 |
| **3. 异常流程测试充足** | 异常流程≥30% | ☐ 是 ☐ 否 |
| **4. 边界条件测试充足** | 边界条件≥40% | ☐ 是 ☐ 否 |
| **5. 覆盖率达标** | DomainService ≥80% | ☐ 是 ☐ 否 |

---

## 🔗 相关文档

- [项目测试策略](./project-testing-strategy.md) - 项目整体测试策略
- [边界测试规范](./boundary-testing.md) - 边界测试详细规范
- [测试检查清单](./test-checklist.md) - 测试质量检查清单
- [单元测试规范](./unit-testing.md) - 单元测试编写规范
- [覆盖率标准](./coverage-standards.md) - 覆盖率验证标准

---

**最后更新**：2026-03-12