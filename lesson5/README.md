# Структурирование приложений FastAPI, SQL Model

### План занятия

1) Структурирование маршрутов и моделей приложения
2) SQLModel

Структурирование относится к размещению компонентов приложения в организованном формате, который может быть модульным для улучшения читабельности кода и содержимого приложения. Приложение с правильной структурой обеспечивает более быструю разработку, более быструю отладку и общее повышение производительности.

### <font color="#46aa63">Структурирование маршрутов и моделей приложения</font>

Давайте разработаем структуру приложения, чтобы она выглядела так:

```txt
.
├── planner
│   ├── __init__.py
│   ├── main.py
│   └── database
│   │   ├── __init__.py
│   │   ├── connection.py
│   └── routers
│   │   ├── __init__.py
│   │   ├── events.py
│   │   └── users.py
│   └── models
│   │   ├── __init__.py
│   │   ├── events.py
│   │   └── users.py

```

Создадим новую папку для приложения и все необходимые файлы:

```shell
mkdir planner && cd planner
touch main.py
mkdir database routes models
touch {database,routes,models}/__init__.py
touch database/connection.py
touch {routes,models}/{events,users}.py
```

Каждый файл имеет свою функцию, как указано здесь:

- Модули в пакете **route**:
	-  `events.py` - этот модуль будет обрабатывать операции маршрутизации, такие как создание, обновление и удаление событий.
	-  `users.py` - этот модуль будет обрабатывать операции маршрутизации, такие как создание, обновление у удаление пользователей.
- Модули в пакете **models**:
	-  `events.py` - этот модуль будет содержать определение модели для операций с событиями.
	- `users.py` - этот модуль будет содержать определение модели для пользовательских операций.

##### <font color="#46aa63">Создание приложения для планирования мероприятий</font>

На этом занятии мы будем создавать приложение планировщика событий. В этом приложении зарегистрированные пользователи смогут создавать, обновлять и удалять события. Созданные события можно просмотреть, перейдя на страницу события, автоматически созданную приложением.

Каждый зарегистрированный пользователь и событие будут иметь уникальный идентификатор. Это сделано для предотвращения конфликтов при управлении пользователями и событиями с одним и тем же идентификатором. В этом разделе мы не будем отдавать приоритет аутентификации или управлению базой данных, так как это будет подробно обсуждаться на следующих занятиях.

Создадим виртуальную среду и активируем ее в каталоге нашего проекта:

```shell
python3 -m venv venv
source venv/bin/activate
```

Установим зависимости приложения:

```shell
pip install fastapi uvicorn "pydantic[email]"
```

Сохраним требования в `requirements.txt`:

```shell
pip freeze > requirements.txt
```

Добавим файл `.gitignore`:

```txt
.idea  
__pycache__  
*venv
```

Поскольку мы будем реализовывать базы данных `SQL` и `NoSQL` c разными `ORM`, давайте зальем наш проект  на `GitHub`. В своем терминале перейдите в каталог проекта, инициализируйте репозиторий GitHub и зафиксируйте существующие файлы:

```shell
git init  
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin git@github.com:ArtemE91/planner.git
git push -u origin main
```

Далее создадим новую ветку:

```shell
git checkout -b planner-modelsql-sync
```

##### <font color="#46aa63">Реализация моделей</font>

Первым шагом в создании нашего приложения является определение моделей для события и пользователя. Модели описывают, как данные будут храниться, вводиться и представляться в нашем приложении. На следующей диаграмме (https://dbdiagram.io/d) показано моделирование пользователя и события, а также их отношения:

![Снимок экрана 2024-09-17 в 21.57.16.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-17%20%D0%B2%2021.57.16.png)

Как показано на предыдущей диаграмме модели, у каждого пользователя будет поле множество Events, представляющее собой список событий, на которые он имеет право собственности.


### <font color="#46aa63">SQLModel</font>

Первым шагом для интеграции базы данных `SQL` в наше приложение планировщика является установка библиотеки **SQLModel**. Она предназначена для упрощения взаимодействия с базами данных `SQL` в приложениях `FastAPI`, она была создана тем же автором.

**SQLModel** сочетает в себе `SQLAlchemy` и `Pydantic` и пытается максимально упростить код, который вы пишете, что позволяет свести дублирование кода к минимуму, но при этом получить наилучший опыт разработчика.

**SQLModel** - это, по сути, тонкий слой поверх `Pydantic` и `SQLAlchemy`, тщательно разработанный, чтобы быть совместимым с обоими.

Установим пакет:

```shell
pip install sqlmodel
```

##### <font color="#46aa63">Реализация моделей с помощью SQLModel</font>

Чтобы создать таблицу с помощью `SQLModel`, сначала определяется класс модели таблицы. Как и в моделях `Pydantic`, таблица определяется, но на этот раз как подкласс класса `SQLModel`. Определение класса также принимает другую переменную конфигурации, таблицу, чтобы указать, что этот класс является таблицей `SQLModel`.

Переменные, определенные в классе, будут представлять столбцы по умолчанию, если они не обозначены как поля. Давайте посмотрим, как будет определена таблица `Event` в `models/events.py`:

```python
from pydantic import BaseModel  
from sqlmodel import SQLModel, Field, Relationship  
  
from models.users import User  
  
  
class Event(SQLModel, table=True):  
    id: int | None = Field(default=None, primary_key=True)  
    title: str  
    description: str  
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

class UpdateEvent(BaseModel):  
    title: str | None = None  
    description: str | None = None  
    user_id: int | None = None  
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "title": "FastAPI Book",  
                "description": "We will be discussing the contents of the FastAPI book in this event.",  
                "user_id": 2  
            }  
        }  
    }
```

Определим модель `User` в `models/users.py`:

```python
from pydantic import EmailStr  
from sqlmodel import SQLModel, Field as SQLField, Relationship  
  
  
class User(SQLModel, table=True):  
    id: int | None = SQLField(default=None, nullable=False, primary_key=True)  
    email: EmailStr  
    username: str  
    events: list['Event'] = Relationship(back_populates="user")  
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "email": "fastapi@packt.com",  
                "username": "fastapi",  
            }  
        }  
    }
```

##### <font color="#46aa63">Создание базы данных</font>

В `SQLModel` подключение к базе данных осуществляется с помощью механизма `SQLAlchemy`. Движок создается методом `create_engine()`, импортированным из библиотеки `SQLMode`l.

Метод `create_engine()` принимает в качестве аргумента URL-адрес базы данных. URL-адрес базы данных имеет вид `sqlite:///database.db` или `sqlite:///database. sqlite.` Он также принимает необязательный аргумент `echo`, который, если установлено значение `True` распечатывает команды `SQL`, выполняемые при выполнении операции.

 Чтобы создать файл базы данных, вызывается метод, `SQLModel.metadata.create_all(engine)`, аргументом которого является экземпляр метода `create_engine()`. Напишем в `connection.py`:

```python
from sqlmodel import SQLModel, create_engine, Session  
  
sqlite_file_name = "database.db"  
sqlite_url = f"sqlite:///{sqlite_file_name}"  
  
connect_args = {"check_same_thread": False}  
engine = create_engine(sqlite_url, echo=True, connect_args=connect_args)  
  
  
def conn():  
    SQLModel.metadata.create_all(engine)  


def drop_database():  
    SQLModel.metadata.drop_all(engine)
  
  
def get_session():  
    with Session(engine) as session:  
        yield session
```

##### <font color="#46aa63">Реализация маршрутов</font>

Следующим шагом в создании нашего приложения является настройка системы маршрутизации нашего API:

- Маршруты `User`:
	- `GET` `/users` - получить список пользователей
	- `GET` `/users/{user_id}` - получить пользователя по `id`
	- `POST` `/users` - создание пользователя
	- `DELETE` `/users/{user_id}` - удаление пользователя
- Маршруты `Event`:
	- `GET` `/events` - получить список событий
	- `GET` `/events/{event_id}` - получить событие по `id`
	- `POST` `/events` - создание события
	- `PATCH` `/events/{event_id}` - обновить событие по `id`
	- `DELETE` `/events` - удалить событие

##### <font color="#46aa63">Маршруты пользователей</font>

Начнем с определения маршрутов пользователя в `routes/users.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status  
from sqlmodel import Session, select  
  
from database.connection import get_session  
from models.users import User  
from models.events import Event  
  
  
user_router = APIRouter(tags=["User"])  
  
  
@user_router.get("/")  
async def get_all_users(session: Session = Depends(get_session)) -> list[User]:  
    users = session.exec(select(User)).all()  
    return users    # noqa  
  
  
@user_router.get("/{user_id}")  
async def get_user(user_id: int, session: Session = Depends(get_session)):  
    user = session.exec(select(User).where(User.id == user_id)).first()  
    if user is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="User with supplied ID doesn't exist"  
        )  
    return user  
  
  
@user_router.post("/")  
async def create_user(user: User, session: Session = Depends(get_session)):  
    user_db = User(**user.model_dump(exclude={"id"}))  
    session.add(user_db)  
    session.commit()  
    session.refresh(user_db)  
    return user_db  
  
  
@user_router.delete("/{user_id}")  
async def delete_user(user_id: int, session: Session = Depends(get_session)):  
    user = session.exec(select(User).where(User.id == user_id)).first()  
    if user is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="User with supplied ID doesn't exist"  
        )  
    session.delete(user)  
    session.commit()  
    return {"message": "User deleted successfully."}
```

Опишем в `main.py` запуск нашего приложения

```python
import uvicorn  
from fastapi import FastAPI  
  
from database.connection import conn, drop_database  
from routers.users import user_router  
  
  
app = FastAPI()  
app.include_router(user_router, prefix="/user")  
  
  
@app.on_event("startup")  
def on_startup():  
    conn()  
      
  
@app.on_event("shutdown")  
def shutdown():  
    drop_database()  
  
  
if __name__ == "__main__":  
    uvicorn.run("main:app", host="127.0.0.1", port=5000, reload=True)
```

Функция `on_startup` будет выполнена перед запуском приложения, в ней мы создадим файл нашей базы данных. Когда приложение будет остановлено выполниться функция `shutdown` в которой мы удалим базу данных.

Запустим наше приложение:

```shell
uvicorn main:app --host 127.0.0.1 --port 5000 --reload 

INFO:     Uvicorn running on http://127.0.0.1:5000 (Press CTRL+C to quit)
INFO:     Started reloader process [61780] using StatReload
INFO:     Started server process [61782]
INFO:     Waiting for application startup.
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine PRAGMA main.table_info("user")
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine [raw sql] ()
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine PRAGMA temp.table_info("user")
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine [raw sql] ()
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine PRAGMA main.table_info("event")
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine [raw sql] ()
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine PRAGMA temp.table_info("event")
2024-09-17 23:03:49,666 INFO sqlalchemy.engine.Engine [raw sql] ()
2024-09-17 23:03:49,667 INFO sqlalchemy.engine.Engine 
CREATE TABLE user (
        id INTEGER NOT NULL, 
        email VARCHAR NOT NULL, 
        username VARCHAR NOT NULL, 
        PRIMARY KEY (id)
)


2024-09-17 23:03:49,667 INFO sqlalchemy.engine.Engine [no key 0.00004s] ()
2024-09-17 23:03:49,668 INFO sqlalchemy.engine.Engine 
CREATE TABLE event (
        id INTEGER NOT NULL, 
        title VARCHAR NOT NULL, 
        description VARCHAR NOT NULL, 
        user_id INTEGER, 
        PRIMARY KEY (id), 
        FOREIGN KEY(user_id) REFERENCES user (id)
)


2024-09-17 23:03:49,668 INFO sqlalchemy.engine.Engine [no key 0.00004s] ()
2024-09-17 23:03:49,668 INFO sqlalchemy.engine.Engine COMMIT
INFO:     Application startup complete.

```

Теперь, когда мы определили маршруты для пользовательских операций, давайте зарегистрируем их в `main.py` и запустим наше приложение:

```python
import uvicorn  
from fastapi import FastAPI  
  
from routes.users import user_router  
  
  
app = FastAPI()  
app.include_router(user_router, prefix="/users") 
  
  
if __name__ == "__main__":  
    uvicorn.run("main:app", host="127.0.0.1", port=5000, reload=True)
```

```shell
python main.py
INFO:     Will watch for changes in these directories: ['/Users/arteme/PycharmProjects/planner']
INFO:     Uvicorn running on http://127.0.0.1:5000 (Press CTRL+C to quit)
INFO:     Started reloader process [40126] using StatReload
INFO:     Started server process [40128]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Теперь, когда наше приложение успешно запустилось, давайте проверим реализованные нами пользовательские маршруты. Начнем с создания пользователей:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "fastapi@packt.com", "username": "FastPackt"}'

{"username":"FastPackt","email":"fastapi@packt.com","id":1} 

 curl -X 'POST' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "pydantic@packt.com", "username": "Pydantic"}'
 
{"username":"Pydantic","email":"pydantic@packt.com","id":2}
```

Проверим маршруты получения пользователей:

```shell
curl -X 'GET' 'http://127.0.0.1:5000/users/' -H 'accept: application/json' -H 'Content-Type: application/json'

[{"id":1,"email":"fastapi@packt.com","username":"FastPackt"},{"id":2,"email":"pydantic@packt.com","username":"Pydantic"}]

curl -X 'GET' 'http://127.0.0.1:5000/users/1' -H 'accept: application/json' -H 'Content-Type: application/json'

{"username":"FastPackt","email":"fastapi@packt.com","id":1}  
```
\
Проверим маршрут удаления пользователя:

```shell
curl -X 'DELETE' 'http://127.0.0.1:5000/users/1' -H 'accept: application/json' -H 'Content-Type: application/json'

{"message":"User deleted successfully."}
```

##### <font color="#46aa63">Маршруты событий</font>

После создания пользовательских маршрутов следующим шагом будет реализация маршрутов для событийных операций в файле `routes/events.py`:

```python
from fastapi import APIRouter, HTTPException, status, Depends  
from sqlmodel import Session, select  
  
from database.connection import get_session  
from models.events import Event, UpdateEvent  
from models.users import User  
  
event_router = APIRouter(tags=["Events"])  
events = []  
  
  
@event_router.get("/")  
async def get_all_events(session: Session = Depends(get_session)) -> list[Event]:  
    events = session.exec(select(Event)).all()  
    return events    # noqa  
  
  
@event_router.get("/{event_id}")  
async def get_event(event_id: int, session: Session = Depends(get_session)):  
    event = session.exec(select(Event).where(Event.id == event_id)).first()  
    if event is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID doesn't exist"  
        )  
    return event  
  
  
@event_router.post("/")  
async def create_event(event: Event, session: Session = Depends(get_session)):  
    user = session.exec(select(User).where(User.id == event.user_id)).first()  
    if user is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="User with supplied ID doesn't exist"  
        )  
  
    event_db = Event(**event.model_dump(exclude={"id"}))  
    session.add(event_db)  
    session.commit()  
    session.refresh(event_db)  
    return event_db  
  
  
@event_router.patch("/{event_id}")  
async def patch_event(event_id: int, event: UpdateEvent, session: Session = Depends(get_session)):  
    event_db = session.exec(select(Event).where(Event.id == event_id)).first()  
    if event_db is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID doesn't exist"  
        )  
  
    for k, v in event.model_dump(exclude={"user_id"}).items():  
        if v is not None:  
            setattr(event_db, k, v)  
  
    if event.user_id:  
        user = session.exec(select(User).where(User.id == event.user_id)).first()  
        if user is None:  
            raise HTTPException(  
                status_code=status.HTTP_404_NOT_FOUND,  
                detail="User with supplied ID doesn't exist"  
            )  
        event_db.user = user  
  
    session.add(event_db)  
    session.commit()  
    session.refresh(event_db)  
  
    return event_db  
  
  
@event_router.delete("/{event_id}")  
async def delete_event(event_id: int, session: Session = Depends(get_session)):  
    event = session.exec(select(Event).where(Event.id == event_id)).first()  
    if event is None:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID doesn't exist"  
        )  
    session.delete(event)  
    session.commit()  
    return {"message": "Event deleted successfully."}
```

Зарегистрируем новые маршруты в `main.py` и запустим наше приложение:

```python
import uvicorn  
from fastapi import FastAPI  
  
from database.connection import conn, drop_database  
from routers.users import user_router  
from routers.events import event_router  
  
  
app = FastAPI()  
app.include_router(user_router, prefix="/users")  
app.include_router(event_router, prefix="/events")  
  
  
@app.on_event("startup")  
def on_startup():  
    conn()  
  
  
@app.on_event("shutdown")  
def shutdown():  
    drop_database()  
  
  
if __name__ == "__main__":  
    uvicorn.run("main:app", host="127.0.0.1", port=5000, reload=True)
```

Проверим маршрут создания событий:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 1", "description": "First test event", "user_id": 1}'

{"user_id":1,"id":1,"title":"Event 1","description":"First test event"}

curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 2", "description": "Second test event", "user_id": 2}'

{"user_id":2, "id":2, "title":"Event 2","description":"Second test event"}
```

Проверим маршруты получения событий:

```shell
curl -X 'GET' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json'

[{"user_id":1,"id":1,"title":"Event 1","description":"First test event"},{"user_id":5,"id":2,"title":"Event 2","description":"Second test event"},{"user_id":2,"id":3,"title":"Event 3","description":"Third test event"}]

curl -X 'GET' 'http://127.0.0.1:5000/events/1' -H 'accept: application/json' -H 'Content-Type: application/json'

{"user_id":1,"id":1,"title":"Event 1","description":"First test event"}
```

Проверим маршрут обновления события:

```shell
curl -X 'PATCH' 'http://127.0.0.1:5000/events/1' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 1 update"}'

{"description":"First test event","title":"Event 1 update","user_id":1,"id":1}

curl -X 'PATCH' 'http://127.0.0.1:5000/events/1' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"title": "Event 1 update user", "user_id": 2}'
{"description":"First test event","title":"Event 1 update user","user_id":2,"id":1}
```

Проверим маршрут удаления событий:

```shell
curl -X 'DELETE' 'http://127.0.0.1:5000/events/1' -H 'accept: application/json' -H 'Content-Type: application/json'

{"message":"Event deleted successfully."}
```


