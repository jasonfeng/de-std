---
name: coding-workflow
description: 开发工作流 Story/PR/Code Review 规范。提交代码或进入 review 状态前必须调用。
---

# 开发工作流规范

## 触发时机

- "提交代码/push"
- "准备 review"
- "完成 Story"
- "创建 Pull Request"
- "进行代码审查"

---

## Story 开发流程

### Step 1: 阅读设计文档

**强制规则**：实现 task 前，必须先读取 Story 文件 `References` 引用的设计文档对应章节。

- 以设计文档为实现的唯一标准
- Task 简述与设计文档冲突时以设计文档为准

### Step 2: TDD 开发（Red → Green → Refactor）

```
RED → GREEN → REFACTOR

1. RED：写一个失败的测试，运行确认失败
2. GREEN：写最少代码让测试通过
3. REFACTOR：优化代码，确保测试仍通过
```

**判定标准**：测试文件最后修改时间不得晚于实现文件。

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

## Epic 完成检查清单

- 所有 Story 状态为 done
- 所有测试通过
- 所有模块覆盖率达标
- E2E 测试通过
- 文档更新完成
- 发布说明准备就绪

---

## 前后端集成规范

### 集成前检查

- API 接口定义一致
- 字段命名约定一致
- 错误码定义一致
- 分页参数一致

### 集成测试

- 前端调用后端真实 API
- 验证数据格式
- 验证错误处理
- 验证边界情况

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| 无测试进入 review | 违反 TDD 原则 |
| 跳过测试运行 | 可能隐藏 bug |
| 遗留 TODO 注释 | 所有问题必须解决 |
| 半成品提交 | 影响团队协作 |
| 手动修改数据库 | Flyway 版本失控 |

---

## 详细规范

- **[story-completion.md](references/story-completion.md)** — Story 完成标准、Done 定义、验收条件
- **[story-completion-checklist.md](references/story-completion-checklist.md)** — Story 完成检查清单、状态流转
- **[epic-completion-checklist.md](references/epic-completion-checklist.md)** — Epic 完成检查清单、发布准备
- **[code-review.md](references/code-review.md)** — 代码审查规范、Checklist、审查流程
- **[git-workflow.md](references/git-workflow.md)** — Git 分支策略、提交规范、PR 流程
- **[deployment.md](references/deployment.md)** — 部署流程、环境配置、发布检查
- **[frontend-backend-integration.md](references/frontend-backend-integration.md)** — 前后端集成规范、接口契约、集成测试
- **[governance.md](references/governance.md)** — 项目治理、决策流程、变更管理
- **[todo-tracking.md](references/todo-tracking.md)** — TODO 追踪、问题管理、优先级排序
