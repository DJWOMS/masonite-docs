# Request Masonite

Запрос и ответ в Masonite работают вместе, чтобы сформировать ответ,
который браузер может прочитать и отобразить. Класс Request используется
для получения любых входящих файлов cookie, заголовков, URL-адресов,
paths, методов запроса и других входящих данных.

## Cookies

!!! note
    Хотя классы запроса и ответа имеют заголовки и куки-файлы, в
    большинстве случаев при получении куки и заголовков их следует
    извлекать из класса Request. При настройке заголовков и файлов cookie
    это следует делать в классе Response.

Чтобы получить файлы cookie:
```py
from masonite.request import Request


def show(self, request: Request):
    request.cookie('Accepted-Cookies')
```
Это извлечет cookie из заголовков входящего запроса.

Вы также можете установить файлы cookie по запросу:
```py
from masonite.request import Request


def show(self, request: Request):
    request.cookie('Accepted-Cookies', 'True')
```

!!! note
    Обратите внимание, что установка файлов cookie по запросу НЕ
    возвращает файл cookie как часть ответа и, следовательно, НЕ сохраняет
    файл cookie от запроса к запросу.

## Заголовки

Хотя класс запроса и ответа имеют заголовки, и файлы cookie, в
большинстве случаев при получении cookie и заголовков их следует
получать из класса Request. При настройке заголовков и cookie это
следует делать в классе Response.

Чтобы получить заголовок запроса:

```py
from masonite.request import Request


def show(self, request: Request):
    request.header('X-Custom')

```
Вы также можете установить заголовок для Request:

```py
from masonite.request import Request


def show(self, request: Request):
    request.header('X-Custom', 'value')
```

!!! note
    Обратите внимание, что установка заголовков в запросе НЕ вернет
    заголовок как часть ответа.


## Пути (paths)

Текущий URI запроса можно получить следующим образом:
```py
from masonite.request import Request


def show(self, request: Request):
    request.get_path() #== /dashboard/1
```
Вы можете получить текущий метод запроса:
```py
from masonite.request import Request


def show(self, request: Request):
    request.get_request_method() #== PUT
```
Чтобы проверить, содержит ли текущий путь схему в стиле glob:
```py
from masonite.request import Request


def show(self, request: Request):
    # URI: /dashboard/1/users
    request.contains("/dashboard/*/users") #== True
```
Вы можете получить текущий субдомен хоста:
```py
from masonite.request import Request


def show(self, request: Request):
    # URI: work.example.com
    request.get_subdomain() #== work
```
Получить текущий хост:
```py
from masonite.request import Request


def show(self, request: Request):
    # URI: work.example.com
    request.get_host() #== example.com
```

## Входные данные (inputs)

Входные данные могут быть получены из любого метода запроса. В Masonite
получение входных данных одинаково, независимо от метода запроса.

Чтобы получить inputs:
```py
from masonite.request import Request


def show(self, request: Request):
    # GET /dashboard?user_id=1
    request.input('user_id') #== 1
```
Если inputs не существует, вы можете передать значение по умолчанию:
```py
from masonite.request import Request


def show(self, request: Request):
    # GET /dashboard
    request.input('user_id', 5) #== 5
```
Чтобы получить все входные данные из запроса:
```py
from masonite.request import Request


def show(self, request: Request):
    request.all() #== {"user_id": 1, "name": "John", "email": "john@masoniteproject.com"}
```
Или мы можем получить только некоторые входные данные:
```py
from masonite.request import Request


def show(self, request: Request):
    request.only("user_id", "name") #== {"user_id": 1, "name": "John"}
```

#### Получение данных из словаря
Если данные в виде словаря, вы можете получить доступ к вложенным данным
двумя способами. Возьмем следующий пример кода:
```json
"""
Request Payload:
{
  "user": {
    "id": 1,
    "addresses": [
      {"id": 1, "street": "A Street"},
      {"id": 2, "street": "B Street"}
    ],
    "name": {
      "first": "user",
      "last": "A"
    }
  }
}
"""
```
Получить доступ, как обычно:
```py
request.input('user')['name']['last'] # A
```
Для простоты вы можете использовать запись через точку:
```py
request.input('user.name.last') # A
```
Вы также можете использовать подстановочный знак `*`, чтобы получить все
значения из списка словаря:
```py
request.input('user.addresses.*.id') # [1, 2]
```

## Параметры

Параметры маршрута - это параметры указанные в маршруте и полученные из URL-адреса:

Если у вас есть такой маршрут:
```py
Route.get('/dashboard/@user_id', 'DashboardController@show')
```
Вы можете получить значение `@user_id` следующим образом:
```py
from masonite.request import Request


def show(self, request: Request):
    # GET /dashboard?user_id=1
    request.param('user_id') #== 1
```

## Пользователь

Для получения пользователя воспользуйтесь непосредственно классом Request, 
если пользователь вошел в систему:
```py
from masonite.request import Request


def show(self, request: Request):
    # GET /dashboard?user_id=1
    request.user() #== <app.User.User>
```
Если пользователь не аутентифицирован, то для этого параметра будет
установлено значение `None`.


## IP-адрес

Получить IP-адрес (ipv4), с которого был сделан запрос, путем добавления
`IpMiddleware` к HTTP middleware:
```py
# Kernel.py

from masonite.middleware import IpMiddleware


class Kernel:

    http_middleware = [
        # ...
        IpMiddleware
    ]
```
Затем вы можете использовать `ip()`:

```py
def show(self, request: Request):
    request.ip()
```
IP адрес извлекается из различных заголовков HTTP-запроса:

-   HTTP_CLIENT_IP
-   HTTP_X_FORWARDED_FOR
-   HTTP_X_FORWARDED
-   HTTP_X_CLUSTER_CLIENT_IP
-   HTTP_FORWARDED_FOR
-   HTTP_FORWARDED
-   REMOTE_ADDR

IP помощник будет возвращать разные заголовки.

Используемые заголовки и их порядок можно настроить, переопределив
`IpMiddleware` и изменив `headers`:
```py
class CustomIpMiddleware(IpMiddleware):

    headers = [
        "HTTP_CLIENT_IP",
        "REMOTE_ADDR"
    ]
```
