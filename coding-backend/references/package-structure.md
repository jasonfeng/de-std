# 包结构与模块划分规范

> 本规范定义项目的包结构和模块划分规则，确保代码组织的一致性和可维护性。

---

## 📋 目录

- [模块划分原则](#模块划分原则)
- [标准模块定义](#标准模块定义)
- [包结构规范](#包结构规范)
- [文件命名规范](#文件命名规范)
- [迁移规则](#迁移规则)
- [自动化检查](#自动化检查)
- [常见问题](#常见问题)

---

## 模块划分原则

### 核心原则

1. **业务领域聚合**：相关功能归入同一模块
2. **单一职责**：每个模块只负责一个业务领域
3. **模块-子功能结构**：采用`模块/子功能`的包结构
4. **避免循环依赖**：模块间依赖应单向流动

### 模块归并规则

| 原模块 | 归并后 | 原因 |
|--------|--------|------|
| user + role + permission | **iam** | 属于同一身份认证与授权领域 |
| form + guidance + help | **system** | 属于系统基础功能 |
| quality + task + execution | **quality** | 质量管理相关 |

---

## 标准模块定义

### iam 模块（身份与访问管理）

**职责**：用户认证、角色授权、权限管理

**子功能**：
- `user` - 用户管理
- `role` - 角色管理
- `permission` - 权限管理

**包结构示例**：
```
interfaces/controller/iam/user/UserController.java
interfaces/controller/iam/role/RoleController.java
interfaces/param/iam/user/UserParam.java
interfaces/vo/iam/user/UserVO.java
application/service/iam/user/UserAppService.java
domain/entity/iam/user/UserEntity.java
domain/repository/iam/user/UserMapper.java
domain/service/iam/user/UserDomainService.java
domain/enums/iam/user/UserStatusEnum.java
```

### system 模块（系统管理）

**职责**：系统基础功能、组织架构、用户引导、帮助文档

**子功能**：
- `department` - 组织架构管理
- `form` - 表单示例
- `guidance` - 用户引导
- `help` - 帮助文档

**包结构示例**：
```
interfaces/controller/system/department/DepartmentController.java
interfaces/controller/system/form/FormExampleController.java
interfaces/controller/system/guidance/TutorialController.java
interfaces/controller/system/help/HelpDocumentController.java
application/service/system/department/DepartmentAppService.java
domain/entity/system/department/DepartmentEntity.java
domain/repository/system/department/DepartmentRepository.java
```

### quality 模块（质量管理）

**职责**：数据质量规则、质量任务、质量执行

**子功能**：
- `rule` - 质量规则管理
- `task` - 质量任务管理
- `execution` - 质量执行记录

**包结构示例**：
```
interfaces/controller/quality/rule/QualityRuleController.java
interfaces/controller/quality/task/QualityTaskController.java
interfaces/controller/quality/execution/QualityExecutionController.java
application/service/quality/rule/QualityRuleAppService.java
domain/entity/quality/rule/QualityRuleEntity.java
domain/repository/quality/rule/QualityRuleRepository.java
```

### standard 模块（数据标准）

**职责**：数据标准管理、逻辑模型、物理模型、数据元素

**子功能**：
- `business-domain` - 业务域
- `logical-model` - 逻辑模型
- `physical-model` - 物理模型
- `data-element` - 数据元素
- `dict-type` - 字典类型
- `dict-item` - 字典项

**包结构示例**：
```
interfaces/controller/standard/logical-model/LogicalModelController.java
application/service/standard/logical-model/LogicalModelAppService.java
domain/entity/standard/logical-model/LogicalModelEntity.java
```

### task 模块（任务管理）

**职责**：数据集成任务、同步任务、批处理任务

**子功能**：
- `instance` - 任务实例
- `worker` - 任务执行器

**包结构示例**：
```
interfaces/controller/task/instance/TaskInstanceController.java
application/service/task/instance/TaskInstanceAppService.java
domain/entity/task/instance/TaskInstanceEntity.java
```

---

## 包结构规范

### 完整目录结构

```
src/main/java/com/dp/dataengine/
├── interfaces/                                # 【层级】接口层
│   ├── controller/                            # REST API控制器
│   │   ├── iam/                               # IAM模块
│   │   │   ├── user/
│   │   │   │   └── UserController.java
│   │   │   └── role/
│   │   │       └── RoleController.java
│   │   ├── system/                            # System模块
│   │   │   ├── department/
│   │   │   │   └── DepartmentController.java
│   │   │   ├── form/
│   │   │   ├── guidance/
│   │   │   └── help/
│   │   ├── quality/                           # Quality模块
│   │   │   ├── rule/
│   │   │   ├── task/
│   │   │   └── execution/
│   │   ├── standard/                          # Standard模块
│   │   │   ├── logical-model/
│   │   │   ├── data-element/
│   │   │   ├── dict-type/
│   │   │   └── dict-item/
│   │   └── task/                              # Task模块
│   │       ├── instance/
│   │       └── worker/
│   ├── param/                                 # 请求参数（前端→后端）
│   │   └── [与controller相同的结构]
│   └── vo/                                    # 响应对象（后端→前端）
│       └── [与controller相同的结构]
│
├── application/                               # 【层级】应用层
│   ├── service/                               # 应用服务
│   │   └── [与controller相同的结构]
│   ├── dto/                                   # 应用层DTO
│   │   └── [与controller相同的结构]
│   └── assembly/                              # DTO组装转换
│       ├── iam/
│       │   ├── user/
│       │   │   └── UserAssembly.java
│       │   └── role/
│       │       └── RoleAssembly.java
│       ├── quality/
│       │   └── QualityAppAssembly.java
│       ├── standard/
│       │   └── StandardAppAssembly.java
│       ├── task/
│       │   └── TaskAssembly.java
│       └── system/
│           └── DepartmentAppAssembly.java
│
├── domain/                                    # 【层级】领域层（贫血模型）
│   ├── entity/                                # 领域实体（带@TableName）⚠️
│   │   └── [与controller相同的结构]
│   ├── repository/                            # Repository（数据访问接口）⚠️
│   │   └── [与controller相同的结构]
│   │
│   ├── service/                               # 领域服务（业务逻辑）
│   │   └── [与controller相同的结构]
│   │
│   └── enums/                                 # 枚举类
│       ├── iam/
│       │   ├── user/
│       │   │   └── UserStatusEnum.java
│       │   └── role/
│       │       └── RoleStatusEnum.java
│       └── system/
│           └── DepartmentStatusEnum.java
│
├── infrastructure/                            # 【层级】基础设施层
│   ├── config/                                # 配置类
│   ├── cache/                                 # 缓存
│   ├── security/                              # 安全相关
│   └── exception/                             # 异常处理
│
└── common/                                    # 通用层
    ├── result/                                # 统一响应封装
    ├── enums/                                 # 通用枚举
    ├── exception/                             # 自定义异常
    └── util/                                  # 工具类
```

### 包命名规范

| 层级 | 模块 | 子功能 | 示例 |
|------|------|--------|------|
| interfaces | controller | iam/user | `interfaces.controller.iam.user.UserController` |
| interfaces | param | iam/user | `interfaces.param.iam.user.UserParam` |
| interfaces | vo | iam/user | `interfaces.vo.iam.user.UserVO` |
| application | service | iam/user | `application.service.iam.user.UserAppService` |
| application | dto | iam/user | `application.dto.iam.user.UserDTO` |
| application | assembly | iam/user | `application.assembly.iam.user.UserAssembly` |
| domain | entity | iam/user | `domain.entity.iam.user.UserEntity` |
| domain | repository | iam/user | `domain.repository.iam.user.UserMapper` |
| domain | service | iam/user | `domain.service.iam.user.UserDomainService` |
| domain | enums | iam/user | `domain.enums.iam.user.UserStatusEnum` |

---

## 文件命名规范

### 实体类

```java
// ✅ 正确
UserEntity.java
QualityRuleEntity.java
LogicalModelEntity.java

// ❌ 错误
User.java
QualityRule.java  // 缺少Entity后缀
```

### Repository

```java
// ✅ 正确
UserMapper.java
QualityRuleRepository.java
LogicalModelRepository.java

// ❌ 错误
UserDao.java
UserRepo.java
```

### 服务类

```java
// ✅ 正确
UserAppService.java
UserDomainService.java
UserAssembly.java

// ❌ 错误
UserService.java  // 不明确是AppService还是DomainService
User.java
```

### Controller类

```java
// ✅ 正确
UserController.java
QualityRuleController.java

// ❌ 错误
UserApi.java
UserControllerV1.java  // 版本号由URL路径管理
```

---

## 迁移规则

### 新增文件时

1. **确定所属模块**：根据业务功能确定文件所属模块
2. **确定子功能**：将文件归入正确的子功能包下
3. **检查依赖**：确保只依赖本模块或下游模块

```java
// ✅ 正确 - iam模块的Controller
package com.dp.dataengine.interfaces.controller.iam.user;

// ❌ 错误 - 放在了错误的模块下
package com.dp.dataengine.interfaces.controller.user;
```

### 重构现有代码时

1. **使用git mv**：保留文件历史
2. **更新导入语句**：所有相关文件的import语句
3. **检查测试文件**：同步更新测试文件的包名

```bash
# ✅ 正确 - 使用git mv保留历史
git mv old/path/UserEntity.java new/path/UserEntity.java

# ❌ 错误 - 使用mv或cp丢失历史
mv old/path/UserEntity.java new/path/UserEntity.java
cp old/path/UserEntity.java new/path/UserEntity.java
```

### 合并模块时

当需要合并多个模块时：

1. **创建新模块目录**：按照规范创建新的模块目录
2. **移动子功能**：将相关的子功能移动到新模块
3. **更新所有导入**：批量更新import语句
4. **删除空包**：清理空的目录

```bash
# 示例：将user和role合并为iam
mkdir -p interfaces/controller/iam/{user,role}
git mv interfaces/controller/user interfaces/controller/iam/user
git mv interfaces/controller/role interfaces/controller/iam/role
find . -name "*.java" -exec sed -i '' 's/interfaces\.controller\.user/interfaces.controller.iam.user/g' {} \;
```

---

## 自动化检查

### ArchUnit测试规则

在测试代码中添加ArchUnit规则，自动检查包结构：

```java
import com.tngtech.archunit.core.domain.JavaClasses;
import com.tngtech.archunit.core.importer.ImportOption;
import com.tngtech.archunit.junit.AnalyzeClasses;
import com.tngtech.archunit.junit.ArchTest;
import com.tngtech.archunit.lang.ArchRule;

import static com.tngtech.archunit.lang.syntax.ArchRuleDefinition.classes;

@AnalyzeClasses(packages = "com.dp.dataengine",
        importOptions = {ImportOption.DoNotIncludeTests.class})
public class PackageStructureTest {

    @ArchTest
    static final ArchRule controllers_should_be_in_correct_modules =
            controllers().should().resideInAPackage("..controller.(iam|system|quality|standard|task)..");

    @ArchTest
    static final ArchRule entities_should_have_Entity_suffix =
            classes().that().areAnnotatedWith(TableName.class)
                    .should().haveSimpleNameEndingWith("Entity");

    @ArchTest
    static final ArchRule services_should_be_in_correct_package =
            classes().that().areAnnotatedWith(Service.class)
                    .should().resideInAPackage("..service..");

    @ArchTest
    static final ArchRule repositories_should_not_be_in_infrastructure =
            classes().that().implement(Mapper.class)
                    .should().not().resideInAPackage("..infrastructure..");

    @ArchTest
    static final ArchRule domain_should_not_depend_on_interfaces =
            noClasses().that().resideInAPackage("..domain..")
                    .should().dependOnClassesThat().resideInAPackage("..interfaces..");

    @ArchTest
    static final ArchRule empty_packages_should_not_exist =
            noClasses().should().resideInEmptyPackage();
}
```

### Pre-commit Hook

创建`.git/hooks/pre-commit`脚本，自动检查包结构：

```bash
#!/bin/bash

# 检查新增的Java文件是否在正确的包路径下
NEW_JAVA_FILES=$(git diff --cached --name-only --diff-filter=A | grep '\.java$')

if [ -n "$NEW_JAVA_FILES" ]; then
    echo "检查新增Java文件的包结构..."

    for file in $NEW_JAVA_FILES; do
        # 检查文件是否在interfaces/application/domain下
        if [[ $file =~ src/main/java/com/dp/dataengine/(interfaces|application|domain)/ ]]; then
            # 检查是否在正确的模块下（iam|system|quality|standard|task）
            if ! [[ $file =~ src/main/java/com/dp/dataengine/(interfaces|application|domain)/(iam|system|quality|standard|task)/ ]]; then
                echo "❌ 错误：文件不在正确的模块下"
                echo "   文件：$file"
                echo "   预期路径：src/main/java/com/dp/dataengine/{interfaces|application|domain}/{iam|system|quality|standard|task}/..."
                exit 1
            fi
        fi
    done

    echo "✅ 包结构检查通过"
fi

# 运行ArchUnit测试
mvn test -Dtest=PackageStructureTest -q
if [ $? -ne 0 ]; then
    echo "❌ ArchUnit测试失败"
    exit 1
fi

exit 0
```

---

## 常见问题

### Q1: 如何确定新功能属于哪个模块？

**A**: 根据以下优先级判断：

1. **是否属于身份认证/授权？** → iam模块
2. **是否属于系统基础功能？** → system模块
3. **是否属于数据质量管理？** → quality模块
4. **是否属于数据标准管理？** → standard模块
5. **是否属于任务执行？** → task模块
6. **都不匹配？** → 创建新模块（需团队讨论）

### Q2: Enum应该放在哪个包下？

**A**: Enum放在`domain/enums/`下，按模块组织：

```
domain/enums/
├── iam/
│   ├── user/UserStatusEnum.java
│   └── role/RoleStatusEnum.java
└── system/
    └── DepartmentStatusEnum.java
```

### Q3: Assembly应该放在哪里？

**A**: Assembly按模块组织：

```
application/assembly/
├── iam/user/UserAssembly.java
├── iam/role/RoleAssembly.java
├── quality/QualityAppAssembly.java
└── standard/StandardAppAssembly.java
```

### Q4: 如何处理跨模块依赖？

**A**: 跨模块依赖必须通过AppService编排，DomainService之间不能直接调用：

```java
// ❌ 错误 - DomainService直接调用其他模块的DomainService
@Service
public class UserDomainService {
    private final OrderDomainService orderDomainService;  // 禁止
}

// ✅ 正确 - 通过AppService编排
@Service
public class UserAppService {
    private final UserDomainService userDomainService;
    private final OrderDomainService orderDomainService;  // 允许
}
```

### Q5: Mapper文件放在哪里？

**A**: Mapper XML文件按模块组织在`src/main/resources/mapper/`下：

```
src/main/resources/mapper/
├── iam/user/UserMapper.xml
├── system/department/DepartmentRepository.xml
├── quality/rule/QualityRuleRepository.xml
└── standard/logical-model/LogicalModelRepository.xml
```

### Q6: 如何处理空包？

**A**: 提交前必须清理空包：

```bash
# 查找空包
find src/main/java -type d -empty

# 删除空包
find src/main/java -type d -empty -delete
```

---

## 🔗 相关文档

- [DDD架构规范](./ddd-architecture.md) - DDD架构详细规范
- [Spring Boot规范](./spring-boot.md) - Spring Boot详细规范
- [MyBatis Plus规范](./mybatis-plus.md) - MyBatis Plus详细规范
- [Java编码规范](./java-coding.md) - Java编码详细规范

---

**最后更新**：2026-03-14
**版本**：1.0.0