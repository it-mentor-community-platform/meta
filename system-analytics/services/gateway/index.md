# Gateway

## Необходимый контекст

- JWT (структура, подпись, claims) - https://auth0.com/docs/secure/tokens/json-web-tokens
- [Бизнес аналитика](../../../business-analytics/functionality/authentication-and-authorization.md) аутентификации и авторизации

## Стек

- Spring Cloud Gateway
- Spring Security
- JJWT для валидации JWT

## Взаимодействия

Входящие:
- REST запросы к API

Исходящие:
- Прокирование REST запросов в конечные микросервисы

## Схема путей

- `/api` - префикс всех путей
- `/api/auth/**` - запросы передаются в Auth Service
- Если маршрут не найден, отвечаем 404

## Правила Security

- Gateway запрещает запросы к `/internal/**` эндпоинтам всех микросервисов. Данные эндпоинты подразумеваются только для внутреннего использования другими микросервисами, без авторизации. Пример: `/api/auth/sign-in` - разрешено, `/api/auth/internal/users` - запрещено. При запрете отвечаем `403 Forbidden`
- Извлеченные из токена значения передаются низлежащим микросервисам в заголвоках запросов:
  - `X-Telegram-User-Id` - число, Telegram id пользователя
  - `X-User-Roles` - comma-separated строка с ролями. Пример - `ADMIN,MENTOR`
