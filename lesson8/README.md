# Тестирование и развертывание приложений FastAPI

### План занятия

1) Модульное тестирование с помощью pytest
2) Настройка тестовой среды
3) Написание тестов для конечных точек REST API
4) Тестовое покрытие
5) Подготовка к развертыванию
6) Развертывание с помощью Docker


Тестирование является неотъемлемой частью цикла разработки приложения. Тестирование приложений выполняется для обеспечения правильного функционирования приложения и легкого обнаружения аномалий в приложении перед развертыванием в рабочей среде. Хотя мы вручную тестировали конечную точку нашего приложения на последних занятиях, мы научимся автоматизировать эти тесты.

### <font color="#46aa63">Модульное тестирование с помощью pytest</font>

Модульное тестирование - это процедура тестирования, при которой тестируются отдельные компоненты приложения. Эта форма тестирования позволяет нам проверить работоспособность отдельных компонентов. Например, модульные тесты используются для тестирования отдельных маршрутов в приложении, чтобы убедиться, что возвращаются правильные ответы.

На этом занятии мы будем использовать `pytest`, библиотеку тестирования `Python`, для проведения наших операций модульного тестирования. Хотя `Python` поставляется со встроенной библиотекой модульного тестирования под названием `unittest` библиотека, `pytest` имеет более короткий синтаксис и более предпочтительна для тестирования приложений. Давайте установим `pytest` и напишем наш первый образец теста.

Давайте установим библиотеку `pytest`:

```shell
pip install pytest
```

Создадим папку с именем `tests`, в которой будут храниться тестовые файлы для нашего приложения:

```shell
mkdir tests && cd tests
touch __init__.py
touch test_arthmetic_operations.py
```

Сначала определим функции, которые выполняют арифметические операции. В модуле тестов добавим следующее:

```python
def add(a: int, b: int) -> int:  
    return a + b  
  
  
def subtract(a: int, b: int) -> int:  
    return b - a  
  
  
def multiply(a: int, b: int) -> int:  
    return a * b  
  
  
def divide(a: int, b: int) -> int:  
    return b // a
```

Теперь, когда мы определили операции для тестирования, мы создадим функции, которые будут обрабатывать эти тесты. В тестовых функциях определяется операция, которая должна быть выполнена. Ключевое слово `assert` используется для проверки того, что вывод в левой части соответствует результат операции в правой части. В нашем случае мы будем проверять, равны ли арифметические операции их соответствующим результатам.

Добавьте следующее в функции в `tests`:

```python
def test_add() -> None:   
    assert add(1, 1) == 2  
          
def test_subtract() -> None:   
    assert subtract(2, 5) == 3  
          
def test_multiply() -> None:   
    assert multiply(10, 10) == 100  
          
def test_divide() -> None:  
    assert divide(25, 100) == 4
```

Имея тестовые функции, мы готовы запустить тестовый модуль. Тесты можно выполнить, выполнив команду `pytest`. Однако эта команда запускает все тестовые модули, содержащиеся в пакете. Для выполнения одного теста в качестве аргумента передается имя тестового модуля. Запустим тестовый модуль:

```shell
pytest tests/test_arithmetic_operations.py
```

Тесты определили все пройдено. Об этом свидетельствует ответ, выделенный зеленым цветом:

![Снимок экрана 2024-09-22 в 20.45.15.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-22%20%D0%B2%2020.45.15.png)

Провалившиеся тесты, а также точка, сбоя выделены красным цветом. Например, скажем, мы модифицируем функцию `test_add()` как таковую:

```python
def test_add() -> None:  
    assert add(1, 1) == 11
```

На следующем рисунке неудачный тест, а также точка отказа выделены красным цветом.

![Снимок экрана 2024-09-22 в 20.47.46.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-22%20%D0%B2%2020.47.46.png)

Тест не пройден в операторе `assert`, где отображается правильный результат `2`.

Сбой резюмируется как `AssertionError`, что говорит нам о том, что тест не пройден из-за неверного утверждения` (2 == 11)`.

Теперь, когда у нас есть представление о том, как работает `pytest` , давайте взглянем на фикстуры в `pytest`.

##### <font color="#46aa63">Устранение повторений с помощью фикстур pytest</font>

Фикстуры - это повторно используемые функции, определенные для возврата данных, необходимых в тестовых функциях. Фикстуры оборачиваются декоратором `pytest.fixture`. Пример использования фикстуры - возврат экземпляра приложения для выполнения тестов для конечных точек API. Фикстура может использоваться для определения клиента приложения, который возвращается и используется в тестовых функциях, что устраняет необходимость переопределять экземпляр приложения в каждом тесте.

Давайте посмотрим на пример:

```python
import pytest  
  
from models.events import EventUpdate


@pytest.fixture  
def event() -> EventUpdate:  
    return EventUpdate(  
        title="FastAPI Book Launch",  
        description="We will be discussing the contents of the FastAPI book in this event. "  
                    "Ensure to come with your own copy to win gifts!",  
        tags=["python", "fastapi", "book", "launch"],  
        location="Google Meet"  
)  
  
  
def test_event_name(event: EventUpdate) -> None:  
    assert event.title == "FastAPI Book Launch"
```

В предыдущем блоке кода мы определили фикстуру, которая возвращает экземпляр `pydantic` модели `EventUpdate`. Это приспособление передается в качестве аргумента в функцию `test_event_name`, что позволяет сделать свойства доступными.

Декоратор фикстуры может дополнительно принимать аргументы. Одним из этих аргументов является область действия - область действия фикстуры сообщает `pytest` какова продолжительность функции фикстуры.  
На этом занятии мы будем использовать две области видимости:

- **session** - эта область сообщает `pytest`, что нужно создать экземпляр функции один раз для всего сеанса тестирования.
- **module** - эта область инструктирует `pytest` выполнять добавленную функцию только после выполнения тестового модуля.

Теперь, когда мы знаем, что такое фикстура, давайте настроим нашу тестовую среду.

### <font color="#46aa63">Настройка тестовой среды</font>

В предыдущем разделе мы узнали об основах тестирования, а также о том, что такое фикстуры. Теперь мы проверим конечные точки для `CRUD` операций, а также аутентификацию пользователей. Чтобы протестировать наши `асинхронные API`, мы будем использовать `httpx` и установим библиотеку `pytest-asyncio`, чтобы мы могли протестировать наш `асинхронный API`.

```shell
pip install httpx pytest-asyncio
```

Для работы с базой данных в тестовом режиме будем использовать пакет `mongomock-motor`. Установим его:

```shell
pip install mongomock-motor
```

Далее мы создадим файл конфигурации с именем `pytest.ini`. Добавим в него следующий код:

```ini
[pytest]  
asyncio_mode=auto
addopts=-p no:warnings
```

Файл конфигурации читается при запуске `pytest`. `pytest` автоматически определит, как обрабатывать асинхронные тесты. Если тестовая функция является корутиной (т.е. определена с помощью `async def`), `pytest` будет использовать асинхронный режим. Если тестовая функция обычная (не асинхронная), будет использован синхронный режим. Параметр `addopts` полностью отключает предупреждения которые выдает `pytest`.

Имея файл конфигурации, давайте создадим тестовый модуль `conftest.py`, который будет отвечать за создание экземпляра нашего приложения, необходимого для тестовых файлов. В папке с тестами создадим модуль `conftest`:

```shell
touch tests/conftest.py
```

Мы начнем с импорта необходимых зависимостей в `conftest.py`:

```python
from beanie import init_beanie  
from httpx import AsyncClient  
  
import pytest  
from mongomock_motor import AsyncMongoMockClient  
  
from main import app  
from models.events import Event  
from models.users import User
```

В предыдущем блоке кода мы импортировали модули `asyncio`, `httpx` и `pytest`. Модуль `asyncio` будет использоваться для создания активного сеанса цикла, чтобы тесты выполнялись в одном потоке, чтобы избежать конфликтов. Тест `httpx` будет действовать как асинхронный клиент для выполнения `CRUD` операций `HTTP`. Библиотека `pytest` необходима для определения фикстур.
`AsyncMongoMockClient` необходим для имитации работы mongo.

Наконец, давайте определим клиентскую фикстуру по умолчанию, которая возвращает экземпляр нашего приложения, работающего асинхронно через `httpx` и `mongo_mock` который будет выполняться при запуске тестов:

```python
@pytest.fixture(autouse=True, scope="session")  
async def mongo_mock():  
    client = AsyncMongoMockClient()  
    await init_beanie(document_models=[Event, User], database=client.get_database(name="db"))  
  
@pytest.fixture(scope="session")
async def test_async_client():  
    async with AsyncClient(app=app, base_url="http://testserver") as client:  
        yield client
```

В предыдущем блоке кода приложение запускается как `AsyncClient`, который остается активным до конца тестового сеанса.

### <font color="#46aa63">Написание тестов для конечных точек REST API</font>

Когда все готово, давайте создадим модуль `test_login.py`, в котором мы будем тестировать маршруты аутентификации:

```shell
touch tests/test_login.py
```

##### <font color="#46aa63">Тестирование маршрута регистрации</font>

Первая конечная точка, которую мы будем тестировать, — это конечная точка регистрации. Мы добавим декоратор `pytest.mark.asyncio`, который сообщает `pytest` что нужно рассматривать это как асинхронный тест. Давайте определим функцию и полезную нагрузку запроса:

```python
import pytest  
from httpx import AsyncClient  
  
  
@pytest.mark.asyncio  
async def test_sign_new_user(test_async_client: AsyncClient) -> None:  
    payload = {  
        "email": "testuser@packt.com", "password": "testpassword", "username": "testuser@packt.com"  
    }  
    headers = {  
        "accept": "application/json", "Content-Type": "application/json"  
    }  
    test_response = {  
        "message": "User created successfully"  
    }  
    response = await test_async_client.post("/users/signup", json=payload, headers=headers)  
    assert response.status_code == 200  
    assert response.json() == test_response
```

Запустим тест:

```shell
pytest tests/test_login.py
```

Маршрут регистрации успешно протестирован:

![Снимок экрана 2024-09-22 в 23.47.02.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-22%20%D0%B2%2023.47.02.png)

##### <font color="#46aa63">Тестирование маршрута входа</font>

Ниже теста для маршрута регистрации давайте определим тест для маршрута входа. Мы начнем с определения полезной нагрузки запроса и заголовков, прежде чем инициировать запрос, как в первом тесте:

```python
@pytest.mark.asyncio  
async def test_sign_user_in(test_async_client: AsyncClient) -> None:  
    payload = {  
        "username": "testuser@packt.com",  
        "password": "testpassword"  
    }  
    headers = {  
        "accept": "application/json",  
        "Content-Type": "application/x-www-form-urlencoded"  
    }  
    response = await test_async_client.post("/users/signin", data=payload, headers=headers)  
    assert response.status_code == 200  
    assert response.json()["token_type"] == "Bearer"
```

Повторим тест:

```shell
pytest tests/test_login.py
```

![Снимок экрана 2024-09-22 в 23.54.18.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-22%20%D0%B2%2023.54.18.png)

Создадим тест для проверки входа пользователя которого нет в базе данных:

```python
@pytest.mark.asyncio  
async def test_sign_user_in_not_found(test_async_client: AsyncClient) -> None:  
    payload = {  
        "username": "notfound@packt.com",  
        "password": "testpassword"  
    }  
    headers = {  
        "accept": "application/json",  
        "Content-Type": "application/x-www-form-urlencoded"  
    }  
    response = await test_async_client.post("/users/signin", data=payload, headers=headers)  
    assert response.status_code == 404  
```

![Снимок экрана 2024-09-22 в 23.59.44.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-22%20%D0%B2%2023.59.44.png)

Как видим у нас ошибка. Давайте обработаем ее исправив наш маршрут в `routers\users.py`:

```python
@user_router.post("/signin", response_model=TokenResponse)  
async def sign_user_in(user: OAuth2PasswordRequestForm = Depends()) -> dict:  
    user_exist = await User.find_one(User.username == user.username)  
    if user_exist is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="User with supplied username does not exist"  
        )  
    if hash_password.verify_hash(user.password, user_exist.password):  
        access_token = create_access_token(user_exist.email)  
        return {  
            "access_token": access_token, "token_type": "Bearer"  
        }
```

Снова запустим тест:

![Снимок экрана 2024-09-23 в 00.03.29.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-23%20%D0%B2%2000.03.29.png)

Мы успешно написали тесты для маршрутов регистрации и входа. Перейдем к тестированию CRUD-маршрутов для API планировщика событий.

#### <font color="#46aa63">Тестирование конечных точек CRUD</font>

Давайте в `conftest.py` добавим 2 фикстуры:

```python
from auth.jwt_handler import create_access_token

@pytest.fixture(scope="module")  
async def access_token() -> str:  
    return create_access_token("testuser@packt.com")  
  
  
@pytest.fixture(scope="module")  
async def mock_event() -> Event:  
    new_event = Event(  
        creator="testuser@packt.com",  
        title="FastAPI",  
        description="Description!",  
        tags=["python", "fastapi"],  
        location="Google Meet"  
    )  
    await Event.insert_one(new_event)  
    yield new_event
```

В предыдущем блоке кода мы описали фикстуру `access_token`, которая при вызове возвращает токен доступа и фикстуру `mock_event`, который при вызове создает событие в базе данных.

Теперь давайте создадим файл `test_routes.py` в котором будем тестировать наши маршруты:

```shell
touch tests/test_routes.py
```

##### <font color="#46aa63">Тестирование конечных точек READ</font>

Давайте напишем тестовую функцию, которая проверяет GET-метод HTTP на маршруте `/events`:

```python
import pytest  
from httpx import AsyncClient  
  
from models.events import Event  
  
  
@pytest.mark.asyncio  
async def test_get_events(test_async_client: AsyncClient, mock_event: Event) -> None:  
    response = await test_async_client.get("/events/")  
    assert response.status_code == 200  
    assert response.json()[0]["_id"] == str(mock_event.id)
```

В предыдущем блоке кода мы тестируем путь маршрута события, чтобы проверить, присутствует ли событие, добавленное в базу данных в фикстуре `mock_event`. Давайте запустим тест:

```shell
pytest tests/test_routes.py

=============== test session starts ==========
collecting ... collected 1 item

test_routes.py::test_get_events PASSED   [100%]

=============== 1 passed in 0.02s =============
```

Далее давайте напишем тестовую функцию для конечной точки `/events/{event_id}`:

```python
@pytest.mark.asyncio  
async def test_get_event(test_async_client: AsyncClient, mock_event: Event) -> None:  
    url = f"/events/{str(mock_event.id)}"  
    response = await test_async_client.get(url)  
    assert response.status_code == 200  
    assert response.json()["creator"] == mock_event.creator  
    assert response.json()["_id"] == str(mock_event.id)
```

В предыдущем блоке кода мы тестируем конечную точку, которая извлекает одно событие. Переданный идентификатор события извлекается из фикстуры `mock_event`, а результат запроса сравнивается с данными, хранящимися в фикстуре `mock_event`. Давайте запустим тест:

```shell
pytest tests/test_routes.py::test_get_event

=============== test session starts ===========
collecting ... collected 1 item

test_routes.py::test_get_event PASSED    [100%]

=============== 1 passed in 0.03s ==============
```

##### <font color="#46aa63">Тестирование конечных точек CREATE</font>

Мы начнем с определения функции и получения токена доступа из ранее созданных `fixture`. Мы создадим полезную нагрузку запроса, которая будет отправлена на сервер, заголовки запроса, которые будут содержать тип контента, а также значение заголовка авторизации. Также будет определен тестовый ответ, после чего инициируется запрос и сравниваются ответы. Добавим следующий код:

```python
@pytest.mark.asyncio  
async def test_post_event(test_async_client: AsyncClient, access_token: str) -> None:  
    payload = {  
        "title": "FastAPI Book Launch",  
        "description": "Description",  
        "tags": ["python", "fastapi"],  
        "location": "Google Meet"  
    }  
  
    headers = {  
        "Content-Type": "application/json", "Authorization": f"Bearer {access_token}"  
    }  
  
    success_response = {  
        "message": "Event created successfully"  
    }  
  
    response = await test_async_client.post("/events/", json=payload, headers=headers)  
  
    assert response.status_code == 200  
    assert response.json() == success_response
```

Запустим тест:

```shell
pytest tests/test_routes.py::test_post_event

============= test session starts ==============
collecting ... collected 1 item

test_routes.py::test_post_event PASSED      [100%]

============== 1 passed in 0.04s ================
```

Давайте напишем тест для проверки количества событий, хранящихся в базе данных (в нашем случае 2). Добавим следующее:

```python

@pytest.mark.asyncio  
async def test_get_events_count(test_async_client: AsyncClient) -> None:  
    response = await test_async_client.get("/events/")  
    events = response.json()  
    assert response.status_code == 200  
    assert len(events) == 2

```


Запустим тесты:

```shell
pytest tests/test_routes.py 

============= test session starts ========================

platform darwin -- Python 3.11.3, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/arteme/PycharmProjects/planner
configfile: pytest.ini
plugins: asyncio-0.24.0, anyio-4.4.0
asyncio: mode=Mode.AUTO, default_loop_scope=None
collected 4 items                                                       
tests/test_routes.py ....                             [100%]

============== 4 passed in 0.05s =========================
```

Мы успешно протестировали конечные точки **GET** `/events` и `/events/{event_id}` и конечную точку **POST** `/events`, соответственно. Давайте проверим конечные точки **UPDATE** и **DELETE**.

##### <font color="#46aa63">Тестирование конечных точек UPDATE</font>

Напишем следующий тест в `test_routes.py`:

```python
@pytest.mark.asyncio  
async def test_update_event(test_async_client: AsyncClient, mock_event: Event, access_token: str) -> None:  
    test_payload = {  
        "title": "Updated FastAPI event"  
    }  
    headers = {  
        "Content-Type": "application/json",  
        "Authorization": f"Bearer {access_token}"  
    }  
    url = f"/events/{str(mock_event.id)}"  
    response = await test_async_client.put(url, json=test_payload, headers=headers)  
    assert response.status_code == 200  
    assert response.json()["title"] == test_payload["title"]
```

В предыдущем блоке кода мы изменяем событие, хранящееся в базе данных, извлекая `ID` из фикстуры `mock_event`. Затем мы определяем полезную нагрузку запроса и заголовки. В переменной response `инициируется` запрос и сравнивается полученный ответ. Давайте подтвердим, что тест работает правильно:

```shell
pytest tests/test_routes.py::test_update_event

=========== test session starts ==========
collecting ... collected 1 item

test_routes.py::test_update_event PASSED   [100%]

=========== 1 passed in 0.10s ===========
```

Давайте изменим ожидаемый ответ, чтобы подтвердить достоверность нашего теста:

```python
assert response.json()["title"] == "This test should fail"
```

Результат:

```shell
pytest tests/test_routes.py::test_update_event

============= 1 failed in 0.19s ===============
FAILED                                 [100%]
tests/test_routes.py:53 (test_update_event)
'Updated FastAPI event' != 'This test should fail'

Expected :'This test should fail'
Actual   :'Updated FastAPI event'

E       AssertionError: assert 'Updated FastAPI event' == 'This test should fail'
```

##### <font color="#46aa63">Тестирование конечной точки DELETE</font>

Напишем тестовую функцию для конечной точки `DELETE`:

```python
@pytest.mark.asyncio  
async def test_delete_event(test_async_client: AsyncClient, mock_event: Event, access_token: str) -> None:  
    success_response = {  
        "message": "Event deleted successfully."  
    }  
    headers = {  
        "Content-Type": "application/json",  
        "Authorization": f"Bearer {access_token}"  
    }  
    url = f"/events/{mock_event.id}"  
    response = await test_async_client.delete(url, headers=headers)  
    assert response.status_code == 200  
    assert response.json() == success_response
```

Как и в предыдущих тестах, определяется ожидаемый ответ теста, а также заголовки. Маршрут `DELETE` задействован, и ответ сравнивается. Давайте запустим тест:

```shell
pytest tests/test_routes.py::test_delete_event

============= test session starts ===============
collecting ... collected 1 item

test_routes.py::test_delete_event PASSED   [100%]

============= 1 passed in 0.09s =================
```

Чтобы убедиться, что документ действительно был удален, добавим финальную проверку:

```python
@pytest.mark.asyncio  
async def test_get_event_again(default_client: AsyncClient, mock_event: Event) -> None:  
    url = f"/events/{str(mock_event.id)}"  
    response = await default_client.get(url)  
    assert response.status_code == 404
```

Ожидаемый ответ - код 404. Давайте попробуем:

```shell
pytest tests/test_routes.py

=================== test session starts ===================
platform darwin -- Python 3.11.3, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/arteme/PycharmProjects/planner
configfile: pytest.ini
plugins: asyncio-0.24.0, anyio-4.4.0
asyncio: mode=Mode.AUTO, default_loop_scope=None
collected 7 items                                                                                                                                             
tests/test_routes.py .......                            [100%]

================== 7 passed in 0.07s ======================
```

Давайте запустим все тесты, присутствующие в нашем приложении:

```shell
pytest

=================== test session starts =========================
platform darwin -- Python 3.11.3, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/arteme/PycharmProjects/planner
configfile: pytest.ini
plugins: asyncio-0.24.0, anyio-4.4.0
asyncio: mode=Mode.AUTO, default_loop_scope=None
collected 10 items                                                      
tests/test_login.py ...                                     [ 30%]
tests/test_routes.py .......                                [100%]

=================== 10 passed in 0.71s ==========================
```

Теперь, когда мы успешно протестировали конечные точки, содержащиеся в API планировщика событий, давайте запустим тест покрытия, чтобы определить процент нашего кода, задействованного в тестовой операции.


### <font color="#46aa63">Тестовое покрытие</font>

Отчет о покрытии тестами полезен для определения процента нашего кода, который был выполнен в ходе тестирования. Давайте установим модуль `coverage`, чтобы мы могли измерить, был ли наш API адекватно протестирован:

```shell
pip install coverage
```

Далее давайте создадим отчет о покрытии, выполнив эту команду:

```shell
coverage run -m pytest
```

Вот результат:

```shell
======================= test session starts =======================
platform darwin -- Python 3.11.3, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/arteme/PycharmProjects/planner
configfile: pytest.ini
plugins: asyncio-0.24.0, anyio-4.4.0
asyncio: mode=Mode.AUTO, default_loop_scope=None
collected 10 items                                                      
tests/test_login.py ...                                      [ 30%]
tests/test_routes.py .......                                  [100%]
======================= 10 passed in 0.71s ========================
```

Далее давайте просмотрим отчет, сгенерированный командой `coverage run - m pytest`. Мы можем выбрать просмотр отчета на терминале или на веб-странице, создав отчет в формате HTML. Мы сделаем оба.

```shell
coverage report

Name                     Stmts   Miss  Cover
--------------------------------------------
auth/__init__.py             0      0   100%
auth/authenticate.py         9      1    89%
auth/hash_password.py        9      0   100%
auth/jwt_handler.py         20      4    80%
config/settings.py           8      0   100%
database/__init__.py         0      0   100%
database/connection.py      13      4    69%
database/objects.py         32      2    94%
main.py                     13      1    92%
models/__init__.py           0      0   100%
models/events.py            17      0   100%
models/users.py             14      0   100%
routers/__init__.py          0      0   100%
routers/events.py           40      4    90%
routers/users.py            41      9    78%
tests/__init__.py            0      0   100%
tests/conftest.py           24      0   100%
tests/test_login.py         23      0   100%
tests/test_routes.py        50      0   100%
--------------------------------------------
TOTAL                      313     25    92%
```

Из предыдущего отчета проценты означают количество кода, выполненного и с которым взаимодействовали. Давайте создадим HTML-отчет, чтобы мы могли проверить блоки кода, с которыми мы взаимодействовали.

```shell
coverage html

Wrote HTML report to htmlcov/index.html
```

Откроем `htmlcov/index.html` в своем браузере:

![Снимок экрана 2024-09-23 в 22.13.47.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-23%20%D0%B2%2022.13.47.png)

Давайте проверим отчет о покрытии для `routes/events.py`. Нажмем на него, чтобы отобразить его:

![Снимок экрана 2024-09-23 в 22.14.02.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-23%20%D0%B2%2022.14.02.png)


### <font color="#46aa63">Подготовка к развертыванию</font>

Перед развертыванием наших приложений мы должны убедиться, что установлены правильные параметры, необходимые для плавного развертывания. Эти параметры включают обеспечение актуальности зависимостей приложения в файле `requirements.txt`, настройку переменных среды и т. д.


##### <font color="#46aa63">Управление зависимостями</font>

На прошлый лекциях мы установили такие пакеты, как `beanie`,`pytest` и другие. Эти пакеты отсутствуют в файле `requirements.txt` ,который служит менеджером зависимостей для нашего приложения. Важно, чтобы файл `requirements.txt` обновлялся.

В Python список пакетов, используемых в среде разработки, можно получить с помощью команды `pip freeze`. Команда `pip freeze` возвращает список всех пакетов, установленных напрямую, и зависимостей для каждого установленного пакета. К счастью, файл `requirements.txt` можно поддерживать вручную, что позволяет нам перечислять только основные пакеты, тем самым упрощая управление зависимостями.

Давайте перечислим зависимости, используемые приложением, прежде чем перезаписывать файл `requirements.txt:`

```shell
pip freeze

alembic==1.13.2  
annotated-types==0.7.0  
anyio==4.4.0  
bcrypt==4.2.0  
beanie==1.26.0  
certifi==2024.8.30  
cffi==1.17.1  
click==8.1.7  
coverage==7.6.1
...
```

Команда возвращает список всех зависимостей, некоторые из которых мы не используем напрямую в приложении. Давайте вручную заполним файл requirements.txt пакетами, которые мы будем использовать:

```txt
fastapi==0.115.0  
uvicorn==0.30.6  
pydantic-settings==2.5.2  
email_validator==2.2.0  
beanie==1.26.0  
passlib==1.7.4  
bcrypt==4.2.0  
python-jose==3.3.0  
python-multipart==0.0.10  
pytest==8.3.3  
mongomock-motor==0.0.34  
httpx==0.27.2  
pytest-asyncio==0.24.0
```

Мы заполнили файл requirements.txt зависимостями, используемыми непосредственно в нашем приложении.

### <font color="#46aa63">Развертывание с помощью Docker</font>

Теперь, когда мы выполнили необходимые шаги по подготовке к развертыванию, давайте перейдем к локальному развертыванию нашего приложения с помощью `Docker`.

`Docker` можно использовать как для локальной разработки, так и для развертывания приложений в рабочей среде. На занятии мы рассмотрим только локальное развертывание, а также будут включены ссылки на официальные руководства по развертыванию в облачных службах.

Для управления приложениями с несколькими контейнерами, такими как контейнер приложения и контейнер базы данных, используется инструмент компоновки. `Compose` — это инструмент, используемый для управления многоконтейнерными приложениями `Docker`, определенными в файле конфигурации, обычно `docker-compose.yaml`. Инструмент компоновки, `docker-compose`, устанавливается вместе с движком `Docker`.

##### <font color="#46aa63">Написание Dockerfile</font>

`Dockerfile` содержит набор инструкций, используемых для создания образа Docker. Созданный образ Docker затем можно распространять в реестры (частные и общедоступные), развертывать на облачных серверах, таких как AWS и Google Cloud, и использовать в разных операционных системах путем создания контейнера.

Давайте создадим `Dockerfile` для сборки образа приложения. В каталоге проекта создадим файл `Dockerfile`:

```txt
FROM python:3.11  
  
WORKDIR /planner  
COPY requirements.txt /planner  
RUN pip install --upgrade pip && pip install -r /planner/requirements.txt
  
EXPOSE 5001
COPY ./ /planner  
  
CMD ["python", "main.py"]
```


Давайте пройдемся по инструкциям, содержащимся в предыдущем файле `Dockerfile`, одну за другой:

- Первая инструкция, которую выполняет `Dockerfile`, — установить базовый образ для нашего собственного образа с помощью ключевого слова **FROM**. Другие варианты этого образа можно найти по адресу https://hub.docker.com/_/python
- В следующей строке ключевое слово **WORKDIR** используется для задания рабочего каталога `/planner`. Рабочий каталог помогает организовать структуру проекта, построенного на образе
- Затем мы копируем файл `requirements.txt `из локального каталога в рабочий каталог контейнера Docker, используя ключевое слово **COPY**
- Следующая инструкция - это команда **RUN**, которая используется для обновления пакета `pip` и последующей установки зависимостей из файла `requirements.txt`
- Следующая команда **EXPOSE** предоставляет порт, через который к нашему приложению можно получить доступ из локальной сети
- Следующая команда **COPY** копирует остальные файлы и папки в рабочий каталог контейнера `Docker`
- Наконец, последняя команда запускает приложение с помощью команды **CMD**

 Каждый набор инструкций, перечисленных в `Dockerfile`, создается как отдельный уровень. `Docker` выполняет интеллектуальную работу по кэшированию каждого слоя во время сборки, чтобы сократить время сборки и исключить повторение. Если слой, который по сути является инструкцией, остается нетронутым, этот слой пропускается и используется ранее созданный. То есть Docker использует систему кэширования при сборке образов.
 
Давайте создадим файл `.dockerignore`, прежде чем приступить к сборке образа:

```shell
touch .dockerignore
```

Запишем в нем следующее:

```txt
venv
.env
.git
```

Файл `.dockerignore` содержит файлы и папки, которые должны быть исключены из инструкций, определенных в файле `Dockerfile`.

##### <font color="#46aa63">Создание Docker образа</font>

Чтобы создать образ приложения, выполните следующую команду в базовом каталоге:

```shell
docker build -t event-planner-api .
```

Эта команда просто указывает `Docker` создать образ с тегом `event-planner-api` из инструкций, определенных в текущем каталоге, который представлен точкой в конце команды. Процесс сборки начинается после запуска команды и выполнения инструкций:

![Снимок экрана 2024-09-25 в 21.36.56.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-25%20%D0%B2%2021.36.56.png)

Теперь, когда мы успешно создали образ нашего приложения, давайте загрузим образ MongoDB:

```shell
docker pull mongo
```

Мы извлекаем образ `MongoDB`, чтобы создать автономный контейнер базы данных, доступный из контейнера API при создании. По умолчанию контейнер `Docker` имеет отдельную сетевую конфигурацию, и подключение к локальному адресу хостов машины запрещено.

![Снимок экрана 2024-09-25 в 21.41.09.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-25%20%D0%B2%2021.41.09.png)

Команда **docker pull** отвечает за загрузку образов из реестра. Если не указано иное, эти образы загружаются из общедоступного реестра `Docker Hub`.

##### <font color="#46aa63">Развертывание приложения локально</font>

Теперь, когда мы создали образы для `API` и загрузили образ для базы данных `MongoDB`, давайте приступим к написанию манифеста для обработки развертывания нашего приложения. Манифест `docker-compose` будет состоять из службы API и службы базы данных `MongoDB`. В корневом каталоге создадим файл манифеста:

```shell
touch docker-compose.yml
```

Содержимое файла манифеста `docker-compose` будет следующим:

```txt
version: "3.7"  
  
services:  
  api:  
    build: .  
    image: event-planner-api:latest  
    ports:  
      - "5001:5000"  
    env_file: .env.prod  
  
  
  database:  
    image: mongo  
    volumes:  
      - data:/data/db  
  
  
volumes:  
  data:
  
```

В файле описан следующий набор инструкций:

- **version**: - это версия API Docker Compose
- **services** - основной раздел, где создаются и описываются сервисы (контейнеры docker).  В нем описывается сервисы:
	-  **api** -  название для нашего сервиса, на основе которого будет создан контейнер. В этом сервисе применяется следующий набор инструкций:
		- **build** -указывает `Docker` на создание образа `event-planner-api:latest` для службы api из файла Dockerfile, расположенного в текущем каталоге, обозначенном «.»
		- **image** - имя образа, который будет использоваться для создания контейнера
		- **ports** - порт 5001 открыт из контейнера, чтобы мы могли получить доступ к службе через `HTTP`.
		- **env_file** - файл среды имеет значение `.env.prod`. 
	- **database**  - название для базы данных. В этом сервисе применяется следующий набор инструкций:
		- **image** - использовать образ `mongo`, который мы получили ранее
		- **volumes** - к службе подключен постоянный том для хранения наших данных. Для этого выделена папка `/data/db`. Том для этого развертывания создается с именем `data`

- Теперь, когда мы поняли содержимое манифеста компоновки, давайте создадим файл среды, `.env.prod`:  

```txt
MONGO_URL="mongodb://database:27017/planner"  
SECRET_KEY="us1v#3o1oj#%85cl8a1^*1!($-(=w*y^h^$$q9emen"
```

##### <font color="#46aa63">Запускаем приложение</font>

Мы настроены на развертывание и запуск приложения из манифеста `docker- compose`. Давайте запустим сервисы с помощью инструмента компоновки:

```shell
docker-compose up -d

Starting planner_api_1    ... done
Starting planner_database_1 ... done
```

Службы приложений созданы и развернуты. Давайте проверим, проверив список запущенных контейнеров:

```shell
docker ps -a

CONTAINER ID   IMAGE                                     COMMAND                  CREATED              STATUS                       PORTS                                                                                                         NAMES
c3d97e2388c9   event-planner-api:latest                  "python main.py"         41 seconds ago       Up 40 seconds                0.0.0.0:5001->5000/tcp                                                                                        planner_api_1
944e41e1deb0   mongo                                     "docker-entrypoint.s…"   About a minute ago   Up About a minute            0.0.0.0:50163->27017/tcp 
```

Команда возвращает список контейнеров, работающих вместе с портами, через которые к ним можно получить доступ. Давайте проверим рабочее состояние, отправив запрос `GET` развернутому приложению:

```shell
curl -X 'GET' 'http://127.0.0.1:5001/events/' -H 'accept: application/json'

[]
```

Замечательно! Развернутое приложение работает корректно. Давайте проверим, что база данных также работает, создав пользователя:

```shell
curl -X 'POST' 'http://localhost:5001/users/signup' -H 'accept:application/json' -H 'Content-Type: application/json' -d '{  
"email": "fastapi@packt.com", "password": "strong!!!", "username": "fastapi@packt.com"}'
```

Мы также получаем положительный ответ:

```json
{"message":"User created successfully"}
```

Чтобы остановить сервер развертывания после проверки, из корневого каталога запускается следующая команда:

```shell
docker-compose down

Stopping planner_api_1      ... done
Stopping planner_database_1 ... done
Removing planner_api_1      ... done
Removing planner_database_1 ... done
Removing network planner_default

```


### Резюме

На этом занятии мы успешно протестировали API, написав тесты для маршрутов аутентификации и маршрута CRUD. Мы узнали, что такое тестирование и как писать тесты pytest, библиотекой быстрого тестирования, созданной для приложений Python. Мы также узнали, что такое фикстуры pytest, и использовали их для создания многократно используемых токенов доступа и объектов базы данных, а также для сохранения экземпляра приложения на протяжении всего сеанса тестирования. Вы смогли утвердить ответы на ваши HTTP-запросы API и проверить поведение вашего API. Наконец, вы научились генерировать отчет о покрытии для своих тестов и различать блоки кода, выполняемые во время сеанса тестирования.

Также узнали, как подготовить приложение к развертыванию. Мы начали с обновления зависимостей, используемых для приложения, и сохранения их в  
файл `requirements.txt`, прежде чем переходить к управлению переменными среды, используемыми для API.

Мы также рассмотрели шаги, связанные с развертыванием приложения в рабочей среде: создание образа `Docker` из `Dockerfile`, настройка манифеста компоновки для API и служб базы данных, а затем развертывание приложения. Узнали новые команды для проверки списка запущенных контейнеров, а также для запуска и остановки контейнеров Docker. Наконец, мы протестировали приложение, чтобы убедиться, что развертывание прошло успешно.


