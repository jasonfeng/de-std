# 前后端联调规范

> 本规范定义前后端联调的检查清单和验证流程，确保功能交付的完整性。

---

## 📋 目录

- [问题背景](#问题背景)
- [检查清单](#检查清单)
- [端到端验证流程](#端到端验证流程)
- [关键代码模式识别](#关键代码模式识别)

---

## 问题背景

### 常见问题

| 问题 | 根因 | 影响 |
|------|------|------|
| 编辑时字段丢失 | 前端硬编码空值或后端未返回 | 用户无法看到/修改数据 |
| 字段不显示 | 后端注释"不返回"或前端未映射 | 数据丢失 |
| 操作改时间戳 | `updateById` 触发自动填充 | 数据被意外修改 |
| 配置不一致 | MyBatis字段名与Entity不匹配 | 自动填充失效 |

### 根本原因

**只测试后端 API（curl），没有验证完整的前后端流程。**

---

## 检查清单

### 1. 代码全链路审查

修改任何字段/接口，必须检查完整链路：

```
Entity → Assembly → VO/DTO → Controller → 前端API类型 → transform → 组件使用
```

| 层级 | 检查点 | 常见问题 |
|------|--------|----------|
| **Entity** | 字段名、类型是否正确 | `createdTime` vs `createdAt` |
| **Assembly** | 是否有"不返回"注释 | `// 注意：密码不返回给前端` |
| **VO/DTO** | 是否包含该字段 | 缺少 `password` 字段 |
| **Service** | 是否正确处理 | 解密密码、格式转换 |
| **前端 API 类型** | `RawXxx` 接口定义 | 缺少字段定义 |
| **transform** | 是否映射该字段 | 遗漏字段映射 |
| **组件** | 是否正确使用字段 | 硬编码空值如 `password: ''` |

### 2. 配置一致性检查

- [ ] MyBatis 自动填充字段名 ≟ Entity 字段名
- [ ] 数据库列名 ≟ Entity `@TableField` 值
- [ ] 前后端枚举值一致（大小写、命名）
- [ ] 前端 API 类型与后端 VO 字段一致

### 3. 快速检查命令

```bash
# 1. 搜索"不返回"注释，评估是否应返回
grep -r "不返回" backend/ --include="*.java"

# 2. 检查 MyBatis 自动填充字段名
grep -A5 "insertFill\|updateFill" backend/**/MybatisPlusConfig.java

# 3. 检查前端组件硬编码空值
grep -r ": ''" frontend/ --include="*.vue" --include="*.ts"

# 4. 检查 API 类型定义完整性
grep -A20 "interface Raw" frontend/**/api/*.ts
```

---

## 端到端验证流程

### 必须使用浏览器实际操作验证

**不能只依赖 curl 测试后端 API！**

```
1. 编译后端 → mvn package -Dmaven.test.skip=true
2. 部署服务 → docker compose up -d
3. 刷新前端页面
4. 实际点击操作验证
5. 检查浏览器 DevTools Network 请求响应
6. 检查前端组件是否正确渲染返回数据
```

### 验证检查项

| 检查项 | 验证方法 |
|--------|----------|
| 列表数据正确 | 打开列表页，检查数据是否显示 |
| 编辑数据正确 | 点击编辑，检查表单是否填充正确 |
| 保存成功 | 修改数据，保存后刷新验证 |
| 时间正确 | 检查创建时间、修改时间是否显示 |
| 测试操作不改时间 | 测试连接后检查修改时间是否不变 |

---

## 关键代码模式识别

### 高风险模式

| 模式 | 风险 | 正确做法 |
|------|------|----------|
| `password: ''` | 编辑时密码丢失 | 从后端获取 `password: row.password` |
| `// 不返回` 注释 | 字段被隐藏 | 评估是否应返回，修改 Assembly |
| `updateById(entity)` | 触发自动填充改时间戳 | 用自定义 SQL 规避 |
| `status: 'healthy'` | 状态转换逻辑错误 | 检查映射是否正确 |

### 正确示例

**Assembly 返回敏感字段：**
```java
// 解密密码返回给前端（用户要求必须看到密码）
if (entity.getPasswordEncrypted() != null) {
    vo.setPassword(domainService.decryptPassword(entity));
}
```

**前端正确获取字段：**
```typescript
const handleEdit = (row: ApiDataSource) => {
  Object.assign(formData, {
    // ...
    password: row.password || '',  // 从后端获取
  })
}
```

**自定义 SQL 避免自动填充：**
```xml
<update id="updateConnectionTestResult">
    UPDATE t_datasource
    SET last_connection_test = #{lastConnectionTest},
        connection_test_result = #{connectionTestResult},
        status = #{status}
    WHERE id = #{id}
</update>
```

---

## 🔗 相关文档

- [Story完成规范](./story-completion.md) - Story验收标准
- [代码审查规范](./code-review.md) - 代码审查详细规范
- [E2E测试规范](../04-testing/e2e-testing.md) - E2E测试完整规范

---

**最后更新**：2026-03-29