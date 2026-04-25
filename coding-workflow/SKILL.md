---
name: coding-workflow
description: 开发工作流 Story/PR/Code Review 规范。提交代码或进入 review 状态前必须调用。
---

# 开发工作流规范

## 触发时机

用户说以下内容时**必须**先调用此 Skill：

- "提交代码/push"
- "准备 review"
- "完成 Story"
- "创建 Pull Request"
- "进行代码审查"

## 调用时机

- 开始 Story 开发之前
- 提交 Pull Request 之前
- 进行代码审查之前
- 标记 Story 为 done 之前

---

## Story 开发流程

### Step 1: 阅读设计文档

**强制规则**：实现 task 前，必须先读取 Story 文件 `References` 引用的设计文档对应章节。

- 以设计文档为实现的唯一标准
- Task 简述与设计文档冲突时以设计文档为准

---

### Step 2: TDD 开发（Red → Green → Refactor）

```
RED → GREEN → REFACTOR

1. RED：写一个失败的测试，运行确认失败
2. GREEN：写最少代码让测试通过
3. REFACTOR：优化代码，确保测试仍通过
```

**判定标准**：测试文件最后修改时间不得晚于实现文件。

**适用范围**：DomainService、AppService、Controller、前端组件

**不适用**：纯配置文件（pom.xml）、SQL 迁移脚本、无逻辑的 DTO/VO/Param

---

### Step 3: 运行测试

```bash
# 后端
mvn test

# 前端
npm run test:coverage
```

**强制规则**：所有测试必须通过才能进入 review 阶段

---

## 代码审查规范

### 审查时机

- 写完代码立即调用 `code-reviewer` agent
- 同类 task 批量完成后统一审查
- Story 完成时调用 `superpowers:code-reviewer`

### 审查 Checklist

| 检查项 | 标准 |
|-------|------|
| 可读性 | 命名清晰、逻辑直观 |
| 函数长度 | <50 行 |
| 文件长度 | <800 行 |
| 嵌套深度 | ≤4 层 |
| 错误处理 | 显式处理所有错误 |
| 测试覆盖 | ≥80% |
| 安全检查 | 无硬编码密钥、无注入漏洞 |

---

## Pull Request 规范

### PR 标题格式

```
<type>(<scope>): <description>

示例：
feat(task): 添加任务管理模块
fix(frontend): 修复列表页面空状态显示
refactor: 清理未使用组件
```

### PR Body 模板

```markdown
## Summary
- <变更点1>
- <变更点2>

## Test plan
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] E2E 测试通过（如有）
- [ ] 手动验证：<具体验证内容>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### PR 检查清单

提交 PR 前必须确认：

- [ ] CI/CD 通过
- [ ] 合并冲突已解决
- [ ] 分支与目标分支同步
- [ ] 测试覆盖率达标
- [ ] 无遗留 TODO 注释

---

## Story 完成标准

### Done 定义

| 条件 | 验证方式 |
|------|---------|
| 功能实现完成 | 对照 AC（验收条件）逐项检查 |
| 单元测试通过 | `mvn test` / `npm test` |
| 覆盖率达标 | DomainService ≥80% |
| 代码审查通过 | 无 CRITICAL/HIGH 问题 |
| 文档更新 | API 文档、README 同步更新 |
| 无遗留问题 | 禁止 "临时方案"、"后续修复" |

### 禁止行为

| 行为 | 原因 |
|------|------|
| 无测试进入 review | 违反 TDD 原则 |
| 跳过测试运行 | 可能隐藏 bug |
| 保留 skip 测试 | 套件处于 blocked 状态 |
| 遗留 TODO 注释 | 所有问题必须解决 |
| 手动修改数据库 | Flyway 版本失控 |

---

## Git 提交规范

### Commit Message 格式

```
<type>: <description>

示例：
feat: 添加用户管理功能
fix: 修复登录验证逻辑
refactor: 重构数据访问层
docs: 更新 API 文档
test: 添加单元测试
chore: 更新依赖版本
```

### 提交时机

- 完成一个完整的功能点
- 修复一个 bug
- 完成一次重构
- 不要提交半成品

---

## Agent 使用指南

| Workflow Step | Agent | 作用 |
|--------------|-------|------|
| Step 5 RED | `tdd-guide` | 指导测试用例设计 |
| Step 5 GREEN | `build-error-resolver` | 修复编译错误 |
| Step 6 | `e2e-runner` | 编写和运行 E2E 测试 |
| Step 8 | `java-reviewer` / `typescript-reviewer` | 批量代码审查 |
| Step 9 | `superpowers:code-reviewer` | Story 最终审查 |
| 安全敏感代码 | `security-reviewer` | 检查安全漏洞 |