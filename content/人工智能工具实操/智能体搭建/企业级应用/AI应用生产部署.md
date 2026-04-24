---
title: AI应用生产部署
date: 2026-04-18
tags:
  - Docker
  - Kubernetes
  - 高可用
  - 监控
  - CI/CD
  - 生产环境
categories:
  - 智能体搭建
  - 企业级应用
alias: Production AI Deployment
---

# AI应用生产部署

> [!abstract] 摘要
> 本文档介绍AI应用在生产环境中的最佳部署实践，包括Docker容器化、Kubernetes编排、高可用架构设计、监控系统配置以及CI/CD流水线构建。通过完整的架构图和配置示例，帮助读者构建企业级的AI应用基础设施。

## 核心关键词速览

| 关键词 | 说明 | 关键词 | 说明 |
|--------|------|--------|------|
| Docker | 容器化部署 | Kubernetes | 容器编排 |
| 高可用 | HA Architecture | 监控 | Prometheus/Grafana |
| CI/CD | 持续集成部署 | 日志收集 | ELK Stack |
| 负载均衡 | Load Balancing | 自动扩缩容 | HPA |
| 健康检查 | Health Check | 滚动更新 | Rolling Update |

## 1. Docker容器化

### 1.1 Dockerfile编写

```dockerfile
# 使用多阶段构建优化镜像大小
FROM python:3.11-slim AS builder

# 安装构建依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/opt/venv -r requirements.txt

# 生产镜像
FROM python:3.11-slim

# 安全配置
RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /home/appuser

# 从构建阶段复制
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 复制应用代码
COPY --chown=appuser:appuser . .

# 切换到非root用户
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python healthcheck.py || exit 1

# 环境变量
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/home/appuser

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--threads", "2", "app.main:app"]
```

### 1.2 requirements.txt

```txt
# 核心框架
fastapi==0.109.0
uvicorn[standard]==0.27.0
gunicorn==21.2.0

# AI/ML
openai==1.10.0
anthropic==0.18.0
httpx==0.26.0

# 数据库
asyncpg==0.29.0
sqlalchemy[asyncio]==2.0.25
redis==5.0.1

# 监控
prometheus-client==0.19.0
opentelemetry-api==1.22.0
opentelemetry-sdk==1.22.0

# 工具
pydantic==2.6.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
```

### 1.3 Docker Compose配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ai-agent-api
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./logs:/home/appuser/logs
    healthcheck:
      test: ["CMD", "python", "healthcheck.py"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - ai-network
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '0.5'
          memory: 1G

  redis:
    image: redis:7-alpine
    container_name: ai-agent-redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - ai-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  postgres:
    image: postgres:15-alpine
    container_name: ai-agent-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: ai_agent
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - ai-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 3

networks:
  ai-network:
    driver: bridge

volumes:
  redis-data:
  postgres-data:
```

## 2. Kubernetes部署

### 2.1 命名空间与配置

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ai-agent
  labels:
    name: ai-agent
    env: production
---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ai-agent-config
  namespace: ai-agent
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  MAX_WORKERS: "4"
  REDIS_HOST: "redis-master"
  REDIS_PORT: "6379"
---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ai-agent-secrets
  namespace: ai-agent
type: Opaque
stringData:
  DATABASE_URL: "postgresql://user:pass@postgres:5432/ai_agent"
  OPENAI_API_KEY: "sk-..."
  REDIS_URL: "redis://redis:6379"
```

### 2.2 Deployment配置

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-agent-api
  namespace: ai-agent
  labels:
    app: ai-agent-api
    version: v1
spec:
  replicas: 3
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
        version: v1
    spec:
      serviceAccountName: ai-agent-api
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
        - name: api
          image: registry.example.com/ai-agent:v1.0.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8000
              name: http
          
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: ai-agent-secrets
                  key: DATABASE_URL
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: ai-agent-secrets
                  key: OPENAI_API_KEY
          
          envFrom:
            - configMapRef:
                name: ai-agent-config
          
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 2000m
              memory: 4Gi
          
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]
```

### 2.3 Service与Ingress

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ai-agent-api
  namespace: ai-agent
spec:
  selector:
    app: ai-agent-api
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 8000
      protocol: TCP
      name: http
---
# k8s/hpa.yaml
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
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ai-agent-ingress
  namespace: ai-agent
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.example.com
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
```

## 3. 高可用架构

### 3.1 架构设计

```mermaid
graph TB
    subgraph "用户层"
        U1[用户1]
        U2[用户2]
        UN[用户N]
    end
    
    subgraph "负载均衡层"
        LB[Load Balancer]
    end
    
    subgraph "Kubernetes集群"
        subgraph "AI Agent服务"
            P1[Pod 1]
            P2[Pod 2]
            P3[Pod 3]
        end
        
        subgraph "数据层"
            PG[(PostgreSQL)]
            RD[(Redis)]
            VS[(向量数据库)]
        end
    end
    
    subgraph "外部服务"
        AI[AI Provider]
        OBS[对象存储]
    end
    
    U1 & U2 & UN --> LB
    LB --> P1 & P2 & P3
    P1 & P2 & P3 --> PG & RD & VS
    P1 & P2 & P3 --> AI & OBS
```

### 3.2 PostgreSQL高可用

```yaml
# k8s/postgres-ha.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: ai-agent-db
  namespace: ai-agent
spec:
  instances: 3
  imageName: ghcr.io/cloudnative-pg/postgresql:15
  
  primaryUpdateStrategy: unsupervised
  
  storage:
    storageClass: ssd
    size: 100Gi
  
  resources:
    limits:
      cpu: 2
      memory: 4Gi
  
  backup:
    volumeSnapshot:
      className: csi-snapclass
      inventoryPolicy: Mine
      
  superuserSecret:
    name: postgres-secret
  
  monitoring:
    enablePodMonitoring: true
```

## 4. 监控系统

### 4.1 Prometheus配置

```yaml
# monitoring/prometheus-config.yaml
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
    
    alerting:
      alertmanagers:
        - static_configs:
            - targets: ["alertmanager:9093"]
    
    rule_files:
      - "/etc/prometheus/rules/*.yml"
    
    scrape_configs:
      - job_name: 'ai-agent-api'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_label_app]
            action: keep
            regex: ai-agent-api
          - source_labels: [__meta_kubernetes_pod_container_port_number]
            action: keep
            regex: "8000"
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
      
      - job_name: 'redis'
        kubernetes_sd_configs:
          - role: service
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_label_app]
            action: keep
            regex: redis
      
      - job_name: 'postgres'
        kubernetes_sd_configs:
          - role: service
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_label_app]
            action: keep
            regex: postgres
```

### 4.2 自定义指标

```python
# app/monitoring.py
from prometheus_client import Counter, Histogram, Gauge, CollectorRegistry
from prometheus_fastapi_instrumentator import Instrumentator

# 定义业务指标
REQUEST_COUNT = Counter(
    'ai_agent_requests_total',
    'Total number of requests',
    ['method', 'endpoint', 'status_code']
)

REQUEST_LATENCY = Histogram(
    'ai_agent_request_duration_seconds',
    'Request latency in seconds',
    ['method', 'endpoint'],
    buckets=[0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0]
)

ACTIVE_SESSIONS = Gauge(
    'ai_agent_active_sessions',
    'Number of active chat sessions'
)

TOKEN_USAGE = Counter(
    'ai_agent_token_usage_total',
    'Total tokens consumed',
    ['model', 'agent_id']
)

LLM_CALLS = Counter(
    'ai_agent_llm_calls_total',
    'Total LLM API calls',
    ['model', 'status']
)

# FastAPI集成
def setup_monitoring(app: FastAPI):
    Instrumentator(
        namespace="ai_agent",
        instrumentinator=app
    ).add(
        REQUEST_COUNT,
        REQUEST_LATENCY
    ).expose(app, endpoint="/metrics")
    
    @app.middleware("http")
    async def monitor_requests(request: Request, call_next):
        import time
        start = time.time()
        
        response = await call_next(request)
        
        duration = time.time() - start
        REQUEST_LATENCY.labels(
            method=request.method,
            endpoint=request.url.path
        ).observe(duration)
        
        REQUEST_COUNT.labels(
            method=request.method,
            endpoint=request.url.path,
            status_code=response.status_code
        ).inc()
        
        return response
```

### 4.3 Grafana仪表盘

```json
{
  "dashboard": {
    "title": "AI Agent 监控面板",
    "panels": [
      {
        "title": "请求量",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(ai_agent_requests_total[5m])) by (endpoint)",
            "legendFormat": "{{endpoint}}"
          }
        ]
      },
      {
        "title": "延迟分布",
        "type": "heatmap",
        "targets": [
          {
            "expr": "sum(rate(ai_agent_request_duration_seconds_bucket[5m])) by (le)",
            "legendFormat": "{{le}}"
          }
        ]
      },
      {
        "title": "错误率",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(rate(ai_agent_requests_total{status_code=~'5..'}[5m])) / sum(rate(ai_agent_requests_total[5m])) * 100"
          }
        ]
      },
      {
        "title": "活跃会话数",
        "type": "gauge",
        "targets": [
          {
            "expr": "ai_agent_active_sessions"
          }
        ]
      },
      {
        "title": "Token消耗",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(ai_agent_token_usage_total[1h])) by (model)",
            "legendFormat": "{{model}}"
          }
        ]
      }
    ]
  }
}
```

## 5. CI/CD流水线

### 5.1 GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: |
          pytest --cov=app tests/
      
      - name: Security scan
        run: |
          pip install safety bandit
          safety check
          bandit -r app/

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha,prefix=
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Deploy to Staging
        run: |
          kubectl config use-context staging
          kubectl set image deployment/ai-agent-api api=${{ needs.build.outputs.image-tag }}
          kubectl rollout status deployment/ai-agent-api -n ai-agent --timeout=300s

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production
    
    steps:
      - name: Deploy to Production
        run: |
          kubectl config use-context production
          kubectl set image deployment/ai-agent-api api=${{ needs.build.outputs.image-tag }}
          kubectl rollout status deployment/ai-agent-api -n ai-agent --timeout=300s
```

### 5.2 滚动更新策略

```yaml
# 安全的滚动更新配置
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%        # 最多增加25%的Pod
      maxUnavailable: 0    # 不能有不可用的Pod
  
  # 更新时暂停控制
  paused: false
```

## 6. 日志与追踪

### 6.1 结构化日志

```python
# app/logging.py
import logging
import json
from datetime import datetime
from typing import Any
from pythonjsonlogger import jsonlogger

class CustomJsonFormatter(jsonlogger.JsonFormatter):
    """自定义JSON日志格式"""
    
    def add_fields(self, log_record: dict, record: logging.LogRecord, message_dict: dict):
        super().add_fields(log_record, record, message_dict)
        
        log_record['timestamp'] = datetime.utcnow().isoformat()
        log_record['level'] = record.levelname
        log_record['logger'] = record.name
        log_record['service'] = 'ai-agent-api'
        
        if hasattr(record, 'request_id'):
            log_record['request_id'] = record.request_id
        
        if hasattr(record, 'user_id'):
            log_record['user_id'] = record.user_id

def setup_logging():
    """配置日志"""
    handler = logging.StreamHandler()
    formatter = CustomJsonFormatter(
        '%(timestamp)s %(level)s %(name)s %(message)s'
    )
    handler.setFormatter(formatter)
    
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    logger.handlers = []
    logger.addHandler(handler)
    
    # uvicorn日志
    logging.getLogger("uvicorn.access").handlers = []
```

### 6.2 分布式追踪

```python
# app/tracing.py
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.resources import Resource
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

def setup_tracing(app: FastAPI):
    """配置分布式追踪"""
    # 创建资源
    resource = Resource.create({
        "service.name": "ai-agent-api",
        "service.version": "1.0.0",
    })
    
    # 创建追踪器
    provider = TracerProvider(resource=resource)
    
    # 配置Jaeger导出器
    jaeger_exporter = JaegerExporter(
        agent_host_name="jaeger",
        agent_port=6831,
    )
    
    provider.add_span_processor(
        BatchSpanProcessor(jaeger_exporter)
    )
    
    trace.set_tracer_provider(provider)
    
    # FastAPI自动instrumentation
    FastAPIInstrumentor.instrument_app(app)
    
    return trace.get_tracer(__name__)
```

## 7. 相关资源

- [[AI应用API化部署]] - API部署方案
- [[工作流设计模式]] - 工作流设计原则
- [[多Agent系统设计]] - 多智能体架构

---

*本文档由归愚知识系统自动生成 last updated: 2026-04-18*
