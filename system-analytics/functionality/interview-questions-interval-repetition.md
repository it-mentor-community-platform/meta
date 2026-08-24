# Системная аналитика - интервальное повторение вопросов для собеседований
## Необходимый контекст

- [Бизнес аналитика - интервальное повторение][(https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/data-import.md](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/interview-questions-interval-repetition.md))

## Инициализация

### Шаги

- Пользователь заходит в раздел «Интервальное повторение»
- Пользователю отображается список всех специализаций и категорий
- Пользователь нажимает «Добавить на повторение»
- Категория добавляется в список категорий пользователя, выбранных для повторения


```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Открывает экран выбора категорий

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/specializations
    IntervalRepetitionService->>Database: Получить специализации и категории
    Database-->>IntervalRepetitionService: Специализации, категории и метрики
    IntervalRepetitionService-->>Frontend: 200 OK
    Frontend-->>User: Отображает список специализаций и категорий

    User->>Frontend: Выбирает категории для повторения

    User->>Frontend: Нажимает «Продолжить»

    Frontend->>IntervalRepetitionService: POST /api/interval-repetition/selected-categories
    Note right of Frontend: categoryIds: [1, 2, 3]

    IntervalRepetitionService->>Database: Сохранить выбранные категории
    Database-->>IntervalRepetitionService: Категории сохранены

    IntervalRepetitionService-->>Frontend: 201 Created
    Frontend-->>User: Переход на главный экран
```

## Вопросы на повторение

### Шаги

- Пользователь заходит в раздел «Вопросы на повторение»
- Отображается список категорий, ранее добавленных пользователем на повторение
- Для каждой категории отображается информация о вопросах, доступных для повторения
- Пользователь может:
  - нажать «Повторить всё» и перейти к повторению вопросов из всех добавленных категорий
  - выбрать конкретную категорию и перейти к повторению вопросов только из неё

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Открывает «Вопросы на повторение»

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/selected-categories
    IntervalRepetitionService->>Database: Получить выбранные категории и метрики
    Database-->>IntervalRepetitionService: Категории и метрики
    IntervalRepetitionService-->>Frontend: 200 OK
    Frontend-->>User: Отображает категории

    User->>Frontend: Выбирает категорию
    Frontend->>Frontend: Переход к повторению категории

    User->>Frontend: Нажимает «Повторить всё»
    Frontend->>Frontend: Переход к повторению всех категорий
```


## Получить вопрос из конкретной категории

### Шаги

- Пользователь выбирает категорию в разделе «Вопросы на повторение»
- Запускается флоу повторения вопросов выбранной категории
- Для повторения выбирается вопрос:
  - который ещё не имеет состояния у пользователя
  - или состояние которого доступно для повторения (`next_review_at <= текущего времени`)
- Новый вопрос имеет приоритет перед вопросами, которые уже имеют состояние
- Выбранный вопрос отображается пользователю

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Открывает Dashboard

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/selected-categories
    IntervalRepetitionService->>Database: Получить выбранные категории пользователя
    Database-->>IntervalRepetitionService: Категории и метрики вопросов
    IntervalRepetitionService-->>Frontend: 200 OK + список выбранных категорий
    Frontend-->>User: Отображает выбранные категории и метрики

    User->>Frontend: Выбирает категорию

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/selected-categories{categoryId}/next-question
    IntervalRepetitionService->>Database: Получить вопрос из категории
    Database-->>IntervalRepetitionService: Вопрос

    IntervalRepetitionService-->>Frontend: 200 OK + вопрос + questions_left
    Frontend-->>User: Отображает вопрос и прогресс
```
Запрос к БД:
```sql
SELECT q.*
FROM question q
LEFT JOIN user_question_schedule uqs
    ON uqs.question_id = q.id
   AND uqs.user_id = :userId
WHERE q.category_id = :categoryId
  AND (
      uqs.question_id IS NULL
      OR uqs.next_review_at <= :now
  )
ORDER BY uqs.next_review_at NULLS FIRST
LIMIT 1;
```


## Получить вопрос из выбранных категорий

### Шаги

- Пользователь нажимает «Повторить всё»
- Запускается флоу повторения вопросов из всех категорий, добавленных пользователем на повторение
- Для повторения выбираются вопросы:
  - которые ещё не имеют состояния у пользователя
  - или состояние которых доступно для повторения (`next_review_at <= текущего времени`)
- Новый вопрос имеет приоритет перед вопросами, которые уже имеют состояние
- Из доступных вопросов выбирается один вопрос и отображается пользователю

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Нажимает «Повторить всё»

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/selected-categories/next-question

    IntervalRepetitionService->>Database: Получить доступный вопрос из выбранных категорий
    Database-->>IntervalRepetitionService: Вопрос

    IntervalRepetitionService-->>Frontend: 200 OK + вопрос + questions_left
    Frontend-->>User: Отображает вопрос и прогресс
```

Запрос к БД:

```sql
SELECT q.*
FROM question q
JOIN user_category_selection ucs
    ON ucs.category_id = q.category_id
   AND ucs.user_id = :userId
LEFT JOIN user_question_schedule uqs
    ON uqs.question_id = q.id
   AND uqs.user_id = :userId
WHERE uqs.question_id IS NULL
   OR uqs.next_review_at <= :now
ORDER BY uqs.next_review_at NULLS FIRST
LIMIT 1;
```

## Оценка ответа

### Шаги

- Пользователь отвечает на вопрос и оценивает свой ответ
- По полученной оценке рассчитывается новое состояние вопроса по алгоритму SM-2
- Результат попытки сохраняется в `review_attempt`
- Новое состояние сохраняется в `user_question_schedule`
- После оценки пользователь переходит к следующему вопросу

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Оценивает свой ответ

    Frontend->>IntervalRepetitionService: POST /api/interval-repetition/review-attempt
    Note right of Frontend: questionId: 123<br/>quality: 4

    IntervalRepetitionService->>IntervalRepetitionService: Рассчитать состояние вопроса по SM-2

    IntervalRepetitionService->>Database: Сохранить результат в review_attempt
    Database-->>IntervalRepetitionService: Результат сохранён

    IntervalRepetitionService->>Database: UPSERT user_question_schedule
    Database-->>IntervalRepetitionService: Состояние сохранено

    IntervalRepetitionService-->>Frontend: 200 OK
    Frontend-->>User: Переход к следующему вопросу
```

UPSERT состояния вопроса:

```sql
INSERT INTO user_question_schedule (
    user_id,
    question_id,
    successful_repetitions,
    ease_factor,
    interval,
    next_review_at,
    last_review_at
)
VALUES (
    :userId,
    :questionId,
    :successfulRepetitions,
    :easeFactor,
    :interval,
    :nextReviewAt,
    :lastReviewAt
)
ON CONFLICT (user_id, question_id)
DO UPDATE SET
    successful_repetitions = EXCLUDED.successful_repetitions,
    ease_factor = EXCLUDED.ease_factor,
    interval = EXCLUDED.interval,
    next_review_at = EXCLUDED.next_review_at,
    last_review_at = EXCLUDED.last_review_at;
```


## Изменение списка категорий на повторение

### Шаги

- Пользователь может перейти к списку всех доступных специализаций и категорий
- Категории, ранее добавленные на повторение, отображаются как выбранные
- Пользователь может добавить новую категорию на повторение, категория добавляется в `user_category_selection`
- Пользователь может удалить категорию из списка повторения, категория удаляется из `user_category_selection`

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Открывает страницу всех категорий

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/specializations
    IntervalRepetitionService->>Database: Получить специализации, категории и selected
    Database-->>IntervalRepetitionService: Специализации, категории и selected
    IntervalRepetitionService-->>Frontend: 200 OK
    Frontend-->>User: Отображает категории с отметками выбранных

    User->>Frontend: Выбирает категорию

    Frontend->>IntervalRepetitionService: POST /api/interval-repetition/selected-categories/{categoryId}
    IntervalRepetitionService->>Database: Добавить категорию в выбранные
    Database-->>IntervalRepetitionService: Категория добавлена

    IntervalRepetitionService->>Database: Создать UserQuestionSchedule для вопросов категории
    Database-->>IntervalRepetitionService: Состояния вопросов созданы

    IntervalRepetitionService-->>Frontend: 201 Created
    Frontend-->>User: Помечает категорию как выбранную

    User->>Frontend: Убирает выбор категории

    Frontend->>IntervalRepetitionService: DELETE /api/interval-repetition/selected-categories/{categoryId}
    IntervalRepetitionService->>Database: Удалить категорию из выбранных
    Database-->>IntervalRepetitionService: Категория удалена
    IntervalRepetitionService-->>Frontend: 204 No Content
    Frontend-->>User: Снимает отметку выбранной категории
```


## Просмотр вопросов одной категории

### Шаги

- Пользователь открывает категорию из списка всех доступных категорий
- Отображается список вопросов и ответов выбранной категории
- Пользователь может ознакомиться с вопросами и ответами
```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant IntervalRepetitionService
    participant Database

    User->>Frontend: Открывает страницу категории

    Frontend->>IntervalRepetitionService: GET /api/interval-repetition/categories/{categoryId}
    IntervalRepetitionService->>Database: Получить категорию, вопросы и состояния пользователя
    Database-->>IntervalRepetitionService: Категория, вопросы и UserQuestionSchedule
    IntervalRepetitionService-->>Frontend: 200 OK + данные категории и список вопросов

    Frontend-->>User: Отображает категорию и список вопросов
```
