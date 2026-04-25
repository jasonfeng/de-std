# 测试用例模板

**更新日期**: 2026-03-14
**用途**: 快速创建符合规范的测试用例

---

## 🧪 DomainService 测试模板

### 必需组件

```java
package com.dp.dataengine.domain.service.{module};

import com.dp.dataengine.domain.entity.{module}.{Entity}Entity;
import com.dp.dataengine.domain.repository.{module}.{Entity}Repository;
import com.dp.dataengine.infrastructure.exception.BusinessException;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

/**
 * {DomainService} 测试
 */
@ExtendWith(MockitoExtension.class)
@DisplayName("{DomainService} 测试")
class {DomainService}Test {

    @Mock
    private {Entity}Repository repository;

    @InjectMocks
    private {DomainService} domainService;

    private {Entity}Entity testEntity;

    @BeforeEach
    void setUp() {
        testEntity = new {Entity}Entity();
        // 设置测试数据
    }

    @Nested
    @DisplayName("创建{Entity}方法测试")
    class Create{Entity}Tests {

        @Test
        @DisplayName("成功创建{Entity}")
        void create{Entity}_Success() {
            // Given
            when(repository.insert(any())).thenReturn(1);

            // When
            Long id = domainService.create(testEntity);

            // Then
            assertNotNull(id);
            verify(repository).insert(testEntity);
        }

        @Test
        @DisplayName("创建{Entity}时参数为空抛出异常")
        void create{Entity}_NullParameter_ThrowsException() {
            // Given
            testEntity.setName(null); // 设置必填字段为空

            // When & Then
            assertThrows(BusinessException.class, () -> {
                domainService.create(testEntity);
            });

            verify(repository, never()).insert(any());
        }
    }

    @Nested
    @DisplayName("查询{Entity}方法测试")
    class Get{Entity}Tests {

        @Test
        @DisplayName("根据ID查询{Entity}成功")
        void get{Entity}ById_Success() {
            // Given
            Long testId = 1L;
            when(repository.selectById(testId)).thenReturn(testEntity);

            // When
            {Entity}Entity result = domainService.getById(testId);

            // Then
            assertNotNull(result);
            verify(repository).selectById(testId);
        }

        @Test
        @DisplayName("查询不存在的{Entity}返回空")
        void get{Entity}ById_NotFound_ReturnsNull() {
            // Given
            Long testId = 999L;
            when(repository.selectById(testId)).thenReturn(null);

            // When
            {Entity}Entity result = domainService.getById(testId);

            // Then
            assertNull(result);
        }

        @Test
        @DisplayName("根据ID查询{Entity}时ID为null抛出异常")
        void get{Entity}ById_NullId_ThrowsException() {
            // When & Then
            assertThrows(BusinessException.class, () -> {
                domainService.getById(null);
            });

            verify(repository, never()).selectById(any());
        }
    }
}
```

---

## 🎯 测试覆盖率要求

| 层级 | 指令覆盖率 | 分支覆盖率 | 方法覆盖率 |
|------|-----------|-----------|-----------|
| DomainService | ≥80% | ≥65% | ≥85% |
| AppService | ≥65% | ≥55% | ≥75% |
| Controller | ≥50% | ≥40% | ≥60% |

### 验证命令

```bash
# 运行测试
mvn clean test

# 查看覆盖率报告
open target/site/jacoco/index.html
```

---

## 📋 必测场景清单

### 成功场景（Happy Path）

- [ ] 正常参数创建/更新实体
- [ ] 查询存在的实体
- [ ] 分页查询返回正确结果
- [ ] 批量操作成功

### 失败场景（Error Cases）

- [ ] 参数为 null 抛出异常
- [ ] 参数格式错误抛出异常
- [ ] 查询不存在的资源返回 null 或抛出异常
- [ ] 违反业务规则抛出 BusinessException

### 边界场景

- [ ] 空字符串参数
- [ ] 超长字符串参数
- [ ] 最大分页大小
- [ ] 并发操作（如需要）

---

## 🔗 相关文档

- [覆盖率标准](./coverage-standards.md)
- [测试规范](./testing-guidelines.md)
- [代码审查快速查阅](../00-code-review-quick-ref.md)

---

**最后更新**: 2026-03-14
**维护者**: Elena (Junior Dev) + Charlie (Senior Dev)
