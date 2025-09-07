# Стек проекта

## Backend

- Java/Kotlin для основных backend сервисов
  - Spring Cloud
    - Gateway
  - Spring Boot 3.x
    - Spring Data JDBC, JdbcTemplate
    - Spring Security
    - Spring Kafka
    - Метрики - Micrometer/Actuator
- Python для Telegram бота
- Хранилища
  - Postgres

## Frontend

- VueJS
- Typescript
- TODO библиотека компонентов для базового UI

## Инфрастуктура

### Организационная инфрастуктура

- Задачи - GitHub Projects / GitHub issues
- Ревью - GitHub Pull requests
- Документация и аналитика - markdown файлы в отдельном GitHub репозитории

### Локально

- Docker
- Docker Compose

### Удалённо

- Kubernetes (Managed Kubernetes в облаке DigitalOcean / Selectel / Yandex) 
- Managed Postgres
- Managed Kafka
- Prometheus / Grafana
- Loki
- Cert Manager

### Хранение артефактов

- GitHub Docker Registry

### CI/CD

- GitHub Actions, облачные раннеры для сборки и пуша образов, прогона тестов, деплоя
