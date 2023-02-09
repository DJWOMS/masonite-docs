# Facades в Masonite
Facades — это простой способ получить доступ к 
классам [Service Container](/architecture/service-container), не создавая их из Service Container.

## Обзор
Facades — это просто ярлык для разрешения ключа в Service Container. 
Вместо того чтобы делать так:

```py
application.make("mail").send()
```

Вы можете сделать:

```py
from masonite.facades import Mail

Mail.send()
```

Чтобы импортировать любые встроенные facades, вы можете импортировать их из пространства имен 
`masonite.facades`:

```py
from masonite.facades import Request

#..
    def show(self):
        avatar = Request.input('avatar_url')
    #..
```

## Встроенные facades

Masonite поставляется с несколькими facades, которые вы можете использовать "из коробки":

- `Auth`
- `Broadcast`
- `Config`
- `Dump`
- `Gate`
- `Hash`
- `Loader`
- `Mail`
- `Notification`
- `Queue`
- `Request`
- `Response`
- `Session`
- `Storage`
- `View`
- `Url`

## Создание собственных facades
Создать свой facade просто:

```py
from masonite.facades import Facade


class YourFacade(metaclass=Facade):
    key = 'container_key'
```

Затем импортируйте и используйте его:

```py
from app.facades import YourFacade


YourFacade.method()
```

Чтобы воспользоваться подсказкой типов при использовании facade (например, в редакторе VSCode), 
вы должны создать файл `.pyi` рядом с вашим файлом facade.

Затем в этом facade вы должны объявить свои типизированные методы. Вот частичный пример подсказки 
типов Mail facade:

```py
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from ..mail import Mailable


class Mail:
    """Обрабатывайте отправку электронных писем из Masonite с помощью разных драйверов."""

    def mailable(mailable: "Mailable") -> "Mail":
        ...
    def send(driver: str = None):
        ...
```

Заметить, что:

- Код методов отсутствует, заменен на `...`.
- Методы не принимают `self` в качестве первого аргумента. Потому что при вызове facade мы на самом 
деле не создаем его экземпляр (даже если мы получаем экземпляр объекта, привязанный к контейнеру). 
Это позволяет иметь правильное поведение подсказки типов.
