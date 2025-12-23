# Часть IV: Образовательное ядро (Edu Core)

## Содержание

- [10.1. Обзор домена](#101-обзор-домена)
- [10.2. Content Management System](#102-content-management-system)
- [10.3. Learning Process Management](#103-learning-process-management)
- [10.4. Social Learning](#104-social-learning)
- [10.5. Live Learning](#105-live-learning)

---

## 10.1. Обзор домена

### Назначение

Edu Core управляет всем образовательным процессом: контентом, учебными сессиями, прогрессом, социальным взаимодействием и живыми занятиями.

### Структура

```
EDU CORE
├── CONTENT MANAGEMENT SYSTEM
│   ├── Course Manager
│   ├── Module Editor
│   ├── Asset Library
│   └── Standard Mapper
│
├── LEARNING PROCESS MANAGEMENT
│   ├── Session Controller
│   ├── Progress Tracker
│   ├── Assignment Manager
│   └── Grade Book
│
├── SOCIAL LEARNING
│   ├── Study Groups
│   ├── Project Workspace
│   ├── Discussion Forum
│   └── Peer Mentoring
│
└── LIVE LEARNING
    ├── Video Conference
    ├── Interactive Whiteboard
    ├── Recording & Archive
    └── Teacher Scheduler
```

---

## 10.2. Content Management System

### 10.2.1. Course Manager

**Структура курса:**

```
Subject (Предмет)
└── Course (Курс для класса)
    └── Unit (Раздел)
        └── Module (Модуль)
            ├── Lesson (Урок)
            ├── Exercise (Упражнение)
            └── Assessment (Проверка)
```

**Модель курса:**

```typescript
interface Course {
  id: UUID;
  subject_id: UUID;
  grade: number;  // 1-11
  
  name: string;
  description: string;
  learning_objectives: string[];
  
  units: Unit[];
  
  standard_alignments: StandardAlignment[];
  prerequisites: UUID[];
  estimated_hours: number;
  
  version: string;
  status: 'draft' | 'review' | 'published' | 'archived';
  
  localization: {
    default_language: string;
    available: string[];
  };
}

interface Unit {
  id: UUID;
  name: string;
  order: number;
  modules: Module[];
  assessment?: UnitAssessment;
}

interface Module {
  id: UUID;
  name: string;
  type: 'lesson' | 'practice' | 'lab' | 'project' | 'assessment';
  
  duration_minutes: number;
  difficulty: number;  // 0-1
  concepts: Concept[];
  
  sections: Section[];
  exercises: Exercise[];
  
  ai_config: {
    tutor_context: string;
    generation_templates: Template[];
    explanation_variants: Explanation[];
  };
}
```

### 10.2.2. Standard Mapper

**Соответствие ФГОС:**

```typescript
interface StandardAlignment {
  standard_id: string;     // "ФГОС-2021-МАТ-5-3.1"
  standard_name: string;
  coverage: number;        // 0-1
  modules: UUID[];
}

interface Standard {
  id: string;
  country: string;
  subject: string;
  grade: number;
  
  requirements: Requirement[];
}

interface Requirement {
  id: string;
  description: string;
  knowledge: string[];     // что должен знать
  skills: string[];        // что должен уметь
  assessment_criteria: string[];
}
```

**Маппинг для разных стран:**

| Страна | Стандарт | Классы | Примечание |
|--------|----------|--------|------------|
| Россия | ФГОС | 1-11 | Основной |
| США | Common Core | K-12 | Для экспансии |
| UK | National Curriculum | Key Stages | Для экспансии |
| Казахстан | ГОС РК | 1-11 | СНГ |

### 10.2.3. Asset Library

**Типы контента:**

| Тип | Форматы | Хранение | Доставка |
|-----|---------|----------|----------|
| Видео | MP4, WebM | S3 | CDN + Adaptive |
| Изображения | PNG, SVG, WebP | S3 | CDN |
| Интерактивы | HTML5, React | S3 | CDN |
| Документы | PDF, DOCX | S3 | Direct |
| 3D модели | GLTF, GLB | S3 | CDN |
| Аудио | MP3, OGG | S3 | CDN |

**Версионирование:**

```typescript
interface Asset {
  id: UUID;
  type: AssetType;
  
  versions: AssetVersion[];
  current_version: string;
  
  metadata: {
    title: string;
    description: string;
    tags: string[];
    duration?: number;
    dimensions?: { width: number; height: number };
  };
  
  usage: {
    modules: UUID[];
    view_count: number;
    avg_engagement: number;
  };
}
```

---

## 10.3. Learning Process Management

### 10.3.1. Session Controller

**Модель сессии:**

```typescript
interface LearningSession {
  id: UUID;
  student_id: UUID;
  started_at: DateTime;
  ended_at?: DateTime;
  
  plan: {
    modules: PlannedActivity[];
    total_duration: number;
  };
  
  execution: {
    activities: CompletedActivity[];
  };
  
  state: {
    current_activity?: UUID;
    paused: boolean;
    attention_level: number;
    fatigue_indicator: number;
  };
}

interface CompletedActivity {
  module_id: UUID;
  started_at: DateTime;
  ended_at: DateTime;
  status: 'completed' | 'partial' | 'skipped';
  
  metrics: {
    time_spent: number;
    exercises_attempted: number;
    exercises_correct: number;
    hints_used: number;
    tutor_interactions: number;
  };
}
```

**Управление сессией:**

```python
async def manage_session(student_id, available_time):
    # 1. Инициализация
    session = create_session(student_id)
    
    # 2. Выбор активностей
    activities = select_activities(
        plan=get_learning_plan(student_id),
        time_budget=available_time,
        state=get_student_state(student_id)
    )
    
    # 3. Цикл выполнения
    for activity in activities:
        present_activity(activity)
        
        while not activity.completed:
            interaction = await wait_for_interaction()
            process_interaction(interaction)
            
            state = assess_state(session)
            
            if state.fatigue > THRESHOLD:
                offer_break()
            if state.attention < THRESHOLD:
                apply_engagement()
            if state.stuck_time > THRESHOLD:
                offer_help()
        
        record_completion(activity)
        
        if should_end(session):
            break
    
    # 4. Завершение
    summary = generate_summary(session)
    update_progress(student_id, session)
    
    return summary
```

### 10.3.2. Progress Tracker

**Модель прогресса:**

```typescript
interface StudentProgress {
  student_id: UUID;
  
  overall: {
    total_modules: number;
    completed: number;
    in_progress: number;
    average_mastery: number;
    total_time: number;
    streak_days: number;
  };
  
  by_subject: Map<SubjectId, SubjectProgress>;
  by_module: Map<ModuleId, ModuleProgress>;
  
  knowledge_map: Map<ConceptId, ConceptMastery>;
  
  achievements: Achievement[];
}

interface ConceptMastery {
  concept_id: UUID;
  mastery: number;        // 0-1
  confidence: number;     // 0-1
  last_practiced: DateTime;
  decay_rate: number;
}

interface ModuleProgress {
  module_id: UUID;
  status: 'not_started' | 'in_progress' | 'completed' | 'mastered';
  attempts: number;
  best_score: number;
  time_spent: number;
  mastery_level: number;
  last_activity: DateTime;
}
```

**Расчёт mastery:**

```python
def calculate_mastery(module_id, student_id):
    records = get_exercise_results(module_id, student_id)
    
    # Взвешенное среднее с учётом:
    # - давности (свежие важнее)
    # - сложности (сложные весят больше)
    # - помощи (без подсказок лучше)
    
    weighted_scores = []
    for record in records:
        recency_weight = exp(-days_ago(record) / 30)
        difficulty_weight = record.difficulty
        hint_penalty = 1 - 0.2 * record.hints_used
        
        weight = recency_weight * difficulty_weight
        score = record.score * hint_penalty
        
        weighted_scores.append((score, weight))
    
    mastery = sum(s * w for s, w in weighted_scores) / sum(w for _, w in weighted_scores)
    
    # Применяем decay
    days_since = days_since_last_practice(module_id, student_id)
    decay = exp(-days_since / decay_half_life(module_id))
    
    return mastery * decay
```

### 10.3.3. Assignment Manager

**Типы заданий:**

| Тип | Дедлайн | Автопроверка | Вес в оценке |
|-----|---------|--------------|--------------|
| Домашнее задание | Мягкий | Да | Низкий |
| Практическая работа | Мягкий | Частично | Средний |
| Контрольная работа | Жёсткий | Да | Высокий |
| Проект | Жёсткий | Нет | Высокий |
| Экзамен | Жёсткий | Да | Критический |

```typescript
interface Assignment {
  id: UUID;
  type: AssignmentType;
  
  module_id: UUID;
  title: string;
  description: string;
  
  exercises: UUID[];
  
  deadline: DateTime;
  deadline_type: 'soft' | 'hard';
  
  grading: {
    auto_grade: boolean;
    rubric?: Rubric;
    weight: number;
  };
  
  status: 'draft' | 'assigned' | 'submitted' | 'graded';
}
```

### 10.3.4. Grade Book

**Модель оценок:**

```typescript
interface GradeBook {
  student_id: UUID;
  academic_year: string;
  
  subjects: Map<SubjectId, SubjectGrades>;
}

interface SubjectGrades {
  subject_id: UUID;
  
  // По периодам
  periods: Period[];
  
  // Текущие
  current: {
    assignments: GradedAssignment[];
    average: number;
    predicted_final: number;
  };
  
  // Итоговые
  final?: {
    grade: number;
    passed: boolean;
    certificate_issued: boolean;
  };
}

interface Period {
  name: string;  // "1 четверть", "1 полугодие"
  start: Date;
  end: Date;
  
  grades: GradedAssignment[];
  average: number;
  comment?: string;
}
```

---

## 10.4. Social Learning

### 10.4.1. Study Groups

**Модель группы:**

```typescript
interface StudyGroup {
  id: UUID;
  name: string;
  type: 'class' | 'interest' | 'project' | 'support';
  
  members: GroupMember[];
  max_size: number;
  
  settings: {
    auto_matching: boolean;
    matching_criteria: MatchingCriteria;
    moderation_level: 'strict' | 'normal' | 'light';
  };
  
  activities: GroupActivity[];
  chat: GroupChat;
}

interface GroupMember {
  student_id: UUID;
  role: 'member' | 'leader' | 'mentor';
  joined_at: DateTime;
}
```

**Автоматическое формирование:**

```python
def form_study_groups(students, criteria):
    """
    Формирование групп по критериям:
    - Уровень (±1 от среднего)
    - Интересы (пересечение)
    - Время активности (пересечение)
    - Цели (совпадение)
    """
    
    # Кластеризация
    clusters = cluster_students(students, criteria)
    
    groups = []
    for cluster in clusters:
        # Разбиение на группы по 4-6 человек
        for chunk in chunked(cluster, size=(4, 6)):
            group = StudyGroup(
                type='auto_matched',
                members=chunk,
                matching_score=calculate_cohesion(chunk)
            )
            groups.append(group)
    
    return groups
```

### 10.4.2. Project Workspace

**Функции:**

| Функция | Описание |
|---------|----------|
| Доска задач | Канбан для распределения работы |
| Совместные документы | Редактирование в реальном времени |
| Чат проекта | Обсуждение в контексте |
| Версии | История изменений |
| Сдача | Отправка на проверку |

```typescript
interface ProjectWorkspace {
  id: UUID;
  project_id: UUID;
  group_id: UUID;
  
  board: TaskBoard;
  documents: SharedDocument[];
  chat: ProjectChat;
  
  submissions: Submission[];
  feedback: TeacherFeedback[];
}

interface TaskBoard {
  columns: Column[];  // To Do, In Progress, Review, Done
  tasks: Task[];
}

interface Task {
  id: UUID;
  title: string;
  assignee?: UUID;
  status: string;
  due_date?: Date;
}
```

### 10.4.3. Discussion Forum

**Модерация:**

```python
async def moderate_message(message, context):
    # 1. AI-проверка
    ai_check = await ai_moderate(message.content)
    
    if ai_check.harmful:
        return block_message(message, ai_check.reason)
    
    if ai_check.suspicious:
        flag_for_review(message)
    
    # 2. Фильтры
    if contains_contact_info(message.content):
        return block_message(message, "Контактные данные запрещены")
    
    if contains_profanity(message.content):
        return censor_message(message)
    
    # 3. Публикация
    return publish_message(message)
```

### 10.4.4. Peer Mentoring

**Система менторства:**

```python
def match_mentor_mentee(mentee):
    """
    Подбор ментора из старших/сильных учеников.
    """
    candidates = get_potential_mentors(
        subject=mentee.weak_subject,
        grade_range=(mentee.grade, mentee.grade + 2),
        min_mastery=0.8
    )
    
    # Ранжирование
    scored = []
    for mentor in candidates:
        score = (
            0.4 * mastery_score(mentor, mentee.weak_subject) +
            0.3 * availability_match(mentor, mentee) +
            0.2 * communication_style_match(mentor, mentee) +
            0.1 * mentor_rating(mentor)
        )
        scored.append((mentor, score))
    
    return max(scored, key=lambda x: x[1])[0]
```

---

## 10.5. Live Learning

### 10.5.1. Video Conference

**Интеграция с Jitsi:**

```typescript
interface LiveSession {
  id: UUID;
  type: 'group_lesson' | 'consultation' | 'exam' | 'workshop';
  
  scheduling: {
    scheduled_at: DateTime;
    duration_minutes: number;
    teacher_id: UUID;
    participants: UUID[];
    max_participants: number;
  };
  
  conference: {
    provider: 'jitsi' | 'zoom';
    room_id: string;
    join_url: string;
    host_url: string;
    recording_enabled: boolean;
  };
  
  content: {
    agenda: AgendaItem[];
    materials: Material[];
    whiteboard_enabled: boolean;
  };
  
  execution: {
    actual_start?: DateTime;
    actual_end?: DateTime;
    attendees: Attendee[];
    recording_url?: string;
    transcript?: Transcript;
  };
}
```

### 10.5.2. Interactive Whiteboard

**Функции:**

| Функция | Описание |
|---------|----------|
| Рисование | Свободное рисование, фигуры |
| Текст | Добавление текста |
| Формулы | LaTeX рендеринг |
| Изображения | Загрузка и аннотация |
| Совместная работа | Множество курсоров |
| История | Отмена/повтор |

### 10.5.3. Teacher Scheduler

**Управление расписанием:**

```typescript
interface TeacherSchedule {
  teacher_id: UUID;
  
  availability: TimeSlot[];
  
  sessions: ScheduledSession[];
  
  preferences: {
    max_hours_per_day: number;
    min_break_between: number;
    preferred_times: TimeRange[];
  };
}

interface TimeSlot {
  day_of_week: number;
  start_time: Time;
  end_time: Time;
  recurring: boolean;
}
```

**Автоматическое планирование:**

```python
def schedule_group_session(group, teacher, duration):
    # 1. Получение доступности
    teacher_slots = get_availability(teacher)
    student_slots = [get_availability(s) for s in group.members]
    
    # 2. Поиск пересечения
    common_slots = find_intersection(teacher_slots, *student_slots)
    
    # 3. Оптимальный слот
    optimal = select_optimal_slot(
        common_slots,
        duration=duration,
        preferences=[
            teacher.preferences,
            *[s.preferences for s in group.members]
        ]
    )
    
    # 4. Создание сессии
    return create_session(
        teacher=teacher,
        participants=group.members,
        time=optimal,
        duration=duration
    )
```

---

*Следующий раздел: [05_APPLICATIONS.md](05_APPLICATIONS.md)*
