# 服务端排序与ID处理检查清单

> 新增列表页面或现有页面改造时，请使用此检查清单确保符合规范。

**适用场景**：
- 新增列表页面
- 现有列表页面改造（添加服务端排序）
- 修复ID精度问题

---

## 后端检查清单

### Entity 层

- [ ] ID 字段使用 `@TableId(type = IdType.ASSIGN_ID)`
- [ ] 统计字段添加 `@TableField(exist = false)`
- [ ] 统计字段类型使用 `Long`

```java
@TableId(value = "id", type = IdType.ASSIGN_ID)
private Long id;

@TableField(exist = false)
private Long itemCount;
```

### DTO 层

- [ ] ID 字段使用 `Long` 类型
- [ ] 统计字段类型使用 `Long`

```java
private Long id;
private Long itemCount;
```

### VO 层

- [ ] ID 字段使用 `String` 类型
- [ ] 统计字段类型使用 `Long`

```java
private String id;  // 避免前端精度丢失
private Long itemCount;
```

### Assembly 层

- [ ] `dtoToVO()` 方法中手动转换 ID

```java
public static DictTypeVO dtoToVO(DictTypeDTO dto) {
    if (dto == null) {
        return null;
    }
    DictTypeVO vo = new DictTypeVO();
    BeanUtils.copyProperties(dto, vo);
    // 手动转换 ID 为 String
    if (dto.getId() != null) {
        vo.setId(dto.getId().toString());
    }
    return vo;
}
```

### DomainService 层

- [ ] 分页查询方法支持 `sortBy` 和 `sortOrder` 参数
- [ ] 实现排序逻辑（switch 或 if-else）
- [ ] 默认按 `updatedAt` 降序
- [ ] 为每个实体统计关联数据（如 itemCount）
- [ ] **复杂排序（多字段组合、业务语义排序、CASE 表达式）必须在 MyBatis XML 中使用 `<choose>/<when>` 实现，禁止在 Java 代码中拼接 SQL 片段**

```java
public IPage<DictTypeEntity> getDictTypesPage(
    Page<DictTypeEntity> page,
    String sortBy,
    String sortOrder
) {
    LambdaQueryWrapper<DictTypeEntity> wrapper = new LambdaQueryWrapper<>();

    // 处理排序
    if (StringUtils.hasText(sortBy)) {
        boolean isAsc = "asc".equalsIgnoreCase(sortOrder);
        switch (sortBy) {
            case "createdAt":
                wrapper.orderBy(true, isAsc, DictTypeEntity::getCreatedAt);
                break;
            case "updatedAt":
            default:
                wrapper.orderBy(true, isAsc, DictTypeEntity::getUpdatedAt);
                break;
        }
    } else {
        wrapper.orderByDesc(DictTypeEntity::getUpdatedAt);
    }

    IPage<DictTypeEntity> resultPage = dictTypeRepository.selectPage(page, wrapper);

    // 统计关联数据
    for (DictTypeEntity entity : resultPage.getRecords()) {
        Integer itemCount = dictItemRepository.countByTypeId(entity.getId());
        entity.setItemCount(itemCount != null ? itemCount.longValue() : 0L);
    }

    return resultPage;
}
```

**复杂排序场景**（多字段组合排序、业务语义排序）：

**职责划分**：DomainService 仅负责排序字段白名单校验和驼峰→下划线命名转换，所有排序逻辑放在 MyBatis XML 的 `<choose>/<when>` 中实现。

```java
// DomainService：只做白名单校验和命名转换
public List<LogicalModelEntity> getAllLogicalModels(String sortBy, String sortOrder) {
    if (sortBy != null && !"createdAt".equals(sortBy) && !"updatedAt".equals(sortBy)
            && !"status".equals(sortBy)) {
        sortBy = "updatedAt";
    }
    // 验证 sortOrder
    if (sortOrder != null && !"asc".equalsIgnoreCase(sortOrder) && !"desc".equalsIgnoreCase(sortOrder)) {
        sortOrder = "desc";
    }
    String dbSortBy = sortBy != null ? camelToSnake(sortBy) : "updated_at";
    String dbSortOrder = sortOrder != null ? sortOrder : "desc";
    return logicalModelRepository.findAllWithFieldCount(dbSortBy, dbSortOrder);
}
```

```xml
<!-- MyBatis XML：使用choose条件处理复杂排序 -->
<if test="sortBy != null and sortOrder != null">
    <choose>
        <when test="sortBy != null and sortBy == 'status'.toString()">
            ORDER BY
                CASE lm.status WHEN 'draft' THEN 1 WHEN 'published' THEN 2 ELSE 0 END ${sortOrder},
                lm.updated_at DESC
        </when>
        <otherwise>
            ORDER BY ${sortBy} ${sortOrder}
        </otherwise>
    </choose>
</if>
```

> OGNL 字符串比较需使用 `sortBy == 'value'.toString()` 确保类型匹配。

### Controller 层

- [ ] 提供 `/page` 分页接口
- [ ] 接受 `pageNum`、`pageSize`、`sortBy`、`sortOrder` 参数
- [ ] 返回 `PageResponseVO<T>` 类型

```java
@GetMapping("/page")
public ResponseEntity<PageResponseVO<DictTypeVO>> getDictTypesPage(
    @RequestParam(defaultValue = "1") Integer pageNum,
    @RequestParam(defaultValue = "10") Integer pageSize,
    @RequestParam(defaultValue = "updatedAt") String sortBy,
    @RequestParam(defaultValue = "desc") String sortOrder
) {
    IPage<DictTypeDTO> dtoPage = dictTypeAppService.getDictTypesPage(
        pageNum, pageSize, sortBy, sortOrder
    );

    PageResponseVO<DictTypeVO> response = new PageResponseVO<>();
    response.setRecords(dtoPage.getRecords().stream()
            .map(StandardAppAssembly::dtoToVO)
            .collect(Collectors.toList()));
    response.setTotal(dtoPage.getTotal());
    response.setPageNum((long) dtoPage.getCurrent());
    response.setPageSize((long) dtoPage.getSize());
    response.setPages((long) dtoPage.getPages());

    return ResponseEntity.ok(response);
}
```

### Jackson 配置

- [ ] **不要**配置全局 Long 转 String 序列化
- [ ] 只配置 JavaTimeModule

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer jacksonCustomizer() {
    return builder -> {
        // 只注册 JavaTimeModule，不序列化 Long 为字符串
        builder.modules(new JavaTimeModule());
    };
}
```

---

## 前端检查清单

### Composable 层

- [ ] 使用 `useServerTable` composable
- [ ] 配置默认排序字段为 `updatedAt`
- [ ] 配置默认排序方向为 `desc`

### API 层

- [ ] ID 字段使用 `string` 类型
- [ ] API 方法支持 `string | number` 类型参数

```typescript
export interface DictType {
  id: string  // 使用string避免精度丢失
  typeCode: string
  // ...
}

export const dictTypeApi = {
  deleteType(id: string | number): Promise<void> {
    return http.delete(`${BASE_URL}/${id}`)
  }
}
```

### 组件层

- [ ] 导入 `useServerTable`
- [ ] 使用 `query` 计算属性
- [ ] 使用 `watch` 监听 query 变化
- [ ] CRUD 操作后调用刷新方法

```typescript
const {
  query,
  sortField,
  sortOrder,
  totalCount,
  handlePageChange,
  handlePageSizeChange,
  handleSort,
  setTotal
} = useServerTable({
  defaultPageSize: 10,
  defaultSortField: 'updatedAt',
  defaultSortOrder: 'desc'
})

// 监听查询变化
watch(() => query.value, () => {
  loadData()
}, { deep: true })

// CRUD后刷新
const handleDelete = async (row: DictType) => {
  await api.delete(row.id)
  await loadData() // 必须刷新
}
```

### 模板层

- [ ] 表头添加排序图标
- [ ] 表头添加点击事件
- [ ] 分页器使用 `useServerTable` 返回的状态

```vue
<th @click="handleSort('updatedAt')">
  更新时间
  <SortIcon :is-active="sortField === 'updatedAt'" :order="sortOrder" />
</th>

<div class="pagination">
  <div class="pagination-info">
    共 {{ totalCount }} 条记录，第 {{ currentPage }}/{{ totalPages }} 页
  </div>
  <!-- 分页控件 -->
</div>
```

---

## 测试检查清单

### 功能测试

- [ ] 点击表头可以切换排序字段
- [ ] 再次点击同一表头可以切换排序方向
- [ ] 分页后排序状态保持
- [ ] 添加数据后统计数字更新
- [ ] 删除数据后统计数字更新
- [ ] 编辑数据后统计数字更新

### 数据验证

- [ ] ID 在前后端保持一致（不丢失精度）
- [ ] 分页数字是 number 类型（不是 string）
- [ ] 统计数字正确显示

### API 测试

```bash
# 测试分页API
curl "http://localhost:8080/api/v1/dict-types/page?pageNum=1&pageSize=10&sortBy=updatedAt&sortOrder=desc"

# 验证响应类型
{
  "records": [
    {
      "id": "2036605412000747522",  // string
      "itemCount": 5,  // number
      "updatedAt": "2026-03-25T08:45:43"
    }
  ],
  "total": 13,  // number
  "pageNum": 1,  // number
  "pageSize": 10,  // number
  "pages": 2  // number
}
```

---

## 常见问题

### Q1: 分页数字显示为字符串

**原因**: Jackson 配置了全局 Long 序列化。

**解决**: 移除全局 Long 序列化配置，ID 字段在 VO 层使用 String 类型。

### Q2: ID 在前端丢失精度

**原因**: 前端使用 number 类型存储雪花算法 ID。

**解决**: 前端接口定义 ID 字段使用 string 类型。

### Q3: 删除后统计数字不更新

**原因**: 只更新了本地数据，没有刷新列表。

**解决**: CRUD 操作成功后调用 `loadData()` 刷新列表。

### Q4: 排序状态丢失

**原因**: 使用前端排序或没有监听 query 变化。

**解决**: 使用 `useServerTable` 并监听 query 变化。

---

## 相关文档

- [Spring Boot规范 - Jackson序列化](../01-backend/spring-boot.md#jackson序列化规范强制)
- [Vue编码规范 - 服务端排序](../02-frontend/vue-coding.md#服务端排序与分页规范强制)
- [服务端排序与ID处理完整指南](../../docs/SERVER_SIDE_SORTING_AND_ID_HANDLING.md)

---

**维护者**: 开发团队
**最后更新**: 2026-03-25
