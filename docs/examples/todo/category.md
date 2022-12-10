# Категории

## Создание контроллера
Все контроллеры по умолчанию расположены в каталоге `app/controllers` и Masonite продвигает идею 
один контролер, один файл. Легко запомнить, где именно находится контроллер, потому что имя файла — 
это контроллер.

Конечно, вы можете перемещать контроллеры куда угодно, но командна `craft` по умолчанию поместит их в 
отдельные файлы.

В большинстве случаев, вы можете создать контроллер с помощью команды `craft`:
```
python craft controller Category
```

Команда `craft` создаст контроллер `app/controllers/CategoryController.py`, который выглядит следующим 
образом:
```py linenums="1" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def show(self, view: View):
        return view.render("")
```
Вы можете заметить, что в контроллере уже есть один метод, `show()`. Данный метод мы рассмотрим позже.

## Создание категорий
Добавим импорты и метод `store()`, он будет использоваться для создания категорий:
```py linenums="1" hl_lines="2 3 6 13-15" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def show(self, view: View):
        return view.render("")

    def store(self, request: Request, response: Response):
        Category.create(name=request.input("name"))
        return response.redirect('/')
```

!!! note warning "Бизнес логика"
    Я считаю, что бизнес логика НЕ должна находиться внутри контроллера и должна быть вынесена в 
    отдельный сервис.
    Но для упрощения примера и ознакомления с фреймворком Masonite, логику напишем в контроллере.

!!! note "Service Container"
    Обратите внимание, что мы сейчас использовали `request: Request` и `response: Response`. 
    Это объекты Request и Response. В этом сила и красота Masonite, и ваше первое знакомство с 
    Service Container. Service Container — чрезвычайно мощная реализация, позволяющая запросить у 
    Masonite объект (в данном случае Request или Response) и получить этот объект. Это называется 
    «внедрением зависимостей», важная концепция для понимания, поэтому обязательно прочитайте 
    [документацию](/architecture/service-container/).

С помощью метода `create()` модели `Category` создадим категорию. В `create()` передаем название столбца и 
данные которые хотим записать. 

Для получения данных из запроса используем метод `input()`.
Masonite не обращает внимание на методы запроса, поэтому для получения данных по запросу `GET`, `POST` 
и т.д. мы используем метод `.input()`.

### Валидация
В данный момент мы не проверяем пришедшие данные. Добавим валидацию.
```py linenums="1" hl_lines="14-16" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def show(self, view: View):
        return view.render("")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()
            
        Category.create(name=request.input("name"))
        return response.redirect('/')
```
На 14-ой строке указываем, что `name` обязателен. Более подробно про валидацию можете прочитать в
[документации](https://docs.masoniteproject.com/features/validation).

Далее если у нас есть ошибки, пользователь будет перенаправлен на страницу с которой был отправлен запрос.

## View
Добавим еще один метод `.create()`, данный метод будет рендерить шаблон добавления категории:
```py linenums="1" hl_lines="13-14" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def show(self, view: View):
        return view.render("")

    def create(self, view: View):
        return view.render("category.create")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()
            
        Category.create(name=request.input("name"))
        return response.redirect('/')
```
Обратите внимание, что здесь мы также "типизируем" класс `View`. Это то, что Masonite называет 
"внедрением зависимостей", о чём говорилось ранее. 

В метод `.render()` передаем путь к шаблону через точку. Далее мы создадим этот шаблон и он будет
доступен по пути `templates/category/create.html`.

## Добавление routes
В файле `routes/web.py` добавим два маршрута.
```py linenums="1" hl_lines="5 6" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/create", "CategoryController@create"),
    Route.post("/create", "CategoryController@store"),
]
```
При `get` запросе будет вызван метод контроллера `create()` и который отобразит страницу.

При `post` запросе будет вызван метод контроллера `store()`, который примет отправленные данные формы.

## Создание html шаблона

Теперь в директории `templates` создадим директорию `category`, а в ней файл `create.html.`

В файл `templates/category/create.html` добавим следующий код:
```html linenums="1" title="templates/category/create.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Title</title>
</head>
<body>
<section>

  <h2>Создать категорию</h2>

  <form action="/create" method="POST">
    {{ csrf_field }}
    <input type="text" name="name" placeholder="Название категории">
    <button class="btn" type="submit">Создать</button>
  </form>

</section>
</body>
</html>
```

Обратите внимание, здесь есть тег `{{ csrf_field }}` под тегом `<form>`. Masonite поставляется с 
защитой от CSRF, поэтому нам нужен токен для отображения скрытого поля с CSRF токеном.

```html linenums="1" hl_lines="4" title="templates/category/create.html"
...
  <h2>Создать категорию</h2>

  <form action="/create" method="POST">
    {{ csrf_field }}
    <input type="text" name="name" placeholder="Название категории">
    <button class="btn" type="submit">Создать</button>
  </form>
...
```
На 4-й строке в атрибуте `action` указываем ссылку куда отправить данные из формы. Данную ссылку
мы описали ранее в `routes/web.py`. Так же указали метод http запроса, используем `POST`.

```html linenums="1" hl_lines="5" title="templates/category/create.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Title</title>
</head>
<body>
<section>

  <h2>Создать категорию</h2>

  <form action="/create" method="POST">
    {{ csrf_field }}
    <input type="text" name="name" placeholder="Название категории">
    <button class="btn" type="submit">Создать</button>
  </form>

</section>
</body>
</html>
```
Здесь указываем ссылку на файл стилей, ниже мы его создадим.

### Статические файлы

Создадим файл `style.css` в директории `storage/static/` и добавим в него следующий код.
```css linenums="1" title="storage/static/style.css"
section {
    width: 700px;
    margin: 0 auto;
    padding: 1px 15px 30px 15px;
    box-shadow: 0 3px 5px 0 grey;
    text-align: center;
}

a {
    text-decoration: none;
}

.list-item {
    display: flex;
    justify-content: space-between;
    margin: 15px 0 5px 0;
    padding: 5px 10px;
    box-shadow: 1px 1px 4px grey;
}

.list-item:hover {
    background-color: #f7f7f7;
}

.btn {
    background-color: #199319;
    color: white;
    padding: 10px 10px;
    text-decoration: none;
    border: none;
    cursor: pointer;
}

.btn:hover {
    background-color: #223094;
}

.btn-del {
    background-color: #e32323;
    color: white;
    padding: 10px 20px;
    text-decoration: none;
}

.btn-del:hover {
    background-color: #c91f1f;
}
```

## Запуск сервера разработки
Выполним команду для запуска сервера разработки:
```
python craft serve
```
И перейдите по адресу `http://localhost:8000/create`

Вы увидите форму, после отправки которой вас перенаправит на `http://localhost:8000/`

## Чтение, редактирование и удаление категорий
Теперь выведем список категорий которые мы создаем. Также сделаю, чтобы можно было 
редактировать и удалять категорию.

### Список категорий

#### Контроллер
В файле `app/controllers/CategoryController.py` добавим следующий код:
```py linenums="1" hl_lines="10-12" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def index(self, view: View):
        categories = Category.all()
        return view.render('category.list', {'categories': categories})
    
    def show(self, view: View):
        return view.render("")

    def create(self, view: View):
        return view.render("category.create")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()
            
        Category.create(name=request.input("name"))
        return response.redirect('/')
```
Здесь мы получаем все категории из БД. Затем указываем какой шаблон рендерить и передаем контекст в 
виде словаря. По ключу словаря будем обращаться к данным в самом шаблоне.

#### Routes
В файле `routes/web.py` добавим url для вывода списка категорий.
```py linenums="1" hl_lines="5" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/", "CategoryController@index"),
    Route.get("/create", "CategoryController@create"),
    Route.post("/create", "CategoryController@store"),
]
```

#### Шаблон html
В директории `templates/category` создадим файл `list.html`.

```html linenums="1" title="templates/category/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>
<section>

  <div class="center">
    <h2>Список категорий</h2>

    <a class="btn" href="{{ route('category_create') }}">Создать категорию</a>
  </div>

  @for category in categories
  <div class="list-item">
    <a href="/task">{{category.name}}</a>
    <a class="edit" href="/single/{{category.id}}">Редактировать</a>
  </div>
  @endfor

</section>
</body>
</html>
```

!!! note success ""
    Masonite использует шаблоны `Jinja2`, поэтому, если вы не понимаете этот шаблон, обязательно 
    ознакомьтесь с [документацией](https://jinja.palletsprojects.com/en/3.0.x/).

!!! note "Наследование шаблонов"
    Jinja2 поддерживает расширение (наследование) шаблонов. Так в Masonite предустановлен базовый шаблон,
    `templates/base.html`.
    В данном руководстве мы не будем использовать наследование шаблонов, для упрощения задачи.

```html linenums="1" hl_lines="14 16 18" title="templates/category/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>
<section>

  <div class="center">
    <h2>Список категорий</h2>

    <a class="btn" href="{{ route('category_create') }}">Создать категорию</a>
  </div>

  @for category in categories
  <div class="list-item">
    <a href="/task">{{category.name}}</a>
    <a class="edit" href="/single/{{category.id}}">Редактировать</a>
  </div>
  @endfor

</section>
</body>
</html>
```

Здесь с помощью шаблонизатора, описываем цикл `for`. Который переберет все переданные категории. 
В фигурных скобках указываем объект категории и через точку обращаемся к атрибуту (столбцу таблицы) 
`name` и `id`.

На строке ТУТ НОМЕР указываем ссылку на список задач, в данный момент у нас еще нет такого url, 
реализуем позже.

На -й строке формируем ссылку на странице редактирования. Ниже реализуем данный функционал.

### Одна категория, редактирование и удаление
Теперь реализуем возможность просматривать одну категорию, чтобы иметь возможность 
редактировать её и удалять.

#### Контроллер одной категории
```py linenums="1" hl_lines="14-16" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def index(self, view: View):
        categories = Category.all()
        return view.render('category.list', {'categories': categories})

    def show(self, view: View, request: Request):
        category = Category.find_or_fail(request.param("id"))
        return view.render('category.single', {'category': category})

    def create(self, view: View):
        return view.render("category.create")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()
            
        Category.create(name=request.input("name"))
        return response.redirect('/')
```

В метод `show()` добавил получение request. 
Затем ищем категорию по `id`, который будет передаваться в url, например, `/single/2`. Здесь `2` и 
есть наш `id`. Если такая категория не будет найдена, мы увидим ошибку 404. Если вы не хотите получать
эту ошибку, можете вызвать метод `find()` у модели.

Затем мы указываем какой шаблон будем использовать и передаем в контекст объект категории.

#### Route одной категории
```py linenums="1" hl_lines="8" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/", "CategoryController@index"),
    Route.get("/create", "CategoryController@create"),
    Route.post("/create", "CategoryController@store"),
    Route.get("/single/@id", "CategoryController@show").name("category_single"),
]

```
Здесь в url указан параметр `id`. Чтобы указать параметр, его нужно прикрепить к символу `@`.

Также я указал имя маршрута, оно используется для получения информации о маршруте в других частях 
проекта. Имя более статично, чем URL-адрес.

#### Шаблон одной категории
В директории `templates/category` создадим файл `single.html`.

```html linenums="1" hl_lines="13 15" title="templates/category/single.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Редактирование категории</title>
</head>
<body>
<section>
  
  <h2>Редактирование категории</h2>
  
  <form action="/update/{{category.id}}" method="POST">
    {{ csrf_field }}
    <input type="text" name="name" value="{{category.name}}">
    <p>
      <button class="btn" type="submit">Сохранить</button>
    </p>
  </form>

  <a class="btn-del" href="/delete/{{category.id}}">Удалить</a>

</section>
</body>
</html>
```

В `action` указал url по которому будет идти отправка формы, контролер и маршрут напишем позже.

В `value` тега `input` передаю значение имени категории. Таким образом форма будет заполнена.

### Контроллер обновления категории
```py linenums="1" hl_lines="34" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def index(self, view: View):
        categories = Category.all()
        return view.render('category.list', {'categories': categories})

    def show(self, view: View, request: Request):
        category = Category.find_or_fail(request.param("id"))
        return view.render('category.single', {'category': category})

    def create(self, view: View):
        return view.render("category.create")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()
            
        Category.create(name=request.input("name"))
        return response.redirect('/')
        
    def update(self, request: Request, response: Response):
        category = Category.find_or_fail(request.param("id"))

        errors = request.validate({"name": "required"})
        if errors:
            return response.redirect(name='category_single', params={"id": request.param("id")})

        category.name = request.input('name')
        category.save()
        return response.redirect('/')
```

Если `name` будет отсутствовать, то пользователь будет перенаправлен на ту же страницу.
Я здесь не использую `back()`, для того чтобы показать как работать с `redirect()`.
Указываем имя маршрута и в параметрах виде словаря передаем `id` категории.

```py linenums="1" hl_lines="30 36 37" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def index(self, view: View):
        categories = Category.all()
        return view.render('category.list', {'categories': categories})

    def show(self, view: View, request: Request):
        category = Category.find_or_fail(request.param("id"))
        return view.render('category.single', {'category': category})

    def create(self, view: View):
        return view.render("category.create")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()
            
        Category.create(name=request.input("name"))
        return response.redirect('/')
        
    def update(self, request: Request, response: Response):
        category = Category.find_or_fail(request.param("id"))

        errors = request.validate({"name": "required"})
        if errors:
            return response.redirect(name='category_single', params={"id": request.param("id")})

        category.name = request.input('name')
        category.save()
        return response.redirect('/')
```
После получения объекта категории, атрибуту `name` присваиваем полученное значение от 
пользователя и сохраняем.

#### Route обновления категории
```py linenums="1" hl_lines="9" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/", "CategoryController@index"),
    Route.get("/create", "CategoryController@create"),
    Route.post("/create", "CategoryController@store"),
    Route.get("/single/@id", "CategoryController@show").name("category_single"),
    Route.post("/update/@id", "CategoryController@update").name("category_update"),
]
```
Для обновления категории добавляем новый маршрут и указываем метод контроллера `update()`. 
Здесь используем метод `post`.

#### Запуск сервера
Теперь можно запустить сервер разработки и проверить как работает редактирование категории.
```
python craft serve
```
Если вы уже создавали категорию, то перейдите по ссылке `http://localhost:8000/single/1`.
У вас должна отобразиться форма с уже заполненными данными. 

Если из поля ввода удалить текст и сохранить, вас должно перенаправить на эту же страницу. 

Вы можете ввести валидные данные и нажать сохранить, категория измениться и вы будете перенаправлены 
на главную страницу.

#### Ссылка на редактирование
Доработаем шаблон списка категорий и укажем ссылку на страницу редактирования.

```html linenums="1" hl_lines="16" title="templates/category/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>

<section>
  <h2>Список категорий</h2>

  <ul class="list">
    @for category in categories
    <li class="list-item">
      <a href="/single/{{category.id}}">{{category.name}}</a>
    </li>
    @endfor
  </ul>
</section>

</body>
</html>
```

### Удаление категории
Настало время реализовать удаление категории.

#### Контроллер удаления категории
В контроллер добавим метод `destroy()` для удаления категории.
```py linenums="1" hl_lines="40-43" title="app/controllers/CategoryController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category


class CategoryController(Controller):
    def index(self, view: View):
        categories = Category.all()
        return view.render('category.list', {'categories': categories})

    def show(self, view: View, request: Request):
        category = Category.find_or_fail(request.param("id"))
        return view.render('category.single', {'category': category})

    def create(self, view: View):
        return view.render("category.create")

    def store(self, request: Request, response: Response):
        errors = request.validate({"name": "required"})
        if errors:
            return response.back()

        Category.create(name=request.input("name"))
        return response.redirect('/')

    def update(self, request: Request, response: Response):
        category = Category.find_or_fail(request.param("id"))

        errors = request.validate({"name": "required"})
        if errors:
            return response.redirect(name='category_single', params={"id": request.param("id")})

        category.name = request.input('name')
        category.save()
        return response.redirect('/')

    def destroy(self, request: Request, response: Response):
        post = Category.find_or_fail(request.param("id"))
        post.delete()
        return response.redirect('/')
```
Здесь ищем категорию и если она существует, то удаляем. Для этого используем метод `delete()`.

#### Маршрут удаления категории
```py linenums="1" hl_lines="10" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/", "CategoryController@index"),
    Route.get("/create", "CategoryController@create"),
    Route.post("/create", "CategoryController@store"),
    Route.get("/single/@id", "CategoryController@show").name("category_single"),
    Route.post("/update/@id", "CategoryController@update").name("category_update"),
    Route.get("/delete/@id", "CategoryController@destroy"),
]
```
Добавляю еще один url для удаления категории. Также будем передавать `id` той категории которую хотим
удалить. Для удаления будем использовать http метод `get`.

#### Шаблон удаления категории
В шаблоне `templates/category/single.html` добавим ссылку на удаление.

```html linenums="1" hl_lines="21" title="templates/category/single.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Редактирование категории</title>
</head>
<body>

<section>
  <h2>Редактирование категории</h2>

  <form action="/update/{{category.id}}" method="POST">
    {{ csrf_field }}
    <input type="text" name="name" value="{{category.name}}">
    <p>
      <button type="submit">Сохранить</button>
    </p>
  </form>
  
  <a href="/delete/{{category.id}}">Удалить</a>

</section>
</body>
</html>
```
Просто добавляем ссылку на url удаления категории и подставляем id категории.

После нажатия по ссылке, категория должна удалиться и вы будете перенаправлены на главную страницу.

#### Ссылка на создание категории
Доработаем шаблон списка категорий `templates/category/single.html` и добавим ссылку на страницу создания категории.

```html linenums="1" hl_lines="13" title="templates/category/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>

<section>
  <h2>Список категорий</h2>
  
  <a href="{{ route('category_create') }}">Создать категорию</a>

  <ul class="list">
    @for category in categories
    <li class="list-item">
      <a href="/single/{{category.id}}">{{category.name}}</a>
    </li>
    @endfor
  </ul>
</section>

</body>
</html>
```

Тут я также добавил просто ссылку, но для построения url использовал `route()`.
В данный метод передаем имя маршрута в виде строки.

Чтобы данную ссылку Masonite смог построить, нужно добавить `name` нашему маршруту создания категории.
```py linenums="1" hl_lines="6" title="routes/web.py"
from masonite.routes import Route


ROUTES = [
    Route.get("/", "CategoryController@index"),
    Route.get("/create", "CategoryController@create").name('category_create'),
    Route.post("/create", "CategoryController@store"),
    Route.get("/single/@id", "CategoryController@show").name("category_single"),
    Route.post("/update/@id", "CategoryController@update").name("category_update"),
    Route.get("/delete/@id", "CategoryController@destroy"),
]
```

Теперь можете запустить сервер и проверить как это работает. 

У вас на главной странице должен выводиться список категорий. Вверху быть ссылка на создание категории.
При переходе на одну из категорий, у вас должна быть возможность ее отредактировать или удалить.

[Часть 5](/examples/todo/task/)
