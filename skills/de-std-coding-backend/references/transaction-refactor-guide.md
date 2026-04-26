# DomainService 事务管理重构指南

**更新日期**: 2026-03-14
**目的**: 将事务管理从 DomainService 层移到 AppService 层

---

## 📋 问题说明

### 当前状态（不符合规范）

```java
@Service
public class DataSourceDomainService {
    @Transactional(rollbackFor = Exception.class)  // ❌ DomainService 不应有事务
    public DataSourceEntity createDataSource(DataSourceEntity entity) {
        // ...
    }
}
```

### 目标状态（符合 DDD 规范）

```java
@Service
public class DataSourceDomainService {
    // ✅ DomainService 不管理事务
    public DataSourceEntity createDataSource(DataSourceEntity entity) {
        // ...
    }
}

@Service
@RequiredArgsConstructor
public class DataSourceAppService {
    private final DataSourceDomainService domainService;

    // ✅ AppService 管理事务
    @Transactional(rollbackFor = Exception.class)
    public DataSourceVO createDataSource(DataSourceParam param) {
        DataSourceEntity entity = assembly.toEntity(param);
        entity = domainService.createDataSource(entity);
        return assembly.toVO(entity);
    }
}
```

---

## 🔄 重构步骤

### 第 1 步：识别需要重构的 DomainService

**检查方法**:
```bash
grep -r "@Transactional" backend/dataengine-api/src/main/java/com/dp/dataengine/domain/service/
```

**结果**:
- `DataSourceDomainService`: 4 处 @Transactional
- `FileDatasourceDomainService`: 3 处 @Transactional

### 第 2 步：确认 AppService 有事务管理

对于每个 DomainService 的 @Transactional 方法，确认对应的 AppService 方法有 @Transactional：

| DomainService 方法 | 对应 AppService 方法 | AppService 有 @Transactional? |
|-------------------|---------------------|-------------------------|
| createDataSource | createDataSource | ✅ 有 |
| updateDataSource | updateDataSource | ✅ 有 |
| deleteDataSource | deleteDataSource | ✅ 有 |

### 第 3 步：移除 DomainService 的 @Transactional

**修改前**:
```java
@Service
public class DataSourceDomainService {
    @Transactional(rollbackFor = Exception.class)
    public DataSourceEntity createDataSource(DataSourceEntity entity) {
        // ...
    }
}
```

**修改后**:
```java
@Service
public class DataSourceDomainService {
    // 移除 @Transactional 注解
    public DataSourceEntity createDataSource(DataSourceEntity entity) {
        // ...（业务逻辑不变）
    }
}
```

### 第 4 步：验证

**测试场景**:
1. 正常创建数据源 → 应该成功
2. 创建数据源时违反业务规则 → 应该回滚
3. 批量操作 → 应该原子性

---

## ⚠️ 注意事项

### 事务边界设计

**✅ 正确**：事务边界在 AppService 层

```java
@Transactional(rollbackFor = Exception.class)
public void syncProcess(Param param) {
    // 多个 DomainService 调用在一个事务中
    domainService1.create(entity1);
    domainService2.create(entity2);
    domainService3.update(entity3);
}
```

**❌ 错误**：DomainService 管理事务导致事务边界不清晰

```java
// DomainService 各自有事务
domainService1.create(entity1);  // 事务1
domainService2.create(entity2);  // 事务2
domainService3.update(entity3);  // 事务3
// 如果事务3失败，事务1和2无法回滚
```

---

## 📋 重构清单

### DataSourceDomainService (4 处 @Transactional)

- [ ] 第 48 行: `createDataSource` - 移除 @Transactional
- [ ] 第 99 行: `updateDataSource` - 移除 @Transactional
- [ ] 第 158 行: `deleteDataSource` - 移除 @Transactional
- [ ] 第 316 行: `testConnection` - 移除 @Transactional（测试方法保留）

### FileDatasourceDomainService (3 处 @Transactional)

- [ ] 第 44 行: `createFileDatasource` - 移除 @Transactional
- [ ] 第 98 行: `updateFileDatasource` - 移除 @Transactional
- [ ] 第 138 行: `deleteFileDatasource` - 移除 @Transactional

---

## ✅ 验证检查表

重构完成后，验证以下内容：

- [ ] 所有 DomainService 没有 @Transactional 注解
- [ ] 所有 AppService 对应方法有 @Transactional 注解
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 事务回滚测试通过

---

## 🔗 相关文档

- [DDD 分层架构](./01-backend/ddd-architecture.md)
- [代码审查快速查阅](../00-code-review-quick-ref.md)

---

**最后更新**: 2026-03-14
**维护者**: Charlie (Senior Dev)
**重构预计工时**: 2-3 小时
**风险**: 中等（需要仔细测试验证）
