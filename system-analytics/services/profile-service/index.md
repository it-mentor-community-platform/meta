# Profile Service

## Стек

- Spring Boot 3
- Spring Data JDBC
- Spring Kafka
- Liquibase

## Взаимодействия

Входящие:
- REST эндпоинты
- Kafka

## Схема БД

```mermaid
erDiagram
    Profiles {
        bigint id PK
        bigint telegram_user_id
    }

    Profiles_Details {
        bigint profile_id FK
        string detail_name
        string detail_value
    }

    Achievements {
        bigint id PK
        bigint profile_id FK
        string achievement_type "PROJECTS_SPRINTER, FIRST_PROJECT, ..."
        bigint earned_timestamp "Unix timestamp в секундах получения ачивки"
        bool publicly_visible "Видна ли ачивка в публичном профиле"
    }

    Projects {
        bigint id PK
        bigint author_telegram_user_id
        string github_repository_url
        string programming_language "Java, Python, ..."
        string roadmap_project "HANGMAN, SIMULATION, ..., OTHER"
        bigint added_timestamp "Unix timestamp в секундах момента добавления проекта"
    }

    Profiles ||--o{ Profiles_Details : has
    Profiles ||--o{ Achievements : has

```

Индексы:
- Unique композитный индекс на колонки `details_name`, `detail_value` таблицы `Profiles_Details`
- Индекс по `Projects.author_telegram_user_id` для поиска проектов по автору
- Unique индекс на `Projects.github_repository_url` для проверки уникальности проекта
- Unique составной индекс на колонки `profile_id`, `achievement_type` таблицы `Achievements` 

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

### Получение публичного профиля текущего пользователя

`GET /api/profile`

Ответ в случае успеха: `200 OK`. Тело:

```
{
  "github_profile_url": "https://github.com/zhukovsd",
  "telegram_url": "https://t.me/zhukovsd",
  "achievements": [
    {
      "type": "FIRST_PROJECT",
      "earned_timestamp": 1766342407
    }
  ]
}
```

`achievements` - публично доступные ачивки текущего пользователя.

Коды ошибок:

- 500 - неизвестная ошибка
- 404 - отсутствие профиля для текущего пользователя

### Получение публичного профиля другого пользователя

`GET /api/profile/${:id}`

Ответ в случае успеха: `200 OK`. Тело аналогично профилю текущего пользователя.

Коды ошибок:

- 500 - неизвестная ошибка
- 404 - отсутствие профиля для данного пользователя

### Обновление публичного профиля текущего пользователя

Метод поддерживает передачу одного или нескольких полей профиля. Ачивки управляются отдельно.

`PATCH /api/profile`

Тело запроса (`Content-Type: application-json`)
```
{
  "github_profile_url": "..."
}
```

Валидация:
- Ссылка на GitHub профиль должна начинаться с "https://github.com/" и содержать валидный GitHub username после последнего слеша
- Ссылка на Telegram профиль должна начинаться с "https://t.me/" и содержать валидный Telegram username после последнего слеша

Ответ в случае успеха: `200 OK`. Тело - текущее состояние профиля пользователя (со всеми полями, а не только теми, что были переданы в PATCH запросе).

Коды ошибок:

- 400 - ошибки валидации (невалидные поля или значения)
- 404 - профиль текущего пользователя не существует
- 500 - неизвестная ошибка

### Получение списка всевозможных ачивок

Метод возвращает список всех поддерживаемых системой ачивок, с информацией о том получена ли каждая из них текущим пользователем.

`GET /api/achievements`

Ответ в случае успеха: `200 OK`. Тело:

```
[
  {
    "type": "FIRST_PROJECT",
    "earned_timestamp": 1766342407, // timestamp >0 означает, что ачивка получена
    "description": "Сдан первый проект!",
    "publicly_visible": true
  },
  {
    "type": "PROJECTS_SPRINTER",
    "earned_timestamp": 0, // timestamp 0 означает, что ачивка не получена
    "description": "Все проекты быстрее чем за полгода",
    "publicly_visible": true
  },
  {
    "type": "BILINGUAL",
    "earned_timestamp": 1766342407, // timestamp >0 означает, что ачивка получена
    "description": "Сданы проекты на 2 разных языках",
    "publicly_visible": false // пример полученной ачивки, которая не будет отображаться в публичном профиле
  }
]
```

### Управление видимостью ачивки в публичном профиле текущего пользователя

`PATCH /api/achievement/type=${:type}`

Тело запроса (`Content-Type: application/json`):

```
{
  "publicly_visible": true
}
```

Проверки:
- Существующий тип ачивки
- У юзера есть ачивка, которую он пытается редактировать

Ответ в случае успеха: `200 OK`. Тело - DTO одной отредактированной ачивки (элемент массива из респонса `GET /api/achievements`).

Коды ошибок:

- 400 - ошибки валидации (невалидный тип)
- 404 - профиль текущего пользователя не существует
- 403 - у юзера нет ачивки, которую  он пытается редактировать
- 500 - неизвестная ошибка

### Внутренний эндпоинт для импорта профилей

Используется при импорте пользователей из Google Spreadsheet. Поддерживает создание и обновление профиля с деталями.

`POST /api/profile/internal/profile`

Тело запроса (`Content-Type: application-json`)
```
{
  "telegram_user_id": 1,
  "details": {
    "github_profile": "https://github.com/zhukovsd",
    "telegram_url": "https://t.me/zhukovsd"
  }
}
```

Ответ в случае успеха: `201 Created` если профиль был создан, `200 OK` если профиль был обновлён.

Коды ошибок:

- 400 - ошибки валидации (невалидные поля)
- 500 - неизвестная ошибка

### Внутренний эндпоинт поиска профиля по ссылке на GitHub аккаунт

`GET /api/profile/internal/profile/by-github-profile-url?url=${:url}`

Ответ в случае успеха: `200 OK`, тело:

```
{
  "telegram_user_id": 1,
  "details": {
    "github_profile": "https://github.com/zhukovsd",
    "telegram_url": "https://t.me/zhukovsd"
  }
}
```

Коды ошибок:

- 400 - ошибки валидации запроса (например, переданная ссылка не является ссылкой на GitHub профиль)
- 500 - неизвестная ошибка
- 404 - профиль не найден

## Kafka

### Consumer для топика `auth.user.created`

Используется для иницилизации профиля пользователя в таблице `Profiles`.

Payload сообщения:
```
{
  "telegram_user_id": bigint
}
```

### Consumer для топика `projects.project.created`

Consumer group - `profile-service-cg`.

[Описание формата](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-projectsprojectcreated) сообщения.
