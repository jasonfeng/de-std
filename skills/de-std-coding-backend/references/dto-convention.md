# DTO 转换最佳实践

**更新日期**: 2026-03-14
**原则**: 所有跨层转换必须通过 Assembly 层

---

## 📋 核心规则

### ✅ 正确做法：使用 Assembly 层

```java
// AppService 层
@Service
@RequiredArgsConstructor
public class DataSourceAppService {

    private final DataSourceDomainService domainService;
    private final DataSourceAssembly assembly; // ✅ 注入 Assembly

    public DataSourceVO getById(Long id) {
        // Entity → VO 转换在 Assembly 层
        return assembly.toVO(domainService.getById(id));
    }

    public Long create(DataSourceParam param) {
        // Param → Entity 转换在 Assembly 层
        DataSourceEntity entity = assembly.toEntity(param);
        return domainService.create(entity);
    }
}
```

```java
// Assembly 层
@Component
public class DataSourceAssembly {

    public DataSourceVO toVO(DataSourceEntity entity) {
        // 使用 MapStruct 或手动转换
        DataSourceVO vo = new DataSourceVO();
        BeanUtils.copyProperties(entity, vo);
        return vo;
    }

    public DataSourceEntity toEntity(DataSourceParam param) {
        DataSourceEntity entity = new DataSourceEntity();
        BeanUtils.copyProperties(param, entity);
        return entity;
    }
}
```

### ❌ 错误做法：在 Service 层直接转换

```java
// ❌ 错误：在 AppService 层直接转换
@Service
public class DataSourceAppService {

    public DataSourceVO getById(Long id) {
        DataSourceEntity entity = domainService.getById(id);

        // ❌ 直接在 Service 层转换，违反分层架构
        DataSourceVO vo = new DataSourceVO();
        vo.setId(entity.getId());
        vo.setName(entity.getName());
        return vo;
    }
}
```

---

## 🎯 Assembly 层职责

### 转换场景覆盖

| 转换类型 | Assembly 方法 | 使用场景 |
|----------|---------------|----------|
| Param → Entity | `toEntity(Param)` | 创建、更新 |
| Entity → VO | `toVO(Entity)` | 查询返回 |
| Entity → DTO | `toDTO(Entity)` | 跨层传递 |
| DTO → Entity | `toEntity(DTO)` | 跨层传递 |
| List<Entity> → List<VO> | `toVOList(List)` | 列表查询 |

### 特殊处理

```java
@Component
public class DataSourceAssembly {

    /**
     * Entity 转 VO（特殊处理）
     */
    public DataSourceVO toVO(DataSourceEntity entity) {
        DataSourceVO vo = new DataSourceVO();
        BeanUtils.copyProperties(entity, vo);

        // 特殊处理：敏感信息脱敏
        if (entity.getPasswordEncrypted() != null) {
            vo.setPasswordEncrypted("***");
        }

        // 特殊处理：枚举转换
        vo.setDbTypeDisplay(DatabaseType.fromCode(entity.getDbType()).getDescription());

        return vo;
    }

    /**
     * 批量 Entity 转 VO
     */
    public List<DataSourceVO> toVOList(List<DataSourceEntity> entities) {
        return entities.stream()
                .map(this::toVO)
                .collect(Collectors.toList());
    }
}
```

---

## ✅ 自查清单

### Assembly 层检查

- [ ] 每个 DomainService 对应的 AppService 有对应的 Assembly
- [ ] Assembly 类放在 `application/assembly/{module}/` 目录
- [ ] Assembly 类使用 `@Component` 注解
- [ ] 所有 Entity ↔ Param/VO 转换通过 Assembly

### 禁止行为

- [ ] 不在 AppService 层直接使用 `BeanUtils.copyProperties` 或 `new VO()`
- [ ] 不在 DomainService 层直接返回 VO 或 Param
- [ ] 不在 Controller 层直接转换 Entity

---

## 🔗 相关文档

- [DDD 分层架构](../01-backend/ddd-architecture.md)
- [代码审查快速查阅](../00-code-review-quick-ref.md)

---

**最后更新**: 2026-03-14
**维护者**: Charlie (Senior Dev)
