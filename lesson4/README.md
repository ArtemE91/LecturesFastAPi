# Шаблоны в FastAPi


### План занятия

1) Понимание Jinja
2) Использование шаблонов Jinja2 в FastAPI

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

![Снимок экрана 2024-09-11 в 22.30.38.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-11%20%D0%B2%2022.30.38.png)

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

![Снимок экрана 2024-09-11 в 22.38.43.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-11%20%D0%B2%2022.38.43.png)

Добавим задачу, чтобы убедиться, что домашняя страница работает должным образом:

![Снимок экрана 2024-09-11 в 22.45.12.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-11%20%D0%B2%2022.45.12.png)

На этом занятии мы узнали, что такое шаблоны, основы системы шаблонов `Jinja` и как использовать ее в FastAPI. Мы также узнали, что такое наследование шаблонов и как оно работает, на примере шаблонов главной страницы и задач.


