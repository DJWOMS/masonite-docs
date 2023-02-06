# Команды в Masonite

Команды в Masonite, предназначены для облегчения вашей разработки. Команды могут варьироваться 
от создания контроллеров и моделей до установки пакетов и публикации assets.

Masonite использует пакет [cleo](https://cleo.readthedocs.io/en/latest/) для командной оболочки.

Доступные команды для Masonite можно отобразить, запустив:

```shell
python craft
```
Она покажет список команд, уже доступных в Masonite.

У каждой команды есть документация, в которой описаны доступные аргументы и параметры команды.
Чтобы просмотреть документацию по команде, добавьте к имени команды префикс `help`. 
Например, чтобы увидеть описание команды `serve`, выполните:

```shell
python craft help serve
```

# Создание команд
Команды могут быть созданы с помощью простого класса команд, наследуемого от класса `Command`:

```py
from masonite.commands import Command


class MyCommand(Command):
    """
    Description of Command

    command:signature
        {user : A positional argument for the command}
        {--f|flag : An optional argument for the command}
        {--o|option=default: An optional argument for the command with default value}
    """

    def handle(self):
        pass
```
Имя, описание и аргументы команды анализируются из строки документации команды.

## Имя и описание

Строка документации должна начинаться с описания команды
```py
"""
Описание команды

"""
```

А затем после пустой строки вы можете определить имя команды.

```py
"""
Описание команды

имя_команды
"""
```


## Позиционные аргументы

После имени команды в строке документации должны идти аргументы. Аргументы определяются с одним отступом 
и заключаются в скобки.

Позиционные (обязательные) аргументы определяются без тире (`-` или `--`).

Вот как определить позиционный аргумент `name` с описанием:
```py
"""
    {name: описание аргумента name}
"""
```

Внутри команды позиционные аргументы могут быть получены с помощью `self.argument(arg_name)`
```py
    def handle(self):
        name = self.argument("name")
```

## Необязательные аргументы

Необязательные аргументы обозначаются тире и могут использоваться в любом порядке при вызове команды. 
Необязательный аргумент `--force` может иметь короткое имя `--f`.

Вот как определить два необязательных аргумента `iterations` и `force` с описанием:
```py
"""
имя_команды
    {--iterations : Описание аргумента iterations}
    {--f|force : Описание аргумента force}
"""
```

Обратите внимание, что мы предоставили короткую версию для аргумента `force`, но не для аргументов 
`iterations`.

Теперь команду можно использовать так:
```shell
python craft command_name --f --iterations
python craft command_name --iterations --force
```

Если необязательный аргумент требует значения, вы должны добавить суффикс `=`:

```py
"""
имя_команды
    {--iterations= : Описание аргумента iterations}
"""
```

Здесь при использовании `iterations` пользователь должен указать значение.

```shell
python craft command_name --iterations 3
python craft command_name --iterations=3
```

Если аргумент может иметь или не иметь значения, вы можете использовать вместо него суффикс `=?`.

```py
"""
имя_команды
    {--iterations=?: Описание аргумента iterations}
"""
```

```shell
python craft command_name --iterations
python craft command_name --iterations 3
```

Наконец, если нужно использовать значение по умолчанию, когда значение не указано, добавьте суффикс 
`={default}`:

```py
"""
имя_команды
    {--iterations=3: Описание аргумента iterations}
"""
```

```shell
# iterations будет равно 3
python craft command_name --iterations
# iterations будет равно 1
python craft command_name --iterations 1
```

Внутри команды необязательные аргументы можно получить с помощью `self.option(arg_name)`

```py
    def handle(self):
        name = self.option("iterations")
```

## Печать сообщений

Вы можете печатать сообщения в консоль с другим форматированием:

- `self.info("Info Message")`: выводит сообщение зеленого цвета.
- `self.warning("Warning Message")`: выводит сообщение желтого цвета.
- `self.error("Error Message")`: выводит сообщение, выделенное жирным красным цветом.
- `self.comment("Comment Message")`: выводит сообщение светло-голубого цвета.

## Advanced
Дополнительную информацию и дополнительные возможности для создания команд можно найти в 
[документации Cleo](https://cleo.readthedocs.io/en/latest/introduction.html).

Класс Masonite `Command` наследует класс Cleo `Command`, поэтому вы сможете использовать все функции 
Cleo, при создании команд.

# Регистрация команд

После создания вы можете зарегистрировать команду в [Service Container](/architecture/service-container)
внутри [Service Provider](/architecture/service-provider) 
(если у вас его нет, вы должны [создать его](/architecture/service-provider#creating-a-provider)):

Метод `add()` принимает одну или несколько команд:

```py
from masonite.providers import Provider
from some.place.YourCommand import YourCommand


class AppProvider(Provider):
    def __init__(self, application):
        self.application = application

    def register(self):
        self.application.make('commands').add(YourCommand())
```

Когда вы запустите `python craft`, вы увидите добавленную вами команду.
