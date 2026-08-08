# Interval Repetition Service

## Стек

- Spring Boot 3
- Spring Data JDBC
- Liquibase

## Взаимодействия

Входящие:
- REST эндпоинты

## Схема БД

```mermaid
erDiagram

    Specializations {
        bigint id PK
        string name UK
    }

    Categories {
        bigint id PK
        bigint specialization_id FK
        string name
    }

    Questions {
        bigint id PK
        bigint category_id FK
        string title
        string answer
        bool enabled
    }

    User_Question_Schedules {
        bigint id PK
        bigint user_id
        bigint question_id FK
        int successful_repetitions
        double ease_factor
        int interval
        bigint next_review_at "Unix Epoch Timestamp"
        bigint last_review_at "Unix Epoch Timestamp"
    }

    Review_Attempts {
        bigint id PK
        bigint user_question_schedule_id FK
        int quality "0..5"
        double ease_factor
        double new_ease_factor
        bigint answered_at "Unix Epoch Timestamp"
    }

    Specializations ||--o{ Categories : "id = specialization_id"

    Categories ||--o{ Questions : "id = category_id"

    Questions ||--o{ User_Question_Schedules : "id = question_id"

    User_Question_Schedules ||--o{ Review_Attempts : "id = user_question_schedule_id"
```
Индексы:
- Составной индекс на колонки user_id, next_review_at таблицы user_question_schedule для быстрого поиска вопросов пользователя, доступных для повторения, с сортировкой по ближайшей дате следующего показа
- Индекс на колонку category_id таблицы question для быстрого получения вопросов определенной категории
- Индекс на колонку user_question_schedule_id таблицы review_attempt для получения истории попыток повторения конкретного состояния вопроса пользователя
- Unique составной индекс на колонки user_id, question_id таблицы user_question_schedule для гарантии уникальности состояния повторения вопроса для пользователя 

## Схема REST API

### Внутренний эндпоинт для импорта вопроса

Используется для загрузки вопросов из Google Spreadsheet. Вопрос идентифицируется по title. Если вопрос не существует в бд, то добавляется с помощью INSERT, иначе вызывается UPDATE.

Специализации и темы добавляются вместе с вопросом, если ещё не существуют.

`POST /api/interval-repetition/internal/question`

Тело запроса (`Content-Type: application-json`)
```
{
  "specialization": "Java",
  "category": "Java Core",
  "title": "Какие типы ссылок существуют в Java?",
  "answer": "Сильная ссылка...",
}
```

Ответ в случае успеха: `200 OK` 

Тело ответа (`Content-Type: application-json`)
```
{
  "question": {
    "id": 1,
    "categoryId": 1,
    "title": "Какие типы ссылок существуют в Java?",
    "answer": "Сильная ссылка...",
    "enabled": true
  },
  "category": {
    "id": 1,
    "specializationId": 1,
    "name": "Java Core"
  },
  "specialization": {
    "id": 1,
    "name": "Java"
  }
}
```


В случае ошибок:

- 400 Bad Request — ошибки валидации.
- 500 Internal Server Error — неизвестная ошибка.

Тело ответа при ошибке (`Content-Type: application-json`)

```
{
  "message": "Validation failed..."
}
```
