# Разработка API в Masonite
Вы можете без труда добавить поддержку API в ваш проект Masonite. 

Masonite поставляется с поддержкой ответов в формате JSON, а также есть несколько полезных функций 
API для аутентификации и авторизации, которые могут быть полезными. 

По умолчанию в проекте нет этих функций, вам нужно зарегистрировать их.

## В начале
### Provider
Сначала зарегистрируйте `ApiProvider` в вашем проекте путем добавления 
`service provider` в ваш список `PROVIDERS`:

```py linenums="1" title="config/providers.py"
from masonite.api.providers import ApiProvider
#..

PROVIDERS = [
    #..
    ApiProvider
]
```

Будут зарегистрированы необходимые классы, используемые для разработки API. Теперь создайте файл `api.py` 
для добавления ваших маршрутов API:

```py linenums="1" title="routes/api.py"
from masonite.routes import Route


ROUTES = [
    Route.get('/users', 'UsersController@index')
]
```

После этого вы должны добавить привязку контейнера к расположению этого файла. Это можно сделать в 
файле `Kernel.py` внутри метода `register_routes`:

```py linenums="1" hl_lines="7" title="Kernel.py"
...
class Kernel:
    ...
    
    def register_routes(self):
        #..
        self.application.bind("routes.api.location", "routes/api")
        
    ...
```
Любые маршруты внутри этого файла будут сгруппированы внутри стека `api middleware`.

### Модели и миграции
Теперь нужно выбрать модель, которая отвечает за аутентификацию. Обычно это ваша модель `User`. 
Вы должны наследовать класс `AuthenticatesTokens`:

```py linenums="1" title="app/models/User.py"
from masoniteorm.models import Model
from masonite.api.authentication import AuthenticatesTokens


class User(Model, AuthenticatesTokens):
    #..
```

Это позволит модели сохранять токены в таблице. 

После этого добавьте столбец в таблицу `users` для сохранения токена. Здесь 
мы назвали его `api_token`, но он настраивается путем добавления атрибута 
`__TOKEN_COLUMN__` в модель. 

Выполните команду для создания файла миграций:
```py
python craft migration add_api_token_users --table users
```

Файл миграции должен выглядеть следующим образом:
```py linenums="1" hl_lines="11"
"""AddApiTokenUsers Migration."""
from masoniteorm.migrations import Migration


class AddApiTokenUsers(Migration):
    def up(self):
        """
        Run the migrations.
        """
        with self.schema.table("users") as table:
            table.string("api_token").nullable()

    def down(self):
        """
        Revert the migrations.
        """
        with self.schema.table("users") as table:
            pass
```

Теперь выполним миграции, запустив команду:

```py
python craft migrate
```

### Конфигурация
Создайте новый конфигурационный файл API, выполнив команду `api:install`:

```py
python craft api:install
```

Команда создаст новый конфигурационный файл `api.py` в вашей директории с настройками, который 
будет выглядеть так:

```py linenums="1" title="config/api.py"
"""API Config"""
from masonite.environment import env

from app.models.User import User

DRIVERS = {
    "jwt": {
        "algorithm": "HS512",
        "secret": env("JWT_SECRET"),
        "model": User,
        "expires": None,
        "authenticates": False,
        "version": None,
    }
}
```
Здесь импортируется модель `User`. Если у вас другая модели или она находится в другом месте, 
вы можете внести изменения в этот код.

Эта команда также сгенерирует секретный ключ, вы должны сохранить его в переменной окружения с 
названием `JWT_SECRET`. Это будет использовано как “соль” для кодирования и декодирования JWT.

Ключ `authenticates` используется для сверки с базой данных при каждом запросе, чтобы убедиться, 
что токен установлен для пользователя. По умолчанию запросы в базу данных для проверки не выполняются. 
Одно из преимуществ JWT - отсутствие необходимости делать запрос в БД, чтобы проверить пользователя, 
но если вы хотите такое поведение, можно установить значение `authenticates` как `True`.

### Маршруты

Вам нужно добавить маршруты в файл `web.py`, которые могут быть использованы для аутентификации 
пользователей, чтобы выдать им JWT токены.

```py linenums="1" title="routes/web.py"
from masonite.api import Api


ROUTES += [
    #.. web routes
]

ROUTES += Api.routes(auth_route="/api/auth", reauth_route="/api/reauth")
```

Приведенные выше параметры (auth_route, reauth_route) уже установлены по умолчанию, так что если 
вы не хотите их менять, просто не указывайте их.

### Middleware

Поскольку маршруты в файле `api.py` обернуты в `api middleware`, вы должны добавить ключ `api` 
и список `middleware` в `route_middleware` в файле `Kernel.py`:

```py linenums="1" title="Kernel.py"
from masonite.api.middleware import JWTAuthenticationMiddleware
#.. 

class Kernel:

    # ..

    route_middleware = {
        # ..
        "api": [
            JWTAuthenticationMiddleware
        ],
    }
```

`JWTAuthenticationMiddleware` позволит всем маршрутам в этом стеке быть защищенными JWT авторизацией.

По умолчанию, все маршруты в фале `routes/api.py` уже имеют `api middleware`, поэтому нет 
необходимости указывать в ваших маршрутах API.

## Создание своего API
Как только настройка завершена, вы можете приступить к созданию API.

### Маршруты
Одним из способов создания API является использование ресурсов контроллера и маршрута. 

Ресурс контроллера - это контроллер с несколькими методами, используемыми для указания каждого 
действия внутри приложения (например, `users`).

Чтобы создать ресурс контроллера вы можете запустить команду контроллера с флагом `-a`.

```py
python craft controller api/UsersController -a
```

Команда создаст контроллер со следующими методами:

- index

- show

- store

- update

- destroy

Затем можете создать ресурс маршрутов:

```py linenums="1" title="routes/api.py"
from masonite.routes import Route


ROUTES = [
    # ..
    Route.api('users', "api.UserController")
]
```

Таким образом будут созданы приведенные ниже маршруты, которые будут соответствовать вашим методам 
API контроллера.

ТАБЛИЦА МЕТОДОВ

|  Method	| URL 	|  Action	| Route Name 	|
|---|---|---|---|
| GET 	| /users 	| index 	| users.show 	| 
| GET 	| /users/@id 	| show 	| users.show 	| 
| POST 	| /users 	| store 	| users.store 	| 
| PUT/PATCH 	| /users/@id 	| update 	| users.update 	| 
| DELETE 	| /users/@id 	| destroy 	| users.destroy 	| 

### Аутентификация

Маршруты, которые мы добавили ранее, содержат 2 метода аутентификации. Маршрут `/api/auth` может 
быть использован, чтобы получить новый токен аутентификации.

Сначала отправим POST запрос с `username` и `password`, чтобы получить JWT:

```py 
{
    "data": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJleHBpcmVzIjpudWxsLCJ2ZXJzaW9uIjpudWxsfQ.OFBijJFsVm4IombW6Md1RUsN5v_btPwl-qtY-QSTBQ0b7N2pca8BnnT4OjfXOVRrKCWaKUM3QsGj8zqxCD4xJg"
}
```

Затем вы должны отправить этот токен с заголовком `Authorization`:

```py
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJleHBpcmVzIjpudWxsLCJ2ZXJzaW9uIjpudWxsfQ.OFBijJFsVm4IombW6Md1RUsN5v_btPwl-qtY-QSTBQ0b7N2pca8BnnT4OjfXOVRrKCWaKUM3QsGj8zqxCD4xJg 
```

### Повторная аутентификация

Если вы не установили время жизни токена, ключ `expires` в конфигурационном файле, то ваш JWT токены 
будут действительны вечно.

Если вы установите время жизни в `expires`, то время жизни JWT истечет после указанного значения 
(количества минут). Если токен истечет, вы должны будете повторно аутентифицироваться, чтобы получить 
новый токен. Отправьте старый токен, чтобы получить взамен новый.

Это можно сделать, послав POST запрос на `/api/reauth` с ключом `token`, 
содержащего текущий JWT. Будет проверена таблица с токенами и если токен будет найден, сгенерируется 
новый токен.

### Версии

Одна из проблем работы с JWT токенами заключается в том, что есть не так много способов сделать 
токен недействительным. Если создается действительный токен, обычно он остается действительным навсегда.

Один из способов сделать JWT токен недействительным и заставить 
пользователей пройти повторную аутентификацию - указать версию. JWT 
токен, которые прошел аутентификацию, будет содержать номер версии. Когда 
токен проверяется, номер версии токена будет сравнен с номером версии в конфигурационном файле. 
Если они не совпадают, токен будет рассматриваться как недействительный и пользователю придется 
пройти повторную аутентификацию для получения нового токена.

## Загрузка пользователей (Loading Users)

Как только мы сохранили активный `api_token` в таблицу, мы получили возможность извлечь пользователя 
используя `LoadUserMiddleware` и новый стек `guard`. 

Создайте ключ `guard`, значением будет список с `GuardMiddleware`. Добавьте `LoadUserMiddleware` 
в список `api`:

```py linenums="1" hl_lines="7 9-11" title="Kernel.py"
##.. 
class Kernel:
    ##.. 
    route_middleware = {
        "api": [
            JWTAuthenticationMiddleware,
            LoadUserMiddleware
        ],
        "guard": [
            GuardMiddleware
        ],
    }
    ##.. 
```

Наконец, в вашем маршруте или группе маршрутов вы можете указать `guard middleware` и указать 
`guard name`:

```py linenums="1" title="routes/api.py"
from masonite.routes import Route


ROUTES = [
    #..
    Route.get('/users', 'UserController@show').middleware(["guard:jwt"])
]
```
