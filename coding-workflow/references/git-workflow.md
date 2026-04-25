# Git工作流规范

> 本规范定义Git工作流的规范，确保代码协作的可追溯性和质量。

---

## 📋 目录

- [分支模型](#分支模型)
- [提交规范](#提交规范)
- [分支管理](#分支管理)
- [代码合并](#代码合并)
- [冲突解决](#冲突解决)

---

## 分支模型

### Git-Flow分支模型

```
main (生产环境)
 ↑
 ├─ release/v1.0.0 (发布分支)
 │  ↑
 │  ├─ develop (开发环境)
 │  │  ↑
 │  │  ├─ feature/user-management (功能分支)
 │  │  ├─ feature/login (功能分支)
 │  │  └─ feature/data-quality (功能分支)
 │  │
 │  └─ hotfix/password-reset (热修复分支)
 │
 └─ tags/v1.0.0 (版本标签)
```

### 分支说明

| 分支类型 | 命名格式 | 说明 | 生命周期 |
|---------|---------|------|---------|
| **main** | `main` | 生产环境分支，始终可部署 | 永久 |
| **develop** | `develop` | 开发环境分支，集成最新功能 | 永久 |
| **feature** | `feature/{功能名}` | 功能开发分支 | 临时 |
| **release** | `release/v{版本号}` | 发布准备分支 | 临时 |
| **hotfix** | `hotfix/{问题描述}` | 生产环境热修复分支 | 临时 |

### 分支命名规范

```bash
# 功能分支
feature/user-management          # 用户管理功能
feature/login-page               # 登录页面功能
feature/data-quality-rules      # 数据质量规则功能

# 发布分支
release/v1.0.0                   # v1.0.0版本发布
release/v1.1.0                   # v1.1.0版本发布

# 热修复分支
hotfix/password-reset-bug       # 密码重置bug修复
hotfix/login-failure             # 登录失败修复
```

---

## 提交规范

### Conventional Commits规范

**格式**：`{类型}({范围}): {描述}`

### 提交类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **feat** | 新功能 | `feat(user): 添加用户创建功能` |
| **fix** | Bug修复 | `fix(login): 修复登录失败问题` |
| **docs** | 文档更新 | `docs(readme): 更新README文档` |
| **style** | 代码格式（不影响功能） | `style(java): 统一代码格式` |
| **refactor** | 重构（不是新增功能或修复） | `refactor(service): 重构用户服务` |
| **test** | 测试相关 | `test(user): 添加用户单元测试` |
| **chore** | 构建/工具相关 | `chore(deps): 更新依赖版本` |
| **perf** | 性能优化 | `perf(query): 优化查询性能` |
| **ci** | CI/CD相关 | `ci(github): 更新CI配置` |

### 提交示例

```bash
# 功能
git commit -m "feat(user): 添加用户创建功能"
git commit -m "feat(login): 实现OAuth2登录"

# 修复
git commit -m "fix(login): 修复登录失败问题"
git commit -m "fix(query): 修复SQL注入漏洞"

# 文档
git commit -m "docs(readme): 更新README文档"
git commit -m "docs(api): 添加API文档"

# 重构
git commit -m "refactor(service): 重构用户服务"
git commit -m "refactor(controller): 简化控制器逻辑"

# 测试
git commit -m "test(user): 添加用户单元测试"
git commit -m "test(integration): 添加集成测试"

# 构建
git commit -m "chore(deps): 更新Spring Boot版本"
git commit -m "chore(maven): 更新依赖版本"
```

### 提交最佳实践

```bash
# ✅ 好的提交
git commit -m "feat(user): 添加用户创建功能"
git commit -m "fix(login): 修复登录失败问题 - 用户名验证错误"

# ❌ 不好的提交
git commit -m "添加功能"
git commit -m "fix bug"
git commit -m "update"
git commit -m "修复bug #123 #456 #789"
```

---

## 分支管理

### 创建功能分支

```bash
# 从develop分支创建功能分支
git checkout develop
git pull origin develop
git checkout -b feature/user-management

# 推送到远程
git push -u origin feature/user-management
```

### 开发功能

```bash
# 添加文件
git add src/main/java/com/dp/dataengine/domain/service/user/UserDomainService.java

# 提交代码
git commit -m "feat(user): 添加用户创建功能"

# 推送到远程
git push origin feature/user-management
```

### 创建发布分支

```bash
# 从develop分支创建发布分支
git checkout develop
git pull origin develop
git checkout -b release/v1.0.0

# 推送到远程
git push -u origin release/v1.0.0
```

### 创建热修复分支

```bash
# 从main分支创建热修复分支
git checkout main
git pull origin main
git checkout -b hotfix/password-reset-bug

# 推送到远程
git push -u origin hotfix/password-reset-bug
```

---

## 代码合并

### 合并到develop

```bash
# 切换到develop分支
git checkout develop
git pull origin develop

# 合并功能分支
git merge --no-ff feature/user-management

# 推送到远程
git push origin develop

# 删除本地和远程功能分支
git branch -d feature/user-management
git push origin --delete feature/user-management
```

### 合并到main（发布）

```bash
# 切换到main分支
git checkout main
git pull origin main

# 合并发布分支
git merge --no-ff release/v1.0.0

# 打标签
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# 推送到远程
git push origin main

# 删除发布分支
git branch -d release/v1.0.0
git push origin --delete release/v1.0.0
```

### 合并热修复

```bash
# 1. 合并到main
git checkout main
git pull origin main
git merge --no-ff hotfix/password-reset-bug
git push origin main

# 2. 打标签
git tag -a v1.0.1 -m "Hotfix v1.0.1"
git push origin v1.0.1

# 3. 合并到develop（避免后续重复）
git checkout develop
git pull origin develop
git merge --no-ff hotfix/password-reset-bug
git push origin develop

# 4. 删除热修复分支
git branch -d hotfix/password-reset-bug
git push origin --delete hotfix/password-reset-bug
```

---

## 冲突解决

### 解决合并冲突

```bash
# 1. 执行合并
git merge feature/user-management

# 2. 查看冲突文件
git status

# 3. 编辑冲突文件
# 打开冲突文件，找到冲突标记
# <<<<<<< HEAD
# 你的代码
# =======
# 合并的代码
# >>>>>>> feature/user-management

# 4. 解决冲突
# 保留需要的代码，删除冲突标记

# 5. 标记冲突已解决
git add 冲突文件

# 6. 完成合并
git commit

# 7. 推送代码
git push origin develop
```

### 解决拉取冲突

```bash
# 1. 拉取代码
git pull origin develop

# 2. 如果有冲突，解决冲突

# 3. 添加解决后的文件
git add 冲突文件

# 4. 完成合并
git commit

# 5. 推送代码
git push origin feature/user-management
```

### 解决rebase冲突

```bash
# 1. 执行rebase
git rebase origin/develop

# 2. 如果有冲突，解决冲突

# 3. 继续rebase
git add 冲突文件
git rebase --continue

# 4. 如果需要跳过某个提交
git rebase --skip

# 5. 如果需要中止rebase
git rebase --abort

# 6. 强制推送（谨慎使用）
git push origin feature/user-management --force
```

---

## 最佳实践

### 1. 频繁提交

```bash
# ✅ 好的做法：频繁提交
git add UserDomainService.java
git commit -m "feat(user): 添加用户创建方法"

git add UserDTO.java
git commit -m "feat(user): 添加用户DTO"

# ❌ 不好的做法：一次性提交所有代码
git add .
git commit -m "添加用户管理功能"
```

### 2. 保持分支更新

```bash
# 定期同步develop分支
git checkout feature/user-management
git pull origin develop
```

### 3. 合并前检查

```bash
# 合并前检查状态
git status
git log --oneline origin/develop..HEAD

# 合并前运行测试
mvn test
mvn jacoco:check
```

### 4. 使用--no-ff合并

```bash
# 使用--no-ff保留分支历史
git merge --no-ff feature/user-management
```

### 5. 删除已合并的分支

```bash
# 删除已合并的功能分支
git branch -d feature/user-management
git push origin --delete feature/user-management
```

---

## 🔗 相关文档

- [代码审查规范](./code-review.md) - 代码审查详细规范
- [Story完成规范](./story-completion.md) - Story完成详细规范
- [部署规范](./deployment.md) - 部署详细规范

---

**最后更新**：2026-03-10