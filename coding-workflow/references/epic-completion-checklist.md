# Epic 完成规范 Checklist

**版本**: 1.0
**更新日期**: 2026-03-14
**用途**: Epic 完成前必须逐项检查，确保所有 Stories 都已完成且符合规范

---

## 📋 使用说明

1. **何时使用**: 所有 Stories 完成后，标记 Epic 为 done 前
2. **如何使用**: 逐项检查，勾选通过的项目
3. **必须全部通过**: 任何一项未通过都需要修复后重新检查
4. **更新 sprint-status.yaml**: 所有项通过后更新 Epic 状态

---

## 🚨 CRITICAL 检查项（必须全部通过）

### Stories 完成检查

- [ ] **所有 Stories 状态为 done**
  - 检查 sprint-status.yaml 中所有 Story 的状态
  - 确认没有 backlog/in-progress 的 Story

- [ ] **所有 Stories 已通过验收**
  - 每个 Story 的验收标准都已满足
  - 功能测试通过

### 代码审查检查

- [ ] **代码审查已完成**
  - 使用 code-reviewer agent 进行审查
  - 或者由团队成员进行人工审查

- [ ] **代码审查结果已记录到 sprint-status.yaml**
  ```yaml
  code_review:
    epic-X:
      status: done
      reviewed_by: Claude Code Reviewer
      reviewed_at: 2026-03-14
      critical_issues: X
      high_issues: X
      summary: "Epic X 完成，修复了所有问题"
  ```

- [ ] **所有 CRITICAL 问题已修复**
  - CRITICAL 问题数量必须为 0（修复后）
  - 或者有明确的修复计划

- [ ] **所有 HIGH 问题已修复或评估**
  - HIGH 问题建议全部修复
  - 未修复的需要评估风险并记录原因

### 文档更新检查

- [ ] **Epic 回顾文档已生成**
  - 文件位置: `_bmad-output/implementation-artifacts/epic-X-retro-{date}.md`
  - 包含：完成情况、问题分析、行动项

- [ ] **sprint-status.yaml 已更新**
  - epic-X: done
  - epic-X-retrospective: done（如已完成回顾）

- [ ] **代码审查记录已添加**
  - code_review.epic-X 条目已添加

---

## 🟠 HIGH 检查项（必须全部通过）

### 测试检查

- [ ] **所有 DomainService 测试覆盖率 ≥ 80%**
  - 使用 JaCoCo 验证
  - 测试报告已生成

- [ ] **关键功能有集成测试**
  - Controller 层集成测试
  - 端到端测试（如需要）

### 数据库检查

- [ ] **数据库迁移脚本已提交**
  - 位置: `backend/dataengine-api/src/main/resources/db/migration/`
  - 文件命名规范: `V{version}__{description}.sql`

- [ ] **迁移脚本符合 PostgreSQL 语法**
  - 使用 BIGSERIAL 而非 AUTO_INCREMENT
  - COMMENT ON 语法正确

### 技术债检查

- [ ] **已知技术债已记录**
  - 记录到 Epic 回顾文档
  - 或记录到专门的 tech-debt 文档

- [ ] **技术债有修复计划**
  - 明确优先级
  - 明确责任人

---

## 🟡 MEDIUM 检查项（建议全部通过）

### 代码质量

- [ ] **无 TODO 注释遗留**
  - 或 TODO 有对应 Issue 跟踪

- [ ] **无调试代码**
  - 无 `System.out.println`
  - 无 `debugger` 语句

- [ ] **代码格式统一**
  - 使用项目统一的代码格式化配置

### API 文档

- [ ] **新增 API 已记录**
  - Swagger/OpenAPI 注解完整
  - 或有独立的 API 文档

### 性能

- [ ] **性能问题已评估**
  - 大数据量查询已优化
  - 缓存策略已考虑

---

## 📊 Epic 完成统计

### Stories 统计

- Epic 编号: ________
- Stories 总数: ________
- 完成数量: ________
- 完成率: ________%

### 代码审查统计

- CRITICAL 问题: ________（修复后: ________）
- HIGH 问题: ________（修复后: ________）
- MEDIUM 问题: ________
- LOW 问题: ________
- 总计: ________

### 测试覆盖率

- DomainService 覆盖率: ________%
- AppService 覆盖率: ________%
- Controller 覆盖率: ________%

---

## ✅ 签名

**开发者**: _________________  **日期**: ________

**Scrum Master**: _________________  **日期**: ________

**产品负责人**: _________________  **日期**: ________

---

## 📝 Epic 完成后更新 sprint-status.yaml

示例：

```yaml
development_status:
  epic-X: done
  completion_date: 2026-03-14
  X-1-story-name: done
  X-2-story-name: done
  # ... 所有 stories
  epic-X-retrospective: done  # 如已完成回顾

code_review:
  epic-X:
    status: done
    reviewed_by: Claude Code Reviewer
    reviewed_at: 2026-03-14
    critical_issues: 0
    high_issues: 2
    summary: "Epic X 完成，修复了所有 CRITICAL 和 HIGH 问题"
```

---

**注意**: 本 checklist 将持续更新，请使用最新版本。

**最后更新**: 2026-03-14
**维护者**: Bob (Scrum Master)
