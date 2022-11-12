# Конфигурация Masonite
Файлы конфигурации в Masonite по умолчанию собраны в одну папку `config`.

Каждая функция может иметь некоторые параметры в своем собственном файле, названном в честь функции. 
Например, вы найдете параметры связанные с почтой, в файле `config/mail.py`.

## Начало
Класс `Configuration` отвечает за загрузку всех файлов конфигурации перед запуском приложения.

Он загрузит все файлы, расположенные по пути, определенному через привязку `config.location`, 
которая по умолчанию имеет значение `config/`.

Затем доступ к значениям осуществляется на основе файла, которому они принадлежат и доступ
к вложенным параметрам можно получить через точку.

Например, файл `config/mail.py`:

```py
from masonite.environment import env

FROM_EMAIL = env("MAIL_FROM", "no-reply@masonite.com")

DRIVERS = {
    "default": env("MAIL_DRIVER", "terminal"),
    "smtp": {
        "host": env("MAIL_HOST"),
        "port": env("MAIL_PORT", "587"),
        "username": env("MAIL_USERNAME"),
        "password": env("MAIL_PASSWORD"),
        "from": FROM_EMAIL,
    }
}
```

- `mail` вернет словарь со всеми параметрами.
- `mail.from_email` вернет значение из `FROM_EMAIL`.
- `mail.drivers.smtp.port` вернет значение порта `smtp`.

## Получение значений
Чтобы прочитать значение конфигурации, можно использовать `Config` из `facade`:
```py
from masonite.facades import Config


Config.get("mail.from_email")
# a default value can be provided
Config.get("database.mysql.port", 3306)
```
Или использовать `config`:
```py
from masonite.configuration import config


config("mail.from_email")
# a default value can be provided
config("database.mysql.port", 3306)
```

## Настройки

Установка значений конфигурации достигается с помощью файлов конфигурации проекта.

## Основное значение
Вы можете на лету переопределить значение конфигурации с помощью `Config`:
```py
Config.set("mail.from_email", "support@masoniteproject.com")
```

!!! note warning
    Это следует делать с осторожностью, так как может иметь неожиданные побочные эффекты в зависимости 
    от того, когда вы переопределяете параметр конфигурации.

Это в основном полезно во время тестов, когда вы хотите переопределить параметр конфигурации для 
проверки определенного поведения:
```py
def test_something(self):
    old_value = Config.get("mail.from_email")
    Config.set("mail.from_email", "support@masoniteproject.com")
    
    # test...
    
    Config.set("mail.from_email", old_value)
```
Но если вы просто хотите иметь различную конфигурацию в зависимости от среды (разработки, 
тестирования или производства), вам следует полагаться на
[переменные окружения](https://masonite.pro/basics/environments/)
