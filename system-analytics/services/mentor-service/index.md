# Mentor Service

## Стек

- Spring Boot 3
- Spring Data JDBC
- Liquibase

## Взаимодействия

Входящие:
- REST эндпоинты
- Kafka

## Схема БД

```mermaid
erDiagram
    Mentors {
        bigint id PK
        bigint mentor_telegram_user_id UK
        string telegram_url "https://t.me/zhukovsd"
        bool is_active "Выдана ли ментору роль 'MENTOR'"
    }

    Guaranteed_Reviews_Prices {
        bigint id PK
        bigint mentor_id FK
        string project_type "HANGMAN, SIMULATION, ..."
        string language "Java, Kotlin, ..."
        int price_usd 
    }

    Mentor_Descriptions {
        bigint id PK
        bigint mentor_user_id FK
        string name
        int cost "FREE, PAID, FREE_AND_PAID"
        string description
    }

    Programming_Languages {
        bigint id PK
        string name UK
    }

    Mentors_Programming_Languages {
        bigint mentor_id FK
        bigint programming_language_id FK
    }

    Services {
        bigint id PK
        string name UK
    }

    Mentors_Services {
        bigint mentor_id FK
        bigint service_id FK
    }

    Mentors ||--o{ Guaranteed_Reviews_Prices : "id = mentor_id"
    Mentors ||--o{ Mentor_Descriptions : "id = mentor_user_id"

    Mentors ||--o{ Mentors_Programming_Languages : "id = mentor_id"
    Programming_Languages ||--o{ Mentors_Programming_Languages : "id = programming_language_id"

    Mentors ||--o{ Mentors_Services : "id = mentor_id"
    Services ||--o{ Mentors_Services : "id = service_id"
```

Индексы:
- `Mentors` - уникальный индекс на значение `mentor_telegram_user_id` для проверки уникальности ментора
- `Mentors` - уникальный композитный индекс на комбинацию значений `mentor_id`, `project_type`, `language` 
- `Mentors_Programming_Languages` - композитный primary key на обе колонки
- `Mentors_Services` - композитный primary key на обе колонки
- `Programming_Languages` - `name` уникальна и case-insensitive (пример - если есть значение "Java", то "JAVA" уже не вставить)
- `Services` - `name` уникальна и case-insensitive

## Схема REST API

Для всех методов передаются [кастомные заголовки запроса](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/gateway/index.md#%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D0%BB%D0%B0-security) с Telegram Id и ролями пользователя.

### Ответ в случае ошибки

Актуально для всех методов.

Код должен соответствовать ситуации (перечислено ниже), тело:
```
{
  "message": "Текст ошибки"
}
```

### Внутренний эндпоинт для добавления ментора

`POST /api/mentor/internal/mentor`

Тело запроса (`Content-Type: application-json`):

```
{
  "mentorTelegramUserId": 123456,
  "telegramUrl": "https://t.me/zhukovsd",
  "description": {
    "name": "Sergey Zhukov",
    "cost": "PAID",
    "description": "Мои услуги - https://zhukovsd.it/services/"
  },
  "programmingLanguages": ["Java", "Python", "Kotlin"],
  "services": ["менторство по поиску работы", "консультации"]
}
```

Ответ в случае успеха: `201 Created` если ментор была создан, `200 OK` если обновлён.

Коды ошибок:

- 400 - ошибки валидации
- 500 - неизвестная ошибка

### Внутренний эндпоинт для добавления цены на гарантированное ревью из Data Importer

`POST /api/mentor/internal/guaranteed-review`

Тело запроса (`Content-Type: application-json`):

```
{
  "telegram_url": "https://t.me/zhukovsd",
  "language": "Java",
  "project_type": "SIMULATION",
  "price_usd": 15
}
```

Ответ в случае успеха: `201 Created` если цена была создана, `200 OK` если обновлена.

Коды ошибок:

- 400 - ошибки валидации (невалидный тип проекта, невалидная ссылка на Telegram профиль)
- 500 - неизвестная ошибка
- 404 - ментор не найден в profile service по ссылке на Telegram профиль 

## Kafka

### Consumer для топика `auth.user.authenticated`

Consumer group - `mentor-service-cg`.

Используется для актуализации поля `is_active` в таблице `Mentors`. Если у пользователя есть роль `MENTOR`, `is_mentor` присваивается значение `true`.

Payload сообщения - https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/auth-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-authuserauthenticated.
