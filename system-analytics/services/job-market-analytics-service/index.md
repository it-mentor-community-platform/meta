# Job Market Analytics Service

## Стек

- Spring Boot 3
- Spring Data JDBC
- Liquibase

## Взаимодействия

Входящие:
- REST эндпоинты
- Kafka

Исходящие:
- API headhunter.ru

## Схема БД

```
erDiagram
    search_queries {
        bigint id PK
        varchar title
        text query
        boolean is_enabled
    }

    market_data_points {
        bigint id PK
        bigint search_query_id FK
        date snapshot_date
        integer vacancy_count
        numeric avg_salary
        numeric median_salary_mid
        timestamp created_at
    }

    search_queries ||--o{ market_data_points : has
```

Индексы:
- `market_data_points` - композитный `UNIQUE` на пару значений `search_query_id`, `snapshot_date`
