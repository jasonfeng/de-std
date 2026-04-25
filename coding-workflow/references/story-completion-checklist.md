# Story 完成规范 Checklist

**版本**: 1.1
**更新日期**: 2026-04-11
**用途**: Story 完成前必须逐项检查，确保符合所有规范

---

## 📋 使用说明

1. **何时使用**: Story 开发完成后，提交代码审查前
2. **如何使用**: 逐项检查，勾选通过的项目
3. **必须全部通过**: 任何一项未通过都需要修复后重新检查
4. **保存 checklist**: 将 checklist 作为 PR 描述的一部分

---

## 🚨 CRITICAL 检查项（必须全部通过）

### Entity 类检查

- [ ] **Entity 类名以 Entity 结尾**
  - 示例: `UserEntity.java` ✅ | `User.java` ❌
  - 位置: `domain/entity/{module}/`

- [ ] **Entity 类包含完整的 MyBatis Plus 注解**
  - `@TableName("t_xxx")` - 指定表名
  - `@TableId(value = "id", type = IdType.ASSIGN_ID)` - 雪花算法主键
  - `@TableField("xxx")` - 所有字段
  - `@TableField(value = "created_by", fill = FieldFill.INSERT)` - 创建人
  - `@TableField(value = "created_time", fill = FieldFill.INSERT)` - 创建时间
  - `@TableField(value = "updated_by", fill = FieldFill.INSERT_UPDATE)` - 更新人
  - `@TableField(value = "updated_time", fill = FieldFill.INSERT_UPDATE)` - 更新时间
  - `@TableLogic` - 软删除字段（deleted）

- [ ] **Entity 类包含完整的 JavaDoc 注释**
  - 类级别 JavaDoc（包含业务描述）
  - 所有字段的 JavaDoc

### SQL 安全检查

- [ ] **SQL 必须在 XML 文件中**
  - 位置: `resources/mapper/{module}/XxxMapper.xml`
  - 禁止使用 @Select/@Insert 等注解

- [ ] **表名/字段名使用参数化或标识符转义**
  - 防止 SQL 注入
  - 使用 `DatabaseAccessValidator.escapeIdentifier()` 或 `isValidTableName()`

- [ ] **用户输入必须验证**
  - 文件上传验证路径遍历攻击
  - 表名/字段名验证只包含合法字符
  - 参数长度限制

### 安全检查

- [ ] **文件上传安全验证**
  - 文件名不包含 `..`, `/`, `\`
  - 文件类型与扩展名匹配
  - 文件大小在限制范围内
  - 使用 `toAbsolutePath().normalize()` 规范化路径
  - 使用 `startsWith()` 验证路径在允许范围内

- [ ] **敏感信息日志脱敏**
  - 密码、token 等敏感信息不记录到日志
  - 使用脱敏工具类

### 代码规范检查

- [ ] **使用构造器注入，禁止 @Autowired**
  - 使用 `@RequiredArgsConstructor` (Lombok)
  - 字段使用 `private final`

- [ ] **使用 BusinessException 而非 RuntimeException**
  - 格式: `new BusinessException("CODE", "message")`

- [ ] **使用枚举替代魔法值**
  - 状态、类型等使用枚举
  - 避免硬编码字符串

---

## 🟠 HIGH 检查项（必须全部通过）

### 分层架构检查

- [ ] **严格遵循 DDD 分层架构**
  ```
  interfaces/      → Controller、Param、VO
  application/     → AppService、DTO、Assembly
  domain/         → Entity、DomainService、Repository
  infrastructure/  → Config、Security、Exception
  common/         → Result、Enums、Exception、Util
  ```

- [ ] **DomainService 不调用 interfaces 层**
  - DomainService 只能调用 Repository
  - 跨层事务编排在 AppService

- [ ] **DTO 转换使用 Assembly 层**
  - 所有 Entity ↔ DTO 转换在 Assembly 层
  - 禁止在 Service 层直接转换

### Repository 检查

- [ ] **列表查询方法支持分页**
  - 使用 MyBatis Plus 的 `Page<T>`
  - 提供 IPage 返回类型

- [ ] **使用 MyBatis Plus 的 LambdaQueryWrapper**
  - 避免硬编码字段名
  - 类型安全的查询

### Controller 检查

- [ ] **路径变量添加正则验证**
  ```java
  @GetMapping("/{id:\\d+}")
  @GetMapping("/{tableName:\\w+}")
  ```

- [ ] **请求参数使用 @Valid 验证**
  - Param 类添加验证注解
  - Controller 方法添加 @Valid

- [ ] **使用正确的 HTTP 状态码**
  - 200: 成功
  - 201: 创建成功
  - 400: 参数错误
  - 404: 资源不存在
  - 500: 服务器错误

### 测试检查

- [ ] **DomainService 测试覆盖率 ≥ 80%**
  - 使用 JaCoCo 验证覆盖率
  - 包含成功和失败场景

- [ ] **测试类命名规范**
  - `XxxDomainServiceTest.java`
  - `XxxAppServiceTest.java`
  - `XxxControllerTest.java`

---

## 🟡 MEDIUM 检查项（建议全部通过）

### 代码质量

- [ ] **无重复代码**
  - 相同逻辑提取为公共方法
  - 使用工具类

- [ ] **变量命名有意义**
  - 避免单字母变量（除循环变量）
  - 使用有意义的名称

- [ ] **代码格式统一**
  - 使用项目统一的代码格式化配置
  - 无不必要的空行

### 性能考虑

- [ ] **无 N+1 查询问题**
  - 使用批量查询
  - 使用 `@TableField(select = false)` 延迟加载

- [ ] **大数据量查询使用分页**
  - 避免 `SELECT *`
  - 使用索引字段查询

### 事务管理

- [ ] **事务边界在 AppService 层**
  - AppService 方法添加 `@Transactional`
  - DomainService 不管理事务

---

## 📝 Story 提交前检查清单

### 代码提交

- [ ] 代码已提交到 Git
- [ ] 提交信息清晰（格式: `type(scope): description`）
- [ ] 无调试代码（如 `System.out.println`）
- [ ] 无调试截图在根目录（PNG 等临时文件应删除或放 `docs/screenshots/`）
- [ ] 无 TODO 注释（或 TODO 有对应 Issue）

### 文档更新

- [ ] API 文档已更新（如有新增接口）
- [ ] 数据库迁移脚本已提交（如有表结构变更）
- [ ] README 已更新（如需要）

### Checklist 本身

- [ ] 本 checklist 已逐项检查
- [ ] 所有 CRITICAL 项已通过
- [ ] 所有 HIGH 项已通过
- [ ] MEDIUM 项已评估

---

## ✅ 签名

**开发者**: _________________  **日期**: ________

**代码审查者**: _________________  **日期**: ________

---

**注意**: 本 checklist 将持续更新，请使用最新版本。

**最后更新**: 2026-03-14
**维护者**: Bob (Scrum Master)
