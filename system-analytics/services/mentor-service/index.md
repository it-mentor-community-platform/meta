# Mentor Service

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
    Mentors {
        bigint id PK
        bigint mentor_telegram_user_id
        bool is_active "выдана ли ментору роль 'MENTOR'"
    }

    Guaranteed_Reviews_Prices {
        bigint id PK
        bigint mentor_id FK
        string project_type "HANGMAN, SIMULATION, ..."
        string language "Java, Kotlin, ..."
        int price_usd 
    }

    Mentors ||--o{ Guaranteed_Reviews_Prices : "id = mentor_id"
```

Индексы:
- Уникальный композитный индекс на комбинацию значений `mentor_id`, `project_type`, `language` 
