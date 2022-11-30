# Обработка ошибок в Masonite

Обработка ошибок Masonite основана на классе `ExceptionHandler`, который
отвечает за обработку всех исключений, создаваемых вашим приложением.

## Глобальный обработчик исключений

Все исключения обрабатываются классом `ExceptionHandler`, который привязан
к Service Container с помощью ключа `exception_handler`.

Этот обработчик умеет принять решения о том, как обрабатывать исключения
в зависимости от типа исключения, типа среды, типа контента, принятого
запросом и настроенных обработчиков исключений.

Обработчик по умолчанию имеет один драйвер `Exceptionite`,
который отвечает за обработку ошибок при разработке, предоставляя много
информации для помощи в отладке вашего приложения.

### Режим отладки

Когда включен режим отладки, все исключения обрабатываются через страницу отладки Exceptionite HTML.

Если этот параметр отключен, страницы ошибок по умолчанию
`errors/500.html`, `errors/404.html`, ` errors/403.html` отображаются в
зависимости от типа ошибки.

!!! note warning "Важно!"
    Никогда не разворачивайте приложение в рабочей среде с включенным
    режимом отладки! Это может привести к раскрытию некоторых
    конфиденциальных данных конфигурации и переменных окружения.

### Жизненный цикл

Когда возникает исключение, оно будет перехвачено `ExceptionHandler`.
Затем произойдет следующий цикл:

#### В режиме разработки

1.  Будет запущено событие `masonite.exception.SomeException`.
2.  Будет использоваться конкретный `ExceptionHandler`, если он существует для данного исключения.
3.  Затем `Exceptionite` обработает ошибку, отобразив ее в консоль.
    -   С ответом Exceptionite JSON, если принятый контент `application/json`.
    -   Со страницей ошибки Exceptionite HTML.

#### В производстве

1.  Будет запущено событие `masonite.exception.SomeException`.
2.  Будет использоваться конкретный `ExceptionHandler`, если он существует для данного исключения. 
    В этом случае `ExceptionHandler` по умолчанию больше не будет обрабатывать исключение.
3.  Если исключение является исключением HTTP, оно будет обработано `HTTPExceptionHandler`, который отвечает за
    выбор шаблона ошибки (`errors/500.html, errors/404.html,
    errors/403.html`) и его отображение.
4.  Если исключение является исключением `Renderable`, оно будет отображаться соответствующим образом.
5.  В противном случае это исключение еще не обработано и будет обрабатываться, как исключение HTTP с кодом состояния 500 Server
    Error.

## Сообщить об исключениях

Чтобы поднять исключение, вы должны просто вызвать его как любое исключение Python:

```py
def index(self, view: View):
    user = User.find(request.param("id"))
    if not user:
        raise RouteNotFoundException("User not found")
```
Существуют различные типы исключений.

### Простые исключения

Простые исключения будут обрабатываться как ошибка сервера 500, если для
них не определены пользовательские обработчики исключений.

### Исключения HTTP

Исключения HTTP - это стандартные исключения, использующие часто коды
состояния HTTP, такие, как 500, 404 или 403.

Эти исключения будут отображаться `HTTPExceptionHandler` с соответствующим
кодом состояния и шаблоном ошибки по умолчанию, если он существует в
`errors/`.

!!! note
    Обратите внимание, что в режиме отладки этот шаблон не будет
    отображаться, а вместо него будет отображаться страница ошибок
    `Exceptionite` по умолчанию.

В Masonite включены следующие исключения HTTP:

-   AuthorizationException (403)
-   RouteNotFoundException (404)
-   ModelNotFoundException (404)
-   MethodNotAllowedException (405)

В проекте Masonite по умолчанию существующими шаблонами ошибок
являются `errors/404.html`, `errors/403.html` и `errors/500.html`. Их можно
настроить.

Вы также можете создать пользовательское исключение HTTP, установив
`is_http_exception=True` и определив методы `get_response()`, `get_status()` и `get_headers()`.

```py
class ExpiredToken(Exception):
    is_http_exception = True

    def __init__(self, token):
        super().__init__()
        self.expired_at = token.expired_at

    def get_response(self):
        return "Expired API Token"

    def get_status(self):
        return 401

    def get_headers(self):
        return {"X-Expired-At": self.expired_at}
```

Когда возникает вышеуказанное исключение, по умолчанию Masonite будет
искать шаблон ошибки `errors/401.html` в папке шаблонов проекта и
отображать его с кодом 401. Содержимое метода `get_response()` будет
передано в качестве переменной контекста `message` в шаблон.

Если этого шаблона не существует, ответ HTML будет непосредственно
содержимым метода `get_response()` с кодом 401.


### Рендер исключений

Если вам нужна большая гибкость для обработки вашего исключения без
использования `HTTPExceptionHandler`, вы можете просто добавить к нему
метод `get_response()`. Этот метод будет указан в качестве первого
аргумента `response.view()`, чтобы вы могли отображать простую строку или
свой собственный шаблон.

```py
class CustomException(Exception):

    def __init__(self, message=""):
        super().__init__(message)
        self.message = message

    def get_response(self):
        return self.message
```

При вызове этого исключения в вашем коде, Masonite будет знать, что он
должен отобразить его, вызвав `get_response()`, и будет отображаться в
виде строки содержащую сообщение.

Обратите внимание, что вы также можете установить код состояния HTTP,
добавив, как и в случае исключений HTTP, `get_status()`.

```py
class CustomException(Exception):

    def __init__(self, message=""):
        super().__init__(message)
        self.message = message

    def get_response(self):
        return self.message

    def get_status(self):
        return 401
```

### Существующие обработчики исключений

-   HTTPExceptionHandler
-   DumpExceptionHandler

Вы можете создать свои собственные обработчики исключений, чтобы
переопределить то, как Masonite обрабатывает исключение или добавить
новое поведение.

### Добавление новых обработчиков

Обработчик - это простой класс с методом `handle()`, который Masonite
будет вызывать, когда приложение выдает определенное исключение.

Например, вы можете обрабатывать исключения `ZeroDivisionError`, которые
могут создаваться вашим приложением. Это будет выглядеть так:

```py
class DivideException:

    def __init__(self, application):
        self.application = application

    def handle(self, exception):
        self.application.make('response').view(
            {
                "error": str(exception),
                "message": "You cannot divide by zero"
            }, 
            status=500
        )
```

Имя класса может быть любым. В методе `handle` вы должны вручную вернуть
ответ, используя класс Response.

Затем вам нужно будет зарегистрировать класс в контейнере, используя
определенную привязку ключа. Привязка ключей будет
`{exception_name}Handler`. Вы можете сделать это в файле `Kernel.py`.

Чтобы зарегистрировать пользовательский обработчик исключений для нашего
`ZeroDivisionError`, мы должны создать привязку, которая выглядит
следующим образом:

```py
from app.exceptions.DivideException import DivideException


self.application.bind("ZeroDivisionErrorHandler", DivideException(self.application))
```

> Вы можете добавить эту привязку в свой `AppProvider` или в `Kernel.py`.

Теперь, когда ваше приложение генерирует `ZeroDivisionError`, Masonite
будет использовать ваш обработчик, а не собственные обработчики
исключений.


## Перехватить все исключения

Если вы хотите подключить службу отслеживания ошибок, такую как Sentry
или Rollbar, можно сделать это с помощью событий listeners: каждый
раз, когда возникает исключение, запускается
`masonite.exception.{TheExceptionType}`, что позволяет
запускать любую пользовательскую логику.

Сначала создайте listener для запуска вашей пользовательской логики:

```py
class SentryListener:

    def handle(self, exception_type: str, exception: Exception):
        # process the exception with Sentry
        # ...
```

Затем в Service Provider вам необходимо зарегистрировать этот listener:

```py
class AppProvider(Provider):

    def register(self):
        self.application.make("event").listen("masonite.exception.*", [SentryListener])
```
