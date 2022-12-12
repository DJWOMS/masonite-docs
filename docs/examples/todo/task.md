# Задачи
Для начала нам нужно создать контроллер для задач. Создадим контроллер сразу с нужными методами, для
этого нужно использовать флаг `-r`.

Выполните команду:
```
python craft controller Task -r
```

### Контроллер задач
Откройте файл `app/controllers/TaskController.py`.

Все нужные нам методы уже присутствуют, это великолепно.
```py linenums="1" title="app/controllers/TaskController.py"
from masonite.controllers import Controller
from masonite.views import View


class TaskController(Controller):
    def index(self, view: View):
        return view.render("")

    def create(self, view: View):
        return view.render("")

    def store(self, view: View):
        return view.render("")

    def show(self, view: View):
        return view.render("")

    def edit(self, view: View):
        return view.render("")

    def update(self, view: View):
        return view.render("")

    def destroy(self, view: View):
        return view.render("")
```
#### Импорт

Сразу добавим нужные нам импорты. Импортируем классы `Request`, `Response` и модели `Category`, `Task`.
```py linenums="1" hl_lines="2 3 6 7"  title="app/controllers/TaskController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category
from app.models.Task import Task


class TaskController(Controller):
    ...
```

Перед тем как писать логику в контроллере добавим маршруты в `ROUTES`.

### Маршруты для задач
Группы маршрутов — отличный способ сгруппировать вместе несколько маршрутов с одинаковыми параметрами, 
такими как префикс или с одним и тем же middleware. 

В файл `routes/web.py` добавим следующий код:
```py linenums="1" hl_lines="11-22" title="routes/web.py"
from masonite.routes import Route

ROUTES = [
    Route.get("/", "CategoryController@index"),
    Route.get("/create", "CategoryController@create").name('category_create'),
    Route.post("/create", "CategoryController@store"),
    Route.get("/single/@id", "CategoryController@show").name("category_single"),
    Route.post("/update/@id", "CategoryController@update").name("category_update"),
    Route.get("/delete/@id", "CategoryController@destroy"),

    Route.group(
        [
            Route.get("/create", "TaskController@create").name("create"),
            Route.post("/create", "TaskController@store").name("store"),
            Route.get("/single/@id", "TaskController@show").name("single"),
            Route.post("/update/@id", "TaskController@update").name("update"),
            Route.get("/delete/@id", "TaskController@destroy").name("delete"),
            Route.get("/@category_id", "TaskController@index").name("list"),
        ],
        prefix="/task",
        name="task."
    )
]
```
Для объединения маршрутов в группу мы используем метод `group()`, который принимает список маршрутов 
и параметры.

В приведенном выше коде, имена маршрутов будут `task.list`, `task.create` и т.д. За это отвечает параметр
`name`.

URL-адрес будет виде `/task`, `/task/create `и т.д. За это отвечает параметр
`prefix`.

Более подробно о группах маршрутов можно посмотреть в 
[документации](https://masonite.pro/basics/routing/).

!!! note warning "Параметр name"
    Обратите внимание, если вы указали параметр `name`, то вам обязательно нужно для каждого маршрута
    в группе указать имя в методе `name()`.
    **Иначе вы получите ошибку.**

### Создание и вывод списка задач
Вернемся в файл `app/controllers/TaskController.py` и доработаем три метода, `index()`, `create()`, 
`store()`.
```py linenums="1" hl_lines="11-13 15-17 19-33"  title="app/controllers/TaskController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category
from app.models.Task import Task


class TaskController(Controller):
    def index(self, view: View, request: Request):
        tasks = Task.where("category_id", request.param("category_id")).get()
        return view.render("task.list", {"tasks": tasks})

    def create(self, view: View):
        categories = Category.all()
        return view.render("task.create", {"categories": categories})

    def store(self, request: Request, response: Response):
        error = request.validate(
            {
                "text": "required",
                "end_date": "required",
                "category_id": "required"
            }
        )
        if error:
            return response.redirect(name="task.create")

        Task.create(**request.only("text", "end_date", "category_id"))
        return response.redirect(
            name="task.list", params={"category_id": request.input("category_id")}
        )

    def show(self, view: View):
        return view.render("")

    def edit(self, view: View):
        return view.render("")

    def update(self, view: View):
        return view.render("")

    def destroy(self, view: View):
        return view.render("")
```

В `index()` мы ищем все задачи по `id` категории, для этого используем метод `where()`. Обратите
внимание, что вызван метод `get()`. Это нужно для того, чтобы получить `Collection`. Подробнее о
`Collection` можно прочитать в [документации Masonite ORM](https://orm.masoniteproject.com/collections)

В методе `create()` мы не просто рендерим страницу, но и передаем список всех категорий, чтобы можно
было выбрать к какой категории привязать задачу.

Наш метод `store()` похож на тот который мы уже писали для категорий, но есть отличия. Указано 
несколько обязательных данных, такие как, `text`, `end_date`, `category_id`.

Также обратите внимание на 30-ю строку. Здесь для получения данных мы используем метод `only()`.
Он вернет только те данные, которые мы перечислили.

Для редиректа пользователя на страницу со списком задач, в метод `redirect()` передаем имя маршрута и
необходимые параметры.


### Шаблон списка задач
В директории `templates` создайте директорию `task`. 

Создайте файл `list.html` в `templates/task`. И добавьте в него код ниже.
```html linenums="1" title="templates/task/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>
<section>

  <h2>Список задач</h2>
  <a class="btn" href="{{ route('task.create') }}">Создать задачу</a>

  @if tasks
    @for task in tasks
    <div class="list-item">
      <div>
          <span class="
            @if task.done == 'on'
              done
            @else
              work
            @endif
            "
          >
          </span>
        <span>Задача: {{task.text}} | </span>
        <span>выполнить до: {{task.end_date}}</span>
      </div>
      <a href="{{ route('task.single', {'id': task.id}) }}">Редактировать</a>
    </div>
    @endfor
  @endif

</section>
</body>
</html>
```
Разберем некоторые моменты более подробно.
```html linenums="1" hl_lines="12 17-24 29" title="templates/task/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>
<section>

  <h2>Список задач</h2>
  <a class="btn" href="{{ route('task.create') }}">Создать задачу</a>

    @for task in tasks
    <div class="list-item">
      <div>
          <span class="
            @if task.done == 'on'
              done
            @else
              work
            @endif
            "
          >
          </span>
        <span>Задача: {{task.text}} | </span>
        <span>выполнить до: {{task.end_date}}</span>
      </div>
      <a href="{{ route('task.single', {'id': task.id}) }}">Редактировать</a>
    </div>
    @endfor

</section>
</body>
</html>
```
Обратите внимание на выделенные строки. На 12-й строке для формирования url используется метод `route()`.

На 29-й строке также используем `route()`, но так как, нам нужно передать `id` задачи, после имени
маршрута указываем словарь для нашего параметра `id`. Обратите внимание, что `task.id` не взять в двойные
фигурные скобки.

Рассмотрим с 17-й по 24-у строки. Здесь для того, чтобы установить нужный класс тегу `span`, проверяем 
значение `task.done`. Так у нас отобразиться галочка или кружок, которые будут информировать о статусе
задачи.

### Шаблон создания задачи
Создайте файл `templates/task/create.html` и добавьте в него код ниже.
```html linenums="1" title="templates/task/create.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Title</title>
</head>
<body>
<section>

  <div class="center">
    <h2>Создать задачу</h2>
  </div>

  <form action="{{ route('task.store') }}" method="POST">
    {{ csrf_field }}
    <input type="text" name="text">
    <input type="datetime-local" name="end_date">
    <select name="category_id">
      <option disabled>Выберите категорию</option>
      @for category in categories
        <option value="{{category.id}}">{{category.name}}</option>
      @endfor
    </select>
    <div>
      <button class="btn" type="submit">Создать</button>
    </div>
  </form>

</section>
</body>
</html>
```
Рассмотрим фрагмент кода с выбором категории.

```html linenums="1" hl_lines="11-13" title="templates/task/create.html"
...

  <h2>Создать задачу</h2>

  <form action="{{ route('task.store') }}" method="POST">
    {{ csrf_field }}
    <input type="text" name="text">
    <input type="datetime-local" name="end_date">
    <select name="category_id">
      <option disabled>Выберите категорию</option>
      @for category in categories
        <option value="{{category.id}}">{{category.name}}</option>
      @endfor
    </select>
    <div>
      <button class="btn" type="submit">Создать</button>
    </div>
  </form>

...
```

Здесь мы перебираем список категорий и формируем `select`. Значение `option` - это `id` категории, 
а пользователю покажем имя категории. 

После создания задачи, нас должно перенаправить на список задач.

### Проверяем работу
Запустите сервер разработки и перейдите по ссылке [http://127.0.0.1:8000](http://127.0.0.1:8000)
и кликните по названию категории. Затем вы сможете создать задачу.

После сохранения вас перенаправит на список задач той категории, которую вы выбрали при создании.

## Вывод, редактирование и удаление задачи
Реализуем функционал, который позволит нам получить одну задачу, а так же отредактировать и удалить ее.

### Контроллер
В контроллере `TaskController` обновим код в методах `show()`, `update()` и `destroy()`.
```py linenums="1" hl_lines="35-38 43-61 63-65"  title="app/controllers/TaskController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category
from app.models.Task import Task


class TaskController(Controller):
    def index(self, view: View, request: Request):
        tasks = Task.where("category_id", request.param("category_id")).get()
        return view.render("task.list", {"tasks": tasks})

    def create(self, view: View):
        categories = Category.all()
        return view.render("task.create", {"categories": categories})

    def store(self, request: Request, response: Response):
        error = request.validate(
            {
                "text": "required",
                "end_date": "required",
                "category_id": "required"
            }
        )
        if error:
            return response.redirect(name="task.create")

        Task.create(**request.only("text", "end_date", "category_id"))
        return response.redirect(
            name="task.list", params={"category_id": request.input("category_id")}
        )

    def show(self, view: View, request: Request):
        task = Task.find_or_fail(request.param("id"))
        categories = Category.all()
        return view.render('task.single', {"task": task, "categories": categories})

    def edit(self, view: View):
        return view.render("")

    def update(self, request: Request, response: Response):
        task = Task.find_or_fail(request.param("id"))

        errors = request.validate(
            {
                "text": "required",
                "end_date": "required",
                "category_id": "required"
            },
        )
        if errors:
            return response.redirect(name='task.single', params={"id": request.param("id")})

        data = request.only("text", "end_date", "category_id")
        data.update({"done": request.input("done", "off")})
        task.update(data)
        return response.redirect(
            name="task.list", params={"category_id": request.input("category_id")}
        )

    def destroy(self, request: Request, response: Response):
        Task.find_or_fail(request.param("id")).delete()
        return response.redirect(name="task.list")
```
Рассмотрим каждый метод по отдельности. Начнем с метода `show()`.

```py linenums="1" hl_lines="6"  title="app/controllers/TaskController.py"
...
class TaskController(Controller):
    ...
    def show(self, view: View, request: Request):
        task = Task.find_or_fail(request.param("id"))
        categories = Category.all()
        return view.render('task.single', {"task": task, "categories": categories})
    ...
```

Помимо того, что получаем задачу по `id`, еще нам потребуется получить все категории. Они нам нужны
для того, чтобы мы могли изменить категорию задачи при редактировании. 

```py linenums="1" hl_lines="17-19"  title="app/controllers/TaskController.py"
...
class TaskController(Controller):
    ...
    def update(self, request: Request, response: Response):
        task = Task.find_or_fail(request.param("id"))

        errors = request.validate(
            {
                "text": "required",
                "end_date": "required",
                "category_id": "required"
            },
        )
        if errors:
            return response.redirect(name='task.single', params={"id": request.param("id")})

        data = request.only("text", "end_date", "category_id")
        data.update({"done": request.input("done", "off")})
        task.update(data)
        return response.redirect(
            name="task.list", params={"category_id": request.input("category_id")}
        )
    ...
```

В методе `update()` на 17-й строке получаем словарь с перечисленными данными.

Так-как `done` необязательно для заполнения и при его отсутствии нам нужно получить значение по умолчанию.
Если пользователь не выберет данный `checkbox`, то он не будет передан вместе с данными формы, т.е. 
будет отсутствовать. И по логики нашего приложения это значит, что `done` установлен в `off`.
Для этого на 18-й строке в словарь добавляем ключ `done` со значением который выбрал пользователь или
по умолчанию `"off"`.

Для обновления нескольких столбцов в таблице, мы используем метод модели `update()` в который 
передаем словарем пришедшие данные.

!!! note success "Обновление записей"
    При обновлении записи применяются только те атрибуты, которые претерпели изменения. 
    Если изменений нет, обновление не будет запущено.
    Вы можете переопределить это поведение несколькими способами:

    - Передать `force=True` методу `update()`:
    ```py linenums="1"
    Task.find(1).update({"text": "Закончить проект."}, force=True)
    ```
    - Определить атрибут `__force_update__` в классе модели:
    ```py linenums="1"
    class Task(Model):
        __force_update__ = True

    Task.find(1).update({"text": "Закончить проект."})
    ```
    - Использовать метод модели `force_update()`:
    ```py linenums="1"
    Task.find(1).force_update({"text": "Закончить проект."})
    ```

```py linenums="1" hl_lines="5"  title="app/controllers/TaskController.py"
...
class TaskController(Controller):
    ...
    def destroy(self, request: Request, response: Response):
        Task.find_or_fail(request.param("id")).delete()
        return response.redirect(name="task.list")
    ...
```

В методе `destroy()` мы вызываем метод `delete()` зразу после `find_or_fail()`.
Не используя лишнюю переменную, как в прошлый раз в категории.
```
post = Category.find_or_fail(request.param("id"))
post.delete()
```

### Шаблон
Создайте файл `single.html` в `templates/task` и добавьте в него следующий код.
```html linenums="1" title="templates/task/single.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Редактирование задачи</title>
</head>
<body>

<section>
  <h2>Редактирование задачи</h2>

  <form action="{{ route('task.update', {'id': task.id}) }}" method="POST">
    {{ csrf_field }}
    <input type="checkbox" name="done" value="on"
           @if task.done == 'on'
            checked
           @endif
    >
    <input type="text" name="text" value="{{task.text}}">
    <input type="datetime-local" name="end_date" value="{{task.end_date}}">
    <select name="category_id">
      <option disabled>Выберите категорию</option>
      @for category in categories

      <option value="{{category.id}}"
              @if task.category_id== category.id
                selected
              @endif
      >
        {{category.name}}
      </option>

      @endfor
    </select>
    <div>
      <button class="btn" type="submit">Сохранить</button>
      <a class="btn-del" href="{{ route('task.delete', {'id': task.id}) }}">Удалить</a>
    </div>
  </form>

</section>
</body>
</html>
```
Разберем фрагмент с `checkbox`.
```html linenums="1" hl_lines="4-8" title="templates/task/single.html"
...
  <form action="{{ route('task.update', {'id': task.id}) }}" method="POST">
    {{ csrf_field }}
    <input type="checkbox" name="done" value="on"
           @if task.done == 'on'
            checked
           @endif
    >
    <input type="text" name="text" value="{{task.text}}">
...
```
Если значение `done` буден `on`, тогда `checkbox` будет отображаться как выбранный.

Теперь рассмотрим более подробно выбор категории.
```html linenums="1" hl_lines="7-9" title="templates/task/single.html"
...
    <select name="category_id">
      <option disabled>Выберите категорию</option>
      @for category in categories

        <option value="{{category.id}}"
              @if task.category_id == category.id
                selected
              @endif
        >
          {{category.name}}
        </option>

      @endfor
    </select>
...
```
Для того чтобы сразу была показана категория, которая была выбрана до этого, используем оператор `if`.
Сравнивая категорию задачи с текущей категорией из цикла. Если категории совпадают устанавливаем
`selected`.

!!! note success "Оператор if"
    Обратите внимание, что оператор `if` находиться внутри html тега. Для того чтобы наш код 
    сработал, мы его написали с новой строки. Это нюанс шаблонизатора.

### Проверим работу
Теперь можно запустить сервер разработки и проверить работу. Перейдите на страницу списка задач и
попробуйте отредактировать или удалить задачу.

## Итог

В данном руководстве вы познакомились с базовыми функциями фреймворка Masonite.

Полный код проекта можно найти на [GitHub](https://github.com/DJWOMS/masonite-todo)
