# Часть VI: Операционные системы (Operations)

## Содержание

- [12.1. Обзор домена](#121-обзор-домена)
- [12.2. Billing Service](#122-billing-service)
- [12.3. CRM Service](#123-crm-service)
- [12.4. Support Service](#124-support-service)
- [12.5. Document Service](#125-document-service)
- [12.6. Notification Service](#126-notification-service)

---

## 12.1. Обзор домена

### Назначение

Operations обеспечивает все бизнес-процессы платформы: управление подписками, работу с клиентами, поддержку пользователей, документооборот и коммуникации.

### Структура домена

```
OPERATIONS
│
├── BILLING SERVICE
│   ├── Subscription Manager
│   ├── Payment Processor
│   ├── Invoice Generator
│   ├── Promo Engine
│   └── Revenue Analytics
│
├── CRM SERVICE
│   ├── Lead Manager
│   ├── Customer Profile
│   ├── Communication History
│   ├── Segmentation Engine
│   └── Campaign Manager
│
├── SUPPORT SERVICE
│   ├── Ticket System
│   ├── AI Chatbot
│   ├── Knowledge Base
│   ├── Escalation Manager
│   └── Satisfaction Tracker
│
├── DOCUMENT SERVICE
│   ├── Contract Generator
│   ├── Certificate Manager
│   ├── Report Generator
│   ├── Digital Signature
│   └── Archive
│
└── NOTIFICATION SERVICE
    ├── Email Sender
    ├── Push Manager
    ├── SMS Gateway
    ├── In-App Notifications
    └── Preference Manager
```

---

## 12.2. Billing Service

### 12.2.1. Модель подписки

**Тарифные планы:**

| План | Цена/мес | Описание | Включено |
|------|----------|----------|----------|
| **Базовый** | 4 990 ₽ | Самостоятельное обучение | AI-тьютор, адаптивная программа, автопроверка |
| **Стандарт** | 9 990 ₽ | С групповыми занятиями | + 4 групповых занятия/мес, психолог, расширенная аналитика |
| **Премиум** | 19 990 ₽ | Полное сопровождение | + индивидуальный куратор, неограниченные консультации, подготовка к олимпиадам |
| **Семейный** | 14 990 ₽ | Для 2-3 детей | Стандарт для всех детей |

**Скидки:**

| Тип | Размер | Условие |
|-----|--------|---------|
| Годовая оплата | 20% | Оплата за 12 месяцев |
| Полугодовая | 10% | Оплата за 6 месяцев |
| Многодетные | 30% | От 3 детей |
| Реферальная | 15% | За приведённого клиента |

### 12.2.2. Модели данных

```typescript
interface Subscription {
  id: UUID;
  customer_id: UUID;
  
  plan: {
    id: string;
    name: string;
    tier: 'basic' | 'standard' | 'premium' | 'family';
    price: Money;
    billing_period: 'monthly' | 'quarterly' | 'yearly';
  };
  
  students: UUID[];  // связанные ученики
  
  status: 'trial' | 'active' | 'past_due' | 'cancelled' | 'expired';
  
  billing: {
    next_billing_date: Date;
    payment_method_id: UUID;
    auto_renew: boolean;
  };
  
  discounts: Discount[];
  
  created_at: DateTime;
  cancelled_at?: DateTime;
  expires_at?: DateTime;
}

interface Payment {
  id: UUID;
  subscription_id: UUID;
  customer_id: UUID;
  
  amount: Money;
  currency: string;
  
  status: 'pending' | 'processing' | 'succeeded' | 'failed' | 'refunded';
  
  provider: 'yookassa' | 'stripe' | 'sbp';
  provider_payment_id: string;
  
  invoice_id?: UUID;
  
  created_at: DateTime;
  completed_at?: DateTime;
  
  metadata: {
    period_start: Date;
    period_end: Date;
    description: string;
  };
}

interface Invoice {
  id: UUID;
  number: string;  // "INV-2024-000123"
  
  customer_id: UUID;
  subscription_id: UUID;
  
  items: InvoiceItem[];
  subtotal: Money;
  discounts: AppliedDiscount[];
  tax: Money;
  total: Money;
  
  status: 'draft' | 'sent' | 'paid' | 'void';
  
  due_date: Date;
  paid_at?: DateTime;
  
  pdf_url?: string;
}
```

### 12.2.3. Платёжные интеграции

```
┌─────────────────────────────────────────────────────────────────┐
│                    ПЛАТЁЖНАЯ АРХИТЕКТУРА                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐                                                   │
│  │  Client  │                                                   │
│  └────┬─────┘                                                   │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────────────┐  │
│  │ Billing  │───▶│ Payment  │───▶│ Payment Providers        │  │
│  │ Service  │    │ Gateway  │    │ ┌────────┐ ┌────────┐    │  │
│  └──────────┘    └──────────┘    │ │ЮKassa  │ │ Stripe │    │  │
│       │                          │ └────────┘ └────────┘    │  │
│       │                          │ ┌────────┐ ┌────────┐    │  │
│       ▼                          │ │  СБП   │ │ PayPal │    │  │
│  ┌──────────┐                    │ └────────┘ └────────┘    │  │
│  │ Webhook  │◀───────────────────│                          │  │
│  │ Handler  │                    └──────────────────────────┘  │
│  └──────────┘                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Конфигурация провайдеров:**

| Провайдер | Регион | Методы | Комиссия |
|-----------|--------|--------|----------|
| ЮKassa | РФ | Карты, ЮMoney, СБП | 2.8% |
| Stripe | Global | Cards, Apple Pay, Google Pay | 2.9% + $0.30 |
| СБП | РФ | Банковские переводы | 0.4% |

### 12.2.4. Процессы биллинга

**Автоматическое продление:**

```python
async def process_subscription_renewal(subscription_id: str):
    """
    Ежедневный процесс продления подписок.
    """
    subscription = await get_subscription(subscription_id)
    
    if not subscription.auto_renew:
        return
    
    if subscription.next_billing_date > today():
        return
    
    # Попытка списания
    payment_result = await charge_payment(
        customer_id=subscription.customer_id,
        amount=subscription.plan.price,
        payment_method=subscription.billing.payment_method_id
    )
    
    if payment_result.success:
        # Успешное продление
        await extend_subscription(subscription, subscription.plan.billing_period)
        await send_notification(
            subscription.customer_id,
            template='payment_success',
            data={'amount': subscription.plan.price}
        )
    else:
        # Неуспешная оплата
        subscription.status = 'past_due'
        await save_subscription(subscription)
        
        # Уведомление и retry
        await send_notification(
            subscription.customer_id,
            template='payment_failed',
            data={'reason': payment_result.error}
        )
        
        # Планируем повторные попытки
        await schedule_retry(subscription_id, delays=[1, 3, 7])  # дни
```

**Возвраты:**

```python
async def process_refund(
    payment_id: str,
    amount: Money,
    reason: str,
    initiated_by: str
) -> RefundResult:
    """
    Обработка возврата.
    """
    payment = await get_payment(payment_id)
    
    # Валидация
    if payment.status != 'succeeded':
        raise InvalidRefundError("Payment not successful")
    
    if amount > payment.amount:
        raise InvalidRefundError("Refund exceeds payment")
    
    days_since = (now() - payment.completed_at).days
    if days_since > 30 and not is_admin(initiated_by):
        raise InvalidRefundError("Refund period expired")
    
    # Запрос к провайдеру
    provider_result = await payment_provider.refund(
        payment.provider_payment_id,
        amount
    )
    
    if provider_result.success:
        # Обновление статуса
        if amount == payment.amount:
            payment.status = 'refunded'
        else:
            payment.partial_refund = amount
        
        await save_payment(payment)
        
        # Корректировка подписки при полном возврате
        if amount == payment.amount:
            await cancel_subscription(payment.subscription_id)
        
        # Уведомления
        await send_notification(
            payment.customer_id,
            template='refund_processed',
            data={'amount': amount}
        )
    
    return RefundResult(success=provider_result.success)
```

### 12.2.5. Промокоды

```typescript
interface PromoCode {
  id: UUID;
  code: string;  // "SUMMER2024"
  
  type: 'percentage' | 'fixed' | 'trial_extension';
  value: number;  // 20 для 20% или 1000 для 1000₽
  
  constraints: {
    min_amount?: Money;
    max_discount?: Money;
    plans?: string[];  // применимо к планам
    first_payment_only?: boolean;
    new_customers_only?: boolean;
  };
  
  limits: {
    total_uses?: number;
    uses_per_customer?: number;
    valid_from: DateTime;
    valid_until: DateTime;
  };
  
  stats: {
    total_uses: number;
    total_discount_given: Money;
  };
  
  status: 'active' | 'expired' | 'depleted' | 'disabled';
}
```

---

## 12.3. CRM Service

### 12.3.1. Воронка продаж

```
┌─────────────────────────────────────────────────────────────────┐
│                      ВОРОНКА ПРОДАЖ                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ЭТАП 1: ПРИВЛЕЧЕНИЕ                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Visitor → Lead                                         │   │
│  │  Источники: SEO, Ads, Referral, Content                 │   │
│  │  Конверсия: ~5%                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          ↓                                      │
│  ЭТАП 2: РЕГИСТРАЦИЯ                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Lead → Registered                                      │   │
│  │  Действия: Регистрация, email подтверждение             │   │
│  │  Конверсия: ~30%                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          ↓                                      │
│  ЭТАП 3: ПРОБНЫЙ ПЕРИОД                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Registered → Trial                                     │   │
│  │  Действия: Диагностика, первые уроки                    │   │
│  │  Длительность: 7-14 дней                                │   │
│  │  Конверсия в платящих: ~25%                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          ↓                                      │
│  ЭТАП 4: АКТИВАЦИЯ                                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Trial → Paying Customer                                │   │
│  │  Действия: Выбор плана, оплата                          │   │
│  │  Средний чек: 8 500 ₽/мес                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          ↓                                      │
│  ЭТАП 5: УДЕРЖАНИЕ                                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Customer → Loyal Customer                              │   │
│  │  Цель: LTV > 12 месяцев                                 │   │
│  │  Retention: 80%                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 12.3.2. Модель лида

```typescript
interface Lead {
  id: UUID;
  
  // Контактные данные
  contact: {
    email: string;
    phone?: string;
    name?: string;
  };
  
  // Источник
  source: {
    channel: 'organic' | 'paid' | 'referral' | 'direct';
    campaign?: string;
    referrer_id?: UUID;
    landing_page?: string;
    utm: {
      source?: string;
      medium?: string;
      campaign?: string;
      content?: string;
    };
  };
  
  // Этап воронки
  stage: 'new' | 'contacted' | 'qualified' | 'trial' | 'negotiation' | 'won' | 'lost';
  
  // Квалификация
  qualification: {
    children_count?: number;
    children_grades?: number[];
    current_school?: string;
    pain_points?: string[];
    budget?: string;
    decision_timeline?: string;
  };
  
  // Взаимодействия
  interactions: Interaction[];
  
  // Назначение
  assigned_to?: UUID;
  
  // Скоринг
  score: number;  // 0-100
  
  // Метаданные
  created_at: DateTime;
  updated_at: DateTime;
  converted_at?: DateTime;
  lost_reason?: string;
}

interface Interaction {
  id: UUID;
  lead_id: UUID;
  
  type: 'email' | 'call' | 'meeting' | 'chat' | 'form' | 'page_view';
  direction: 'inbound' | 'outbound';
  
  content: {
    subject?: string;
    body?: string;
    duration?: number;
    recording_url?: string;
  };
  
  performed_by: UUID | 'system';
  created_at: DateTime;
}
```

### 12.3.3. Автоматизация CRM

**Автоматические триггеры:**

| Триггер | Условие | Действие |
|---------|---------|----------|
| Welcome | Новый лид | Email приветствие |
| Follow-up | Нет активности 2 дня | Напоминание |
| Trial ending | 2 дня до конца триала | Email + звонок |
| Abandoned cart | Начал оплату, не завершил | Email + промокод |
| Win-back | Отменил 30+ дней назад | Специальное предложение |
| Referral ask | Клиент 3+ месяцев, высокий NPS | Запрос рекомендации |

**Lead Scoring:**

```python
def calculate_lead_score(lead: Lead) -> int:
    """
    Расчёт скоринга лида (0-100).
    """
    score = 0
    
    # Демографические факторы (max 30)
    if lead.qualification.children_count:
        score += min(lead.qualification.children_count * 5, 15)
    if lead.contact.phone:
        score += 10
    if lead.qualification.budget in ['ready', 'high']:
        score += 5
    
    # Поведенческие факторы (max 40)
    recent_interactions = get_recent_interactions(lead.id, days=7)
    score += min(len(recent_interactions) * 3, 15)
    
    if has_visited_pricing(lead):
        score += 10
    if has_started_trial(lead):
        score += 15
    
    # Вовлечённость (max 30)
    if lead.stage in ['qualified', 'trial', 'negotiation']:
        score += 15
    
    email_engagement = get_email_engagement(lead.email)
    score += min(email_engagement * 10, 15)
    
    return min(score, 100)
```

---

## 12.4. Support Service

### 12.4.1. Система тикетов

```typescript
interface Ticket {
  id: UUID;
  number: string;  // "SUP-2024-001234"
  
  // Заявитель
  requester: {
    user_id: UUID;
    role: 'student' | 'parent' | 'teacher';
    email: string;
    name: string;
  };
  
  // Классификация
  category: 'technical' | 'billing' | 'content' | 'account' | 'other';
  subcategory?: string;
  priority: 'low' | 'normal' | 'high' | 'urgent';
  
  // Содержание
  subject: string;
  description: string;
  attachments: Attachment[];
  
  // Статус
  status: 'new' | 'open' | 'pending' | 'on_hold' | 'solved' | 'closed';
  
  // Назначение
  assigned_to?: UUID;
  group?: string;
  
  // SLA
  sla: {
    first_response_due: DateTime;
    resolution_due: DateTime;
    first_response_at?: DateTime;
    resolved_at?: DateTime;
    breached: boolean;
  };
  
  // История
  messages: TicketMessage[];
  events: TicketEvent[];
  
  // Связи
  related_tickets?: UUID[];
  
  // Метаданные
  tags: string[];
  custom_fields: Record<string, any>;
  
  created_at: DateTime;
  updated_at: DateTime;
}
```

### 12.4.2. SLA матрица

| Приоритет | Первый ответ | Решение | Эскалация |
|-----------|--------------|---------|-----------|
| Urgent | 1 час | 4 часа | 2 часа |
| High | 4 часа | 24 часа | 8 часов |
| Normal | 8 часов | 48 часов | 24 часа |
| Low | 24 часа | 5 дней | 48 часов |

### 12.4.3. AI Chatbot

**Архитектура:**

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│    User      │───▶│   Chatbot    │───▶│  Knowledge   │
│   Message    │    │   Router     │    │    Base      │
└──────────────┘    └──────┬───────┘    └──────────────┘
                          │
            ┌─────────────┼─────────────┐
            ▼             ▼             ▼
     ┌──────────┐  ┌──────────┐  ┌──────────┐
     │   FAQ    │  │   LLM    │  │  Ticket  │
     │  Match   │  │  Answer  │  │  Create  │
     └──────────┘  └──────────┘  └──────────┘
            │             │             │
            └─────────────┼─────────────┘
                          ▼
                   ┌──────────────┐
                   │   Response   │
                   └──────────────┘
```

**Возможности чат-бота:**

| Категория | Примеры вопросов | Автоматизация |
|-----------|------------------|---------------|
| FAQ | "Как сменить пароль?" | 100% |
| Биллинг | "Когда следующий платёж?" | 100% |
| Статус | "Какой прогресс у ребёнка?" | 100% |
| Технические | "Не загружается видео" | 80% → тикет |
| Контент | "Ошибка в задании" | 50% → тикет |
| Сложные | "Хочу вернуть деньги" | 0% → оператор |

**Процесс обработки:**

```python
async def process_chat_message(message: str, user_id: str) -> ChatResponse:
    """
    Обработка сообщения в чате поддержки.
    """
    # 1. Классификация намерения
    intent = await classify_intent(message)
    
    # 2. Извлечение сущностей
    entities = await extract_entities(message)
    
    # 3. Поиск в FAQ
    faq_match = await search_faq(message)
    if faq_match and faq_match.confidence > 0.85:
        return ChatResponse(
            text=faq_match.answer,
            source='faq',
            follow_up=faq_match.related_questions
        )
    
    # 4. Проверка на простые запросы
    if intent in ['check_balance', 'check_progress', 'check_schedule']:
        data = await fetch_user_data(user_id, intent)
        return ChatResponse(
            text=format_response(intent, data),
            source='data'
        )
    
    # 5. Попытка LLM-ответа
    if intent not in ['refund', 'complaint', 'urgent_technical']:
        llm_response = await generate_llm_response(
            message=message,
            context=get_user_context(user_id),
            knowledge_base=get_relevant_docs(message)
        )
        
        if llm_response.confidence > 0.7:
            return ChatResponse(
                text=llm_response.text,
                source='llm',
                offer_human=True
            )
    
    # 6. Эскалация на оператора
    ticket = await create_ticket_from_chat(user_id, message)
    return ChatResponse(
        text="Я передам ваш вопрос специалисту. Он ответит в течение часа.",
        source='escalation',
        ticket_id=ticket.id
    )
```

### 12.4.4. Satisfaction Tracking

**CSAT опросы:**

| Триггер | Вопрос | Шкала |
|---------|--------|-------|
| После тикета | "Насколько вы удовлетворены решением?" | 1-5 |
| После чата | "Помог ли вам чат-бот?" | 👍 / 👎 |
| Ежемесячно | NPS: "Порекомендуете ли вы нас?" | 0-10 |

---

## 12.5. Document Service

### 12.5.1. Типы документов

| Документ | Генерация | Подпись | Хранение |
|----------|-----------|---------|----------|
| Договор | По шаблону | ЭЦП / SMS | 5 лет |
| Счёт | Автоматически | Не требуется | 5 лет |
| Акт | Автоматически | ЭЦП | 5 лет |
| Справка | По запросу | Печать школы | 3 года |
| Аттестат | По окончании | Школа-партнёр | Постоянно |
| Табель | Ежечетвертно | Печать | 5 лет |

### 12.5.2. Генератор документов

```typescript
interface DocumentTemplate {
  id: UUID;
  type: DocumentType;
  name: string;
  
  // Шаблон
  template: {
    format: 'docx' | 'pdf' | 'html';
    content: string;  // с плейсхолдерами {{variable}}
    styles?: object;
  };
  
  // Переменные
  variables: Variable[];
  
  // Правила
  rules: {
    required_approvals?: string[];
    auto_send?: boolean;
    archive_after_days?: number;
  };
  
  version: string;
  status: 'draft' | 'active' | 'deprecated';
}

interface GeneratedDocument {
  id: UUID;
  template_id: UUID;
  
  // Связи
  customer_id?: UUID;
  student_id?: UUID;
  subscription_id?: UUID;
  
  // Содержимое
  data: Record<string, any>;  // подставленные значения
  
  // Файл
  file: {
    url: string;
    format: string;
    size: number;
    hash: string;
  };
  
  // Подпись
  signature?: {
    type: 'digital' | 'sms' | 'manual';
    signed_at: DateTime;
    signed_by: string;
    certificate?: string;
  };
  
  // Доставка
  delivery: {
    sent_via: ('email' | 'app' | 'post')[];
    sent_at?: DateTime;
  };
  
  created_at: DateTime;
  expires_at?: DateTime;
}
```

### 12.5.3. Интеграция с ФИС ФРДО

```python
async def submit_to_fis_frdo(certificate: Certificate):
    """
    Отправка данных об аттестате в ФИС ФРДО.
    """
    # Формирование данных
    data = {
        "documentType": "ATTESTAT",
        "series": certificate.series,
        "number": certificate.number,
        "issueDate": certificate.issue_date.isoformat(),
        "student": {
            "lastName": certificate.student.last_name,
            "firstName": certificate.student.first_name,
            "middleName": certificate.student.middle_name,
            "birthDate": certificate.student.birth_date.isoformat(),
            "snils": certificate.student.snils
        },
        "organization": {
            "inn": PARTNER_SCHOOL_INN,
            "ogrn": PARTNER_SCHOOL_OGRN
        },
        "educationLevel": "BASIC_GENERAL" if certificate.grade == 9 else "SECONDARY_GENERAL",
        "grades": format_grades(certificate.grades)
    }
    
    # Подпись ЭЦП
    signed_data = sign_with_certificate(data, SCHOOL_CERTIFICATE)
    
    # Отправка
    response = await fis_frdo_client.submit(signed_data)
    
    if response.success:
        certificate.frdo_id = response.registration_id
        await save_certificate(certificate)
    else:
        await alert_admin(f"FRDO submission failed: {response.error}")
        raise FRDOSubmissionError(response.error)
```

---

## 12.6. Notification Service

### 12.6.1. Каналы уведомлений

| Канал | Провайдер | Типы сообщений | Лимиты |
|-------|-----------|----------------|--------|
| Email | SendGrid | Все | 100K/день |
| Push (iOS) | APNs | Важные, напоминания | Без лимита |
| Push (Android) | FCM | Важные, напоминания | Без лимита |
| Push (Web) | OneSignal | Важные | 10K/день |
| SMS | Twilio | Критичные, OTP | 10K/день |
| In-App | Собственный | Все | Без лимита |

### 12.6.2. Шаблоны уведомлений

```typescript
interface NotificationTemplate {
  id: string;  // "payment_success"
  
  channels: {
    email?: {
      subject: string;
      body_html: string;
      body_text: string;
    };
    push?: {
      title: string;
      body: string;
      image?: string;
      action?: string;
    };
    sms?: {
      body: string;  // до 160 символов
    };
    in_app?: {
      title: string;
      body: string;
      type: 'info' | 'success' | 'warning' | 'error';
      action?: { label: string; url: string };
    };
  };
  
  // Переменные
  variables: string[];  // ["amount", "next_date"]
  
  // Настройки
  settings: {
    priority: 'low' | 'normal' | 'high' | 'critical';
    category: string;
    default_channels: string[];
    allow_unsubscribe: boolean;
  };
}
```

### 12.6.3. Матрица уведомлений

| Событие | Email | Push | SMS | In-App |
|---------|:-----:|:----:|:---:|:------:|
| Регистрация | ✓ | | | ✓ |
| Подтверждение email | ✓ | | | |
| Оплата успешна | ✓ | ✓ | | ✓ |
| Оплата не прошла | ✓ | ✓ | ✓ | ✓ |
| Новое достижение | | ✓ | | ✓ |
| Напоминание об уроке | | ✓ | | ✓ |
| Новое ДЗ | | ✓ | | ✓ |
| Оценка за контрольную | ✓ | ✓ | | ✓ |
| Пробел обнаружен | ✓ | ✓ | | ✓ |
| Неактивность 3 дня | | ✓ | | |
| Неактивность 7 дней | ✓ | ✓ | ✓ | |
| Окончание триала | ✓ | ✓ | ✓ | ✓ |
| Живое занятие через 1 час | | ✓ | | ✓ |
| Ответ от учителя | | ✓ | | ✓ |

### 12.6.4. Preference Management

```typescript
interface NotificationPreferences {
  user_id: UUID;
  
  // Глобальные настройки
  global: {
    quiet_hours: { start: Time; end: Time };
    timezone: string;
  };
  
  // По каналам
  channels: {
    email: boolean;
    push: boolean;
    sms: boolean;
  };
  
  // По категориям
  categories: {
    marketing: boolean;
    achievements: boolean;
    reminders: boolean;
    reports: boolean;
    billing: boolean;  // нельзя отключить
    security: boolean;  // нельзя отключить
  };
  
  // Частота
  digest: {
    enabled: boolean;
    frequency: 'daily' | 'weekly';
    day_of_week?: number;
    time: Time;
  };
}
```

### 12.6.5. Процесс отправки

```python
async def send_notification(
    user_id: str,
    template: str,
    data: dict,
    channels: list[str] = None
) -> NotificationResult:
    """
    Отправка уведомления пользователю.
    """
    # 1. Получение шаблона
    template_obj = await get_template(template)
    
    # 2. Получение предпочтений
    prefs = await get_preferences(user_id)
    
    # 3. Определение каналов
    if channels is None:
        channels = template_obj.settings.default_channels
    
    # Фильтр по предпочтениям
    channels = [c for c in channels if is_channel_enabled(prefs, c, template_obj)]
    
    # 4. Проверка quiet hours
    if is_quiet_hours(prefs) and template_obj.settings.priority != 'critical':
        await schedule_for_later(user_id, template, data, channels)
        return NotificationResult(scheduled=True)
    
    # 5. Рендеринг и отправка
    results = {}
    for channel in channels:
        rendered = render_template(template_obj.channels[channel], data)
        
        if channel == 'email':
            results['email'] = await send_email(user_id, rendered)
        elif channel == 'push':
            results['push'] = await send_push(user_id, rendered)
        elif channel == 'sms':
            results['sms'] = await send_sms(user_id, rendered)
        elif channel == 'in_app':
            results['in_app'] = await create_in_app(user_id, rendered)
    
    # 6. Логирование
    await log_notification(user_id, template, channels, results)
    
    return NotificationResult(sent=results)
```

---

*Следующий раздел: [07_INFRASTRUCTURE.md](07_INFRASTRUCTURE.md)*
