# Системная аналитика - учёт ревью

## Необходимый контекст

- [Бизнес аналитика](https://github.com/it-mentor-community-platform/meta/blob/main/business-analytics/functionality/reviews-bookkeeping.md) учёта ревью

## Добавление ревью через Telegram Mini App

### Шаги

- Пользователь добавляет ревью через форму в Mini App, указывая ссылку на ревью, и отревьювленный проект
- Mini App совершает POST запрос `POST /api/project/review`. Gateway направляет запрос к Project Service
- Project Service сохраняет ревью в свою SQL БД
- Project Service формирует Kafka сообщение для топика `reviews.review.created`. Тело содержит информацию о ревью и проекте
  - Telegram Bot формирует пост о добавлении нового ревью и публикует его в чат сообщества
  - Telegram Bot формирует уведомление о сделанном ревью и отправляет его автору проекта
  - Data Importer добавляет в Google таблицу новый проект
  - Profile Service пересчитывает бейджи (ачивки) пользователя, связанные с написанием ревью
