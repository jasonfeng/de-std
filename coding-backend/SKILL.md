---
name: coding-backend
description: 后端 Java/Spring Boot/MyBatis Plus 编码规范。写 Java 代码前必须调用。
---

# 后端编码规范

## 触发时机

用户说以下内容时**必须**先调用此 Skill：

- "写一个 Service/Mapper/Controller"
- "添加后端 API"
- "创建 Entity/DTO/VO"
- "实现业务逻辑"

## 调用时机

- 写 Java 类之前
- 写 DomainService/AppService/Controller 之前
- 写 Mapper/Repository 之前
- 写任何后端代码之前

## DDD分层架构规范

### 包结构

```
com.dp.dataengine/
├── interfaces/
│   ├── controller/    # REST API
│   ├── param/         # 请求参数
│   └── vo/            # 响应视图
├── application/
│   ├── service/       # AppService（事务编排）
│   ├── dto/           # DTO
│   └── assembly/      # 跨层转换
├── domain/
│   ├── entity/        # Entity（必须带Entity后缀）
│   ├── service/       # DomainService（业务逻辑）
│   └── repository/    # Repository
├── infrastructure/
│   ├── config/
│   └── security/
└── common/
```

### 调用关系

```
Controller → AppService → DomainService → Repository
```

**禁止**：DomainService 直接调用其他 DomainService

---

## Spring Boot 规范

### 依赖注入

```java
// ❌ 禁止
@Autowired
private UserMapper userMapper;

// ✅ 正确
@Service
@RequiredArgsConstructor
public class UserAppService {
    private final UserMapper userMapper;
    private final UserDomainService userDomainService;
}
```

### 事务管理

```java
// AppService：事务边界
@Service
@RequiredArgsConstructor
public class OrderAppService {
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(OrderCreateParam param) {
        // ...
    }
}

// DomainService：无事务注解
@Component
public class OrderDomainService {
    // ❌ 禁止 @Transactional
    public void validateOrder(OrderEntity order) {
        // ...
    }
}
```

---

## MyBatis Plus 规范

### SQL必须在XML文件中

```java
// ❌ 禁止使用注解
@Select("SELECT * FROM t_user")
List<UserEntity> findAll();

// ✅ 正确：XML方式
public interface UserMapper extends BaseMapper<UserEntity> {
    List<UserEntity> findAll();
}
```

```xml
<!-- resources/mapper/user/UserMapper.xml -->
<select id="findAll" resultType="...UserEntity">
    SELECT * FROM t_user WHERE deleted = 0
</select>
```

### 主键策略

```java
@TableId(value = "id", type = IdType.ASSIGN_ID)  // ✅ 雪花算法
private Long id;
```

### 逻辑删除

```java
@TableLogic
private Integer deleted;  // 0=未删除, 1=已删除
```

---

## DTO/VO/Param 规范

| 类型 | 位置 | 用途 | 示例 |
|------|------|------|------|
| Param | interfaces/param | 接收请求 | `UserCreateParam` |
| VO | interfaces/vo | 返回响应 | `UserVO` |
| DTO | application/dto | 跨层传递 | `UserDTO` |
| Entity | domain/entity | 数据库映射 | `UserEntity` |

---

## 禁止行为汇总

| 行为 | 原因 |
|------|------|
| `@Autowired` 字段注入 | 无法使用 final，测试困难 |
| `@Select/@Insert` 注解 | SQL难以维护和审查 |
| `IdType.AUTO` 自增 | 分布式不友好 |
| DomainService 加 `@Transactional` | 事务边界混乱 |
| 留空 `// TODO` 注释 | 所有问题必须解决 |