# TODO 跟踪机制规范

**版本**: v1.0
**更新日期**: 2026-03-14
**状态**: 活跃

---

## 📋 目录

1. [规范概述](#规范概述)
2. [TODO 注释格式](#todo-注释格式)
3. [跟踪机制](#跟踪机制)
4. [现有清单](#现有清单)
5. [管理流程](#管理流程)

---

## 规范概述

### 目的

建立代码中 TODO/FIXME 注释的标准化管理机制，确保临时方案不被遗忘，技术债务可控。

### 核心原则

| 原则 | 说明 |
|------|------|
| **必须有责任人** | 每个 TODO 必须指定负责人 |
| **必须有截止日期** | 每个 TODO 必须有明确的完成时间 |
| **必须有优先级** | 按 P0/P1/P2 分级 |
| **必须有跟踪编号** | 便于在 issue 系统中追踪 |

---

## TODO 注释格式

### 标准格式

```java
// TODO:[编号] [优先级] [描述] (@责任人 [日期])
// 示例：
// TODO:001 P1 实现租户隔离 (@Charlie 2026-03-20)
```

### 格式组成

| 部分 | 说明 | 示例 |
|------|------|------|
| **编号** | 从 TODO-001 开始递增 | TODO:001 |
| **优先级** | P0(紧急) / P1(高) / P2(中) / P3(低) | P1 |
| **描述** | 简洁描述待办事项 | 实现租户隔离 |
| **责任人** | GitHub 用户名或姓名 | @Charlie |
| **日期** | 预计完成日期 | 2026-03-20 |

### 优先级定义

| 优先级 | 响应时间 | 说明 |
|--------|----------|------|
| **P0** | 24小时内 | 阻塞发布或严重缺陷 |
| **P1** | 本周内 | 重要功能缺失或优化 |
| **P2** | 本迭代内 | 改进项或技术债务 |
| **P3** | 持续关注 | 优化项或备选方案 |

### 类型标签

| 标签 | 说明 | 示例 |
|------|------|------|
| `TODO` | 待办事项 | 需要实现的功能 |
| `FIXME` | 缺陷修复 | 已知问题需要修复 |
| `HACK` | 临时方案 | 需要重构的代码 |
| `XXX` | 风险提示 | 需要关注的代码 |
| `NOTE` | 说明注释 | 重要逻辑说明 |

---

## 跟踪机制

### 1. 代码中的 TODO

```java
@Service
public class DataSourceService {

    // TODO:001 P1 实现租户隔离 (@Charlie 2026-03-20)
    // TODO:002 P2 添加缓存层 (@Elena 2026-03-25)
    public List<DataSource> listDataSources() {
        // FIXME:003 P1 SQL注入风险 (@Charlie 2026-03-15)
        String sql = "SELECT * FROM t_datasource WHERE name = '" + name + "'";
        // ...
    }

    // HACK:004 P2 临时方案，需要重构 (@Bob 2026-03-30)
    private Object legacyMethod() {
        // 旧代码逻辑
    }
}
```

### 2. TODO 清单文档

维护 `/docs/development/todo-list.md` 文件：

```markdown
# TODO 清单

**更新日期**: 2026-03-14
**维护者**: 开发团队

---

## 统计

| 类型 | 数量 |
|------|------|
| TODO | 15 |
| FIXME | 3 |
| HACK | 5 |

---

## P0 - 紧急

| 编号 | 描述 | 文件 | 责任人 | 截止日期 |
|------|------|------|--------|----------|
| TODO:003 | SQL注入风险 | DataSourceService.java | @Charlie | 2026-03-15 |

---

## P1 - 高优先级

| 编号 | 描述 | 文件 | 责任人 | 截止日期 |
|------|------|------|--------|----------|
| TODO:001 | 实现租户隔离 | DataSourceService.java | @Charlie | 2026-03-20 |
```

### 3. Issue 追踪

每个 P0/P1 级别的 TODO 应创建对应的 GitHub Issue：

```
Title: [TODO:001] 实现租户隔离
Labels: priority-high, todo
Assignee: @Charlie
Milestone: Sprint 5
Due Date: 2026-03-20

Description:
- Location: DataSourceService.java:45
- Priority: P1
- Owner: Charlie
```

---

## 现有清单

**最后扫描**: 2026-03-14

### 按类型分类

| 类型 | 数量 | 占比 |
|------|------|------|
| TODO | 8 | 80% |
| FIXME | 2 | 20% |
| **总计** | **10** | 100% |

### 按优先级分类

| 优先级 | 数量 |
|--------|------|
| P0 | 0 |
| P1 | 2 |
| P2 | 5 |
| P3 | 3 |

### 详细清单

#### P1 - 高优先级

| 编号 | 描述 | 文件 | 行号 |
|------|------|------|------|
| TODO:001 | 从 SecurityContext 获取租户ID | FileDatasourceController.java | 47 |
| TODO:002 | 从 SecurityContext 获取租户ID | FileDatasourceController.java | 118 |
| TODO:003 | 从 SecurityContext 获取租户ID | FileDatasourceController.java | 137 |
| TODO:004 | 从 SecurityContext 获取租户ID | DataSourceController.java | 118 |

#### P2 - 中优先级

| 编号 | 描述 | 文件 | 行号 |
|------|------|------|------|
| TODO:005 | 从 SecurityContext 获取部门ID | DataSourceController.java | 44 |
| TODO:006 | 从模型获取表名 | MaterializationController.java | 74 |
| TODO:007 | 根据代码查找数据元ID | ExcelImportDomainService.java | 116 |
| TODO:008 | 根据代码查找字典类型ID | ExcelImportDomainService.java | 123 |
| TODO:009 | 根据不同数据源类型实现数据读取逻辑 | DefaultQualityEngine.java | 172 |

---

## 管理流程

### 1. 新增 TODO

```bash
# 添加 TODO 前先检查编号
grep -r "TODO:" src/ | wc -l

# 按规范添加 TODO 注释
// TODO:00X P1 描述 (@责任人 日期)
```

### 2. 定期审查

**每周五下午** 进行 TODO 审查：

1. 检查过期项并更新优先级
2. 将完成的 TODO 从代码中移除
3. 将长期未完成的 TODO 转为 Issue
4. 更新 todo-list.md 文档

### 3. 完成移除

```bash
# 完成后移除 TODO 注释
git commit -m "fix: 完成 TODO:001 - 实现租户隔离"
```

### 4. 自动化检查

使用 pre-commit hook 检查 TODO 格式：

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 检查 TODO 格式
invalid=$(grep -rE "TODO|FIXME|HACK" src/ | grep -vE "TODO:[0-9]{3}|FIXME:[0-9]{3}|HACK:[0-9]{3}")

if [ -n "$invalid" ]; then
    echo "发现格式不规范的 TODO:"
    echo "$invalid"
    echo "请使用格式: TODO:001 P1 描述 (@责任人 日期)"
    exit 1
fi
```

---

## 工具支持

### 查找 TODO

```bash
# 查找所有 TODO
grep -rn "TODO\|FIXME\|HACK\|XXX" --include="*.java" backend/

# 统计 TODO 数量
grep -r "TODO\|FIXME\|HACK\|XXX" --include="*.java" backend/ | wc -l

# 按文件分组
grep -rn "TODO\|FIXME" --include="*.java" backend/ | cut -d: -f1 | sort | uniq -c
```

### VSCode 插件

推荐使用以下插件：
- **Todo Tree**: 可视化显示 TODO 注释
- **Todo Highlight**: 高亮显示 TODO 注释

---

## 附录

### TODO 模板

```java
// TODO:[编号] [优先级] [简短描述]
// 详细说明：
// - 背景：为什么需要这个 TODO
// - 方案：计划如何实现
// - 依赖：是否有前置条件
// 责任人：@xxx
// 截止日期：YYYY-MM-DD
```

### 示例

```java
// TODO:010 P1 实现数据源连接池管理
// 详细说明：
// - 背景：当前每次查询都创建新连接，性能较差
// - 方案：使用 HikariCP 连接池
// - 依赖：无
// 责任人：@Charlie
// 截止日期：2026-03-20
public Connection getConnection() {
    // 当前实现
    return DriverManager.getConnection(url, user, password);
}
```

---

**维护者**: 开发团队
**更新周期**: 每周审查
