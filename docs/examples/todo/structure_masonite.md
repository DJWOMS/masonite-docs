# Структура проекта Masonite

**app/** - основная директория проекта, где вы пишите свою логику.

- **/controllers** - это место, где вы обрабатываете запрос и возвращаете ответы, которые видите в 
веб-браузере. Ответы могут быть словарями, списками, html (view) или любым классом, который может 
отображать ответ.
- **/middleware** - (промежуточное программное обеспечение) является чрезвычайно важным аспектом 
веб-приложений, поскольку оно позволяет запускать важный код до или после каждого запроса или 
определенных маршрутов.
- **/models** - это самый простой способ взаимодействия с вашими таблицами. Модель используем для 
запроса данных в таблице или для создания новых записей, извлечения связанных записей между 
таблицами и т.д.
- **/providers** - это ключевые строительные блоки для Masonite. Единственное, что они делают, 
регистрируют вещи в Service Container или запускают логику по запросам.

**config/** - различные настройки проекта.

- **/application.py** - настройки секретного ключа, debug режима и т.д.
- **/auth.py** - настройка авторизации: модель пользователя и т.д.
- **/broadcast.py** - настройки драйвера Pusher.
- **/cache.py** - настройки кеширования для Redis, Memcached и файлового драйвера.
- **/database.py** - настройки базы данных для mysql, postgres, mssql, sqlite.
- **/filesystem.py** - настройки для драйвера хранения файлов, локально или в облаке (S3).
- **/mail.py** - настройки для драйвера отправки почты.
- **/notification.py** - настройки для отправки уведомлений в slack, sms или в БД.
- **/queue.py** - настройки драйверов очередей, async, database и amqp.
- **/security.py** - настройки CORS.
- **/session.py** - настройка сессий, cookie и redis. 
- **/providers.py** содержит список всех Service Providers.

**databases/** - миграции и загрузка демо данных (seed)

**resources/** - статические файлы, например такие как js и css.

**routes/** - ваши маршруты.

**storage/** - билд ваших файлов фронтенда.

**templates/** - ваши html файлы.

**tests/** - ваши тесты.

**.env** - переменные окружения.

**.env.testing** - переменные окружения для тестов.

**craft.py** - этот файл будет использоваться всякий раз, когда вы запускаете команду `python craft`.

**Kernel.py** - файл, который используется для настройки всего вашего проекта.

## Начало

### Routes
Рассмотрим файл `routes/web.py`.
```py hl_lines="4" linenums="1" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/", "WelcomeController@show")
]
```
ROUTES - это список содержит ваши юрл.

```py hl_lines="5" linenums="1" title="routes/web.py" 
from masonite.routes import Route


ROUTES = [
    Route.get("/", "WelcomeController@show")
]
```

Здесь у класса `Route` указываем какой (http) метод мы используем. Затем передаем сам путь и в текстовом 
варианте название класса контроллера и через символ `@` метод который нужно вызвать.

Подробнее о routes можно посмотреть в [документации](https://masonite.pro/basics/routing/).

### Controller
В директории `app/controllers` откройте файл `WelcomeController.py`
```py hl_lines="6" linenums="1" title="app/controllers/WelcomeController.py" 
"""A WelcomeController Module."""
from masonite.views import View
from masonite.controllers import Controller


class WelcomeController(Controller):
"""WelcomeController Controller Class."""

    def show(*self*, view: View):
        return view.render("welcome")
```
Наш класс контроллера должен наследоваться от класса `Controller`.

В самом классе мы описываем методы, которые будут вызываться по определенным url.

Метод `show` принимает `view` - это класс который рендерить шаблоны. 

```py hl_lines="9 10" linenums="1" title="app/controllers/WelcomeController.py" 
"""A WelcomeController Module."""
from masonite.views import View
from masonite.controllers import Controller


class WelcomeController(Controller):
"""WelcomeController Controller Class."""

    def show(*self*, view: View):
        return view.render("welcome")
```

Данный метод вернет готовый html. В метод `.render()` мы передаем название html файла.
Сам шаблон находиться в директории `templates/welcome.html`.

### Чистим
Удалим файл `WelcomeController.py`.

Из списка `ROUTES` в `routes/web.py` удалите `Route.get("/", "WelcomeController@show")`.

```py hl_lines="5" linenums="1" title="routes/web.py" 
from masonite.routes import Route


ROUTES = [

]
```
[Часть 3](/examples/todo/database_architecture/)
