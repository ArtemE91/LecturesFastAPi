# MongoDB, ODM Beanie

### План занятия

1) MongoDB
2) ODM Beanie

### <font color="#46aa63">Mongo DB</font>

##### <font color="#46aa63">NoSQL</font>

`NoSQL` (Not Only SQL) баз данных — это класс систем управления базами данных, которые предоставляют альтернативные модели хранения и обработки данных по сравнению с традиционными реляционными СУБД. 

`NoSQL` баз данных предназначены для работы с неструктурированными и полуструктурированными данными. Они позволяют хранить данные в формах, которые не требуют строгой схемы, как это делается в реляционных СУБД.

Существует несколько основных типов `NoSQL` баз данных, каждый из которых подходит для определенных сценариев:

- **Документо-ориентированные базы данных** (например, `MongoDB`, `CouchDB`): Хранят данные в виде документов, обычно в формате JSON или BSON. Эти базы данных идеально подходят для хранения неструктурированных данных.
    
- **Колонно-ориентированные базы данных** (например, `Apache Cassandra`, `ClickHouse`): Оптимизированы для работы с большими объемами данных и быстро обрабатывают записи по столбцам, а не по строкам.
    
- **Графовые базы данных** (например, `Neo4j`, `ArangoDB`): Используются для хранения и обработки графов данных, что делает их идеальными для приложений, связанных с социальными сетями и анализом взаимосвязей.
    
- **Ключ-значение базы данных** (например, `Redis`, `Amazon DynamoDB`): Хранят данные в виде пар "ключ-значение", что обеспечивает высокую скорость доступа к данным.


Преимущества NoSQL баз данных:

- **Гибкость схемы**: не требуется заранее определять схему данных, что упрощает адаптацию к изменяющимся требованиям.
- **Масштабируемость**: многие NoSQL базы данных позволяют горизонтальное масштабирование, что делает их более эффективными для обработки больших объемов данных.
- **Высокая производительность**: оптимизированы для определенных типов операций, таких как запись и чтение больших объемов данных.
- **Устойчивость к сбоям**: Многие NoSQL базы данных предлагают встроенные механизмы репликации и резервного копирования.

Недостатки NoSQL баз данных:

- **Отсутствие стандартов**: не существует единых стандартов для NoSQL баз данных, что может затруднить выбор решения.
- **Сложности с консистентностью**: многие NoSQL базы данных используют модели "в конечном итоге согласованности", что может быть проблемой для критически важных приложений.
- **Ограниченная поддержка транзакций**: в отличие от реляционных баз данных, поддержка ACID-транзакций может быть ограничена.

Сценарии использования:

NoSQL базы данных хорошо подходят для:

- Хранения больших объемов данных с высокой скоростью записи (например, логирование).
- Работа с данными, которые быстро меняются или не имеют фиксированной структуры (например, данные из социальных сетей).
- Обработки графовых данных и анализа взаимосвязей.

NoSQL базы данных представляют собой мощный инструмент для решения современных задач хранения и обработки данных. Их использование зависит от конкретных требований и особенностей приложения.

##### <font color="#46aa63">Что такое MongoDB</font>

**MongoDB** — это популярная NoSQL база данных, ориентированная на документо-ориентированное хранение данных. Она разработана для обработки больших объемов неструктурированных и полуструктурированных данных, обеспечивая высокую производительность, гибкость и масштабируемость. Вот ключевые аспекты MongoDB:

**Документо-ориентированная структура**

- **Документы**: Данные хранятся в виде документов, обычно в формате BSON (Binary JSON). Каждый документ представляет собой набор полей и значений, что позволяет легко моделировать сложные структуры данных.
- **Коллекции**: Документы группируются в коллекции, которые аналогичны таблицам в реляционных базах данных, но не требуют строгой схемы.

**Гибкость схемы**
   
   MongoDB позволяет хранить данные без заранее определенной схемы, что упрощает адаптацию к изменяющимся требованиям. Вы можете добавлять новые поля в документы без необходимости изменять всю структуру базы данных.

**Высокая производительность**
 
 MongoDB обеспечивает высокую скорость операций записи и чтения благодаря использованию индексов и механизма кэширования. Это делает его подходящим для приложений с высокой нагрузкой.

**Масштабируемость**

MongoDB поддерживает горизонтальное масштабирование (sharding), что позволяет распределять данные по нескольким серверам и легко расширять систему по мере роста нагрузки.

**Агрегация данных**

MongoDB предоставляет мощный фреймворк для агрегации данных, который позволяет выполнять сложные запросы, фильтрацию и преобразование данных с помощью этапов, таких как `$match`, `$group`, `$sort`, и `$project`.

**Репликация и доступность**

MongoDB поддерживает репликацию, что позволяет создавать резервные копии данных на нескольких серверах и обеспечивает высокую доступность. Если один из узлов выходит из строя, другой может взять на себя его задачи.

**Безопасность**

MongoDB предлагает различные механизмы аутентификации и авторизации, а также поддержку шифрования данных.

**Сценарии использования**

- Приложений с неструктурированными данными (например, социальные сети, интернет-магазины).
- Систем реального времени, требующих быстрой обработки данных.
- Проектов, где структура данных может изменяться со временем (например, стартапы и MVP).

MongoDB является мощным инструментом для работы с данными в современных приложениях, предлагая гибкость и масштабируемость, которые могут быть критически важны для успеха бизнеса. Его простота в использовании и возможности делают его популярным выбором среди разработчиков.

##### <font color="#46aa63">Основные компоненты MongoDB</font>

**Серверы**
    
- **MongoDB Server**: это основное приложение, которое управляет базой данных. Оно обрабатывает запросы от клиентов, управляет данными и обеспечивает безопасность и доступность.
- **Репликация**: серверы могут быть организованы в репликационные наборы, где один сервер является основным (primary), а остальные — вторичными (secondary). Вторичные серверы могут использоваться для резервирования и повышения доступности.

 **Клиенты**
    
- **MongoDB Shell**: инструмент командной строки для взаимодействия с сервером MongoDB. Позволяет выполнять запросы, управлять базами данных и коллекциями.
- **Клиентские библиотеки**: существуют библиотеки для различных языков программирования (например, Python, Java, Node.js), которые позволяют разработчикам взаимодействовать с MongoDB из приложений.

**Базы данных**
    
- В MongoDB можно создавать несколько баз данных. Каждая база данных содержит свои коллекции. Название базы данных должно быть уникальным в пределах одного сервера.
- База данных может содержать разные коллекции, и каждая из них может иметь свою уникальную структуру данных.

**Коллекции**
    
- Коллекция — это группа документов, аналогичная таблице в реляционных базах данных. В отличие от таблиц, коллекции не требуют строгой схемы, и документы в одной коллекции могут иметь различные поля и структуры.
- Коллекции могут содержать неограниченное количество документов.

**Документы**
    
- Документ — это основной элемент данных в MongoDB. Он представлен в формате BSON (Binary JSON), который позволяет хранить данные в виде пар "ключ-значение".
- Каждый документ имеет уникальный идентификатор `_id`, который автоматически создается, если не указан вручную.

**Структура документа: BSON (Binary JSON)**

- **BSON**: Это бинарный формат, используемый для хранения документов в MongoDB. Он поддерживает более широкий набор типов данных, чем стандартный JSON, включая:
    - **Целые числа**
    - **Даты**
    - **Массивы**
    - **Объекты** (вложенные документы)
    - **Двоичные данные**
- **Преимущества BSON**:
    - Более эффективное хранение и обработка данных по сравнению с текстовым JSON.
    - Позволяет быстро сериализовать и десериализовать данные, что увеличивает производительность.

**Схема данных: динамическая и гибкая**

- **Динамическая схема**: MongoDB позволяет хранить данные без заранее определенной схемы. Это означает, что вы можете добавлять новые поля в документы в любой момент, не беспокоясь о том, что это повлияет на другие документы в той же коллекции.
    
- **Гибкость**: Гибкость схемы делает MongoDB идеальным выбором для приложений, где структура данных может изменяться со временем, например, в стартапах и проектах с изменяющимися требованиями.
    
##### <font color="#46aa63">Создание и удаление баз данных и коллекций</font>

**Создание базы данных**

Для создания базы данных в MongoDB, вы можете использовать команду `use`. База данных создается автоматически при первой записи в нее.

```javascript
use myDatabase
```

**Удаление базы данных**

Для удаления базы данных можно использовать команду `dropDatabase()`.

```javascript
db.dropDatabase()
```

**Создание и удаление коллекций**

Чтобы создать коллекцию, можно использовать команду `createCollection()` или просто вставить документ в новую коллекцию, и она будет создана автоматически.

```javascript
db.createCollection("myCollection")
```

Для удаления коллекции используется команда `drop()`.

```javascript
db.myCollection.drop()
```

**Вставка документов**

Для вставки одного документа в коллекцию используется метод `insertOne()`.

```javascript
db.myCollection.insertOne({ name: "John", age: 30 })
```

Для вставки нескольких документов можно использовать метод `insertMany()`.

```javascript
db.myCollection.insertMany([     { name: "Alice", age: 25 },     { name: "Bob", age: 22 } ])
```

**Чтение документов**

Для чтения документов из коллекции используется метод `find()`. Он возвращает курсор, который можно перебрать.

```javascript
db.myCollection.find()
```

Для получения одного документа можно использовать метод `findOne()`.

```javascript
db.myCollection.findOne({ name: "John" })
```

**Обновление документов**

Для обновления одного документа используется метод `updateOne()`. Он принимает фильтр и обновление.

```javascript
db.myCollection.updateOne({ name: "John" }, {$set: { age: 31}})
```

Чтобы обновить несколько документов, используется метод `updateMany()`.

```javascript
db.myCollection.updateMany({age: {$lt: 30}}, {$set: {status: "young" } })
```


**Удаление документов**

Для удаления одного документа используется метод `deleteOne()`.

```javascript
db.myCollection.deleteOne({ name: "John" })
```

Для удаления нескольких документов используется метод `deleteMany()`.

```javascript
db.myCollection.deleteMany({ age: { $lt: 25 } })
```

##### <font color="#46aa63">Структура запросов в MongoDB</font>

MongoDB предоставляет мощный и гибкий синтаксис для выполнения запросов, включая использование фильтров, операторов и индексов для оптимизации производительности.

**Использование фильтров и операторов**

**Фильтры** позволяют ограничить набор данных, которые вы хотите получить или изменить. Вы можете использовать различные операторы для более точного поиска.

**Операторы сравнения**

MongoDB поддерживает несколько операторов сравнения, которые можно использовать в фильтрах:

- **$eq**: равен
- **$ne**: не равен
- **$gt**: больше
- **$gte**: больше или равно
- **$lt**: меньше
- **$lte**: меньше или равно

**Пример**: Найти документы, где `age` больше 25.

```javascript
db.myCollection.find({ age: { $gt: 25 } })
```

**Логические операторы**

MongoDB также поддерживает логические операторы для создания сложных условий:

- **$and**: логическое И
- **$or**: логическое ИЛИ
- **$not**: логическое НЕ
- **$nor**: логическое ИЛИ с отрицанием

**Пример**: Найти документы, где `age` меньше 30 и `status` равен "young".

```javascript
db.myCollection.find({
    $and: [
        { age: { $lt: 30 } },
        { status: "young" }
    ]
})
```

**Комбинирование операторов**

Вы можете комбинировать операторы для более сложных запросов.

**Пример**: Найти документы, где `age` больше 25 или `status` равен "young".

```javascript
db.myCollection.find({
    $or: [
        { age: { $gt: 25 } },
        { status: "young" }
    ]
})
```

**Создание и использование индексов для оптимизации производительности**

Индексы в MongoDB позволяют ускорить выполнение запросов, особенно в больших коллекциях.

Чтобы создать индекс, используйте метод `createIndex()`. Вы можете создать индекс по одному или нескольким полям.

**Пример**: Создать индекс по полю `age`.

```javascript
db.myCollection.createIndex({ age: 1 })  // 1 для сортировки по возрастанию, -1 для убывания
```

**Использование составных индексов**

Если ваши запросы часто используют несколько полей, рассмотрите возможность создания составного индекса.

**Пример**: Создать составной индекс по полям `age` и `status`.

```javascript
db.myCollection.createIndex({ age: 1, status: 1 })
```

**Проверка индексов**

Чтобы посмотреть все индексы в коллекции, используйте метод `getIndexes()`.

```javascript
db.myCollection.getIndexes()
```

**Удаление индекса**

Если индекс больше не нужен, его можно удалить с помощью метода `dropIndex()`.

```javascript
db.myCollection.dropIndex("index_name")
```

##### <font color="#46aa63">Понятие агрегации в MongoDB</font>

Агрегация в MongoDB — это мощный инструмент для обработки и анализа данных, позволяющий выполнять сложные операции на наборах документов. Агрегация предоставляет возможность выполнять вычисления, группировку и трансформацию данных, что особенно полезно для анализа и отчетности.

**Основные стадии агрегации**

Агрегация в MongoDB обычно осуществляется с помощью метода `aggregate()`, который принимает массив стадий обработки. Вот основные стадии агрегации:

**$match**
Стадия `$match` используется для фильтрации документов на основе заданных условий. Она аналогична команде `find()`.

**Пример**:
Фильтрация документов, где `age` больше 30.

```javascript
db.myCollection.aggregate([
    { $match: { age: { $gt: 30 } } }
])
```

**$group**
Стадия `$group` используется для группировки документов по одному или нескольким полям и выполнения агрегатных операций, таких как подсчет, сумма и среднее.

**Пример**:
Группировка документов по `status` и подсчет количества пользователей в каждой группе.

```javascript
db.myCollection.aggregate([
    { $group: { _id: "$status", count: { $sum: 1 } } }
])
```

**$sort**
Стадия `$sort` используется для сортировки документов по заданным полям. Вы можете сортировать как по возрастанию, так и по убыванию.

**Пример**:
Сортировка по `age` в порядке возрастания.

```javascript
db.myCollection.aggregate([
    { $sort: { age: 1 } }
])
```

**$project**
Стадия `$project` используется для изменения структуры документов, определяя, какие поля включать или исключать из результата. Вы также можете создавать новые поля на основе существующих.

**Пример**:
Выбор только полей `name` и `age`, а также добавление нового поля `isAdult`.

```javascript
db.myCollection.aggregate([
    { $project: { name: 1, age: 1, isAdult: { $gt: ["$age", 18] } } }
])
```

### <font color="#46aa63">ODM Beanie</font>

#### <font color="#46aa63">Настройка MongoDB</font>

Существует ряд библиотек, которые позволяют нам интегрировать `MongoDB` в наше приложение `FastAPI`. Однако мы будем использовать **Beanie**, асинхронную библиотеку `Object Document Mapper (ODM)`, для выполнения операций с базой данных из нашего приложения.

Установим библиотеку `beanie`, выполнив следующую команду:

```shell
pip install beanie
```

Прежде чем погрузиться в интеграцию, давайте рассмотрим некоторые методы из библиотеки `Beanie`, а также то, как создаются таблицы базы данных.

##### <font color="#46aa63">Документ</font>

В `SQL` данные, хранящиеся в строках и столбцах, содержатся в таблице. В базе данных `NoSQL` это называется документом. Документ представляет, как данные будут храниться в коллекции базы данных. Документы определяются так же, как и модель `Pydantic`, за исключением того, что вместо этого наследуется класс `Document` из библиотеки `Beanie`.

Пример документа определяется следующим образом:

```python
from beanie import Document

class Event(Document): 
	name: str
    location: str

	class Settings: 
		name = "events"
```

Подкласс `Settings` определен, чтобы указать библиотеке создать имя коллекции, переданное в базе данных MongoDB.

Теперь, когда мы знаем, как создать документ, давайте рассмотрим методы, используемые для выполнения `CRUD` операций:

- `.insert()` и `.create()`: вызываются экземпляром документа для создания новой записи в базе данных. Вы также можете использовать метод `.insert_one()` для добавления отдельной записи в базу данных. Чтобы вставить много записей в базу данных, вызывается метод `.insert_many()`, который принимает список экземпляров документа, например:
  
```python
event = Event(name="Packt office launch", location="Hybrid")
await event.create()  
await Event.insert_one(event)
```

- .`find()`и `.get()`: метод `.find()` используется для поиска списка документов, соответствующих критериям поиска, переданным в качестве аргумента метода. Метод `.get()` используется для получения одного документа, соответствующего предоставленному идентификатору. Отдельный документ, соответствующий критерию поиска, можно найти с помощью метода `.find_one()`, например следующего:
  
```python
event = await Event.get("74478287284ff")

event = await Event.find(Event.location == "Hybrid").to_ list() # Returns a list of matching items

event = await.find_one(Event.location == "Hybrid") # Returns a single event
```

- `.save()`, `.update()`и `.upsert()`: для обновления документа можно использовать любой из этих методов. Метод `.update()` принимает запрос на обновление, а метод `.upsert()` используется, когда документ не соответствует критериям поиска. Запрос на обновление — это инструкция, за которой следует база данных MongoDB, например, следующая:
  
	```python
event = await Event.get("74478287284ff") 
update_query = {"$set": {"location": "virtual"}} 
await event.update(update_query)
```
  
  В этом блоке кода мы сначала извлекаем событие, а затем создаем запрос на обновление, чтобы установить для поля location в коллекции событий значение virtual.

- `.delete()`: этот метод отвечает за удаление записи документа из базы данных, например:
  
  ```python
  event = await Event.get("74478287284ff") await event.delete()
```

Теперь, когда мы узнали, как работают методы, содержащиеся в библиотеке `Beanie`, давайте инициализируем базу данных в нашем приложении планировщика событий, определим наши документы и реализуем CRUD операции.

##### <font color="#46aa63">Инициализация базы данных</font>

`Pydantic` позволяет нам читать переменные среды, создавая дочерний класс родительского класса `BaseSettings`. При создании `веб-API` стандартной практикой является хранение переменных конфигурации в файле среды. Создадим файл `config/settings.py`:

```shell
mkdir config && touch config/settings.py && touch config/.env
```

Установим пакет `pydantic-settings`:

```shell
pip install pydantic-settings
```

В `settings.py` добавим следующее:

```python
from pathlib import Path  
  
from pydantic_settings import BaseSettings, SettingsConfigDict  
  
  
BASE_DIR = Path(__file__).parent.parent
  
  
class Settings(BaseSettings):  
    model_config = SettingsConfigDict(  
        env_nested_delimiter='__',  
        env_file_encoding='utf-8',  
        env_file=BASE_DIR / "config" / ".env"  
    )  
  
    mongo_url: str = "mongodb://localhost:27017/<dbname>"


settings = Settings()
```

В `.env` добавим следующее:

```txt
MONGO_URL="mongodb://localhost:27017/planner"
```

Теперь опишем модели, чтобы включить документы `MongoDB`. В `models/events.py` запишем следующее:

```python

from beanie import Document  
  
  
class Event(Document):  
  
    class Settings:  
        name = "events"  
  
    title: str | None  
    description: str | None  
    tags: list[str] | None  
    location: str | None  
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "title": "FastAPI Book Launch",  
                "description": "We will be discussing the contents of the FastAPI book in this event.",  
                "tags": ["Book", "FastAPI"],  
                "location": "Google Meet"  
            }  
        }  
    }

class EventUpdate(BaseModel):  
    title: str | None = None  
    description: str | None = None  
    tags: list[str] | None = None  
    location: str | None = None
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "title": "FastAPI Book Launch",  
                "description": "We will be discussing the contents of the FastAPI book in this event.",  
                "tags": ["Book", "FastAPI"],  
                "location": "Google Meet"  
            }  
        }  
    }

```

В `models/users.py` запишем следующее:

```python
from beanie import Document, Link  
  
from pydantic import EmailStr  
from models.events import Event  
  
class User(Document):  
    class Settings:  
        name = "users"  
  
    email: EmailStr  
    username: str  
    events: list[Link[Event]] | None = None
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "email": "fastapi@packt.com",  
                "username": "fastapi",  
            }  
        }  
    }
```

Теперь, когда мы определили документы, давайте опишем `connection.py:

```python
from beanie import init_beanie  
from fastapi import FastAPI  
from motor.motor_asyncio import AsyncIOMotorClient  
from contextlib import asynccontextmanager  
  
from config.settings import settings  
from models.users import User  
from models.events import Event  
  
  
@asynccontextmanager  
async def lifespan(app: FastAPI):  
    client = AsyncIOMotorClient(settings.mongo_url)  
    await init_beanie(database=client["planner"], document_models=[Event, User])  
  
    yield  
    client.close()
```


Давайте опишем все операции `CRUD` в отдельном классе в файле `database/objects.py`:

```python
from typing import Any  
  
from beanie import PydanticObjectId, Document  
from pydantic import BaseModel  
  
  
class Database:  
    def __init__(self, model):  
        self.model = model  
  
    @staticmethod  
    async def create(document: Document) -> None:  
        await document.create()  
        return  
  
    async def get(self, doc_id: PydanticObjectId) -> Any:  
        doc = await self.model.get(doc_id)  
        if doc:  
            return doc  
  
    async def get_all(self) -> list[Any]:  
        docs = await self.model.find_all().to_list()  
        return docs  
  
    async def update(self, doc_id: PydanticObjectId, body: BaseModel) -> Any:  
        des_body = body.dict()  
        des_body = {k: v for k, v in des_body.items() if v is not None}  
        update_query = {"$set": {  
            field: value for field, value in  
            des_body.items()}  
        }  
        doc = await self.get(doc_id)  
        if not doc:  
            return False  
        await doc.update(update_query)  
        return doc  
  
    async def delete(self, doc_id: PydanticObjectId) -> bool:  
        doc = await self.get(doc_id)  
        if not doc:  
            return False  
  
        await doc.delete()  
        return True
```

Теперь приступим к описанию маршрутов для событий в `routers/events.py`:

```python
from beanie import PydanticObjectId  
from fastapi import APIRouter, HTTPException, status  
  
from database.objects import Database  
from models.events import Event, EventUpdate  
  
event_router = APIRouter(tags=["Event"])  
event_database = Database(Event)  
  
@event_router.get("/", response_model=list[Event])  
async def retrieve_all_events() -> list[Event]:  
    events = await event_database.get_all() 
    return events  
  
  
@event_router.get("/{event_id}", response_model=Event)  
async def retrieve_event(event_id: PydanticObjectId) -> Event:  
    event = await event_database.get(event_id)  
    if not event:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID does not exist"  
        )  
    return event  
  
  
@event_router.post("/")  
async def create_event(body: Event) -> dict:  
    await event_database.create(body)  
    return {"message": "Event created successfully"}  
  
  
@event_router.put("/{event_id}", response_model=Event)  
async def update_event(event_id: PydanticObjectId, body: EventUpdate) -> Event:  
    updated_event = await event_database.update(event_id, body)  
    if not updated_event:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID does not exist"  
        )  
    return updated_event  
  
  
@event_router.delete("/{event_id}")  
async def delete_event(event_id: PydanticObjectId) -> dict:  
    event = await event_database.delete(event_id)  
    if not event:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID does not exist"  
        )  
    return {"message": "Event deleted successfully."}
```

Опишем маршруты пользователей  в `routers/users.py`:

```python
from beanie import PydanticObjectId  
from fastapi import APIRouter, HTTPException, status  
  
from database.objects import Database  
from models.users import User  
  
  
user_router = APIRouter(tags=["User"])  
user_database = Database(User)  
  
  
@user_router.get("/", response_model=list[User])  
async def retrieve_all_users() -> list[User]:  
    users = await User.get_all()  
    return users  
  
  
@user_router.get("/{user_id}", response_model=User)  
async def retrieve_user(user_id: PydanticObjectId) -> User:  
    user = await user_database.get(user_id)  
    if not user:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="User with supplied ID does not exist"  
        )  
    return user  
  
  
@user_router.post("/")  
async def create_user(body: User) -> dict:  
    await user_database.create(body)  
    return {"message": "User created successfully"}
```

Запустим `mongo` и наше приложение:

```shell
docker run --name planner-mongodb -p 27017:27017 -d mongodb/mongodb-community-server:latest

python main.py
```

Проверим что все запросы работают:

 -  POST `Event`
   
```shell
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 1", "description": "Test event 1", "tags": ["1", "2"], "location": "Goggle Meet"}'

{"message":"Event created successfully"}

curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 2", "description": "Test event 2", "tags": ["4", "5"], "location": "Goggle Meet"}'

{"message":"Event created successfully"}
```

- GET `Event`
  
```shell
curl -X 'GET' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json'

[{"_id":"66ec10b91f81569c77be30a2","title":"Event 1","description":"Test event 1","tags":["1","2"],"location":"Goggle Meet"},{"_id":"66ec10e01f81569c77be30a3","title":"Event 2","description":"Test event 2","tags":["4","5"],"location":"Goggle Meet"}]

curl -X 'GET' 'http://127.0.0.1:5000/events/66ec10b91f81569c77be30a2' -H 'accept: application/json' -H 'Content-Type: application/json'

{"_id":"66ec10b91f81569c77be30a2","title":"Event 1","description":"Test event 1","tags":["1","2"],"location":"Goggle Meet"}

```

- DELETE `Event`

```shell
curl -X 'DELETE' 'http://127.0.0.1:5000/events/66ec10e01f81569c77be30a3' -H 'accept: application/json' -H 'Content-Type: application/json'


{"message":"Event deleted successfully." 
```

- PUT `Event`

```shell
curl -X 'PUT' 'http://127.0.0.1:5000/events/66ec10b91f81569c77be30a2' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 1 update"}'

{"_id":"66ec10b91f81569c77be30a2", "title":"Event 1 update", "description":"Test event 1", "tags":["1","2"], "location":"Goggle Meet"}
```

- POST `User`

```shell
curl -X 'POST' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "fastapi@packt.com", "username": "FastPackt"}'

{"message":"User created successfully"}

curl -X 'POST' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "pydantic@packt.com", "username": "Pydantic", "events": ["66ec10b91f81569c77be30a2"]}'

{"message":"User created successfully"}
```

- GET `User`

```shell
curl -X 'GET' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json'

[{"_id":"66ec1e55793312d379675eef","email":"fastapi@packt.com","username":"FastPackt","events":null},{"_id":"66ec636d12b5f4bb39de99a3","email":"pydantic@packt.com","username":"Pydantic","events":[{"id":"66ec10b91f81569c77be30a2","collection":"events"}]}]

curl -X 'GET' 'http://127.0.0.1:5000/users/66ec636d12b5f4bb39de99a3' -H 'accept: application/json' -H 'Content-Type: application/json'
```


Давайте закомитим наши изменения:

```shell
git checkout -b planner-beanie
git add .
git commit -m "[Feature] Incorporate a Mongo database and implement CRUD operations"
git push -u origin planner-beanie
```

На этом занятии мы узнали, как добавлять базы данных SQL и NoSQL с помощью SQLModel и Beanie соответственно. Мы использовали все наши знания из предыдущих глав. Мы также проверили маршруты, чтобы убедиться, что они работают по плану.


### Задание

Предположим, у вас есть база данных `ecommerce`, содержащая следующие коллекции:

1. **products**: информация о товарах.
   - Поля: `product_id`, `name`, `category`, `price`, `stock`, `ratings`
  
2. **orders**: информация о заказах.
   - Поля: `order_id`, `user_id`, `product_id`, `quantity`, `order_date`, `status`

3. **users**: информация о пользователях.
   - Поля: `user_id`, `name`, `email`, `join_date`, `loyalty_points`

#### Задание 1

Найдите все товары в категории "Electronics", которые стоят меньше 500.

#### Задание 2

Получите все заказы, сделанные пользователем с `user_id = 1`, за последние 30 дней.

#### Задание 3

Найдите пользователей, которые сделали более 3 заказов.

#### Задание 4

Подсчитайте общее количество товаров в каждой категории.

#### Задание 5

Найдите общую выручку (сумма всех заказов) за последний месяц.
   
#### Задание 6

Создайте запрос, который показывает среднюю оценку товаров по категориям.
   
#### Задание 7

Найдите пользователей, которые не сделали ни одного заказа.
