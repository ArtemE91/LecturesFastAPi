# Alembic, ODM Beanie

### План занятия

1) Alembic
2) ODM Beanie

### <font color="#46aa63">Alembic</font>

При запуске приложения мы постоянно создавали таблицы и удаляли их после завершения работы приложения. Это делалось для того что таблицы базы данных всегда были актуальны описаным моделям. Проблема в этом подходе что при выключении приложения мы затирали все созданные записи. Решить нашу проблему нам поможет **Alembic**.

**Alembic** обеспечивает создание, управление и вызов сценариев управления изменениями для реляционной базы данных, используя `SQLAlchemy` в качестве базового движка.

Использование **Alembic** начинается с создания `среды миграции`. Это каталог скриптов, предназначенных для конкретного приложения. Среда миграции создается только один раз и затем поддерживается вместе с исходным кодом самого приложения. Среда создается с помощью команды `init`, а затем настраивается в соответствии с конкретными потребностями приложения.

Структура этой среды, включая некоторые сгенерированные сценарии миграций, выглядит следующим образом:

```txt
yourproject/
    alembic/
        env.py
        README
        script.py.mako
        versions/
            3512b954651e_add_account.py
            2b1ae634e5cd_add_order_id.py
            3adcc9a56557_rename_username_field.py
```

Каталог содержит следующие `каталоги/файлы`:

- `yourproject` - это корневой каталог исходного кода вашего приложения или какой-либо каталог внутри него.
  
- `alembic` - этот каталог находится в дереве исходного кода вашего приложения и является домашней средой миграции. Он может называться как угодно, и проект, использующий несколько баз данных, может даже содержать более одной.
  
- `env.py` - это скрипт на `Python`, который запускается всякий раз, когда вызывается инструмент миграции `alembic`. По крайней мере, в нем содержатся инструкции по настройке и созданию модуля `SQLAlchemy`, получению соединения от этого модуля вместе с транзакцией, а затем вызову модуля миграции, используя это соединение в качестве источника подключения к базе данных. 
  
  Сценарий `env.py` является частью сгенерированной среды, поэтому способ выполнения миграции полностью настраивается. Здесь приведены точные сведения о том, как подключаться, а также о том, как вызывается среда миграции. Сценарий может быть изменен таким образом, чтобы можно было использовать несколько движков, в среду миграции можно было передавать пользовательские аргументы, загружать и предоставлять доступ к библиотекам и моделям, специфичным для приложения.
  
  Alembic включает в себя набор шаблонов инициализации, которые содержат различные варианты `env.py` для различных случаев использования.
  
- `README` - входит в состав различных шаблонов среды и должен содержать что-то информативное.
  
- `script.py.mako` - это файл шаблона `Mako`, который используется для создания новых сценариев миграции. Все, что находится здесь, используется для создания новых файлов в `versions/`. Это возможно с помощью сценария, так что можно управлять структурой каждого файла миграции, включая стандартный импорт, который должен быть в каждом из них, а также изменения в структуре функций `upgrade()` и `downgrade()`. Например, среда `multidb` позволяет создавать множество функций, используя схему именования `upgrade_engine1()`, `upgrade_engine2()`.
  
- `versions/` - в этом каталоге хранятся отдельные версии скриптов. Пользователи других инструментов миграции могут заметить, что в файлах здесь не используются целые числа по возрастанию, а вместо этого используется частичный `GUID`. В `Alembic` порядок следования сценариев версий определяется директивами внутри самих сценариев, и теоретически возможно “сращивать” файлы версий между другими, позволяя объединять последовательности переноса из разных ветвей, хотя и аккуратно вручную.

Имея общее представление о том, что такое среда, мы можем создать ее с помощью команды `alembic init`. Это позволит создать среду с использованием шаблона `generic`. Установим пакет alembic и выполним команду `init alembic`:

```shell
pip install alembic
alembic init alembic

  Creating directory '/Users/arteme/PycharmProjects/planner/alembic' ...  done
  Creating directory '/Users/arteme/PycharmProjects/planner/alembic/versions' ...  done
  Generating /Users/arteme/PycharmProjects/planner/alembic/script.py.mako ...  done
  Generating /Users/arteme/PycharmProjects/planner/alembic/env.py ...  done
  Generating /Users/arteme/PycharmProjects/planner/alembic/README ...  done
  Generating /Users/arteme/PycharmProjects/planner/alembic.ini ...  done
  Please edit configuration/connection/logging settings in
  '/Users/arteme/PycharmProjects/planner/alembic.ini' before proceeding.
```

**Alembic** поместил файл `alembic.ini` в текущий каталог. Это файл, который ищет скрипт `alembic` при вызове. Этот файл может находиться в другом каталоге, местоположение которого указывается либо параметром `--config` для `alembic` runner,  переменной среды `ALEMBIC_CONFIG` (первое имеет приоритет).

Для создания первой миграции нам необходимо добавить несколько строк в `alembic/env.py`:

```python
from logging.config import fileConfig  
  
from sqlmodel import SQLModel  # NEW
from sqlalchemy import engine_from_config  
from sqlalchemy import pool  
  
from alembic import context  
  
from database.connection import sqlite_url  # NEW
from models.events import Event  # NEW
from models.users import User  # NEW
  
  
config = context.config  
  
  
if config.config_file_name is not None:  
    fileConfig(config.config_file_name)  
  
  
target_metadata = SQLModel.metadata  
  
  
def run_migrations_offline() -> None:  
    url = config.get_main_option("sqlalchemy.url")  
    context.configure(  
        url=url,  
        target_metadata=target_metadata,  
        literal_binds=True,  
        dialect_opts={"paramstyle": "named"},  
    )  
  
    with context.begin_transaction():  
        context.run_migrations()  
  
  
def run_migrations_online() -> None:  
    config.set_main_option("sqlalchemy.url", sqlite_url)  # NEW
  
    connectable = engine_from_config(  
        config.get_section(config.config_ini_section, {}),  
        prefix="sqlalchemy.",  
        poolclass=pool.NullPool,  
    )  
  
    with connectable.connect() as connection:  
        context.configure(  
            connection=connection, target_metadata=target_metadata  
        )  
  
        with context.begin_transaction():  
            context.run_migrations()  
  
  
if context.is_offline_mode():  
    run_migrations_offline()  
else:  
    run_migrations_online()
```

Выполним команду для создания миграции:

```shell
alembic revision --autogenerate -m "init"

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added table 'event'
  Generating /Users/arteme/PycharmProjects/planner/alembic/versions/d483bb54873e_init.py ...  done
```

В папке`versions` создался новый файл `d483bb54873e_init.py`. Посмотрим на него:

```python
"""init  
  
Revision ID: d483bb54873e  
Revises: Create Date: 2024-09-18 11:36:09.198610  
  
"""  
from typing import Sequence, Union  
  
import sqlmodel  
from alembic import op  
import sqlalchemy as sa  
  
  
# revision identifiers, used by Alembic.  
revision: str = 'd483bb54873e'  
down_revision: Union[str, None] = None  
branch_labels: Union[str, Sequence[str], None] = None  
depends_on: Union[str, Sequence[str], None] = None  
  
  
def upgrade() -> None:  
    # ### commands auto generated by Alembic - please adjust! ###  
    op.create_table('user',  
    sa.Column('id', sa.Integer(), nullable=False),  
    sa.Column('email', sqlmodel.sql.sqltypes.AutoString(), nullable=False),  
    sa.Column('username', sqlmodel.sql.sqltypes.AutoString(), nullable=False),  
    sa.PrimaryKeyConstraint('id')  
    )  
    op.create_table('event',  
    sa.Column('id', sa.Integer(), nullable=False),  
    sa.Column('title', sqlmodel.sql.sqltypes.AutoString(), nullable=False),  
    sa.Column('description', sqlmodel.sql.sqltypes.AutoString(), nullable=False),  
    sa.Column('user_id', sa.Integer(), nullable=True),  
    sa.ForeignKeyConstraint(['user_id'], ['user.id'], ),  
    sa.PrimaryKeyConstraint('id')  
    )  
    # ### end Alembic commands ###  
  
  
def downgrade() -> None:  
    # ### commands auto generated by Alembic - please adjust! ###  
    op.drop_table('event')  
    op.drop_table('user')  
    # ### end Alembic commands ###
```


Файл содержит некоторую информацию о заголовке, идентификаторы текущей версии и версии `downgrade` импорт основных директив `Alembic` и  функции `upgrade()` и `downgrade()`, которые будут применять набор изменений к нашей базе данных. Обычно требуется upgrade(), в то время как downgrade() требуется только в том случае, если требуется возможность пересмотра в сторону понижения.

Еще одна вещь, на которую следует обратить внимание, - это переменная `down_revision`. Именно так `Alembic` определяет правильный порядок применения изменений. Когда мы создаем следующую редакцию, идентификатор `down_revision` нового файла будет указывать на предыдущую.

Применим наши изменения:

```shell
alembic upgrade head

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> d483bb54873e, init

```

Команда выполнилась успешно и теперь мы подправим `main.py`:

```python
import uvicorn  
from fastapi import FastAPI  
  
from routers.users import user_router  
from routers.events import event_router  
  
  
app = FastAPI()  
app.include_router(user_router, prefix="/users")  
app.include_router(event_router, prefix="/events")  
  
  
if __name__ == "__main__":  
    uvicorn.run("main:app", host="127.0.0.1", port=5000, reload=True)
```

Запустим наше приложение и добавим несколько записей:

```shell
uvicorn main:app --host 127.0.0.1 --port 5000 --reload

curl -X 'POST' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "fastapi@packt.com", "username": "FastPackt"}'

 curl -X 'POST' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "pydantic@packt.com", "username": "Pydantic"}'
 
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 1", "description": "First test event", "user_id": 1}'

curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 2", "description": "Second test event", "user_id": 2}'
```

Как видим все работает. Например мы решили изменить модель `Event` добавив в нее поле `date_create`:

```python
class Event(SQLModel, table=True):  
    id: int | None = Field(default=None, primary_key=True)  
    title: str  
    description: str  
    date_create: datetime = Field(default_factory=datetime.utcnow, nullable=False)  
    user_id: int | None = Field(default=None, foreign_key="user.id")  
    user: User | None = Relationship(back_populates="events")  
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "title": "FastAPI Book Launch",  
                "description": "We will be discussing the contents of the FastAPI book in this event.",  
                "user_id": 1  
            }  
        }  
    }
```

Давайте создадим миграции:

```shell
alembic revision --autogenerate -m "add date_create in Event"
```

В папке `versions` появился новый файл миграции `610c0310c549_add_date_create_in_event.py`, мы видим что в функции  `upgrade()` создается новая колонка `date_create` в таблице `event`. Применим миграцию и создадим новый `Event`:

```shell
alembic upgrade head

sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) Cannot add a NOT NULL column with default value NULL
```

Как видим у нас возникла ошибка. Поле `date_create` обязательное и в базе уже есть записи, необходимо указать значение которое впишется в старые записи. Для этого в миграции мы добавим значение `server_default`:

```python
"""add date_create in Event  
  
Revision ID: 610c0310c549  
Revises: d483bb54873e  
Create Date: 2024-09-18 14:41:39.849824  
  
"""  
from datetime import datetime  
from typing import Sequence, Union  
  
from alembic import op  
import sqlalchemy as sa  
  
  
# revision identifiers, used by Alembic.  
revision: str = '610c0310c549'  
down_revision: Union[str, None] = 'd483bb54873e'  
branch_labels: Union[str, Sequence[str], None] = None  
depends_on: Union[str, Sequence[str], None] = None  
  
  
def upgrade() -> None:  
    # ### commands auto generated by Alembic - please adjust! ###  
    op.add_column('event', sa.Column('date_create', sa.DateTime(), nullable=False, server_default=str(datetime.utcnow())))  
    # ### end Alembic commands ###  
  
  
def downgrade() -> None:  
    # ### commands auto generated by Alembic - please adjust! ###  
    op.drop_column('event', 'date_create')  
    # ### end Alembic commands ###
```

Снова попробуем выполнить миграцию:

```shell
alembic upgrade head 

INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade d483bb54873e -> 610c0310c549, add date_create in Event
```

Все прошло успешно. Теперь в таблице `event` появился столбец `date_create`:

![Снимок экрана 2024-09-18 в 14.58.04.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-18%20%D0%B2%2014.58.04.png)

Добавим новое событие чтобы проверить что все работает:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 4", "description": "Test event", "user_id": 1}'

{"id":4,"user_id":1,"description":"Test event","date_create":"2024-09-18T11:56:15.030437","title":"Event 4"}
```

Мы успешно внедрили базу данных SQL в наше приложение с помощью SQLModel, а также реализовали CRUD операции. Давайте зафиксируем изменения, внесенные в приложение, прежде чем научиться реализовывать CRUD операции в MongoDB:

```shell
git add .

git commit -m "[Feature] Incorporate a ModelSQL database and implement CRUD operations"

git push -u origin planner-modelsql-sync
```

Вернемся на ветку `main`:

```shell
git checkout main
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

