# DDD架构规范

> 本规范定义DDD（领域驱动设计）架构的使用规范，确保架构的一致性和可维护性。

---

## 📋 目录

- [5层架构结构](#5层架构结构)
- [分层职责](#分层职责)
- [调用关系](#调用关系)
- [双Assembly设计](#双assembly设计)
- [开发示例](#开发示例)
- [核心约束](#核心约束)

---

## 5层架构结构

### 架构层次

```
interfaces/      → 接口层：对外接口、参数、响应
application/     → 应用层：业务流程编排、事务控制
domain/         → 领域层：核心业务逻辑⭐核心
infrastructure/ → 基础设施层：技术支持（非DB操作）
common/         → 通用层：公共组件
```

### 目录结构

```
src/main/java/com/dp/dataengine/
├── interfaces/                        # 【层级】接口层
│   ├── controller/{module}/           # REST API控制器
│   ├── param/{module}/                # 请求参数（前端→后端）
│   └── vo/{module}/                   # 响应对象（后端→前端）
│
├── application/                       # 【层级】应用层
│   ├── service/{module}/              # 应用服务（直接写类，非必要不接口）
│   ├── dto/{module}/                  # 应用层DTO（合并command/query）
│   └── assembly/                      # DTO组装转换（使用Apache BeanUtils）
│
├── domain/                            # 【层级】领域层（贫血模型）
│   ├── entity/{module}/               # 领域实体（带@TableName）⚠️
│   ├── repository/{module}/           # Repository（数据访问接口）⚠️
│   ├── dto/{module}/                  # 领域层DTO
│   ├── assembly/                      # 领域层DTO和Entity转换
│   └── service/{module}/              # 领域服务（业务逻辑）
│
├── infrastructure/                    # 【层级】基础设施层（非DB操作）⚠️
│   ├── config/                        # 配置类
│   ├── cache/                         # 缓存
│   ├── security/                      # 安全相关
│   └── exception/                     # 异常处理
│
└── common/                            # 通用层
    ├── result/                        # 统一响应封装
    ├── enums/                         # 枚举类
    ├── exception/                     # 自定义异常
    └── util/                          # 工具类
```

### 包命名规范

| 层级 | 子包 | 模块 | 示例 |
|------|------|------|------|
| interfaces | controller | user | `interfaces.controller.user.UserController` |
| interfaces | param | user | `interfaces.param.user.UserParam` |
| interfaces | vo | user | `interfaces.vo.user.UserVO` |
| application | service | user | `application.service.user.UserAppService` |
| application | dto | user | `application.dto.user.UserDTO` |
| domain | entity | user | `domain.entity.user.UserEntity` ⚠️ |
| domain | repository | user | `domain.repository.user.UserRepository` ⚠️ |
| domain | service | user | `domain.service.user.UserDomainService` |

---

## 分层职责

### interfaces - 接口层

**职责**：
- 接收HTTP请求
- 参数验证
- 转换Param → DTO
- 调用AppService
- 转换DTO → VO
- 返回HTTP响应

**规范**：
- 只调用AppService
- 不包含业务逻辑
- 转换Param/VO

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserAppService userAppService;
    private final UserAssembly userAssembly;

    /**
     * 创建用户。
     */
    @PostMapping
    public ResponseEntity<UserVO> createUser(@Valid @RequestBody CreateUserParam param) {
        // Param → DTO
        UserDTO dto = userAssembly.toDTO(param);

        // 调用AppService
        UserDTO created = userAppService.createUser(dto);

        // DTO → VO
        UserVO vo = userAssembly.toVO(created);

        return ResponseEntity.status(HttpStatus.CREATED).body(vo);
    }
}
```

### application - 应用层

**职责**：
- 业务流程编排
- 事务控制
- 调用多个DomainService
- 转换DTO/Entity

**规范**：
- 使用`@Transactional`
- 调用DomainService
- 不直接操作Repository

```java
@Service
@Transactional
@RequiredArgsConstructor
public class UserAppService {

    private final UserDomainService userDomainService;
    private final DepartmentDomainService departmentDomainService;

    /**
     * 创建用户并分配部门。
     */
    public UserDTO createUserWithDepartment(UserDTO userDto, Long departmentId) {
        // 验证部门
        departmentDomainService.validateDepartment(departmentId);

        // 创建用户
        UserDTO created = userDomainService.createUser(userDto);

        // 分配部门
        userDomainService.assignDepartment(created.getId(), departmentId);

        return created;
    }
}
```

### domain - 领域层（⭐核心）

**职责**：
- 单一业务领域的核心逻辑
- Entity操作
- 调用Repository
- 使用Domain Assembly

**规范**：
- 包含核心业务逻辑
- 可调用Repository
- 不直接调用其他DomainService（跨领域通过AppService编排）

```java
@Service
@RequiredArgsConstructor
public class UserDomainService {

    private final UserRepository userRepository;
    private final UserDomainAssembly userAssembly;

    /**
     * 创建用户。
     */
    public UserDTO createUser(UserDTO dto) {
        // 业务逻辑
        validateUsername(dto.getUsername());

        // DTO → Entity
        UserEntity entity = userAssembly.toEntity(dto);

        // 插入数据库
        userRepository.insert(entity);

        // Entity → DTO
        return userAssembly.toDTO(entity);
    }

    private void validateUsername(String username) {
        if (StringUtils.isBlank(username)) {
            throw new BusinessException("用户名不能为空");
        }
        if (username.length() < 3 || username.length() > 50) {
            throw new BusinessException("用户名长度必须在3-50之间");
        }
        if (existsByUsername(username)) {
            throw new BusinessException("用户名已存在");
        }
    }

    private boolean existsByUsername(String username) {
        return userRepository.existsByUsername(username);
    }
}
```

### infrastructure - 基础设施层

**职责**：
- 技术支持
- 配置类
- 缓存
- 安全
- 异常处理

**规范**：
- 非DB操作
- 技术细节封装

```java
@Configuration
public class MyBatisConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}

@Service
public class RedisCacheService {
    public void set(String key, Object value) {
        // Redis操作
    }

    public Object get(String key) {
        // Redis操作
    }
}
```

### common - 通用层

**职责**：
- 通用组件
- 工具类
- 统一响应封装
- 枚举类
- 自定义异常

```java
/**
 * 统一响应结果。
 */
@Data
public class Result<T> {
    private Integer code;
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        return result;
    }

    public static <T> Result<T> error(String message) {
        Result<T> result = new Result<>();
        result.setCode(500);
        result.setMessage(message);
        return result;
    }
}
```

---

## 调用关系

### 基本调用链

```
Controller → AppService → DomainService → Repository → Database
   ↓          ↓              ↓
Param      事务编排        业务逻辑+操作Entity
VO        转换DTO
```

### 详细流程

```
1. Controller接收HTTP请求
   ↓
2. Controller转换Param → DTO
   ↓
3. Controller调用AppService
   ↓
4. AppService开启事务
   ↓
5. AppService调用DomainService（业务逻辑）
   ↓
6. DomainService转换DTO → Entity
   ↓
7. DomainService调用Repository（数据访问）
   ↓
8. Repository执行SQL（XML）
   ↓
9. AppService提交事务
   ↓
10. AppService返回DTO
   ↓
11. Controller转换DTO → VO
   ↓
12. Controller返回HTTP响应
```

### 跨领域调用

**错误方式**：
```
UserDomainService → OrderDomainService  // ❌ 领域服务直接调用其他领域服务
```

**正确方式**：
```
UserDomainService → UserAppService → OrderDomainService  // ✅ 通过AppService编排
```

---

## 双Assembly设计

### Domain Assembly（领域层Assembly）

**位置**：`domain/assembly/`

**职责**：Entity ↔ DomainDTO

**示例**：
```java
package com.dp.dataengine.domain.assembly.user;

import com.dp.dataengine.domain.dto.user.UserDTO;
import com.dp.dataengine.domain.entity.user.UserEntity;
import org.springframework.stereotype.Component;

/**
 * 用户领域组装工具。
 */
@Component
public class UserDomainAssembly {

    /**
     * Entity转DTO。
     */
    public UserDTO toDTO(UserEntity entity) {
        if (entity == null) {
            return null;
        }

        UserDTO dto = new UserDTO();
        dto.setId(entity.getId());
        dto.setUsername(entity.getUsername());
        dto.setEmail(entity.getEmail());
        dto.setDepartmentId(entity.getDepartmentId());
        dto.setDeleted(entity.getDeleted());
        dto.setCreatedAt(entity.getCreatedAt());
        dto.setUpdatedAt(entity.getUpdatedAt());
        return dto;
    }

    /**
     * DTO转Entity。
     */
    public UserEntity toEntity(UserDTO dto) {
        if (dto == null) {
            return null;
        }

        UserEntity entity = new UserEntity();
        entity.setId(dto.getId());
        entity.setUsername(dto.getUsername());
        entity.setEmail(dto.getEmail());
        entity.setDepartmentId(dto.getDepartmentId());
        entity.setDeleted(dto.getDeleted());
        entity.setCreatedAt(dto.getCreatedAt());
        entity.setUpdatedAt(dto.getUpdatedAt());
        return entity;
    }

    /**
     * Entity列表转DTO列表。
     */
    public List<UserDTO> toDTOList(List<UserEntity> entities) {
        if (entities == null || entities.isEmpty()) {
            return new ArrayList<>();
        }
        return entities.stream()
                .map(this::toDTO)
                .collect(Collectors.toList());
    }
}
```

### Application Assembly（应用层Assembly）

**位置**：`application/assembly/`

**职责**：Param/VO/DTO ↔ Entity/DomainDTO

**示例**：
```java
package com.dp.dataengine.application.assembly.user;

import com.dp.dataengine.application.dto.user.UserDTO;
import com.dp.dataengine.domain.dto.user.UserDomainDTO;
import com.dp.dataengine.domain.entity.user.UserEntity;
import com.dp.dataengine.interfaces.param.user.CreateUserParam;
import com.dp.dataengine.interfaces.vo.user.UserVO;
import org.springframework.stereotype.Component;

/**
 * 用户应用组装工具。
 */
@Component
public class UserAssembly {

    /**
     * Param转DTO。
     */
    public UserDTO toDTO(CreateUserParam param) {
        if (param == null) {
            return null;
        }

        UserDTO dto = new UserDTO();
        dto.setUsername(param.getUsername());
        dto.setEmail(param.getEmail());
        dto.setDepartmentId(param.getDepartmentId());
        return dto;
    }

    /**
     * DTO转VO。
     */
    public UserVO toVO(UserDTO dto) {
        if (dto == null) {
            return null;
        }

        UserVO vo = new UserVO();
        vo.setId(dto.getId());
        vo.setUsername(dto.getUsername());
        vo.setEmail(dto.getEmail());
        vo.setDepartmentId(dto.getDepartmentId());
        return vo;
    }

    /**
     * DTO列表转VO列表。
     */
    public List<UserVO> toVOList(List<UserDTO> dtoList) {
        if (dtoList == null || dtoList.isEmpty()) {
            return new ArrayList<>();
        }
        return dtoList.stream()
                .map(this::toVO)
                .collect(Collectors.toList());
    }
}
```

---

## 开发示例

### 完整示例：创建用户

#### 1. Entity（领域层）

```java
@Data
@TableName("t_user")
public class UserEntity {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    private String username;
    private String email;
    private Long departmentId;
    @TableLogic
    private Integer deleted;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;
}
```

#### 2. DomainDTO（领域层）

```java
@Data
public class UserDTO {
    private Long id;
    private String username;
    private String email;
    private Long departmentId;
    private Integer deleted;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

#### 3. DomainAssembly（领域层）

```java
@Component
public class UserDomainAssembly {
    public UserDTO toDTO(UserEntity entity) { /* ... */ }
    public UserEntity toEntity(UserDTO dto) { /* ... */ }
}
```

#### 4. Repository（领域层）

```java
@Mapper
public interface UserRepository extends BaseMapper<UserEntity> {
    UserEntity findByUsername(String username);
    boolean existsByUsername(String username);
}
```

#### 5. DomainService（领域层）

```java
@Service
@RequiredArgsConstructor
public class UserDomainService {
    private final UserRepository userRepository;
    private final UserDomainAssembly userAssembly;

    public UserDTO createUser(UserDTO dto) {
        validateUsername(dto.getUsername());

        UserEntity entity = userAssembly.toEntity(dto);
        userRepository.insert(entity);

        return userAssembly.toDTO(entity);
    }

    private void validateUsername(String username) { /* ... */ }
}
```

#### 6. AppService（应用层）

```java
@Service
@Transactional
@RequiredArgsConstructor
public class UserAppService {
    private final UserDomainService userDomainService;
    private final DepartmentDomainService departmentDomainService;

    public UserDTO createUser(UserDTO dto) {
        // 编排业务流程
        return userDomainService.createUser(dto);
    }
}
```

#### 7. Param（接口层）

```java
@Data
public class CreateUserParam {
    @NotBlank(message = "用户名不能为空")
    private String username;

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;

    private Long departmentId;
}
```

#### 8. VO（接口层）

```java
@Data
public class UserVO {
    private Long id;
    private String username;
    private String email;
    private Long departmentId;
}
```

#### 9. Application Assembly（应用层）

```java
@Component
public class UserAssembly {
    public UserDTO toDTO(CreateUserParam param) { /* ... */ }
    public UserVO toVO(UserDTO dto) { /* ... */ }
}
```

#### 10. Controller（接口层）

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserAppService userAppService;
    private final UserAssembly userAssembly;

    @PostMapping
    public ResponseEntity<UserVO> createUser(@Valid @RequestBody CreateUserParam param) {
        UserDTO dto = userAssembly.toDTO(param);
        UserDTO created = userAppService.createUser(dto);
        UserVO vo = userAssembly.toVO(created);
        return ResponseEntity.status(HttpStatus.CREATED).body(vo);
    }
}
```

---

## 核心约束

| 设计点 | 规范要求 |
|--------|---------|
| **Entity位置** | `domain/entity/{module}/`（必须带Entity后缀） |
| **Repository位置** | `domain/repository/{module}/` |
| **基础设施层** | 非DB操作（配置/缓存/MQ） |
| **贫血模型** | Entity只有数据，业务逻辑在DomainService |
| **非必要不接口** | 直接写实现类 |
| **跨领域调用** | 通过AppService编排，不直接调用其他DomainService |

---

## 🔗 相关文档

- [Spring Boot规范](./spring-boot.md) - Spring Boot详细规范
- [MyBatis Plus规范](./mybatis-plus.md) - MyBatis Plus详细规范
- [Java编码规范](./java-coding.md) - Java编码详细规范

---

**最后更新**：2026-03-10