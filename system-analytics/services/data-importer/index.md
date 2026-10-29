# Data Importer

## Необходимый контекст

- [Бизнес аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/data-import.md) импорта данных

## Стек

- Spring Boot 3
- [Google API Java Client](https://github.com/googleapis/google-api-java-client) для работы с Google Sheets API

У сервиса нет своей БД.

## Взаимодействия

Входящие:

- REST эндпоинты
- Kafka

Исходящие:

- Запросы к REST эндпоинту [`POST /api/auth/internal/user`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/auth-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D1%8F-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D0%B5%D0%B9) Auth Servic'а для создания пользователя

## ER-диаграмма запуска импорта

### Диаграмма

```mermaid
erDiagram
    ImportType ||--o{ ImportExecution : "имеет запуски импорта"

    ImportType {
        string id PK
        string code
    }

    ImportExecution {
        string id PK
        string type FK
        string status
        datetime startedAt
        datetime finishedAt
        int successCount
        int errorCount
        string sourceReadStatus
    }
```
[Описание сущностей и связей](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/data-importer/er-import-entities-and-relations.md)

## Схема REST API

Для всех методов передаются [кастомные заголовки запроса](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/gateway/index.md#%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D0%BB%D0%B0-security) с Telegram Id и ролями пользователя (но принимаются только необходимые в рамках конкретного запроса).

### Ответ в случае ошибки

Актуально для всех методов.

Код должен соответствовать ситуации (перечислено ниже), тело:
```
{
  "message": "Текст ошибки"
}
```

### Запуск импорта пользователей

`POST /api/data-importer/start-users-import`

Метод требует наличия роли `ADMIN`.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`

### Запуск импорта профилей

`POST /api/data-importer/start-profiles-import`

Метод требует наличия роли `ADMIN`.

Для импорта используется [эндпоинт](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/profile-service/index.md#внутренний-эндпоинт-для-импорта-профиля) `PUT /api/profile/internal/profile`.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`

Обратите внимание, что в Google таблице хранятся Telegram юзернеймы, а при импорте нам нужны ссылки на профиль. Необходимо преобразовать профиль в ссылку, пример `@zhukovsd` -> `https://t.me/zhukovsd`.

### Запуск импорта проектов

`POST /api/data-importer/start-projects-import`

Метод требует наличия роли `ADMIN`.

[Системная аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/projects-bookkeeping.md) процесса.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`

### Запуск импорта коллекции ревью

`POST /api/data-importer/start-reviews-import`

Метод требует наличия роли `ADMIN`.

Для импорта используется [эндпоинт](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D1%80%D0%B5%D0%B2%D1%8C%D1%8E-%D0%B8%D0%B7-data-importer) `POST /api/project/internal/review`.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`

### Запуск импорта менторов

`POST /api/data-importer/start-mentors-import`

Метод требует наличия роли `ADMIN`.

Шаги описаны в [системной аналитике](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/data-import.md#%D0%B8%D0%BC%D0%BF%D0%BE%D1%80%D1%82-%D0%BC%D0%B5%D0%BD%D1%82%D0%BE%D1%80%D0%BE%D0%B2) импорта данных.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`

### Запуск импорта гарантированных ревью

`POST /api/data-importer/start-guaranteed-reviews-import`

Метод требует наличия роли `ADMIN`.

Шаги описаны в [системной аналитике](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/data-import.md#%D0%B8%D0%BF%D0%BC%D0%BF%D0%BE%D1%80%D1%82-%D0%B3%D0%B0%D1%80%D0%B0%D0%BD%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D1%8B%D1%85-%D1%80%D0%B5%D0%B2%D1%8C%D1%8E) импорта данных.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`

### Запуск импорта вопросов

`/api/data-importer/start-questions-import`

Метод требует наличия роли `ADMIN`.

Шаги описаны в [системной аналитике](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/data-import.md#%D0%B8%D0%BF%D0%BC%D0%BF%D0%BE%D1%80%D1%82-%D0%B3%D0%B0%D1%80%D0%B0%D0%BD%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D1%8B%D1%85-%D1%80%D0%B5%D0%B2%D1%8C%D1%8E) импорта данных.

Ответ в случае успеха: `200 OK`.

Коды ошибок:
- 401 - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`)
- 403 - у пользователя нет роли `ADMIN`


## Kafka

### Consumer для топика `projects.project.created`

Consumer group - `data-importer-cg`.

[Описание формата](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-projectsprojectcreated) сообщения.

Используется для добавления записи о проекте в Google Spreadsheet таблицу. 

[Системная аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/projects-bookkeeping.md) процесса.

### Consumer для топика `reviews.review.created`

Consumer group - `data-importer-cg`.

Используется для добавления записи о ревью в Google Spreadsheet таблицу. 

Payload сообщения - https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-reviewsreviewcreated.

[data-importer-rest-api.md](https://github.com/user-attachments/files/34144441/data-importer-rest-api.md)

## Схема REST API

Для всех методов передаются кастомные заголовки запроса с Telegram Id и ролями пользователя (но принимаются только необходимые в рамках конкретного запроса).

Все методы требуют наличия роли `ADMIN`.

### Ответ в случае ошибки

Актуально для всех методов.

Код должен соответствовать ситуации (перечислено ниже), тело:

```json
{
  "message": "Текст ошибки"
}
```

### Запуск импорта пользователей

`POST /api/data-importer/start-users-import`

Запускает асинхронный импорт пользователей.

Перед запуском проверяется отсутствие выполняющегося импорта типа `USERS`.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 125,
  "type": "USERS",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт пользователей уже выполняется;
- `500` - неизвестная ошибка.

### Запуск импорта профилей

`POST /api/data-importer/start-profiles-import`

Запускает асинхронный импорт профилей.

Метод требует наличия роли `ADMIN`.

Для импорта используется эндпоинт `PUT /api/profile/internal/profile`.

Перед запуском проверяется отсутствие выполняющегося импорта типа `PROFILES`.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 126,
  "type": "PROFILES",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт профилей уже выполняется;
- `500` - неизвестная ошибка.

### Запуск импорта проектов

`POST /api/data-importer/start-projects-import`

Запускает асинхронный импорт проектов.

Метод требует наличия роли `ADMIN`.

Перед запуском проверяется:

- отсутствие выполняющегося импорта типа `PROJECTS`;
- наличие успешно завершённого импорта `USERS`;
- наличие успешно завершённого импорта `PROFILES`.

Если условия запуска не выполнены, новый `ImportExecution` не создаётся.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 127,
  "type": "PROJECTS",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт проектов уже выполняется;
- `409` - не выполнены условия для запуска импорта проектов;
- `500` - неизвестная ошибка.

### Запуск импорта коллекции ревью

`POST /api/data-importer/start-reviews-import`

Запускает асинхронный импорт коллекции ревью.

Метод требует наличия роли `ADMIN`.

Для импорта используется эндпоинт `POST /api/project/internal/review`.

Перед запуском проверяется отсутствие выполняющегося импорта типа `REVIEWS`.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 128,
  "type": "REVIEWS",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт ревью уже выполняется;
- `500` - неизвестная ошибка.

### Запуск импорта менторов

`POST /api/data-importer/start-mentors-import`

Запускает асинхронный импорт менторов.

Метод требует наличия роли `ADMIN`.

Перед запуском проверяется отсутствие выполняющегося импорта типа `MENTORS`.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 129,
  "type": "MENTORS",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт менторов уже выполняется;
- `500` - неизвестная ошибка.

### Запуск импорта гарантированных ревью

`POST /api/data-importer/start-guaranteed-reviews-import`

Запускает асинхронный импорт гарантированных ревью.

Метод требует наличия роли `ADMIN`.

Шаги описаны в системной аналитике импорта данных.

Перед запуском проверяется отсутствие выполняющегося импорта типа `GUARANTEED_REVIEWS`.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 130,
  "type": "GUARANTEED_REVIEWS",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт гарантированных ревью уже выполняется;
- `500` - неизвестная ошибка.

### Запуск импорта вопросов

`POST /api/data-importer/start-questions-import`

Запускает асинхронный импорт вопросов.

Метод требует наличия роли `ADMIN`.

Перед запуском проверяется отсутствие выполняющегося импорта типа `QUESTIONS`.

В случае успешного запуска создаётся новый `ImportExecution` со статусом `RUNNING`.

Ответ в случае успеха: `202 Accepted`.

В header `Location` возвращается эндпоинт для получения состояния созданного запуска:

```text
/api/data-importer/imports/{id}
```

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 131,
  "type": "QUESTIONS",
  "status": "RUNNING"
}
```

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт вопросов уже выполняется;
- `500` - неизвестная ошибка.

### Получение статуса конкретного импорта

`GET /api/data-importer/imports/{id}`

Метод требует наличия роли `ADMIN`.

Возвращает текущее состояние конкретного запуска импорта.

`id` - идентификатор `ImportExecution`.

Ответ в случае успеха: `200 OK`.

Тело ответа (`Content-Type: application/json`):

```json
{
  "id": 125,
  "type": "USERS",
  "status": "SUCCESS",
  "startedAt": "2026-10-29T14:30:00",
  "finishedAt": "2026-10-29T14:34:27",
  "successCount": 498,
  "errorCount": 2,
  "sourceReadStatus": "SUCCESS"
}
```

Возможные значения `status`:

- `RUNNING` - импорт выполняется;
- `SUCCESS` - импорт завершён успешно;
- `FAILED` - импорт завершён с ошибкой.

Если импорт ещё выполняется, `finishedAt` имеет значение `null`.

Пример:

```json
{
  "id": 125,
  "type": "USERS",
  "status": "RUNNING",
  "startedAt": "2026-10-29T14:30:00",
  "finishedAt": null,
  "successCount": 320,
  "errorCount": 2,
  "sourceReadStatus": "SUCCESS"
}
```

`sourceReadStatus` содержит результат чтения источника данных.

Если источник данных прочитан успешно:

```text
SUCCESS
```

Если получить данные из источника не удалось:

```text
FAILED
```

Пример импорта, завершившегося ошибкой при чтении источника:

```json
{
  "id": 132,
  "type": "USERS",
  "status": "FAILED",
  "startedAt": "2026-10-29T14:30:00",
  "finishedAt": "2026-10-29T14:30:04",
  "successCount": 0,
  "errorCount": 0,
  "sourceReadStatus": "FAILED"
}
```

Если источник был успешно прочитан, но импорт завершился ошибкой на последующих этапах:

```json
{
  "id": 133,
  "type": "PROJECTS",
  "status": "FAILED",
  "startedAt": "2026-10-29T15:00:00",
  "finishedAt": "2026-10-29T15:02:15",
  "successCount": 120,
  "errorCount": 1,
  "sourceReadStatus": "SUCCESS"
}
```

Статус `FAILED` в теле ответа характеризует результат выполнения самого импорта. При успешном получении информации о запуске HTTP-код ответа остаётся `200 OK`.

Коды ошибок:

- `401` - пользователь неавторизован (в запросе отсутствует заголовок `X-Telegram-User-Id`);
- `403` - у пользователя нет роли `ADMIN`;
- `404` - запуск импорта с указанным `id` не найден;
- `500` - неизвестная ошибка.

