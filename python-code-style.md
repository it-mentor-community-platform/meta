# Python Code Style

[Java Code Style](./code-style.md)

## Общее

Проект должен использовать **Python 3.12**. Синтаксис и возможности более новых версий Python использовать очевидно нельзя

[Black](https://github.com/psf/black) обязателен для форматирования Python-кода. Новый код и изменённый код должны быть отформатированы Black

Если очень нужно отформатировать что-то вручную без black, то можно поставить комментарий на отключение форматирования, пример в env.py файле

[basedpyright](https://github.com/detachhead/basedpyright) желателен для статической проверки типов

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
TELEGRAM_BOT_TOKEN = "Some string"
```


## Логирование

Для получения logger в каждом модуле:

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

Очень желательно не плодить ошибки/варнинги для типов 

В текущем коде, такие ошибки есть в местах где нужно сильно заморочиться чтобы правильно расставить типы, в таком случае типы лучше не ставить

**Не** использовать deprecated формы:

```python
List[str]
Dict[str, int]
Optional[str]
```

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

## Imports

Все импорты из самого проекта должны использовать абсолютный `src.`

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

## Environment variables

Работа с environment variables централизуется в `src/config/env.py`.

В application code не следует постоянно делать:

```python
os.getenv("TELEGRAM_BOT_TOKEN")
```

Вместо этого значение должно быть получено в `src.config.env` и импортировано:

```python
from src.config.env import TELEGRAM_BOT_TOKEN
```

Environment variables объявляются как module-level constants:

```python
TELEGRAM_BOT_TOKEN: str = os.getenv("TELEGRAM_BOT_TOKEN")
```

Для переменных с допустимым default value default указывается непосредственно при чтении:

```python
SEND_PROJECTS_TO_CHAT: bool = (
    os.getenv("SEND_PROJECTS_TO_CHAT", "false").lower() == "true"
)
```

Проект уже использует именно такой подход.

### Required environment variables

Если environment variable **обязательна для работы приложения**, после её чтения должен существовать invariant, гарантирующий, что значение не `None`:

```python
assert TELEGRAM_BOT_TOKEN is not None, (
    "TELEGRAM_BOT_TOKEN environment variable is not set"
)
```

В `src/config/env.py` проект уже проверяет таким образом обязательные настройки.

Не делайте optional значение обязательным только через type annotation:

```python
TELEGRAM_BOT_TOKEN: str = os.getenv("TELEGRAM_BOT_TOKEN")
```

Сам annotation не гарантирует наличие значения. Runtime invariant должен быть явно проверен через `assert`.

## 7. Assertions и invariants

`assert` используется для проверки **invariants** — условий, которые должны быть истинными в корректно работающем приложении.

Например:

```python
chat = update.effective_chat

assert chat is not None, "Chat cannot be None"
```

После этого кода можно безопасно считать `chat` существующим.

То же относится к configuration:

```python
assert POSTGRES_HOST is not None, (
    "POSTGRES_HOST environment variable is not set"
)
```

Смысл assertion здесь:

> Если это условие не выполняется, приложение находится в недопустимом состоянии и продолжать нормальную работу нельзя.

Используйте `assert` для таких внутренних invariants:

```python
assert user is not None
assert configuration_value is not None
assert required_dependency is initialized
```

Не используйте `assert` для обычной обработки пользовательского ввода или других ожидаемых runtime ошибок.

Например, это неправильно:

```python
assert user_input in allowed_commands
```

Здесь нужно обычное условие:

```python
if user_input not in allowed_commands:
    ...
    return
```

Разница принципиальная:

* `assert` — «это **обязательно должно быть истинно**, иначе состояние программы некорректно»;
* `if` — «это нормальная ситуация, которую программа должна обработать».

## 8. Checklist

Перед созданием PR проверьте:

* [ ] Используется Python 3.12 syntax.
* [ ] Код отформатирован **Black**.
* [ ] Код проходит проверку **basedpyright**.
* [ ] Functions и variables используют `snake_case`.
* [ ] Classes используют `PascalCase`.
* [ ] Constants используют `SCREAMING_SNAKE_CASE`.
* [ ] Новые функции имеют type annotations.
* [ ] Используются современные annotations: `list[str]`, `dict[str, ...]`, `X | None` и т. п.
* [ ] Logger получен через `logging.getLogger(__name__)`.
* [ ] Для application logging не используется `print()`.
* [ ] Imports проекта используют только `src.*`.
* [ ] Environment variables читаются через `src/config/env.py`.
* [ ] Обязательные environment variables проверяются через `assert`.
* [ ] Assertions используются только для invariants, которые обязаны быть истинными.
* [ ] Пользовательский input проверяется через обычный control flow, а не через `assert`.
* [ ] Async-код использует `async`/`await` корректно.
* [ ] Внешние операции имеют необходимую обработку ошибок.
* [ ] Не внесено unrelated formatting/refactoring.
