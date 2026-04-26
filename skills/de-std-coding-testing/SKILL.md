---
name: coding-testing
description: 测试规范 TDD/JUnit/Vitest/E2E。写测试代码前必须调用。
---

# 测试编码规范

## 触发时机

- "写单元测试/集成测试"
- "添加测试用例"
- "写 E2E 测试"
- "检查测试覆盖率"
- **实现功能前（TDD 要求先写测试）**

---

## TDD 强制流程

```
RED → GREEN → REFACTOR

1. RED：写一个失败的测试，运行确认失败
2. GREEN：写最少代码让测试通过
3. REFACTOR：优化代码，确保测试仍通过
```

**判定标准**：测试文件最后修改时间不得晚于实现文件。

---

## 覆盖率标准

### 后端覆盖率

| 层级 | 指令覆盖率 | 分支覆盖率 | 方法覆盖率 |
|------|-----------|-----------|-----------|
| DomainService | ≥80% | ≥65% | ≥85% |
| AppService | ≥65% | ≥55% | ≥75% |
| Controller | ≥50% | ≥40% | ≥60% |
| Common/Infra | ≥70% | ≥60% | ≥75% |

### 前端覆盖率

| 类型 | 覆盖率要求 |
|------|-----------|
| 工具函数 | ≥90% |
| API 函数 | ≥80% |
| Store | ≥80% |
| 组件 | ≥70% |
| 页面 | ≥60% |

---

## 单元测试规范（Java）

### 测试数据构造（Instancio）

使用 Instancio 自动生成测试数据，减少 70-90% 样板代码：

```java
import static org.instancio.Instancio.*;
import static org.instancio.Select.field;

// 一行生成完整对象
UserEntity user = create(UserEntity.class);

// 只覆盖测试相关字段
UserEntity user = of(UserEntity.class)
    .set(field(UserEntity::getUsername), "admin")
    .create();

// 批量生成
List<UserEntity> users = ofList(UserEntity.class).size(10).create();

// JUnit 5 参数注入
@ExtendWith(InstancioExtension.class)
class MyTest {
    @Test
    void test(@Given UserEntity user) { /* user 已自动填充 */ }
}
```

| 场景 | 推荐方式 |
|------|---------|
| 3+ 字段的对象 | `Instancio.create()` 或 `@Given` |
| 只关心部分字段 | `of().set().create()` |
| 1-2 字段简单对象 | 手动构造即可 |

详细规范：[test-data.md](references/test-data.md)

### 测试命名

```java
// ✅ 正确命名
@Test
void returnsEmptyListWhenNoUsersMatchQuery() { }

@Test
void throwsExceptionWhenApiKeyMissing() { }

@Test
void calculatesTotalCorrectlyWhenCartHasMultipleItems() { }
```

### AAA 模式

```java
@Test
void calculatesSimilarityCorrectly() {
    // Arrange
    Vector vector1 = new Vector(1, 0, 0);
    Vector vector2 = new Vector(0, 1, 0);

    // Act
    double similarity = calculator.cosineSimilarity(vector1, vector2);

    // Assert
    assertThat(similarity).isEqualTo(0.0);
}
```

### Mock 使用

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock
    private UserMapper userMapper;

    @InjectMocks
    private UserDomainService userDomainService;

    @Test
    void shouldReturnUser_WhenIdExists() {
        // Given
        when(userMapper.findById(1L)).thenReturn(new UserEntity());

        // When
        UserEntity result = userDomainService.getById(1L);

        // Then
        assertThat(result).isNotNull();
    }
}
```

---

## 前端测试规范（Vitest）

### 组件测试

```typescript
describe('DataTable', () => {
  it('shows empty state when no data', () => {
    const wrapper = mount(DataTable, {
      props: { data: [] }
    })
    expect(wrapper.text()).toContain('暂无数据')
  })
})
```

### 测试文件位置

```
src/
├── views/
│   └── user/
│       ├── UserList.vue
│       └── __tests__/
│           └── UserList.spec.ts
└── composables/
    └── usePagination.ts
    └── __tests__/
        └── usePagination.spec.ts
```

---

## E2E 测试规范（Playwright）

### 强制规则

- **禁止调用后端 API**：只通过浏览器操作
- **完整用户旅程**：创建→查询→编辑→删除全链路
- **data-testid 定位**：禁止依赖 CSS class

### 测试示例

```typescript
test('user CRUD journey', async ({ page }) => {
  // 创建
  await page.goto('/users')
  await page.getByTestId('btn-create').click()
  await page.getByTestId('input-name').fill('e2e_test_user')
  await page.getByTestId('btn-submit').click()

  // 查询
  await expect(page.getByText('e2e_test_user')).toBeVisible()

  // 删除
  await page.getByTestId('btn-delete').click()
  await expect(page.getByText('e2e_test_user')).not.toBeVisible()
})
```

### 四层测试体系

| 层级 | 类型 | 工具 | 比例 |
|------|------|------|------|
| L1 | 单元测试 | JUnit/Vitest | 50% |
| L2 | 组件测试 | Vitest | 15% |
| L3 | API集成测试 | Testcontainers | 25% |
| L4 | E2E测试 | Playwright | 10% |

---

## 边界测试规范

### 边界测试分类

| 类型 | 示例 |
|------|------|
| 空值/空字符串 | null、""、[] |
| 长度边界 | 最大长度、最小长度 |
| 数值边界 | 0、负数、最大值 |
| ID 边界 | 无效ID、超长ID |
| 状态/枚举 | 无效状态值 |
| 特殊字符 | XSS字符、SQL注入 |

### 风险驱动模型

| 风险等级 | 测试要求 |
|---------|---------|
| P0（核心功能） | 必须覆盖所有边界 |
| P1（重要功能） | 覆盖主要边界 |
| P2（一般功能） | 覆盖基本边界 |
| P3（低风险） | 可选边界测试 |

---

## 测试用例数量公式

```
测试用例数 = 方法数 × 3 + 分支数 × 2 + 边界点数 × 1
```

### 复杂度要求

| 复杂度 | 测试用例数 |
|-------|-----------|
| 简单方法 | ≥5 |
| 中等方法 | ≥8 |
| 复杂方法 | ≥12 |
| 核心方法 | ≥15 |

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| 先写实现后补测试 | 违反 TDD |
| E2E 调用后端 API | 不是真实用户操作 |
| 测试覆盖率未达标 | Story 不能完成 |
| 依赖 CSS class 定位 | UI 变化导致测试失败 |
| skip 存在时标记 done | 套件处于 blocked 状态 |

---

## 详细规范

- **[tdd-workflow.md](references/tdd-workflow.md)** — TDD 开发流程、Red-Green-Refactor 循环、Story 状态流转
- **[unit-testing.md](references/unit-testing.md)** — 单元测试框架、Mock使用、断言规范
- **[integration-testing.md](references/integration-testing.md)** — 集成测试配置、Testcontainers、L3 API 测试
- **[e2e-testing.md](references/e2e-testing.md)** — E2E 测试规范 v4.0、Playwright、元素定位、四层测试体系
- **[coverage-standards.md](references/coverage-standards.md)** — 覆盖率标准、JaCoCo配置、Vitest配置
- **[boundary-testing.md](references/boundary-testing.md)** — 边界测试规范、风险驱动模型、测试模板
- **[test-checklist.md](references/test-checklist.md)** — 提交前检查、PR审查、Story完成检查
- **[test-data.md](references/test-data.md)** — 测试数据管理、数据隔离、数据工厂
- **[test-quantity-guide.md](references/test-quantity-guide.md)** — 测试用例数量计算、复杂度要求
- **[test-template.md](references/test-template.md)** — DomainService测试模板、必须测试场景
- **[project-testing-strategy.md](references/project-testing-strategy.md)** — 项目测试策略、测试金字塔、层级职责
