# 样式/CSS规范

> 本规范定义CSS样式的使用规范，包括CSS命名、主题系统、响应式设计和动画效果。
> **重要**：本规范遵循 `ui-checklist.md` 中的 UI 开发检查清单，不得违反其核心原则。

---

## 📋 目录

- [⚠️ UI 开发强制规范](#️-ui-开发强制规范)
- [📐 标准页面布局结构](#️-标准页面布局结构)
- [CSS命名规范](#css命名规范)
- [主题系统](#主题系统)
- [颜色使用](#颜色使用)
- [间距系统](#间距系统)
- [文字排版](#文字排版)
- [响应式设计](#响应式设计)
- [动画效果](#动画效果)
- [组件样式](#组件样式)
- [工具类](#工具类)
- [最佳实践](#最佳实践)

---

## ⚠️ UI 开发强制规范

### 绝对禁止（违反将导致代码审查失败）

#### 1. 禁止使用渐变背景

**❌ 错误**：
```css
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
background: linear-gradient(135deg, #4066E5 0%, #6680EE 100%);
```

**✅ 正确**：
```css
background: var(--bg-primary);
```

#### 2. 禁止使用大于 4px 的圆角

**❌ 错误**：
```css
border-radius: 12px;
border-radius: 16px;
```

**✅ 正确**：
```css
border-radius: 4px;
```

#### 3. 禁止使用花哨的阴影

**❌ 错误**：
```css
box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
box-shadow: 0 10px 40px rgba(102, 126, 234, 0.3);
```

**✅ 正确**：
```css
box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
```

#### 4. 禁止硬编码颜色值

**❌ 错误**：
```css
color: #2563eb;
background: #ffffff;
```

**✅ 正确**：
```css
color: var(--primary-blue);
background: var(--bg-primary);
```

#### 5. 禁止使用过大的字号

**❌ 错误**：
```css
.stat-card__value { font-size: 32px; }
.page-title { font-size: 24px; }
```

**✅ 正确**：
```css
.stat-card__value { font-size: 18px; }
.page-title { font-size: 16px; }
```

### 必须遵守

1. **开发前必读**：
   - 查看原型设计：`docs/prototypes/index.html`
   - 查看设计系统：`docs/plans/design-system.md`
   - 查看页面规范：`docs/plans/page-design-guidelines.md`

2. **使用组件库组件**：
   - StatCard：`src/components/common/StatCard.vue`
   - ActionBar：`src/components/common/ActionBar.vue`

3. **CSS 变量**：
   - 主色：`--primary-blue` (#2563eb)
   - 背景：`--bg-primary` (#ffffff)
   - 边框：`--border-color` (#e2e8f0)

---

## CSS命名规范

---

## CSS命名规范

### BEM命名法

**BEM** = Block（块） + Element（元素） + Modifier（修饰符）

```css
/* Block */
.page-header { }

/* Element */
.page-header__title { }
.page-header__description { }

/* Modifier */
.page-header--enhanced { }
.action-bar--compact { }
```

### 命名规则

| 类型 | 格式 | 示例 |
|------|------|------|
| Block | `block-name` | `page-header`, `action-bar` |
| Element | `block-name__element` | `page-header__title` |
| Modifier | `block-name--modifier` | `page-header--enhanced` |
| Utility | `utility-name` | `text-primary`, `mt-2` |

### 组件命名示例

```vue
<template>
  <!-- 组件名使用PascalCase -->
  <UserCard />

  <!-- CSS类名使用kebab-case -->
  <div class="user-card user-card--large">
    <div class="user-card__avatar">
      <img src="avatar.jpg" alt="Avatar" />
    </div>
    <div class="user-card__info">
      <h3 class="user-card__name">张三</h3>
      <p class="user-card__email">zhangsan@example.com</p>
    </div>
  </div>
</template>

<style scoped>
.user-card {
  /* Block样式 */
}

.user-card__avatar {
  /* Element样式 */
}

.user-card--large {
  /* Modifier样式 */
}
</style>
```

---

## 主题系统

### CSS变量定义

```css
/* src/styles/theme.css */
:root {
  /* 品牌色 */
  --brand-primary: #4066E5;
  --brand-primary-light: #6680EE;
  --brand-primary-dark: #2B4BC4;
  --brand-success: #34A853;
  --brand-warning: #FBBC04;
  --brand-error: #EA4335;

  /* 情感色 */
  --emotion-primary: #FF6B6B;
  --emotion-secondary: #4ECDC4;
  --emotion-accent: #FFE66D;

  /* 文字色 */
  --text-primary: #202124;
  --text-secondary: #5F6368;
  --text-hint: #9AA0A6;

  /* 背景色 */
  --bg-primary: #FFFFFF;
  --bg-secondary: #F8F9FA;
  --bg-tertiary: #F1F3F4;

  /* 边框色 */
  --border-color: #DADCE0;
  --border-hover: #BDC1C6;

  /* 阴影 */
  --shadow-sm: 0 1px 2px 0 rgba(60, 64, 67, 0.3);
  --shadow-md: 0 4px 6px 0 rgba(60, 64, 67, 0.3);
  --shadow-lg: 0 10px 20px 0 rgba(60, 64, 67, 0.3);

  /* 动画时长 */
  --transition-fast: 150ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-normal: 300ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-slow: 500ms cubic-bezier(0.4, 0, 0.2, 1);

  /* 布局 */
  --content-max-width: 1440px;
  --spacing-unit: 8px;
}
```

### 暗色主题

```css
/* 暗色主题变量 */
[data-theme="dark"] {
  --brand-primary: #6680EE;
  --text-primary: #FFFFFF;
  --text-secondary: #BDC1C6;
  --text-hint: #9AA0A6;

  --bg-primary: #1F1F1F;
  --bg-secondary: #2D2D2D;
  --bg-tertiary: #3D3D3D;

  --border-color: #3D3D3D;
  --border-hover: #4D4D4D;

  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.5);
  --shadow-md: 0 4px 6px 0 rgba(0, 0, 0, 0.5);
  --shadow-lg: 0 10px 20px 0 rgba(0, 0, 0, 0.5);
}
```

### 主题切换

```typescript
// stores/theme.ts
import { defineStore } from 'pinia'

export const useThemeStore = defineStore('theme', {
  state: () => ({
    isDark: false
  }),

  actions: {
    toggleTheme() {
      this.isDark = !this.isDark
      document.documentElement.setAttribute(
        'data-theme',
        this.isDark ? 'dark' : 'light'
      )
    }
  }
})
```

---

## 颜色使用

### 主色调使用

```css
/* 主要操作 */
.btn-primary {
  background-color: var(--brand-primary);
  color: white;
}

/* 成功状态 */
.status-success {
  color: var(--brand-success);
}

/* 警告状态 */
.status-warning {
  color: var(--brand-warning);
}

/* 错误状态 */
.status-error {
  color: var(--brand-error);
}
```

### 渐变色使用

```css
/* 主色渐变 */
.gradient-primary {
  background: linear-gradient(135deg, #4066E5 0%, #6680EE 100%);
}

/* 成功渐变 */
.gradient-success {
  background: linear-gradient(135deg, #10B981 0%, #34A853 100%);
}

/* 警告渐变 */
.gradient-warning {
  background: linear-gradient(135deg, #F59E0B 0%, #FBBC04 100%);
}

/* 危险渐变 */
.gradient-danger {
  background: linear-gradient(135deg, #EF4444 0%, #EA4335 100%);
}
```

### 质量评分颜色

```css
/* 优秀 - 90-100 */
.quality-excellent { color: #10B981; }

/* 良好 - 80-89 */
.quality-good { color: #34A853; }

/* 一般 - 60-79 */
.quality-fair { color: #FBBC04; }

/* 较差 - 0-59 */
.quality-poor { color: #EA4335; }
```

---

## 间距系统

### 间距单位

基于 **8px** 基准的间距系统：

| 变量 | 值 | 用途 |
|------|-----|------|
| `--spacing-1` | 4px | 微小间距 |
| `--spacing-2` | 8px | 基础间距 |
| `--spacing-3` | 12px | 小间距 |
| `--spacing-4` | 16px | 中间距 |
| `--spacing-5` | 20px | 大间距 |
| `--spacing-6` | 24px | 超大间距 |
| `--spacing-8` | 32px | 特大间距 |

### 间距工具类

```css
/* Margin */
.mt-1 { margin-top: 8px; }
.mt-2 { margin-top: 16px; }
.mt-3 { margin-top: 24px; }
.mt-4 { margin-top: 32px; }

.mb-1 { margin-bottom: 8px; }
.mb-2 { margin-bottom: 16px; }
.mb-3 { margin-bottom: 24px; }
.mb-4 { margin-bottom: 32px; }

/* Padding */
.p-1 { padding: 8px; }
.p-2 { padding: 16px; }
.p-3 { padding: 24px; }
.p-4 { padding: 32px; }
```

---

## 文字排版

### 字体大小

```css
/* 字体大小系统 */
.text-xs { font-size: 12px; }
.text-sm { font-size: 14px; }
.text-base { font-size: 16px; }
.text-lg { font-size: 18px; }
.text-xl { font-size: 20px; }
.text-2xl { font-size: 24px; }
.text-3xl { font-size: 30px; }
```

### 字体粗细

```css
.font-normal { font-weight: 400; }
.font-medium { font-weight: 500; }
.font-semibold { font-weight: 600; }
.font-bold { font-weight: 700; }
```

### 文字颜色

```css
.text-primary { color: var(--text-primary); }
.text-secondary { color: var(--text-secondary); }
.text-hint { color: var(--text-hint); }
.text-placeholder { color: #c0c4cc; }
```

### 行高

```css
.leading-tight { line-height: 1.25; }
.leading-normal { line-height: 1.5; }
.leading-relaxed { line-height: 1.75; }
```

---

## 响应式设计

### 断点系统

```css
/* 断点定义 */
--breakpoint-xs: 480px;
--breakpoint-sm: 640px;
--breakpoint-md: 768px;
--breakpoint-lg: 1024px;
--breakpoint-xl: 1280px;
--breakpoint-2xl: 1536px;
```

### 响应式类

```css
/* 移动端隐藏 */
.hidden-xs {
  display: none;
}

@media (min-width: 640px) {
  .hidden-xs {
    display: block;
  }
}

/* 平板以下隐藏 */
.hidden-sm-and-down {
  display: none;
}

@media (min-width: 768px) {
  .hidden-sm-and-down {
    display: block;
  }
}

/* 移动端显示 */
.visible-xs {
  display: block;
}

@media (min-width: 640px) {
  .visible-xs {
    display: none;
  }
}
```

### 响应式容器

```css
.container {
  width: 100%;
  margin: 0 auto;
  padding: 0 16px;
}

@media (min-width: 768px) {
  .container {
    max-width: 720px;
    padding: 0 24px;
  }
}

@media (min-width: 1024px) {
  .container {
    max-width: 960px;
  }
}

@media (min-width: 1280px) {
  .container {
    max-width: 1140px;
  }
}

@media (min-width: 1536px) {
  .container {
    max-width: 1320px;
  }
}
```

---

## 动画效果

### 缓动函数

```css
/* 缓动函数 */
--ease-in: cubic-bezier(0.4, 0, 1, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
```

### 基础动画

```css
/* 淡入淡出 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity var(--transition-normal);
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 滑动 */
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all var(--transition-normal);
}

.slide-up-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.slide-up-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

/* 缩放 */
.scale-enter-active,
.scale-leave-active {
  transition: all var(--transition-normal);
}

.scale-enter-from {
  opacity: 0;
  transform: scale(0.9);
}

.scale-leave-to {
  opacity: 0;
  transform: scale(1.1);
}
```

### 特殊动画

```css
/* 心跳动画 */
@keyframes heartbeat {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.6; }
}

.heartbeat-indicator {
  animation: heartbeat 2s ease-in-out infinite;
}

/* 温暖发光 */
@keyframes warm-glow {
  0% { box-shadow: 0 0 0 0 rgba(255, 107, 107, 0.4); }
  70% { box-shadow: 0 0 0 10px rgba(255, 107, 107, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 107, 107, 0); }
}

.warm-glow {
  animation: warm-glow 2s infinite;
}
```

---

## 组件样式

### 页面头部

```css
.page-header {
  margin-bottom: 12px;
}

.page-header__title {
  font-size: 16px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 4px;
  letter-spacing: -0.01em;
}

.page-header__description {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.5;
}

/* 增强版本 - 符合原型规范 */
.page-header--enhanced {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 4px;
  padding: 16px 20px;
  margin-bottom: 12px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
}

.page-header--enhanced .page-header__title {
  color: var(--text-primary);
  font-size: 16px;
}

.page-header--enhanced .page-header__description {
  color: var(--text-secondary);
  font-size: 13px;
}
```

### 操作栏

```css
/* 符合原型规范的操作栏样式 */
.action-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 16px;
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 4px;
  padding: 12px 16px;
  margin-bottom: 12px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  transition: all 0.15s ease;
}

.action-bar:hover {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
}

.action-bar--compact {
  padding: 12px 16px;
  background: var(--bg-primary);
}

.action-bar__actions {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.action-bar__filters {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}
```

### 卡片容器

```css
/* 符合原型规范的卡片容器 */
.card-container {
  background: var(--bg-primary);
  border-radius: 4px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  padding: 16px 20px;
  margin-bottom: 12px;
  border: 1px solid var(--border-color);
  transition: all 0.15s ease;
}

.card-container:hover {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
}
```

### 统计卡片

```css
/* 符合原型规范的统计卡片 */
.stat-card {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 4px;
  padding: 12px 16px;
  display: flex;
  align-items: center;
  gap: 12px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  transition: all 0.15s ease;
}

.stat-card:hover {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
}

.stat-card__icon {
  width: 32px;
  height: 32px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.stat-card__icon--primary {
  background: var(--primary-lighter);
  color: var(--primary-blue);
}

.stat-card__icon--success {
  background: #d1fae5;
  color: var(--success);
}

.stat-card__icon--warning {
  background: #fef3c7;
  color: var(--warning);
}

.stat-card__icon--danger {
  background: #fee2e2;
  color: var(--danger);
}

.stat-card__value {
  font-size: 18px;
  font-weight: 600;
  color: var(--text-primary);
}

.stat-card__label {
  font-size: 13px;
  color: var(--text-secondary);
}
```

### 章节标题

```css
.section-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.section-title::before {
  content: '';
  display: inline-block;
  width: 4px;
  height: 16px;
  background: var(--primary-blue);
  border-radius: 2px;
}
```

---

## 工具类

### Flexbox

```css
.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.items-start { align-items: flex-start; }
.items-end { align-items: flex-end; }
.justify-center { justify-content: center; }
.justify-between { justify-content: space-between; }
.justify-start { justify-content: flex-start; }
.justify-end { justify-content: flex-end; }
.gap-1 { gap: 8px; }
.gap-2 { gap: 16px; }
.gap-3 { gap: 24px; }
.gap-4 { gap: 32px; }
```

### 显示

```css
.hidden { display: none; }
.block { display: block; }
.inline { display: inline; }
.inline-block { display: inline-block; }
.flex { display: flex; }
```

### 定位

```css
.relative { position: relative; }
.absolute { position: absolute; }
.fixed { position: fixed; }
.sticky { position: sticky; }
```

### 圆角

```css
.rounded-none { border-radius: 0; }
.rounded-sm { border-radius: 2px; }
.rounded { border-radius: 4px; }
.rounded-lg { border-radius: 8px; }
.rounded-xl { border-radius: 12px; }
.rounded-full { border-radius: 9999px; }
```

### 阴影

```css
.shadow-sm { box-shadow: var(--shadow-sm); }
.shadow-md { box-shadow: var(--shadow-md); }
.shadow-lg { box-shadow: var(--shadow-lg); }
```

---

## 最佳实践

### 使用 scoped 样式

```vue
<template>
  <div class="my-component">
    <!-- 组件内容 -->
  </div>
</template>

<style scoped>
/* scoped样式只在当前组件生效 */
.my-component {
  /* 样式 */
}
</style>
```

### 使用 CSS 模块

```vue
<template>
  <div :class="$style.container">
    <!-- 组件内容 -->
  </div>
</template>

<style module>
.container {
  /* 样式 */
}
</style>
```

### 避免深层嵌套

```css
/* ❌ 不推荐：嵌套过深 */
.page-container .content-wrapper .data-table .table-header .title {
  font-size: 18px;
}

/* ✅ 推荐：扁平结构 */
.table-header__title {
  font-size: 18px;
}
```

### 使用语义化类名

```css
/* ❌ 不推荐：无意义的类名 */
.red { color: red; }
.big-text { font-size: 24px; }

/* ✅ 推荐：语义化类名 */
.status-error { color: var(--brand-error); }
.heading-large { font-size: 24px; }
```

### 使用 CSS 变量

```css
/* ❌ 不推荐：硬编码颜色 */
.button {
  background-color: #4066E5;
}

/* ✅ 推荐：使用 CSS 变量 */
.button {
  background-color: var(--brand-primary);
}
```

---

## 📐 标准页面布局结构

> **重要经验**：多个页面布局问题的根本原因是 CSS 类名与模板结构不匹配。必须严格遵循以下标准结构。

### 标准页面模板结构

所有列表页面必须遵循以下 HTML 结构：

```vue
<template>
  <div class="page-content">
    <!-- 统计卡片区域 -->
    <div class="stats-grid">
      <!-- 3或4个统计卡片 -->
    </div>

    <!-- 数据表格卡片 -->
    <div class="card">
      <div class="card-header">
        <div class="card-actions-row">
          <!-- 左侧：操作按钮 -->
          <div class="card-actions">
            <button class="btn btn-primary">新建</button>
          </div>
          <!-- 右侧：查询筛选 -->
          <div class="card-actions">
            <div class="search-box">...</div>
            <button class="btn">查询</button>
            <button class="btn">重置</button>
          </div>
        </div>
      </div>
      <div class="card-body">
        <div class="table-container">
          <table class="table">...</table>
        </div>
        <PaginationBar ... />
      </div>
    </div>
  </div>
</template>
```

### 必须配套的 CSS 样式

```css
/* 页面内容 */
.page-content {
  padding: 16px;
  background: #f8fafc;
  min-height: calc(100vh - 60px);
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 统计卡片网格 */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 或 repeat(4, 1fr) */
  gap: 16px;
  flex-shrink: 0;
}

/* 卡片头部 */
.card-header {
  padding: 16px 20px;
  border-bottom: 1px solid #e2e8f0;
  flex-shrink: 0;
}

.card-actions-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.card-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}

/* 卡片主体 */
.card-body {
  padding: 0;
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 响应式 */
@media (max-width: 768px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }

  .card-actions-row {
    flex-direction: column;
    align-items: stretch;
    gap: 12px;
  }

  .card-actions {
    flex-wrap: wrap;
    justify-content: flex-end;
  }

  .search-box {
    width: 100%;
  }
}
```

### ⚠️ 常见错误与修复

#### 错误 1：CSS类名与模板不匹配

**问题**：模板使用 `card-header` + `card-actions-row` 结构，但 CSS 中仍保留旧的 `.action-bar`、`.bar-left`、`.bar-right` 样式。

**表现**：操作区域元素"挤在一起"，布局错乱。

**修复**：删除旧的 CSS 样式，添加与模板匹配的新样式。

```css
/* ❌ 错误：保留旧样式 */
.action-bar { ... }
.bar-left { ... }
.bar-right { ... }

/* ✅ 正确：使用新结构 */
.card-header { ... }
.card-actions-row { ... }
.card-actions { ... }
```

#### 错误 2：遗漏响应式 CSS 更新

**问题**：响应式媒体查询中仍使用旧的类名。

**表现**：小屏幕下布局仍然错乱。

**修复**：同步更新响应式 CSS。

```css
/* ❌ 错误 */
@media (max-width: 768px) {
  .action-bar { flex-direction: column; }
  .bar-right { justify-content: space-between; }
}

/* ✅ 正确 */
@media (max-width: 768px) {
  .card-actions-row { flex-direction: column; gap: 12px; }
  .card-actions { flex-wrap: wrap; justify-content: flex-end; }
}
```

#### 错误 3：修改了错误的文件

**问题**：路由指向 `@/views/user/UserList.vue`，但修改了 `@/views/system/Users.vue`。

**表现**：修改后页面无变化。

**修复**：先查看路由配置确认实际组件路径。

```bash
# 检查路由配置
grep -r "component:" src/router/
```

### 操作区布局原则

**规则**：**操作在左，查询在右**

```
┌─────────────────────────────────────────────────────────────┐
│  [新建按钮] [批量删除]          [搜索框] [查询] [重置] [刷新] │
│  ← 左侧操作区                  右侧查询区 →                  │
└─────────────────────────────────────────────────────────────┘
```

| 位置 | 内容 |
|------|------|
| **左侧** | 新建、批量操作、导出等**主动操作**按钮 |
| **右侧** | 搜索框、筛选下拉、查询、重置、刷新等**查询相关** |

### 必须包含的组件

1. **统计卡片**：3-4 个，使用 `.stats-grid` 横向排列
2. **分页器**：必须使用 `PaginationBar` 组件，不能省略
3. **空状态**：数据为空时显示 `EmptyState` 组件

---

## 🔗 相关文档

- [组件设计规范](./component-design.md) - 组件设计详细规范
- [Element Plus使用规范](./element-plus.md) - UI组件使用规范
- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范

---

**最后更新**：2026-03-21