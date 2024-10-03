# Защита приложений FastAPI

### План занятия

1) Методы аутентификации в FastApi
2) Защита приложения с помощью OAuth2 и JWT
3) Защита маршрутов с помощью внедрения зависимостей
4) Настройка CORS


На предыдущих занятиях мы рассмотрели, как подключить приложение FastAPI к базе данных SQL и NoSQL. Мы успешно реализовали методы базы данных и обновили существующие маршруты, чтобы обеспечить взаимодействие между приложением и базой данных. Однако приложение планировщика по-прежнему позволяет любому пользователю добавлять событие, а не только пользователям, прошедшим проверку подлинности. В этой главе мы защитим приложение с помощью **JSON веб-токенах (JWT)** и ограничим некоторые операции с событиями только для аутентифицированных пользователей.

Защита приложения включает в себя дополнительные меры безопасности для ограничения доступа к функциям приложения от неавторизованных лиц для предотвращения взлома или незаконных модификаций приложения. Аутентификация — это процесс проверки учетных данных, переданных объектом, а авторизация просто означает предоставление объекту разрешения на выполнение определенных действий. После проверки учетных данных объект получает право выполнять различные действия.

### <font color="#46aa63">Методы аутентификации в FastAPI</font>

В `FastAPI` доступно несколько методов аутентификации. `FastAPI` поддерживает распространенные методы аутентификации: базовую HTTP-аутентификацию, файлы cookie и аутентификацию на основе токенов. Давайте кратко рассмотрим, что влечет за собой каждый метод:

- **Базовая HTTP-аутентификация**: В этом методе аутентификации учетные данные пользователя, которые обычно представляют собой имя пользователя и пароль, отправляются через HTTP-заголовок авторизации. Запрос, в свою очередь, возвращает заголовок WWW-Authenticate содержащий, базовое значение и необязательный параметр области, который указывает ресурс, к которому выполняется запрос аутентификации.

- **Cookies**: Файлы cookie используются, когда данные должны храниться на стороне клиента, например, в веб-браузерах. Приложения `FastAPI` также могут использовать файлы cookie для хранения пользовательских данных, которые могут быть получены сервером в целях аутентификации.

- **Аутентификацию на основе токенов**: Этот метод аутентификации включает использование токенов безопасности, называемых токенами носителя. Эти токены отправляются вместе с ключевым словом `Bearer` в запросе заголовка авторизации. Наиболее часто используемым токеном является **JWT**, который обычно представляет собой словарь, содержащий идентификатор пользователя и срок действия токена.

Каждый из перечисленных здесь методов аутентификации имеет свои конкретные варианты использования, а также свои плюсы и минусы. Однако в этой главе мы будем использовать аутентификацию по токену.

Методы аутентификации внедряются в приложения FastAPI в виде зависимостей, которые вызываются во время выполнения. Это просто означает, что, когда методы аутентификации определены, они бездействуют до тех пор, пока не будут внедрены в место их использования. Это действие называется внедрением зависимостей.

##### <font color="#46aa63">Внедрение зависимостей</font>

Внедрение зависимостей — это шаблон, в котором объект — в данном случае функция — получает переменную экземпляра, необходимую для дальнейшего выполнения функции.

В FastAPI зависимости внедряются путем объявления их в аргументах функции операции пути. Например:

```python
@user_router.post("/signup")  
async def sign_user_up(user: User) -> dict:
	user_exist = await User.find_one(User.email == user.email)
```

В этом блоке кода определенная зависимость — это класс модели `User`, который вводится в функцию `sign_user_up()`. Внедрив модель `User` в аргумент пользовательской функции, мы можем легко получить атрибуты объекта.

##### <font color="#46aa63">Создание и использование зависимости</font>

В FastAPI зависимость может быть определена либо как функция, либо как класс. Созданная зависимость дает нам доступ к ее базовым значениям или методам, избавляя от необходимости создавать эти объекты в наследующих их функциях. Внедрение зависимостей помогает уменьшить повторение кода в некоторых случаях, например, при принудительной проверке подлинности и авторизации.

Пример зависимости определяется следующим образом:

```python
async def get_user(token: str): 
	user = decode_token(token) 
	return user
```

Эта зависимость представляет собой функцию, которая принимает `token` в качестве аргумента и возвращает `user` параметр из внешней функции, `decode_token`. Чтобы использовать эту зависимость, объявленный аргумент зависимой функции должен иметь параметр `Depends`, например:

```python
from fastapi import Depends

@router.get("/user/me")
async def get_user_details(user: User = Depends(get_user)): 
	return user
```

Функция маршрута здесь зависит от функции `get_user`, которая служит ее зависимостью. Это означает, что для доступа к предыдущему маршруту должна быть удовлетворена зависимость `get_user`.

Класс `Depends`, который импортируется из библиотеки `FastAPI`, отвечает за прием функции, переданной в качестве аргумента, и выполнение ее при вызове конечной точки, автоматически делая доступными для конечной точки, они возвращают значение переданной ей функции.

Теперь, когда у вас есть представление о том, как создается зависимость и как она используется, давайте создадим зависимость аутентификации для приложения планировщика событий.


### <font color="#46aa63">Защита приложения с помощью OAuth2 и JWT</font>

На этом занятии мы создадим систему аутентификации для приложения планировщика событий. Мы будем использовать поток паролей `OAuth2`, который требует от клиента отправки имени пользователя и пароля в качестве данных формы. Имя пользователя в нашем случае — это электронная почта, используемая при создании учетной записи.  

Когда данные формы отправляются на сервер от клиента, в качестве ответа отправляется токен доступа, который является подписанным `JWT`. Обычно выполняется фоновая проверка для проверки учетных данных, отправленных на сервер, перед созданием токена для дальнейшей авторизации. Чтобы авторизовать аутентифицированного пользователя, `JWT` имеет префикс `Bearer` при отправке через заголовок для авторизации действия на сервере.

**JWT** — это закодированная строка, обычно содержащая словарь, содержащий полезную нагрузку, подпись и алгоритм. **JWT** подписываются с использованием уникального ключа, известного только серверу и клиенту, чтобы избежать подделки закодированной строки внешним органом.

![Снимок экрана 2024-09-20 в 23.08.58.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-20%20%D0%B2%2023.08.58.png)

Теперь, когда у нас есть представление о том, как работает процесс аутентификации, давайте создадим необходимую папку и файлы, необходимые для настройки системы аутентификации в нашем приложении:

```shell
mkdir auth
cd auth && touch {__init__,jwt_handler,authenticate,hash_password}.py
```

Предыдущая команда создает четыре файла:

- **jwt_handler.py** - этот модуль будет содержать функции, необходимые для кодирования и декодирования строк JWT.
- **authenticate.py** - этот модуль будет содержать зависимость аутентификации, которая будет внедрена в наши маршруты для принудительной аутентификации и авторизации.
- **hash_password.py** - этот модуль будет содержать функции, которые будут использоваться для шифрования пароля пользователя при регистрации и сравнения паролей при входе.
- **init .py** - этот файл указывает содержимое папки как пакет. Теперь, когда файлы созданы, давайте создадим отдельные компоненты. Начнем с создания компонентов для хеширования паролей пользователей.

##### <font color="#46aa63">Хэширование паролей</font>

Хранить пароли пользователей в виде обычного текста это очень небезопасная и запрещенная практика при создании API. Пароли должны быть зашифрованы или хешированы с использованием соответствующих библиотек. Мы будем шифровать пароли пользователей с помощью **bcrypt** (адаптивная криптографическая хеш-функция формирования ключа, используемая для защищенного хранения паролей).

Давайте установим библиотеку `passlib`. В этой библиотеке находится алгоритм хеширования `bcrypt` , который мы будем использовать для хеширования паролей пользователей:

```shell
pip install passlib bcrypt
```

Теперь, когда мы установили библиотеку, давайте создадим функции для хеширования паролей в `hash_password.py`:

```python
from passlib.context import CryptContext  
  
  
pwd_context = CryptContext(schemes=["bcrypt"])  
  
  
class HashPassword:  
  
    @staticmethod  
    def create_hash(password: str):  
        return pwd_context.hash(password)  
  
    @staticmethod  
    def verify_hash(plain_password: str, hashed_password: str):  
        return pwd_context.verify(plain_password, hashed_password)
```

В предыдущем блоке кода мы начинаем с импорта `CryptContext`, который использует схему `bcrypt` для хеширования переданных ему строк. Контекст хранится в переменной контекста `pwd_context`, что дает нам доступ к методам, необходимым для выполнения нашей задачи.

Затем определяется класс `HashPassword`, который содержит два метода: `create_hash` и `verify_hash`:

-  `create_hash` принимает строку и возвращает хешированное значение.
- `verify_hash` берет простой пароль и хешированный пароль и сравнивает их. Функция возвращает логическое значение, указывающее, совпадают ли переданные значения или нет.

Теперь, когда мы создали класс для обработки хеширования паролей, давайте создадим маршрут регистрации, чтобы хешировать пароль пользователя перед его сохранением в базе данных:

```python
from auth.hash_password import HashPassword

hash_password = HashPassword()  
  
  
@user_router.post("/signup")  
async def sign_user_up(user: User) -> dict:  
    user_exist = await User.find_one(User.email == user.email)  
    if user_exist:  
        raise HTTPException(  
            status_code=status.HTTP_409_CONFLICT,  
            detail="User with email provided exists already."  
        )  
    hashed_password = hash_password.create_hash(user.password)  
    user.password = hashed_password  
    await user_database.create(user)  
    return {"message": "User created successfully"}
```

Добавим поле `password` в модель нашего пользователя:

```python
class User(Document):  
    class Settings:  
        name = "users"  
  
    email: EmailStr  
    username: str  
    password: str  
    events: list[Link[Event]] | None = None  
  
    model_config = {  
        "json_schema_extra": {  
            "example": {  
                "email": "fastapi@packt.com",  
                "username": "fastapi",  
                "password": "my-password",  
                "events": ["66ec10e01f81569c77be30a3"]  
            }  
        }  
    }
```


Теперь, когда мы добавили маршрут регистрации пользователя и хеширование пароля перед сохранением, давайте создадим нового пользователя для подтверждения. Запустим `mongodb` и наше приложение :

```shell
docker run --name planner-mongodb -p 27017:27017 -d mongodb/mongodb-community-server:latest

python main.py
```

Создадим пользователя:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/users/signup' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "fastapi@packt.com", "username": "FastPackt", "password": "my-password"}'

{"message":"User created successfully"}
```

Теперь, когда мы создали пользователя, давайте проверим, что пароль, отправленный в базу данных, был хеширован. Для этого мы создадим интерактивный сеанс `MongoDB`, который позволит нам запускать команды из базы данных.

В окне терминала выполним следующие команды:

```
docker ps -a | grep mongodb

c972f0d28519   mongodb/mongodb-community-server:latest   "python3 /usr/local/…"   37 hours ago    Up 37 hours                  0.0.0.0:27018->27017/tcp

docker exec -t -i c972f0d28519 /bin/bash
mongosh
use planner
db.users.find({})
```

В окне терминала увидим следующее:

![Снимок экрана 2024-09-20 в 23.44.21.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-20%20%D0%B2%2023.44.21.png)

Предыдущая команда возвращает список пользователей, и теперь мы можем подтвердить, что пароль пользователя был хеширован до того, как он был сохранен в базе данных. Теперь, когда мы успешно создали компоненты для безопасного хранения паролей пользователей, давайте создадим компоненты для создания и проверки `JWTS`.

##### <font color="#46aa63">Проверка и создание токенов доступа</font>

Создание `JWT` делает нас на шаг ближе к защите нашего приложения. Полезная нагрузка токена будет содержать идентификатор пользователя и время истечения срока действия перед кодированием в длинную строку, как показано здесь:

![Снимок экрана 2024-09-21 в 22.54.57.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%B2%2022.54.57.png)

Ранее мы узнали, почему `JWT` подписаны. `JWT` подписываются секретным ключом, известным только отправителю и получателю. Давайте обновим класс `Settings` в модуле `config/settings.py`, а также файл среды, `.env`, включив в него переменную `SECRET_KEY` , которая будет использоваться для подписи `JWTs`:

```python
class Settings(BaseSettings):  
    model_config = SettingsConfigDict(  
        env_nested_delimiter='__',  
        env_file_encoding='utf-8',  
        env_file=BASE_DIR / "config" / ".env"  
    )  
  
    secret_key: str | None = None  
    mongo_url: str = "mongodb://localhost:27017/planner"
```

```txt
MONGO_URL="mongodb://localhost:27018/planner"  
SECRET_KEY="us1v#3o1oj#%85cl8a1^*1!($-(=w*y^h^$$q9emen"
```

После этого добавим следующий импорт в `jwt_handler.py`:

```python
import time  
from datetime import datetime  
from fastapi import HTTPException, status  
from jose import jwt, JWTError  
from config.settings import settings
```

В предыдущем блоке кода мы импортировали модули `time`, класс `HTTPException`, а также статус из `FastAPI`. Мы также импортировали библиотеку `jose` отвечающую за кодирование и декодирование `JWT`, и наш `settings`.

Установим пакет `python-jose`:

```shell
pip install python-jose
```

Далее мы создадим функцию, отвечающую за создание токена:

```python
def create_access_token(user: str) -> str:  
    payload = {  
        "user": user,  
        "expires": time.time() + 3600  
    }  
    token = jwt.encode(payload, settings.secret_key, algorithm="HS256")  
    return token
```

В предыдущем блоке кода функция принимает строковый аргумент, который передается в словарь полезной нагрузки. Словарь полезной нагрузки содержит пользователя и время истечения срока действия, которое возвращается при декодировании `JWT`.

Срок действия устанавливается равным часу с момента создания. Затем полезная нагрузка передается методу `encode()`, который принимает три параметра:

- **Payload** - словарь, содержащий значения для кодирования.
- **Key** - ключ, используемый для подписи полезной нагрузки.
- **Algorithm** - алгоритм, используемый для подписи полезной нагрузки. По умолчанию и наиболее распространенным является алгоритм `HS256`.

Далее давайте создадим функцию для проверки подлинности токена, отправленного в наше приложение:

```python
def verify_access_token(token: str) -> dict:   
    try:  
        data = jwt.decode(token, settings.SECRET_KEY, algorithms=["HS256"])  
        expire = data.get("expires")  
        if expire is None:  
            raise HTTPException(  
                status_code=status.HTTP_400_BAD_REQUEST,  
                detail="No access token supplied"   
)  
        if datetime.utcnow() > datetime.utcfromtimestamp(expire):  
            raise HTTPException(  
                status_code=status.HTTP_403_FORBIDDEN,   
                detail="Token expired!"  
            )   
        return data  
    except JWTError:  
        raise HTTPException(  
            status_code=status.HTTP_400_BAD_REQUEST,  
            detail="Invalid token"  
        )
```

В предыдущем блоке кода функция принимает в качестве аргумента строку токена и выполняет несколько проверок в блоке `try`. Сначала функция проверяет срок действия токена. Если срок действия не указан, значит, токен не был предоставлен. Вторая проверка - валидность токена - генерируется исключение, информирующее пользователя об истечении срока действия токена. Если токен действителен, возвращается декодированная полезная нагрузка.

В блоке `except` для любой ошибки `JWT` выдается исключение неверного запроса.

Теперь, когда мы реализовали функции для создания и проверки токенов, отправляемых в приложение, давайте создадим функцию, которая проверяет аутентификацию пользователя и служит зависимостью.

##### <font color="#46aa63">Обработка аутентификации пользователя</font>

Мы успешно внедрили компоненты для хеширования и сравнения паролей, а также компоненты для создания и декодирования `JWT`. Давайте реализуем функцию зависимости, которая будет внедрена в маршруты событий. Эта функция будет служить единственным источником правды для извлечения пользователя для активного сеанса.

В `auth/authenticate.py` добавим следующее:

```python
from fastapi import Depends, HTTPException, status   
from fastapi.security import OAuth2PasswordBearer  
from auth.jwt_handler import verify_access_token   
  
  
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/user/signin")  
async def authenticate(token: str = Depends(oauth2_scheme)) -> str:  
    if not token:  
        raise HTTPException(  
            status_code=status.HTTP_403_FORBIDDEN,  
            detail="Sign in for access"  
        )  
          
    decoded_token = verify_access_token(token)   
    return decoded_token["user"]
```

В предыдущем блоке кода мы начинаем с импорта необходимых зависимостей:

- **Depends** - это вводит `oauth2_scheme` в функцию в качестве зависимости
- **OAuth2PasswordBearer** - этот класс сообщает приложению, что схема безопасности присутствует.
- **verify_access_token** - эта функция, определенная  в разделе создания и проверки токена доступа, будет использоваться для проверки действительности токена.

Затем мы определяем `URL` токена для схемы `OAuth2` и функцию аутентификации. Функция аутентификации принимает токен в качестве аргумента. В функцию в качестве зависимости внедрена схема `OAuth`. Токен декодируется, и пользовательское поле полезной нагрузки возвращается, если токен действителен, в противном случае возвращаются адекватные ответы об ошибках, как определено в функции `verify_access_token`.

Теперь, когда мы успешно создали зависимость для защиты маршрутов, давайте обновим поток аутентификации в маршрутах, а также добавим функцию аутентификации в маршруты событий.

##### <font color="#46aa63">Обновление приложения</font>

В `routes/users.py` добавим маршрут для входа пользователя:

```python
from beanie import PydanticObjectId  
from fastapi import APIRouter, HTTPException, status, Depends  
from fastapi.security import OAuth2PasswordRequestForm  
  
from auth.hash_password import HashPassword  
from auth.jwt_handler import create_access_token  
from database.objects import Database  
from models.users import User  
  
  
user_router = APIRouter(tags=["User"])  
user_database = Database(User)  
hash_password = HashPassword()  
  
  
@user_router.post("/signin", response_model=TokenResponse)  
async def sign_user_in(user: OAuth2PasswordRequestForm = Depends()) -> dict:  
    user_exist = await User.find_one(User.username == user.username)  
    if hash_password.verify_hash(user.password, user_exist.password):  
        access_token = create_access_token(user_exist.email)  
        return {  
            "access_token": access_token, "token_type": "Bearer"  
        }```

В предыдущем блоке кода мы внедрили класс `OAuth2PasswordRequestForm` в качестве зависимости для этой функции, гарантируя строгое соблюдение спецификации `OAuth`. В теле функции мы сравниваем пароль и возвращаем токен доступа и тип токена. Прежде чем протестировать обновленный маршрут, давайте создадим модель ответа для маршрута входа в `models/users.py`:

```python
class TokenResponse(BaseModel):  
    access_token: str  
    token_type: str
```

Обновим импорт и модель ответа для маршрута входа:

```python
from models.users import User, TokenResponse


@user_router.post("/signin", response_model=TokenResponse)  
...
```

Так как мы используем `Form` в маршруте входа пользователя, нам необходимо установить пакет `python-multipart` (https://fastapi.tiangolo.com/ru/tutorial/request-forms/):

```shell
pip install python-multipart
```

Давайте посетим интерактивные документы, чтобы убедиться, что тело запроса соответствует спецификациям `OAuth2` по адресу http://127.0.0.1:5000/docs:

![Снимок экрана 2024-09-21 в 21.36.36.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%B2%2021.36.36.png)

Давайте войдем в систему, чтобы убедиться, что маршрут работает правильно

```shell
curl -X 'POST' 'http://127.0.0.1:5000/users/signin' -H 'accept:application/json' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=FastPackt&password=my-password'

```

Возвращаемый ответ представляет собой токен доступа и тип токена:

```json
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZmFzdGFwaUBwYWNrdC5jb20iLCJleHBpcmVzIjoxNzI2OTQ4ODkzLjc2NTAwNH0.L7k_-aMuaoiRBYayHyAPCNLIF-wkEdtlutQdLD7mV48","token_type":"Bearer"}
```

Теперь, когда мы убедились, что маршрут работает должным образом, давайте обновим маршруты событий, чтобы разрешать только авторизованным пользователям события `CREATE`, `UPDATE` и `DELETE`.

### <font color="#46aa63">Защита маршрутов с помощью внедрения зависимостей</font>

Теперь, когда у нас есть наша аутентификация, давайте добавим зависимость аутентификации в функции маршрута `POST`, `PUT` и `DELETE`:

```python
from fastapi import APIRouter, HTTPException, status, Depends
from auth.authenticate import authenticate 

@event_router.post("/")  
async def create_event(body: Event, user: str = Depends(authenticate)) -> dict:
	...

@event_router.put("/{event_id}", response_model=Event)  
async def update_event(event_id: PydanticObjectId, body: EventUpdate, user: str = Depends(authenticate)) -> Event:
	...


@event_router.delete("/{event_id}")  
async def delete_event(event_id: PydanticObjectId, user: str = Depends(authenticate)) -> dict:
	...

```

После внедрения зависимостей веб-сайт интерактивной документации автоматически обновляется для отображения защищенных маршрутов. Если мы войдем в http://127.0.0.1:5000/docs, мы увидим кнопку авторизации в правом верхнем углу и замки на маршрутах событий:

![Снимок экрана 2024-09-21 в 22.22.38.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%B2%2022.22.38.png)

Если мы нажмем кнопку **«Authorize»**, отобразится модальное окно входа. Ввод наших учетных данных и пароля возвращает следующий экран:

![Снимок экрана 2024-09-21 в 22.27.56.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%B2%2022.27.56.png)

Теперь, когда мы успешно вошли в систему, мы можем создать событие:

![Снимок экрана 2024-09-21 в 22.30.20.png](images%2F%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%B2%2022.30.20.png)

Те же операции можно выполнить из командной строки. Во-первых, давайте получим наш токен доступа:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/users/signin' -H 'accept:application/json' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=FastPackt&password=my-password'
```

Отправленный запрос возвращает токен доступа, который представляет собой строку `JWT`, и тип токена, который имеет тип `Bearer`:

```json
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZmFzdGFwaUBwYWNrdC5jb20iLCJleHBpcmVzIjoxNzI2OTUxNDEzLjU3NzM1NH0.dgmQ5K64BbI7oIkxKuhFDvqRGJfpbbn8LxpBL9yp34o","token_type":"Bearer"}
```

Теперь давайте создадим новое событие из командной строки:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZmFzdGFwaUBwYWNrdC5jb20iLCJleHBpcmVzIjoxNzI2OTUxNDEzLjU3NzM1NH0.dgmQ5K64BbI7oIkxKuhFDvqRGJfpbbn8LxpBL9yp34o' -H 'Content-Type: application/json' -d '{
"title": "FastAPI Book Launch CLI",  
"description": "We will be discussing the contents of the FastAPI book in this event.Ensure to come with your own copy to win gifts!",
"tags": [ "python", "fastapi", "book", "launch"], "location": "Google Meet"}'
```

В отправленном здесь запросе также отправляется заголовок `Authorization: Bearer,` чтобы сообщить приложению, что мы уполномочены выполнять это действие. Полученный ответ следующий:

```json
{"message":"Event created successfully"}
```

Если мы попытаемся создать событие без передачи заголовка авторизации с действительным токеном, будет возвращена ошибка `HTTP 401 Unauthorized`:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{
"title": "FastAPI Book Launch CLI",  
"description": "We will be discussing the contents of the FastAPI book in this event.Ensure to come with your own copy to win gifts!",
"tags": [ "python", "fastapi", "book", "launch"], "location": "Google Meet"}'
```

Ответ:

```json
{"detail":"Not authenticated"}
```

Теперь, когда мы успешно защитили маршруты для событий, давайте обновим защищенные маршруты следующим образом:

- `POST` - добавление созданного события в список событий, принадлежащих пользователю
- `UPDATE` - может обновить только пользователь который создал это событие
- `DELETE` - удалить событие может только пользователь который его создал

В предыдущем разделе мы успешно внедрили зависимости аутентификации в наши операции маршрутизации. Чтобы легко идентифицировать события и предотвратить удаление пользователем события другого пользователя, мы обновим класс документа события, а также маршруты.

##### <font color="#46aa63">Обновление класса документа события и маршрутов</font>

Добавим поле `creator` в класс документа `Event` в `models/events.py`:

```python
class Event(Document):  
  
    class Settings:  
        name = "events"  
  
    creator: str | None = None  
    title: str | None = None  
    description: str | None = None  
    tags: list[str] | None = None  
    location: str | None = None
```

Это поле позволит нам ограничить операции, выполняемые с событием, только пользователем.

Удалим все созданные до этого события:

```shell
docker exec -t -i c972f0d28519 /bin/bash
mongosh
use planner
db.events.deleteMany({})
```

Далее давайте изменим маршрут POST, чтобы обновить поле `creator` при создании нового события в `routes/events.py`:

```python
@event_router.post("/")  
async def create_event(body: Event, user: str = Depends(authenticate)) -> dict:  
    body.creator = user  
    await event_database.create(body)  
    return {"message": "Event created successfully"}
```

В предыдущем блоке кода мы обновили маршрут `POST`, чтобы добавить адрес электронной почты текущего пользователя в качестве создателя события. Если вы создаете новое мероприятие, оно сохраняется вместе с адресом электронной почты создателя:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/events/' -H 'accept: application/json' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZmFzdGFwaUBwYWNrdC5jb20iLCJleHBpcmVzIjoxNzI2OTUxNDEzLjU3NzM1NH0.dgmQ5K64BbI7oIkxKuhFDvqRGJfpbbn8LxpBL9yp34o' -H 'Content-Type: application/json' -d '{
"title": "FastAPI Book Launch CLI",  
"description": "We will be discussing the contents of the FastAPI book in this event.Ensure to come with your own copy to win gifts!",
"tags": [ "python", "fastapi", "book", "launch"], "location": "Google Meet"}'
```

Ответ, возвращенный из запроса выше:

```json
{"message":"Event created successfully"}
```

Далее давайте получим список событий, хранящихся в базе данных:

```shell
curl -X 'GET' 'http://127.0.0.1:5000/events/' -H 'accept:application/json'
```

Ответ:

```json
[{"_id":"66ef27349b2ec29fe898ca49","creator":"fastapi@packt.com","title":"FastAPI Book Launch CLI","description":"We will be discussing the contents of the FastAPI book in this event.Ensure to come with your own copy to win gifts!","tags":["python","fastapi","book","launch"],"location":"Google Meet"}]
```

Далее давайте обновим маршрут `UPDATE`:

```python
@event_router.put("/{event_id}", response_model=Event)  
async def update_event(event_id: PydanticObjectId, body: EventUpdate, user: str = Depends(authenticate)) -> Event:  
    event = await event_database.get(event_id)  
    if event.creator != user:  
        raise HTTPException(  
            status_code=status.HTTP_400_BAD_REQUEST,   
            detail="Operation not allowed"  
        )  
    updated_event = await event_database.update(event_id, body)  
    if not updated_event:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID does not exist"  
        )  
    return updated_event
```

В предыдущем блоке кода функция маршрута проверяет, может ли текущий пользователь редактировать событие, прежде чем продолжить, в противном случае она вызывает исключение неверного запроса `HTTP 400`.  

Создадим нового пользователя и получим его токен:

```shell
curl -X 'POST' 'http://127.0.0.1:5000/users/signup' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"email": "pydantic@packt.com", "username": "Pydantic", "password": "111111"}'

curl -X 'POST' 'http://127.0.0.1:5000/users/signin' -H 'accept:application/json' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=Pydantic&password=111111'
```

Ответ:

```json
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoicHlkYW50aWNAcGFja3QuY29tIiwiZXhwaXJlcyI6MTcyNjk1MzYwMi42MjkxNX0.ADYe5VXbl7rC6vbNCBZTD3roBKMfmuvx6P5772DuHoY","token_type":"Bearer"}
```

Теперь попробуем обновить событие из под этого пользователя:

```shell
curl -X 'PUT' 'http://127.0.0.1:5000/events/66ef27349b2ec29fe898ca49' -H 'accept: application/json' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoicHlkYW50aWNAcGFja3QuY29tIiwiZXhwaXJlcyI6MTcyNjk1MzYwMi42MjkxNX0.ADYe5VXbl7rC6vbNCBZTD3roBKMfmuvx6P5772DuHoY' -H 'Content-Type: application/json' -d '{"title": "FastAPI Book Launch"}'
```

Ответ:

```json
{"detail":"Operation not allowed"}
```

Давайте обновим маршрут `DELETE`:

```python
@event_router.delete("/{event_id}")  
async def delete_event(event_id: PydanticObjectId, user: str = Depends(authenticate)) -> dict:  
    event = await event_database.get(event_id)  
    if event.creator != user:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event not found"  
        )  
    event = await event_database.delete(event_id)  
    if not event:  
        raise HTTPException(  
            status_code=status.HTTP_404_NOT_FOUND,  
            detail="Event with supplied ID does not exist"  
        )  
    return {"message": "Event deleted successfully."}
```

В предыдущем блоке кода мы указываем функции маршрута сначала проверить, является ли текущий пользователь создателем, в противном случае вызвать исключение. Давайте рассмотрим пример, когда другой пользователь пытается удалить событие другого пользователя:

```shell
curl -X 'DELETE' 'http://127.0.0.1:5000/events/66ef27349b2ec29fe898ca49' -H 'accept: application/json' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoicHlkYW50aWNAcGFja3QuY29tIiwiZXhwaXJlcyI6MTcyNjk1MzYwMi42MjkxNX0.ADYe5VXbl7rC6vbNCBZTD3roBKMfmuvx6P5772DuHoY'
```

Ненайденное событие возвращается в качестве ответа:

```json
{"detail":"Event not found"}
```

Однако владелец может удалить событие:

```shell
curl -X 'DELETE' 'http://127.0.0.1:5000/events/66ef27349b2ec29fe898ca49' -H 'accept: application/json' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZmFzdGFwaUBwYWNrdC5jb20iLCJleHBpcmVzIjoxNzI2OTUxNDEzLjU3NzM1NH0.dgmQ5K64BbI7oIkxKuhFDvqRGJfpbbn8LxpBL9yp34o'
```

Ответ:

```shell
{"message": "Event deleted successfully."}
```

Мы успешно защитили наше приложение и его маршруты. Давайте завершим эту лекцию, настроив промежуточное ПО `Cross-Origin Resource Sharing (CORS)`.

### <font color="#46aa63">Настройка CORS</font>

Совместное использование ресурсов между источниками (CORS) служит правилом, которое предотвращает доступ незарегистрированных клиентов к ресурсу.

Когда наш веб-API используется внешним приложением, браузер не разрешает HTTP-запросы из разных источников. Это означает, что доступ к ресурсам возможен только из точного источника, как API или источников, разрешенных API.

FastAPI предоставляет `CORSMiddleware`, которое позволяет нам регистрировать домены, которые могут получить доступ к нашему API. `Middleware` принимает массив источников, которым будет разрешен доступ к ресурсам на сервере.

**Middleware** — это функция, выступающая посредником между операциями. В веб-API middleware служит посредником в операции запрос-ответ.

Например, чтобы разрешить доступ к нашему `API` только `Packt`, мы определяем URL- адреса в исходном массиве:

```python
origins = [ “http://packtpub.com”, “https://packtpub.com”]
```

Чтобы разрешить запросы от любого клиента, массив `origins` будет содержать только одно значение — звездочку (`*`). Звездочка - это подстановочный знак, который указывает нашему API разрешать запросы из любого места.

В `main.py` настроим приложение так, чтобы оно принимало запросы отовсюду:

```python
import uvicorn  
from fastapi import FastAPI  
from fastapi.middleware.cors import CORSMiddleware  
  
from database.connection import lifespan  
from routers.users import user_router  
from routers.events import event_router  
  
origins = ["*"]  
  
app = FastAPI(lifespan=lifespan)  
  
app.add_middleware(  
    CORSMiddleware,  
    allow_origins=origins,  
    allow_credentials=True,  
    allow_methods=["*"],  
    allow_headers=["*"]  
)  
  
app.include_router(user_router, prefix="/users")  
app.include_router(event_router, prefix="/events")  
  
  
if __name__ == "__main__":  
    uvicorn.run("main:app", host="127.0.0.1", port=5000, reload=True)
```

В приведенном выше блоке кода мы начали с импорта класса `CORSMiddleware` из `FastAPI`. Мы зарегистрировали массив `origins` и, наконец, зарегистрировали промежуточное ПО в приложении с помощью метода `add_middleware` (https://fastapi.tiangolo.com/tutorial/cors/#use-corsmiddleware).

Мы успешно настроили наше приложение, чтобы разрешить запросы из любого источника.

Закомитим наши изменения и зальем на удаленный сервер:

```shell
git add.
git commit -m "[Feature] Added JWT authorization and route protection for events"
git push origin planner-beanie                                          
```

### Резюме:

На этой лекции мы узнали, как защитить приложение FastAPI с помощью OAuth и JWT. Мы также узнали, что такое внедрение зависимостей, как оно используется в приложениях FastAPI и как защитить маршруты от неавторизованных пользователей. Мы также добавили промежуточное ПО CORS, чтобы разрешить доступ к нашему API из любого клиента. Мы использовали знания из предыдущих занятий.



