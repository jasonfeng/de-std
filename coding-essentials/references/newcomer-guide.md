# 新人入门指南

欢迎加入数擎项目！本文档将帮助您快速上手项目开发。

---

## 🎯 第1天：环境搭建

### 1. 安装必需软件

**后端开发**：
- [JDK 17+](https://adoptium.net/)（必需，不支持Java 11/8）
- [Maven 3.9+](https://maven.apache.org/)
- [PostgreSQL 14+](https://www.postgresql.org/download/)（或 Docker）
- [IntelliJ IDEA](https://www.jetbrains.com/idea/)（推荐）

**前端开发**：
- [Node.js 18 LTS](https://nodejs.org/)（必需）
- [npm 10+](https://docs.npmjs.com/)（或 [pnpm](https://pnpm.io/)）
- [VS Code](https://code.visualstudio.com/)（推荐）

**Docker环境**（可选）：
- Docker Desktop
- Docker Compose

### 2. 克隆项目并安装依赖

```bash
# 克隆项目
git clone <repository-url>
cd data-engine

# 后端依赖
cd backend/dataengine-api
mvn clean install

# 前端依赖
cd frontend/dataengine-frontend
npm install
```

### 3. 启动开发环境

**方式1：使用Docker（推荐）**
```bash
# 启动所有服务
docker-compose up -d

# 查看日志
docker-compose logs -f app
```

**方式2：手动启动**
```bash
# 启动PostgreSQL
docker-compose up -d dataengine-postgres

# 启动后端
cd backend/dataengine-api
mvn spring-boot:run

# 启动前端
cd frontend/dataengine-frontend
npm run dev
```

### 4. 验证安装

```bash
# 后端：访问 http://localhost:8080/actuator/health
# 前端：访问 http://localhost:5173
```

---

## 📚 第1周：必读核心规范

### 1. 阅读核心规则

**必读文档**：[00-essential/essential-rules.md](./essential-rules.md)

这10条规则是所有开发者必须遵守的核心规则。花15分钟熟悉它们，会在代码审查时严格检查。

### 2. 了解架构

**后端架构**：[01-backend/ddd-architecture.md](../01-backend/ddd-architecture.md)

- 5层DDD架构（interfaces/application/domain/infrastructure/common）
- 调用关系：Controller → AppService → DomainService → Repository

**前端架构**：[02-frontend/component-design.md](../02-frontend/component-design.md)

- Vue 3 Composition API
- 组件设计规范
- 状态管理（Pinia）

### 3. 学习编码规范

**后端**：[01-backend/java-coding.md](../01-backend/java-coding.md)
- 命名规范
- Lombok使用
- JavaDoc规范

**前端**：[02-frontend/vue-coding.md](../02-frontend/vue-coding.md)
- Composition API使用
- 组件命名
- TypeScript严格模式

### 4. 了解数据库规范

**PostgreSQL语法**：[03-database/postgresql.md](../03-database/postgresql.md)
- ⚠️ 禁止MySQL特有语法
- 使用BIGSERIAL代替AUTO_INCREMENT
- COMMENT ON COLUMN代替行内COMMENT

**表名规范**：[03-database/naming-conventions.md](../03-database/naming-conventions.md)
- 表名使用t_前缀
- 实体类使用Entity后缀
- 主键使用雪花算法

---

## 🚀 第2-4周：深入阅读

### 按角色深入阅读

**后端开发者**：
1. [01-backend/spring-boot.md](../01-backend/spring-boot.md)
2. [01-backend/mybatis-plus.md](../01-backend/mybatis-plus.md)
3. [01-backend/dto-convention.md](../01-backend/dto-convention.md)

**前端开发者**：
1. [02-frontend/typescript.md](../02-frontend/typescript.md)
2. [02-frontend/state-management.md](../02-frontend/state-management.md)
3. [02-frontend/api-calling.md](../02-frontend/api-calling.md)

**数据库开发者**：
1. [03-database/flyway.md](../03-database/flyway.md)
2. [03-database/mybatis-mapping.md](../03-database/mybatis-mapping.md)

**测试工程师**：
1. [04-testing/unit-testing.md](../04-testing/unit-testing.md)
2. [04-testing/integration-testing.md](../04-testing/integration-testing.md)
3. [04-testing/coverage-standards.md](../04-testing/coverage-standards.md)

---

## 🎓 学习路径建议

### 新人第一个任务建议

**第1周**：熟悉环境和核心规则
- [ ] 完成环境搭建
- [ ] 阅读核心规则
- [ ] 运行项目，熟悉功能

**第2周**：简单Bug修复或文档更新
- [ ] 选择一个简单的Issue
- [ ] 按照规范修复代码
- [ ] 提交PR，通过代码审查

**第3-4周**：参与Feature开发
- [ ] 选择一个标记为"good-first-issue"的Issue
- [ ] 按照Story规范实现
- [ ] 编写测试，确保覆盖率达标
- [ ] 参与Code Review

---

## 🆘 获取帮助

### 遇到问题时的查找顺序

1. **查看核心规则**：[essential-rules.md](./essential-rules.md)
2. **搜索相关规范**：使用 `grep -r "关键词" CODING_STANDARDS/`
3. **查看示例代码**：[06-templates/](../06-templates/)
4. **询问团队成员**：Slack/钉钉群

### 常见问题

**Q1: 我应该从哪个规范开始？**
A: 从 [essential-rules.md](./essential-rules.md) 开始，然后按角色阅读相关规范。

**Q2: 我忘记某个规范怎么写了？**
A: 使用 `grep -r "关键词" CODING_STANDARDS/` 搜索，或查看 [00-quick-reference.md](../00-quick-reference.md)。

**Q3: 我不确定这样写是否符合规范？**
A: 先按规范写，提交PR时会有Code Review检查。

**Q4: 规范太多了记不住怎么办？**
A: 先记住核心10条规则，其他规范按需查阅。IDE插件和代码审查也会帮助检查。

---

## 📋 新人检查清单

在提交第一个PR之前，确认：

- [ ] 已阅读 [essential-rules.md](./essential-rules.md) 核心规则
- [ ] 已了解项目的DDD架构
- [ ] 已配置开发环境并能正常运行
- [ ] 已安装代码格式化插件（后端IDEA、前端VS Code）
- [ ] 已运行测试并全部通过
- [ ] 已阅读本次任务的Story规范（如有）

---

## 🔗 相关文档

- [核心规则](./essential-rules.md)
- [项目术语表](./glossary.md)
- [API契约规范](./api-contract.md)
- [完整规范索引](../README.md)

---

**文档元数据**：
- **维护者**: 技术架构团队
- **目标读者**: 新加入团队的成员
- **最后更新**: 2026-03-14
- **下次审查**: 每季度
