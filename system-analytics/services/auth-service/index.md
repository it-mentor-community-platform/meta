# Auth Service

## Необходимый контекст

- Telegram Mini App init data https://docs.telegram-mini-apps.com/platform/init-data, https://docs.telegram-mini-apps.com/platform/launch-parameters
- JWT (структура, подпись, claims) - https://auth0.com/docs/secure/tokens/json-web-tokens
- [Бизнес аналитика](../../../business-analytics/functionality/authentication-and-authorization.md) аутентификации и авторизации

## Стек

- Spring Boot 3
- Spring Data JDBC
- Liquibase
- JJWT для генерации JWT

## Взаимодействия

Входящие:
- REST эндпоинты

Исходящие:
- Kafka

## Схема БД

```mermaid
erDiagram
    Users {
        int id "Автоинкремент, primary key"
        bigint telegram_user_id "64-bit signed, unique index"
    }

    Roles {
        int id "Автоинкремент, primary key"
        string name "Unique index"
    }

    Users_Roles {
        int user_id "Внешний ключ на Users.id"
        int role_id "Внешний ключ на Roles.id"
    }

    Users ||--o{ Users_Roles : has
    Roles ||--o{ Users_Roles : has
```

Дополнение к схеме - необходим unique композитный индекс на колонки `user_id` и `role_id` таблицы `User_Roles`.

Список ролей перечислен в [бизнес аналитике](../../../business-analytics/functionality/authentication-and-authorization.md) аутентификации и авторизации.

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

### Авторизация через Telegram

`POST /api/auth/by-telegram`

Тело запроса (`Content-Type: application-json`)
```
{
  "initDataRaw": "..."
}
```

Ответ в случае успеха: `200 OK` с JWT токеном, установленном в кастомный заголовок `X-Access-Token`.

Коды ошибок:

- 400 - ошибки валидации (initDataRaw отсутствует или некорректна)
- 500 - неизвестная ошибка

### Авторизация через Dummy метод

`POST /api/auth/by-dummy`

Тело запроса (`Content-Type: application-json`)
```
{
  "user_id": 1, // Users.id, а не Users.telegram_user_id!
  "roles": ["ROLE_1", "ROLE_2"],
  "telegram_username": "zhukovsd"
}
```

Ответ в случае успеха: `200 OK` с JWT токеном, установленном в кастомный заголовок `X-Access-Token`.

Коды ошибок:

- 400 - ошибки валидации (пример - неизвестные системе роли)
- 500 - неизвестная ошибка

### Внутренний эндпоинт для создания пользователей

Используется при импорте пользователей из Google Spreadsheet. Поддерживает создание и обновление пользователей и их ролей.

`POST /api/auth/internal/user`

Тело запроса (`Content-Type: application-json`)
```
{
  "telegram_user_id": 1,
  "roles": ["ROLE_1", "ROLE_2"]
}
```

Ответ в случае успеха: `201 Created` если пользователь был создан, `200 OK` если профиль был обновлён.

Коды ошибок:

- 400 - ошибки валидации (пример - неизвестные системе роли)
- 500 - неизвестная ошибка

### Внутренний эндпоинт для получения списка пользователей по их telegram user id

`GET /api/auth/internal/users`

GET параметры:
- `telegram_user_ids` - список telegram user id пользователей через запятую

```
[
  {
    "telegram_user_id": 1,
    "roles": ["ROLE_1", "ROLE_2"]
  }
]
```

Если один или несколько пользователей для переданных `telegram_user_ids` не найден, мы не включаем его в ответ.

Коды ошибок:

- 400 - ошибки валидации (пример - некорректно сформированный список `telegram_user_ids`)

### Админский эндпоинт для управления ролями пользователей

`PATCH /api/auth/admin/user?telegram_user_id=${:id}`

Тело запроса (`Content-Type: application-json`)
```
{
  "roles": ["ROLE_1", "ROLE_2"]
}
```

[Бизнес-аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/admin-functionality.md#%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D1%80%D0%BE%D0%BB%D1%8F%D0%BC%D0%B8) управления ролями.

Коды ошибок:

- 400 - ошибки валидации (примеры - невалидная роль, невалидный или отсутствующий `telegram_user_id`)
- 403 - у юзера, делающего запрос, нет роли ADMIN; попытка добавить/убрать роль `ADMIN` у себя или другого пользователя
- 404 - юзер не найден

## Kafka

### Producer для топика `auth.user.created`

Сообщение отправляется в топик при создании нового пользователя - при авторизации через Telegram или импорте.

Payload сообщения:
```
{
  "telegram_user_id": bigint,
  "telegram_username": string, // может быть null
  "first_name": string, // может быть null
  "last_name": string  // может быть null
}
```

### Producer для топика `auth.user.updated`

Сообщение отправляется в топик при обновлении пользователя. Сценарии:
- Пользователь авторизовался через Telegram, и его username, first_name, last_name обновились со времени последнего логина.
- Роль существующего пользователя изменилась в результате вызова `PATCH /api/auth/admin/user?telegram_user_id=${:id}`

Payload сообщения:
```
{
  "telegram_user_id": bigint,
  "telegram_username": string, // может быть null
  "first_name": string, // может быть null
  "last_name": string, // может быть null
  "roles": ["ROLE_1", "ROLE_2"]
}
```
