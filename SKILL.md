---
name: de-std
description: DataEngine 编码规范 Skill 包。包含核心规则、后端规范、前端规范、数据库规范、测试规范、工作流规范。
---

# DataEngine 编码规范 Skill 包

本 Skill 包包含 DataEngine 项目的完整编码规范，写代码前必须调用对应规范。

## Skill 列表

| Skill | 调用命令 | 适用场景 |
|-------|---------|---------|
| `coding-essentials` | `/de-std:coding-essentials` | **写任何代码前** - 10条核心规则 |
| `coding-backend` | `/de-std:coding-backend` | Java/Spring Boot/MyBatis 后端代码 |
| `coding-frontend` | `/de-std:coding-frontend` | Vue 3/TypeScript 前端代码 |
| `coding-database` | `/de-std:coding-database` | SQL/Flyway/PostgreSQL 数据库 |
| `coding-testing` | `/de-std:coding-testing` | JUnit/Vitest/E2E 测试代码 |
| `coding-workflow` | `/de-std:coding-workflow` | Git提交/Code Review/Story完成 |

## 使用流程

```
1. 开始写代码 → 先调用 /de-std:coding-essentials
2. 根据代码类型 → 调用具体规范 skill
3. 完成代码 → 调用 /de-std:coding-workflow 检查
```

## ⚠️ 强制执行

不调用规范 Skill 的代码提交将被拒绝。

## 规范来源

基于项目 `CODING_STANDARDS/` 目录整理，包含28000+行详细规范。