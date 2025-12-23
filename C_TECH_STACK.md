# Приложение C: Технологический стек

## Содержание

- [C.1. Обзор стека](#c1-обзор-стека)
- [C.2. Backend](#c2-backend)
- [C.3. Frontend](#c3-frontend)
- [C.4. Mobile](#c4-mobile)
- [C.5. ML/AI](#c5-mlai)
- [C.6. Данные](#c6-данные)
- [C.7. Инфраструктура](#c7-инфраструктура)
- [C.8. Внешние сервисы](#c8-внешние-сервисы)

---

## C.1. Обзор стека

### Архитектурная схема

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              TECH STACK                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  FRONTEND                     BACKEND                    ML/AI             │
│  ┌─────────────┐             ┌─────────────┐            ┌─────────────┐    │
│  │ React 18    │             │ Python 3.11 │            │ PyTorch     │    │
│  │ Next.js 14  │             │ FastAPI     │            │ Transformers│    │
│  │ TypeScript  │             │ Pydantic    │            │ LangChain   │    │
│  │ TailwindCSS │             │ SQLAlchemy  │            │ OpenAI API  │    │
│  │ Zustand     │             │ Celery      │            │ Anthropic   │    │
│  └─────────────┘             └─────────────┘            └─────────────┘    │
│                                                                             │
│  MOBILE                       DATA                       INFRA             │
│  ┌─────────────┐             ┌─────────────┐            ┌─────────────┐    │
│  │ Swift (iOS) │             │ PostgreSQL  │            │ Kubernetes  │    │
│  │ Kotlin      │             │ MongoDB     │            │ Terraform   │    │
│  │ (Android)   │             │ Redis       │            │ GitHub      │    │
│  │ — или —     │             │ Elastic     │            │ Actions     │    │
│  │ Flutter     │             │ S3/Minio    │            │ Prometheus  │    │
│  └─────────────┘             └─────────────┘            └─────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Критерии выбора

| Критерий | Вес | Комментарий |
|----------|-----|-------------|
| Зрелость | 25% | Проверенные технологии в production |
| Экосистема | 20% | Библиотеки, инструменты, сообщество |
| Найм | 20% | Доступность специалистов на рынке |
| Производительность | 15% | Скорость, масштабируемость |
| Developer Experience | 10% | Удобство разработки |
| Стоимость | 10% | Лицензии, хостинг |

---

## C.2. Backend

### Основной стек

| Компонент | Технология | Версия | Обоснование |
|-----------|------------|--------|-------------|
| Язык | Python | 3.11+ | ML-экосистема, скорость разработки |
| Web Framework | FastAPI | 0.100+ | Async, автодокументация, типизация |
| ORM | SQLAlchemy | 2.0 | Зрелость, гибкость |
| Валидация | Pydantic | 2.0 | Интеграция с FastAPI |
| Очереди | Celery | 5.3 | Фоновые задачи |
| Брокер | Redis / RabbitMQ | — | Очереди сообщений |
| gRPC | grpcio | — | Внутренние коммуникации |

### Альтернатива для высоконагруженных сервисов

| Компонент | Технология | Применение |
|-----------|------------|------------|
| Язык | Go | API Gateway, высоконагруженные сервисы |
| Framework | Gin / Echo | REST API |
| gRPC | grpc-go | Межсервисная коммуникация |

### Структура сервиса (Python)

```
service/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app
│   ├── api/
│   │   ├── v1/
│   │   │   ├── endpoints/
│   │   │   └── router.py
│   │   └── deps.py          # Dependencies
│   ├── core/
│   │   ├── config.py        # Settings
│   │   └── security.py      # Auth
│   ├── models/              # SQLAlchemy models
│   ├── schemas/             # Pydantic schemas
│   ├── services/            # Business logic
│   └── repositories/        # Data access
├── tests/
├── alembic/                 # Migrations
├── Dockerfile
└── pyproject.toml
```

---

## C.3. Frontend

### Основной стек

| Компонент | Технология | Версия | Обоснование |
|-----------|------------|--------|-------------|
| Язык | TypeScript | 5.0+ | Типобезопасность |
| Framework | React | 18+ | Экосистема, найм |
| Meta-framework | Next.js | 14+ | SSR, оптимизации |
| Стили | TailwindCSS | 3.4+ | Utility-first |
| State | Zustand | — | Простота |
| Forms | React Hook Form | — | Производительность |
| Запросы | TanStack Query | — | Кэширование, мутации |
| UI Kit | Radix UI | — | Доступность |
| Charts | Recharts | — | Графики |
| Video | Video.js | — | Видеоплеер |

### Структура проекта

```
frontend/
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── (auth)/
│   │   ├── (student)/
│   │   ├── (parent)/
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/              # Базовые компоненты
│   │   ├── features/        # Фичи
│   │   └── layouts/
│   ├── hooks/
│   ├── stores/              # Zustand stores
│   ├── services/            # API clients
│   ├── types/
│   └── utils/
├── public/
├── tests/
└── package.json
```

---

## C.4. Mobile

### Вариант 1: Нативная разработка

| Платформа | Технология | Обоснование |
|-----------|------------|-------------|
| iOS | Swift + SwiftUI | Производительность, UX |
| Android | Kotlin + Jetpack Compose | Современный стек |

**Плюсы:** Максимальная производительность, доступ к платформенным API
**Минусы:** Две кодовые базы, больше разработчиков

### Вариант 2: Кросс-платформенная разработка

| Технология | Версия | Обоснование |
|------------|--------|-------------|
| Flutter | 3.16+ | Единая кодовая база, производительность |
| Dart | 3.0+ | Язык Flutter |

**Плюсы:** Одна команда, быстрее разработка
**Минусы:** Некоторые ограничения платформенных API

### Рекомендация

**Flutter** для MVP и первых версий, с возможностью перехода на натив для критичных модулей.

---

## C.5. ML/AI

### Основной стек

| Компонент | Технология | Применение |
|-----------|------------|------------|
| Framework | PyTorch | Обучение моделей |
| Transformers | HuggingFace | NLP модели |
| LLM интеграция | LangChain | Работа с LLM |
| Embeddings | sentence-transformers | Векторные представления |
| Vector DB | Pinecone / Qdrant | RAG |
| Experiment tracking | MLflow / W&B | Отслеживание экспериментов |
| Model serving | TorchServe / Triton | Inference |

### LLM провайдеры

| Провайдер | Модель | Применение |
|-----------|--------|------------|
| OpenAI | GPT-4o | AI Tutor, генерация |
| Anthropic | Claude 3 | AI Tutor, проверка |
| YandexGPT | YaGPT | Fallback для РФ |

### Feature Store

| Компонент | Технология | Назначение |
|-----------|------------|------------|
| Feature Store | Feast | Хранение признаков |
| Model Registry | MLflow | Версии моделей |
| Data Pipeline | Apache Airflow | ETL |

---

## C.6. Данные

### Хранилища

| Тип | Технология | Применение |
|----|------------|------------|
| OLTP | PostgreSQL 15+ | Основные транзакционные данные |
| Document | MongoDB 7+ | Контент, профили, логи |
| Cache | Redis 7+ | Кэш, сессии, pub/sub |
| Search | Elasticsearch 8+ | Полнотекстовый поиск |
| Object | S3 / Minio | Файлы, видео |
| Vector | Qdrant | Embeddings для RAG |
| Time-series | TimescaleDB | Метрики, аналитика |

### Распределение данных

| Сущность | Хранилище | Обоснование |
|----------|-----------|-------------|
| Users, Subscriptions | PostgreSQL | ACID, связи |
| Courses, Modules | MongoDB | Гибкая схема |
| Progress, Grades | PostgreSQL | Транзакции |
| TutorSessions | MongoDB | Документы |
| Sessions, Tokens | Redis | TTL, скорость |
| Content files | S3 | Бинарные данные |
| Search index | Elasticsearch | Полнотекстовый |

---

## C.7. Инфраструктура

### Облачная платформа

| Вариант | Комментарий |
|---------|-------------|
| Яндекс.Облако | Для РФ, 152-ФЗ |
| AWS | Глобальная инфраструктура |
| GCP | ML-инструменты |

### Container Orchestration

| Компонент | Технология |
|-----------|------------|
| Orchestration | Kubernetes (EKS/GKE/YC MK8S) |
| Service Mesh | Istio / Linkerd |
| Ingress | NGINX Ingress / Traefik |

### CI/CD

| Этап | Инструмент |
|------|------------|
| VCS | GitHub |
| CI | GitHub Actions |
| CD | ArgoCD |
| Registry | GitHub Container Registry |

### IaC

| Компонент | Технология |
|-----------|------------|
| Infrastructure | Terraform |
| Configuration | Ansible |
| Secrets | HashiCorp Vault |

### Мониторинг

| Компонент | Технология |
|-----------|------------|
| Metrics | Prometheus + Grafana |
| Logs | Loki / ELK |
| Traces | Jaeger |
| APM | Sentry |
| Alerts | Alertmanager + PagerDuty |

---

## C.8. Внешние сервисы

### Коммуникации

| Канал | Сервис |
|-------|--------|
| Email | SendGrid |
| SMS | Twilio |
| Push (iOS) | APNs |
| Push (Android) | FCM |
| Push (Web) | OneSignal |

### Платежи

| Регион | Сервис |
|--------|--------|
| РФ | ЮKassa |
| Global | Stripe |
| СНГ | Тинькофф |

### Видео

| Функция | Сервис |
|---------|--------|
| Конференции | Jitsi Meet (self-hosted) |
| Hosting | Kinescope / Cloudflare Stream |
| CDN | Cloudflare |

### Аутентификация

| Метод | Сервис |
|-------|--------|
| Social OAuth | Google, VK, Яндекс |
| Apple | Apple Sign In |
| Госуслуги | ЕСИА |

### Аналитика

| Функция | Сервис |
|---------|--------|
| Product Analytics | Amplitude |
| Error Tracking | Sentry |
| APM | Datadog / Grafana Cloud |

---

*Назад к документации: [README.md](../README.md)*
