# Введение и установка masonite
Документация **Masonite** на русском языке.

Masonite — это инструмент разработки, ориентированный на разработчиков, со всеми функциями, 
необходимыми для быстрой разработки. Masonite идеально подходит для начинающих, разрабатывающих 
своё первое веб-приложение, или для опытных разработчиков и компаний, которым необходимо использовать 
весь набор доступных функций.

[Официальная документация на английском](https://docs.masoniteproject.com/).

!!! note success "Masonite - python веб фреймворк."
    Прекратите использовать старые фреймворки с запутанным функционалом. 


Masonite прилагает все усилия, чтобы быть быстрым и простым от установки до развертывания. 
Разработчики могут перейти от концепции к созданию быстрее и эффективнее. 
Используйте Masonite для своего следующего SaaS!

!!! note success
    Если вам нужны видео уроки, то посетите сайт [masonitecasts.com](masonitecasts.com) 
    или youtube канал [Django School](https://www.youtube.com/c/DjangoSchool). 
    [Telegram](https://t.me/masonite_channel)


### Некоторые примечательные возможности встроенные в Masonite

- Поддержка быстрой отправки электронных писем.
- Поддержка очередей для ускорения вашего приложения. Отправка заданий на выполнения в очередь или асинхронно.
- Простой и эффективный способ отправки уведомлений вашим пользователям.
- Планирование и запуск задач по расписанию (например, каждый день в полночь).
- События, которые вы можете прослушивать, при выполнении ваших задач, когда в вашем приложении происходят определенные события.
- КРАСИВЫЙ ORM в стиле Active Record под названием Masonite ORM. 
- Много других функций, которые вам нужны, вы можете найти в документации!

Это и многое другое, поставляется из коробки и готово к работе.

## Зависимости
Чтобы использовать Masonite, вам понадобятся:

- Python 3.7+
- Последняя версия OpenSSL
- Pip3

!!! note warning
    Все команды python и pip в этой документации предполагают, что они выполняются с использованием версии Python 3.7+. 
    Если у вас возникли проблемы с какими-либо этапами установки, просто убедитесь, что команды выполняются с 
    использованием Python 3.7+, а не 2.7 или ниже.

### Linux
Если вы используете Linux, вам понадобятся пакет Python dev и libssl. Вы можете загрузить эти пакеты, запустив:

Дистрибутивы Linux на базе Debian и Ubuntu

    $ sudo apt install python3-dev python3-pip libssl-dev build-essential python3-venv

Dам может потребоваться указать python3.x-dev версию:

    $ sudo apt-get install python3.7-dev python3-pip libssl-dev build-essential python3-venv

Дистрибутивы на базе Enterprise Linux (Fedora, CentOS, RHEL, ...)

    # dnf установить python-devel openssl-devel

## Установка

!!! note success
    Не забудьте присоединиться к сообществу офф. [Discord](https://slack.masoniteproject.com/) или нашему
    [telegram](https://t.me/django_school), чтобы получить помощь или рекомендации.

Masonite отличается простой установкой и запуском. Если вы используете предыдущие версии Masonite, порядок некоторых шагов установки немного изменился.

Во-первых, откройте терминал и перейдите в каталог, в котором вы хотите создать свое приложение. 
Вы можете создать его в каталоге **programming**, например:

    $ cd ~/programming
    $ mkdir myapp
    $ cd myapp

Если вы работаете в Windows, вы можете просто создать каталог и открыть его в Powershell.

## Активация виртуального окружения (необязательно)
Хотя этот шаг технически необязателен, он настоятельно рекомендуется. Вы можете создать виртуальное 
окружение, если не хотите устанавливать все зависимости masonite. Создайте своё виртуальное окружение, выполнив:

    $ python -m venv venv
    $ source venv/bin/activate

Если вы используете Windows:

    $ python -m venv venv
    $ ./venv/Scripts/activate

!!! note info
    Команда python здесь использует Python 3. Ваш компьютер может запускать Python 2 (обычно 2.7) по 
    умолчанию для машин UNIX. Вы можете установить псевдоним на своем компьютере для Python 3 или просто 
    запустить python3 когда увидите команду python.

Например, вы запустите `python3 -m venv venv` вместо `python -m venv venv`

## Установка
Сначала установите Masonite:

    $ pip install masonite

Затем создайте новый проект:

    $ project start .

Это создаст новый проект в текущем каталоге.
Если вы хотите создать проект в новом каталоге (например my_project), вы должны указать имя каталога 
**project start my_project**.

Затем установите зависимости Masonite:

    $ project install

Если вы создали проект в новом каталоге, вы должны перейти в этот каталог перед запуском **project install**.
После установки вы можете запустить сервер разработки:

    $ python craft serve

Поздравляем! Вы настроили свой первый проект masonite!
