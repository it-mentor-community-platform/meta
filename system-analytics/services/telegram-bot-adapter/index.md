# Telegram Bot Adapter

## Контекст

- [Системная аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/telegram-bot-integration.md) интеграции нашего бэкенда с Telegram ботом сообщества

## Стек

- Kotlin
- Spring Boot 3
- Spring Data JDBC
- Spring Kafka
- Liquibase

## Взаимодействия

Входящие:
- REST эндпоинты
- Kafka

## Kafka

### Consumer для `projects.project.created`

Используется для уведомления других сервисов о создании нового проекта.

Payload сообщения - https://github.com/it-mentor-community-platform/meta/blob/main/system-analytics/services/project-service/index.md#producer-%D0%B4%D0%BB%D1%8F-%D1%82%D0%BE%D0%BF%D0%B8%D0%BA%D0%B0-projectsprojectcreated
