---
title: AI应用生产部署：从容器到云原生
date: 2026-04-24
tags:
  - Docker
  - Kubernetes
  - 高可用
  - 监控
  - CI/CD
  - 生产环境
  - 安全
  - 灾备
  - 云原生
categories:
  - 智能体搭建
  - 企业级应用
alias: Production AI Deployment
description: 深入讲解AI应用的生产级部署实践，包括Docker容器化、Kubernetes编排、高可用架构、监控系统、CI/CD流水线、安全配置、灾备恢复等企业级必备知识，通过完整的配置示例和最佳实践帮助构建可靠的AI基础设施。
---

# AI应用生产部署：从容器到云原生

> [!NOTE] 这篇指南讲什么
> 把AI应用跑起来是一回事，跑好是另一回事。这篇指南从生产级部署的各个方面讲起，包括容器化、K8s部署、高可用、监控告警、CI/CD、安全加固、灾备恢复等，让你真正掌握企业级AI应用的部署运维。

## 核心关键词速览

| 关键词 | 说明 | 关键词 | 说明 |
|--------|------|--------|------|
| Docker | 容器化部署 | Kubernetes | 容器编排 |
| 高可用 | HA Architecture | 监控 | Prometheus/Grafana |
| CI/CD | 持续集成部署 | 日志收集 | ELK Stack |
| 负载均衡 | Load Balancing | 自动扩缩容 | HPA/VPA |
| 健康检查 | Health Check | 滚动更新 | Rolling Update |
| 安全加固 | Security Hardening | 灾备恢复 | DR/BCP |

## 1. 为什么要生产级部署？

### 1.1 开发 vs 生产

很多人开发时是这样的：

```bash
# 开发模式 - 随便跑跑
python app.py
# 或者
uvicorn app:app --reload
```

生产环境的问题：
- 进程挂了怎么办？
- 请求太多扛不住怎么办？
- 代码更新怎么不停服？
- 日志怎么收集分析？
- 怎么知道系统健康不健康？

### 1.2 生产级部署架构

```
┌─────────────────────────────────────────────────────────────┐
│                      生产级部署架构                              │
│                                                            │
│   ┌─────────────────────────────────────────────────────┐  │
│   │                    用户请求                           │  │
│   └─────────────────────────────────────────────────────┘  │
│                           ↓                                 │
│   ┌─────────────────────────────────────────────────────┐  │
│   │              负载均衡 / API网关                       │  │
│   │         (限流、认证、日志、SSL终结)                     │  │
│   └─────────────────────────────────────────────────────┘  │
│                           ↓                                 │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│   │  Pod 1  │  │  Pod 2  │  │  Pod 3  │  │  Pod N  │  │
│   │  (API)  │  │  (API)  │  │  (API)  │  │  (API)  │  │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│         ↓              ↓              ↓                     │
│   ┌─────────────────────────────────────────────────────┐  │
│   │                   数据层                              │  │
│   │   PostgreSQL (主从)  │  Redis (集群)  │  向量库     │  │
│   └─────────────────────────────────────────────────────┘  │
│                                                            │
│   ┌─────────────────────────────────────────────────────┐  │
│   │                   监控层                              │  │
│   │       Prometheus  │  Grafana  │  日志收集            │  │
│   └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 2. Docker容器化最佳实践

### 2.1 多阶段构建Dockerfile

```dockerfile
# ============================================================
# 第一阶段：构建
# ============================================================
FROM python:3.11-slim AS builder

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安装Python依赖
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/opt/venv -r requirements.txt

# ============================================================
# 第二阶段：运行
# ============================================================
FROM python:3.11-slim

# 安全：创建非root用户
RUN groupadd --gid 1000 appgroup \
    && useradd --uid 1000 --gid appgroup --shell /bin/bash --create-home appuser

WORKDIR /home/appuser

# 从构建阶段复制虚拟环境
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安装运行时依赖（不含build-tools）
RUN apt-get update && apt-get install -y \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

# 复制应用代码（先创建目录结构）
RUN mkdir -p /home/appuser/{app,logs}
COPY --chown=appuser:appuser ./app /home/appuser/app

# 设置环境变量
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/home/appuser \
    APP_ENV=production

# 切换到非root用户
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import httpx; httpx.get('http://localhost:8000/health', timeout=5)" || exit 1

# 暴露端口
EXPOSE 8000

# 使用gunicorn运行
CMD ["gunicorn", "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--threads", "2", \
     "--worker-class", "uvicorn.workers.UvicornWorker", \
     "--worker-tmp-dir", "/dev/shm", \
     "--access-logfile", "-", \
     "--error-logfile", "-", \
     "--capture-output", \
     "--enable-stdio-inheritance", \
     "app.main:app"]
```

### 2.2 .dockerignore

```gitignore
# Git
.git
.gitignore

# Python
__pycache__
*.py[cod]
*$py.class
*.so
.Python
*.egg
*.egg-info
.eggs
dist
build

# 虚拟环境
venv
.venv
env

# IDE
.idea
.vscode
*.swp
*.swo

# 测试
.coverage
htmlcov
.pytest_cache
tests

# 文档
docs
*.md
!requirements.txt

# 本地配置
.env.local
.env.*.local

# 日志
*.log
logs/

# 临时文件
tmp
temp
*.tmp

# Docker相关（不要复制Dockerfile本身）
docker-compose*.yml
Dockerfile
.dockerignore
```

### 2.3 requirements.txt

```txt
# ===========================================
# 核心框架
# ===========================================
fastapi==0.109.2
uvicorn[standard]==0.27.1
gunicorn==21.2.0
httpx==0.26.0

# ===========================================
# 数据库
# ===========================================
asyncpg==0.29.0
sqlalchemy[asyncio]==2.0.25
redis==5.0.1
alembic==1.13.1

# ===========================================
# AI/ML
# ===========================================
openai==1.12.0
anthropic==0.18.0
tiktoken==0.5.2

# ===========================================
# 认证与安全
# ===========================================
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9

# ===========================================
# 监控与可观测性
# ===========================================
prometheus-client==0.19.0
opentelemetry-api==1.22.0
opentelemetry-sdk==1.22.0
opentelemetry-instrumentation-fastapi==0.43b0
opentelemetry-exporter-prometheus==0.43b0

# ===========================================
# 工具库
# ===========================================
pydantic==2.6.1
pydantic-settings==2.1.0
python-dotenv==1.0.1
structlog==24.1.0
```

### 2.4 健康检查脚本

```python
# healthcheck.py
#!/usr/bin/env python3
"""
健康检查脚本 - 用于Docker健康检查和K8s探针
"""
import sys
import httpx


def check_health():
    """执行健康检查"""
    try:
        # 检查主服务
        response = httpx.get(
            "http://localhost:8000/health",
            timeout=3
        )
        
        if response.status_code != 200:
            print(f"Health check failed: HTTP {response.status_code}")
            return False
        
        data = response.json()
        
        # 检查依赖服务
        checks = data.get("checks", {})
        
        # 检查数据库
        if checks.get("database") != "ok":
            print("Database check failed")
            return False
        
        # 检查Redis
        if checks.get("redis") != "ok":
            print("Redis check failed")
            return False
        
        print("All health checks passed")
        return True
        
    except httpx.ConnectError:
        print("Cannot connect to service")
        return False
    except httpx.TimeoutException:
        print("Health check timeout")
        return False
    except Exception as e:
        print(f"Health check error: {e}")
        return False


if __name__ == "__main__":
    success = check_health()
    sys.exit(0 if success else 1)
```

## 3. Docker Compose配置

### 3.1 开发环境

```yaml
# docker-compose.yml
version: '3.9'

services:
  # ============================================
  # API服务
  # ============================================
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    container_name: ai-agent-api-dev
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - .:/home/appuser
      - ./logs:/home/appuser/logs
    environment:
      - APP_ENV=development
      - DATABASE_URL=postgresql+asyncpg://postgres:postgres@postgres:5432/ai_agent_dev
      - REDIS_URL=redis://redis:6379/0
      - LOG_LEVEL=DEBUG
    env_file:
      - .env.local
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - ai-network
    profiles:
      - dev

  # ============================================
  # PostgreSQL数据库
  # ============================================
  postgres:
    image: postgres:15-alpine
    container_name: ai-agent-postgres-dev
    restart: unless-stopped
    environment:
      POSTGRES_DB: ai_agent_dev
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres-dev-data:/var/lib/postgresql/data
      - ./docker/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d ai_agent_dev"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    networks:
      - ai-network
    profiles:
      - dev

  # ============================================
  # Redis缓存
  # ============================================
  redis:
    image: redis:7-alpine
    container_name: ai-agent-redis-dev
    restart: unless-stopped
    command: >
      redis-server
      --appendonly yes
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
    volumes:
      - redis-dev-data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - ai-network
    profiles:
      - dev

  # ============================================
  # 向量数据库 (Milvus)
  # ============================================
  milvus:
    image: milvusdb/milvus:v2.3.3
    container_name: ai-agent-milvus-dev
    restart: unless-stopped
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - milvus-dev-data:/var/lib/milvus
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - etcd
      - minio
    networks:
      - ai-network
    profiles:
      - dev

  etcd:
    image: quay.io/coreos/etcd:v3.5.5
    container_name: ai-agent-etcd-dev
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000
    volumes:
      - etcd-dev-data:/etcd
    networks:
      - ai-network
    profiles:
      - dev

  minio:
    image: minio/minio:latest
    container_name: ai-agent-minio-dev
    restart: unless-stopped
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - minio-dev-data:/minio_data
    ports:
      - "9000:9000"
      - "9001:9001"
    command: server /minio_data --console-address ":9001"
    networks:
      - ai-network
    profiles:
      - dev

networks:
  ai-network:
    driver: bridge

volumes:
  postgres-dev-data:
  redis-dev-data:
  milvus-dev-data:
  etcd-dev-data:
  minio-dev-data:
```

### 3.2 生产环境

```yaml
# docker-compose.prod.yml
version: '3.9'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    image: registry.example.com/ai-agent:${IMAGE_TAG:-latest}
    container_name: ai-agent-api
    restart: always
    expose:
      - "8000"
    environment:
      - APP_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    env_file:
      - .env.production
    volumes:
      - api-logs:/home/appuser/logs
    healthcheck:
      test: ["CMD", "python", "healthcheck.py"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '0.5'
          memory: 1G
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
    networks:
      - ai-network

  # 使用外部托管数据库时的配置
  # postgres:
  #   external: true
  #   name: ${EXTERNAL_POSTGRES}
  #
  # redis:
  #   external: true
  #   name: ${EXTERNAL_REDIS}

networks:
  ai-network:
    driver: overlay
    attachable: true

volumes:
  api-logs:
```

## 4. Kubernetes深度配置

### 4.1 完整K8s配置

```yaml
# k8s/00-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ai-agent
  labels:
    name: ai-agent
    env: production
    app.kubernetes.io/managed-by: kubectl
---
# k8s/01-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ai-agent-config
  namespace: ai-agent
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  LOG_FORMAT: "json"
  MAX_WORKERS: "4"
  REDIS_HOST: "redis-master"
  REDIS_PORT: "6379"
  WORKER_CLASS: "uvicorn.workers.UvicornWorker"
  KEEPALIVE: "65"
  TIMEOUT: "120"
---
# k8s/02-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ai-agent-secrets
  namespace: ai-agent
type: Opaque
stringData:
  DATABASE_URL: "postgresql+asyncpg://user:password@postgres:5432/ai_agent"
  REDIS_URL: "redis://redis:6379/0"
  OPENAI_API_KEY: "sk-..."
  SECRET_KEY: "your-super-secret-key-change-this"
---
# k8s/03-pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: ai-agent-api-pdb
  namespace: ai-agent
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: ai-agent-api
```

```yaml
# k8s/04-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-agent-api
  namespace: ai-agent
  labels:
    app: ai-agent-api
    version: v1.0.0
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: ai-agent-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: ai-agent-api
        version: v1.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8000"
        prometheus.io/path: "/metrics"
    spec:
      # 服务账号
      serviceAccountName: ai-agent-api
      
      # 终止gracePeriod
      terminationGracePeriodSeconds: 60
      
      # 安全上下文
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      
      # 初始化容器 - 等待依赖服务
      initContainers:
        - name: wait-for-dependencies
          image: busybox:1.36
          command:
            - sh
            - -c
            - |
              echo "Waiting for database..."
              nc -z postgres 5432 || exit 1
              echo "Database is ready"
              echo "Waiting for Redis..."
              nc -z redis 6379 || exit 1
              echo "Redis is ready"
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
      
      # 主容器
      containers:
        - name: api
          image: registry.example.com/ai-agent:v1.0.0
          imagePullPolicy: Always
          
          ports:
            - name: http
              containerPort: 8000
              protocol: TCP
          
          # 环境变量
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: ai-agent-secrets
                  key: DATABASE_URL
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: ai-agent-secrets
                  key: REDIS_URL
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: ai-agent-secrets
                  key: OPENAI_API_KEY
          
          envFrom:
            - configMapRef:
                name: ai-agent-config
          
          # 资源限制
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 2000m
              memory: 4Gi
          
          # 存活探针
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
            successThreshold: 1
          
          # 就绪探针
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
            successThreshold: 1
          
          # 启动探针（冷启动优化）
          startupProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 30
          
          # 生命周期钩子
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]
          
          # 安全上下文
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          
          # 挂载
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: logs
              mountPath: /home/appuser/logs
      
      # 亲和性调度
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - ai-agent-api
                topologyKey: kubernetes.io/hostname
      
      # 容忍
      tolerations:
        - key: "node-type"
          operator: "Equal"
          value: "ai-agent"
          effect: "NoSchedule"
      
      volumes:
        - name: tmp
          emptyDir:
            medium: Memory
            sizeLimit: 100Mi
        - name: logs
          emptyDir:
            medium: Memory
            sizeLimit: 500Mi
```

```yaml
# k8s/05-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ai-agent-api
  namespace: ai-agent
  labels:
    app: ai-agent-api
spec:
  type: ClusterIP
  selector:
    app: ai-agent-api
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
---
# k8s/06-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ai-agent-api-hpa
  namespace: ai-agent
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ai-agent-api
  minReplicas: 3
  maxReplicas: 20
  
  metrics:
    # CPU指标
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    
    # 内存指标
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    
    # 自定义指标 (Prometheus)
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "100"
  
  # 扩缩容行为
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
        - type: Pods
          value: 4
          periodSeconds: 60
      selectPolicy: Max
    
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
      selectPolicy: Min
```

### 4.2 Ingress配置

```yaml
# k8s/07-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ai-agent-ingress
  namespace: ai-agent
  annotations:
    # SSL配置
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    
    # Nginx Ingress配置
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    
    # 代理配置
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "10"
    
    # 限流
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-connections: "50"
    nginx.ingress.kubernetes.io/limit-rpm: "1000"
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://app.example.com"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "Content-Type, Authorization, X-Request-ID"
    
    # WebSocket支持
    nginx.ingress.kubernetes.io/use-regex: "true"
    
    # 日志
    nginx.ingress.kubernetes.io/log-format-upstream: '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $request_length $request_time [$proxy_upstream_name] [$upstream_addr] [$upstream_response_length] [$upstream_response_time] [$upstream_status] $req_id'
    
spec:
  ingressClassName: nginx
  
  tls:
    - hosts:
        - api.example.com
        - "*.api.example.com"
      secretName: api-tls-secret
  
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: ai-agent-api
                port:
                  number: 80
            annotations:
              nginx.ingress.kubernetes.io/rewrite-target: /
          
          - path: /ws
            pathType: Prefix
            backend:
              service:
                name: ai-agent-api
                port:
                  number: 80
            annotations:
              nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
              nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
              nginx.ingress.kubernetes.io/upstream-hash-by: "$remote_addr"
```

## 5. 高可用架构

### 5.1 多区域部署

```yaml
# k8s/multi-region/deployment-us-east.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-agent-api-us-east
  namespace: ai-agent
  labels:
    app: ai-agent-api
    region: us-east-1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-agent-api
      region: us-east-1
  template:
    metadata:
      labels:
        app: ai-agent-api
        region: us-east-1
    spec:
      nodeSelector:
        topology.kubernetes.io/region: us-east-1
      tolerations:
        - key: "topology.kubernetes.io/region"
          operator: "Equal"
          value: "us-east-1"
          effect: "NoSchedule"
      containers:
        - name: api
          image: registry.example.com/ai-agent:v1.0.0
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 2000m
              memory: 4Gi
---
# k8s/multi-region/service-mesh.yaml
apiVersion: v1
kind: Service
metadata:
  name: ai-agent-api-global
  namespace: ai-agent
spec:
  type: ClusterIP
  # 使用外部负载均衡器
  externalTrafficPolicy: Local
  sessionAffinity: ClientIP
```

### 5.2 数据库高可用

```yaml
# k8s/postgres-ha.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: ai-agent-db
  namespace: ai-agent
spec:
  instances: 3
  imageName: ghcr.io/cloudnative-pg/postgresql:15.2
  
  # 副本配置
  replicationSlots:
    highAvailability:
      enabled: true
  
  # 存储配置
  storage:
    storageClass: ssd-premium
    size: 100Gi
    resizeInUseVolumes: true
  
  # 资源限制
  resources:
    limits:
      cpu: 2
      memory: 4Gi
  
  # 备份配置
  backup:
    retentionPolicy: "30d"
    volumeSnapshot:
      className: csi-snapclass
      inventoryPolicy: Mine
  
  # WAL归档
  wal:
    compression: zstd
    storage:
      storageClass: ssd-premium
      size: 10Gi
  
  # 连接池
  bootstrap:
    initdb:
      database: ai_agent
      owner: ai_agent_user
  
  # 监控
  monitoring:
    enablePodMonitoring: true
  
  # 滚动更新策略
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

### 5.3 Redis集群

```yaml
# k8s/redis-cluster.yaml
apiVersion: redis.redis.opstreelabs.in/v1beta1
kind: RedisCluster
metadata:
  name: ai-agent-redis
  namespace: ai-agent
spec:
  clusterSize: 3
  
  kubernetesConfig:
    image: quay.io/opstree/redis:v7.2.0
    resources:
      requests:
        cpu: 500m
        memory: 1Gi
      limits:
        cpu: 1000m
        memory: 2Gi
  
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: ssd-premium
        resources:
          requests:
            storage: 10Gi
  
  redisExporter:
    enabled: true
    image: quay.io/opstree/redis-exporter:v1.44.0
  
  # 高可用配置
  redisLeader:
    replicas: 1
    affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                role: leader
            topologyKey: kubernetes.io/hostname
  
  redisFollower:
    replicas: 2
    affinity:
      podAntiAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  role: follower
              topologyKey: kubernetes.io/hostname
```

## 6. 监控系统配置

### 6.1 Prometheus配置

```yaml
# monitoring/00-prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
      external_labels:
        cluster: 'ai-agent-prod'
        env: 'production'
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets: ['alertmanager.monitoring.svc:9093']
    
    rule_files:
      - '/etc/prometheus/rules/*.yml'
    
    scrape_configs:
      # Prometheus自身
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      
      # API服务
      - job_name: 'ai-agent-api'
        kubernetes_sd_configs:
          - role: pod
            namespaces:
              names:
                - ai-agent
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_label_app]
            action: keep
            regex: ai-agent-api
          - source_labels: [__meta_kubernetes_pod_container_port_number]
            action: keep
            regex: "8000"
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
          - source_labels: [__meta_kubernetes_pod_name]
            target_label: pod
      
      # Redis
      - job_name: 'redis'
        kubernetes_sd_configs:
          - role: service
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_label_app]
            action: keep
            regex: redis
      
      # PostgreSQL
      - job_name: 'postgres'
        kubernetes_sd_configs:
          - role: service
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_label_app]
            action: keep
            regex: postgres
```

```yaml
# monitoring/01-alert-rules.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-alert-rules
  namespace: monitoring
data:
  alert-rules.yml: |
    groups:
      - name: ai-agent.rules
        rules:
          # API服务告警
          - alert: APIHighErrorRate
            expr: |
              sum(rate(http_requests_total{status=~"5.."}[5m])) 
              / sum(rate(http_requests_total[5m])) > 0.05
            for: 5m
            labels:
              severity: critical
            annotations:
              summary: "API错误率超过5%"
              description: "API 5xx错误率: {{ $value | humanizePercentage }}"
          
          - alert: APIHighLatency
            expr: |
              histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 2
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "API延迟过高"
              description: "P95延迟: {{ $value }}s"
          
          - alert: APIHighMemoryUsage
            expr: |
              (container_memory_usage_bytes{pod=~"ai-agent-api-.*"} / container_spec_memory_limit_bytes) > 0.85
            for: 10m
            labels:
              severity: warning
            annotations:
              summary: "内存使用率过高"
              description: "Pod {{ $labels.pod }} 内存使用: {{ $value | humanizePercentage }}"
          
          - alert: APIHighCPUUsage
            expr: |
              (rate(container_cpu_usage_seconds_total{pod=~"ai-agent-api-.*"}[5m])) > 1.8
            for: 10m
            labels:
              severity: warning
            annotations:
              summary: "CPU使用率过高"
              description: "Pod {{ $labels.pod }} CPU使用: {{ $value | humanizePercentage }}"
          
          # 数据库告警
          - alert: PostgreSQLDown
            expr: |
              pg_up == 0
            for: 1m
            labels:
              severity: critical
            annotations:
              summary: "PostgreSQL不可用"
              description: "PostgreSQL实例不可用"
          
          - alert: PostgreSQLHighConnections
            expr: |
              sum(pg_stat_database_numbackends) > 80
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "数据库连接数过高"
              description: "当前连接数: {{ $value }}"
          
          # Redis告警
          - alert: RedisDown
            expr: |
              redis_up == 0
            for: 1m
            labels:
              severity: critical
            annotations:
              summary: "Redis不可用"
              description: "Redis实例不可用"
          
          - alert: RedisHighMemoryUsage
            expr: |
              (redis_memory_used_bytes / redis_memory_max_bytes) > 0.85
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "Redis内存使用率过高"
              description: "Redis内存使用: {{ $value | humanizePercentage }}"
          
          # 业务告警
          - alert: HighLLMCallFailureRate
            expr: |
              sum(rate(llm_calls_total{status="error"}[5m])) 
              / sum(rate(llm_calls_total[5m])) > 0.1
            for: 5m
            labels:
              severity: critical
            annotations:
              summary: "LLM调用失败率过高"
              description: "LLM调用失败率: {{ $value | humanizePercentage }}"
          
          - alert: HighTokenUsage
            expr: |
              sum(rate(token_usage_total[1h])) > 10000000
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "Token使用量异常"
              description: "过去1小时Token使用量: {{ $value }}"
```

### 6.2 Grafana仪表盘

```json
{
  "dashboard": {
    "title": "AI Agent 监控面板",
    "uid": "ai-agent-overview",
    "timezone": "browser",
    "refresh": "30s",
    "panels": [
      {
        "title": "请求量 (QPS)",
        "type": "timeseries",
        "gridPos": {"x": 0, "y": 0, "w": 12, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (endpoint)",
            "legendFormat": "{{endpoint}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "reqps",
            "custom": {
              "drawStyle": "line",
              "lineWidth": 2,
              "fillOpacity": 10
            }
          }
        }
      },
      {
        "title": "错误率",
        "type": "timeseries",
        "gridPos": {"x": 12, "y": 0, "w": 12, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))",
            "legendFormat": "5xx Error Rate"
          },
          {
            "expr": "sum(rate(http_requests_total{status=~\"4..\"}[5m])) / sum(rate(http_requests_total[5m]))",
            "legendFormat": "4xx Error Rate"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit",
            "max": 1
          }
        }
      },
      {
        "title": "延迟分布 (P50/P95/P99)",
        "type": "timeseries",
        "gridPos": {"x": 0, "y": 8, "w": 12, "h": 8},
        "targets": [
          {
            "expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
            "legendFormat": "P50"
          },
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
            "legendFormat": "P95"
          },
          {
            "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
            "legendFormat": "P99"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s",
            "custom": {
              "drawStyle": "line"
            }
          }
        }
      },
      {
        "title": "活跃会话数",
        "type": "gauge",
        "gridPos": {"x": 12, "y": 8, "w": 6, "h": 8},
        "targets": [
          {
            "expr": "ai_agent_active_sessions"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "min": 0,
            "max": 1000,
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 500, "color": "yellow"},
                {"value": 800, "color": "red"}
              ]
            }
          }
        }
      },
      {
        "title": "Token消耗",
        "type": "timeseries",
        "gridPos": {"x": 18, "y": 8, "w": 6, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(token_usage_total[1h])) by (model)",
            "legendFormat": "{{model}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "short"
          }
        }
      },
      {
        "title": "Pod状态",
        "type": "table",
        "gridPos": {"x": 0, "y": 16, "w": 24, "h": 8},
        "targets": [
          {
            "expr": "kube_pod_status_phase{namespace=\"ai-agent\", pod=~\"ai-agent-api-.*\"}",
            "format": "table"
          }
        ]
      }
    ]
  }
}
```

### 6.3 自定义指标

```python
# app/monitoring.py
from prometheus_client import Counter, Histogram, Gauge, Info
from typing import Callable
from fastapi import Request, Response
import time

# ============================================
# 业务指标
# ============================================
INFO = Info(
    'ai_agent',
    'AI Agent application information'
).info({'version': '1.0.0', 'environment': 'production'})

# 请求指标
REQUEST_COUNT = Counter(
    'ai_agent_requests_total',
    'Total number of requests',
    ['method', 'endpoint', 'status_code', 'app_version']
)

REQUEST_LATENCY = Histogram(
    'ai_agent_request_duration_seconds',
    'Request latency in seconds',
    ['method', 'endpoint'],
    buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0]
)

REQUEST_SIZE = Histogram(
    'ai_agent_request_size_bytes',
    'Request size in bytes',
    ['endpoint'],
    buckets=[100, 1000, 10000, 100000, 1000000]
)

RESPONSE_SIZE = Histogram(
    'ai_agent_response_size_bytes',
    'Response size in bytes',
    ['endpoint'],
    buckets=[100, 1000, 10000, 100000, 1000000]
)

# 业务指标
ACTIVE_SESSIONS = Gauge(
    'ai_agent_active_sessions',
    'Number of active chat sessions',
    ['agent_id']
)

TOKEN_USAGE = Counter(
    'ai_agent_token_usage_total',
    'Total tokens consumed',
    ['model', 'agent_id', 'token_type']
)

LLM_CALLS = Counter(
    'ai_agent_llm_calls_total',
    'Total LLM API calls',
    ['model', 'status', 'error_type']
)

LLM_LATENCY = Histogram(
    'ai_agent_llm_latency_seconds',
    'LLM API call latency',
    ['model'],
    buckets=[0.5, 1.0, 2.0, 5.0, 10.0, 30.0, 60.0]
)

# 缓存指标
CACHE_HITS = Counter(
    'ai_agent_cache_hits_total',
    'Total cache hits',
    ['cache_type']
)

CACHE_MISSES = Counter(
    'ai_agent_cache_misses_total',
    'Total cache misses',
    ['cache_type']
)

CACHE_HIT_RATIO = Gauge(
    'ai_agent_cache_hit_ratio',
    'Cache hit ratio',
    ['cache_type']
)

# 数据库指标
DB_QUERY_COUNT = Counter(
    'ai_agent_db_queries_total',
    'Total database queries',
    ['operation', 'table']
)

DB_QUERY_LATENCY = Histogram(
    'ai_agent_db_query_duration_seconds',
    'Database query latency',
    ['operation', 'table'],
    buckets=[0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0]
)


# ============================================
# 中间件
# ============================================
async def metrics_middleware(request: Request, call_next: Callable) -> Response:
    """请求监控中间件"""
    # 跳过metrics端点本身
    if request.url.path == "/metrics":
        return await call_next(request)
    
    # 记录开始时间
    start_time = time.time()
    
    # 获取endpoint标识
    endpoint = request.url.path
    method = request.method
    
    # 记录请求大小
    content_length = request.headers.get("content-length", 0)
    if content_length:
        REQUEST_SIZE.labels(endpoint=endpoint).observe(int(content_length))
    
    # 执行请求
    response = await call_next(request)
    
    # 计算延迟
    duration = time.time() - start_time
    
    # 记录指标
    REQUEST_COUNT.labels(
        method=method,
        endpoint=endpoint,
        status_code=response.status_code,
        app_version="1.0.0"
    ).inc()
    
    REQUEST_LATENCY.labels(
        method=method,
        endpoint=endpoint
    ).observe(duration)
    
    # 记录响应大小
    response_size = response.headers.get("content-length", 0)
    if response_size:
        RESPONSE_SIZE.labels(endpoint=endpoint).observe(int(response_size))
    
    return response
```

## 7. CI/CD流水线

### 7.1 GitHub Actions完整配置

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  HELM_VERSION: "3.13.0"
  KUBECTL_VERSION: "1.28.0"

jobs:
  # ============================================
  # 代码检查
  # ============================================
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: pip install ruff black
      
      - name: Run ruff
        run: ruff check app/ tests/
      
      - name: Run black
        run: black --check app/ tests/

  # ============================================
  # 测试
  # ============================================
  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: ai_agent_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-asyncio httpx
      
      - name: Run tests
        env:
          DATABASE_URL: postgresql+asyncpg://test:test@localhost:5432/ai_agent_test
          REDIS_URL: redis://localhost:6379/0
        run: |
          pytest tests/ -v --cov=app --cov-report=xml --cov-report=html
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml
          fail_ci_if_error: false
      
      - name: Security scan
        run: |
          pip install bandit safety
          bandit -r app/
          safety check

  # ============================================
  # 构建镜像
  # ============================================
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha,prefix=
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_SHA=${{ github.sha }}
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
          labels: |
            org.opencontainers.image.source=${{ github.repositoryUrl }}
            org.opencontainers.image.revision=${{ github.sha }}

  # ============================================
  # 部署到Staging
  # ============================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' || github.event_name == 'pull_request'
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}
      
      - name: Set up kubectl
        uses: azure/setup-kubectl@v4
        with:
          version: ${{ env.KUBECTL_VERSION }}
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.STAGING_KUBECONFIG }}" | base64 -d > kubeconfig
          echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV
      
      - name: Deploy to Staging
        run: |
          helm upgrade --install ai-agent ./charts/ai-agent \
            --namespace ai-agent-staging \
            --create-namespace \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ needs.build.outputs.image-tag }} \
            --wait --timeout 10m \
            --atomic \
            --cleanup-on-fail
      
      - name: Run smoke tests
        run: |
          kubectl wait --for=condition=available \
            deployment/ai-agent-api-staging \
            -n ai-agent-staging \
            --timeout=300s
          
          kubectl exec -n ai-agent-staging \
            deploy/ai-agent-api-staging \
            -- python healthcheck.py

  # ============================================
  # 部署到Production
  # ============================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}
      
      - name: Set up kubectl
        uses: azure/setup-kubectl@v4
        with:
          version: ${{ env.KUBECTL_VERSION }}
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.PRODUCTION_KUBECONFIG }}" | base64 -d > kubeconfig
          echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV
      
      - name: Backup database
        run: |
          kubectl exec -n ai-agent \
            deploy/ai-agent-db-0 \
            -- pg_dump -U postgres ai_agent > backup_$(date +%Y%m%d_%H%M%S).sql
      
      - name: Deploy to Production
        run: |
          helm upgrade --install ai-agent ./charts/ai-agent \
            --namespace ai-agent \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ needs.build.outputs.image-tag }} \
            --wait --timeout 15m \
            --atomic \
            --cleanup-on-fail \
            --dry-run=client
      
          helm upgrade --install ai-agent ./charts/ai-agent \
            --namespace ai-agent \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ needs.build.outputs.image-tag }} \
            --wait --timeout 15m \
            --atomic \
            --cleanup-on-fail
      
      - name: Verify deployment
        run: |
          kubectl rollout status deployment/ai-agent-api -n ai-agent --timeout=600s
          
          kubectl exec -n ai-agent \
            deploy/ai-agent-api \
            -- python healthcheck.py
      
      - name: Notify success
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: ${{ secrets.SLACK_CHANNEL }}
          payload: |
            {
              "text": "🚀 AI Agent v${{ github.ref_name }} deployed to production",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Deployment Successful*\n• Version: ${{ github.ref_name }}\n• Commit: ${{ github.sha }}"
                  }
                }
              ]
            }
```

## 8. 安全加固

### 8.1 Pod安全策略

```yaml
# k8s/security/psp.yaml
apiVersion: policy/v1
kind: PodSecurityPolicy
metadata:
  name: ai-agent-api
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'runtime/default'
    apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
    seccomp.security.alpha.kubernetes.io/defaultProfileName:  'runtime/default'
    apparmor.security.beta.kubernetes.io/defaultProfileName:  'runtime/default'
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  allowedCapabilities:
    - NET_BIND_SERVICE
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'secret'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

### 8.2 NetworkPolicy

```yaml
# k8s/security/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ai-agent-api-network-policy
  namespace: ai-agent
spec:
  podSelector:
    matchLabels:
      app: ai-agent-api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    # 允许来自Ingress Controller的流量
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8000
    # 允许Prometheus抓取指标
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
          podSelector:
            matchLabels:
              app: prometheus
      ports:
        - protocol: TCP
          port: 8000
  
  egress:
    # 允许访问PostgreSQL
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              app: postgres
      ports:
        - protocol: TCP
          port: 5432
    
    # 允许访问Redis
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              app: redis
      ports:
        - protocol: TCP
          port: 6379
    
    # 允许访问外部API
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 10.0.0.0/8
              - 172.16.0.0/12
              - 192.168.0.0/16
      ports:
        - protocol: TCP
          port: 443
    
    # 允许DNS
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

## 9. 灾难恢复

### 9.1 备份策略

```yaml
# k8s/backup/backup-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ai-agent-backup
  namespace: ai-agent
spec:
  schedule: "0 2 * * *"  # 每天凌晨2点
  successfulJobsHistoryLimit: 7
  failedJobsHistoryLimit: 3
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: backup-sa
          securityContext:
            runAsUser: 1000
            runAsGroup: 1000
          containers:
            - name: backup
              image: postgres:15-alpine
              command:
                - /bin/sh
                - -c
                - |
                  # 数据库备份
                  pg_dump -h postgres -U postgres -d ai_agent | gzip > /backup/db_$(date +%Y%m%d_%H%M%S).sql.gz
                  
                  # 保留最近30天的备份
                  find /backup -name "*.sql.gz" -mtime +30 -delete
                  
                  # 上传到对象存储
                  mc cp /backup/db_*.sql.gz minio/ai-agent-backups/
                  
                  # 删除本地旧备份
                  rm /backup/db_*.sql.gz
              env:
                - name: PGPASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: ai-agent-secrets
                      key: DATABASE_PASSWORD
              volumeMounts:
                - name: backup-volume
                  mountPath: /backup
          volumes:
            - name: backup-volume
              emptyDir: {}
          restartPolicy: OnFailure
```

### 9.2 恢复流程

```bash
#!/bin/bash
# restore.sh - 灾难恢复脚本

set -e

# 配置
BACKUP_DATE=${1:-$(date +%Y%m%d)}
NAMESPACE="ai-agent"
POSTGRES_POD="ai-agent-db-0"

echo "=== 开始恢复 ==="
echo "备份日期: $BACKUP_DATE"

# 1. 停止服务
echo "1. 停止应用服务..."
kubectl scale deployment ai-agent-api --replicas=0 -n $NAMESPACE

# 2. 等待现有连接断开
echo "2. 等待连接断开..."
sleep 30

# 3. 删除旧数据
echo "3. 删除旧数据..."
kubectl exec -n $NAMESPACE $POSTGRES_POD -- psql -U postgres -c "DROP DATABASE IF EXISTS ai_agent;"
kubectl exec -n $NAMESPACE $POSTGRES_POD -- psql -U postgres -c "CREATE DATABASE ai_agent;"

# 4. 恢复数据
echo "4. 恢复数据..."
kubectl exec -n $NAMESPACE $POSTGRES_POD -- \
  sh -c "mc cat minio/ai-agent-backups/db_${BACKUP_DATE}_*.sql.gz | gunzip | psql -U postgres -d ai_agent"

# 5. 验证数据
echo "5. 验证数据..."
kubectl exec -n $NAMESPACE $POSTGRES_POD -- psql -U postgres -d ai_agent -c "SELECT COUNT(*) FROM users;"

# 6. 启动服务
echo "6. 启动服务..."
kubectl scale deployment ai-agent-api --replicas=3 -n $NAMESPACE

# 7. 验证服务
echo "7. 验证服务..."
kubectl rollout status deployment/ai-agent-api -n $NAMESPACE

echo "=== 恢复完成 ==="
```

## 10. 总结

### 部署检查清单

| 阶段 | 检查项 | 说明 |
|------|--------|------|
| **容器化** | ✅ 多阶段构建 | 减小镜像体积 |
| | ✅ 非root用户 | 安全加固 |
| | ✅ 健康检查 | K8s探针 |
| | ✅ 日志配置 | JSON格式 |
| **K8s** | ✅ 资源限制 | 防止资源耗尽 |
| | ✅ 探针配置 | 存活/就绪/启动 |
| | ✅ 滚动更新 | 不停机发布 |
| | ✅ PDB | 保证可用性 |
| **监控** | ✅ 指标暴露 | Prometheus |
| | ✅ 告警规则 | 及时发现问题 |
| | ✅ 仪表盘 | 可视化 |
| **CI/CD** | ✅ 测试覆盖 | 质量保证 |
| | ✅ 安全扫描 | 代码安全 |
| | ✅ 灰度发布 | 平滑过渡 |
| **安全** | ✅ 网络策略 | 最小权限 |
| | ✅ 密钥管理 | 不明文存储 |
| | ✅ Pod安全 | 加固配置 |

---

## 相关资源

- [[AI应用API化部署]] - API部署方案
- [[工作流设计模式]] - 工作流设计原则
- [[多Agent系统设计]] - 多智能体架构
- [[插件开发]] - AI平台插件开发

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
