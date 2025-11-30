# Системная аналитика - учёт проектов

## Необходимый контекст

- [Бизнес аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/projects-bookkeeping.md) учёта проектов

## Добавление проекта через Telegram Mini App

### Шаги

- Пользователь добавляет проект через форму в Mini App, указывая ссылку на GitHub репозиторий, язык, ссылку на деплой (если есть)
- Mini App совершает POST запрос `/api/projects/project`, тело содержит введённые пользователем данные и его Telegram user id. Gateway направляет запрос к Project Service
- Project Service сохраяет проект в свою SQL БД
- Project Service запрашивает у Profile Service данные пользователя (ссылку на Telegram и GitHub профили) пользователя - GET `/api/profile/internal/profile?telegram_user_id=${:id}`. 
- Project Service формирует Kafka сообщение для топика `projects.project.created`. Тело содержит введённые пользователем данные, его Telegram user id, ссылки на Telegram и GitHub профили. Консьюмеры:
  - Telegram Bot формирует Telegram пост и публикует его в чат
  - Data Importer добавляет в Google таблицу новый проект
  - Profile Service пересчитывает бейджы (ачивки) пользователя, связанные с исполнением проектов
 
  ```mermaid
  sequenceDiagram
    participant MiniApp as Mini App
    participant ProjectService as Project Service
    participant SQLDB as SQL БД (Projects)
    participant ProfileService as Profile Service
    participant TelegramBot as Telegram Bot
    participant DataImporter as Data Importer

    MiniApp ->> ProjectService: POST /api/projects/project<br/>(данные проекта + telegram_user_id)

    ProjectService ->> SQLDB: Сохранение проекта в БД
    SQLDB -->> ProjectService: OK

    ProjectService ->> ProfileService: GET /api/profile/internal/profile<br/>?telegram_user_id=${id}
    ProfileService -->> ProjectService: Профиль пользователя<br/>(Telegram, GitHub)

    ProjectService ->> ProjectService: Формирование события project.created<br/>(данные проекта + ссылки профилей)

    ProjectService ->> TelegramBot: Отправка события project.created<br/>→ сформировать пост и опубликовать в чат

    ProjectService ->> DataImporter: Отправка события project.created<br/>→ добавить проект в Google таблицу

    ProjectService ->> ProfileService: Отправка события project.created<br/>→ пересчитать бейджи пользователя

  ```
