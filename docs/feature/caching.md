# Кэширование в Masonite

Masonite предоставляет мощную функцию кэширования любых данных, которые могут 
быть ненужными или дорогими для извлечения при каждом запросе. Кэширование Masonite позволяет сохранять 
и извлекать данные, устанавливать время истечения срока действия и управлять различными хранилищами кеша.

Masonite поддерживает следующие драйверы кеша: [Redis](#redis), [Memcached](#memcached) и базовый 
драйвер [File](#file-cache).

В этой документации мы рассмотрим, как настроить и использовать кеш.

## Конфигурация

Конфигурация кеша находится в файле `config/cache.py`. В этом файле можно указать разные
драйверы в словаре `STORES` и значение по умолчанию для использования в вашем приложении 
в ключе `default`.

По умолчанию Masonite настроен на использование драйвера файлового кэша, ключ `local`.
```py linenums="1" title="config/cache.py"
STORES = {
    "default": "local",
    "local": {
        "driver": "file",
        "location": "storage/framework/cache"
    },
    "redis": {
        "driver": "redis",
        "host": "127.0.0.1",
        "port": "6379",
        "password": "",
        "name": "masonite4",
    },
    "memcache": {
        "driver": "memcache",
        "host": "127.0.0.1",
        "port": "11211",
        "password": "",
        "name": "project_name",
    },
}
```

!!! note info 
    Для высоконагруженных приложений рекомендуется использовать более эффективный драйвер, например 
    `Memcached` или `Redis`.

### <a name="file-cache"></a> Файловый кэш
Драйвер файлового кэша хранит данные, сохраняя их в файловой системе сервера. Его можно использовать 
как есть без стороннего сервиса.

Место где Masonite будет хранить файлы данных кеша, по умолчанию `storage/framework/cache`, 
но его можно изменить с помощью параметра `location`.

### <a name="redis"></a> Redis
Для драйвера кеша Redis требуется пакет Python `redis`, который вы можете установить с помощью:
```shell
pip install redis
```

Затем вы должны определить Redis в качестве хранилища по умолчанию и настроить его с параметрами 
вашего сервера Redis:
```py linenums="1" title="config/cache.py"
from masonite.environment import env


STORES = {
    "default": "redis",
    "redis": {
        "driver": "redis",
        "host": env("REDIS_HOST", "127.0.0.1"),
        "port": env("REDIS_PORT", "6379"),
        "password": env("REDIS_PASSWORD"),
        "name": env("REDIS_PREFIX", "project name"),
    },
}
```
Наконец, убедитесь, что сервер Redis работает и вы готовы [начать использовать кеш](#using-the-cache).


### <a name="memcached"></a> Memcached
Для драйвера кеша Memcached требуется пакет Python `pymemcache`, который вы можете установить с помощью:

```shell
pip install pymemcache
```

Затем вы должны определить Memcached как хранилище по умолчанию и настроить его с параметрами вашего 
сервера Memcached:

```py linenums="1" title="config/cache.py"
from masonite.environment import env


STORES = {
    "default": "memcache",
    "memcache": {
        "driver": "memcache",
        "host": env("MEMCACHED_HOST", "127.0.0.1"),
        "port": env("MEMCACHED_PORT", "11211"),
        "password": env("MEMCACHED_PASSWORD"),
        "name": env("MEMCACHED_PREFIX", "project name"),
    },
}
```

Убедитесь, что сервер Memcached запущен и вы готовы [начать использовать кеш](#using-the-cache).


## <a name="using-the-cache"></a> Использование кеша

Вы можете получить доступ к службе Cache через фасад `Cache` или разрешив его из 
[service container](/architecture/service-container).

### Хранение данных
Доступны два метода: `add` и `put`.

#### add
Вы можете легко добавить данные в кеш, используя метод `add`. Это либо извлечет данные, уже находящиеся 
в кеше, если срок их действия не истек, либо вставит новое значение.

```py linenums="1"
from masonite.cache import Cache


def store(self, cache: Cache):
    data = cache.add('age', '21')
```

Если ключ `age` существует в кеше и срок его действия не истек, то в кеш будет добавлено и возвращено 
значение `21`. Если ключ `age` не существует или срок его действия не истек, он вернет все данные, 
находящиеся в кеше для этого ключа.

#### put

Метод `put` помещает данные в кеш независимо от того, существует ли он. Это хороший способ перезаписать 
данные в кеше:

```py
cache.put('age', '21')
```

Вы можете указать количество секунд, в течение которых кэш должен быть действителен. Не указывайте 
время или укажите `None`, чтобы сохранить данные навсегда.

```py
cache.put('age', '21', seconds=300) # хранится 5 минут
```

Вы также можете кэшировать списки и словари, которые сохранят типы данных:

```py
cache.put('user', {"name": "Joe"}, секунды=300) # хранится 5 минут
cache.get("user")['name'] #== "Joe"
```

### Получение данных

Вы можете получить данные из кеша. Если срок действия данных истек, вернет `None` или
значение по умолчанию, которое вы укажете:

```py
cache.get('age', '40')
```

Это либо извлечет правильные данные `age` из кеша, либо вернет значение по умолчанию `40`.

### Проверка существования данных

Вы также можете проверить, существует ли значение в кеше и срок его действия не истек:

```py
cache.has('age')
```

### Удаление данных

Если вы хотите удалить элемент в кеше:
```py
cache.forget('age')
```

Будет удален этот элемент из кеша.

### Увеличение/уменьшение значения

Вы можете увеличивать и уменьшать значение, если оно целое число:

```py
cache.get('age') #== 21
cache.increment('age') #== 22
cache.decrement('age') #== 21
```

### Запоминание

Запоминание — отличный способ сохранить что-то в кеше с помощью вызываемого объекта:

```py linenums="1"
from app.models import User


cache.remember("total_users", lambda cache: (
    cache.put("total_users", User.all().count(), 300)
))
```

### Очистка кеша

Чтобы удалить всё в кеше, вы можете просто сбросить его:

```py
cache.flush()
```
