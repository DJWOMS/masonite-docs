# Задачи
Для начала нам нужно создать контроллер для задач. Создадим контроллер сразу с нужными методами, для
этого нужно использовать флаг `-r`.

Выполните команду:
```
python craft controller Task -r
```

### Контроллер задач
Откройте файл `app/controllers/TaskController.py`.

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

Сразу добавим нужные нам импорты. Импортируем классы `Request`, `Response` и модель `Task`.
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
```py linenums="1" hl_lines="11-23" title="routes/web.py"
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
            Route.get("/", "TaskController@index").name("list"),
            Route.get("/create", "TaskController@create").name("create"),
            Route.post("/create", "TaskController@store").name("store"),
            Route.get("/single/@id", "TaskController@show").name("single"),
            Route.post("/update/@id", "TaskController@update").name("update"),
            Route.get("/delete/@id", "TaskController@destroy").name("delete"),
            Route.post("/edit/@id", "TaskController@edit").name("done"),
        ],
        prefix="/task",
        name="task."
    )
]
```
Для этого мы используем метод `group()`, который принимает список маршрутов и параметры.

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
```py linenums="1" hl_lines="11-13 15-17 19-31"  title="app/controllers/TaskController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category
from app.models.Task import Task


class TaskController(Controller):
    def index(self, view: View):
        tasks = Task.all()
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
        return response.redirect(name="task.list")

    def show(self, view: View):
        return view.render("")

    def edit(self, view: View):
        return view.render("")

    def update(self, view: View):
        return view.render("")

    def destroy(self, view: View):
        return view.render("")
```

Метод `index()` похож на метод который мы писали для категорий. Здесь мы получаем список задач.

В методе `create()` мы не просто рендерим страницу, но и передаем список всех категорий, чтобы можно
было выбрать к какой категории привязать задачу.

Наш метод `store()` также похож на тот который мы уже писали, но есть отличия. Указано 
несколько обязательных данных, такие как, `text`, `end_date`, `category_id`.

Также обратите внимание на 30-ю строку. Здесь для получения данных мы используем метод `only()`.
Он вернет только те данные которые мы указали.

### Шаблон списка задач
В директории `templates` создайте директорию `task`. 

Создайте файл `templates/task/list.html`.
```html linenums="1" hl_lines="12 17" title="templates/task/list.html"
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/static/style.css" rel="stylesheet">
  <title>Категории</title>
</head>
<body>

<section>
  <h2>Список всех задач</h2>
  <a href="{{ route('task.create') }}">Создать задачу</a>

  <ul class="list">
    @for task in tasks
    <li class="list-item">
      <a href="{{ route('task.single', {'id': task.id}) }}">{{task.text}}</a>
    </li>
    @endfor
  </ul>
</section>

</body>
</html>
```
Данный шаблон похож на тот, который ми писали для списка категорий, но есть пару отличий.

Обратите внимание на выделенные строки. На 12-й строке для формирования url используется метод `route()`.

На 17-й строке также используем `route()`, но так как, нам нужно передать `id` задачи, после имени
маршрута указываем словарь для нашего параметра `id`. Обратите внимание, что `task.id` не взять в двойные
фигурные скобки.

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
    <button type="submit">Создать</button>
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
    <button type="submit">Создать</button>
  </form>

...
```

Здесь мы перебираем список категорий и формируем `select`. Значение `option` - `id` категории, 
а пользователю покажем имя категории. 

После создания категории, нас должно перенаправить на список задач.

## Вывод, редактирование и удаление задачи
Реализуем функционал, который позволит нам получить одну задачу, а так же отредактировать и удалить ее.

### Контроллер
```py linenums="1" hl_lines="33-36 41-55 57-59"  title="app/controllers/TaskController.py"
from masonite.controllers import Controller
from masonite.request import Request
from masonite.response import Response
from masonite.views import View

from app.models.Category import Category
from app.models.Task import Task


class TaskController(Controller):
    def index(self, view: View):
        tasks = Task.all()
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
        return response.redirect(name="task.list")

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
            }
        )
        if errors:
            return response.redirect(name='task.update', params={"id": request.param("id")})

        task.force_update(request.only("text", "end_date", "category_id"))
        return response.redirect(name="task.list")

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

```py linenums="1" hl_lines="17"  title="app/controllers/TaskController.py"
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
            }
        )
        if errors:
            return response.redirect(name='task.update', params={"id": request.param("id")})

        task.update(request.only("text", "end_date", "category_id"))
        return response.redirect(name="task.list")
    ...
```

Метод update похож на тот который мы писали для редактирования категорий. Но есть одно отличие. 
Здесь для обновления нескольких столбцов в таблице, мы используем метод `update()` в который 
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
Создайте файл `templates/task/single.html` и добавьте в него следующий код.
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
    <input type="text" name="text" value="{{task.text}}">
    <input type="datetime-local" name="end_date" value="{{task.end_date}}">
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
    <button type="submit">Сохранить</button>
  </form>

  <a href="{{ route('task.delete', {'id': task.id}) }}">Удалить</a>
  </section>

</body>
</html>
```
Рассмотрим более подробно выбор категории.
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
    сработал, мы его написали с новой строки. Это нюансы шаблонизатора.
