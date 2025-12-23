# Часть II: Системная архитектура

## Содержание

- [5. Общая архитектура](#5-общая-архитектура)
- [6. Модель данных](#6-модель-данных)
- [7. Интеграционная архитектура](#7-интеграционная-архитектура)
- [8. Архитектура безопасности](#8-архитектура-безопасности)

---

## 5. Общая архитектура

### 5.1. Высокоуровневая схема

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         КЛИЕНТСКИЙ УРОВЕНЬ                                  │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐               │
│  │Web App  │ │iOS App  │ │Android  │ │Admin    │ │Partner  │               │
│  │(React)  │ │(Swift)  │ │(Kotlin) │ │Panel    │ │API      │               │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘               │
│       │          │          │          │          │                        │
└───────┼──────────┼──────────┼──────────┼──────────┼────────────────────────┘
        │          │          │          │          │
        └──────────┴──────────┴──────────┴──────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────────────────┐
│                            CDN / EDGE                                      │
│                    Cloudflare (статика, видео, кэш)                       │
└─────────────────────────────┬─────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────────────────┐
│                           API GATEWAY                                      │
│              Kong / AWS API Gateway / Custom                               │
│         (аутентификация, rate limiting, маршрутизация)                    │
└─────────────────────────────┬─────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────────────────┐
│                      СЕРВИСНЫЙ УРОВЕНЬ                                     │
│                                                                            │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                         AI CORE                                    │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐  │   │
│  │  │Adaptive  │ │AI Tutor  │ │Content   │ │Assessment│ │Analytics│  │   │
│  │  │Engine    │ │Service   │ │Generator │ │Engine    │ │Engine   │  │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └─────────┘  │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                         EDU CORE                                   │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐  │   │
│  │  │Content   │ │Learning  │ │Social    │ │Live      │ │Schedule │  │   │
│  │  │Mgmt      │ │Process   │ │Learning  │ │Sessions  │ │Service  │  │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └─────────┘  │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                        OPERATIONS                                  │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐  │   │
│  │  │Billing   │ │CRM       │ │Support   │ │Documents │ │Notify   │  │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └─────────┘  │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                      IDENTITY & ACCESS                             │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                            │   │
│  │  │Auth      │ │User      │ │Permission│                            │   │
│  │  │Service   │ │Service   │ │Service   │                            │   │
│  │  └──────────┘ └──────────┘ └──────────┘                            │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
└─────────────────────────────┬─────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────────────────┐
│                        УРОВЕНЬ ДАННЫХ                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│  │PostgreSQL│ │MongoDB   │ │Redis     │ │Elastic-  │ │S3/Minio  │         │
│  │(основные)│ │(контент) │ │(кэш)     │ │search    │ │(файлы)   │         │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘         │
└─────────────────────────────┬─────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────────────────┐
│                         ML PLATFORM                                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│  │Feature   │ │Model     │ │Model     │ │Experiment│ │LLM       │         │
│  │Store     │ │Registry  │ │Serving   │ │Tracking  │ │Gateway   │         │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘         │
└───────────────────────────────────────────────────────────────────────────┘
```

### 5.2. Описание доменов

| Домен | Ответственность | Сервисы | Зависимости |
|-------|-----------------|---------|-------------|
| **AI Core** | Все ИИ-функции | Adaptive Engine, AI Tutor, Content Generator, Assessment Engine, Analytics Engine | ML Platform, LLM Gateway |
| **Edu Core** | Образовательный процесс | Content Management, Learning Process, Social Learning, Live Sessions, Schedule | AI Core, Identity |
| **Operations** | Бизнес-операции | Billing, CRM, Support, Documents, Notifications | Identity |
| **Identity & Access** | Пользователи и права | Auth, User, Permission | — (базовый) |

### 5.3. Потоки данных

**Пример: Ученик начинает урок**

```
┌────────┐    ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Client │───▶│   API   │───▶│   Auth   │───▶│ Learning │───▶│ Adaptive │
│        │    │ Gateway │    │ Service  │    │ Process  │    │  Engine  │
└────────┘    └─────────┘    └──────────┘    └──────────┘    └────┬─────┘
                                                                  │
                                                                  ▼
┌────────┐    ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│Response│◀───│ Content │◀───│ Content  │◀───│ Content  │◀───│   User   │
│        │    │ Rendered│    │ Mgmt     │    │ Generator│    │ Service  │
└────────┘    └─────────┘    └──────────┘    └──────────┘    └──────────┘
```

**Последовательность вызовов:**

```
1. POST /api/v1/sessions/start
   └── Headers: Authorization: Bearer <token>

2. API Gateway → Auth Service
   └── Валидация токена → user_id

3. API Gateway → Learning Process
   └── StartSession(student_id)

4. Learning Process → Adaptive Engine
   └── GetNextActivity(student_id, plan_id)

5. Adaptive Engine → User Service
   └── GetCognitiveProfile(student_id)

6. Adaptive Engine → Content Management
   └── GetModule(module_id, personalization)

7. Content Management → Content Generator (если нужна генерация)
   └── GenerateExercise(template, difficulty, context)

8. Response → Client
```

### 5.4. Принципы взаимодействия сервисов

| Принцип | Описание | Реализация |
|---------|----------|------------|
| **Синхронные вызовы** | Для критичных операций с ожиданием ответа | REST/gRPC |
| **Асинхронные события** | Для некритичных уведомлений | Message Queue (RabbitMQ/Kafka) |
| **Eventual consistency** | Не все данные нужны немедленно | Saga pattern |
| **Circuit breaker** | Защита от каскадных отказов | Resilience4j |
| **Retry with backoff** | Повторные попытки при сбоях | Exponential backoff |

---

## 6. Модель данных

### 6.1. Концептуальная схема

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       КОНЦЕПТУАЛЬНАЯ МОДЕЛЬ                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ПОЛЬЗОВАТЕЛИ                                                               │
│                                                                             │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐               │
│  │  User   │────▶│ Student │────▶│Cognitive│     │ Teacher │               │
│  │         │     │         │     │ Profile │     │         │               │
│  └─────────┘     └────┬────┘     └─────────┘     └─────────┘               │
│       │               │                                                     │
│       │               │          ┌─────────┐                               │
│       └───────────────┼─────────▶│ Parent  │                               │
│                       │          └─────────┘                               │
│                       │                                                     │
│  КОНТЕНТ              │                                                     │
│                       │                                                     │
│  ┌─────────┐     ┌────┴────┐     ┌─────────┐     ┌─────────┐               │
│  │ Subject │────▶│ Course  │────▶│ Module  │────▶│ Lesson  │               │
│  └─────────┘     └─────────┘     └────┬────┘     └─────────┘               │
│                                       │                                     │
│                                       │          ┌─────────┐               │
│                                       └─────────▶│Exercise │               │
│                                                  └─────────┘               │
│                                                                             │
│  ПРОГРЕСС                                                                   │
│                                                                             │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                               │
│  │Learning │────▶│Progress │────▶│Assessment│                              │
│  │  Plan   │     │ Record  │     │ Result   │                              │
│  └─────────┘     └─────────┘     └─────────┘                               │
│                                                                             │
│  ИИ-ВЗАИМОДЕЙСТВИЯ                                                         │
│                                                                             │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                               │
│  │ Tutor   │     │Generated│     │Knowledge│                               │
│  │ Session │     │ Content │     │   Gap   │                               │
│  └─────────┘     └─────────┘     └─────────┘                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2. Основные сущности

#### User

```typescript
interface User {
  id: UUID;
  email: string;
  phone?: string;
  password_hash: string;
  role: 'student' | 'parent' | 'teacher' | 'admin';
  status: 'active' | 'inactive' | 'suspended';
  email_verified: boolean;
  created_at: DateTime;
  updated_at: DateTime;
  last_login_at?: DateTime;
  
  settings: {
    language: string;
    timezone: string;
    notifications: NotificationSettings;
  };
}
```

#### Student

```typescript
interface Student {
  id: UUID;
  user_id: UUID;
  
  // Основные данные
  first_name: string;
  last_name: string;
  birth_date: Date;
  grade: number;  // 1-11
  
  // Профиль
  cognitive_profile: CognitiveProfile;
  
  // Текущий план
  learning_plan_id: UUID;
  
  // Связи
  parent_ids: UUID[];
  class_ids: UUID[];
  
  // Статус
  enrolled_at: DateTime;
  status: 'active' | 'paused' | 'graduated' | 'withdrawn';
}
```

#### CognitiveProfile

```typescript
interface CognitiveProfile {
  // Стиль обучения
  learning_style: 'visual' | 'auditory' | 'read_write' | 'kinesthetic' | 'mixed';
  learning_style_scores: {
    visual: number;      // 0-1
    auditory: number;
    read_write: number;
    kinesthetic: number;
  };
  
  // Когнитивные характеристики
  processing_speed: number;         // 0-1
  working_memory_capacity: number;  // 1-10
  attention_span_minutes: number;
  
  // Оптимальные параметры
  optimal_session_duration: number;
  optimal_time_of_day: 'morning' | 'afternoon' | 'evening';
  preferred_formats: ContentFormat[];
  
  // Сильные/слабые стороны
  strength_areas: SubjectArea[];
  challenge_areas: SubjectArea[];
  
  // Мотивация
  motivation_drivers: MotivationType[];
  
  // Метаданные
  last_updated: DateTime;
  confidence_score: number;
}

type MotivationType = 
  | 'achievement'   // достижения
  | 'social'        // признание
  | 'curiosity'     // интерес
  | 'mastery'       // мастерство
  | 'practical';    // практическая польза
```

#### Course

```typescript
interface Course {
  id: UUID;
  subject_id: UUID;
  grade: number;
  
  // Метаданные
  name: string;
  description: string;
  learning_objectives: string[];
  
  // Структура
  units: Unit[];
  
  // Стандарты
  standard_alignments: StandardAlignment[];
  
  // Требования
  prerequisites: UUID[];
  estimated_hours: number;
  
  // Версионирование
  version: string;
  status: 'draft' | 'review' | 'published' | 'archived';
  
  // Локализация
  default_language: string;
  available_languages: string[];
}

interface Unit {
  id: UUID;
  name: string;
  order: number;
  modules: Module[];
  unit_assessment?: Assessment;
}

interface Module {
  id: UUID;
  name: string;
  type: 'lesson' | 'practice' | 'lab' | 'project' | 'assessment';
  
  duration_minutes: number;
  difficulty: number;  // 0-1
  concepts: Concept[];
  
  sections: Section[];
  
  entry_quiz?: Quiz;
  practice_exercises: Exercise[];
  exit_assessment?: Assessment;
  
  ai_config: {
    tutor_context: string;
    generation_templates: Template[];
    explanation_variants: Explanation[];
  };
}
```

#### LearningPlan

```typescript
interface LearningPlan {
  id: UUID;
  student_id: UUID;
  
  target_completion: Date;
  
  planned_modules: PlannedModule[];
  
  pace: 'accelerated' | 'normal' | 'relaxed' | 'remedial';
  weekly_hours_target: number;
  
  adjustments: PlanAdjustment[];
  
  created_at: DateTime;
  updated_at: DateTime;
}

interface PlannedModule {
  module_id: UUID;
  scheduled_date: Date;
  estimated_duration: number;
  priority: 'critical' | 'high' | 'normal' | 'optional';
  status: 'not_started' | 'in_progress' | 'completed' | 'skipped';
  prerequisites_met: boolean;
}
```

#### ProgressRecord

```typescript
interface ProgressRecord {
  id: UUID;
  student_id: UUID;
  module_id: UUID;
  
  status: 'not_started' | 'in_progress' | 'completed' | 'mastered';
  
  attempts: number;
  time_spent: number;  // секунды
  best_score: number;
  mastery_level: number;  // 0-1
  
  sections_completed: UUID[];
  exercises_completed: ExerciseResult[];
  
  started_at?: DateTime;
  completed_at?: DateTime;
  last_activity_at: DateTime;
}
```

### 6.3. Связи между сущностями

| Сущность A | Связь | Сущность B | Кардинальность |
|------------|-------|------------|----------------|
| User | is-a | Student/Parent/Teacher | 1:1 |
| Parent | has | Student | M:N |
| Student | has | CognitiveProfile | 1:1 |
| Student | has | LearningPlan | 1:1 |
| Subject | contains | Course | 1:N |
| Course | contains | Unit | 1:N |
| Unit | contains | Module | 1:N |
| Module | contains | Section | 1:N |
| Module | contains | Exercise | 1:N |
| Student | progresses | Module | M:N |
| Student | submits | Exercise | M:N |
| Student | interacts | AITutor | 1:N |

### 6.4. Распределение по хранилищам

| Хранилище | Данные | Обоснование |
|-----------|--------|-------------|
| **PostgreSQL** | Users, Students, Parents, Teachers, LearningPlans, ProgressRecords, Subscriptions, Payments | Транзакционность, связи, ACID |
| **MongoDB** | Courses, Modules, Lessons, Exercises, CognitiveProfiles, TutorSessions | Гибкая схема, вложенные документы |
| **Redis** | Sessions, Tokens, Cache, Real-time state | Скорость, TTL |
| **Elasticsearch** | Content search, Logs, Analytics | Полнотекстовый поиск |
| **S3/Minio** | Videos, Images, Documents, Exports | Файлы, бинарные данные |

---

## 7. Интеграционная архитектура

### 7.1. Внешние интеграции

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ВНЕШНИЕ ИНТЕГРАЦИИ                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  LLM-ПРОВАЙДЕРЫ              ПЛАТЕЖИ                 КОММУНИКАЦИИ          │
│  ┌───────────────┐          ┌───────────────┐       ┌───────────────┐      │
│  │ OpenAI        │          │ ЮKassa        │       │ SendGrid      │      │
│  │ Anthropic     │          │ Stripe        │       │ Twilio        │      │
│  │ YandexGPT     │          │ СБП           │       │ Firebase      │      │
│  │ GigaChat      │          │ PayPal        │       │ OneSignal     │      │
│  └───────────────┘          └───────────────┘       └───────────────┘      │
│                                                                             │
│  ВИДЕО                       АУТЕНТИФИКАЦИЯ          ГОСУДАРСТВЕННЫЕ       │
│  ┌───────────────┐          ┌───────────────┐       ┌───────────────┐      │
│  │ Jitsi Meet    │          │ Google OAuth  │       │ ФИС ФРДО      │      │
│  │ Kinescope     │          │ VK OAuth      │       │ ЕСИА          │      │
│  │ Cloudflare    │          │ Яндекс OAuth  │       │ Региональные  │      │
│  │ Stream        │          │ Apple Sign In │       │ системы       │      │
│  └───────────────┘          └───────────────┘       └───────────────┘      │
│                                                                             │
│  АНАЛИТИКА                   ПАРТНЁРЫ                                      │
│  ┌───────────────┐          ┌───────────────┐                              │
│  │ Amplitude     │          │ Школы-партнёры│                              │
│  │ Mixpanel      │          │ Издательства  │                              │
│  │ Sentry        │          │ Репетиторы    │                              │
│  │ DataDog       │          │               │                              │
│  └───────────────┘          └───────────────┘                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.2. LLM Gateway

**Назначение:** Единая точка интеграции со всеми LLM-провайдерами.

**Конфигурация провайдеров:**

| Сценарий | Основной | Fallback 1 | Fallback 2 | Max Latency |
|----------|----------|------------|------------|-------------|
| ИИ-тьютор (диалог) | Anthropic Claude | OpenAI GPT-4 | YandexGPT | 2 сек |
| Генерация задач | OpenAI GPT-4 | Anthropic Claude | — | 5 сек |
| Проверка эссе | Anthropic Claude | OpenAI GPT-4 | — | 10 сек |
| Генерация объяснений | OpenAI GPT-4 | YandexGPT | — | 3 сек |

**Функции:**

```typescript
interface LLMGateway {
  // Основной метод
  complete(request: LLMRequest): Promise<LLMResponse>;
  
  // Стриминг для диалогов
  stream(request: LLMRequest): AsyncIterator<LLMChunk>;
  
  // Кэширование
  cache: {
    get(key: string): Promise<LLMResponse | null>;
    set(key: string, response: LLMResponse, ttl: number): Promise<void>;
  };
  
  // Мониторинг
  metrics: {
    latency: Histogram;
    tokens_used: Counter;
    errors: Counter;
    cache_hits: Counter;
  };
}

interface LLMRequest {
  scenario: string;
  messages: Message[];
  parameters: {
    temperature?: number;
    max_tokens?: number;
    stop_sequences?: string[];
  };
  context?: {
    student_id?: string;
    topic?: string;
  };
  options?: {
    cache_ttl?: number;
    priority?: 'high' | 'normal' | 'low';
  };
}
```

### 7.3. Основные API эндпоинты

#### Аутентификация

```yaml
/api/v1/auth:
  /register:
    POST:
      body: { email, password, role }
      response: { user_id, tokens }
      
  /login:
    POST:
      body: { email, password }
      response: { access_token, refresh_token, user }
      
  /refresh:
    POST:
      body: { refresh_token }
      response: { access_token, refresh_token }
      
  /logout:
    POST:
      headers: Authorization
      response: { success }
```

#### Обучение

```yaml
/api/v1/learning:
  /sessions:
    POST:
      description: Начать учебную сессию
      body: { student_id }
      response: { session_id, activities }
      
    /{session_id}:
      GET:
        description: Состояние сессии
        response: { session, current_activity, progress }
        
      /complete:
        POST:
          description: Завершить сессию
          response: { summary, progress_update }
          
  /activities/{activity_id}:
    GET:
      description: Получить контент
      response: { content, exercises, ai_context }
      
    /submit:
      POST:
        description: Отправить ответ
        body: { answer, time_spent }
        response: { result, feedback, next_action }
```

#### ИИ-тьютор

```yaml
/api/v1/tutor:
  /chat:
    POST:
      description: Сообщение тьютору
      body: { student_id, message, context }
      response: { response, suggestions?, escalate? }
      
  /chat/stream:
    POST:
      description: Стриминг ответа
      response: Server-Sent Events
      
  /explain:
    POST:
      description: Объяснение концепции
      body: { concept_id, student_id, previous_attempts? }
      response: { explanation, examples, check_question }
```

#### Прогресс

```yaml
/api/v1/progress:
  /students/{student_id}:
    GET:
      description: Общий прогресс
      response: { overall, by_subject, achievements }
      
    /subjects/{subject_id}:
      GET:
        description: Прогресс по предмету
        response: { modules, mastery, predictions }
        
  /reports:
    /weekly:
      GET:
        description: Недельный отчёт
        query: { student_id, week? }
        response: { summary, highlights, concerns }
```

---

## 8. Архитектура безопасности

### 8.1. Модель угроз

| Угроза | Вероятность | Влияние | Меры защиты |
|--------|-------------|---------|-------------|
| Утечка данных детей | Средняя | Критическое | Шифрование, access control, аудит |
| Несанкционированный доступ | Высокая | Высокое | MFA, rate limiting, anomaly detection |
| DDoS-атака | Средняя | Высокое | CDN, WAF, auto-scaling |
| Вредоносный контент | Средняя | Высокое | Модерация, фильтрация |
| Компрометация API-ключей | Низкая | Критическое | Rotation, secrets management |
| Буллинг в чатах | Высокая | Среднее | AI-модерация, reporting |

### 8.2. Уровни защиты

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         УРОВНИ БЕЗОПАСНОСТИ                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  УРОВЕНЬ 1: ПЕРИМЕТР                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ • WAF (Web Application Firewall)                                    │   │
│  │ • DDoS Protection                                                   │   │
│  │ • Rate Limiting                                                     │   │
│  │ • Геоблокировка                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  УРОВЕНЬ 2: АУТЕНТИФИКАЦИЯ                                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ • JWT (15 мин expiry)                                               │   │
│  │ • Refresh token rotation                                            │   │
│  │ • MFA для критичных операций                                        │   │
│  │ • RBAC + ABAC                                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  УРОВЕНЬ 3: ЗАЩИТА ДАННЫХ                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ • Шифрование at rest (AES-256)                                      │   │
│  │ • Шифрование in transit (TLS 1.3)                                   │   │
│  │ • PII в отдельном контуре                                           │   │
│  │ • Маскирование в логах                                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  УРОВЕНЬ 4: ЗАЩИТА ДЕТЕЙ                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ • AI-модерация контента                                             │   │
│  │ • Фильтрация общения                                                │   │
│  │ • Родительский контроль                                             │   │
│  │ • Аудит действий учителей                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  УРОВЕНЬ 5: COMPLIANCE                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ • 152-ФЗ: локализация в РФ                                          │   │
│  │ • GDPR: право на удаление                                           │   │
│  │ • COPPA: требования для детей                                       │   │
│  │ • Регулярный pentest                                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.3. Матрица доступа

| Ресурс | Ученик | Родитель | Учитель | Методист | Админ |
|--------|:------:|:--------:|:-------:|:--------:|:-----:|
| Свой профиль | RW | R* | RW | RW | RW |
| Учебный контент | R | R | R | RW | RW |
| Свой прогресс | R | R* | R** | — | R |
| Оценки | R | R* | RW** | — | R |
| Чат с тьютором | RW | R* | — | — | R |
| Групповые чаты | RW | R | RW | — | RW |
| Платёжная информация | — | RW | — | — | R |
| Системные настройки | — | — | — | — | RW |
| Аналитика | — | — | R** | R | RW |

R — чтение, W — запись, * — только своих детей, ** — только своих учеников

### 8.4. Аутентификация и авторизация

**Схема токенов:**

```
┌────────────────────────────────────────────────────────────────┐
│                    СХЕМА АУТЕНТИФИКАЦИИ                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  LOGIN                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│  │  Client  │───▶│   Auth   │───▶│   User   │                 │
│  │          │    │ Service  │    │   DB     │                 │
│  └──────────┘    └────┬─────┘    └──────────┘                 │
│       ▲               │                                        │
│       │               ▼                                        │
│       │         ┌──────────┐                                   │
│       └─────────│  Tokens  │                                   │
│                 │ access:  │                                   │
│                 │  15 min  │                                   │
│                 │ refresh: │                                   │
│                 │  7 days  │                                   │
│                 └──────────┘                                   │
│                                                                │
│  API REQUEST                                                   │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│  │  Client  │───▶│   API    │───▶│  Auth    │                 │
│  │ +Bearer  │    │ Gateway  │    │ Verify   │                 │
│  └──────────┘    └────┬─────┘    └────┬─────┘                 │
│                       │               │                        │
│                       │    ┌──────────┘                        │
│                       │    │ user_id, role, permissions        │
│                       ▼    ▼                                   │
│                 ┌──────────┐                                   │
│                 │ Service  │                                   │
│                 └──────────┘                                   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**JWT Payload:**

```json
{
  "sub": "user_uuid",
  "role": "student",
  "permissions": ["read:content", "write:progress"],
  "student_id": "student_uuid",
  "grade": 5,
  "iat": 1700000000,
  "exp": 1700000900
}
```

---

*Следующий раздел: [03_AI_CORE.md](03_AI_CORE.md) — Ядро ИИ-системы*
