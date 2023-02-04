# Трансляция событий в Masonite (Broadcasting)

Masonite предлагает мощный способ трансляции событий в вашем приложении. Эти события можно 
прослушивать на стороне клиента с помощью Javascript.

Это могут быть новое уведомление, которое можно показать на клиентской стороне без перезагрузки страницы.

Masonite поставляется с одним серверным драйвером: [Pusher](https://pusher.com/channels).

## Конфигурация

### Сервер

Конфигурация на стороне сервера для бродкаста выполняется в файле 
`config/broadcast.py`. На данный момент доступен один драйвер [Pusher](https://pusher.com/channels).

Вы должны создать учетную запись на Pusher Channels, а затем создать приложение 
Pusher в своей учетной записи и получить соответствующие учетные данные **(client, app_id, secret)**
и имя местоположения кластера.

Добавьте их в параметры broadcast pusher.

```py linenums="1" hl_lines="6-13" title="config/broadcast.py"
from masonite.environment import env


BROADCASTS = {
    "default": "pusher",
    "pusher": {
        "driver": "pusher",
        "app_id": "3456678",
        "client": "478b45309560f3456211", # key
        "secret": "ab4229346et64aa8908",
        "cluster": "eu",
        "ssl": False,
    },
}
```

Убедитесь, что вы установили пакет `pusher`.

```shell
pip install pusher
```

### <a name="client"></a>Клиент
Чтобы иметь возможность получать события в браузере, вам необходимо 
установить [Javascript Pusher SDK](https://pusher.com/docs/channels/getting_started/javascript).

Подключить скрипт pusher-js на свою страницу.

```html
<script src="https://js.pusher.com/7.0.3/pusher.min.js"></script>
```

!!! note warning
    Это самый быстрый способ установить Pusher. Но для реального приложения вы часто будете 
    использовать систему [сборки assets](/feature/compiling-assets/#Компиляция), установите SDK 
    Javascript Pusher с помощью `npm install pusher-js`, а затем импортируйте класс Pusher  
    `import Pusher from 'pusher-js';`.

Создайте экземпляр Pusher, настроенный с вашими учетными данными

```javascript
const pusher = new Pusher("478b45309560f3456211", {
  cluster: "eu",
});
```

!!! note warning
    Рекомендуется использовать переменные среды вместо жестко заданных учетных данных на стороне 
    клиента. Если вы используете Laravel Mix для 
    [сборки assets](/feature/compiling-assets/#Компиляция), вам следует добавить к переменным 
    среды префикс `MIX_`.

Теперь вы готовы подписаться на каналы и прослушивать события.

## Создание событий
Трансляция (broadcast) события — это простые классы, наследуемые от `CanBroadcast`. Вы можете использовать 
любой класс, включая классы Masonite Event.

Трансляция события будет выглядеть так:

```py linenums="1"
from masonite.broadcasting import CanBroadcast, Channel


class UserAdded(CanBroadcast):

    def broadcast_on(self):
        return Channel("channel_name")
```

!!! note info
    Обратите внимание, что имя события, передаваемое клиенту, будет именем класса. Здесь это будет 
    `UserAdded`.

## Трансляция событий
Вы можете транслировать событие, используя фасад `Broadcast` или класс `Broadcast` из 
контейнера.

Вы можете легко транслировать, не создавая событие Broadcast

```py
from masonite.facades import Broadcast

Broadcast.channel('channel_name', "event_name", {"key": "value"})
```
Или вы можете транслировать класс событий, созданный ранее

```py
from masonite.facades import Broadcast
from app.broadcasts import UserAdded


broadcast.channel(UserAdded())
```

Вы также можете транслировать на нескольких каналах:

```py
Broadcast.channel(['channel1', 'channel2'], "event_name", {"key": "value"})
```

!!! note info
    Этот тип вещания будет транслировать все каналы как общедоступные. Для реализации **частных каналов** и
    **каналов присутствия** продолжайте читать.

## Прослушивание событий
В этом разделе мы будем использовать клиентский экземпляр pusher [настроенный ранее](#client).

Чтобы прослушивать события на стороне клиента, вы должны сначала подписаться на канал, на котором 
происходят события.

```javascript
const channel = pusher.subscribe("my-channel");
```

Затем вы можете прослушивать события
```javascript
channel.bind("my-event", (data) => {
  // Получении события
});
```

## Типы каналов
В Masonite существуют различные типы каналов.

### <a name="public-channels"></a>Общедоступные каналы
Внутри класса события вы можете указать общедоступный канал. Эти каналы позволяют любому, у кого 
есть подключение, прослушивать события на этом канале:

```py linenums="1"
from masonite.broadcasting import CanBroadcast
from masonite.broadcasting import Channel


class UserAdded(CanBroadcast):

    def broadcast_on(self):
        return Channel("channel_name")
```

### <a name="private-channels"></a>Частные каналы
Частные каналы требуют авторизации пользователя для подключения к каналу. Вы можете использовать 
этот канал для отправки только тех событий, которые пользователь должен прослушивать.

Частные каналы — это каналы, начинающиеся с префикса `private-`. При использовании частных каналов 
префикс будет добавлен автоматически.

Частные каналы могут транслироваться только в том случае, если пользователи вошли в систему. Когда 
канал авторизован, он проверяет, аутентифицирован ли пользователь в настоящее время, прежде чем начать 
трансляцию. Если пользователь не аутентифицирован, он ничего не будет транслировать на этом канале.

```py linenums="1"
from masonite.broadcasting import CanBroadcast
from masonite.broadcasting import PrivateChannel


class UserAdded(CanBroadcast):

    def broadcast_on(self):
      return PrivateChannel("channel_name")
```
Будут генерироваться события на канале `private-channel_name`.

#### <a name="routing"></a>Маршрутизация
На клиентской стороне, когда вы подключаетесь к частному каналу, клиент 
инициирует запрос POST для аутентификации частного канала. Masonite поставляется с этим маршрутом 
аутентификации. 

Всё, что вам нужно сделать, это добавить его в свои маршруты:

```py linenums="1"
from masonite.broadcasting import Broadcast


ROUTES = [
  # Normal route list here
]

ROUTES += Broadcast.routes()
```

Это создаст маршрут, по которому вы сможете аутентифицировать свой частный канал на клиентской стороне. 
Маршрут авторизации будет `/broadcasting/authorize`, но вы можете изменить его:

```py
ROUTES += Broadcast.routes(auth_route="/pusher/user-auth")
```

Pusher ожидает, что маршрут аутентификации будет `/pusher/user-auth`, 
[по умолчанию](https://pusher.com/docs/channels/server_api/authenticating-users/#client-side-setting-the-authentication-endpoint). 

Если вы хотите изменить его, вы можете сделать это при создании экземпляра Pusher.

```javascript
const pusher = new Pusher("478b45309560f3456211", {
  cluster: "eu",
  userAuthentication: {
    endpoint: "/broadcast/auth",
  },
});
```

Вам также нужно будет добавить маршрут `/pusher/user-auth` в исключение CSRF.

```py linenums="1" hl_lines="7" title="app/middlewares/VerifyCsrfToken.py"
from masonite.middleware import VerifyCsrfToken as Middleware


class VerifyCsrfToken(Middleware):

    exempt = [
        '/pusher/user-auth'
    ]
```
Причина этого в том, что broadcast клиент не будет отправлять токен CSRF вместе с POST запросом 
авторизации.

Если вы хотите сохранить защиту CSRF, вы можете узнать больше об этом 
[здесь](https://pusher.com/docs/channels/server_api/authenticating-users/#csrf-protected-authentication-endpoint).

#### <a name="authorizing"></a>Авторизация
Поведение по умолчанию — разрешить всем доступ к любым частным каналам.

Если вы хотите настроить логику авторизации каналов, вы можете добавить свой собственный маршрут 
авторизации broadcast с помощью специального контроллера.

Давайте представим, что вы хотите аутентифицировать каналы для каждого пользователя, что означает, 
что пользователь с идентификатором `1` сможет аутентифицироваться на канале `private-1`, пользователь 
с идентификатором `2` — на канале `private-2` и так далее.

Сначала вам нужно удалить `Broadcast.routes()` из ваших маршрутов и добавить свой собственный маршрут:

```py title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.post("/pusher/user-auth", "BroadcastController@authorize")
    #..
]
```
Затем вам нужно создать собственный контроллер для реализации вашей логики.

```py linenums="1" title="app/controllers/BroadcastController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.broadcasting import Broadcast
from masonite.helpers import optional


class BroadcastController(Controller):

    def authorize(self, request: Request, broadcast: Broadcast):
        channel_name = request.input("channel_name")
        _, user_id = channel_name.split("-")

        if int(user_id) == optional(request.user()).id:
            return broadcast.driver("pusher").authorize(
                channel_name, request.input("socket_id")
            )
        else:
            return False
```

### Каналы присутствия
Каналы присутствия работают точно так же, как частные каналы, за исключением того, что вы можете 
видеть, кто еще находится внутри этого канала. Это отлично подходит для приложений типа чата.

Для каналов присутствия пользователь также должен пройти аутентификацию.

```py linenums="1"
from masonite.broadcasting import CanBroadcast
from masonite.broadcasting import PresenceChannel


class UserAdded(CanBroadcast):

    def broadcast_on(self):
        return PresenceChannel("channel_name")
```
Это будет генерировать события на канале `presence-channel_name`.

#### Маршрутизация
Добавление маршрута аутентификации аналогично [приватным каналам](#routing).

#### Авторизация
Авторизация каналов аналогична [приватным каналам](#authorizing).

## Примеры
Чтобы упростить начало работы с broadcast в Masonite, здесь доступны два небольших примера:

- Отправка общедоступных уведомлений о выпуске приложений всем пользователям
используя [общедоступные каналы](#public-channels).

- Отправка частных предупреждений пользователям-администраторам 
используя [частные каналы](#private-channels).

### Отправка общедоступных уведомлений о выпуске приложений
Для этого нам нужно создать событие `NewRelease` и вызвать это событие на бэкэнде.

```py linenums="1" title="app/broadcasts/NewRelease.py"
from masonite.broadcasting import CanBroadcast, Channel


class NewRelease(CanBroadcast):

    def __init__(self, version):
        self.version = version

    def broadcast_on(self):
        return Channel("releases")

    def broadcast_with(self):
        return {"message": f"Version {self.version} has been released !"}
```

```py
from masonite.facades import Broadcast
from app.broadcasts import NewRelease

Broadcast.channel(NewRelease("4.0.0"))
```
На клиентской стороне нам нужно прослушивать канал `releases` и подписаться на события 
`NewRelease`, чтобы отображать окно предупреждения с сообщением.

```html
<html lang="en">
<head>
  <title>Document</title>
  <script src="https://js.pusher.com/7.0/pusher.min.js"></script>
</head>
<body>
  <script>
    const pusher = new Pusher("478b45309560f3456211", {
      cluster: "eu"
    });

    const channel = pusher.subscribe('releases');
    channel.bind('NewRelease', (data) => {
      alert(data.message)
    })
  </script>
</body>
</html>
```

### Отправка личных оповещений администраторам
Давайте представим, что наша модель пользователя имеет две роли: `basic` и `admin`, и мы 
хотим отправлять оповещения только администраторам. Обычные пользователи не должны иметь права 
подписаться на оповещения.

Чтобы добиться этого на бэкэнде, нам нужно:

- создать собственный маршрут аутентификации для авторизации администраторов только на 
канале `private-admins`.
- создать broadcast `AdminUserAlert` и вызвать это событие.

Давайте сначала создадим маршрут аутентификации и контроллер

```py linenums="1" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.post("/pusher/user-auth", "BroadcastController@authorize")
    #..
]
```

```py linenums="1" title="app/controllers/BroadcastController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.broadcasting import Broadcast
from masonite.helpers import optional


class BroadcastController(Controller):

    def authorize(self, request: Request, broadcast: Broadcast):
        channel_name = request.input("channel_name")
        authorized = True

        # проверка permissions для канала private-admins, иначе авторизуйте все остальные каналы
        if channel_name == "private-admins":
            # убедитесь, что пользователь вошел в систему и является администратором
            if optional(request.user()).role != "admin":
                authorized = False

        if authorized:
            return broadcast.driver("pusher").authorize(
                channel_name, request.input("socket_id")
            )
        else:
            return False
```

```py linenums="1" title="app/broadcasts/UserAlert.py"
from masonite.broadcasting import CanBroadcast, Channel


class AdminUserAlert(CanBroadcast):

    def __init__(self, message, level="error"):
        self.message = message
        self.level = level

    def broadcast_on(self):
        return Channel("private-admins")

    def broadcast_with(self):
        return {"message": self.message, "level": self.level}
```

```py
from masonite.facades import Broadcast
from app.broadcasts import AdminUserAlert


Broadcast.channel(AdminUserAlert("Some dependencies are outdated !", level="warning"))
```

На фронтэнде нам нужно прослушивать канал `private-admins` и подписаться на события 
`AdminUserAlert`, чтобы отображать окно предупреждения с сообщением.

```html
<html lang="en">
<head>
  <title>Document</title>
  <script src="https://js.pusher.com/7.0/pusher.min.js"></script>
</head>
<body>
  <script>
    const pusher = new Pusher("478b45309560f3456211", {
      cluster: "eu"
    });

    const channel = pusher.subscribe('private-admins');
    channel.bind('AdminUserAlert', (data) => {
      alert(`[${data.level.toUpperCase()}] ${data.message}`)
    })
  </script>
</body>
</html>
```

Вы готовы начать трансляцию событий в своем приложении!
