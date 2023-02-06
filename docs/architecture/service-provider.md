# Service Providers в Masonite

Service Providers являются важнейшими "строительными блоками" Masonite. Единственное, 
что они делают - это регистрируют элементы в [Service Container](/architecture/service-container/) 
или запускают логику по запросу. 
Если вы заглянете внутрь файла `config/provider.py`, вы найдете список `PROVIDERS`, который содержит 
все Service Provider, участвующие в сборке фреймворка.

Вы можете создать свой собственный Service Provider и добавить его в список `PROVIDERS`, 
чтобы расширить функционал Masonite или даже удалить некоторые `provider`, если вам не нужен их 
функционал. Если вы создаете свои собственные Service Provider, будет хорошей практикой сделать их 
доступными на PyPi для того, чтобы другие могли установить их в свой фреймворк.


## <a name="creating-a-provider"></a> Создание Provider
Мы можем создать Service Provider, просто используя команду:
```py
python craft provider DashboardProvider
```

Это создаст новый Service Provider, расположенный в `app/providers/DashboardProvider.py`. Новый 
Service Provider будет иметь два простых метода, `register()` и `boot()`. Чуть позже рассмотрим их 
подробнее.

## Реализация Service Provider
Вот несколько архитектурных примеров, которые мы рассмотрим, чтобы познакомить вас с тем как Service 
Providers работают под капотом. Давайте посмотрим на простой provider и изучим его.
```py
from masonite.providers import Provider


class YourProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.bind('User', User)

    def boot(self):
        print(self.application.make('User'))
```
Мы видим здесь простой provider, который регистрирует модель `User` в контейнере. Есть несколько 
ключевых особенностей, которые мы должны рассмотреть более детально.

### Register
Важно, что в нашем методе `register()` мы только связали объекты с контейнером. Когда Provider впервые 
регистрируется в контейнере, метод `register()` запускается и ваши классы регистрируются в контейнере.

### Boot
Метод `boot()` будет иметь доступ ко всему, что зарегистрировано в контейнере. Этот метод
запускается во время запроса и фактически регулируется контейнером. Благодаря этому, мы можем
переписать наш provider следующим образом:

```py
from masonite.provider import ServiceProvider
from app.User import User


class UserModelProvider(ServiceProvider):
    """Binds the User model into the Service Container"""

    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.bind('User', User)

    def boot(self, user: User):
        print(user)
```

!!! note ""
    Эта запись идентична той, что показана ранее. Обратите внимание, что метод `boot()` регулируется контейнером.

## Регистрация Service Provider

Как только вы создали ваш собственный provider, его необходимо зарегистрировать в списке `PROVIDERS`. 
Скорее всего, он находится по адресу `config/providers.py`.
```py
PROVIDERS = [
    #..
    UserModelProvider,
]
```
После регистрации он будет учитывать ваш метод `register()` и метод `boot()` всякий раз при запуске или 
регистрации фреймворка.
