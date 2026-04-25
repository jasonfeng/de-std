# TDD 开发工作流规范

> 本规范定义测试驱动开发（TDD）在数擎项目中的操作流程，约束开发过程中"什么时候写测试"和"怎么写测试"。
>
> **核心铁律**：前后端开发模式严格为 TDD。所有新功能开发、Bug 修复、代码重构均须遵循 Red → Green → Refactor 循环。实现代码的最后修改时间不得早于对应测试文件的最后修改时间。无测试的代码不予合并。
>
> **适用范围**：
> - **后端**（Java / JUnit 5）：DomainService、AppService、Controller、Infrastructure 层
> - **前端**（Vue 3 / Vitest）：组件、Composable、Store、工具函数
> - **豁免项**：纯配置文件（pom.xml、.env）、SQL 迁移脚本、无逻辑的 DTO/VO/Param/枚举
>
> **版本**: 1.0
> **创建日期**: 2026-04-05
> **维护者**: 开发团队

---

## 一、为什么需要 TDD

### TDD 解决的问题

| 问题 | 没有 TDD | 有 TDD |
|------|---------|--------|
| 代码写完后才补测试 | 测试只是走过场，覆盖率虚高 | 测试验证实际行为，覆盖率真实 |
| 发现 bug 时已上线 | 修复成本高，影响用户 | 开发阶段就能发现 |
| 重构后功能退化 | 不知道改坏了什么 | 测试守护已有功能 |
| Code Review 时没有测试 | 审查者无法验证正确性 | 测试即文档，审查有依据 |

### 数擎项目的特殊要求

我们是一个 **AI 团队**，TDD 和 E2E 是保证代码质量的核心手段：
1. **TDD** 保证代码写完是对的（逻辑正确）
2. **E2E** 保证用户使用路径是对的（体验正确）
3. 两者结合，尽可能多地发现问题，不回避问题

---

## 二、TDD 循环：Red → Green → Refactor

### 2.1 三步循环

```
  ┌─────────────────────────────────────────────────┐
  │                                                   │
  │    ① Red（红）：写一个失败的测试                      │
  │         ↓                                         │
  │    ② Green（绿）：写最少的代码让测试通过               │
  │         ↓                                         │
  │    ③ Refactor（重构）：在不改变行为的前提下优化代码     │
  │         ↓                                         │
  │    回到 ①，下一个测试                                │
  │                                                   │
  └─────────────────────────────────────────────────┘
```

### 2.2 每一步的规则

| 步骤 | 做什么 | 禁止什么 |
|------|--------|---------|
| **Red** | 写一个测试，描述你期望的行为。运行测试，确认它失败 | 禁止在测试失败前写实现代码 |
| **Green** | 写最少的代码让测试通过。不要过度设计 | 禁止在测试通过前开始优化 |
| **Refactor** | 消除重复、改善命名、简化逻辑。运行测试确认仍然通过 | 禁止在重构时改变行为（测试必须仍然通过） |

### 2.3 判断是否违反 TDD 的方法

如果以下任何一条为 **否**，说明违反了 TDD：

- [ ] 写实现代码之前，有没有对应的测试？
- [ ] 测试是否先于实现失败过（Red 阶段）？
- [ ] 实现代码是否是最小化的（没有过度设计）？

---

## 三、按 DDD 分层的 TDD 流程

### 3.1 DomainService（最高优先级）

DomainService 是业务核心，必须严格遵循 TDD。

```
开发一个新方法：

1. 打开 XxxDomainServiceTest.java
2. 写 @Nested 分组（按方法分组）
3. 写第一个测试：正常路径（Success）
4. 运行测试 → 确认失败（Red）
5. 在 XxxDomainService.java 中写实现
6. 运行测试 → 确认通过（Green）
7. 写异常路径测试（Exception）
8. 运行测试 → 确认失败 → 补实现 → 通过
9. 写边界条件测试（Boundary）
10. 运行测试 → 确认失败 → 补实现 → 通过
11. 检查代码，重构（Refactor）
12. 运行全部测试 → 确认仍然通过
```

**示例**：开发 `createDirectPhysicalModel` 方法

```java
// Step 1: Red — 写测试
@Nested
@DisplayName("直接创建物理模型")
class CreateDirectPhysicalModelTests {

    @Test
    @DisplayName("应该成功创建物理模型（有 Doris）")
    void shouldCreateSuccessfullyWithDoris() {
        // Given
        String tableName = "test_direct_table";
        List<FieldDefinitionDTO> fields = List.of(
            FieldDefinitionDTO.builder().fieldName("id").fieldType("BIGINT").build()
        );
        when(physicalModelRepository.findByTableName(tableName)).thenReturn(null);
        when(ddlGeneratorService.generateCreateTableDdlForFields(anyString(), anyString(), anyList(), isNull()))
            .thenReturn("CREATE TABLE test_direct_table (id BIGINT)");

        // When
        PhysicalModelEntity result = service.createDirectPhysicalModel(
            tableName, "test_db", "REBUILD", null, fields);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getTableName()).isEqualTo(tableName);
        assertThat(result.getCreationSource()).isEqualTo("DIRECT");
    }
}

// 此时运行测试 → 失败（因为方法还不存在）

// Step 2: Green — 写最小实现
public PhysicalModelEntity createDirectPhysicalModel(
        String tableName, String database, String strategy,
        String comment, List<FieldDefinitionDTO> fields) {
    if (physicalModelRepository.findByTableName(tableName) != null) {
        throw new BusinessException("物理模型已存在");
    }
    // ... 最小实现 ...
}

// 运行测试 → 通过

// Step 3: 继续写异常路径测试
@Test
@DisplayName("当表名已存在时应抛出异常")
void shouldThrowWhenTableExists() {
    when(physicalModelRepository.findByTableName("existing_table"))
        .thenReturn(existingEntity);

    assertThatThrownBy(() -> service.createDirectPhysicalModel(
        "existing_table", "test_db", "REBUILD", null, fields))
        .isInstanceOf(BusinessException.class);
}

// 补充异常处理逻辑 → 测试通过
```

### 3.2 AppService（应用层编排）

AppService 侧重于事务编排和 DTO 转换，TDD 侧重点不同：

| 测试重点 | 说明 |
|---------|------|
| 参数校验 | 参数为空、格式错误时抛异常 |
| DTO 转换 | Entity → DTO 转换正确 |
| 事务边界 | 调用 DomainService + Repository 的正确组合 |
| 异常处理 | DomainService 抛异常时正确包装返回 |

### 3.3 Controller（接口层）

Controller 测试偏轻量，主要验证：
- 参数绑定（@Valid 校验）
- 返回格式（Result 包装）
- HTTP 方法映射

Controller 不需要 TDD 驱动，可以在 Service 层测试通过后补充。

### 3.4 前端组件

前端 TDD 侧重于组件逻辑（不包含样式）：

| 测试重点 | 说明 |
|---------|------|
| 事件处理 | 按钮点击、表单提交的逻辑 |
| 条件渲染 | 不同 props 下显示不同内容 |
| 数据转换 | API 响应到展示数据的转换 |
| 异常处理 | API 失败时的 UI 反馈 |

---

## 四、TDD 与 Story 状态流转

### 4.1 状态机

```
  ┌──────────────┐     测试先于代码      ┌──────────────┐
  │   planning   │ ──────────────────→  │ in-progress  │
  └──────────────┘                      └──────┬───────┘
                                               │
                                    ┌──────────┴──────────┐
                                    │                     │
                                    ▼                     ▼
                          ┌─────────────┐        ┌──────────────┐
                          │  测试通过    │        │  测试未通过  │
                          │  覆盖率达标  │        │  或未编写    │
                          └──────┬──────┘        └──────┬──────┘
                                 │                      │
                                 ▼                      │
                          ┌─────────────┐               │
                          │   review    │  ← 禁止进入 ──┘
                          └──────┬──────┘
                                 │
                                 ▼
                          ┌─────────────┐
                          │review-complete│
                          └──────┬──────┘
                                 │
                                 ▼
                          ┌─────────────┐
                          │    done     │
                          └─────────────┘
```

### 4.2 门禁规则

| 状态转换 | 门禁条件 | 违反后果 |
|---------|---------|---------|
| `planning` → `in-progress` | 测试文件已创建（骨架） | 拒绝转换 |
| `in-progress` → `review` | 单元测试通过 + 覆盖率达标 + E2E 已编写 | 拒绝转换 |
| `review` → `review-complete` | Code Review 通过 | 正常流程 |

### 4.3 反模式：禁止的做法

| 反模式 | 为什么错误 | 正确做法 |
|--------|-----------|---------|
| 先写完所有代码再补测试 | 测试变成"证明代码能跑"而不是"验证行为正确" | 每个方法先写测试 |
| 测试只验证 happy path | 异常路径是 bug 高发区 | 至少覆盖：正常、异常、边界 |
| `mvn test` 跳过失败 | 掩盖问题 | 修复失败再继续 |
| 覆盖率够了就不写了 | 覆盖率不等于质量 | 覆盖率是底线，不是目标 |

---

## 五、开发节奏

### 5.1 一个 Feature 的 TDD 节奏

```
Feature 开发（以"物理模型直接创建"为例）：

Day 1 上午：
  [1] 读取 Story 需求
  [2] 创建 PhysicalModelDomainServiceTest.java（骨架 + @Mock 注入）
  [3] 写第一个测试：shouldCreateSuccessfullyWithDoris
  [4] 运行测试 → 失败（Red）✓
  [5] 在 DomainService 中写 createDirectPhysicalModel 方法
  [6] 运行测试 → 通过（Green）✓

Day 1 下午：
  [7] 写异常路径测试：shouldThrowWhenTableExists
  [8] 运行测试 → 失败 → 补实现 → 通过
  [9] 写边界测试：shouldThrowWhenDorisExecutionFails
  [10] 运行测试 → 失败 → 补实现 → 通过
  [11] 写 DDL 预览测试
  [12] 运行测试 → 失败 → 补实现 → 通过
  [13] 运行全部测试 → 全部通过 → 检查覆盖率

Day 2：
  [14] AppService 测试 → 实现
  [15] Controller 测试 → 实现
  [16] E2E 测试编写（用户旅程）
  [17] 前后端联调
  [18] 状态转换：in-progress → review
```

### 5.2 微观节奏（每次提交）

每次 git commit 之前，确保：

```bash
# 1. 运行单元测试
mvn test -pl backend/dataengine-api

# 2. 检查覆盖率（DomainService ≥ 80%）
mvn jacoco:report -pl backend/dataengine-api

# 3. 如果有前端变更，运行 E2E
npx playwright test
```

### 5.3 测试文件的创建时机

| 阶段 | 创建什么 | 在哪里 |
|------|---------|--------|
| Story 开始时 | 测试类骨架 + @Mock 注入 | `src/test/java/.../XxxDomainServiceTest.java` |
| 每写一个方法前 | 该方法的测试用例 | 同上 |
| AppService 编写时 | AppService 测试 | `src/test/java/.../XxxAppServiceTest.java` |
| UI 组件完成时 | E2E 测试 | `e2e/xxx/xxx-page.spec.ts` |
| Story 提交 review 前 | 确认全部通过 | 运行 `mvn test` + `npx playwright test` |

---

## 六、常见问题

### Q1: 修改已有代码时要不要先写测试？

**要**。修改前先写测试锁定当前行为（ characterization test），确保修改不破坏已有功能。

### Q2: 紧急 bug 修复要不要 TDD？

**要**。Bug 修复流程：
1. 写一个测试复现 bug（测试应该失败）
2. 修复 bug（测试应该通过）
3. 检查是否有其他测试失败

这保证了 bug 不会回归。

### Q3: 如果 DomainService 依赖了外部服务（Doris、Redis），怎么 TDD？

**Mock 外部依赖**。DomainService 的单元测试通过 @Mock 隔离所有外部依赖。

```java
@Mock
private JdbcTemplate dorisJdbcTemplate;  // Mock Doris
@Mock
private PhysicalModelRepository physicalModelRepository;  // Mock 数据库

// 测试只验证 DomainService 自身的逻辑
```

### Q4: 覆盖率 80% 和 TDD 是什么关系？

覆盖率是 TDD 的**底线检查**，不是目标：
- TDD 保证每个行为都有测试验证（质量维度）
- 覆盖率保证没有大段代码没被测试覆盖（数量维度）
- 两者结合才是完整的质量保障

### Q5: TDD 会拖慢开发速度吗？

短期看会慢，长期看更快：
- 减少调试时间（bug 在写代码时就发现了）
- 减少返工（Review 时不用回来修 bug）
- 重构有信心（测试守护已有行为）

---

## 七、检查清单

### Story 开始时

- [ ] 测试文件已创建（至少是骨架）
- [ ] 依赖的 Mock 对象已声明

### 每写一个方法后

- [ ] 该方法的正常路径有测试
- [ ] 该方法的异常路径有测试
- [ ] 该方法的边界条件有测试
- [ ] `mvn test` 通过

### Story 提交 Review 前

- [ ] 所有单元测试通过
- [ ] DomainService 覆盖率 ≥ 80%
- [ ] E2E 测试已编写（如有 UI 变更）
- [ ] `mvn test` 无失败
- [ ] `npx playwright test` 无失败（如有 UI 变更）

---

## 八、与其他规范的关系

| 规范 | 覆盖范围 | 关系 |
|------|---------|------|
| 本文档（tdd-workflow.md） | **WHEN**：什么时候写测试 | 核心约束 |
| [unit-testing.md](./unit-testing.md) | **HOW**：怎么写单元测试 | 技术细节 |
| [coverage-standards.md](./coverage-standards.md) | **TARGET**：覆盖率要达到多少 | 量化目标 |
| [e2e-testing.md](./e2e-testing.md) | **E2E**：端到端测试规范 | E2E 标准 |
| [story-completion.md](../05-workflow/story-completion.md) | **GATE**：Story 完成门禁 | 状态流转 |

---

## 九、版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-04-05 | 初始版本，定义 TDD 工作流和 DDD 分层实践 |

---

**最后更新**: 2026-04-05
**维护者**: 开发团队
