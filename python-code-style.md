# Python Code Style

[Java Code Style](./code-style.md)

## Общее

Проект должен использовать **Python 3.12**.
Синтаксис и возможности более новых версий Python использовать очевидно нельзя.

[Black](https://github.com/psf/black) обязателен для форматирования Python-кода.
Новый код и изменённый код должны быть отформатированы Black.

Если очень нужно отформатировать что-то вручную, без black,
то можно поставить комментарий на отключение форматирования, пример в `env.py` файле.

[basedpyright](https://github.com/detachhead/basedpyright) желателен для статической проверки типов.

## Нейминг

Дефолт Python conventions:

* `snake_case` - функции, методы, локальные переменные
* `PascalCase` - классы.
* `SCREAMING_SNAKE_CASE` — константы и переменные окружения.
* `_private_name` — внутренние/private объекты/функции, если это необходимо.

Примеры:

```python
async def process_project(project_id: int) -> None:
    pass


class TelegramRenderer:
    pass


MAX_MESSAGE_LENGTH = 4096

def _internal_helper(arg: str) -> None
    pass
```

## Логирование

Для получения logger'a:

```python
import logging

log = logging.getLogger(__name__)
```

Для логирования предпочтительно использовать параметризованные сообщения:

```python
log.error("Failed to process project: %s", project_id)
```

а не:

```python
log.error(f"Failed to process project: {project_id}")
```

## Типы

Очень желательно не плодить ошибки/варнинги для типов.

В текущем коде, такие ошибки есть в местах где нужно сильно заморочиться чтобы
правильно расставить типы, в таком случае типы лучше не ставить.

**Не** использовать deprecated формы:

~`List[str]`~ -> `list[str]`
~`Dict[str, int]`~ -> `dict[str, int]`
~`Optional[str]`~ -> `str | None`

Новые функции должны иметь типы для аргументов и возвращаемого значения:

```python
def find_project(project_id: int) -> Project | None:
    pass
```

Для async-функций:

```python
async def process_project(project_id: int) -> None:
    pass
```

## Импорты

Все импорты из самого проекта должны использовать абсолютный `src`.

Правильно:

```python
from src.config import constants
from src.config.env import TELEGRAM_BOT_TOKEN
from src.handler import util
```

Неправильно:

```python
from config import constants
from ..config import constants
from .util import something
```

## Переменные окружения

Для переменных с допустимым дефолтом, он указывается непосредственно при чтении:

```python
SEND_PROJECTS_TO_CHAT: bool = (
    os.getenv("SEND_PROJECTS_TO_CHAT", "false").lower() == "true"
)
```

Если переменная окружения **обязательна для работы приложения**, надо добавить ассерт:

```python
assert TELEGRAM_BOT_TOKEN is not None, (
    "TELEGRAM_BOT_TOKEN environment variable is not set"
)
```

## Ассерты

`assert` используется для проверки условий, которые должны быть истинными
в корректно работающем приложении>

Например:

```python
chat = update.effective_chat

assert chat is not None, "Chat cannot be None"
```

После этого кода можно безопасно считать `chat` существующим.

Смысл ассертов:

> Если это условие не выполняется, приложение находится в недопустимом состоянии
> и продолжать нормальную работу нельзя

`assert` не нужны для обычной обработки пользовательского ввода или других ожидаемых ошибок.

Например, это неправильно:

```python
assert user_input in allowed_commands
```

Здесь нужно обычное условие:

```python
if user_input not in allowed_commands:
    return
```
