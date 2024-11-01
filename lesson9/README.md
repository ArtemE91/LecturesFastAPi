# ClickHouse, RabbitMQ и FastStream

### План занятия

1) ClickHouse
2) RabbitMQ
3) FastStream


### <font color="#46aa63">Что такое ClickHouse?</font>

[ClickHouse](https://clickhouse.com/docs/ru) — это открытая колоночная база данных, разработанная Яндексом, предназначенная для высокопроизводительной аналитики. Она оптимизирована для обработки больших объемов данных и предназначена для выполнения сложных запросов в реальном времени.

В обычной, «строковой» СУБД, данные хранятся в таком порядке:

| Строка | WatchID     | JavaEnable | Title              | GoodEvent | EventTime           |
| ------ | ----------- | ---------- | ------------------ | --------- | ------------------- |
| #0     | 89354350662 | 1          | Investor Relations | 1         | 2016-05-18 05:19:20 |
| #1     | 90329509958 | 0          | Contact us         | 1         | 2016-05-18 08:10:20 |
| #2     | 89953706054 | 1          | Mission            | 1         | 2016-05-18 07:38:00 |
| #N     | ...         | ...        | ...                | ...       | ...                 |

То есть, значения, относящиеся к одной строке, физически хранятся рядом.
Примеры строковых СУБД: MySQL, PostgreSQL, MS SQL Server. 

В столбцовых СУБД данные хранятся в таком порядке:

|Строка:|#0|#1|#2|#N|
|---|---|---|---|---|
|WatchID:|89354350662|90329509958|89953706054|...|
|JavaEnable:|1|0|1|...|
|Title:|Investor Relations|Contact us|Mission|...|
|GoodEvent:|1|1|1|...|
|EventTime:|2016-05-18 05:19:20|2016-05-18 08:10:20|2016-05-18 07:38:00|...|

В примерах изображён только порядок расположения данных. То есть значения из разных столбцов хранятся отдельно, а данные одного столбца — вместе.

Примеры столбцовых СУБД: Vertica, Paraccel (Actian Matrix, Amazon Redshift), Sybase IQ, Exasol, Infobright, InfiniDB, MonetDB (VectorWise, Actian Vector), LucidDB, SAP HANA и прочий треш, Google Dremel, Google PowerDrill, Druid, kdb+. 

Разный порядок хранения данных лучше подходит для разных сценариев работы. Сценарий работы с данными — это то, какие производятся запросы, как часто и в каком соотношении; сколько читается данных на запросы каждого вида — строк, столбцов, байтов; как соотносятся чтения и обновления данных; какой рабочий размер данных и насколько локально он используется; используются ли транзакции и с какой изолированностью; какие требования к дублированию данных и логической целостности; требования к задержкам на выполнение и пропускной способности запросов каждого вида и т. п.

Чем больше нагрузка на систему, тем более важной становится специализация под сценарий работы, и тем более конкретной становится эта специализация. Не существует системы, одинаково хорошо подходящей под существенно различные сценарии работы. Если система подходит под широкое множество сценариев работы, то при достаточно большой нагрузке система будет справляться со всеми сценариями работы плохо, или справляться хорошо только с одним из сценариев работы.


##### <font color="#46aa63">Ключевые особенности OLAP-сценария работы</font>

- подавляющее большинство запросов — на чтение;
- данные обновляются достаточно большими пачками (> 1000 строк), а не по одной строке, или не обновляются вообще;
- данные добавляются в БД, но не изменяются;
- при чтении «вынимается» достаточно большое количество строк из БД, но только небольшое подмножество столбцов;
- таблицы являются «широкими», то есть содержат большое количество столбцов;
- запросы идут сравнительно редко (обычно не более сотни в секунду на сервер);
- при выполнении простых запросов, допустимы задержки в районе 50 мс;
- значения в столбцах достаточно мелкие — числа и небольшие строки (например, 60 байт на URL);
- требуется высокая пропускная способность при обработке одного запроса (до миллиардов строк в секунду на один сервер);
- транзакции отсутствуют;
- низкие требования к согласованности данных;
- в запросе одна большая таблица, все остальные таблицы из запроса — маленькие;
- результат выполнения запроса существенно меньше исходных данных — то есть данные фильтруются или агрегируются; результат выполнения помещается в оперативную память одного сервера.

Легко видеть, что OLAP-сценарий работы существенно отличается от других распространённых сценариев работы (например, OLTP или Key-Value сценариев работы). Таким образом, не имеет никакого смысла пытаться использовать OLTP-системы или системы класса «ключ — значение» для обработки аналитических запросов, если вы хотите получить приличную производительность («выше плинтуса»). Например, если вы попытаетесь использовать для аналитики MongoDB или Redis — вы получите анекдотически низкую производительность по сравнению с OLAP-СУБД.

##### <font color="#46aa63">Основные характеристики ClickHouse</font>

1. **Колоночное хранилище** - данные хранятся по столбцам, что значительно ускоряет чтение и агрегирование данных, особенно для аналитических запросов.

2. **Высокая производительность** - ClickHouse обеспечивает быстрое выполнение запросов даже на больших наборах данных, благодаря использованию параллельной обработки и эффективным алгоритмам сжатия.

3. **SQL-подобный язык запросов** - поддержка привычного SQL, что упрощает интеграцию для пользователей, знакомых с реляционными базами данных.

4. **Сжатие данных** -  ClickHouse использует различные методы сжатия для экономии пространства, что позволяет хранить большие объемы данных более эффективно.

5. **Масштабируемость** - поддержка горизонтального масштабирования, что позволяет добавлять новые узлы в кластер для увеличения производительности и емкости.

6. **Поддержка аналитических функций** - включает оконные функции, агрегации и группы, что делает его подходящим для аналитических задач.

ClickHouse широко используется в областях, где необходима быстрая обработка и анализ данных, таких как веб-аналитика, мониторинг и аналитика в реальном времени.


### <font color="#46aa63">Установка и настройка</font>

##### <font color="#46aa63">Системные требования</font>

ClickHouse может работать на любой операционной системе Linux, FreeBSD или Mac OS X с архитектурой процессора x86-64, AArch64 или PowerPC64LE.

Предварительно собранные пакеты компилируются для x86-64 и используют набор инструкций SSE 4.2, поэтому, если не указано иное, его поддержка в используемом процессоре, становится дополнительным требованием к системе. Вот команда, чтобы проверить, поддерживает ли текущий процессор SSE 4.2:

```shell
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"
```

Чтобы запустить ClickHouse на процессорах, которые не поддерживают SSE 4.2, либо имеют архитектуру AArch64 или PowerPC64LE, необходимо самостоятельно [собрать ClickHouse из исходного кода](https://clickhouse.com/docs/ru/getting-started/install#from-sources) с соответствующими настройками конфигурации.

##### <font color="#46aa63">Доступные варианты установки</font>

##### Из deb-пакетов

Рекомендуется использовать официальные скомпилированные `deb`-пакеты для Debian или Ubuntu. Для установки пакетов выполните:

```shell
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL 'https://packages.clickhouse.com/rpm/lts/repodata/repomd.xml.key' | sudo gpg --dearmor -o /usr/share/keyrings/clickhouse-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/clickhouse-keyring.gpg] https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update
```

Установка ClickHouse server и client:

```
sudo apt-get install -y clickhouse-server clickhouse-client
```

Запуск ClickHouse server:

```
sudo service clickhouse-server startclickhouse-client # or "clickhouse-client --password" if you've set up a password.
```

##### Из Docker образа

Для запуска ClickHouse в Docker нужно следовать инструкции на [Docker Hub](https://hub.docker.com/r/clickhouse/clickhouse-server/). Внутри образов используются официальные `deb`-пакеты.

Пример запуска:

```shell
docker run -d --name some-clickhouse-server --ulimit nofile=262144:262144 clickhouse/clickhouse-server
```


Остальные варианты установки можно посмотреть на [официальном сайте](https://clickhouse.com/docs/ru/getting-started/install).


Давайте запустим `Clickhouse`:

```shell
docker volume create volume-clickhouse

docker run -d --name some-clickhouse-server -v volume-clickhouse:/etc/clickhouse-server -p 8123:8123 -p 9000:9000 -p 9009:9009 --ulimit nofile=262144:262144 clickhouse/clickhouse-server
```

Подключимся к `clickhouse`:

```shell
docker exec -i -t <CONTAINER_ID> /bin/bash
clickhouse-client
```

Давай рассмотрим основные операции в ClickHouse, включая создание таблиц, вставку данных и выполнение запросов.

##### <font color="#46aa63">Создание таблиц</font>

Для создания таблиц в ClickHouse используется SQL-подобный синтаксис. Основные элементы, которые нужно определить:

- **Имя таблицы**
- **Структура данных** (названия столбцов и их типы)
- **Движение данных** (например, использование `ENGINE` для выбора типа хранения)

**Пример создания таблицы:**

```sql
CREATE TABLE orders (
    order_id UInt32,
    customer_id UInt32,
    order_date DateTime,
    amount Float64,
    status String
) ENGINE = MergeTree()
ORDER BY order_id;
```

В этом примере:
- `MergeTree` — это тип движка, подходящий для больших объемов данных.
- `ORDER BY` определяет, по какому столбцу будет происходить сортировка данных для ускорения запросов.

##### <font color="#46aa63">Вставка данных</font>

ClickHouse поддерживает несколько способов вставки данных:

- **Batch Insert**: Позволяет вставлять несколько строк данных за один раз, что значительно ускоряет процесс.

**Пример вставки данных:**

```sql
INSERT INTO orders (order_id, customer_id, order_date, amount, status) VALUES
(1, 1001, '2023-11-01 12:00:00', 250.0, 'completed'),
(2, 1002, '2023-11-01 12:05:00', 150.0, 'pending');
```

- **Вставка из файлов**: Можно загружать данные из CSV, TSV и других форматов.

**Пример вставки из CSV файла:**

```sql
INSERT INTO orders FORMAT CSV
'1,1001,2023-11-01 12:00:00,250.0,completed
2,1002,2023-11-01 12:05:00,150.0,pending';
```

##### <font color="#46aa63">Запросы</font>

ClickHouse поддерживает широкий спектр SQL-запросов. Вот несколько примеров:

- **Простой запрос:**

```sql
SELECT * FROM orders WHERE status = 'completed';
```

- **Агрегация:**

```sql
SELECT customer_id, COUNT(*) AS order_count, SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING total_spent > 500;
```

- **Сложный запрос с объединением:**

Предположим, есть еще одна таблица `customers`:

```sql
CREATE TABLE customers (
    customer_id UInt32,
    customer_name String
) ENGINE = MergeTree()
ORDER BY customer_id;
```

Теперь мы можем объединить таблицы:

```sql
SELECT c.customer_name, COUNT(o.order_id) AS order_count
FROM customers AS c
LEFT JOIN orders AS o ON c.customer_id = o.customer_id
GROUP BY c.customer_name;
```


Создадим базу данных и таблицу для практике:

```sql
CREATE DATABASE moex  
GRANT ALL PRIVILEGES ON moex.records TO default  
  
  
CREATE TABLE moex.records (  
    board_id String,  
    trade_date Date,  
    short_name String,  
    sec_id String,  
    num_trades Nullable(UInt64),  
    value Nullable(Float64),  
    open_val Nullable(Float64),  
    low_val Nullable(Float64),  
    high_val Nullable(Float64),  
    legal_close_price Nullable(Float64),  
    wap_price Nullable(Float64),  
    close_val Nullable(Float64),  
    volume Nullable(Float64),  
    market_price_2 Nullable(Float64),  
    market_price_3 Nullable(Float64),  
    admittedquote Nullable(Float64),  
    mp_2_val_trd Nullable(Float64),  
    market_price_3_trades_value Nullable(Float64),  
    admittedvalue Nullable(Float64),  
    waval UInt32 DEFAULT 0,  
    trading_session Nullable(Float64),  
    curren_cyid String,  
    trendclspr Nullable(Float64)  
) ENGINE = MergeTree()  
    Partition by toYYYYMM(trade_date)  
    ORDER BY (trade_date, sec_id);
```

Установим пакет для работы с ClickHouse через python:

```shell
pip install clickhouse-connect
```

Опишем `client`:

```python
import clickhouse_connect  
  
client = clickhouse_connect.get_client(host='127.0.0.1', port=8123, username='default', database="moex")
```

Попробуем отправить запрос через наш `client`:

```python
from database.connection import client  
  
 
if __name__ == "__main__":  
    result = client.query('SELECT * FROM records')  
    print(result.result_rows)

# []
```

Давайте загрузим тестовые данные для нашей базы данных с MOEX. Для этого установим пакеты и напишем скрипт:

```shell
pip install loguru requests
```

```python
from datetime import datetime  
  
import requests  
from loguru import logger  
  
from database.connection import client  
  
date_from = "2015-01-01"  
  
tickers = [  
    'SBER', 'GAZP', 'LKOH', 'ROSN', 'NVTK', 'TATN', 'ALRS',  
    'CHMF', 'MTSS', 'AFLT', 'POLY', 'KZOS', 'MTLR', 'YNDX',  
    'DSKY', 'RUAL', 'SNGS', 'NLMK', 'MAGN', 'FIVE', 'SIBN',  
    'IRAO', 'TRMK', 'BANE', 'MGTI', 'AERO', 'CHMK', 'PRTK',  
    'KAZT', 'UPRO', 'PHOR', 'BSPB', 'KMAZ', 'MGNT', 'MRKC'  
]  
  
column_names = [  
    'board_id', 'trade_date', 'short_name', 'sec_id',  
    'num_trades', 'value', 'open_val', 'low_val', 'high_val',  
    'legal_close_price', 'wap_price', 'close_val', 'volume',  
    'market_price_2', 'market_price_3', 'admittedquote', 'mp_2_val_trd',  
    'market_price_3_trades_value', 'admittedvalue', 'waval', 'trading_session',  
    'curren_cyid', 'trendclspr'  
]  
  
  
def upload_data(items: list, ticker: str):  
    for item in items:  
        item[1] = datetime.strptime(item[1], '%Y-%m-%d').date()  
        item[19] = item[19] if item[19] is not None else 0  # waval  
    try:  
        client.insert(  
            'records',  
            items,  
            column_names=column_names  
        )  
    except Exception as e:  
        logger.error(ticker)  
        logger.error(e)  
  
  
if __name__ == "__main__":  
  
    for ticker in tickers:  
        logger.success(f"Start ticker {ticker}. Date from {date_from}")  
        url = f"https://iss.moex.com/iss/history/engines/stock/markets/shares/securities/{ticker}.json?from={date_from}&start=0"  
        try:  
            response = requests.get(url=url)  
        except Exception as e:  
            logger.error(e)  
            continue  
        data = response.json()  
        records = data["history"]["data"]  
        upload_data(items=records, ticker=ticker)  
        index, total, page_size = data["history.cursor"]["data"][0]  
        total_page = total // page_size  
  
        for i in range(1, total_page + 1):  
            logger.info(f"{ticker} - Page {i}/{total_page}")  
            url = f"https://iss.moex.com/iss/history/engines/stock/markets/shares/securities/{ticker}.json?from={date_from}&start={i * 100}"  
            try:  
                response = requests.get(url=url)  
            except Exception as e:  
                logger.error(e)  
                continue  
            data = response.json()  
            records = data["history"]["data"]  
            upload_data(items=records, ticker=ticker)
```

Напишем запрос:

```sql
SELECT toYear(trade_date) AS year, sec_id, corrStable(low_val, high_val) FROM moex.records GROUP BY sec_id, year order by sec_id, year;
```

Отлично! Давай подробнее разберем каждый из этих пунктов, чтобы создать полноценное введение в RabbitMQ.


### <font color="#46aa63">RabbitMQ</font>

RabbitMQ — это брокер сообщений с открытым исходным кодом, который позволяет приложениям общаться друг с другом через обмен сообщениями. Он реализует протокол AMQP (Advanced Message Queuing Protocol) и поддерживает различные другие протоколы. RabbitMQ особенно полезен в распределенных системах, где необходимо организовать асинхронное взаимодействие между компонентами.

**Зачем использовать RabbitMQ?**
- **Надежная доставка сообщений**: RabbitMQ гарантирует, что сообщения не потеряются, даже если получатель временно недоступен.
- **Балансировка нагрузки**: Позволяет распределять задачи между несколькими потребителями, что улучшает производительность и отзывчивость системы.
- **Асинхронное взаимодействие**: Компоненты системы могут работать независимо друг от друга, что позволяет улучшить масштабируемость и уменьшить связность.

##### <font color="#46aa63">Архитектура RabbitMQ</font>

**Компоненты:**
- **Producer**: Это приложение или сервис, который отправляет сообщения в RabbitMQ. Примеры использования: веб-приложения, службы обработки данных.
- **Queue**: Очередь — это место, где сообщения хранятся до тех пор, пока их не обработает потребитель. Очереди могут быть долговечными, чтобы сохранять сообщения даже после перезапуска RabbitMQ.
- **Consumer**: Приложение или сервис, который получает и обрабатывает сообщения из очереди. Это может быть любой сервис, который выполняет задачи, связанные с обработкой данных.
- **Exchange**: Компонент, который определяет, как сообщения направляются в очереди. Он принимает сообщения от продюсеров и маршрутизирует их в очереди на основе правил маршрутизации.

**Типы Exchange:**
- **Direct**: Сообщения направляются в очереди, соответствующие конкретному маршруту.
- **Fanout**: Сообщения отправляются во все очереди, привязанные к exchange, без учета маршрута.
- **Topic**: Позволяет маршрутизировать сообщения по шаблонам. Это особенно полезно для сложных сценариев.
- **Headers**: Использует заголовки сообщений для маршрутизации, позволяя более сложные критерии фильтрации.

##### <font color="#46aa63">Установка и настройка</font>

**Установка:**
RabbitMQ можно установить на различных операционных системах, включая Linux, macOS и Windows. Установка может быть выполнена через пакетный менеджер (например, `apt`, `brew`) или скачиванием дистрибутива с официального сайта.

**Настройка:**
- Основные настройки хранятся в конфигурационном файле `rabbitmq.config`, где можно задать параметры, такие как обменники, очереди, пользователи и права доступа.
- RabbitMQ также предлагает веб-интерфейс для управления, что упрощает настройку и мониторинг.

##### <font color="#46aa63">Основные операции</font>

**Создание и настройка очередей:**
Пример создания очереди с параметрами:

```bash
rabbitmqadmin declare queue name=my_queue durable=true
```

**Отправка сообщений:**
Пример кода на Python с использованием библиотеки `pika` для отправки сообщения:

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='my_queue', durable=True)
channel.basic_publish(exchange='', routing_key='my_queue', body='Hello, RabbitMQ!')
print(" [x] Sent 'Hello, RabbitMQ!'")

connection.close()
```

**Получение сообщений:**
Пример кода для консюмера:

```python
import pika

def callback(ch, method, properties, body):
    print(f" [x] Received {body}")

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='my_queue', durable=True)

channel.basic_consume(queue='my_queue', on_message_callback=callback, auto_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

##### <font color="#46aa63">Примеры использования</font>

**Асинхронная обработка:**
RabbitMQ часто используется для обработки фоновых задач, таких как отправка электронных писем или обработка изображений, что позволяет веб-приложениям оставаться отзывчивыми.

**Микросервисы:**
RabbitMQ облегчает взаимодействие между микросервисами, позволяя им обмениваться сообщениями и выполнять задачи независимо друг от друга. Это улучшает устойчивость к сбоям, поскольку даже если один сервис выходит из строя, остальные продолжают работать.

### <font color="#46aa63">FastStream</font>

[**FastStream**](https://github.com/airtai/faststream) - это python-фреймворк для работы с брокерами сообщений. Он был создан для максимального упрощения разработки event-driven систем.

Если просто - это очень толстый клиент для брокеров, который позволяет вам писать меньше инфраструктурного кода и сконцентрироваться на бизнес-логике ваших приложений. Поэтому гораздо уместнее будет сравнение с **aiokafka**/**aio-pika** или **Kombu** (чьим логическим продолжением и является **FastStream**). Однако, мы предоставляем дополнительный функционал, напрямую не связанный с кодом, но крайне необходимый для современных систем: автодокументирование проекта, удобство тестирования и Observability из коробки.
##### <font color="#46aa63">Базовый API</font>

В первую очередь - это очень простой API. На данный момент **FastStream** поддерживает **Kafka**, **RabbitMQ**, **NATS** и **Redis** в качестве брокера сообщений.

Фреймворк предоставляет как унифицированный способ взаимодействия с брокерами, так и специфичные для каждого из них методы. Типичный эндпоинт для сообщений с использованием **FastStream** выглядит следующим образом:

```python
from faststream import FastStream
from faststream.rabbit import RabbitBroker
broker = RabbitBroker("amqp://guest:guest@localhost:5672/")
app = FastStream(broker)

@broker.subscriber("test-queue")  # название очереди RMQasync 
def handle(msg: str):    
	print(msg)
```

Запускается это все тоже просто:

```shell
faststream run main:app
```


Давайте опишем наше приложение которое будет работать с Rabbit.
Установим необходимые пакеты:

```shell
pip install "faststream[rabbit]" "faststream[cli]" fastapi pydantic-settings
```

Создадим папку `config` и опишем  в ней файл `settings.py`:

```python
from pathlib import Path  
  
from pydantic import BaseModel  
from pydantic_settings import BaseSettings, SettingsConfigDict  
  
  
BASE_DIR = Path(__file__).parent.parent  
  
  
class ConnectCh(BaseModel):  
    host: str = "127.0.0.1"  
    port: int = 8123  
    username: str = "default"  
    database: str = "moex"  
  
  
class Settings(BaseSettings):  
    model_config = SettingsConfigDict(  
        env_nested_delimiter='__',  
        env_file_encoding='utf-8',  
        env_file=BASE_DIR / "config" / ".env"  
    )  
  
    clickhouse: ConnectCh = ConnectCh()  
    rabbit_url: str = "amqp://guest:guest@127.0.0.1:5672/en0"  
  
  
settings = Settings()
```

В этой же папке создадим `.env`:

```env
CLICKHOUSE__HOST="195.133.30.62"  
CLICKHOUSE__PORT=8123  
CLICKHOUSE__USERNAME="default"  
CLICKHOUSE__DATABASE="moex"  
  
RABBIT_URL="amqp://guest:guest@127.0.0.1:5672/"
```

В папке `routers` создадим файл `moex.py` и опишем наш route:

```python
from fastapi import APIRouter  
  
from rabbit.publisher import publish_task_ticker  
  
moex_router = APIRouter(tags=["Moex"])  
  
@moex_router.get("/{ticker}")  
async def retrieve_all_records(ticker: str) -> dict:  
    await publish_task_ticker(ticker=ticker)  
    return {"message": "the ticker has been successfully added to the queue"}
```

Опишем `serve.py` для запуска сервера:

```python
import uvicorn  
from fastapi import FastAPI  
  
  
from routers.moex import moex_router  
  
app = FastAPI()  
  
app.include_router(moex_router, prefix="/moex")  
  
  
if __name__ == "__main__":  
    uvicorn.run("serve:app", host="127.0.0.1", port=5000, reload=True)
```

Создадим папку `rabbit`, добавим файлы `publisher.py` и  `subscriber.py`. В `publisher.py` опишем следующее:

```python
from faststream.rabbit import RabbitBroker  
  
from config.settings import settings  
  
  
broker = RabbitBroker(url=settings.rabbit_url)  
  
async def publish_task_ticker(ticker: str):  
    async with broker as br:  
        await br.publish(ticker, "tasks")
```

Создадим файл `docker-compose.yaml` и опишем в нем запуск rabbit:

```yaml
version: '3.7'  
  
services:  
  rabbit:  
      image: "rabbitmq:3-management"  
      environment:  
        RABBITMQ_DEFAULT_USER: "guest"  
        RABBITMQ_DEFAULT_PASS: "guest"  
      ports:  
        - "5672:5672"  
        - "15672:15672"
```

Запустим `compose`:

```shell
docker compose up -d
```

Перейдем на вебку rabbit и создадим очередь `tasks`. Теперь запустим наш сервер и отправим запрос:

```shell
python serve.py
curl -X 'GET' 'http://127.0.0.1:5000/moex/SBER' -H 'accept: application/json'
```

Как видим в нашей очереди появился первое сообщение.
Давайте опишем `subscriber.py`:

```python
from faststream import FastStream  
from faststream.rabbit import RabbitBroker, RabbitQueue  
  
from config.settings import settings  
  
  
broker = RabbitBroker(url=settings.rabbit_url)  
app = FastStream(broker)  
  
  
@broker.subscriber(RabbitQueue(name="tasks", durable=True))  
async def task(ticker: str):  
    print(ticker)
```

И запустим `faststream`:

```shell
faststream run rabbit.subscriber:app
```

Как видим мы прочитали сообщение из очереди. 

### Резюме

В ходе занятия были изучены ключевые аспекты RabbitMQ и ClickHouse, а также их взаимосвязь. Это позволит эффективно строить системы, основанные на асинхронной обработке данных и аналитике, обеспечивая надежность, производительность и гибкость. 


### Домашнее задание: Интеграция RabbitMQ, ClickHouse и FastAPI

#### Цель задания
Создать простое приложение на FastAPI, которое будет использовать RabbitMQ для асинхронной обработки сообщений и ClickHouse для хранения и анализа данных.

#### Задание
1. **Установите необходимые библиотеки**
2. **Настройка RabbitMQ**:
   - Установите RabbitMQ и убедитесь, что он запущен.
   - Создайте очередь для обработки сообщений, например, `data_queue`.
3. **Создайте FastAPI приложение**:
   - Создайте файл `main.py` и реализуйте следующие функции:
     - **Endpoint для отправки данных**:
       - Создайте POST-метод `/send`, который принимает JSON-данные и отправляет их в очередь RabbitMQ.
     - **Функция для обработки сообщений**:
       - Создайте функцию, которая будет извлекать сообщения из очереди и отправлять их в ClickHouse для хранения.
     - **Endpoint для получения данных**:
       - Реализуйте GET-метод `/data`, который будет извлекать данные из ClickHouse и возвращать их в формате JSON.
4. **Тестирование**:
   - Используйте Postman или curl для отправки POST-запроса на `/send` с JSON-данными.
   - Запустите функцию обработки сообщений в отдельном потоке или процессе, чтобы она постоянно обрабатывала новые сообщения.
   - Проверьте, что данные успешно сохраняются в ClickHouse, и используйте GET-запрос на `/data` для их получения.
