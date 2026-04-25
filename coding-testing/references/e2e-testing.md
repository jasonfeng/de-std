# E2E 端到端测试规范 v4.0

> 本文档定义数擎项目的E2E测试标准。
>
> **核心理念**（v4.0 更新）：E2E 测试专注于跨模块核心旅程和浏览器环境验证。单模块功能验证由 API 集成测试（后端）和 Vitest 组件测试（前端）负责。四层测试体系确保 AI 团队的全自动化质量保证。
>
> **版本**: 4.0
> **创建日期**: 2026-03-22
> **重大更新**: 2026-04-10（v4.0：四层测试体系、E2E 聚焦核心旅程、API 测试替代单模块 E2E）
> **维护者**: 测试团队

---

## 一、核心原则

### ⚠️ 最高优先级规则

| # | 规则 | 说明 | 违反后果 |
|---|------|------|---------|
| 1 | **禁止在E2E中调用后端API** | 禁止 `fetch()`、`request.post()`、`apiGet()` 等 | 变成API集成测试，不走用户真实路径 |
| 2 | **通过浏览器UI操作完成全链路** | 创建数据、查询数据、编辑数据、删除数据都必须通过页面操作 | 跳过任何一步 = 漏掉该环节的bug |
| 3 | **验证功能结果，不只是"不报错"** | 必须断言实际结果（表格内容、提示信息、数据变化） | "不报错"不等于"功能正确" |
| 4 | **每个测试独立完成完整旅程** | 测试内部自行创建所需数据，测试结束自行清理 | 共享状态导致级联失败 |
| 5 | **用 data-testid 定位元素** | 不用 CSS class | 产品样式变更不应导致测试失败 |
| 6 | **全链路真实环境** | 前端、后端、所有中间件必须全部启动 | Mock数据掩盖真实问题 |

### v3.0 vs v2.0 vs v1.0

| 维度 | v1.0（已废弃） | v2.0（已废弃） | v3.0（当前标准） |
|------|---------------|---------------|-----------------|
| 数据创建 | `fetch()` API | 回避创建，依赖已有数据 | **通过UI操作创建** |
| 验证深度 | 验证API响应 | 只验证"不报错" | **验证功能结果正确** |
| 测试路径 | API调用链路 | 只测异常边界 | **完整用户旅程** |
| 发现bug能力 | 中（能发现后端bug） | 低（回避了核心功能） | **高（全链路覆盖）** |
| 测试独立性 | beforeAll共享数据 | 每个测试独立但内容空 | **每个测试独立且内容完整** |

### v2.0 为什么被废弃

v2.0 的"探针式测试"理念存在根本性问题：
1. **"优先用已有数据"** → 新环境无数据时，排序/分页/选择/删除全部变成空测试
2. **"不报错就算通过"** → 搜索返回了错误结果也不会被发现
3. **"异常路径优先"** → 用户90%的时间走正常流程，正常流程才是最容易出bug的地方
4. **"减少测试代码量"** → 这是测试偷懒的理由，不是设计目标

**E2E的价值在于走用户真实路径，代码量大是正常的，不应该为了减少代码量而牺牲测试深度。**

---

## 二、环境要求

### 2.1 必须启动的服务

| 服务 | 端口 | 验证方式 |
|------|------|---------|
| PostgreSQL | 5432 | `pg_isready -h localhost -p 5432` |
| Redis | 6379 | `redis-cli ping` |
| 后端API | 8080 | `curl -s http://localhost:8080/actuator/health` |
| 前端 | 5173 | `curl -s http://localhost:5173` |

### 2.2 运行命令

```bash
# 运行所有E2E测试
npx playwright test

# 运行特定模块
npx playwright test e2e/standard/

# 调试模式
npx playwright test --ui

# 生成报告
npx playwright show-report
```

---

## 三、测试模型

### 概览

```
Layer 1: 页面冒烟      → 页面能打开，不白屏，不崩溃
Layer 2: 用户旅程测试    → 模拟真实用户操作，完成完整业务流程（核心）
Layer 3: 异常路径探测    → 快速连点、特殊字符、边界条件
```

### Layer 1: 页面冒烟

验证页面能打开、不白屏、不报错。每个可导航页面一个冒烟测试。

```typescript
test('物化模型页面能正常打开', async ({ page }) => {
  await page.goto('http://localhost:5173/standard/physical-model');
  await page.waitForLoadState('networkidle');

  await expect(page.getByTestId('materialization-page')).toBeVisible();
  await expect(page.getByTestId('physical-model-table')).toBeVisible();
});
```

### Layer 2: 用户旅程测试（核心）

**这是E2E测试的重点，投入最多的测试用例。**

每个页面至少覆盖以下用户旅程：

| 旅程 | 说明 | 必须 |
|------|------|------|
| 创建数据 | 通过UI操作完成新建流程，验证创建成功 | ✅ |
| 查看数据 | 验证表格正确显示创建的数据 | ✅ |
| 搜索/筛选 | 输入条件后验证搜索结果正确（不仅仅是"不报错"） | ✅ |
| 编辑数据（如适用） | 修改后验证更新成功 | 推荐 |
| 删除数据 | 删除后验证数据消失 | ✅ |
| 批量操作（如适用） | 选择多条后执行批量操作 | 推荐 |

#### 数据创建规范

**数据只能通过UI操作创建，禁止用API。**

```typescript
test('完整旅程：创建物理模型→验证表格→搜索→删除', async ({ page }) => {
  const testTableName = `e2e_test_${Date.now()}`;

  // Step 1: 打开页面
  await page.goto('http://localhost:5173/standard/physical-model');
  await page.waitForLoadState('networkidle');

  // Step 2: 通过UI创建数据
  await page.getByTestId('create-btn').click();
  await page.getByRole('menuitem', { name: '直接创建' }).click();

  // 填写表单（模拟真实用户操作）
  await page.getByPlaceholder('请输入表名').fill(testTableName);
  await page.getByRole('button', { name: '下一步' }).click();

  // 添加字段
  await page.getByRole('button', { name: '+ 添加字段' }).click();
  await page.locator('.field-table-row').last().locator('input').first().fill('id');
  await page.getByRole('button', { name: '下一步' }).click();

  // 确认创建
  await page.getByRole('button', { name: '确认创建' }).click();

  // Step 3: 验证创建成功（断言具体结果，不是"不报错"）
  await expect(page.locator('.el-message--success')).toBeVisible({ timeout: 5000 });
  const table = page.getByTestId('physical-model-table');
  await expect(table.locator('tbody')).toContainText(testTableName);

  // Step 4: 搜索验证（断言搜索结果正确）
  const searchInput = page.getByTestId('search-box').locator('input');
  await searchInput.fill(testTableName);
  await searchInput.press('Enter');
  await page.waitForLoadState('networkidle');
  // 断言：搜索后只显示匹配的结果
  await expect(table.locator('tbody tr')).toHaveCount(1);
  await expect(table.locator('tbody')).toContainText(testTableName);

  // Step 5: 清理数据（删除）
  await table.locator('tbody tr').filter({ hasText: testTableName })
      .locator('a', { hasText: '删除' }).click();
  await page.getByRole('button', { name: '确定' }).click();
  await expect(page.locator('.el-message--success')).toBeVisible({ timeout: 5000 });
});
```

#### 关键规则

| 规则 | 说明 |
|------|------|
| **测试数据名带 `e2e_test_` 前缀** | 便于识别和清理 |
| **每个测试独立创建和清理** | 禁止 beforeAll 共享数据 |
| **断言具体结果** | 不是 `not.toBeVisible('.error')`，而是 `toContainText('预期内容')` |
| **清理放在 afterEach 中** | 即使测试失败也要尝试清理 |
| **外部依赖不可用时 skip** | 如 Doris 不可用导致无法创建表，skip 而非 fail |

#### 数据清理模式

```typescript
test.afterEach(async ({ page }) => {
  // 测试结束后清理创建的数据（无论测试是否通过）
  try {
    const table = page.getByTestId('physical-model-table');
    const testRow = table.locator('tbody tr').filter({ hasText: testTableName });
    if (await testRow.isVisible({ timeout: 2000 })) {
      await testRow.locator('a', { hasText: '删除' }).click();
      await page.getByRole('button', { name: '确定' }).click();
    }
  } catch {
    // 清理失败不影响测试结果，只记录日志
  }
});
```

### Layer 3: 异常路径探测

在用户旅程测试基础上，额外探测异常场景：

```typescript
test('搜索XSS特殊字符不报错', async ({ page }) => {
  await page.goto('http://localhost:5173/standard/physical-model');
  await page.waitForLoadState('networkidle');

  const searchInput = page.getByTestId('search-box').locator('input');
  await searchInput.fill('<script>alert(1)</script>');
  await searchInput.press('Enter');
  await page.waitForLoadState('networkidle');

  await expect(page.locator('.el-message--error')).not.toBeVisible({ timeout: 3000 });
  await expect(page.getByTestId('materialization-page')).toBeVisible();
});

test('快速连续点击刷新按钮', async ({ page }) => {
  await page.goto('http://localhost:5173/standard/physical-model');
  await page.waitForLoadState('networkidle');

  const refreshBtn = page.getByTestId('refresh-btn');
  await refreshBtn.click();
  await refreshBtn.click();
  await refreshBtn.click();
  await page.waitForLoadState('networkidle');

  await expect(page.getByTestId('materialization-page')).toBeVisible();
});
```

---

## 四、元素定位规范

### 4.1 优先级

| 优先级 | 方式 | 示例 | 稳定性 |
|-------|------|------|-------|
| 1 | `data-testid` | `page.getByTestId('physical-model-table')` | 最高 |
| 2 | `getByRole` | `page.getByRole('button', { name: '确定' })` | 高 |
| 3 | `getByText` | `page.getByText('模型名称')` | 中 |
| 4 | `getByPlaceholder` | `page.getByPlaceholder('请输入表名')` | 中 |
| 5 | CSS class（仅限页面特有） | `page.locator('.field-table-row')` | 低 |

### 4.2 禁止的定位方式

```typescript
// ❌ 禁止：依赖元素层级结构
page.locator('.card > .card-header > .filter-bar > .el-select')

// ❌ 禁止：依赖颜色、字号等样式
page.locator('[style*="color: red"]')

// ❌ 禁止：使用 nth-child 等位置选择器
page.locator('tbody tr:nth-child(3)')
```

---

## 五、禁止事项

| 禁止行为 | 原因 | 正确做法 |
|---------|------|---------|
| ❌ 在E2E中用 `fetch()` 调用API | 不是用户真实行为 | 通过浏览器UI操作 |
| ❌ 只验证"不报错" | 回避了功能正确性验证 | 断言具体结果 |
| ❌ `beforeAll` 中创建测试数据 | 失败导致整个describe全挂 | 每个test独立创建 |
| ❌ 用CSS class定位元素 | 样式变更导致测试失败 | 用 `data-testid` |
| ❌ 用 `if (result)` 跳过断言 | 假性通过 | 直接 `expect()` |
| ❌ `test.skip()` 存在时报 completed | 虚假完成 | 报告 blocked |
| ❌ 测试间共享可变状态 | 级联失败 | 每个测试完全独立 |
| ❌ 测试结束不清理数据 | 污染后续测试 | afterEach 兜底清理 |

---

## 六、E2E vs 单元测试 vs API测试

三者是独立的测试类型，各司其职：

| 维度 | 单元测试 | API集成测试 | E2E测试 |
|------|---------|------------|---------|
| **负责验证** | 代码逻辑正确 | 后端接口正确 | 用户操作体验正确 |
| **运行环境** | JVM / Node.js | Node.js | 浏览器 |
| **测试框架** | JUnit / Vitest | Vitest | Playwright |
| **操作方式** | 调用方法 | `fetch()` / `request.post()` | 点击、输入、导航 |
| **数据准备** | Mock 对象 | API调用 | 通过UI操作 |
| **发现bug类型** | 逻辑bug、边界bug | 数据格式bug、权限bug | 交互bug、流程断裂、UI缺陷 |
| **规范位置** | `CODING_STANDARDS/04-testing/unit-testing.md` | `CODING_STANDARDS/04-testing/integration-testing.md` | 本文档 |

---

## 七、完成标准

### 测试质量判断

| 状态 | 含义 | 操作 |
|------|------|------|
| 全部 passed | 全链路验证通过 | 标记 completed |
| 部分 skipped | 有测试因依赖缺失被跳过 | 标记 **blocked**，向用户报告 |
| 部分 failed | 有测试失败 | 分析：产品bug还是测试bug |

### 判断失败来源

| 失败现象 | 问题来源 | 处理方式 |
|---------|---------|---------|
| 500错误/接口报错 | **产品bug** | 修产品代码 |
| 创建成功但表格不显示数据 | **产品bug** | 修产品代码（前后端数据流问题） |
| 搜索结果不正确 | **产品bug** | 修产品代码 |
| 元素找不到 | 可能是测试定位问题 | 检查 data-testid |
| 清理失败 | 测试基础设施问题 | 优化 afterEach |
| 超时 | 可能是产品性能bug | 检查接口响应时间 |

### 禁止假性通过

```typescript
// ❌ 假性通过
if (result) { expect(result).toBe(expected); }

// ❌ 假性通过（只验证不报错）
await expect(page.locator('.el-message--error')).not.toBeVisible();

// ✅ 真正验证
expect(result).toBeDefined();
await expect(table.locator('tbody')).toContainText('预期的数据');
```

---

## 八、API变更测试同步规则

### ⚠️ 强制要求

**任何后端API变更必须同步更新测试**

| 变更类型 | 必须更新的测试 |
|---------|---------------|
| 新增端点 | E2E测试 + 后端单元测试 + 前端API类型定义 |
| 修改参数 | E2E测试 + 前端类型定义 |
| 修改返回值 | E2E测试 + 后端单元测试 |
| 删除端点 | 删除对应E2E测试用例 + 删除前端调用 |

### 违反后果

- **PR不予合并**
- Story不能标记为 completed

### 执行时机

| 阶段 | 检查内容 | 负责人 |
|------|---------|--------|
| 开发前 | 创建API时同步创建测试骨架 | 开发者 |
| 提交PR | 自查测试是否更新 | 开发者 |
| 代码审查 | 检查测试覆盖是否完整 | 审查者 |
| CI流水线 | 自动运行测试，失败则阻止合并 | 自动化 |

### 测试骨架模板

新增API时，先创建测试骨架：

```typescript
test.skip('完整旅程：[功能名称]', async ({ page }) => {
  // TODO: 等待后端API实现
  // 步骤1: 打开页面
  // 步骤2: 通过UI操作调用新API
  // 步骤3: 验证结果
});
```

实现完成后：
1. 移除 `.skip`
2. 填充测试步骤
3. 确保测试通过

---

## 九、版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-03-22 | 初始版本，使用API创建数据 |
| v2.0 | 2026-04-04 | 探针式测试，废弃API创建数据 |
| v3.0 | 2026-04-05 | 用户旅程驱动，重写全部核心理念 |
| v3.1 | 2026-04-05 | 新增API变更测试同步规则 |
| v3.2 | 2026-04-09 | 新增 Element Plus 交互规范、表格数据匹配规范（实战总结） |

---

**最后更新**: 2026-04-09
**维护者**: 测试团队

---

## 十、Element Plus 组件交互规范（v3.2 新增）

> 本节来自 Story 7.5-10/7.5-11 E2E 开发的实战踩坑总结，耗费大量调试时间提炼。后续所有涉及 Element Plus 组件的 E2E 测试必须遵循。

### 10.1 el-select 下拉选择

**核心问题**：Element Plus el-select 的输入区域会被内部元素拦截点击事件（pointer-events），不能直接 `.click()` 打开下拉。下拉选项通过 `teleported` 传送到 `body` 底部，可能存在多个 popper 同时在 DOM 中（已关闭的 popper 仍然存在于 DOM，只是不可见）。

#### 正确的交互模式

```typescript
/**
 * Element Plus el-select 标准交互流程：
 * 1. 点击后缀图标（suffix icon）打开下拉
 * 2. 等待至少一个 [role="option"] 变为可见
 * 3. 用 isVisible() 逐个过滤可见选项（排除其他关闭的 popper 中的选项）
 * 4. 选择目标选项
 */
async function selectElOption(page: any, selectLocator: any, optionText?: string) {
  // Step 1: 点击后缀图标（不能直接点击 input/placeholder）
  const suffixIcon = selectLocator.locator('.el-select__suffix .el-icon, .el-select__suffix svg').first();
  if (await suffixIcon.isVisible({ timeout: 2000 })) {
    await suffixIcon.click();
  } else {
    await selectLocator.click();
  }
  await page.waitForTimeout(500);

  // Step 2: 等待下拉面板打开（用 waitForFunction 检测可见 option）
  await page.waitForFunction(() => {
    const options = document.querySelectorAll('[role="option"]');
    for (const opt of options) {
      if ((opt as HTMLElement).offsetParent !== null) return true;
    }
    return false;
  }, { timeout: 5000 }).catch(() => {});

  // Step 3: 收集可见选项（关键：必须逐个检查 isVisible）
  const allOptions = page.locator('[role="option"]');
  const optCount = await allOptions.count();
  const visibleOpts: { index: number; text: string }[] = [];
  for (let i = 0; i < optCount; i++) {
    const opt = allOptions.nth(i);
    if (await opt.isVisible()) {
      visibleOpts.push({ index: i, text: (await opt.textContent()) || '' });
    }
  }

  if (visibleOpts.length === 0) return false;

  // Step 4: 选择目标选项
  if (optionText) {
    const escaped = optionText.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const match = visibleOpts.find(v => new RegExp(escaped).test(v.text));
    if (match) {
      await allOptions.nth(match.index).click();
      return true;
    }
  }

  // 回退：选第一个非 e2e_test_ 选项
  const nonTest = visibleOpts.find(v => !v.text.startsWith('e2e_test_'));
  if (nonTest) {
    await allOptions.nth(nonTest.index).click();
    return true;
  }

  await allOptions.nth(visibleOpts[0].index).click();
  return true;
}
```

#### 常见错误与正确做法

| ❌ 错误做法 | ✅ 正确做法 | 原因 |
|------------|-----------|------|
| `selectLocator.click()` | 点击 `.el-select__suffix svg` | placeholder 拦截 pointer events |
| `page.locator('.el-select-dropdown__item')` | `page.locator('[role="option"]')` | Element Plus 使用 `[role="option"]` 而非 CSS class |
| 直接选第一个 `[role="option"]` | 逐个检查 `isVisible()` | DOM 中可能有多个已关闭的 popper，其选项不可见 |
| `elSelect.locator('input').inputValue()` | `elSelect.textContent()` | el-select 的内部 input 不暴露 v-model 值 |
| 在选项上用 CSS class 选择器 | 用 `isVisible()` + `textContent()` 过滤 | Element Plus 版本升级可能改变 DOM 结构 |

#### 原生 `<select>` vs Element Plus `el-select`

| 维度 | 原生 `<select>` | Element Plus `el-select` |
|------|----------------|------------------------|
| 选项选择 | `selectOption({ index: 1 })` | 点击后缀图标 + 选 `[role="option"]` |
| 获取当前值 | `inputValue()` | `textContent()` |
| 选项 DOM 位置 | `<option>` 在 `<select>` 内部 | `[role="option"]` 在 teleported popper 中 |
| 示例页面 | Reports.vue 的任务筛选 | 向导页的所有下拉、规则选择器 |

### 10.2 表格数据匹配（truncateText 问题）

**核心问题**：项目中 17+ 个 Vue 组件使用 `truncateText(text, maxLen)` 截断表格中的长文本。`textContent()` 返回截断后的文本，不是原始值。所有被截断的列都通过 `:title` 属性保留了完整值。

#### 受影响的组件和截断长度

| 组件 | 截断字段 | 截断长度 | title 属性位置 |
|------|---------|---------|--------------|
| quality/Tasks.vue | taskName | 20 | `<a>` |
| quality/Tasks.vue | targetTable | 16 | `<td>` |
| quality/Reports.vue | taskName | 20 | `<td>` |
| quality/Rules.vue | ruleName | 15 | `<span>` |
| integration/SyncTasks.vue | taskName | 15 | `<a>` |
| integration/SyncTasks.vue | sourceTable | 12 | `<span>` |
| integration/SyncTasks.vue | targetTable | 12 | `<span>` |
| standard/Models.vue | modelName | 20 | `<a>` |
| task/TaskManage.vue | name | 15 | `<a>` |
| task/TaskInstance.vue | taskName | 15 | `<span>` |
| task/TaskScheduling.vue | scheduleName | 20 | `<span>` |

#### 正确的表格行匹配模式

```typescript
/**
 * 在表格中查找包含指定任务名的行
 * 关键：不能直接用 textContent() 匹配完整名称，因为显示被截断
 */
const taskPrefix = taskName.substring(0, 20); // 取前缀做快速过滤
const rows = page.locator('.table tbody tr');
const count = await rows.count();

for (let i = 0; i < count; i++) {
  const row = rows.nth(i);
  const text = await row.textContent();
  if (!text?.includes(taskPrefix)) continue; // 快速过滤

  // 精确匹配：用 title 属性获取完整值
  const nameLink = row.locator('[data-testid="link-task-name"]').first();
  const titleAttr = await nameLink.getAttribute('title').catch(() => '');
  if (titleAttr === taskName) {
    // 找到目标行，执行操作
    break;
  }
}
```

#### 常见错误

| ❌ 错误做法 | ✅ 正确做法 | 原因 |
|------------|-----------|------|
| `text?.includes(fullTaskName)` | 用 `title` 属性匹配 | `textContent()` 返回截断文本 |
| `rows.filter({ hasText: taskName })` | 前缀过滤 + `title` 精确匹配 | Playwright filter 也受截断影响 |
| 假设行在第一页 | 提交后等待跳转完成 | 新创建的数据可能在列表首行，但需要等待加载 |

### 10.3 向导页面交互规范

**核心问题**：多步骤向导页面的步骤导航（`.wizard-page .nav-item`）点击行为不可靠，必须使用底部的 "下一步" 按钮。

#### 正确的向导交互流程

```typescript
// Step 1: 填写基本信息
await page.locator('[data-testid="input-task-name"]').fill(taskName);
await page.locator('[data-testid="btn-next"]').click();  // 用 btn-next，不用 nav-item

// Step 2: 选择目标表（el-select 交互）
await selectElOption(page, page.locator('[data-testid="select-target-table"]'));
await page.locator('[data-testid="btn-next"]').click();

// Step 3: 添加规则
await selectElOption(page, page.locator('[data-testid="select-field-rule-field"]'));
await selectElOption(page, page.locator('[data-testid="select-field-rule"]'));
await page.locator('[data-testid="btn-next"]').click();

// Step 4: 提交
await page.locator('[data-testid="btn-submit"]').click();
await page.waitForLoadState('networkidle');
```

#### 常见错误

| ❌ 错误做法 | ✅ 正确做法 | 原因 |
|------------|-----------|------|
| `.wizard-page .nav-item:nth-child(2).click()` | `[data-testid="btn-next"]` | nav-item 点击可能跳过表单验证 |
| `.wizard-page .nav-item:last-child.click()` | 逐步点击 btn-next | 跳步导致中间数据未加载 |
| `page.locator('.nav-item.active').last().click()` | 每步用 btn-next | active 状态判断不可靠 |

### 10.4 Playwright 断言陷阱

| ❌ 错误做法 | ✅ 正确做法 | 原因 |
|------------|-----------|------|
| `expect(locators).toContainText('X')` | `allTextContents().join('\|')` 模式 | 多元素 locator 触发 strict mode violation |
| `page.locator('.el-select-dropdown__item')` | `page.locator('[role="option"]')` | Element Plus 使用 ARIA role 而非特定 CSS class |
| `await locator.count() > 0` 后直接操作 | `await locator.first().isVisible()` | count 返回 DOM 中所有匹配（含隐藏的） |

### 10.5 调试技巧

当 E2E 测试反复 skip 或 fail 时：

1. **用 Playwright MCP 浏览器手动走一遍流程**，确认每一步的 DOM 结构
2. **在关键步骤加 `console.log`**，输出 `page.url()`、行数、选项数
3. **检查 `textContent()` vs `title` 属性**，确认是否有截断
4. **检查 el-select 的 DOM 结构**，确认选项的实际定位方式
5. **Docker 容器使用最新代码**：`mvn package` → `docker compose build` → `docker compose up -d`

---

## 十一、E2E 测试文件模板（v3.2 新增，v4.0 更新）

基于 v3.0 规范和 v3.2 实战经验，提供标准测试文件模板。

> **v4.0 注意**：此模板适用于 **L4 核心旅程测试**（跨模块用户旅程）。单模块功能测试不再使用浏览器 E2E，请改用：
> - 后端功能：L3 API 集成测试（`@SpringBootTest` + `MockMvc`），见 `integration-testing.md`
> - 前端功能：L2 Vitest 组件测试，见 `docs/e2e-testing-optimization-proposal.md` 第 4 节

```typescript
/**
 * [跨模块旅程名称] E2E 测试（v4.0 核心旅程）
 *
 * 覆盖的跨模块流程：
 * [描述从哪个模块到哪个模块的完整用户旅程]
 *
 * 注意：单模块功能验证由 L3 API 集成测试和 L2 Vitest 组件测试负责，
 * 此文件只验证跨模块的用户旅程和浏览器环境。
 *
 * @version 4.0
 * @since YYYY-MM-DD
 */

import { test, expect } from '@playwright/test';

const PAGE_URL = '/module/page';
const CREATE_URL = '/module/page/create';
const E2E_PREFIX = 'e2e_test_';
const TIMESTAMP = new Date().toISOString().replace(/[:.]/g, '-').substring(0, 19);
const createdResources: string[] = [];

// ---- el-select 交互（复用 10.1 的 selectElOption 函数）----
async function selectElOption(page: any, selectLocator: any, optionText?: string) {
  /* 参见第 10.1 节 */
}

// ---- 清理函数（afterEach 兜底）----
async function cleanupTestData(page: any) {
  for (const name of createdResources) {
    try {
      await page.goto(PAGE_URL);
      await page.waitForLoadState('networkidle');
      await page.waitForTimeout(500);
      // 用前缀 + title 匹配目标行（参见 10.2）
      const prefix = name.substring(0, 20);
      const rows = page.locator('.table tbody tr');
      const count = await rows.count();
      for (let i = 0; i < count; i++) {
        const row = rows.nth(i);
        const text = await row.textContent();
        if (!text?.includes(prefix)) continue;
        const titleAttr = await row.locator('[data-testid="link-task-name"], [title]').first()
          .getAttribute('title').catch(() => '');
        if (titleAttr === name) {
          const deleteBtn = row.locator('[data-testid="btn-delete"]').first();
          if (await deleteBtn.isVisible({ timeout: 1000 })) {
            await deleteBtn.click();
            const confirmBtn = page.locator('.el-message-box .el-button--primary').first();
            if (await confirmBtn.isVisible({ timeout: 2000 })) {
              await confirmBtn.click();
              await page.waitForLoadState('networkidle');
            }
          }
          break;
        }
      }
    } catch {
      // 清理失败不影响测试结果
    }
  }
  createdResources.length = 0;
}

// ---- 测试套件 ----
test.describe('[功能名称]用户旅程（v3.0）', () => {

  test.afterEach(async ({ page }) => {
    await cleanupTestData(page);
  });

  test.describe('前置条件', () => {
    test('页面应能正常加载', async ({ page }) => {
      await page.goto(PAGE_URL);
      await page.waitForLoadState('networkidle');
      await expect(page.locator('[data-testid="page-container"]')).toBeVisible();
    });
  });

  test.describe('完整用户旅程', () => {
    test('应能创建、查看、删除', async ({ page }) => {
      const name = `${E2E_PREFIX}item_${TIMESTAMP}`;
      createdResources.push(name);

      // 创建
      await page.goto(CREATE_URL);
      await page.waitForLoadState('networkidle');
      await page.locator('[data-testid="input-name"]').fill(name);
      await page.locator('[data-testid="btn-next"]').click();
      // ... 更多步骤
      await page.locator('[data-testid="btn-submit"]').click();
      await page.waitForLoadState('networkidle');
      await page.waitForTimeout(3000);

      // 验证（用 title 属性匹配，参见 10.2）
      // ...
    });
  });
});
```

---

## 十二、v4.0 四层测试体系（2026-04-10 新增）

> 本节来自 Epic 7.5 回顾会的优化方案。核心变化：E2E 从"覆盖每个 Story"调整为"只做跨模块核心旅程"，单模块功能验证由 API 集成测试和 Vitest 组件测试替代。
>
> 详细方案见 `docs/e2e-testing-optimization-proposal.md`

### 12.1 四层测试体系总览

```
┌─────────────────┐
│  L4 浏览器 E2E  │  ← 3-5 个核心旅程 + 页面冒烟（本规范）
│  (Playwright)   │
├─────────────────┤
│  L3 API 集成测试 │  ← 覆盖所有 Controller（integration-testing.md）
│ (Spring Boot)   │
├─────────────────┤
│  L2 组件测试     │  ← 覆盖关键前端组件（新增，见下方）
│  (Vitest)       │
├─────────────────┤
│  L1 后端单元测试 │  ← DomainService 业务逻辑（unit-testing.md）
│  (JUnit 5)      │
└─────────────────┘
```

### 12.2 E2E 测试职责调整

| 职责 | 归属 | 说明 |
|------|------|------|
| 后端 CRUD 验证 | **L3 API 集成测试** | MockMvc 更快更稳定 |
| 表单校验验证 | **L2 Vitest 组件测试** | 组件测试更快更可靠 |
| 页面渲染验证 | **L2 Vitest 组件测试** | 组件测试无 DOM 陷阱 |
| 页面冒烟 | **L4 E2E** | 验证页面能打开不崩溃 |
| **跨模块用户旅程** | **L4 E2E（唯一）** | 验证模块间衔接 |
| **路由/导航流程** | **L4 E2E（唯一）** | 验证路由跳转正确 |
| **浏览器兼容性** | **L4 E2E（唯一）** | 验证真实浏览器环境 |

### 12.3 v4.0 规则变更

| 原 v3.0 规则 | v4.0 调整 | 原因 |
|-------------|----------|------|
| 每个 Story 必须有 E2E 测试 | 每个 Story 必须有对应的自动化测试（API 集成测试 / Vitest 组件测试 / E2E 旅程之一） | AI 团队实测：浏览器 E2E 对单模块 Story 的 ROI 极低 |
| API 变更必须更新 E2E | API 变更必须更新 **L3 API 集成测试**；如涉及核心旅程则同步更新 E2E | L3 是 API 验证的主力 |
| E2E 测试禁止调用后端 API | **保持**：测试内部仍禁止；新增 globalSetup 例外 | 平衡真实性与效率 |
| 有 skip 不算 done | 扩展到所有测试层级（API、Vitest、E2E） | 统一标准 |

### 12.4 E2E 测试分类（v4.0）

#### A. 页面冒烟测试

每个可导航页面一个冒烟测试，验证页面能打开不崩溃：

```typescript
test('数据质量规则页面能正常打开', async ({ page }) => {
  await page.goto('/quality/rules')
  await page.waitForLoadState('networkidle')
  await expect(page.getByTestId('quality-rules-page')).toBeVisible()
})
```

#### B. 核心旅程测试（3-5 个）

只覆盖跨模块用户旅程，不重复单模块功能：

```typescript
test('数据质量旅程: 创建规则→创建任务→执行→查看报告', async ({ page }) => {
  // 跨越 质量规则 → 质量任务 → 质量报告 三个模块
})
```

### 12.5 数据清理：globalSetup 机制（v4.0 新增）

**原则**：E2E 测试内部不调用后端 API（保持 v3.0 规则 1）。

**新增例外**：在 `globalSetup` 阶段可以通过 API 清理上次残留的测试数据。这不是测试逻辑，而是测试基础设施。

```typescript
// e2e/global-setup.ts
import { FullConfig, request } from '@playwright/test'

async function globalSetup(config: FullConfig) {
  const apiContext = await request.newContext({
    baseURL: 'http://localhost:8080'
  })
  try {
    await apiContext.delete('/api/internal/test-cleanup?prefix=e2e_test_')
  } catch {
    console.warn('测试数据清理失败')
  }
  await apiContext.dispose()
}

export default globalSetup
```

> 后端需新增 `TestCleanupController`（仅在 `profile=test` 时激活）。

### 12.6 Story 完成标准（v4.0 更新）

Story 的测试完成标准从"必须有 E2E"调整为：

| 变更类型 | 必须的测试 |
|---------|-----------|
| 后端新功能/修改 | L1 单元测试 + L3 API 集成测试 |
| 前端新功能/修改 | L2 Vitest 组件测试 |
| 跨模块功能 | L4 E2E 核心旅程（如适用） |
| 修复 Bug | 覆盖该 Bug 的回归测试（任一层级） |

### 12.7 E2E 测试标签规范（v4.0 新增）

所有 E2E 测试文件必须添加优先级标签，用于选择性执行：

| 标签 | 含义 | 文件数 | 执行策略 |
|------|------|--------|---------|
| `@p0` | 核心跨模块旅程 | ~5 | 每次提交必跑 |
| `@p1` | 模块级用户旅程 | ~33 | 每日构建 |
| `@p2` | 冒烟/无障碍/视觉回归 | ~25 | 每周构建 |

**使用方式**：
```typescript
test.describe('@p0 质量完整旅程', () => { ... });
test.describe('@p1 数据标准基本CRUD', () => { ... });
test.describe('@p2 无障碍性检查', () => { ... });
```

**执行命令**：
```bash
npx playwright test --grep @p0          # 只跑 P0（发布阻断）
npx playwright test --grep-invert @p2   # 跑 P0 + P1（日常开发）
npx playwright test                       # 全量（每周构建）
```

---

## 十三、版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-03-22 | 初始版本，使用API创建数据 |
| v2.0 | 2026-04-04 | 探针式测试，废弃API创建数据 |
| v3.0 | 2026-04-05 | 用户旅程驱动，重写全部核心理念 |
| v3.1 | 2026-04-05 | 新增API变更测试同步规则 |
| v3.2 | 2026-04-09 | 新增 Element Plus 交互规范、表格数据匹配规范 |
| **v4.0** | **2026-04-10** | **四层测试体系、E2E 聚焦核心旅程、API 测试替代单模块 E2E、globalSetup 清理** |

---

**最后更新**: 2026-04-10
**维护者**: 测试团队
