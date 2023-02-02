# Авторизация в Masonite
Masonite также предоставляет простой способ авторизации действий пользователя в отношении любого ресурса. 
Это достигается с помощью двух концепций: gates и policies. 

Gates являются проверкой авторизации, которую вы сможете вызывать 
для проверки доступа пользователя. 

Policies — это способ сгруппировать логику авторизации вокруг модели. 

## Gates
Gates — это простые вызываемые элементы, которые определяют, авторизован ли пользователь для 
выполнения данного действия. 
Удобный facade `Gate` позволяет легко управлять gates. 

### Регистрация gates
Gates получают экземпляр пользователя в качестве своего первого аргумента и могут получать 
дополнительные аргументы, такие как экземпляр модели. Вам необходимо определить `Gates` 
в методе `boot()` вашего service provider.

В следующем примере мы добавляем gates, чтобы проверить, может ли пользователь создавать и 
обновлять сообщения. Пользователи могут создавать сообщения, если они являются администраторами 
и могут изменять сообщения, только созданные ими. 

```py linenums="1"
from masonite.facades import Gate


class MyAppProvider(Provider):
    def boot(self):
        Gate.define("create-post", lambda user: user.is_admin)
        Gate.define("update-post", lambda user, post: post.user_id == user.id)
```
Затем вы можете проверить, существует ли `gate` или был ли он зарегистрирован, используя `has()`, 
который возвращает логическое значение: 

```python 
Gate.has("create-post") 
``` 

!!! note info
    Если используется неизвестный `gate`, будет возбуждено исключение GateDoesNotExist. 

### Авторизация Actions
Затем в любом месте кода (часто в контроллере) вы можете использовать эти gates, чтобы проверить, 
авторизован ли **текущий аутентифицированный пользователь** для выполнения данного действия, 
определенного gate. 

Gates предоставляет различные методы для выполнения проверки: `allows()`, `denies()`, `none()`, 
`any()`, `authorize()`, `inspect()`. 

`allows()`, `denies()`, `none()` и `any()` возвращают логическое значение, указывающее, 
авторизован ли пользователь 

```py
if not Gate.allows("create-post"):
    return response.redirect("/")

# ...
post = Post.find(request.input("id"))
if Gate.denies("update-post", post):
    return response.redirect("/")

# ...
Gate.any(["delete-post", "update-post"], post)

# ...
Gate.none(["force-delete-post", "restore-post"], post)
```

`authorize()` не возвращает логическое значение, а вместо этого вызывает исключение 
`AuthorizationException`, которое будет отображаться как HTTP-ответ с кодом состояния 403. 

```py
Gate.authorize("update-post", post) 
# если мы доходим до этой части, пользователь авторизован 
# в противном случае возникает исключение, которое отображается как 403 с содержимым 
# "Action not authorized". 
``` 

Наконец, для лучшего контроля над проверкой авторизации вы можете проанализировать ответ 
с помощью `inspect()`: 

```py
response = Gate.inspect("update-post", post)
if response.allowed():
     # сделай что-нибудь
else:
     # не авторизован, и мы можем получить доступ к сообщению
     Session.flash("errors", response.message())
``` 

### Через модель пользователя 

Класс `Authorizes` можно добавить в вашу модель пользователя, чтобы добавить быструю проверку 
разрешений 
```py
from masonite.authentication import Authenticates
from masonite.authorization import Authorizes


class User(Model, Authenticates, Authorizes):
    #..
``` 
Api авторизации теперь будет доступен для экземпляров модели пользователя:
```py
user.can("delete-post", post)
user.cannot("access-admin")
user.can_any(["delete-post", "force-delete-post"], post)
``` 
Все эти методы получают имя gate в качестве первого аргумента, а затем, если требуется, 
некоторые дополнительные аргументы. 

### С пользователем
Вы можете использовать метод `for_user()`, чтобы выполнить проверку для данного 
пользователя, а не для аутентифицированного пользователя. 

```py
from masonite.facades import Gate


user = User.find(1)
Gate.for_user(user).allows("create-post")
``` 

### Gate Hooks
Во время процесса авторизации gate, могут быть активированы хуки `before` и `after`. 

Хук `before` можно добавить следующим образом: 

```python
# здесь администраторы всегда будут авторизованы на основе логического значения 
# этого ответа 
Gate.before(lambda user, permission: user.role == "admin")
```

Хук `after` работает так же: 

``` python 
Gate.after(lambda user, permission, result: user.role == "admin")
``` 

Если обратный вызов `after` возвращает значение, он будет иметь приоритет над проверкой 
результата gate. 

## Policies 

Policies — это классы, которые организуют логику авторизации вокруг определенной модели. 

### Создание Policies 

Выполните команду craft:
``` 
python craft policy AccessAdmin 
``` 

Вы также можете создать policy с набором предопределенных gates, используя флаг `--model`:
``` 
python craft policy Post --model 
``` 

Модель Policy поставляется с общими действиями, которые мы можем выполнять с моделью: 

```py
from masonite.authorization import Policy


class PostPolicy(Policy):
    def create(self, user):
        return False

    def view_any(self, user):
        return False

    def view(self, user, instance):
        return False

    def update(self, user, instance):
        return False

    def delete(self, user, instance):
        return False

    def force_delete(self, user, instance):
        return False

    def restore(self, user, instance):
        return False
``` 

Вы можете добавлять любые другие методы в свои policy: 

```python 
from masonite.authorization import Policy


class PostPolicy(Policy):
    #.. 

    def publish(self, user):
        return user.email == "admin@masonite.com"
``` 

### Регистрация Policies 

Затем в вашем сервис-провайдере (что касается определения gates) вы должны зарегистрировать 
policies и связать их с моделью: 

```py
from masonite.facades import Gate
from app.models.Post import Post
from app.models.User import User

from app.policies.PostPolicy import PostPolicy
from app.policies.UserPolicy import UserPolicy


class MyAppProvider(Provider):

    #..

    def register(self):
        Gate.register_policies(
            [(Post, PostPolicy), (User, UserPolicy)],
        )
``` 

Пример policy для модели Post может выглядеть следующим образом: 

```py
from masonite.authorization import Policy


class PostPolicy(Policy):
    def create(self, user):
        return user.email == "idmann509@gmail.com"

    def view(self, user, instance):
        return True

    def update(self, user, instance):
        return user.id == instance.user_id
``` 

!!! note info
    Если используется неизвестная policy, будет вызвано исключение `PolicyDoesNotExist`. 


## Авторизация Actions
Затем вы можете использовать методы `Gate` для авторизации действий, определенных в 
ваших policy. 

С ранее определенной `PostPolicy` мы могли бы сделать следующие вызовы:
```py
from masonite.facades import Gate

post = Post.find(1)
Gate.allows("update", post)
Gate.denies("view", post)
Gate.allows("force_delete", post)
``` 

Методы `create()` или `view_any()` не принимают экземпляр модели, поэтому должен быть 
предоставлен класс модели, чтобы механизм Gate мог сделать вывод к какой policy принадлежат 
эти методы. 

```py
Gate.allows("create", Post)
Gate.allows("view_any", Post)
```
