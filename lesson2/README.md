# Маршрутизация в FastAPI

### План занятия

1) Маршрутизация в FastAPI
2) Класс APIRouter
3) Валидация с использованием моделей Pydantic
4) Путь, параметры и тело запроса
5) Автоматические документы FastAPI
6) Создание простого CRUD приложения

## <font color="#46aa63">Маршрутизация в FastAPI</font>

Маршрутизация является важной частью создания веб-приложения. Маршрутизация в FastAPI гибкая и простая. Маршрутизация — это процесс обработки HTTP-запросов, отправляемых клиентом на сервер. HTTP-запросы отправляются по определенным маршрутам, для которых определены обработчики для обработки запросов и ответа. Эти обработчики называются обработчиками маршрутов. Знание маршрутизации в FastAPI необходимо при создании малых и больших приложений.

##### <font color="#46aa63">Понимание маршрутизации в FastAPI</font>

Маршрут определяется для приема запросов от метода HTTP-запроса и, при необходимости, для получения параметров. Когда запрос отправляется на маршрут, приложение проверяет, определен ли маршрут перед обработкой запроса в обработчике маршрута. С другой стороны, обработчик маршрута — это функция, которая обрабатывает запрос, отправленный на сервер. Примером обработчика маршрута является функция, извлекающая записи из базы данных при отправке запроса на маршрутизатор через маршрут.

HTTP-методы — это идентификаторы для указания типа выполняемого действия. Стандартные методы включают GET, POST, PUT, PATCH, and DELETE. Вы можете узнать больше о методах HTTP на https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods

##### <font color="#46aa63">Пример маршрутизации</font>

На предыдущем занятии мы создали приложение с одним маршрутом. Маршрутизация была обработана экземпляром FastAPI() инициированным в переменной приложения:
```python
from fastapi import FastAPI    


app = FastAPI()  
  
  
@app.get("/")  
async def welcome() -> dict:  
    return {"message": "Hello World"}  
  
```

Традиционно экземпляр FastAPI() может использоваться для операций маршрутизации, как показано ранее. Однако этот метод обычно используется в приложениях, которым требуется один путь во время маршрутизации. В ситуации, когда с помощью экземпляра FastAPI создается отдельный маршрут, выполняющий уникальную функцию, приложение не сможет запустить оба маршрута, так как uvicorn может запустить только одну точку входа.

##### <font color="#46aa63">Маршрутизация с помощью класса APIRouter</font>

Класс APIRouter принадлежит пакету FastAPI и создает операции пути для нескольких маршрутов. Класс APIRouter поощряет модульность и организацию маршрутизации и логики приложений.

Класс APIRouter импортируется из пакета fastapi и создается экземпляр. Методы маршрута создаются и распространяются из созданного экземпляра, например:

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/hello")
async def hello() -> dict:
	return {"message": "Hello!"}
```

Cоздадим новую операцию пути с классом APIRouter для создания и получения задач. Создадим в папке проекта новый файл <font color="orange">todo.py</font>. Запишем в файл следующий код:

```python
from fastapi import APIRouter  
  
todo_router = APIRouter()  
todo_list = []  
  
@todo_router.post("/todo")  
async def add_todo(todo: dict) -> dict:  
    todo_list.append(todo)  
    return {"message": "Todo added successfully"}  
  
  
@todo_router.get("/todo")  
async def retrieve_todos() -> dict:  
    return {"todos": todo_list}
```

Выше мы создали два маршрута для наших операций с задачами и  <font color="orange">todo_list</font> для хранения записей. Первый маршрут добавляет задачу в список задач с помощью метода POST, а второй маршрут извлекает все элементы задачи из списка задач с помощью метода GET.

Класс APIRouter работает так же, как и класс FastAPI. Однако uvicorn не может использовать экземпляр APIRouter для обслуживания приложения, в отличие от FastAPI.  

Маршруты, определенные с помощью класса APIRouter, добавляются в экземпляр fastapi для обеспечения их видимости.

Чтобы обеспечить видимость маршрутов todo, мы включим обработчик операций пути todo_router в основной экземпляр FastAPI с помощью метода <font color="orange">include_router()</font>. Метод <font color="orange">include_router(router, ...)</font> отвечает за добавление маршрутов, определенных с помощью класса APIRouter, в экземпляр основного приложения, чтобы сделать маршруты видимыми.

Добавим todo_router в api.py:

```python
from fastapi import FastAPI  
from todo import todo_router  
  
app = FastAPI()  
  
@app.get("/")  
async def welcome() -> dict:  
    return {"message": "Hello World"}  
  
  
app.include_router(todo_router)
```

Запустим приложение:
```shell
uvicorn api:app --port 8000 --reload
```

Протестируем приложение, отправив запрос GET с помощью curl:

```shell
curl http://0.0.0.0:8000/

{"message":"Hello World"}

curl -X 'GET' 'http://127.0.0.1:8000/todo' -H 'accept: application/json'

{"todos":[]}

curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 1,"item": "First Todo is to finish this book!"}'

{"message":"Todo added successfully"}

curl -X 'GET' 'http://127.0.0.1:8000/todo' -H 'accept: application/json'

{"todos":[{"id":1,"item":"First Todo is to finish this book!"}]}
```

##### <font color="#46aa63">Валидация тела запроса с использованием моделей Pydantic</font>

В FastAPI тела запросов могут быть проверены, чтобы гарантировать отправку только определенных данных. Это крайне важно, поскольку служит для очистки данных запросов и снижения рисков вредоносных атак. Этот процесс известен как валидация.

Модель в FastAPI — это структурированный класс, который определяет, как следует получать или анализировать данные. Модели создаются путем определения подкласса класса <font color="orange">BaseModel</font> Pydantic. <font color="orange">Pydantic</font> — это библиотека Python, которая выполняет проверку данных с помощью аннотаций типа Python.

Модели, когда они определены, используются в качестве подсказок типа для объектов тела запроса и объектов запроса-ответа.

Примерная модель выглядит следующим образом:

```python
from pydantic import BaseModel

class Todo(BaseModel):
	id: int
	item: str 
```

В предыдущем блоке кода выше мы определили модель <font color="orange">Todo</font> как подкласс класса BaseModel Pydantic. Тип переменной, подсказанный классу Todo, может принимать только два поля, как определено ранее. В следующих нескольких примерах мы видим, как Pydantic помогает в проверке входных данных.

В нашем приложении todo ранее мы определили маршрут для добавления элемента в список todo. В определении маршрута мы устанавливаем тело запроса в словарь:

```python
async def add_todo(todo: dict) -> dict: 
	...
```

В примере запроса POST отправленные данные были в следующем формате:

```json
{  
	"id": id,
	"item": item 
}
```

Однако пустой словарь также мог быть отправлен без возврата какой-либо ошибки. Пользователь может отправить запрос с телом, отличным от показанного ранее. Создание модели с требуемой структурой тела запроса и присвоение ее в качестве типа телу запроса гарантирует, что будут переданы только те поля данных, которые присутствуют в модели.

Например, чтобы в предыдущем примере поля содержались только в теле запроса, создадим новый файл <font color="orange">model.py</font> и добавим в него наш класс <font color="orange">Todo</font>. Импортируем наш класс Todo в api.py и заменим тип переменной тела запроса с dict на Todo:

```python
from model import Todo

@todo_router.post("/todo")  
async def add_todo(todo: Todo) -> dict:
	...
```

Проверим новый валидатор тела запроса, отправив пустой словарь в качестве тела запроса:

```shell
curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{}'
```

Получаем ответ, указывающий на отсутствие поля `id` и `item` в теле запроса:

```json
{
	"detail":[
		{
			"type":"missing",
			"loc":["body","id"],
			"msg":"Field required",
			"input":{}
		},
		{
			"type":"missing",
			"loc":["body","item"],
			"msg":"Field required",
			"input":{}
		}
	]
}
```

Отправка запроса с правильными данными возвращает успешный ответ:

```shell
curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 2, "item": "Validation models help with input types"}'

{"message":"Todo added successfully"}
```

##### <font color="#46aa63">Вложенные модели</font>

Модели Pydantic также могут быть вложенными, например, следующие:

```python
class Item(BaseModel) 
	item: str
	status: str

class Todo(BaseModel) 
	id: int
	item: Item
```

В результате задача типа Todo будет представлена следующим образом:

```json
{  
	"id": 1,
	"item": {  
		"item": "Nested models",
		"status": "completed" 
		}
}
```

### <font color="#46aa63">Путь, параметры и тело запроса</font>

Параметры пути — это параметры, включенные в маршрут API для идентификации ресурсов. Эти параметры служат идентификатором, а иногда и связующим звеном, позволяющим выполнять дальнейшие операции в веб-приложении.

В настоящее время у нас есть маршруты для добавления задачи и получения всех задач в нашем приложении задач. Давайте создадим новый маршрут для получения одной задачи, добавив идентификатор задачи в качестве параметра пути.

В `todo.py`, добавим новый маршрут:

```python
from fastAPI import APIRouter, Path

@todo_router.get("/todo/{todo_id}")  
async def get_single_todo(todo_id: int = Path(..., title="The ID of the todo to retrieve.")) -> dict:  
    for todo in todo_list:  
        if todo.id == todo_id:  
            return {"todo": todo}  
    return {"message": "Todo with supplied ID doesn't exist."}
```

В предыдущем блоке кода, `{todo_id}` является параметром пути. Этот параметр позволяет приложению возвращать совпадающую задачу с переданным идентификатором.

Проверим маршрут:

```shell
curl -X 'GET' 'http://127.0.0.1:8000/todo/1' -H 'accept: application/json'
```

В предыдущем запросе `GET` , 1 — это параметр пути. Здесь мы говорим нашему приложению `todo` вернуть элемент с идентификатором 1.
Выполнение предыдущего запроса приводит к следующему ответу:

```json
{
	"todo": {
		"id": 1,
		"item": "First Todo is to finish this book!"
	}
}
```

FastAPI также предоставляет класс <font color="orange">Path</font>, который отличает параметры пути от других аргументов, присутствующих в функции маршрута. Класс `Path` также помогает дать параметрам маршрута больше контекста во время документации, автоматически предоставляемой OpenAPI через Swagger и ReDoc, и действует как валидатор.

Изменим определение маршрута в `todo.py`:

```python
@todo_router.get("/todo/{todo_id}")  
async def get_single_todo(todo_id: int = Path(..., title="The ID of the todo to retrieve")) -> dict:
	...
```

Класс `Path` принимает первый позиционный аргумент, равный `None` или многоточие (...). Если в качестве первого аргумента задано многоточие (...), параметр пути становится обязательным. Класс `Path` также содержит аргументы, используемые для числовой проверки, если параметр пути является числом. Определения включают `gt` и `le` – `gt` означает больше, а `le` означает меньше. При использовании маршрут будет проверять параметр пути на соответствие этим аргументам.

##### <font color="#46aa63">Параметры запроса</font>

Параметр запроса — это необязательный параметр, который обычно появляется после вопросительного знака в URL-адресе. Он используется для фильтрации запросов и возврата определенных данных на основе предоставленных запросов.

В функции обработчика маршрута аргумент, не совпадающий с параметром пути, является запросом. Вы также можете определить запрос, создав экземпляр класса FastAPI `Query()` в аргументе функции, например:

```python
async query_route(query: Query(None))
	return query
```

Мы рассмотрим варианты использования параметров запроса позже в , когда будем обсуждать, как создавать более продвинутые приложения, чем приложение todo.

##### <font color="#46aa63">Тело запроса</font>

Тело запроса — это данные, которые вы отправляете в свой `API`, используя метод маршрутизации, такой как `POST` и `UPDATE`.

Метод `POST` используется, когда необходимо выполнить вставку на сервер, а метод `UPDATE` используется, когда необходимо обновить существующие данные на сервере.

Выше мы отправляли запрос `POST`, где тело запроса выглядело следующим образом:

```json
{  
	"id": 2,
	"item": "Validation models help with input types.." 
}
```

Всего лишь с помощью аннотации типов Python, `FastAPI`:
- Читает тело запроса как JSON.
- Приводит к соответствующим типам (если есть необходимость).
- Проверяет корректность данных.
    - Если данные некорректны, будет возращена читаемая и понятная ошибка, показывающая что именно и в каком месте некорректно в данных.
- Складывает полученные данные в параметр `todo`.
    - Поскольку внутри функции вы объявили его с типом `Todo`, то теперь у вас есть поддержка со стороны редактора (автодополнение и т.п.) для всех атрибутов и их типов.
- Генерирует декларативное описание модели в виде [JSON Schema](https://json-schema.org/), так что вы можете его использовать где угодно, если это имеет значение для вашего проекта.
- Эти схемы являются частью сгенерированной схемы OpenAPI и используются для автоматического документирования UI.

### <font color="#46aa63">Автоматические документы FastAPI</font>

FastAPI генерирует определения схемы JSON для наших моделей и автоматически документирует наши маршруты, включая их тип тела запроса, параметры пути и запроса, а также модели ответов. Эта документация бывает двух типов:
- Swagger
- ReDoc

##### <font color="#46aa63">Swagger</font>

Документация, размещенная на swagger, предоставляет интерактивную среду для тестирования нашего API. Вы можете получить к нему доступ, добавив `/docs` к адресу приложения. В веб-браузере перейдите по адресу http://127.0.0.1:8000/docs:

![Снимок экрана 2024-09-10 в 11.47.23.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-10%20%D0%B2%2011.47.23.png)

Интерактивная документация позволяет нам тестировать наши методы. Добавим задачу из интерактивной документации:

![Снимок экрана 2024-09-10 в 12.02.08.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-10%20%D0%B2%2012.02.08.png)

##### <font color="#46aa63">Redoc</font>

Документация ReDoc дает более подробное и прямое представление о моделях, маршрутах и API. Вы можете получить к нему доступ, добавив `/redoc` к адресу приложения. В веб-браузере перейдите по адресу http://127.0.0.1:8000/redoc:

![Снимок экрана 2024-09-10 в 12.05.55.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-10%20%D0%B2%2012.05.55.png)

Чтобы правильно сгенерировать JSON схему, вы можете указать примеры того, как пользователь будет заполнять данные в модели. Давайте добавим пример схемы в нашу модель Todo:

```python
from pydantic import BaseModel  
  
  
class Todo(BaseModel):  
    id: int  
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

Обновим страницу документации для `ReDoc` и нажмем `Add Todo` на левой панели. Пример показан на правой панели:

![Снимок экрана 2024-09-10 в 12.14.10.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-10%20%D0%B2%2012.14.10.png)

### <font color="#46aa63">Создание простого CRUD приложения</font>

Мы создали маршруты для создания и получения задач. Построим маршруты для обновления и удаления добавленных задач. Начнем с создания модели тела запроса для маршрута `UPDATE` в `model.py`:

```python
class TodoItem(BaseModel):  
    item: str  
  
    model_config = {  
        "json_schema_extra": {  
            "example": [  
                {  
                    "item": "Read the next chapter of the book",  
                }  
            ]  
        }  
    }
```

Далее давайте напишем маршрут для обновления задачи в `todo.py`:

```python
from model import Todo, TodoItem


@todo_router.put("/todo/{todo_id}")  
async def update_todo(todo_data: TodoItem, todo_id: int = Path(..., title="The ID of the todo to be updated")) -> dict:  
    for todo in todo_list:  
        if todo.id == todo_id:  
            todo.item = todo_data.item   
            return {"message": "Todo updated successfully."}  
    return {"message": "Todo with supplied ID doesn't exist."}
```

Протестируем новый маршрут. Во-первых, давайте добавим задачу:

```shell
curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 1, "item": "Example Schema!"}'

{"message":"Todo added successfully"}
```

Далее давайте обновим задачу, отправив запрос `PUT`:

```shell
curl -X 'PUT' 'http://127.0.0.1:8000/todo/1' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"item": "Read the next chapter of the book."}'

{"message":"Todo updated successfully."}
```

Проверим, что наша задача действительно была обновлена:

```shell
curl -X 'GET' 'http://127.0.0.1:8000/todo/1' -H 'accept: application/json'

{"todo":{"id":1,"item":"Read the next chapter of the book."}}
```

Из возвращенного ответа мы видим, что задача успешно обновлена. Теперь давайте создадим маршрут для удаления задачи и всех задач.

В `todo.py`, добавим новые маршруты:
```python
@todo_router.delete("/todo/{todo_id}")  
async def delete_single_todo(todo_id: int) -> dict:  
    for index in range(len(todo_list)):  
        todo = todo_list[index]  
        if todo.id == todo_id:  
            todo_list.pop(index)   
            return {"message": "Todo deleted successfully."}  
    return {"message": "Todo with supplied ID doesn't exist."}  
  
  
@todo_router.delete("/todo")  
async def delete_all_todo() -> dict:  
    todo_list.clear()   
    return {"message": "Todos deleted successfully."}
```

Протестируем маршрут удаления. Сначала мы добавим задачи:

```shell
curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 1, "item": "Example Schema 1"}'

curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 2, "item": "Example Schema 2"}'

curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 3, "item": "Example Schema 3"}'
```

Удалим задачу с `id=1`:
```shell
curl -X 'DELETE' 'http://127.0.0.1:8000/todo/1' -H 'accept: application/json'

{"message":"Todo deleted successfully."}
```

Проверим, что задача была удалена, отправив запрос GET для получения задачи:

```shell
curl -X 'GET' 'http://127.0.0.1:8000/todo/1' -H 'accept: application/json'

{"message":"Todo with supplied ID doesn't exist."}
```

Удалим оставшиеся задачи и проверим что они были успешно удалены:

```shell
curl -X 'DELETE' 'http://127.0.0.1:8000/todo' -H 'accept: application/json'

{"message":"Todos deleted successfully."}

curl -X 'GET' 'http://127.0.0.1:8000/todo' -H 'accept: application/json'

{"todos":[]}
```

На этом занятии мы создали приложение с CRUD операциями, объединив уроки, извлеченные из предыдущих разделов. Подтвердив тело запроса, мы смогли убедиться, что в API отправляются правильные данные. Включение параметров пути в наши маршруты также позволило нам получить и удалить одну задачу из нашего списка задач.