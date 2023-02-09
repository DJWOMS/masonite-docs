# События в Masonite
Masonite поставляется с функцией событий (events) в стиле «pub and sub», которая позволяет вам подписываться 
на различные события и запускать listeners или дополнительную логику, когда эти события испускаются.

## Создание события
Первым шагом в событиях является создание события для прослушивания.

События — это простые классы, которые вы можете создавать где угодно:

```py
python craft event UserAdded
```

Эта команда создаст простой класс, который мы можем позже использовать.

```py linenums="1" title="app/events/UserAdded.py"
class UserAdded:
    pass
```

Вы также можете запускать события без класса `Event`. Событие будет просто определенным ключом, 
которое вы можете прослушать.

## Создание listener
Listener будет запускать логику, когда событие генерируется. Вы можете создать столько listener, 
сколько захотите и зарегистрировать столько listener на события, сколько вам нужно.

Чтобы создать listener, просто выполните команду:

```py
python craft listener WelcomeEmail
```

Будет создан следующий класс:

```py linenums="1" title="app/listeners/WelcomeEmail.py"
class WelcomeEmail:
    def handle(self, event):
        pass
```

### Метод обработки
Метод `handle` будет выполняться при запуске listener. Он передаст событие в качестве первого параметра 
и любые дополнительные аргументы, которые исходят от события в качестве дополнительных параметров.

## Регистрация событий и listener
После того как ваши события и listener созданы, вам нужно будет зарегистрировать их в классе событий.

Вы можете сделать это через `AppProvider` или [Service Provider](/architecture/service-provider), 
которые вы создадите сами:

```py
from masonite.providers import Provider


class EventsProvider(Provider):
    def register(self):
        self.application.make('event').listen(UserAddedEvent, [WelcomeListener])
```

Вы также можете прослушивать события без классов событий:

```py
from masonite.providers import Provider


class EventsProvider(Provider):
    def register(self):
        self.application.make('event').listen("users.added", [WelcomeListener])
```

Использование строк событий позволяет использовать прослушивание событий с подстановочными знаками. 
Например, если приложение генерирует несколько событий такие как `users.added`, `users.updated` и 
`users.deleted`, вы можете прослушивать все эти события одновременно:

```py
event.listen("users.*", [UsersListener])
```

## Активация событий
Чтобы запустить событие, вы можете использовать метод `fire` из класса `Event`:

```py
from app.events import UserAddedEvent
from masonite.events import Event


class RegisterController:
    def register(self, event: Event):
        # Зарегистрировать пользователя
        event.fire(UserAddedEvent)
```

Чтобы запустить простое событие без класса, вы будете использовать тот же метод:

```py
from app.events import UserAddedEvent
from masonite.events import Event


class RegisterController:
    def register(self, event: Event):
        # ...
        # Зарегистрировать пользователя
        event.fire("users.added", user)
```

## Создание listener приветственных писем
Например, чтобы создать listener, который отправляет электронное письмо:

Сначала создайте listener:

```py
python craft listener WelcomeEmail
```

Чтобы отправить электронное письмо, нам нужно импортировать класс `mailable` и отправить электронное 
письмо с помощью ключа `mail` из контейнера:

```py linenums="1" title="app/listeners/WelcomeEmail.py"
from app.mailables.WelcomeMailable import WelcomeMailable


class WelcomeEmail:
    def handle(self, event):
        from wsgi import application

        application.make("mail").send(
            WelcomeMailable().to('idmann509@gmail.com')
        )
```

Затем вы можете зарегистрировать событие внутри провайдера:

```py linenums="1"
from masonite.providers import Provider


class EventsProvider(Provider):
    def register(self):
        self.application.make('event').listen(UserAddedEvent, [WelcomeListener])
```

Когда вы создаете событие `UserAdded` внутри контроллера или где-то еще в проекте, оно будет 
отправлять это электронное письмо.

Вы можете зарегистрировать столько listener, сколько захотите, просто добавив больше listener 
в список.
