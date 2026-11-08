# Data Importer

## Необходимый контекст

- [Бизнес аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/data-import.md) импорта данных
- [Системная аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/data-import.md) импорта данных


## Стек

- Spring Boot 3
- [Google API Java Client](https://github.com/googleapis/google-api-java-client) для работы с Google Sheets API
- Spring Kafka
- Spring Data JDBC
- Liquibase


## Взаимодействия

Входящие:

- REST эндпоинты
- Kafka

Исходящие:

- Запросы к REST эндпоинту [`POST /api/auth/internal/user`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/auth-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D1%8F-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D0%B5%D0%B9) Auth Service для создания пользователя
- Запросы к REST эндпоинту [`POST /api/profile/internal/profile`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/profile-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B8%D0%BC%D0%BF%D0%BE%D1%80%D1%82%D0%B0-%D0%BF%D1%80%D0%BE%D1%84%D0%B8%D0%BB%D1%8F) Profile Service для создания профиля
- Запросы к REST эндпоинту [`POST /api/project/internal/project`](http://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BF%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B0-%D0%B8%D0%B7-telegram-%D0%B1%D0%BE%D1%82%D0%B0-%D0%B8%D0%BB%D0%B8-data-importer) Project Service для создания реализации проекта студента
- Запросы к REST эндпоинту [`POST /api/project/internal/review`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D1%80%D0%B5%D0%B2%D1%8C%D1%8E-%D0%B8%D0%B7-data-importer) Project Service для создания ревью
- Запросы к REST эндпоинту [`POST /api/mentor/internal/mentor`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/mentor-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BC%D0%B5%D0%BD%D1%82%D0%BE%D1%80%D0%B0) Mentor Service для создания ментора
- Запросы к REST эндпоинту [`POST /api/mentor/internal/guaranteed-review`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/mentor-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D1%86%D0%B5%D0%BD%D1%8B-%D0%BD%D0%B0-%D0%B3%D0%B0%D1%80%D0%B0%D0%BD%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D0%B5-%D1%80%D0%B5%D0%B2%D1%8C%D1%8E) Mentor Service для создания условия на гарантирванное ревью
- Запросы к REST эндпоинту [`POST /api/interval-repetition/internal/question`](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/interval-repetition-service/index.md#%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9-%D1%8D%D0%BD%D0%B4%D0%BF%D0%BE%D0%B8%D0%BD%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B8%D0%BC%D0%BF%D0%BE%D1%80%D1%82%D0%B0-%D0%B2%D0%BE%D0%BF%D1%80%D0%BE%D1%81%D0%B0) Interval Repetition Service для создания вопроса и связанной с ним категории
 для создания пользователя



## Схема БД

```mermaid
erDiagram
    ImportType ||--o{ Import_Execution : "id = type_id"

    ImportType {
        bigint id PK
        string code "USERS, PROFILES, PROJECTS, ... QUESTIONS"
    }

    Import_Execution {
        bigint id PK
        bigint type_id FK
        string status "RUNNING, FAILED, SUCCESS"
        bigint started_at "Unix timestamp в секундах момента запуска импорта"
        bigint finished_at "Unix timestamp в секундах момента окончания импорта"
        int success_count "Количество успешно импортированых сущностей"
        int error_count "Количество сущностей которое не удалось импортировать"
        string source_read_status "FAILED, SUCCESS"
    }
```
Индексы:
- Индекс по `Import_Execution.status` для фильтрации по статусу

## Схема REST API

Для всех методов передаются кастомные заголовки запроса с ролями пользователя.

Все методы требуют наличия роли `ADMIN`.

### Ответ в случае ошибки

Актуально для всех методов.

Код должен соответствовать ситуации (перечислено ниже), тело:

```json
{
  "message": "Текст ошибки"
}
```

### Админский эндпоинт запуска импорта

`POST /api/data-importer/import`

Запускает асинхронный импорт данных указанного типа.

Тело запроса (`Content-Type: application/json`):

```json
{
  "type": "USERS"
}
```

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

- `403` - у пользователя нет роли `ADMIN`;
- `409` - импорт указанного типа уже выполняется;
- `409` - не выполнены условия для запуска импорта указанного типа;
- `500` - неизвестная ошибка.

### Админский эндпоинт получения статуса конкретного импорта

`GET /api/data-importer/imports/{id}`

Возвращает текущее состояние конкретного запуска импорта.

Ответ в случае успеха: `200 OK`. Тело (`Content-Type: application/json`):

```json
{
  "id": 125,
  "type": "USERS",
  "status": "SUCCESS",
  "startedAt": 1761834600,
  "finishedAt": 1761834867,
  "successCount": 498,
  "errorCount": 2,
  "sourceReadStatus": "SUCCESS"
}
```

Правила:

- Если импорт ещё выполняется, поле `finishedAt` имеет значение `null`, поле `status` сохраняет состояние `RUNNING`.
- `sourceReadStatus` содержит результат чтения источника данных. Может иметь два варианта значения:
    - Если источник данных прочитан успешно: `SUCCESS`
    - Если получить данные из источника не удалось: `FAILED`
- Статус `FAILED` в теле ответа характеризует результат выполнения самого импорта. При успешном получении информации о запуске HTTP-код ответа остаётся `200 OK`.

Коды ошибок:

- `400` - передан не валидный параметр пути `id`, например, отрицательный, 
- `403` - у пользователя нет роли `ADMIN`
- `404` - запуск импорта с указанным `id` не найден
- `500` - неизвестная ошибка

## Kafka

### Consumer для топика `projects.project.created`

Consumer group - `data-importer-cg`.

[Описание формата](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-projectsprojectcreated) сообщения.

Используется для добавления записи о проекте в Google Spreadsheet таблицу. 

[Системная аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/functionality/projects-bookkeeping.md) процесса.

### Consumer для топика `projects.project.updated`

Consumer group - `data-importer-cg`.

[Описание формата](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-projectsprojectupdated) сообщения.

Используется, чтобы после смены ссылки на репозиторий реализации Google Spreadsheet таблица не расходилась с БД. Таблица ссылается на проект по ссылке на репозиторий, поэтому без обновления в ней остаётся старая ссылка, повторный импорт проектов создаст дубликат по старой ссылке, а импорт ревью не найдёт проект.

Изменение ссылки на репозиторий доступно пользователям через запрос `PATCH /api/project/projects/${:id}`.

Обработка:

- Если `previous_github_repository_url` равна `github_repository_url` - сообщение игнорируется (изменилась только ссылка на деплой, её в таблице нет)
- Лист "Projects": во всех рядах, где колонка E (ссылка на репозиторий) совпадает со старой ссылкой, перезаписываются колонки D (имя репозитория), E (ссылка на репозиторий), F (GitHub username) и G (ссылка на GitHub профиль). Значения вычисляются из новой ссылки так же, как при добавлении ряда по `projects.project.created`. Остальные колонки не трогаем, формулы в H и I пересчитаются сами
- Лист "Reviews": во всех рядах, где колонка D (ссылка на репозиторий проекта) совпадает со старой ссылкой, D заменяется на новую ссылку
- Лист "Спонсируемые ревью": во всех рядах, где колонка B совпадает со старой ссылкой, B заменяется на новую ссылку (на неё опирается формула «есть ревью» в колонке H листа "Projects")
- Ссылки сравниваются без учёта регистра, пробелов по краям и завершающего `/` - часть рядов внесена ботом и вручную
- Если на листе "Projects" нет ряда со старой ссылкой:
  - есть ряд с новой ссылкой - изменение уже применено (повторная доставка), сообщение считается обработанным
  - нет ни того, ни другого - ошибка, `warn` в лог, сообщение пропускается
- Тип источника не проверяется - изменение реализации возможно только с фронтенда

### Consumer для топика `reviews.review.created`

Consumer group - `data-importer-cg`.

Используется для добавления записи о ревью в Google Spreadsheet таблицу. 

Payload сообщения - https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-reviewsreviewcreated.
