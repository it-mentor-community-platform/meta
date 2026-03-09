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

## Схема REST API

Для всех методов передаются [кастомные заголовки запроса](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/gateway/index.md#%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D0%BB%D0%B0-security) с Telegram Id и ролями пользователя.

Формат адресов запроса - `/api/имя сервиса/эндпоинт`.

Управление поисковыми запросами доступно только пользователям с ролью `ADMIN`.

---

### Ответ в случае ошибки

Актуально для всех методов.

Код должен соответствовать ситуации (перечислено ниже), тело:

```
{
  "message": "Текст ошибки"
}
```

### Эндпоинт для создания поискового запроса

`POST /api/job-market-analytics/search-query`

Тело запроса (`Content-Type: application/json`):

```
{
  "title": "Java Developer",
  "query": "NAME:(!"Java") NOT QA NOT AQA",
  "isEnabled": true
}
```

Ответ в случае успеха: `201 Created`.

Тело ответа:

```
{
  "id": 1,
  "title": "Java Developer",
  "query": "NAME:(!"Java") NOT QA NOT AQA",
  "isEnabled": true
}
```

Коды ошибок:

- 400 - ошибки валидации
- 403 - недостаточно прав
- 409 - поисковый запрос с таким `title` уже существует
- 500 - неизвестная ошибка

### Эндпоинт для редактирования поискового запроса

`PUT /api/job-market-analytics/search-query/{id}`

Тело запроса (`Content-Type: application/json`):

```
{
  "title": "Java Developer",
  "query": "NAME:(!"Java") NOT QA NOT AQA",
  "isEnabled": true
}
```

Ответ в случае успеха: `200 OK`.

Коды ошибок:

- 400 - ошибки валидации
- 403 - недостаточно прав
- 404 - поисковый запрос не найден
- 500 - неизвестная ошибка

### Эндпоинт для получения списка поисковых запросов

`GET /api/job-market-analytics/search-queries`

GET Параметры:

- `isEnabled` — optional, фильтр по признаку активности

Пример:

`GET /api/job-market-analytics/search-query?isEnabled=true`

Ответ в случае успеха: `200 OK`.

Тело ответа:

```
[
  {
    "id": 1,
    "title": "Java Developer",
    "query": "NAME:(!"Java") NOT QA NOT AQA",
    "isEnabled": true
  },
  {
    "id": 2,
    "title": "Python Developer",
    "query": "Name:(python or django or drf or backend or fastapi or flask) and DESCRIPTION:(django or drf or fastapi or flask)",
    "isEnabled": true
  }
]
```

Коды ошибок:

- 400 - невалидное значение query param
- 403 - недостаточно прав
- 500 - неизвестная ошибка
