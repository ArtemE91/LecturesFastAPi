# Модели ответов и обработка ошибок


### План занятия

1) Ответы в FastAPI
2) Построение модели ответа
3) mypy
4) Обработка ошибок

## <font color="#46aa63">Ответы в FastAPI</font>

Модели ответов служат шаблонами для возврата данных из пути маршрута API. Они построены на Pydantic для правильной обработки ответов на запросы, отправленные на сервер.

Обработка ошибок включает методы и действия, связанные с обработкой ошибок в приложении. Эти методы включают возврат адекватных кодов состояния ошибки и сообщений об ошибках.

Заголовок ответа состоит из статуса запроса и дополнительной информации, необходимой для доставки тела ответа. Примером информации, содержащейся в заголовке ответа, является `Content-Type`, который сообщает клиенту возвращаемый тип содержимого.

Тело ответа, с другой стороны, представляет собой данные, запрошенные клиентом с сервера. Тело ответа определяется из переменной заголовка `Content-Type` наиболее часто используемой является `application/json`. В предыдущей главе возвращаемый список задач был телом ответа.

##### <font color="#46aa63">Коды состояния</font>

Коды состояния — это уникальные короткие коды, выдаваемые сервером в ответ на запрос клиента. Коды состояния ответа сгруппированы в пять категорий, каждая из которых обозначает отдельный ответ:

- `1XX`: Запрос получен
- `2XX`: Запрос выполнен успешно
- `3XX`: Запрос перенаправлен
- `4XX`: Ошибка клиента
- `5XX`: Ошибка сервера

Полный список кодов состояния HTTP можно найти по адресу https://developer.mozilla.org/ru/docs/Web/HTTP/Status.

Первая цифра кода состояния определяет его категорию. Общие коды состояния включают `200` для успешного запроса, `404` для запроса, не найденного и `500` указывающего на внутреннюю ошибку сервера.

Стандартная практика создания веб-приложений, независимо от платформы, заключается в возврате соответствующих кодов состояния для отдельных событий. Код состояния `400` не должен возвращаться в случае ошибки сервера. Точно так же код состояния `200` не должен возвращаться для неудачной операции запроса.

## <font color="#46aa63">Построение моделей ответа</font>

Мы можем объявить тип ответа, указав аннотацию **возвращаемого значения** для  функции операции пути.

FastAPI позволяет использовать **аннотации типов** таким же способом, как и для ввода данных в  **параметры** функции, вы можете использовать модели Pydantic, списки, словари, скалярные типы (такие, как int, bool и т.д.).

Пример:

```python
class Item(BaseModel): 
	name: str 
	description: str | None = None 
	price: float 
	tax: float | None = None 
	
@app.post("/items/") 
async def create_item(item: Item) -> Item: 
	return item
```

FastAPI будет использовать этот возвращаемый тип для:

- **Валидации** ответа.
- Если данные невалидны (например, отсутствует одно из полей), это означает, что код _вашего_ приложения работает некорректно и функция возвращает не то, что вы ожидаете. В таком случае приложение вернет server error вместо того, чтобы отправить неправильные данные. Таким образом, вы и ваши пользователи можете быть уверены, что получите корректные данные в том виде, в котором они ожидаются.
- Добавит **JSON схему** для ответа внутри операции пути OpenAPI.
- Она будет использована для **автоматически генерируемой документации**.
- А также - для автоматической кодогенерации пользователями.

Но самое важное:

- Ответ будет **ограничен и отфильтрован** - т.е. в нем останутся только те данные, которые определены в возвращаемом типе.
- Это особенно важно для **безопасности**, далее мы рассмотрим эту тему подробнее.

##### <font color="#46aa63">Параметр response_model</font>

Бывают случаи, когда вам необходимо (или просто хочется) возвращать данные, которые не полностью соответствуют объявленному типу.

Допустим, вы хотите, чтобы ваша функция **возвращала словарь (dict)** или объект из базы данных, но при этом **объявляете выходной тип как модель Pydantic**. Тогда именно указанная модель будет использована для автоматической документации, валидации и т.п. для объекта, который вы вернули (например, словаря или объекта из базы данных).

Но если указать аннотацию возвращаемого типа, статическая проверка типов будет выдавать ошибку (абсолютно корректную в данном случае). Она будет говорить о том, что ваша функция должна возвращать данные одного типа (например, dict), а в аннотации вы объявили другой тип (например, модель Pydantic).

В таком случае можно использовать параметр `response_model` внутри декоратора операции пути вместо аннотации возвращаемого значения функции.

Параметр `response_model` может быть указан для любой операции пути:

- `@app.get()`
- `@app.post()`
- `@app.put()`
- `@app.delete()`
- и др.

Пример:

```python
class Item(BaseModel): 
	name: str 
	description: str | None = None 
	price: float 
	tax: float | None = None 
	
@app.post("/items/", response_model=Item) 
async def create_item(item: Item) -> Any: 
	return item
```

`response_model` принимает те же типы, которые можно указать для какого-либо поля в модели Pydantic. Таким образом, это может быть как одиночная модель Pydantic, так и `список (list)` моделей Pydantic. Например, `List[Item]`.

FastAPI будет использовать значение `response_model` для того, чтобы автоматически генерировать документацию, производить валидацию и т.п. А также для **конвертации и фильтрации выходных данных** в объявленный тип.

##### <font color="#46aa63">Приоритет response_model</font>

Если одновременно указать аннотацию типа для ответа функции и параметр `response_model` - последний будет иметь больший приоритет и FastAPI будет использовать именно его.

Таким образом вы можете объявить корректные аннотации типов к вашим функциям, даже если они возвращают тип, отличающийся от указанного в `response_model`. Они будут считаны во время статической проверки типов вашими помощниками, например, mypy. При этом вы все так же используете возможности FastAPI для автоматической документации, валидации и т.д. благодаря `response_model`.

Вы можете указать значение `response_model=None`, чтобы отключить создание модели ответа для данной операции пути. Это может понадобиться, если вы добавляете аннотации типов для данных, не являющихся валидными полями Pydantic.

На втором занятии мы описали с вами маршрут который возвращает список  `todo`:

```python
@todo_router.get("/todo")  
async def retrieve_todos() -> dict:  
    return {"todos": todo_list}
```

Маршрут возвращает весь контент, хранящийся в массиве `todo_list` но мы не хотим возвращать  `id` записи. Чтобы указать возвращаемую информацию, нам пришлось бы либо отделить отображаемые данные, либо ввести дополнительную логику. К счастью, мы можем создать модель, содержащую поля, которые мы хотим вернуть, и добавить ее в определение нашего маршрута, используя аргумент `response_model` либо в аннотацию типа.

Давайте обновим маршрут, который извлекает все задачи, чтобы он возвращал массив только элементов задач, а не идентификаторов. Начнем с определения нового класса модели для возврата списка дел в `model.py`:

```python
class TodoItems(BaseModel):  
    todos: list[TodoItem]  
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "todos": [  
                    {"item": "Example schema 1!"},  
                    {"item": "Example schema 2!"},  
                ]  
            }  
        }  
    }
```

В предыдущем блоке кода мы определили новую модель, `TodoItems`, которая возвращает список переменных, содержащихся в модели `TodoItem`. Давайте обновим наш маршрут в `todo.py` добавив в него модель ответа:

```python
from model import Todo, TodoItem, TodoItems

@todo_router.get("/todo", response_model=TodoItems)  
async def retrieve_todos() -> dict:  
    return {"todos": todo_list}
```

Активируем виртуальную среду и запустим приложение:

```shell
source venv/bin/activate
uvicorn api:app --host=127.0.0.1 --port 8000 --reload
```

Добавим новые задачи:

```shell
curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 1, "item": "Example Schema 1"}'

curl -X 'POST' 'http://127.0.0.1:8000/todo' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"id": 2, "item": "Example Schema 2"}'
```

Получим список дел:

```shell
curl -X 'GET' 'http://127.0.0.1:8000/todo' -H 'accept: application/json'

{"todos":[{"item":"Example Schema 1"},{"item":"Example Schema 2"}]}
```

Мы указали в `response_model` модель `TodoItems`, в которой отсутствует поле, содержащее `id` - и оно будет исключено из ответа. Таким образом **FastAPI** позаботится о фильтрации ответа и исключит из него всё, что не указано в выходной модели (при помощи Pydantic).

##### <font color="#46aa63">Другие аннотации типов</font>

Бывают случаи, когда вы возвращаете что-то, что не является валидным типом для Pydantic и вы указываете аннотацию ответа функции только для того, чтобы работала поддержка различных инструментов (редактор кода, mypy и др.).

Самый частый сценарий использования - это возвращать `Response`:

```python
from fastapi import FastAPI, Response
from fastapi.responses import JSONResponse, RedirectResponse

app = FastAPI()


@app.get("/portal")
async def get_portal(teleport: bool = False) -> Response:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return JSONResponse(content={"message": "Here's your interdimensional portal."})
```

Это поддерживается FastAPI по-умолчанию, т.к. аннотация проставлена в классе (или подклассе) `Response`.

И ваши помощники разработки также будут счастливы, т.к. оба класса `RedirectResponse`и `JSONResponse` являются подклассами `Response`. Таким образом мы получаем корректную аннотацию типа.

Мы также можем указать подкласс `Response` в аннотации типа:

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()


@app.get("/teleport")
async def get_teleport() -> RedirectResponse:
    return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

Это сработает, потому что `RedirectResponse` является подклассом `Response` и FastAPI автоматически обработает этот простейший случай.

##### <font color="#46aa63">Некорректные аннотации типов</font>

Но когда вы возвращаете какой-либо другой произвольный объект, который не является допустимым типом Pydantic (например, объект из базы данных), и вы аннотируете его подобным образом для функции, FastAPI попытается создать из этого типа модель Pydantic и потерпит неудачу.

То же самое произошло бы, если бы у вас было что-то вроде Union различных типов и один или несколько из них не являлись бы допустимыми типами для Pydantic. Например, такой вариант приведет к ошибке 💥:

```python
from fastapi import FastAPI, Response
from fastapi.responses import RedirectResponse

app = FastAPI()


@app.get("/portal")
async def get_portal(teleport: bool = False) -> Response | dict:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return {"message": "Here's your interdimensional portal."}
```

...такой код вызовет ошибку, потому что в аннотации указан неподдерживаемый Pydantic тип. А также этот тип не является классом или подклассом `Response`.

##### <font color="#46aa63">Возможно ли отключить генерацию модели ответа?</font>

Продолжим рассматривать предыдущий пример. Допустим, что мы хотим отказаться от автоматической валидации ответа, документации, фильтрации и т.д.

Но в то же время, хотим сохранить аннотацию возвращаемого типа для функции, чтобы обеспечить работу помощников и анализаторов типов (например, mypy).

В таком случае, мы можем отключить генерацию модели ответа, указав `response_model=None`:

```python
from fastapi import FastAPI, Response
from fastapi.responses import RedirectResponse

app = FastAPI()


@app.get("/portal", response_model=None)
async def get_portal(teleport: bool = False) -> Response | dict:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return {"message": "Here's your interdimensional portal."}
```

Тогда FastAPI не станет генерировать модель ответа и мы сможем сохранить такую аннотацию типа, которая нам требуется, никак не влияя на работу FastAPI. 

##### <font color="#46aa63">Параметры модели ответа</font>

Модель ответа может иметь значения по умолчанию, например:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

- `description: str | None = None` , где `None` является значением по умолчанию.
- `tax: float = 10.5`, где `10.5` является значением по умолчанию.
- `tags: list[str] = []`, где пустой список `[]` является значением по умолчанию.

но мы, возможно, хотели бы исключить их из ответа, если данные поля не были заданы явно.

Например, у нас есть модель с множеством необязательных полей в NoSQL базе данных, но мы не хотим отправлять в качестве ответа очень длинный JSON с множеством значений по умолчанию.

Для этого мы будем использовать параметр `response_model_exclude_unset`:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

Тогда значения по умолчанию не будут включены в ответ. В нем будут только те поля, значения которых фактически были установлены.

Итак, если мы отправим запрос на данную маршрут для элемента, с ID = `Foo` - ответ (с исключенными значениями по-умолчанию) будет таким:

```json
{
    "name": "Foo",
    "price": 50.2
}
```

Мы также можем использовать параметры декоратора операции пути, такие, как `response_model_include` и `response_model_exclude`.

Они принимают аргументы типа `set`, состоящий из строк (`str`) с названиями атрибутов, которые либо требуется включить в ответ (при этом исключив все остальные), либо наоборот исключить (оставив в ответе все остальные поля).

Это можно использовать как быстрый способ исключить данные из ответа, не создавая отдельную модель Pydantic.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}


@app.get(
    "/items/{item_id}/name",
    response_model=Item,
    response_model_include={"name", "description"},
)
async def read_item_name(item_id: str):
    return items[item_id]


@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude={"tax"})
async def read_item_public_data(item_id: str):
    return items[item_id]
```

Используйте параметр `response_model` у _декоратора операции пути_ для того, чтобы задать модель ответа и в большей степени для того, чтобы быть уверенным, что приватная информация будет отфильтрована.

А также используйте `response_model_exclude_unset`, чтобы возвращать только те значения, которые были заданы явно.


## <font color="#46aa63">mypy</font>

**Mypy** - это средство статической проверки типов для Python.

Средства проверки типов помогают убедиться в том, что вы правильно используете переменные и функции в своем коде. С вашей помощью в ваши программы на Python можно добавлять подсказки по типам (`PEP 484`), и они понадобятся вам при неправильном использовании этих типов.

Python - динамический язык, поэтому обычно вы увидите ошибки в своем коде только при попытке запустить его. Mary - это статический инструмент проверки, который позволяет находить ошибки в ваших программах, даже не запуская их!

Установим пакет `mypy`:

```shell
pip install mypy
```

Опишем конфигурационный файл `mypy.ini` в корне проекта:

```ini
 [mypy]  
python_version = 3.11  
  
ignore_errors = False  
ignore_missing_imports = True  
implicit_reexport = False  
strict_optional = False  
strict_equality = True  
no_implicit_optional = True  
warn_unused_ignores = True  
warn_redundant_casts = True  
warn_unused_configs = True  
warn_unreachable = True  
warn_no_return = True  
  
exclude = venv  
  
plugins = pydantic.mypy  
  
[mypy-tenacity.*]  
implicit_reexport = True
```

Подробно по настройки файла и значениям аргументов можно почитать здесь https://mypy.readthedocs.io/en/stable/config_file.html#config-file

Проверим наш код `mypy`:
```shell
mypy .

Success: no issues found in 4 source files
```


## <font color="#46aa63">Обработка ошибок</font>

Ошибки запросов могут возникать из-за попыток доступа к несуществующим ресурсам, защищенным страницам без достаточных разрешений и даже из-за ошибок сервера. Ошибки в FastAPI обрабатываются путем создания исключения с использованием класса FastAPI’s **HTTPException**.

Класс `HTTPException` принимает три аргумента:

- `status_code`: Код состояния, который будет возвращен для этого сбоя
- `detail`: Сопроводительное сообщение для отправки клиенту  
- `headers`: Необязательный параметр для ответов, требующих заголовков

В наших определениях пути маршрута задачи мы возвращаем сообщение, когда задача не может быть найдена. Мы будем обновлять его, чтобы вызывать `HTTPException`. `HTTPException` позволяет нам вернуть адекватный код ответа на ошибку.

В нашем текущем приложении получение несуществующей задачи возвращает код статуса ответа `200` вместо кода статуса ответа `404` на http://127.0.0.1:8000/docs:

![Снимок экрана 2024-09-11 в 15.17.02.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-11%20%D0%B2%2015.17.02.png)

Обновляя маршруты для использования класса `HTTPException`, мы можем возвращать соответствующие детали в нашем ответе. В `todo.py`, обновим маршруты для получения, обновления и удаления списка дел:

```python
from fastapi import APIRouter, Path, HTTPException, status

@todo_router.get("/todo/{todo_id}")  
async def get_single_todo(todo_id: int = Path(..., title="The ID of the todo to retrieve")) -> dict:  
    for todo in todo_list:  
        if todo.id == todo_id:  
            return {"todo": todo}  
    raise HTTPException(  
        status_code=status.HTTP_404_NOT_FOUND,  
        detail="Todo with supplied ID doesn't exist"  
    )  
  
  
@todo_router.put("/todo/{todo_id}")  
async def update_todo(todo_data: TodoItem, todo_id: int = Path(..., title="The ID of the todo to be updated")) -> dict:  
    for todo in todo_list:  
        if todo.id == todo_id:  
            todo.item = todo_data.item  
            return {"message": "Todo updated successfully."}  
    raise HTTPException(  
        status_code=status.HTTP_404_NOT_FOUND,  
        detail="Todo with supplied ID doesn't exist",  
    )  
  
  
@todo_router.delete("/todo/{todo_id}")  
async def delete_single_todo(todo_id: int) -> dict:  
    for index in range(len(todo_list)):  
        todo = todo_list[index]  
        if todo.id == todo_id:  
            todo_list.pop(index)  
            return {"message": "Todo deleted successfully."}  
    raise HTTPException(  
        status_code=status.HTTP_404_NOT_FOUND,  
        detail="Todo with supplied ID doesn't exist"  
    )
```

Теперь давайте повторим попытку получения несуществующей задачи, чтобы убедиться, что возвращается правильный код ответа:

![Снимок экрана 2024-09-11 в 15.16.29.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-11%20%D0%B2%2015.16.29.png)

На этом занятии мы узнали, как возвращать правильные коды ответов клиентам, а также переопределять код состояния по умолчанию. Также важно отметить, что код состояния успешный по умолчанию — 200.