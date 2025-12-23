# Часть III: Ядро ИИ-системы (AI Core)

## Содержание

- [9.1. Обзор домена](#91-обзор-домена)
- [9.2. Adaptive Learning Engine](#92-adaptive-learning-engine)
- [9.3. AI Tutor Service](#93-ai-tutor-service)
- [9.4. Content Generation Engine](#94-content-generation-engine)
- [9.5. Assessment Engine](#95-assessment-engine)
- [9.6. Analytics Engine](#96-analytics-engine)

---

## 9.1. Обзор домена

### Назначение

AI Core — центральный домен платформы, обеспечивающий все функции искусственного интеллекта.

### Структура

```
AI CORE
├── ADAPTIVE LEARNING ENGINE
│   ├── Cognitive Profiler
│   ├── Learning Path Optimizer
│   ├── Gap Detector
│   ├── Difficulty Calibrator
│   └── Engagement Monitor
│
├── AI TUTOR SERVICE
│   ├── Dialog Manager
│   ├── Context Builder
│   ├── Socratic Engine
│   ├── Emotion Recognizer
│   ├── Knowledge Retriever
│   └── Escalation Handler
│
├── CONTENT GENERATION ENGINE
│   ├── Exercise Generator
│   ├── Explanation Generator
│   ├── Example Personalizer
│   └── Quality Validator
│
├── ASSESSMENT ENGINE
│   ├── Auto Grader
│   ├── Essay Analyzer
│   ├── Handwriting Recognizer
│   ├── Proctoring System
│   └── Feedback Generator
│
└── ANALYTICS ENGINE
    ├── Performance Predictor
    ├── Risk Assessor
    └── Recommendation Engine
```

---

## 9.2. Adaptive Learning Engine

### 9.2.1. Cognitive Profiler

**Функции:**

| Функция | Вход | Выход |
|---------|------|-------|
| `runInitialDiagnosis` | Ответы на тесты | CognitiveProfile |
| `detectLearningStyle` | Тесты + поведение | LearningStyle |
| `assessWorkingMemory` | Специальные тесты | Score (1-10) |
| `identifyStrengths` | История обучения | StrengthAreas[] |
| `updateProfile` | Взаимодействия | UpdatedProfile |

**Алгоритм определения стиля:**

```python
def detect_learning_style(test_results):
    # Группировка по формату: visual, auditory, read_write, kinesthetic
    grouped = group_by_format(test_results)
    
    # Расчёт эффективности
    scores = {}
    for format_type, results in grouped.items():
        # efficiency = correctness * (1/time) * confidence
        scores[format_type] = calculate_efficiency(results)
    
    # Определение стиля
    max_score = max(scores.values())
    mean_score = mean(scores.values())
    
    if max_score > 1.3 * mean_score:
        return get_dominant_style(scores)
    else:
        return 'mixed'
```

### 9.2.2. Learning Path Optimizer

**Алгоритм оптимизации:**

```python
def optimize_sequence(knowledge_graph, profile, target_modules):
    # 1. Топологическая сортировка по зависимостям
    sorted_modules = topological_sort(target_modules, knowledge_graph)
    
    # 2. Расчёт приоритетов
    for module in sorted_modules:
        module.priority = (
            w1 * count_dependents(module) +
            w2 * closes_gap(module) +
            w3 * matches_interests(module, profile) +
            w4 * exam_importance(module)
        )
    
    # 3. Ограничения
    constrained = apply_constraints(sorted_modules, [
        max_hard_in_row(3),
        subject_diversity(),
        optimal_time(profile)
    ])
    
    # 4. Распределение по дням
    return schedule(constrained, profile.optimal_duration)
```

### 9.2.3. Gap Detector

**Алгоритм:**

```python
def detect_gap(error_history, knowledge_graph, student_state):
    # 1. Кластеризация ошибок
    clusters = cluster_by_concept(error_history)
    
    # 2. Выявление аномалий
    suspicious = [c for c in clusters 
                  if error_rate(c) > 0.4 
                  and error_rate(c) > historical(c) * 1.5]
    
    # 3. Поиск корневой причины
    for concept in suspicious:
        prereqs = knowledge_graph.prerequisites(concept)
        weak = [p for p in prereqs if student_state[p].mastery < 0.7]
        root = max(weak, key=impact_score) if weak else concept
    
    # 4. План устранения
    return Gap(
        root_cause=root,
        criticality=calculate_criticality(root),
        remediation=find_shortest_path(student_state, root)
    )
```

### 9.2.4. Difficulty Calibrator

**IRT-модель:**

```python
def success_probability(ability, difficulty):
    """3-параметрическая IRT модель"""
    c = 0.2  # угадывание
    a = 1.0  # дискриминативность
    return c + (1 - c) / (1 + exp(-a * (ability - difficulty)))

def update_ability(ability, difficulty, success, lr=0.1):
    p = success_probability(ability, difficulty)
    if success:
        return ability + lr * (1 - p)
    else:
        return ability - lr * p

def select_next_item(ability, items):
    """Выбор задания с максимальной информативностью"""
    def information(item):
        p = success_probability(ability, item.difficulty)
        return item.discrimination**2 * p * (1 - p)
    return max(items, key=information)
```

**Зона ближайшего развития:**

| Success Rate | Действие |
|--------------|----------|
| > 85% | Увеличить сложность |
| 70-85% | Оптимальная зона |
| < 70% | Уменьшить сложность |

---

## 9.3. AI Tutor Service

### 9.3.1. Dialog Manager

**Состояния:**

```
Greeting → Explaining → Questioning → Helping → Summarizing
              ↓              ↓
         FrustrationHandling ←
```

**Обработка:**

```python
async def process_message(message, session):
    # 1. Предобработка
    text = preprocess(message)  # speech-to-text, OCR
    
    # 2. Классификация интента
    intent = classify_intent(text, session.context)
    # AskQuestion, RequestHelp, ExpressFrustration, 
    # ConfirmUnderstanding, Chitchat
    
    # 3. Определение эмоции
    emotion = detect_emotion(text)
    
    # 4. Выбор стратегии
    strategies = {
        'AskQuestion': socratic_response,
        'RequestHelp': help_with_exercise,
        'ExpressFrustration': emotional_support,
        'ConfirmUnderstanding': verify_and_proceed,
        'Chitchat': gentle_redirect
    }
    strategy = strategies[intent]
    
    # 5. Проверка эскалации
    if emotion.frustration > 0.8 or repeated_question > 3:
        strategy = escalate_to_human
    
    return await generate_response(strategy, session)
```

### 9.3.2. Socratic Engine

```python
async def socratic_dialogue(question, knowledge_base, profile):
    # Анализ вопроса
    target = identify_target_concept(question)
    current = get_understanding(profile, target)
    
    # Путь к пониманию
    steps = find_path(current, target)
    
    # Наводящий вопрос
    for step in steps:
        if student_knows(profile, step.prerequisite):
            return generate_leading_question(
                f"Ты знаешь {step.prerequisite}. "
                f"Что из этого следует для {step.next}?"
            )
    
    # Fallback
    return direct_explanation(target)
```

**Типы вопросов:**

| Тип | Пример |
|-----|--------|
| Clarifying | "Что ты имеешь в виду?" |
| Probing | "Почему так думаешь?" |
| Connecting | "Как связано с...?" |
| Hypothetical | "Что если...?" |
| Consequential | "К чему приводит?" |

### 9.3.3. Knowledge Retriever (RAG)

```python
async def retrieve(query, topic, level):
    # 1. Понимание запроса
    entities = extract_entities(query)
    
    # 2. Поиск в графе знаний
    graph_results = knowledge_graph.search(entities)
    
    # 3. Векторный поиск
    embedding = embed(query)
    vector_results = vector_db.search(
        embedding, 
        filter={'level': level},
        top_k=10
    )
    
    # 4. Ранжирование и сборка
    ranked = rank(graph_results + vector_results)
    return assemble_context(ranked[:5])
```

---

## 9.4. Content Generation Engine

### 9.4.1. Exercise Generator

```python
async def generate_exercise(topic, difficulty, profile):
    # 1. Выбор шаблона
    template = select_template(topic, difficulty)
    
    # 2. Контекст из интересов
    theme = select_diverse(profile.interests)
    
    # 3. Генерация значений
    values = {}
    for var in template.variables:
        if var.type == 'number':
            values[var.name] = sample_number(var.constraints, difficulty)
        else:
            values[var.name] = contextualize(var, theme)
    
    # 4. Рендеринг
    text = template.render(values)
    solution = template.solution.render(values)
    hints = generate_hints(template.hints, values)
    
    # 5. Валидация
    validate(text, solution)
    
    return Exercise(text, solution, hints, difficulty)
```

### 9.4.2. Explanation Generator

```python
async def generate_explanation(concept, profile, context):
    # Формат под стиль
    format_map = {
        'visual': ['diagram', 'video'],
        'auditory': ['conversational'],
        'read_write': ['text', 'examples']
    }
    formats = format_map[profile.learning_style]
    
    # Уровень детализации
    detail = {
        'new_topic': 'full',
        'confusion': 'simplified',
        'review': 'summary'
    }[context]
    
    # Аналогии из интересов
    analogy = find_or_generate_analogy(concept, profile.interests)
    
    # Связь с известным
    bridge = find_bridge(profile.known_concepts, concept)
    
    return Explanation(
        hook=generate_hook(concept),
        bridge=bridge,
        core=generate_core(concept, formats, detail),
        analogy=analogy,
        example=generate_example(concept, profile),
        check=generate_check_question(concept)
    )
```

---

## 9.5. Assessment Engine

### 9.5.1. Auto Grader

**Методы:**

| Тип | Метод | Сложность |
|-----|-------|-----------|
| Выбор ответа | Точное совпадение | Низкая |
| Число | Сравнение ± допуск | Низкая |
| Формула | SymPy | Средняя |
| Код | Тесты | Высокая |
| Текст | Семантика | Высокая |
| Эссе | LLM + рубрика | Очень высокая |

**Результат:**

```typescript
interface GradingResult {
  score: number;
  feedback: {
    summary: string;
    correct_parts: string[];
    errors: Error[];
    next_steps: string;
  };
  confidence: number;
  needs_review: boolean;
}
```

### 9.5.2. Essay Analyzer

```python
async def analyze_essay(text, assignment, profile):
    # 1. Оригинальность
    plagiarism = check_plagiarism(text)
    ai_generated = detect_ai(text)
    
    # 2. Структура
    structure = {
        'has_intro': detect_intro(text),
        'has_thesis': detect_thesis(text),
        'body_count': count_paragraphs(text),
        'has_conclusion': detect_conclusion(text)
    }
    
    # 3. Содержание (LLM)
    content = await llm_analyze(text, assignment.rubric)
    
    # 4. Язык
    language = {
        'grammar': check_grammar(text),
        'spelling': check_spelling(text),
        'vocabulary': lexical_diversity(text)
    }
    
    # 5. Оценка по рубрике
    scores = calculate_rubric_scores(structure, content, language)
    
    return EssayAnalysis(
        scores=scores,
        total=weighted_sum(scores),
        feedback=generate_feedback(scores),
        needs_review=(plagiarism > 0.2)
    )
```

---

## 9.6. Analytics Engine

### 9.6.1. Performance Predictor

**Модели:**

| Горизонт | Модель |
|----------|--------|
| < 2 недели | Linear Regression |
| 2-12 недель | Gradient Boosting |
| > 12 недель | Neural + Bayesian |

**Features:**

```python
def extract_features(student_id):
    history = get_history(student_id)
    return {
        'avg_score_30d': avg_score(history, 30),
        'score_trend': trend(history.scores),
        'study_regularity': regularity(history.sessions),
        'tutor_usage': tutor_usage(history),
        'hint_dependency': hint_usage(history),
        'days_inactive': days_since_last(history),
        'subject_affinity': is_strength(student_id),
        'prereq_strength': prereq_score(student_id)
    }
```

### 9.6.2. Risk Assessor

**Типы рисков:**

| Риск | Индикаторы | Действие |
|------|------------|----------|
| Отставание | Прогресс < 70% плана | Корректировка |
| Потеря мотивации | Engagement ↓ 20% | Геймификация |
| Отчисление | Неактивен > 7 дней | Звонок куратора |
| Выгорание | Слишком много часов | Рекомендация отдыха |

```python
def assess_risks(student_id):
    data = get_data(student_id)
    risks = []
    
    # Отставание
    if data.progress < 0.7 * expected:
        risks.append(Risk('falling_behind', severity))
    
    # Мотивация
    trend = engagement_trend(data, days=14)
    if trend < -0.2:
        risks.append(Risk('motivation_loss', abs(trend)))
    
    # Отчисление
    inactive = days_since(data.last_activity)
    if inactive > 7:
        risks.append(Risk('churn', min(1, inactive/30)))
    
    return sorted(risks, key=lambda r: r.severity, reverse=True)
```

---

*Следующий раздел: [04_EDU_CORE.md](04_EDU_CORE.md)*
