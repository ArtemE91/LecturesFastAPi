# Фоновые задачи, Redis, Celery, Шаблоны в FastAPI

### План занятия

1) Фоновые задачи
2) Redis
3) Celery
4) Понимание Jinja
5) Использование шаблонов Jinja2 в FastAPI

## <font color="#46aa63">Фоновые задачи</font>

Мы можем создавать фоновые задачи, которые будут выполнятся после возвращения ответа сервером.

Это может быть полезно для функций, которые должны выполниться после получения запроса, но ожидание их выполнения необязательно для пользователя.

К примеру:

- Отправка писем на почту после выполнения каких-либо действий:
    - Т.к. соединение с почтовым сервером и отправка письма идут достаточно "долго" (несколько секунд), вы можете отправить ответ пользователю, а отправку письма выполнить в фоне.
- Обработка данных:
    - К примеру, если вы получаете файл, который должен пройти через медленный процесс, вы можете отправить ответ "Accepted" (HTTP 202) и отправить работу с файлом в фон.

##### <font color="#46aa63">Использование класса BackgroundTasks</font>

Давайте импортируем `BackgroundTasks`, и добавим в функцию параметр с типом `BackgroundTasks`:

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)


@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```

**FastAPI** создаст объект класса `BackgroundTasks` для нас и запишет его в параметр. 

Мы создали обычную функцию, которая Принимает параметры `email` и `message`.
Функция может быть как асинхронной `async def`, так и обычной `def` функцией, **FastAPI** знает, как правильно ее выполнить. Так как операция записи не использует `async` и `await`, мы определим ее как обычную `def`.
В нашем примере фоновая задача будет вести запись в файл (симулируя отправку письма).

Внутри функции мы вызываем метод `.add_task()` у объекта _background tasks_ и передаем ему функцию, которую хотим выполнить в фоне. `.add_task()` принимает следующие аргументы:
- Функцию, которая будет выполнена в фоне (`write_notification`). Обратите внимание, что передается объект функции, без скобок.
- Любое упорядоченное количество аргументов, которые принимает функция (`email`).
- Любое количество именованных аргументов, которые принимает функция (`message="some notification"`).

##### <font color="#46aa63">Встраивание зависимостей</font>

Класс `BackgroundTasks` также работает с системой встраивания зависимостей, мы можем определить `BackgroundTasks` на разных уровнях: как параметр функции, как завимость, как подзависимость и так далее.

**FastAPI** знает, что нужно сделать в каждом случае и как переиспользовать тот же объект `BackgroundTasks`, так чтобы все фоновые задачи собрались и запустились вместе в фоне:

```python
from fastapi import BackgroundTasks, Depends, FastAPI

app = FastAPI()


def write_log(message: str):
    with open("log.txt", mode="a") as log:
        log.write(message)


def get_query(background_tasks: BackgroundTasks, q: str | None = None):
    if q:
        message = f"found query: {q}\n"
        background_tasks.add_task(write_log, message)
    return q


@app.post("/send-notification/{email}")
async def send_notification(
    email: str, background_tasks: BackgroundTasks, q: str = Depends(get_query)
):
    message = f"message to {email}\n"
    background_tasks.add_task(write_log, message)
    return {"message": "Message sent"}
```

В этом примере сообщения будут записаны в `log.txt` после того, как ответ сервера был отправлен.

Если бы в запрос был передан query-параметр `q`, он бы первыми записался в `log.txt`фоновой задачей (потому что вызывается в зависимости `get_query`).

После другая фоновая задача, которая была сгенерирована в функции, запишет сообщение из параметра `email`.

##### <font color="#46aa63">Технические детали</font>

Класс `BackgroundTasks` основан на [`starlette.background`](https://www.starlette.io/background/).

Он интегрирован в FastAPI, так что вы можете импортировать его прямо из `fastapi` и избежать случайного импорта `BackgroundTask` (без `s` на конце) из `starlette.background`.

При использовании `BackgroundTasks` (а не `BackgroundTask`), вам достаточно только определить параметр функции с типом `BackgroundTasks` и **FastAPI** сделает все за вас, также как при использовании объекта `Request`.

Вы все равно можете использовать `BackgroundTask` из `starlette` в FastAPI, но вам придется самостоятельно создавать объект фоновой задачи и вручную обработать `Response` внутри него.

Подробнее изучить его можно в [Официальной документации Starlette для BackgroundTasks](https://www.starlette.io/background/).

##### <font color="#46aa63">Предостережение</font>

Если нужно выполнить тяжелые вычисления в фоне, которым необязательно быть запущенными в одном процессе с приложением **FastAPI** (к примеру, вам не нужны обрабатываемые переменные или вы не хотите делиться памятью процесса и т.д.), вы можете использовать более серьезные инструменты, такие как [Celery](https://docs.celeryproject.org/).

Их тяжелее настраивать, также им нужен брокер сообщений наподобие `RabbitMQ` или `Redis`, но зато они позволяют вам запускать фоновые задачи в нескольких процессах и даже на нескольких серверах.

Но если вам нужен доступ к общим переменным и объектам вашего **FastAPI** приложения или вам нужно выполнять простые фоновые задачи (наподобие отправки письма из примера) вы можете просто использовать `BackgroundTasks`.

## <font color="#46aa63">Redis</font>

[Redis](http://redis.io/) (Remote Dictionary Service) — это опенсорсный сервер баз данных типа ключ-значение.

Точнее всего описать `Redis` можно, сказав, что это — сервер структур данных. Уникальные особенности сервера `Redis` стали основной причиной его популярности и того, что он применяется во множестве реальных проектов.

Вместо того чтобы работать со строками базы данных, перебирать, сортировать, упорядочивать их, что если информация с самого начала будет находиться в структурах данных, которые нужны программисту? Первое время `Redis` использовали практически так же, как `Memcached`. Но, по мере развития `Redis`, эта система управления базами данных (СУБД) нашла применение и во многих других ситуациях. В частности — в реализациях механизма издатель/подписчик, в задачах потоковой обработки данных, в системах, где нужно работать с очередями.

Вот какие типы данных поддерживает Redis:

- Строка (String)
- Битовый массив (Bitmap)
- Битовое поле (Bitfield)
- Хеш-таблица (Hash)
- Список (List)
- Множество (Set)
- Упорядоченное множество (Sorted set)
- Геопространственные данные (Geospatial)
- Структура HyperLogLog (HyperLogLog)
- Поток (Stream)

`Redis` — это база данных, размещаемая в памяти, которая используется, в основном, в роли кеша, находящегося перед другой, «настоящей» базой данных, вроде `MySQL` или `PostgreSQL`. Кеш, основанный на `Redis`, помогает улучшить производительность приложений. Он эффективно использует скорость работы с данными, характерную для памяти, и смягчает нагрузку центральной базы данных приложения, связанную с обработкой следующих данных:
- Данные, которые редко меняются, к которым часто обращается приложение.
- Данные, не относящиеся к критически важным, которые часто меняются.

Примеры таких данных могут включать в себя сессионные кеши или кеши данных, а так же содержимое панелей управления — вроде списков лидеров и отчётов, включающих в себя данные, агрегированные из разных источников.

Но во многих случаях Redis гарантирует достаточно высокий уровень сохранности данных, что позволяет использовать эту СУБД в роли настоящей основной базы данных. А добавление в систему плагинов Redis и различных конфигураций высокой доступности (High Availability, HA) делает базу данных Redis крайне интересной для определённых сценариев использования и рабочих нагрузок.

Ещё одна важная особенность Redis заключается в том, что эта СУБД размывает границы между кешем и хранилищем данных. Тут важно понять то, что чтение данных из памяти и работа с данными, находящимися в памяти, гораздо быстрее чем те же операции, выполняемые традиционными СУБД, использующими обычные жёсткие диски (HDD) или твердотельные накопители (SSD).


##### <font color="#46aa63">Работа с Redis</font>

Запустим Docker контейнер:

```shell
docker run -p 6379:6379 -it redis/redis-stack:latest
```

Проверим что он развернулся и подключимся в терминале:

```shell
docker ps -a | grep redis

be328a4cb311   redis/redis-stack:latest   "/entrypoint.sh"

docker exec -it be328a4cb311 /bin/bash

redis-cli
```

Установим пакет `redis`:

```shell
pip install redis
```

Напишем следующий код:

```python
import redis  
  
r = redis.Redis(host='localhost', port=6379)  
  
  
if __name__ == "__main__":  
    r.set('foo', 'bar')  
    item = r.get('foo')  
    print(item.decode('utf-8'))
```

Мы положили в `redis` значение `bar`, которое лежит по ключу `foo`. Также мы его получили и вывели в консоль. Давайте попробуем получить это значение через `redis-cli`:

```shell
redis-cli
GET foo
"bar"
```

Давайте создадим и наполним список в `Redis`:

```python
import redis  
  
  
r = redis.Redis(host='localhost', port=6379)  
list_name = "array"  
  
if __name__ == "__main__":  
    m = [i for i in range(0, 100_000)]  
    r.rpush(list_name, *m)
```

Теперь давайте считаем его кусками:

```python
import redis  
  
  
r = redis.Redis(host='localhost', port=6379)  
list_name = "array"  
chunk_size = 10_000  
  
# Функция для считывания списка кусками  
def read_chunks(redis_client, l_name, chunk_size):  
    while True:  
        chunk = redis_client.lrange(l_name, 0, chunk_size - 1)  
        if not chunk:  
            break  
        # Декодирование байтов в строки и вывод  
        print(f"Read {len(chunk)} elements")  
        redis_client.ltrim(l_name, len(chunk), -1)  
  
  
if __name__ == "__main__":  
    read_chunks(r, list_name, chunk_size)
```


##### <font color="#46aa63">Основные функции и команды</font>

**Установка значения по ключу**:

```python
r.set('my_key', 'my_value')
```

**Получение значения по ключу**:

```python
value = r.get('my_key')
print(value.decode('utf-8'))  # Декодируем байты в строку
```

**Добавление элементов в список**:

```python
r.rpush('my_list', 'element1')  # Добавляет в конец
r.lpush('my_list', 'element2')  # Добавляет в начало
```

**Получение элементов из списка**:

```python
elements = r.lrange('my_list', 0, -1)  # Получает все элементы
print([element.decode('utf-8') for element in elements])
```

**Удаление элемента из списка**:

```python
r.lrem('my_list', 0, 'element1')  # Удаляет все вхождения 'element1'
```

**Добавление элементов в множество**:

```python
r.sadd('my_set', 'value1', 'value2', 'value3')
```

**Получение всех элементов из множества**:

```python
members = r.smembers('my_set') print([member.decode('utf-8') for member in members])
```

**Проверка на вхождение**:

```python
is_member = r.sismember('my_set', 'value1')
print(is_member)  # True или False
```

**Удаление ключа**:

```python
r.delete('my_key')
```

Эти функции представляют собой основные операции, которые вы можете выполнять с Redis с использованием Python. Redis поддерживает множество других структур данных и команд, таких как Sorted Sets, Pub/Sub и т.д., которые можно использовать в зависимости от нужд.
## <font color="#46aa63">Cellery</font>

**Celery** - это самый популярный инструмент для асинхронной обработки задач. Он позволяет выполнять задачи в фоновом режиме и гибко настраивать параметры для каждой задачи. Кроме того, `Celery` предоставляет возможности для планирования, работы с разными очередями и мониторинга выполнения задач.

Установим пакет celery:

```shell
pip install celery
```

Создадим task `add`:

```python
from celery import Celery  
  
app = Celery('myapp', broker='redis://127.0.0.1:6379')  
  
@app.task  
def add(x: int, y: int):  
    print(f"Start add: x = {x}, y= {y}")
```

Запустим `celery`:

```shell
celery -A tasks worker --loglevel=INFO
```

В Celery существуют несколько методов для вызова задач, каждый из которых имеет свои особенности и подходит для различных сценариев. Давайте рассмотрим `delay` и `apply_async`.

`delay` — это простой и удобный способ отправить задачу в очередь. Это фактически сокращенная версия `apply_async`, и она принимает только аргументы для задачи.

```python
result = add.delay(4, 6)
```

`apply_async` предоставляет больше возможностей, чем `delay`. С его помощью вы можете передавать дополнительные параметры, такие как задержка выполнения, приоритет задачи, маршрутизация и т.д.

```python
add.apply_async((4, 6), countdown=5)  
# Вызов с определенной датой и временем  
from datetime import datetime, timedelta  
  
eta = datetime.utcnow() + timedelta(seconds=15)  
add.apply_async((4, 6), eta=eta)
```

Для запуска задач в Celery с заданным интервалом, например каждую минуту, вы можете использовать `Celery Beat`. Это компонент, который управляет периодическим выполнением задач.

```python
from celery import Celery  
from celery.schedules import crontab  
  
app = Celery('myapp', broker='redis://127.0.0.1:6379')  
  
@app.task  
def add(x: int, y: int):  
    print(f"Start add: x = {x}, y= {y}")  
  
  
@app.task  
def my_periodic_task():  
    print("Задача выполнена!")  
  
  
app.conf.beat_schedule = {  
    'run-every-minute': {  
        'task': 'tasks.my_periodic_task',  
        'schedule': crontab(minute="*"),          # Каждую минуту  
    },  
}
```

Запустим `Celery Worker` и `Beat` в разных терминалах: 

```shell
celery -A tasks worker --loglevel=info
celery -A tasks beat --loglevel=info
```


## <font color="#46aa63">Понимание Jinja</font>

Теперь, когда мы узнали, как обрабатывать ответы на запросы, включая ошибки на предыдущих занятиях мы можем приступить к отображению ответов на запросы на веб-странице. На этом занятии мы узнаем, как отображать ответы от нашего API на веб- странице, используя шаблоны на основе `Jinja`, который представляет собой язык шаблонов, написанный на Python, предназначенный для облегчения процесса визуализации ответов API.

**Шаблонирование** — это процесс отображения данных, полученных от API, в различных форматах. Шаблоны действуют как компонент интерфейса в веб- приложениях.

**Jinja** — это механизм шаблонов, написанный на Python, предназначенный для облегчения процесса рендеринга ответов API. В каждом языке шаблонов есть переменные, которые заменяются фактическими значениями, переданными им при отображении шаблона, и есть теги, управляющие логикой шаблона.

Механизм шаблонов Jinja использует фигурные скобки `{ }`, чтобы отличить свои выражения и синтаксис от обычного HTML, текста и любой другой переменной в файле шаблона.

Синтаксис `{{ }}` называется блоком переменных. Синтаксис `{% %}` содержит управляющие структуры, такие как `if/else`, `циклы` и `макросы`.

Три общих синтаксических блока, используемых в языке шаблонов Jinja, включают следующее:
- `{% ... %}` – этот синтаксис используется для операторов, таких как управляющие структуры
-  `{{ todo.item }}` – этот синтаксис используется для вывода значений переданных ему выражений
- `{# This is a great API book! #}` – этот синтаксис используется при написании комментариев и не отображается на веб-странице

Переменные шаблона `Jinja` могут относиться к любому типу или объекту Python, если их можно преобразовать в строки. Тип модели, списка или словаря можно передать шаблону и отобразить его атрибуты, поместив эти атрибуты во второй блок, указанный ранее.

##### <font color="#46aa63">Фильтры</font>

Несмотря на сходство синтаксиса Python и Jinja, такие модификации, как объединение строк, установка первого символа строки в верхний регистр и т. д., не могут быть выполнены с использованием синтаксиса Python в Jinja. Поэтому для выполнения таких модификаций у нас в Jinja есть фильтры.

Фильтр отделяется от переменной вертикальной чертой `|` и может содержать необязательные аргументы в круглых скобках. Фильтр определяется в этом формате:

```txt
{{ variable | filter_name(*args) }}
```

Если нет аргументов, определение становится следующим:

```txt
{{ variable | filter_name }}
```

##### <font color="#46aa63">Фильтр по умолчанию</font>

Переменная фильтра по умолчанию используется для замены вывода переданного значения, если оно оказывается `None`:

```txt
{{ todo.item | default('This is a default todo item') }} 
This is a default todo item
```

##### <font color="#46aa63">Эвакуационный фильтр</font>

Этот фильтр преобразует символы `&`, `<`, `>`, `'` и `"` в строках  в безопасные для HTML последовательности. Используйте эту функцию, если необходимо отобразить текст, который может содержать такие символы в HTML:

```txt
{{ "<title>Todo Application</title>" | escape }} 
<title>Todo Application</title>
```

##### <font color="#46aa63">Фильтры преобразования</font>

Эти фильтры включают фильтры int и float, используемые для преобразования из одного типа данных в другой:

```txt
{{ 3.142 | int}} 
3  
{{ 31 | float }} 
31.0
```

##### <font color="#46aa63">Фильтр объединения</font>

Этот фильтр используется для объединения элементов списка в строку, как в Python:

```txt
{{ ['Packt', 'produces', 'great', 'books!'] | join(' ') }} 
Packt produces great books!
```

##### <font color="#46aa63">Фильтр длины</font>

Этот фильтр используется для возврата длины переданного объекта. Он выполняет ту же роль, что и `len()` в Python:

```txt
Todo count: {{ todos | length }}
Todo count: 4
```

Полный список фильтров и дополнительные сведения можно посмотреть в документации https://jinja.palletsprojects.com/en/3.1.x/templates/#builtin-filters.

##### <font color="#46aa63">Использование операторов if</font>

Использование операторов `if` в `Jinja` аналогично их использованию в Python. `if` операторы используются в блоках управления `{% %}`. Пример

```txt
{% if todo | length < 5 %}  
You don't have much items on your todo list!

{% else %}  
You have a busy day it seems!

{% endif %}
```

##### <font color="#46aa63">Циклы</font>

Мы также можем перебирать переменные в `Jinja`. Это может быть список, например:

```txt
{% for todo in todos %}  
<b> {{ todo.item }} </b>

{% endfor %}
```

Мы можете получить доступ к специальным переменным внутри цикла `for`, таким как `loop.index`, который дает индекс текущей итерации. Ниже приведен список специальных переменных и их описания:

| Переменная         | Описание                                                                                                    |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| loop.index         | Текущая итерация цикла. (проиндексировано c 1)                                                              |
| loop.index0        | Текущая итерация цикла. (проиндексировано c 0)                                                              |
| loop.revindex      | Количество итераций с конца цикла (проиндексировано с 1)                                                    |
| loop.revindex0     | Количество итераций с конца цикла (проиндексировано с 1)                                                    |
| loop.first         | `True`, если первая итерация.                                                                               |
| loop.last          | `True`, если последняя итерация.                                                                            |
| loop.length        | Количество элементов в последовательности.                                                                  |
| loop.cycle         | Вспомогательная функция для переключения между списками последовательностей.                                |
| loop.depth         | Указывает, насколько глубоко в рекурсивном цикле находится рендеринг в данный момент. Начинается с уровня 1 |
| loop.depth0        | Указывает, насколько глубоко в рекурсивном цикле находится рендеринг в данный момент. Начинается с уровня 0 |
| loop.previtem      | Элемент из предыдущей итерации цикла. Не определено во время первой итерации.                               |
| loop.nextitem      | Элемент из следующей итерации цикла. Не определен во время последней итерации.                              |
| loop.changed(*val) | `True`, если ранее вызывался с другим значением (или не вызывался вообще).                                  |

##### <font color="#46aa63">Макросы</font>

**Макрос в Jinja** — это функция, которая возвращает строку HTML. Основной вариант использования макросов — избежать повторения кода и вместо этого использовать один вызов функции. Например, макрос ввода определен для сокращения непрерывного определения тегов ввода в HTML-форме:

```txt
{% macro input(name, value='', type='text', size=20 %} <div class="form">

<input type="{{ type }}" name="{{ name }}" value="{{ value|escape }}" size="{{ size }}">

</div>  
{% endmacro %}
```

Теперь, чтобы быстро создать ввод в вашей форме, вызывается макрос:

```txt
{{ input('item') }}
```

Это вернет следующее:

```html
<div class="form">  
<input type="text" name="item" value="" size="20">
</div>
```

##### <font color="#46aa63">Наследование шаблонов</font>

Самая мощная функция `Jinja` — наследование шаблонов. Эта функция продвигает принцип «не повторяйся» (`DRY`) и удобна в больших веб-приложениях. Наследование шаблона — это ситуация, когда базовый шаблон определен, а дочерние шаблоны могут взаимодействовать, наследовать и заменять определенные разделы базового шаблона.

Разберем пример. Этот шаблон, который мы назовем `base.html`, определяет простой HTML-каркас документа, который вы могли бы использовать для простой страницы с двумя столбцами. Задача “дочерних” шаблонов - заполнять пустые блоки содержимым:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    {% block head %}
    <link rel="stylesheet" href="style.css" />
    <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
</head>
<body>
    <div id="content">{% block content %}{% endblock %}</div>
    <div id="footer">
        {% block footer %}
        &copy; Copyright 2008 by <a href="http://domain.invalid/">you</a>.
        {% endblock %}
    </div>
</body>
</html>
```

В этом примере теги `{% block %}` определяют четыре блока, которые могут заполнять дочерние шаблоны. Все, что делает тег `block`, - это сообщает обработчику шаблонов, что дочерний шаблон может переопределять эти заполнители в шаблоне.

Теги блоков могут находиться внутри других блоков, таких как `if`, но они всегда будут выполняться независимо от того, был ли блок `if` фактически отрисован.

Дочерний шаблон может выглядеть следующим образом:

```html
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
    {{ super() }}
    <style type="text/css">
        .important { color: #336699; }
    </style>
{% endblock %}
{% block content %}
    <h1>Index</h1>
    <p class="important">
      Welcome to my awesome homepage.
    </p>
{% endblock %}
```

Ключевым здесь является тег `{% extends %}`. Он сообщает обработчику шаблонов, что этот шаблон “расширяет” другой шаблон. Когда система шаблонов оценивает этот шаблон, она сначала находит родительский. Тег `extends` должен быть первым тегом в шаблоне. Все, что было до этого, выводится на печать в обычном режиме и может привести к путанице.  Кроме того, блок всегда будет заполнен независимо от того, оценивается ли окружающее условие как true или false.

Узнать больше о наследовании шаблонов Jinja можно в документации https://jinja.palletsprojects.com/en/3.0.x/templates/?highlight=template%20inheritance.

## <font color="#46aa63">Использование шаблонов Jinja в FastAPI</font>

Для начала нам нужно установить пакет `Jinja` и создать новую папку, `templates`, в каталоге нашего проекта. В этой папке будут храниться все наши файлы `Jinja`, которые представляют собой файлы `HTML`, смешанные с синтаксисом `Jinja`. Мы будем использовать библиотеку `CSS Bootstrap` и не будем писать собственные стили.

Библиотека Bootstrap будет загружена из CDN при загрузке страницы. Однако дополнительные активы можно хранить в другой папке. Мы рассмотрим обслуживание статических файлов на следующих занятиях.

Мы начнем с создания шаблона домашней страницы, на котором будет размещен раздел для создания новых задач. 

Установим пакет `jinja2`, `python-multipart` и создадим папку `templates`:

```shell
pip install jinja2 python-multipart
mkdir templates
```

Cоздадим в папке `templates` два новых файла, `home.html` и `todo.html`:

```shell
cd templates
touch {home,todo}.html
```

Прежде чем перейти к созданию наших шаблонов, давайте настроим `Jinja` в нашем приложении FastAPI. Создадим файл `settings.py`:

```shell
touch settings.py
```

```python
from pathlib import Path  
  
  
BASE_DIR = Path(__file__).parent
```

Изменим `POST` маршрут компонента API задач, `todo.py`:
   
```python
from fastapi import APIRouter, Path, HTTPException, status, Request, Depends
from fastapi.templating import Jinja2Templates
from settings import BASE_DIR

templates = Jinja2Templates(directory=BASE_DIR / "templates")

@todo_router.post("/todo")  
async def add_todo(request: Request, todo: Annotated[Todo, Form()]):  
    todo.id = len(todo_list) + 1  
    todo_list.append(todo)  
    return templates.TemplateResponse("todo.html", {  
        "request": request,  
        "todos": todo_list  
    })
```

Обновим маршруты GET:
   
```python
@todo_router.get("/todo", response_model=TodoItems)  
async def retrieve_todos(request: Request):  
    return templates.TemplateResponse("todo.html", {  
        "request": request,  
        "todos": todo_list  
    })
  
  
@todo_router.get("/todo/{todo_id}")  
async def get_single_todo(  
        request: Request,   
        todo_id: int = Path(..., title="The ID of the todo to retrieve")  
):  
    for todo in todo_list:  
        if todo.id == todo_id:  
            return templates.TemplateResponse("todo.html", {  
                "request": request,  
                "todo": todo  
            })  
    raise HTTPException(  
        status_code=status.HTTP_404_NOT_FOUND,  
        detail="Todo with supplied ID doesn't exist"  
    )
```

В предыдущем блоке кода мы настроили `Jinja` для просмотра каталога `templates` для обслуживания шаблонов, переданных в `templates.` 

Метод для добавления задачи также был обновлен, чтобы включить зависимость от переданного ввода. 

Сделаем поле `id`  в классе `Todo` не обязательным так как с формы мы будем получать только `item` а `id` будем назначать сами:
   
```python
from fastapi import Form

class Todo(BaseModel):  
    id: int | None = None  
    item: str  
  
    model_config = {  
        "json_schema_extra": {  
            "example": [  
                {  
                    "id": 1,  
                    "item": "Example schema!",  
                }  
            ]  
        }  
    }
```

Теперь, когда мы обновили наш код API, давайте напишем наши шаблоны. Мы начнем с написания базового шаблона `home.html`:

```html
<!DOCTYPE html> <html lang="en">  
  
<head>  
    <meta charset="utf-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1">  
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">  
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>  
  
    <title>Packt Todo Application</title>  
</head>  
  
<body>  
    <header>       <nav class="navar">  
          <div class="container-fluid" style="text-align: center;">  
                <h1>Packt Todo Application</h1>  
          </div>        </nav>    </header>  
    <div class="container-fluid">  
       {% block todo_container %}{% endblock %}  
    </div>  
</body>  
  
</html>
```

Блок **todo_container** будет определяться дочерним шаблоном. Содержимое дочернего шаблона, содержащего блок `todo_container` и расширяющего родительский шаблон, будет отображаться там.

Запустим наше приложение и посмотрим что получилось:

![[Снимок экрана 2024-09-11 в 22.30.38.png]]
Напишем шаблон todo в `todo.html`:

```html
{% extends "home.html" %}  
  
{% block todo_container %}  
<main class="container">  
    <hr>  
    <section class="container-fluid">  
        <form method="post">  
            <div class="col-auto">  
                <div class="input-group mb-3">  
                    <input aria-describedby="button-addon2" aria-label="Add a todo" class="form-control" name="item"  
                           placeholder="Purchase Packt's Python workshop course" type="text"  
                           value="{{ item }}"/>  
                    <button class="btn btn-outline-primary" data-mdb-ripple-color="dark" id="button-addon2"  
                            type="submit">  
                        Add Todo  
                    </button>  
                </div>            </div>        </form>    </section>  
  
    {% if todo %}  
    <article class="card container-fluid">  
        <br/>        <h4>Todo ID: {{ todo.id }} </h4>  
        <p>            <strong>                Item: {{ todo.item }}  
            </strong>  
        </p>    </article>    {% else %}  
    <section class="container-fluid">  
        <h2 align="center">Todos</h2>  
        <br>        <div class="card">  
            <ul class="list-group list-group-flush">  
                {% for todo in todos %}  
                <li class="list-group-item">  
                    {{ loop.index }}. <a href="/todo/{{ loop.index }}"> {{ todo.item }} </a>  
                </li>                {% endfor %}  
            </ul>  
        </div>        {% endif %}  
    </section>  
</main>  
{% endblock %}
```

В предыдущем блоке кода шаблон `todo` наследует шаблон домашней страницы. Мы также определили блок `todo_container`, содержимое которого будет отображаться в родительском шаблоне.

Шаблон задачи используется как для получения всех задач, так и для одной задачи. В результате шаблон отображает различный контент в зависимости от используемого маршрута.

В шаблоне `Jinja` проверяет, передается ли переменная `todo` с помощью блока `{% if todo %}`. Подробная информация о задаче отображается, если передается переменная задачи, в противном случае она отображает содержимое в блоке `{% else %}`, который является списком.

Обновите веб-браузер, чтобы просмотреть последние изменения:

![[Снимок экрана 2024-09-11 в 22.38.43.png]]
Добавим задачу, чтобы убедиться, что домашняя страница работает должным образом:

![[Снимок экрана 2024-09-11 в 22.45.12.png]]
На этом занятии мы узнали, что такое шаблоны, основы системы шаблонов `Jinja` и как использовать ее в FastAPI. Мы также узнали, что такое наследование шаблонов и как оно работает, на примере шаблонов главной страницы и задач.


# Задание

### Задание 1: Основы синтаксиса

Создайте шаблон Jinja, который принимает список имен и выводит их в виде маркированного списка HTML.

**Требования:**

- Используйте цикл `for`.
- Отображайте каждое имя в `<li>` элементе.


### Задание 2: Условные конструкции

Напишите шаблон, который принимает число и выводит следующее сообщение:

- "Число четное", если число четное.
- "Число нечетное", если число нечетное.

**Требования:**

- Используйте условный оператор `if`.

### Задание 3: Наследование шаблонов

Создайте базовый шаблон, который имеет заголовок и футер. Затем создайте дочерний шаблон, который наследует базовый и добавляет контент в основной блок.

**Требования:**

- Используйте `{% extends %}` и `{% block %}`.


### Задание 4: Работа с контекстом

Создайте шаблон, который принимает объект с информацией о пользователе (имя, возраст, город) и отображает это в виде карточки пользователя.

**Требования:**

- Используйте доступ к атрибутам объекта через точечную нотацию.

### Задание 5: Простое хранилище данных

Создайте класс, который позволяет пользователю добавлять, получать и удалять элементы по ключу из Redis.

Реализуйте функции:

- `set_value(key, value)`: добавляет значение по указанному ключу.
- `get_value(key)`: получает значение по ключу.
- `delete_value(key)`: удаляет значение по ключу.

### Задание 6: Счетчик

Создайте класс, который реализует простой счетчик в Redis.

Реализуйте функции:

- `increment_counter(key)`: увеличивает значение счетчика на 1.
- `get_counter(key)`: возвращает текущее значение счетчика.


### Задание 7: Хранилище списков

Создайте класс, который управляет списками задач в Redis.

Реализуйте функции:

- `add_task(task)`: добавляет задачу в список.
- `get_tasks()`: возвращает все задачи из списка.
- `remove_task(task)`: удаляет задачу из списка.



