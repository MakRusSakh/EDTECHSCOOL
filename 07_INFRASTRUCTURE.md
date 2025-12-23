# Часть VII: Инфраструктура

## Содержание

- [13.1. Обзор инфраструктуры](#131-обзор-инфраструктуры)
- [13.2. Облачная архитектура](#132-облачная-архитектура)
- [13.3. Kubernetes](#133-kubernetes)
- [13.4. CI/CD Pipeline](#134-cicd-pipeline)
- [13.5. Мониторинг](#135-мониторинг)
- [13.6. Безопасность](#136-безопасность)
- [13.7. Disaster Recovery](#137-disaster-recovery)

---

## 13.1. Обзор инфраструктуры

### Принципы

| Принцип | Описание | Реализация |
|---------|----------|------------|
| **IaC** | Инфраструктура как код | Terraform |
| **Immutable** | Не изменяем, а пересоздаём | Containers |
| **GitOps** | Git — источник истины | ArgoCD |
| **Zero Trust** | Не доверяем по умолчанию | mTLS |

### Структура

```
ИНФРАСТРУКТУРА
├── COMPUTE
│   ├── Kubernetes Cluster
│   ├── GPU Nodes (ML)
│   └── Serverless
├── DATA
│   ├── PostgreSQL
│   ├── MongoDB
│   ├── Redis
│   ├── Elasticsearch
│   └── S3
├── NETWORKING
│   ├── Load Balancer
│   ├── CDN
│   └── Service Mesh
├── SECURITY
│   ├── WAF
│   ├── Secrets Manager
│   └── Certificates
└── OBSERVABILITY
    ├── Prometheus
    ├── Grafana
    ├── Loki
    └── Jaeger
```

---

## 13.2. Облачная архитектура

### Провайдер: Yandex Cloud (РФ)

| Сервис | YC | Применение |
|--------|-----------|------------|
| Compute | Compute Cloud | K8s nodes |
| Kubernetes | Managed K8s | Оркестрация |
| Database | Managed PostgreSQL | OLTP |
| Database | Managed MongoDB | Documents |
| Cache | Managed Redis | Кэш |
| Storage | Object Storage | Файлы |
| CDN | CDN | Статика |
| LB | ALB | Балансировка |

### Сетевая топология

```
INTERNET
    │
    ▼
┌─────────────────────────────────┐
│  CDN + DDoS (Cloudflare)       │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  WAF + Load Balancer           │
└─────────────────────────────────┘
    │
════╪═════════════════════════════
    │   VPC: 10.0.0.0/16
    ▼
┌─────────────────────────────────┐
│  PUBLIC SUBNET 10.0.1.0/24     │
│  • Ingress Controller          │
│  • NAT Gateway                 │
│  • Bastion                     │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  PRIVATE SUBNET 10.0.10.0/24   │
│  ┌────────────────────────────┐│
│  │   KUBERNETES CLUSTER       ││
│  │   Node 1 │ Node 2 │ Node N ││
│  └────────────────────────────┘│
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  DATA SUBNET 10.0.20.0/24      │
│  PostgreSQL │ MongoDB │ Redis  │
└─────────────────────────────────┘
```

### Зоны доступности

| Компонент | Зоны | Реплики |
|-----------|------|---------|
| K8s Nodes | 3 | 3+ |
| PostgreSQL | 2 | 3 |
| MongoDB | 3 | 3 |
| Redis | 2 | 6 |

---

## 13.3. Kubernetes

### Структура кластера

```
NAMESPACES
├── production
│   ├── ai-core
│   ├── edu-core
│   ├── operations
│   └── frontend
├── staging
├── monitoring
│   ├── prometheus
│   ├── grafana
│   └── loki
└── system
    ├── argocd
    ├── cert-manager
    └── external-secrets
```

### Node Pools

| Pool | Specs | Назначение |
|------|-------|------------|
| general | 4 vCPU, 16GB | API services |
| compute | 8 vCPU, 32GB | AI/ML workloads |
| gpu | T4/A10 | Model inference |
| spot | variable | Batch jobs |

### Пример Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-tutor-service
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      containers:
        - name: ai-tutor
          image: registry/ai-tutor:v1.2.3
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2000m"
              memory: "4Gi"
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
```

### Autoscaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ai-tutor-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ai-tutor-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 13.4. CI/CD Pipeline

### Схема

```
Developer
    │ git push
    ▼
┌─────────────────────────────────┐
│          GitHub                 │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│              GITHUB ACTIONS                     │
│  Lint → Test → Build → Scan → Push to Registry │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│      Container Registry         │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│          ArgoCD                 │
│      (GitOps Sync)             │
└─────────────────────────────────┘
    │
    ├──────────┬──────────┐
    ▼          ▼          ▼
 Staging   Production  (Manual)
```

### GitHub Actions

```yaml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          pip install ruff mypy
          ruff check .

  test:
    needs: lint
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
    steps:
      - uses: actions/checkout@v4
      - run: pytest --cov=app

  build:
    needs: test
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

  security-scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: aquasecurity/trivy-action@master
        with:
          severity: 'CRITICAL,HIGH'

  deploy-staging:
    needs: security-scan
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Update staging via ArgoCD"

  deploy-production:
    needs: security-scan
    if: github.ref == 'refs/heads/main'
    environment: production
    runs-on: ubuntu-latest
    steps:
      - run: echo "Update production via ArgoCD"
```

---

## 13.5. Мониторинг

### Стек

| Компонент | Инструмент |
|-----------|------------|
| Metrics | Prometheus |
| Visualization | Grafana |
| Logs | Loki |
| Traces | Jaeger |
| Errors | Sentry |
| Alerts | Alertmanager |
| On-call | PagerDuty |

### SLI/SLO

| Сервис | SLI | SLO |
|--------|-----|-----|
| API Gateway | Latency p99 | < 200ms |
| API Gateway | Success rate | > 99.9% |
| AI Tutor | Latency p95 | < 2s |
| AI Tutor | Success rate | > 99% |
| Video | Buffering | < 1% |
| Platform | Uptime | > 99.9% |

### Alerting Rules

```yaml
groups:
  - name: sla-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) 
          / sum(rate(http_requests_total[5m])) > 0.01
        for: 5m
        labels:
          severity: critical
          
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
          ) > 0.5
        for: 5m
        labels:
          severity: warning
          
      - alert: AITutorSlow
        expr: |
          histogram_quantile(0.95, 
            sum(rate(ai_tutor_response_seconds_bucket[5m])) by (le)
          ) > 3
        for: 5m
        labels:
          severity: warning
```

---

## 13.6. Безопасность

### Уровни защиты

| Уровень | Защита | Инструмент |
|---------|--------|------------|
| Edge | DDoS | Cloudflare |
| Edge | WAF | Cloudflare WAF |
| Ingress | Rate limiting | NGINX |
| Internal | Network policies | K8s |
| Internal | mTLS | Istio |

### Управление секретами

```
┌───────────────────────────────────┐
│        HashiCorp Vault            │
└─────────────┬─────────────────────┘
              │
              ▼
┌───────────────────────────────────┐
│    External Secrets Operator      │
└─────────────┬─────────────────────┘
              │
              ▼
┌───────────────────────────────────┐
│      Kubernetes Secrets           │
└─────────────┬─────────────────────┘
              │
              ▼
┌───────────────────────────────────┐
│         Pods (env vars)           │
└───────────────────────────────────┘
```

### Compliance

| Требование | Реализация | Статус |
|------------|------------|--------|
| 152-ФЗ: Локализация | Данные в РФ | ✓ |
| 152-ФЗ: Согласие | Форма | ✓ |
| Шифрование at rest | AES-256 | ✓ |
| Шифрование in transit | TLS 1.3 | ✓ |
| Аудит доступа | Logging | ✓ |
| Backup | Daily | ✓ |
| Pentest | Quarterly | Plan |

---

## 13.7. Disaster Recovery

### Backup стратегия

| Данные | Частота | Retention |
|--------|---------|-----------|
| PostgreSQL | 1 час | 30 дней |
| MongoDB | 6 часов | 30 дней |
| Redis | 1 час | 7 дней |
| Object Storage | Continuous | 90 дней |
| Config | On change | Forever |

### RTO/RPO

| Сценарий | RPO | RTO |
|----------|-----|-----|
| Сбой сервиса | 0 | 5 мин |
| Сбой ноды | 0 | 10 мин |
| Сбой зоны | 5 мин | 30 мин |
| Сбой региона | 1 час | 4 часа |
| Полная потеря | 1 час | 24 часа |

### Runbook: Database Recovery

```bash
# 1. Остановить приложения
kubectl scale deployment --all --replicas=0 -n production

# 2. Восстановить из бэкапа
pg_restore -h $DB_HOST -U admin -d sphere_prod backup.dump

# 3. Проверить целостность
psql -h $DB_HOST -c "SELECT count(*) FROM users;"

# 4. Запустить приложения
kubectl scale deployment --all --replicas=3 -n production

# 5. Мониторинг
kubectl logs -f deployment/api-gateway -n production
```

---

*Следующий раздел: [08_ROADMAP.md](08_ROADMAP.md)*
