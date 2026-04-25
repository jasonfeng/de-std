# 覆盖率标准

> 本规范定义测试覆盖率的标准和验证流程，确保代码质量。

---

## 📋 目录

- [覆盖率要求](#覆盖率要求)
- [JaCoCo配置](#jacoco配置)
- [覆盖率验证](#覆盖率验证)
- [覆盖率报告](#覆盖率报告)
- [排除规则](#排除规则)

---

## 覆盖率要求

### 按层级的覆盖率要求

| 层级 | 指令覆盖率 | 分支覆盖率 | 方法覆盖率 | 说明 |
|------|-----------|-----------|-----------|------|
| **DomainService** | ≥80% | ≥65% | ≥85% | 核心业务逻辑（L1 单元测试） |
| **AppService** | ≥65% | ≥55% | ≥75% | 应用服务编排（L1 单元测试） |
| **Controller** | ≥60% | ≥50% | ≥70% | 接口层（L3 API 集成测试） |
| **Common/Infrastructure** | ≥70% | ≥60% | ≥75% | 工具类/基础设施 |
| **Entity/VO/Param/DTO** | 不强制 | 不强制 | 不强制 | 数据载体类 |

> **v2.1 更新（2026-04-11）**：Controller 层覆盖率阈值提升（指令 50%→60%，分支 40%→50%，方法 60%→70%），由 L3 Testcontainers 集成测试驱动。

### 覆盖率类型说明

| 覆盖率类型 | 说明 | 计算方式 |
|-----------|------|---------|
| **指令覆盖率** | 字节码指令覆盖率 | 执行的字节码数 / 总字节码数 |
| **分支覆盖率** | 条件分支覆盖率 | 执行的分支数 / 总分支数 |
| **方法覆盖率** | 方法调用覆盖率 | 执行的方法数 / 总方法数 |
| **类覆盖率** | 类使用覆盖率 | 执行的类数 / 总类数 |
| **行覆盖率** | 源代码行覆盖率 | 执行的代码行数 / 总代码行数 |

---

## JaCoCo配置

### pom.xml配置

```xml
<build>
    <plugins>
        <!-- Maven Surefire Plugin（单元测试） -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0-M5</version>
            <configuration>
                <includes>
                    <include>**/*Test.java</include>
                    <include>**/*Tests.java</include>
                </includes>
            </configuration>
        </plugin>

        <!-- Maven Failsafe Plugin（集成测试） -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>3.0.0-M5</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- JaCoCo Plugin（覆盖率） -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.8</version>
            <executions>
                <!-- 准备Agent -->
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>

                <!-- 生成覆盖率报告 -->
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>

                <!-- 检查覆盖率 -->
                <execution>
                    <id>check</id>
                    <goals>
                        <goal>check</goal>
                    </goals>
                    <configuration>
                        <rules>
                            <!-- DomainService覆盖率检查 -->
                            <rule>
                                <element>CLASS</element>
                                <includes>
                                    <include>com.dp.dataengine.domain.service.**</include>
                                </includes>
                                <limits>
                                    <limit>
                                        <counter>INSTRUCTION</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.80</minimum>
                                    </limit>
                                    <limit>
                                        <counter>BRANCH</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.65</minimum>
                                    </limit>
                                </limits>
                            </rule>

                            <!-- AppService覆盖率检查 -->
                            <rule>
                                <element>CLASS</element>
                                <includes>
                                    <include>com.dp.dataengine.application.service.**</include>
                                </includes>
                                <limits>
                                    <limit>
                                        <counter>INSTRUCTION</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.65</minimum>
                                    </limit>
                                    <limit>
                                        <counter>BRANCH</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.55</minimum>
                                    </limit>
                                </limits>
                            </rule>

                            <!-- Controller覆盖率检查 -->
                            <rule>
                                <element>CLASS</element>
                                <includes>
                                    <include>com.dp.dataengine.interfaces.controller.**</include>
                                </includes>
                                <limits>
                                    <limit>
                                        <counter>INSTRUCTION</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.50</minimum>
                                    </limit>
                                    <limit>
                                        <counter>BRANCH</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.40</minimum>
                                    </limit>
                                </limits>
                            </rule>

                            <!-- Common/Infrastructure覆盖率检查 -->
                            <rule>
                                <element>CLASS</element>
                                <includes>
                                    <include>com.dp.dataengine.common.**</include>
                                    <include>com.dp.dataengine.infrastructure.**</include>
                                </includes>
                                <limits>
                                    <limit>
                                        <counter>INSTRUCTION</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.70</minimum>
                                    </limit>
                                    <limit>
                                        <counter>BRANCH</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.60</minimum>
                                    </limit>
                                </limits>
                            </rule>
                        </rules>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

---

## 覆盖率验证

### 运行测试并生成覆盖率报告

```bash
# 运行所有测试并生成覆盖率报告
mvn clean test

# 只运行测试（不生成报告）
mvn test

# 运行特定测试类
mvn test -Dtest=UserDomainServiceTest

# 运行特定测试方法
mvn test -Dtest=UserDomainServiceTest#createUser_Success

# 跳过测试（不推荐）
mvn clean install -DskipTests
```

### 查看覆盖率报告

```bash
# HTML报告位置
target/site/jacoco/index.html

# 使用浏览器打开
open target/site/jacoco/index.html
```

### 查看覆盖率命令

```bash
# 查看覆盖率摘要
mvn jacoco:report

# 检查覆盖率是否达标
mvn jacoco:check
```

---

## 覆盖率报告

### HTML报告结构

```
target/site/jacoco/
├── index.html              # 总览页面
├── com.dp.dataengine/      # 包级覆盖率
│   ├── application/        # 应用层覆盖率
│   │   ├── service/        # 应用服务覆盖率
│   │   └── dto/            # DTO覆盖率
│   ├── domain/             # 领域层覆盖率
│   │   ├── service/        # 领域服务覆盖率
│   │   ├── entity/         # 实体覆盖率
│   │   └── repository/     # Repository覆盖率
│   ├── interfaces/         # 接口层覆盖率
│   │   ├── controller/     # 控制器覆盖率
│   │   ├── param/          # 参数覆盖率
│   │   └── vo/             # VO覆盖率
│   ├── common/             # 通用组件覆盖率
│   └── infrastructure/     # 基础设施覆盖率
└── jacoco.exec            # 覆盖率数据文件
```

### 报告指标说明

| 指标 | 说明 | 目标值 |
|------|------|--------|
| **Missed Instructions** | 未执行的字节码指令数 | - |
| **Cov** | 指令覆盖率 | ≥80% (DomainService) |
| **Missed Branches** | 未执行的分支数 | - |
| **Cov** | 分支覆盖率 | ≥65% (DomainService) |
| **Missed Cxty** | 未执行的圈复杂度 | - |
| **Cov** | 圈复杂度覆盖率 | ≥65% (DomainService) |
| **Missed Methods** | 未执行的方法数 | - |
| **Cov** | 方法覆盖率 | ≥85% (DomainService) |
| **Missed Classes** | 未执行的类数 | - |
| **Cov** | 类覆盖率 | ≥85% (DomainService) |

---

## 排除规则

### 排除不需要测试的类

在pom.xml中配置排除规则：

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <configuration>
        <!-- 排除不需要测试的类 -->
        <excludes>
            <!-- 实体类（数据载体） -->
            <exclude>**/entity/*.class</exclude>
            <exclude>**/dto/*.class</exclude>
            <exclude>**/param/*.class</exclude>
            <exclude>**/vo/*.class</exclude>

            <!-- 配置类 -->
            <exclude>**/config/*.class</exclude>

            <!-- 异常类 -->
            <exclude>**/exception/*.class</exclude>

            <!-- 枚举类 -->
            <exclude>**/enums/*.class</exclude>

            <!-- 常量类 -->
            <exclude>**/constant/*.class</exclude>

            <!-- Lombok生成的代码 -->
            <exclude>**/*$*.class</exclude>

            <!-- 应用主类 -->
            <exclude>**/Application.class</exclude>
        </excludes>
    </configuration>
</plugin>
```

### 排除不需要测试的方法

在测试类中使用@Ignore注解：

```java
@Test
@Ignore("暂不测试")
@DisplayName("暂不测试的方法")
void testMethod() {
    // ...
}
```

---

## 覆盖率提升策略

### 1. 增加测试用例

```java
// 原始测试（覆盖率50%）
@Test
void createUser_Success() {
    UserDTO dto = new UserDTO();
    dto.setUsername("admin");

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
}

// 增加测试用例（覆盖率80%）
@Test
void createUser_Success() {
    UserDTO dto = new UserDTO();
    dto.setUsername("admin");

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo("admin");
}

@Test
void createUser_UsernameBlank_Exception() {
    UserDTO dto = new UserDTO();
    dto.setUsername("");

    assertThatThrownBy(() -> userDomainService.createUser(dto))
        .isInstanceOf(BusinessException.class)
        .hasMessage("用户名不能为空");
}

@Test
void createUser_DuplicateUsername_Exception() {
    UserDTO dto = new UserDTO();
    dto.setUsername("admin");

    when(userRepository.existsByUsername("admin")).thenReturn(true);

    assertThatThrownBy(() -> userDomainService.createUser(dto))
        .isInstanceOf(BusinessException.class)
        .hasMessage("用户名已存在");
}
```

### 2. 使用参数化测试

```java
@ParameterizedTest
@ValueSource(strings = {"admin", "user", "test"})
@DisplayName("创建用户 - 多个用户名")
void createUser_MultipleUsernames(String username) {
    UserDTO dto = new UserDTO();
    dto.setUsername(username);

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo(username);
}

@ParameterizedTest
@CsvSource({
    "admin, admin@example.com",
    "user, user@example.com",
    "test, test@example.com"
})
@DisplayName("创建用户 - 多个用户")
void createUser_MultipleUsers(String username, String email) {
    UserDTO dto = new UserDTO();
    dto.setUsername(username);
    dto.setEmail(email);

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo(username);
    assertThat(result.getEmail()).isEqualTo(email);
}
```

### 3. Mock外部依赖

```java
@Test
void createUser_ExternalService_Success() {
    UserDTO dto = new UserDTO();
    dto.setUsername("admin");

    // Mock外部服务
    when(externalService.checkUsername("admin")).thenReturn(true);
    when(userRepository.insert(any(UserEntity.class))).thenReturn(1);

    UserDTO result = userDomainService.createUser(dto);

    assertThat(result).isNotNull();
    verify(externalService, times(1)).checkUsername("admin");
}
```

---

## 前端覆盖率标准（v2.0 新增，2026-04-10）

> 配合四层测试体系新增 Vitest 组件测试层。详见 `docs/e2e-testing-optimization-proposal.md`

### 按组件类型的覆盖率要求

| 组件类型 | 行覆盖率 | 说明 |
|---------|---------|------|
| 表单组件（向导页等） | ≥80% | 核心交互组件 |
| 列表组件 | ≥70% | 展示型组件 |
| 公共组件 | ≥80% | DataTable、FormDialog 等 |
| 页面组件 | ≥60% | 整体页面逻辑 |

### Vitest 覆盖率配置（v4.1 更新，2026-04-16）

**强制 thresholds 配置**：

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      // 扩展覆盖范围
      include: [
        'src/components/**/*.vue',     // 所有组件
        'src/composables/**/*.ts',     // Composables
        'src/stores/**/*.ts',          // 状态管理
        'src/views/**/*.vue'           // 页面组件
      ],
      exclude: [
        'node_modules/',
        'src/**/*.d.ts',
        'src/**/__tests__/**',         // 测试文件本身不统计
        'src/**/types/**',             // 纯类型定义文件
        'src/**/constants/**'          // 常量定义文件
      ],
      // 过渡期阈值（2026-05-01 截止，逐步提升）
      thresholds: {
        lines: 30,                      // 最低行覆盖率
        branches: 25,                   // 最低分支覆盖率
        functions: 30,                  // 最低函数覆盖率
        perFile: true                   // 每个文件单独检查
      },
      reportOnFailure: true            // 失败时也生成报告
    }
  }
})
```

**豁免文件追踪**：`docs/vitest-tech-debt-list.md`

### 运行覆盖率

```bash
# 生成前端覆盖率报告
npx vitest run --coverage

# 查看报告
open coverage/index.html
```

---

## L3 API 集成测试覆盖率目标（v2.1 新增，2026-04-11）

> 使用 Testcontainers 的 L3 集成测试为 Controller 层提供真实覆盖率。

| 模块 | Controller | L3 测试类 | 目标覆盖率 |
|------|-----------|----------|-----------|
| **质量规则** | QualityRuleController | QualityRuleApiIntegrationTest | ≥70% |
| **质量任务** | QualityTaskController | QualityTaskApiIntegrationTest | ≥60% |
| **质量报告** | QualityReportController | QualityReportApiIntegrationTest | ≥60% |
| **业务域** | BusinessDomainController | BusinessDomainApiIntegrationTest | ≥60% |
| **数据源** | DataSourceController | DataSourceApiIntegrationTest | ≥60% |
| **同步任务** | SyncTaskController | SyncTaskApiIntegrationTest | ≥60% |
| **任务中心** | TaskController | TaskApiIntegrationTest | ≥60% |

**注意**：L3 覆盖率与 L1 单元测试覆盖率独立。JaCoCo 合并两层结果。

---

## 🔗 相关文档

- [TDD 开发工作流](./tdd-workflow.md) - **什么时候写测试**（开发流程约束）
- [单元测试规范](./unit-testing.md) - 单元测试详细规范
- [集成测试规范](./integration-testing.md) - 集成测试详细规范
- [测试数据管理](./test-data.md) - 测试数据管理详细规范
- [E2E 测试规范](./e2e-testing.md) - E2E 测试规范（v4.0 四层体系）
- [项目测试策略](./project-testing-strategy.md) - 测试金字塔与分层策略

---

**最后更新**：2026-04-10