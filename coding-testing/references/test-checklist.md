# 测试检查清单

> 本规范定义测试质量的检查清单，确保测试覆盖率和测试质量达标。

---

## 📋 目录

- [代码提交前检查](#代码提交前检查)
- [PR审查检查](#pr审查检查)
- [Story完成检查](#story完成检查)
- [发布前检查](#发布前检查)

---

## 代码提交前检查

开发者在提交代码前，需要自行检查以下内容。

### 检查清单

| # | 检查项 | 检查内容 | 结果 |
|---|--------|---------|------|
| **1** | **测试类存在** | 每个DomainService必须有对应的测试类 | ☐ 是 ☐ 否 |
| **2** | **边界测试类存在** | 每个DomainService必须有BoundaryTest类 | ☐ 是 ☐ 否 |
| **3** | **测试用例数量达标** | 测试用例数量达到最少要求 | ☐ 是 ☐ 否 |
| **4** | **测试类型分布** | 正常30%、异常30%、边界40% | ☐ 是 ☐ 否 |
| **5** | **所有测试通过** | 所有测试用例必须通过 | ☐ 是 ☐ 否 |

### 详细说明

#### 1. 测试类存在

**检查内容**：每个DomainService必须有对应的测试类。

**位置**：
```
src/test/java/com/dp/dataengine/domain/service/
├── user/
│   ├── UserDomainServiceTest.java          # 主测试类
│   └── UserDomainServiceBoundaryTest.java  # 边界测试类
├── role/
│   ├── RoleDomainServiceTest.java
│   └── RoleDomainServiceBoundaryTest.java
└── system/
    ├── DepartmentDomainServiceTest.java
    └── DepartmentDomainServiceBoundaryTest.java
```

**检查命令**：
```bash
# 检查是否有测试类
ls -la src/test/java/com/dp/dataengine/domain/service/
```

#### 2. 边界测试类存在

**检查内容**：每个DomainService必须有BoundaryTest类。

**检查命令**：
```bash
# 检查是否有边界测试类
find src/test/java -name "*BoundaryTest.java"
```

#### 3. 测试用例数量达标

**检查内容**：测试用例数量达到最少要求。

**最少用例数**：
- 简单方法：5个
- 中等方法：8个
- 复杂方法：12个
- 核心方法：15个

**检查命令**：
```bash
# 运行测试并统计测试用例数
mvn test -Dtest=UserDomainServiceTest

# 查看测试报告
cat target/surefire-reports/com.dp.dataengine.domain.service.user.UserDomainServiceTest.txt
```

#### 4. 测试类型分布

**检查内容**：测试类型分布符合标准。

**标准分布**：
- 正常流程：30%
- 异常流程：30%
- 边界条件：40%

**检查方法**：
- 查看测试方法的命名，统计各类型数量
- 确保分布符合标准

#### 5. 所有测试通过

**检查内容**：所有测试用例必须通过。

**检查命令**：
```bash
# 运行所有测试
mvn test

# 运行特定测试类
mvn test -Dtest=UserDomainServiceTest

# 运行特定测试方法
mvn test -Dtest=UserDomainServiceTest#createUser_Success
```

---

## PR审查检查

审查者在审查PR时，需要检查以下内容。

### 检查清单

| # | 检查项 | 检查内容 | 结果 |
|---|--------|---------|------|
| **1** | **测试类命名规范** | 测试类命名遵循 `{Service}Test` 格式 | ☐ 是 ☐ 否 |
| **2** | **边界测试类存在** | 边界测试类命名遵循 `{Service}BoundaryTest` 格式 | ☐ 是 ☐ 否 |
| **3** | **测试方法命名规范** | 测试方法命名遵循 `{Method}_{Scenario}_{Expected}` 格式 | ☐ 是 ☐ 否 |
| **4** | **测试有@DisplayName** | 每个测试必须有中文@DisplayName | ☐ 是 ☐ 否 |
| **5** | **测试覆盖null参数** | 每个方法必须测试null参数 | ☐ 是 ☐ 否 |

### 详细说明

#### 1. 测试类命名规范

**检查内容**：测试类命名遵循 `{Service}Test` 格式。

**示例**：
- `UserDomainServiceTest.java`
- `RoleDomainServiceTest.java`
- `DepartmentDomainServiceTest.java`

#### 2. 边界测试类存在

**检查内容**：边界测试类命名遵循 `{Service}BoundaryTest` 格式。

**示例**：
- `UserDomainServiceBoundaryTest.java`
- `RoleDomainServiceBoundaryTest.java`
- `DepartmentDomainServiceBoundaryTest.java`

#### 3. 测试方法命名规范

**检查内容**：测试方法命名遵循 `{Method}_{Scenario}_{Expected}` 格式。

**示例**：
- `createUser_Success`
- `createUser_EmptyUsername_Exception`
- `getUserById_NotFound_Null`

#### 4. 测试有@DisplayName

**检查内容**：每个测试必须有中文@DisplayName。

**示例**：
```java
@Test
@DisplayName("创建用户 - 成功")
void createUser_Success() { /* ... */ }

@Test
@DisplayName("创建用户 - 用户名为空应抛出异常")
void createUser_EmptyUsername_Exception() { /* ... */ }
```

#### 5. 测试覆盖null参数

**检查内容**：每个方法必须测试null参数。

**示例**：
```java
@Test
@DisplayName("创建用户 - 用户为null应抛出异常")
void createUser_NullUser_Exception() {
    // Given
    UserEntity user = null;

    // When & Then
    assertThatThrownBy(() -> userDomainService.createUser(user))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("用户不能为空");

    verify(userRepository, never()).insert(any(UserEntity.class));
}
```

---

## Story完成检查

Story完成时，需要检查以下内容。

### 检查清单

| # | 检查项 | 检查内容 | 结果 |
|---|--------|---------|------|
| **1** | **所有测试通过** | 所有测试用例必须通过 | ☐ 是 ☐ 否 |
| **2** | **覆盖率达标** | DomainService覆盖率≥80% | ☐ 是 ☐ 否 |
| **3** | **边界测试覆盖率≥90%** | 边界测试覆盖率必须≥90% | ☐ 是 ☐ 否 |
| **4** | **测试用例数量达标** | 测试用例数量达到最少要求 | ☐ 是 ☐ 否 |
| **5** | **CI/CD通过** | CI/CD流水线必须通过 | ☐ 是 ☐ 否 |

### 详细说明

#### 1. 所有测试通过

**检查内容**：所有测试用例必须通过。

**检查命令**：
```bash
# 运行所有测试
mvn test
```

#### 2. 覆盖率达标

**检查内容**：DomainService覆盖率≥80%。

**检查命令**：
```bash
# 运行测试并生成覆盖率报告
mvn clean test jacoco:report

# 查看HTML报告
open target/site/jacoco/index.html
```

**覆盖率标准**：
- DomainService：指令覆盖率≥80%，分支覆盖率≥65%
- AppService：指令覆盖率≥65%，分支覆盖率≥55%
- Controller：指令覆盖率≥50%，分支覆盖率≥40%

#### 3. 边界测试覆盖率≥90%

**检查内容**：边界测试覆盖率必须≥90%。

**计算方法**：
```
边界测试覆盖率 = (已测试边界点 / 总边界点) × 100%
```

#### 4. 测试用例数量达标

**检查内容**：测试用例数量达到最少要求。

**最少用例数**：
- 系统模块：每Service≥15个用例
- 质量模块：每Service≥30个用例
- 标准模块：每Service≥25个用例

#### 5. CI/CD通过

**检查内容**：CI/CD流水线必须通过。

**检查内容**：
- 编译成功
- 所有测试通过
- 覆盖率达标
- 代码检查通过

---

## 发布前检查

发布前，需要检查以下内容。

### 检查清单

| # | 检查项 | 检查内容 | 结果 |
|---|--------|---------|------|
| **1** | **所有测试通过** | 所有测试用例必须通过 | ☐ 是 ☐ 否 |
| **2** | **覆盖率达标** | 所有模块覆盖率达标 | ☐ 是 ☐ 否 |
| **3** | **边界测试覆盖率≥90%** | 所有模块边界测试覆盖率≥90% | ☐ 是 ☐ 否 |
| **4** | **E2E测试通过** | E2E测试必须通过 | ☐ 是 ☐ 否 |

### 详细说明

#### 1. 所有测试通过

**检查内容**：所有测试用例必须通过。

**检查命令**：
```bash
# 运行所有测试
mvn test

# 运行E2E测试
cd frontend/dataengine-frontend
npm run test:e2e
```

#### 2. 覆盖率达标

**检查内容**：所有模块覆盖率达标。

**检查命令**：
```bash
# 运行测试并生成覆盖率报告
mvn clean test jacoco:report

# 查看HTML报告
open target/site/jacoco/index.html
```

#### 3. 边界测试覆盖率≥90%

**检查内容**：所有模块边界测试覆盖率≥90%。

**检查方法**：
- 查看边界测试覆盖率报告
- 确保所有模块边界测试覆盖率≥90%

#### 4. E2E测试通过

**检查内容**：E2E测试必须通过。

**检查命令**：
```bash
# 运行E2E测试
cd frontend/dataengine-frontend
npm run test:e2e
```

---

## 检查清单使用指南

### 代码提交前检查流程

1. **检查测试类存在**
   ```bash
   ls -la src/test/java/com/dp/dataengine/domain/service/
   ```

2. **检查边界测试类存在**
   ```bash
   find src/test/java -name "*BoundaryTest.java"
   ```

3. **检查测试用例数量**
   ```bash
   mvn test -Dtest=UserDomainServiceTest
   ```

4. **检查测试类型分布**
   - 查看测试方法命名
   - 统计各类型数量

5. **运行所有测试**
   ```bash
   mvn test
   ```

### PR审查检查流程

1. **检查测试类命名规范**
   - 查看测试类名是否符合 `{Service}Test` 格式

2. **检查边界测试类存在**
   - 查看是否有 `{Service}BoundaryTest` 类

3. **检查测试方法命名规范**
   - 查看测试方法名是否符合 `{Method}_{Scenario}_{Expected}` 格式

4. **检查@DisplayName**
   - 查看每个测试是否有中文@DisplayName

5. **检查null参数测试**
   - 查看是否测试了null参数

### Story完成检查流程

1. **运行所有测试**
   ```bash
   mvn test
   ```

2. **检查覆盖率**
   ```bash
   mvn clean test jacoco:report
   open target/site/jacoco/index.html
   ```

3. **检查边界测试覆盖率**
   - 查看边界测试覆盖率报告

4. **检查测试用例数量**
   - 统计测试用例数量

5. **检查CI/CD**
   - 查看CI/CD流水线状态

### 发布前检查流程

1. **运行所有测试**
   ```bash
   mvn test
   cd frontend/dataengine-frontend
   npm run test:e2e
   ```

2. **检查覆盖率**
   ```bash
   mvn clean test jacoco:report
   open target/site/jacoco/index.html
   ```

3. **检查边界测试覆盖率**
   - 查看边界测试覆盖率报告

4. **检查E2E测试**
   ```bash
   cd frontend/dataengine-frontend
   npm run test:e2e
   ```

---

## 🔗 相关文档

- [项目测试策略](./project-testing-strategy.md) - 项目整体测试策略
- [边界测试规范](./boundary-testing.md) - 边界测试详细规范
- [测试用例数量指南](./test-quantity-guide.md) - 测试用例数量计算指南
- [单元测试规范](./unit-testing.md) - 单元测试编写规范
- [覆盖率标准](./coverage-standards.md) - 覆盖率验证标准

---

**最后更新**：2026-03-12