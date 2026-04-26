# 部署规范

> 本规范定义部署流程和规范，确保部署的稳定性和可追溯性。

---

## 📋 目录

- [部署环境](#部署环境)
- [部署流程](#部署流程)
- [部署检查](#部署检查)
- [回滚流程](#回滚流程)
- [监控告警](#监控告警)

---

## 部署环境

### 环境划分

| 环境 | 用途 | URL | 说明 |
|------|------|-----|------|
| **dev** | 开发环境 | http://dev.example.com | 开发人员日常使用 |
| **test** | 测试环境 | http://test.example.com | 测试人员使用 |
| **staging** | 预发布环境 | http://staging.example.com | 上线前的验证环境 |
| **prod** | 生产环境 | http://prod.example.com | 正式生产环境 |

### 环境配置

#### application-dev.yml

```yaml
# 开发环境配置
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/dataengine_dev
    username: postgres
    password: postgres

  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update

logging:
  level:
    com.dp.dataengine: DEBUG
```

#### application-test.yml

```yaml
# 测试环境配置
spring:
  datasource:
    url: jdbc:postgresql://test-db.example.com:5432/dataengine_test
    username: test_user
    password: test_password

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    com.dp.dataengine: INFO
```

#### application-staging.yml

```yaml
# 预发布环境配置
spring:
  datasource:
    url: jdbc:postgresql://staging-db.example.com:5432/dataengine_staging
    username: staging_user
    password: staging_password

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    com.dp.dataengine: INFO
```

#### application-prod.yml

```yaml
# 生产环境配置
spring:
  datasource:
    url: jdbc:postgresql://prod-db.example.com:5432/dataengine_prod
    username: prod_user
    password: ${DB_PASSWORD}

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    com.dp.dataengine: WARN
```

---

## 部署流程

### 1. 编译打包

```bash
# 清理并打包
mvn clean package -DskipTests

# 打包并运行测试
mvn clean package

# 打包为特定环境
mvn clean package -DskipTests -Pdev
mvn clean package -DskipTests -Ptest
mvn clean package -DskipTests -Pstaging
mvn clean package -DskipTests -Pprod
```

### 2. Docker镜像构建

```bash
# 构建Docker镜像
docker build -t dataengine:latest .

# 推送到镜像仓库
docker tag dataengine:latest registry.example.com/dataengine:latest
docker push registry.example.com/dataengine:latest

# 推送版本标签
docker tag dataengine:latest registry.example.com/dataengine:v1.0.0
docker push registry.example.com/dataengine:v1.0.0
```

### Dockerfile示例

```dockerfile
FROM openjdk:17-jdk-alpine

WORKDIR /app

# 复制JAR文件
COPY target/dataengine-backend-1.0.0.jar app.jar

# 暴露端口
EXPOSE 8080

# 设置JVM参数
ENV JAVA_OPTS="-Xms512m -Xmx1024m -XX:+UseG1GC"

# 启动应用
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### 3. Kubernetes部署

#### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dataengine
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dataengine
  template:
    metadata:
      labels:
        app: dataengine
    spec:
      containers:
      - name: dataengine
        image: registry.example.com/dataengine:v1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
```

#### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: dataengine
  namespace: production
spec:
  selector:
    app: dataengine
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

### 4. 部署到Kubernetes

```bash
# 部署到命名空间
kubectl apply -f deployment.yaml -n production
kubectl apply -f service.yaml -n production

# 查看部署状态
kubectl get pods -n production
kubectl get services -n production

# 查看日志
kubectl logs -f deployment/dataengine -n production

# 扩缩容
kubectl scale deployment dataengine --replicas=5 -n production
```

### 5. 数据库迁移

```bash
# 执行Flyway迁移
mvn flyway:migrate

# 查看迁移历史
mvn flyway:info

# 验证迁移
mvn flyway:validate
```

---

## 部署检查

### 部署前检查

| 检查项 | 说明 | 通过/不通过 |
|--------|------|------------|
| 代码已合并 | 代码已合并到对应分支 | ☐ |
| 测试通过 | 所有测试通过 | ☐ |
| 覆盖率达标 | 覆盖率达到要求 | ☐ |
| 代码审查通过 | 代码审查通过 | ☐ |
| 文档已更新 | 文档已更新 | ☐ |
| 数据库迁移 | 数据库迁移脚本已准备 | ☐ |
| 配置文件已更新 | 配置文件已更新 | ☐ |

### 部署后验证

| 检查项 | 说明 | 通过/不通过 |
|--------|------|------------|
| 健康检查 | /actuator/health正常 | ☐ |
| 服务可用 | 服务可正常访问 | ☐ |
| 数据库连接 | 数据库连接正常 | ☐ |
| 日志正常 | 无错误日志 | ☐ |
| 性能正常 | 性能指标正常 | ☐ |
| 监控正常 | 监控数据正常 | ☐ |

### 验证脚本

```bash
#!/bin/bash
# verify-deployment.sh

# 健康检查
echo "检查健康状态..."
curl -f http://localhost:8080/actuator/health || exit 1

# 检查服务可用性
echo "检查服务可用性..."
curl -f http://localhost:8080/api/health || exit 1

# 检查数据库连接
echo "检查数据库连接..."
curl -f http://localhost:8080/actuator/health/db || exit 1

echo "部署验证通过！"
```

---

## 回滚流程

### 回滚触发条件

以下情况触发回滚：

1. 健康检查失败
2. 服务不可用
3. 数据库连接失败
4. 出现严重错误日志
5. 性能严重下降
6. 用户反馈严重问题

### 回滚流程

```bash
# 1. 停止部署
kubectl rollout undo deployment/dataengine -n production

# 2. 查看回滚状态
kubectl rollout status deployment/dataengine -n production

# 3. 查看回滚历史
kubectl rollout history deployment/dataengine -n production

# 4. 回滚到指定版本
kubectl rollout undo deployment/dataengine --to-revision=2 -n production

# 5. 验证回滚
./verify-deployment.sh
```

### 回滚后检查

| 检查项 | 说明 | 通过/不通过 |
|--------|------|------------|
| 健康检查 | /actuator/health正常 | ☐ |
| 服务可用 | 服务可正常访问 | ☐ |
| 数据库连接 | 数据库连接正常 | ☐ |
| 日志正常 | 无错误日志 | ☐ |
| 用户反馈 | 用户反馈正常 | ☐ |

---

## 监控告警

### 监控指标

| 指标 | 说明 | 阈值 |
|------|------|------|
| **CPU使用率** | CPU使用率 | >80% |
| **内存使用率** | 内存使用率 | >85% |
| **响应时间** | API响应时间 | >1s |
| **错误率** | API错误率 | >1% |
| **QPS** | 每秒请求数 | - |
| **数据库连接** | 数据库连接数 | >80% |
| **线程数** | 活动线程数 | >200 |

### 告警配置

```yaml
# prometheus-alerts.yml
groups:
- name: dataengine-alerts
  rules:
  - alert: HighCPUUsage
    expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "CPU使用率过高"
      description: "CPU使用率超过80%"

  - alert: HighMemoryUsage
    expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "内存使用率过高"
      description: "内存使用率超过85%"

  - alert: HighResponseTime
    expr: http_request_duration_seconds > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "响应时间过长"
      description: "API响应时间超过1秒"

  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.01
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "错误率过高"
      description: "API错误率超过1%"
```

### 日志收集

```yaml
# logback-spring.xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/dataengine.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/dataengine.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

---

## 最佳实践

### 1. 灰度发布

```bash
# 逐步滚动更新
kubectl rollout status deployment/dataengine -n production --timeout=600s

# 设置滚动更新策略
kubectl set image deployment/dataengine dataengine=registry.example.com/dataengine:v1.0.0 -n production --record
```

### 2. 配置外部化

```bash
# 使用ConfigMap存储配置
kubectl create configmap dataengine-config --from-file=application-prod.yml -n production

# 使用Secret存储敏感信息
kubectl create secret generic db-secret --from-literal=password=your-password -n production
```

### 3. 健康检查

```bash
# 配置健康检查端点
curl http://localhost:8080/actuator/health
curl http://localhost:8080/actuator/health/liveness
curl http://localhost:8080/actuator/health/readiness
```

### 4. 优雅停机

```yaml
# application.yml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

---

## 🔗 相关文档

- [Git工作流规范](./git-workflow.md) - Git工作流详细规范
- [代码审查规范](./code-review.md) - 代码审查详细规范
- [Story完成规范](./story-completion.md) - Story完成详细规范

---

**最后更新**：2026-03-10