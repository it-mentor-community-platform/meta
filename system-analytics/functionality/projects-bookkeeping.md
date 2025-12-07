# Системная аналитика - учёт проектов

## Необходимый контекст

- [Бизнес аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/projects-bookkeeping.md) учёта проектов

## Добавление проекта через Telegram Mini App

### Шаги

- Пользователь добавляет проект через форму в Mini App, указывая ссылку на GitHub репозиторий, язык, какой проекта роадмапа сдаётся, ссылку на деплой (если есть)
- Mini App совершает POST запрос `/api/projects/project`, тело содержит введённые пользователем данные, Telegram user id можно узнать, посмотрев в subject JWT токена. Gateway направляет запрос к Project Service
- Project Service сохраяет проект в свою SQL БД. Дата добавления проекта - текущий момент времени
- Project Service формирует Kafka сообщение для топика `projects.project.created`. Тело содержит введённые пользователем данные, его Telegram user id, ссылки на Telegram и GitHub профили, источник проекта (mini app). Консьюмеры:
  - Telegram Bot формирует Telegram пост и публикует его в чат
  - Data Importer добавляет в Google таблицу новый проект
  - Profile Service пересчитывает бейджи (ачивки) пользователя, связанные с исполнением проектов
 
  ```mermaid
  sequenceDiagram
    participant MiniApp as Mini App
    participant ProjectService as Project Service
    participant SQLDB as SQL БД (Projects)
    participant ProfileService as Profile Service
    participant TelegramBot as Telegram Bot
    participant Chat as Чат
    participant DataImporter as Data Importer

    MiniApp ->>+ ProjectService: POST /api/projects/project<br/>(данные проекта в теле +<br/>telegram_user_id в токене)

    ProjectService ->> SQLDB: Сохранение проекта в БД
    ProjectService -->>- MiniApp: 201 Created

    ProjectService ->> ProjectService: Формирование события project.created<br/>(данные проекта + ссылки профилей + источник проекта)

    ProjectService ->> TelegramBot: Отправка события project.created
    TelegramBot ->> Chat: формирование поста и публикация

    ProjectService ->> DataImporter: Отправка события project.created<br/>→ добавить проект в Google таблицу

    ProjectService ->> ProfileService: Отправка события project.created<br/>→ пересчитать бейджи пользователя
  ```

## Добавление проекта через Telegram Bot

### Шаги

- Пользователь публикует ссылку на проект в чате сообщества, указывая ссылку на GitHub репозиторий
- Администратор вызывает команду `/addproject ${:language} ${:project_name}` 
- В Telegram Bot'е срабатывает handler команды `/addproject` и он вызывает эндпоинт `POST /api/projects/internal/project`, тело содержит введённые пользователем данные и его Telegram user id
- Project Service сохраяет проект в свою SQL БД. Дата добавления проекта - текущий момент времени
- Project Service формирует Kafka сообщение для топика `projects.project.created`. Тело содержит введённые пользователем данные, его Telegram user id, ссылки на Telegram и GitHub профили, источник проекта (Telegram bot). Консьюмеры:
  - Telegram Bot игнорирует сообщение потому что источник проекта - Telegram bot
  - Data Importer добавляет в Google таблицу новый проект
  - Profile Service пересчитывает бейджи (ачивки) пользователя, связанные с исполнением проектов

```mermaid
sequenceDiagram
    actor User as Пользователь
    actor Admin as Администратор
    participant Chat as Чат
    participant TelegramBot as Telegram Bot
    participant ProjectService as Project Service
    participant SQLDB as SQL БД (Projects)
    participant ProfileService as Profile Service
    participant DataImporter as Data Importer

    User ->> Chat: Публикует ссылку на проект<br/>в чате сообщества (GitHub repo)
    Admin ->> Chat: Команда /addproject ${language} ${project_name}

    Chat ->>+ TelegramBot: Обработка команды /addproject<br/>+ извлечение данных проекта и telegram_user_id
    TelegramBot ->>+ ProjectService: POST /api/projects/internal/project<br/>(данные проекта + telegram_user_id)

    ProjectService ->> SQLDB: Сохранение проекта в БД
    ProjectService -->>- TelegramBot: 201 Created
    TelegramBot -->>- Chat: Публикация в чат результата вызова команды

    ProjectService ->> ProjectService: Формирование события project.created<br/>(данные + ссылки профилей + source=Telegram bot)

    ProjectService ->> TelegramBot: Событие project.created<br/>(source = Telegram bot)
    TelegramBot ->> TelegramBot: Игнорирует событие<br/>(источник = Telegram bot)

    ProjectService ->> DataImporter: Событие project.created<br/>→ добавить проект в Google-таблицу
    DataImporter ->> DataImporter: Добавление строки<br/>в Google-таблицу с проектом

    ProjectService ->> ProfileService: Событие project.created<br/>→ пересчитать бейджи пользователя
    ProfileService ->> ProfileService: Пересчёт бейджей<br/>по проектам пользователя
```

## Добавление проекта через Data Importer

### Шаги

- Администратор запускает импорт проектов через `POST /api/data-importer/start-projects-import`
- Data Importer читает проекты из [таблицы](https://docs.google.com/spreadsheets/d/1E66YrdvO7B_j0Ykge-JJDMtB1RfKhIzN_SsO7UPDbrU/edit?gid=0#gid=0) (лист "Projects")
- Для каждого проекта, Data Importer делает запрос к Profile Service, чтобы найти пользователя по GitHub профилю - `GET /api/profile/internal/profile/by-github-profile-url?url=${:url}`. Из ответа извлекается Telegram user id автора
- Data Importer вызывает эндпоинт `POST /api/projects/internal/project`, тело содержит введённые пользователем данные и его Telegram user id и timestamp добавления проекта (извлекается из Google таблицы)
- Project Service сохраяет проект в свою SQL БД. Дата добавления проекта - из запроса
- Project Service формирует Kafka сообщение для топика `projects.project.created`. Тело содержит введённые пользователем данные, его Telegram user id, ссылки на Telegram и GitHub профили, источник проекта (Data Importer). Консьюмеры:
  - Telegram Bot игнорирует сообщение потому что источник проекта - Data Importer
  - Data Importer игнорирует сообщение потому что источник проекта - Data Importer
  - Profile Service пересчитывает бейджи (ачивки) пользователя, связанные с исполнением проектов

```mermaid
sequenceDiagram
    actor Admin as Администратор
    participant DataImporter as Data Importer
    participant GoogleSheet as Google Sheets<br/>(лист "Projects")
    participant ProjectService as Project Service
    participant SQLDB as SQL БД (Projects)
    participant ProfileService as Profile Service
    participant TelegramBot as Telegram Bot

    Admin ->>+ DataImporter: POST /api/data-importer/start-projects-import<br/>Запуск импорта проектов

    DataImporter ->>+ GoogleSheet: Чтение списка проектов<br/>из таблицы "Projects"
    GoogleSheet -->>- DataImporter: Данные проектов

    loop Для каждого проекта
        DataImporter ->>+ ProfileService: GET /api/profile/internal/profile/by-github-profile-url<br/>?url=${url}
        ProfileService -->>- DataImporter: Telegram user id автора

        DataImporter ->>+ ProjectService: POST /api/projects/internal/project<br/>(данные проекта + telegram_user_id)

        ProjectService ->> SQLDB: Сохранение проекта в БД
        ProjectService -->>- DataImporter: 201 Created

        ProjectService ->> ProjectService: Формирование события project.created<br/>(source = Data Importer)

        ProjectService ->> TelegramBot: Событие project.created<br/>(source = Data Importer)
        TelegramBot ->> TelegramBot: Игнорирует событие<br/>(источник = Data Importer)

        ProjectService ->> DataImporter: Событие project.created<br/>(source = Data Importer)
        DataImporter ->> DataImporter: Игнорирует событие<br/>(источник = Data Importer)

        ProjectService ->> ProfileService: Событие project.created<br/>→ пересчитать бейджи пользователя
        ProfileService ->> ProfileService: Пересчёт бейджей<br/>по проектам пользователя
    end
```
