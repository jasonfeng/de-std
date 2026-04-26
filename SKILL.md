---
name: de-std
description: DataEngine 编码规范 Skill 包。写代码前必须调用的强制规范集合。
version: 2.0.0
---

# DataEngine 编码规范 Skill 包

**版本**：2.0.0 | **更新**：2026-04-25

本 Skill 包包含 DataEngine 项目的完整编码规范，所有原始规范内容完整保留在 references/ 目录中。

## 触发示例

用户说以下内容时应主动调用对应 Skill：

| 用户说 | 应调用 |
|-------|--------|
| "帮我添加一个用户管理功能" | `/de-std-coding-essentials` + `/de-std-coding-backend` |
| "写一个前端列表页面" | `/de-std-coding-frontend` |
| "创建一个数据库表" | `/de-std-coding-database` |
| "写单元测试" | `/de-std-coding-testing` |
| "提交代码/准备 review" | `/de-std-coding-workflow` |

## Skill 列表

| Skill | 调用命令 | 内容概要 |
|-------|---------|---------|
| `coding-essentials` | `/de-std-coding-essentials` | 10条核心规则、API契约、术语表、新人指南（4个完整文档） |
| `coding-backend` | `/de-std-coding-backend` | DDD架构、Spring Boot、MyBatis Plus、Java编码（11个完整文档） |
| `coding-frontend` | `/de-std-coding-frontend` | Vue 3、TypeScript、Element Plus、组件设计、样式规范（14个完整文档） |
| `coding-database` | `/de-std-coding-database` | PostgreSQL、Flyway、MyBatis映射、命名规范（4个完整文档） |
| `coding-testing` | `/de-std-coding-testing` | TDD流程、单元测试、集成测试、E2E测试、覆盖率标准（11个完整文档） |
| `coding-workflow` | `/de-std-coding-workflow` | Story流程、代码审查、PR规范、Git提交、部署流程（9个完整文档） |

## 使用流程

```
1. 开始写代码 → 先调用 /de-std-coding-essentials
2. 根据代码类型 → 调用具体规范 skill
3. 完成代码 → 调用 /de-std-coding-workflow 检查
```

## 内容完整性

每个 Skill 的结构：

```
skill-name/
├── SKILL.md          # 核心要点（<500行）+ 禁止行为汇总 + references 索引
└── references/       # 完整规范原文（不精简、不丢失）
    ├── topic-a.md
    ├── topic-b.md
    └── ...
```

**总内容量**：
- 原始 CODING_STANDARDS：~26,500 行
- Skill 包保留：100% 完整

## ⚠️ 强制执行

不调用规范 Skill 的代码提交将被拒绝。

## 规范来源

基于项目 `CODING_STANDARDS/` 目录整理，包含以下原始规范文件：

| 目录 | 文件数 | 原始行数 |
|------|--------|---------|
| 00-essential | 4 | 1,025 |
| 01-backend | 11 | 6,008 |
| 02-frontend | 14 | 9,348 |
| 03-database | 4 | 1,749 |
| 04-testing | 11 | 5,546 |
| 05-workflow | 9 | 2,828 |
| **总计** | **53** | **26,504** |
