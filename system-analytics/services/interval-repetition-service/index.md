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

    SPECIALIZATION {
        bigint id
        string name
    }

    CATEGORY {
        bigint id
        bigint specialization_id
        string name
    }

    QUESTION {
        bigint id
        bigint category_id
        string title
        string expected_answer
        boolean enabled
    }

    USER_QUESTION_SCHEDULE {
        bigint id
        bigint user_id
        bigint question_id
        int successful_repetitions
        double ease_factor
        int interval
        datetime next_review
        datetime last_review
    }

    REVIEW_ATTEMPT {
        bigint id
        bigint user_question_schedule_id
        int quality
        double ease_factor
        double new_ease_factor
        datetime answered_at
    }

    SPECIALIZATION ||--o{ CATEGORY : contains
    CATEGORY ||--o{ QUESTION : contains
    QUESTION ||--o{ USER_QUESTION_SCHEDULE : scheduled
    USER_QUESTION_SCHEDULE ||--o{ REVIEW_ATTEMPT : attempts
```
Индексы:
- Составной индекс на колонки user_id, next_review_at таблицы user_question_schedule для быстрого поиска вопросов пользователя, доступных для повторения, с сортировкой по ближайшей дате следующего показа
- Индекс на колонку category_id таблицы question для быстрого получения вопросов определенной категории
- Индекс на колонку user_question_schedule_id таблицы review_attempt для получения истории попыток повторения конкретного состояния вопроса пользователя
- Unique составной индекс на колонки user_id, question_id таблицы user_question_schedule для гарантии уникальности состояния повторения вопроса для пользователя 

## Схема REST API

TODO
