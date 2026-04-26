# 项目术语表

> 本文档定义数擎项目中使用的专业术语，确保团队沟通一致性。

---

## 📋 术语表

| 术语 | 英文 | 定义 | 示例 |
|------|------|------|------|
| **实体** | Entity | 领域对象，对应数据库表，必须带Entity后缀 | `UserEntity` |
| **DTO** | Data Transfer Object | 应用层数据传输对象，跨层传输 | `UserDTO` |
| **VO** | View Object | 视图对象，返回给前端的数据 | `UserVO` |
| **Param** | Parameter | 请求参数对象，接收前端数据 | `UserParam` |
| **AppService** | Application Service | 应用服务，事务编排和跨层协调 | `UserAppService` |
| **DomainService** | Domain Service | 领域服务，核心业务逻辑 | `UserDomainService` |
| **Repository** | Repository | 仓储接口，数据访问抽象 | `UserRepository` |
| **Mapper** | Mapper | MyBatis映射器，SQL执行接口 | `UserMapper` |
| **Controller** | Controller | 控制器，REST API入口 | `UserController` |
| **Assembly** | Assembly | 装配器，对象转换工具 | `UserAssembly` |
| **雪花算法** | Snowflake | 分布式ID生成算法 | `@TableId(type = IdType.ASSIGN_ID)` |
| **Flyway** | Flyway | 数据库版本管理工具 | `V24__create_table.sql` |
| **DDD** | Domain-Driven Design | 领域驱动设计 | 5层架构 |
| **LTS** | Long Term Support | 长期支持版本 | Node.js 18 LTS |
| **ADR** | Architecture Decision Record | 架构决策记录 | 技术选型决策文档 |

---

## 📦 数据对象层次

```
Param (请求) → DTO (传输) → Entity (领域) → VO (响应)
    ↓           ↓            ↓           ↓
 前端传入    应用层使用   数据库映射    前端展示
```

| 层次 | 作用 | 生命周期 | 必须带后缀 |
|------|------|---------|-----------|
| **Param** | 接收前端请求 | 请求短生命周期 | 是（Param后缀） |
| **DTO** | 跨层传输 | 应用层内部 | 是（DTO后缀） |
| **Entity** | 领域业务对象 | 数据库映射 | 是（Entity后缀） |
| **VO** | 返回前端响应 | 响应短生命周期 | 是（VO后缀） |

---

## 🏗️ 服务层职责

| 服务层 | 职责 | 依赖 | 事务边界 |
|--------|------|------|---------|
| **Controller** | REST API入口、参数验证 | AppService | 无 |
| **AppService** | 事务编排、跨层协调 | DomainService | ✅ 是 |
| **DomainService** | 核心业务逻辑、操作Entity | Repository | ❌ 否 |
| **Repository** | 数据访问抽象 | Mapper | ❌ 否 |

**调用关系**：
```
Controller → AppService → DomainService → Repository → Database
```

---

## 🗄️ 数据库术语

| 术语 | 说明 | 示例 |
|------|------|------|
| **Flyway迁移** | 数据库版本控制脚本 | `V24__create_table.sql` |
| **雪花ID** | 分布式唯一ID | 1734567890123456789 |
| **t_前缀** | 表名统一前缀 | `t_user` |
| **BIGSERIAL** | PostgreSQL自增类型 | `id BIGSERIAL PRIMARY KEY` |
| **软删除** | 逻辑删除标记 | `deleted` 字段 |

---

## 🎨 前端术语

| 术语 | 英文 | 定义 | 规范 |
|------|------|------|------|
| **Composition API** | Composition API | Vue 3组合式API | `<script setup lang="ts">` |
| **Pinia** | Pinia | Vue状态管理库 | Setup模式 |
| **Setup模式** | Setup Pattern | Pinia Store定义方式 | `defineStore('id', () => {})` |
| **严格模式** | Strict Mode | TypeScript严格类型检查 | `"strict": true` |
| **单文件组件** | SFC | Single File Component | `.vue` 文件 |

---

## 🧪 测试术语

| 术语 | 说明 | 要求 |
|------|------|------|
| **指令覆盖率** | Instruction Coverage | 字节码执行比例 | DomainService ≥80% |
| **分支覆盖率** | Branch Coverage | if/分支执行比例 | DomainService ≥65% |
| **E2E测试** | End-to-End Testing | 端到端测试 | Playwright |
| **单元测试** | Unit Testing | 单个方法测试 | JUnit 5 |

---

## 🔄 工作流术语

| 术语 | 说明 | 用途 |
|------|------|------|
| **Story** | 用户故事 | 功能需求单位 |
| **Epic** | 史诗 | 大型功能分组 |
| **PR** | Pull Request | 代码合并请求 |
| **Code Review** | 代码审查 | 质量保障流程 |
| **CI/CD** | Continuous Integration/Deployment | 持续集成/部署 |

---

## 📖 常见缩写

| 缩写 | 全称 | 中文 |
|------|------|------|
| **API** | Application Programming Interface | 应用程序接口 |
| **DTO** | Data Transfer Object | 数据传输对象 |
| **VO** | View Object | 视图对象 |
| **CRUD** | Create/Read/Update/Delete | 增删改查 |
| **REST** | Representational State Transfer | 表述性状态转移 |
| **SQL** | Structured Query Language | 结构化查询语言 |
| **DDL** | Data Definition Language | 数据定义语言 |
| **DML** | Data Manipulation Language | 数据操作语言 |
| **LTS** | Long Term Support | 长期支持 |
| **ADR** | Architecture Decision Record | 架构决策记录 |

---

## 🔗 相关文档

- [核心规则](./essential-rules.md)
- [新人入门指南](./newcomer-guide.md)
- [API契约规范](./api-contract.md)
- [DDD架构](../01-backend/ddd-architecture.md)

---

**文档元数据**：
- **维护者**: 技术架构团队
- **目标读者**: 所有团队成员
- **最后更新**: 2026-03-14
- **下次审查**: 每季度
