# Аутентификация в Masonite

Masonite делает аутентификацию очень простой.

## Создание базовой системы аутентификации

В Masonite есть команда для создания базовой системы аутентификации. Вы можете использовать 
это как отправную точку для добавления аутентификации в ваше приложение. Эта команда создаст 
контроллеры, шаблоны и файлы для email.

**Если вы хотите внедрить собственную аутентификацию с нуля, вы можете перейти к разделам ниже.**

Сначала выполните команду, чтобы создать новые файлы:

```python
python craft auth
```

Затем добавьте `routes` аутентификации в файл маршрутов:

```py linenums="1" hl_lines="1 9" title="routes/web.py"
from masonite.authentication import Auth
from masonite.routes import Route


ROUTES = [
  # routes
]

ROUTES += Auth.routes()
```

Вы можете перейти по url `/login` или `/register` для реализации вашей аутентификации.

## Конфигурация

Конфигурация аутентификации Masonite довольно проста:

```py linenums="1" title="config/auth.py"
from app.User import User


GUARDS = {
    "default": "web",
    "web": {"model": User},
    "password_reset_table": "password_resets",
    "password_reset_expiration": 1440,  # В минутах. 24 часа. None, если отключено
}
```

## Вход

Вы можете попытаться войти в систему, используя класс `Auth` и метод `attempt`:

```py linenums="1" title="app/controllers/auth/LoginController.py"
from masonite.controllers import Controller
from masonite.authentication import Auth
from masonite.request import Request


class LoginController(Controller):
    def login(self, auth: Auth, request: Request):
      user = auth.attempt(request.input('email'), request.input("password"))
```

Если попытка увенчается успехом, пользователь будет аутентифицирован и результатом будет 
модель пользователя.

Если попытка не удалась, результатом будет `None`.

Если вы знаете первичный ключ модели, вы можете попробовать найди по `id`:
```py linenums="1" title="app/controllers/auth/LoginController.py"
from masonite.controllers import Controller
from masonite.authentication import Auth
from masonite.request import Request


class LoginController(Controller):
    def login(self, auth: Auth, request: Request):
        user = auth.attempt_by_id(1)
```

## Выход:

```py linenums="1" title="app/controllers/auth/LoginController.py"
from masonite.controllers import Controller
from masonite.authentication import Auth
from masonite.request import Request


class LoginController(Controller):
    def logout(self, auth: Auth):
        user = auth.logout()
```

## Пользователь

Вы можете получить текущего аутентифицированного пользователя:

```py linenums="1" title="app/controllers/auth/LoginController.py"
from masonite.controllers import Controller
from masonite.authentication import Auth
from masonite.request import Request


class LoginController(Controller):
    def login(self, auth: Auth, request: Request):
        user = auth.user() #== <app.User.User>
```

Если пользователь не аутентифицирован, возвращается `None`.

## Маршруты

Быстро зарегистрировать несколько маршрутов, используя класс `auth`:

```py linenums="1" title="routes/web.py"
from masonite.authentication import Auth
from masonite.routes import Route


ROUTES = [
  # routes
]

ROUTES += Auth.routes()
```

Это зарегистрирует следующие маршруты:

| URI | Описание |
| -----------| ----------------------------------------|
| GET /login | Отображает форму входа для пользователя |
| POST /login | Попытки входа в систему для пользователя |
| GET /home | Домашняя страница пользователя после успешной попытки входа в систему |
| GET /register | Отображает регистрационную форму для пользователя |
| POST /register | Сохранил размещенную информацию и создал нового пользователя |
| GET /password_reset | Отображает форму сброса пароля |
| POST /password_reset | Попытки сбросить пароль пользователя |
| GET /change_password | Отображает форму для запроса нового пароля |
| POST /change_password | Запрашивает новый пароль |

## Guards

Guards — это инкапсулированная логика для входа в систему, регистрации и получения пользователей. 
Guard использует драйвер `cookie`, который устанавливает в cookie `token`. Который 
используется позже для получения пользователя.

Вы можете переключать защиту на лету, чтобы попытаться выполнить аутентификацию на разных защитах:

```py linenums="1" title="app/controllers/auth/LoginController.py"
from masonite.controllers import Controller
from masonite.authentication import Auth
from masonite.request import Request


class LoginController(Controller):
    def login(self, auth: Auth, request: Request):
        user = auth.guard("custom").attempt(request.input('email'), request.input("password")) #== <app.User.User>
```
