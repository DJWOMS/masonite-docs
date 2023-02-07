# CSRF защита в Masonite

## Введение
Защита CSRF обычно влечет за собой установку уникального токена для пользователя при запросе 
страницы, который соответствует тому же токену на сервере. Это предотвращает отправку формы без 
правильного токена.

## Начиная
Функции CSRF для Masonite расположены в Service Provider `CsrfProvider` и `CsrfMiddleware`. Если вы 
не хотите иметь защиту CSRF, вы можете безопасно удалить их оба.

`CsrfProvider` просто загружает функции CSRF в контейнер, а `CsrfMiddleware` — это то, что фактически 
генерирует ключи и проверяет их действительность.

## Шаблоны
По умолчанию для всех запросов POST требуется CSRF токен. Мы можем добавить CSRF токен в 
формы, используя тег `{{ csrf_field }}`, следующим образом:

```html
<form action="/dashboard" method="POST">
    {{ csrf_field }}

    <input type="text" name="first_name">
</form>
```
Это добавит скрытое поле, которое выглядит так:

```html
<input type="hidden" name="__token" value="8389hdnajs8...">
```
Если этот токен будет изменен, Masonite выдаст исключение `InvalidCsrfToken` из 
middleware.

Если вы попытаетесь отправить запрос `POST` без `{{ csrf_field }}`, вы получите исключение 
`InvalidCsrfException`. Это просто означает, что вам либо не хватает тега Jinja2, либо вам не хватает 
этого маршрута из атрибута класса `exempt` в вашем middleware.

Вы можете получить сгенерированный токен. Это полезно при использовании JS, где вам нужно передать 
токен CSRF в AJAX.

```html
<p> Token: {{ csrf_token }} </p>
```

## AJAX/Vue/Axios
Для ajax лучший способ передать токены CSRF — это установить токен внутри родительского 
шаблона внутри мета-тега, например:

```html
<meta name="csrf-token" content="{{ csrf_token }}">
```
И тогда вы можете получить токен и поместить его туда, где вам нужно:

```javascript
token = document.head.querySelector('meta[name="csrf-token"]')
```
Затем вы можете передать токен через заголовок X-CSRF-TOKEN вместо `__token`.

## Освободить маршруты от защиты

Не для всех маршрутов может потребоваться защита CSRF, например аутентификация OAuth или 
webhooks. Чтобы освободить маршруты от защиты, мы можем добавить его в атрибут 
класса `exempt` в middleware, расположенном в `app/middlewares/VerifyCsrfToken.py`:

```py linenums="1" title="app/middlewares/VerifyCsrfToken.py"
from masonite.middleware import VerifyCsrfToken as Middleware


class VerifyCsrfToken(Middleware):

    exempt = ['/oauth/github']
```
Теперь любые POST запросы, ведущие к `your-domain.com/oauth/github`, не защищены CSRF и для этого 
маршрута не будет выполняться никаких проверок. Используйте это с осторожностью, так как защита CSRF 
имеет решающее значение для безопасности приложений, но не для всех маршрутов она нужна.

### Исключение нескольких маршрутов
Вы также можете использовать подстановочные знаки `*` для исключения нескольких маршрутов с одним и 
тем же префиксом. Например, вам может понадобиться сделать это:

```py linenums="1" title="app/middlewares/VerifyCsrfToken.py"
from masonite.middleware import VerifyCsrfToken as Middleware


class VerifyCsrfToken(Middleware):

    exempt = [
        '/api/document/reject-reason',
        '/api/document/*/reject',
        '/api/document/*/approve',
        '/api/document/*/process/@user',
    ]

    ...
```
Мы немного повторяемся, поэтому вместо этого вы можете указать подстановочный знак `*`:

```py linenums="1" title="app/middlewares/VerifyCsrfToken.py"
from masonite.middleware import VerifyCsrfToken as Middleware


class VerifyCsrfToken(Middleware):

    exempt = [
        '/api/document/*',
    ]

    ...
```
