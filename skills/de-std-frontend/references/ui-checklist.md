# UI 开发检查清单

> 本清单确保所有前端开发符合原型设计和设计规范，避免出现样式不一致的问题。

---

## 📋 目录

- [开发前](#开发前)
- [开发中](#开发中)
- [代码审查前](#代码审查前)
- [提交前](#提交前)

---

## 开发前

### 必读文档
- [ ] 已阅读 `docs/prototypes/index.html` 首页原型
- [ ] 已阅读 `docs/prototypes/data-standards.html` 数据页原型
- [ ] 已阅读 `docs/prototypes/DESIGN_DECISIONS.md` 设计决策
- [ ] 已阅读 `docs/plans/design-system.md` 设计系统规范
- [ ] 已阅读 `docs/plans/page-design-guidelines.md` 页面设计规范

### 组件确认
- [ ] 确认所需组件已在 `src/components/common/` 中存在
- [ ] 确认组件的 props 和用法
- [ ] 如需新组件，先查阅设计规范再实现

---

## 开发中

### 样式规范（绝对禁止违反）

#### ❌ 绝对禁止

1. **禁止使用渐变背景**（除原型明确要求外）
   ```css
   /* 错误 */
   background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

   /* 正确 */
   background: var(--bg-primary);
   ```

2. **禁止使用大于 4px 的圆角**
   ```css
   /* 错误 */
   border-radius: 12px;
   border-radius: 16px;

   /* 正确 */
   border-radius: 4px;
   ```

3. **禁止使用花哨的阴影**
   ```css
   /* 错误 */
   box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
   box-shadow: 0 10px 40px rgba(102, 126, 234, 0.3);

   /* 正确 */
   box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
   box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
   ```

4. **禁止硬编码颜色值**
   ```css
   /* 错误 */
   color: #2563eb;
   background: #ffffff;

   /* 正确 */
   color: var(--primary-blue);
   background: var(--bg-primary);
   ```

5. **禁止使用过大的字号**
   - 统计卡片值：**18px**（不是 32px）
   - 页面标题：**16px**（不是 24px）
   - 卡片标题：**15px**（不是 18px）
   - 表头：**13px**，不加粗（font-weight: 500）

6. **禁止在表格列宽中使用固定像素值**
   ```html
   <!-- 错误 -->
   <th style="width: 150px;">模型名称</th>

   <!-- 正确 -->
   <th style="width: 14%;">模型名称</th>
   ```

#### ✅ 必须遵守

1. **组件使用**
   - [ ] 优先使用 `src/components/common/` 下的组件
   - [ ] StatCard 用于统计展示
   - [ ] ActionBar 用于操作栏
   - [ ] CardContainer 用于内容容器

2. **间距系统**
   - [ ] 使用 8px 基准（4px/8px/12px/16px/24px）
   - [ ] 卡片之间间距：12px
   - [ ] 内容区内边距：16px 20px

3. **布局结构**
   - [ ] 遵循三段式布局：统计卡片区 → 操作区 → 数据区
   - [ ] 操作和数据分离（操作在 ActionBar，数据在 CardContainer）

4. **CSS 变量**
   - [ ] 主色：`--primary-blue` (#2563eb)
   - [ ] 背景：`--bg-primary` (#ffffff)
   - [ ] 边框：`--border-color` (#e2e8f0)
   - [ ] 文字：`--text-primary` (#1e293b)

5. **表格列宽**
   - [ ] 所有列表页表格列宽使用百分比
   - [ ] 禁止使用固定像素值（px）
   - [ ] 长文本列（如描述）使用较大百分比
   - [ ] 固定内容列（如状态、可空）使用较小百分比

6. **表格单元格禁止换行**
   - [ ] 所有表格单元格使用 `white-space: nowrap`
   - [ ] 时间列、操作列禁止换行
   - [ ] 长文本使用 `:title` 属性显示完整内容

7. **操作列固定**
   - [ ] 操作列使用 `position: sticky; right: 0` 固定右侧
   - [ ] 操作列添加 `border-left: 1px solid #e2e8f0` 分隔线
   - [ ] 滚动时操作列始终可见

8. **表格数据左对齐**
   - [ ] 所有列表页表格数据默认左对齐
   - [ ] 移除 `align="center"` 和 `align="right"` 属性
   - [ ] 标签类数据（状态、类型）也左对齐
   - [ ] 仅数值类数据（如金额、数量）可考虑右对齐

9. **data-testid 可测试性（E2E 必需）**
   - [ ] 页面容器元素添加 `data-testid="{模块}-{页面}-page"`
   - [ ] 所有操作按钮添加 `data-testid="btn-{动作}"`
   - [ ] 所有表单输入添加 `data-testid="input-{名称}"` 或 `data-testid="select-{名称}"`
   - [ ] 表格行操作链接添加 `data-testid="btn-{动作}"`
   - [ ] 表格名称列链接添加 `data-testid="link-{名称}"` + `:title` 属性（配合 truncateText）
   - [ ] 筛选/搜索组件添加 `data-testid="filter-{名称}"` 或 `data-testid="input-search"`
   - [ ] 弹窗/对话框添加 `data-testid="{功能}-modal"`
   - [ ] 图表组件添加 `data-testid="{类型}-chart"`
   - [ ] 导出/下载按钮添加 `data-testid="export-{名称}"` 或 `data-testid="download-{名称}"`
   - [ ] 详细规范参见 [组件设计规范 - data-testid 章节](./component-design.md)

10. **truncateText 复用（禁止重复定义）**
    - [ ] 使用公共函数 `import { truncateText } from '@/api/transformers'`
    - [ ] 禁止在组件内部重新定义 `truncateText` 函数
    - [ ] 截断文本列必须同时添加 `:title` 属性保留完整值

---

## 代码审查前

### 自动检查
- [ ] 运行 `npm run lint:style` 无错误
- [ ] 运行 `npm run lint:vue` 无错误
- [ ] 运行 `npm run type-check` 无错误

### 视觉对比
- [ ] 对比原型设计截图
- [ ] 检查圆角是否为 4px
- [ ] 检查背景是否为白色（无渐变）
- [ ] 检查阴影是否轻微
- [ ] 检查字号是否符合规范

### 组件检查
- [ ] StatCard 组件使用正确
- [ ] ActionBar 组件使用正确
- [ ] 没有内联样式覆盖组件样式
- [ ] 没有使用 `!important`

### data-testid 检查（E2E 测试必需）
- [ ] 页面容器有 `data-testid`
- [ ] 所有交互元素（按钮、输入框、下拉）有 `data-testid`
- [ ] 表格行操作和名称列有 `data-testid`
- [ ] 弹窗/对话框有 `data-testid`
- [ ] 截断文本列有 `:title` 属性
- [ ] `truncateText` 引用自公共模块，非局部定义
- [ ] 运行 E2E 测试验证定位正确（如已编写）

---

## 提交前

### 文档更新
- [ ] 如新增组件，已更新组件文档
- [ ] 如修改样式，已确认与原型一致
- [ ] 已更新相关 Story（如使用 Storybook）

### 团队审查
- [ ] 请至少一名团队成员进行 UI Review
- [ ] 准备好原型截图和实现截图对比
- [ ] 确认所有检查项通过

---

## 🚨 常见错误对照

| 问题 | 错误示例 | 正确示例 |
|------|---------|---------|
| 渐变背景 | `linear-gradient(135deg, #667eea 0%, #764ba2 100%)` | `var(--bg-primary)` |
| 大圆角 | `border-radius: 12px` 或 `16px` | `border-radius: 4px` |
| 花哨阴影 | `box-shadow: 0 4px 15px rgba(...)` | `box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04)` |
| 大字号 | `font-size: 32px` | `font-size: 18px` |
| 硬编码颜色 | `color: #2563eb` | `color: var(--primary-blue)` |
| 加粗表头 | `font-weight: 600` 或 `700` | `font-weight: 500` |
| 固定列宽 | `width: 150px` | `width: 14%` |
| 单元格换行 | 无 `white-space` | `white-space: nowrap` |
| 操作列滚动消失 | 无固定 | `position: sticky; right: 0` |
| 列数据居中/右对齐 | `align="center"` | 默认左对齐，移除 align 属性 |
| 缺少 data-testid | 按钮无 data-testid | `data-testid="btn-create-task"` |
| 重复定义 truncateText | 组件内部定义函数 | `import { truncateText } from '@/api/transformers'` |
| 截断文本无 title | `<td>{{ truncateText(name, 20) }}</td>` | `<td :title="name">{{ truncateText(name, 20) }}</td>` |

---

## 📚 参考文档

- [原型设计](../../../docs/prototypes/)
- [设计系统](../../../docs/plans/design-system.md)
- [页面设计规范](../../../docs/plans/page-design-guidelines.md)
- [前端编码规范](./README.md)

---

**最后更新**：2026-03-29
**维护者**：前端团队
