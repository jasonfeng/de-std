---
name: coding-testing
description: 测试规范 TDD/JUnit/Vitest/E2E。写测试代码前必须调用。
---

# 测试编码规范

## 触发时机

用户说以下内容时**必须**先调用此 Skill：

- "写单元测试/集成测试"
- "添加测试用例"
- "写 E2E 测试"
- "检查测试覆盖率"
- **实现功能前（TDD 要求先写测试）**

## 调用时机

- 写单元测试之前
- 写集成测试之前
- 写 E2E 测试之前
- 开始任何功能开发之前（TDD）

## TDD 强制流程

```
RED → GREEN → REFACTOR

1. RED：写一个失败的测试
2. GREEN：写最少代码让测试通过
3. REFACTOR：优化代码，确保测试仍通过
```

**强制约束**：测试文件最后修改时间不得晚于实现文件

---

## 覆盖率标准

| 层级 | 指令覆盖率 | 分支覆盖率 | 方法覆盖率 |
|------|-----------|-----------|-----------|
| DomainService | ≥80% | ≥65% | ≥85% |
| AppService | ≥65% | ≥55% | ≥75% |
| Controller | ≥50% | ≥40% | ≥60% |
| Entity/DTO/VO | 不强制 | 不强制 | 不强制 |

---

## 单元测试规范（Java）

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

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| 先写实现后补测试 | 违反 TDD |
| E2E 调用后端 API | 不是真实用户操作 |
| 测试覆盖率未达标 | Story 不能完成 |
| 依赖 CSS class 定位 | UI 变化导致测试失败 |
| skip 存在时标记 done | 套件处于 blocked 状态 |