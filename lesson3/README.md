# Pydantic, Модели ответов и обработка ошибок

### План занятия

1) Pydantic
2) Ответы в FastAPI
3) Построение модели ответа
4) mypy
5) Обработка ошибок


### <font color="#46aa63">Pydantic</font>

**Pydantic** — это **быстрая и обширная библиотека для валидации и сериализации данных**. Она входит в список основных зависимостей `FastAPI`, так как они тесно связаны друг с другом.

Один из основных способов определения схемы в `Pydantic` - это использование моделей. Модели - это просто классы, которые наследуются от базовой модели и определяют поля как аннотированные атрибуты.

Вы можете рассматривать модели как аналогичные структурам в таких языках, как C, или как требования к одной конечной точке в API.

Модели имеют много общего с классами данных Python, но при разработке были учтены некоторые незначительные, но важные отличия, которые упрощают определенные рабочие процессы, связанные с проверкой, сериализацией и генерацией схемы JSON. 

Ненадежные данные могут быть переданы в модель, и после анализа и проверки `Pydantic` гарантирует, что поля результирующего экземпляра модели будут соответствовать типам полей, определенным в модели.

При определении моделей `Pydantic` в значительной степени опирается на существующие типизационные конструкции Python. Более подробно типизационные конструкции можно посмотреть на следующих ресурсах:

- https://mypy.readthedocs.io/en/latest/
- https://typing.readthedocs.io/en/latest/guides/index.html

Пример использования базовой модели:

```python
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name: str = 'Jane Doe'
```

В этом примере `User` - это модель с двумя полями:

- **id** - которое представляет собой целое число и является обязательным
- **name** - которое является строкой и не является обязательным (оно имеет значение по умолчанию).

Теперь можно создать экземпляр модели:

```python
user = User(id='123')
```

**user** - это экземпляр `User`. При инициализации объекта будут выполнены все операции синтаксического анализа и проверки. Если не возникает исключения [ValidationError](https://docs.pydantic.dev/latest/api/pydantic_core/#pydantic_core.ValidationError), вы знаете, что результирующий экземпляр модели является допустимым.

Доступ к полям модели можно получить как к обычным атрибутам пользовательского объекта:

```python
assert user.name == 'Jane Doe'  
assert user.id == 123  
assert isinstance(user.id, int)
```

Экземпляр модели может быть сериализован с помощью метода [model_dump](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_dump).

```python
assert user.model_dump() == {'id': 123, 'name': 'Jane Doe'}
```

Вызов `dict`(deprecated) для экземпляра также предоставит словарь, но вложенные поля не будут рекурсивно преобразованы в словари. `model_dump` также предоставляет множество аргументов для настройки результата сериализации.

По умолчанию модели изменчивы, и значения полей могут быть изменены с помощью присвоения атрибутов:

```python
user.id = 321
assert user.id == 321
```

##### <font color="#46aa63">Методы и свойства модели</font>

Приведенный выше пример показывает только верхушку айсберга возможностей моделей. Модели обладают следующими методами и атрибутами:

- [model_validate](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate) - Проверяет соответствие данного объекта модели Pydantic. 
- [model_validate_json](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate_json) - Проверяет данные JSON на соответствие модели Pydantic.
- [model_construct](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_construct) -  Создает модели без выполнения проверки.
- [model_dump](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_dump) -  Возвращает словарь полей и значений модели.
- [model_dump_json](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_dump_json) -  Возвращает строковое представление `model_dump` в формате JSON. 
- [model_copy](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_copy) -  Возвращает копию (по умолчанию, неполную копию) модели. 
- [model_json_schema](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_json_schema) -  Возвращает словарь json, представляющий схему JSON модели. 
- [model_fields](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_fields) -  Сопоставление между именами полей и их определениями (экземплярами [FieldInfo](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.FieldInfo)).
- [model_computed_fields](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_computed_fields) -  Сопоставление между именами вычисляемых полей и их определениями (экземплярами [ComputedFieldInfo](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.ComputedFieldInfo)).
- [model_extra](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_extra) - Дополнительные поля, заданные во время проверки.
- [model_fields_set](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_fields_set) - Набор полей, которые были явно указаны при инициализации модели.
- [model_parametrized_name](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_parametrized_name) - Вычисляет имя класса для параметризации универсальных классов.
 - [model_post_init](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_post_init) -  Выполняет дополнительные действия после инициализации модели.
- [model_rebuild](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_rebuild) - Перестраивает схему модели, которая также поддерживает построение рекурсивных универсальных моделей.

##### <font color="#46aa63">Вложенные модели</font>

Более сложные иерархические структуры данных могут быть определены с использованием самих моделей в качестве типов в аннотациях.

```python
from typing import List, Optional
from pydantic import BaseModel


class Foo(BaseModel):
    count: int
    size: Optional[float] = None


class Bar(BaseModel):
    apple: str = 'x'
    banana: str = 'y'


class Spam(BaseModel):
    foo: Foo
    bars: List[Bar]


m = Spam(foo={'count': 4}, bars=[{'apple': 'x1'}, {'apple': 'x2'}])
print(m)
"""
foo=Foo(count=4, size=None) bars=[Bar(apple='x1', banana='y'), Bar(apple='x2', banana='y')]
"""
print(m.model_dump())
"""
{
    'foo': {'count': 4, 'size': None},
    'bars': [{'apple': 'x1', 'banana': 'y'}, {'apple': 'x2', 'banana': 'y'}],
}
"""
```

Поддерживаются модели с самостоятельными ссылками:

```python
from pydantic import BaseModel


class Foo(BaseModel):
    a: int = 123
    #: The sibling of `Foo` is referenced by string
    sibling: 'Foo' = None


print(Foo())
#> a=123 sibling=None
print(Foo(sibling={'a': '321'}))
#> a=123 sibling=Foo(a=321, sibling=None)
```

Если использовать аннотацию `from __future__ import annotations`, мы можем просто ссылаться на модель по названию ее типа:

```python
from __future__ import annotations

from pydantic import BaseModel


class Foo(BaseModel):
    a: int = 123
    #: The sibling of `Foo` is referenced directly by type
    sibling: Foo = None


print(Foo())
#> a=123 sibling=None
print(Foo(sibling={'a': '321'}))
#> a=123 sibling=Foo(a=321, sibling=None)
```

##### <font color="#46aa63">Обработка ошибок</font>

`Pydantic` будет генерировать исключение `ValidationError` всякий раз, когда обнаружит ошибку в проверяемых данных.

Независимо от количества найденных ошибок будет генерироваться одно исключение, и эта ошибка проверки будет содержать информацию обо всех ошибках и о том, как они произошли.

Пример:

```python
from pydantic import BaseModel, ValidationError


class Model(BaseModel):
    list_of_ints: list[int]
    a_float: float


data = dict(
    list_of_ints=['1', 2, 'bad'],
    a_float='not a float',
)

try:
    Model(**data)
except ValidationError as e:
    print(e)
    """
    2 validation errors for Model
    list_of_ints.2
      Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='bad', input_type=str]
    a_float
      Input should be a valid number, unable to parse string as a number [type=float_parsing, input_value='not a float', input_type=str]
    """
```

##### <font color="#46aa63">Проверка данных</font>

`Pydantic` предоставляет три метода для анализа данных в классах `models`:

- [model_validate](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate) -  это очень похоже на метод __init__ в model, за исключением того, что он использует словарь или объект, а не аргументы ключевого слова. Если переданный объект не может быть проверен или если он не является словарем или экземпляром рассматриваемой модели, будет вызван `ValidationError`.
- [model_validate_json](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate_json) -  проверяются предоставленные данные в виде строки JSON или байтового объекта. Если ваши входящие данные представляют собой полезную нагрузку в формате JSON, это обычно считается более быстрым способом .
- [model_validate_strings](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate_strings) -  берет словарь (может быть вложенным) со строковыми ключами и значениями и проверяет данные в режиме JSON, чтобы указанные строки можно было привести к правильным типам.

Пример:

```python
from datetime import datetime
from typing import Optional

from pydantic import BaseModel, ValidationError


class User(BaseModel):
    id: int
    name: str = 'John Doe'
    signup_ts: Optional[datetime] = None


m = User.model_validate({'id': 123, 'name': 'James'})
print(m)
#> id=123 name='James' signup_ts=None

try:
    User.model_validate(['not', 'a', 'dict'])
except ValidationError as e:
    print(e)
    """
    1 validation error for User
      Input should be a valid dictionary or instance of User [type=model_type, input_value=['not', 'a', 'dict'], input_type=list]
    """

m = User.model_validate_json('{"id": 123, "name": "James"}')
print(m)
#> id=123 name='James' signup_ts=None

try:
    m = User.model_validate_json('{"id": 123, "name": 123}')
except ValidationError as e:
    print(e)
    """
    1 validation error for User
    name
      Input should be a valid string [type=string_type, input_value=123, input_type=int]
    """

try:
    m = User.model_validate_json('invalid JSON')
except ValidationError as e:
    print(e)
    """
    1 validation error for User
      Invalid JSON: expected value at line 1 column 1 [type=json_invalid, input_value='invalid JSON', input_type=str]
    """

m = User.model_validate_strings({'id': '123', 'name': 'James'})
print(m)
#> id=123 name='James' signup_ts=None

m = User.model_validate_strings(
    {'id': '123', 'name': 'James', 'signup_ts': '2024-04-01T12:00:00'}
)
print(m)
#> id=123 name='James' signup_ts=datetime.datetime(2024, 4, 1, 12, 0)

try:
    m = User.model_validate_strings(
        {'id': '123', 'name': 'James', 'signup_ts': '2024-04-01'}, strict=True
    )
except ValidationError as e:
    print(e)
    """
    1 validation error for User
    signup_ts
      Input should be a valid datetime, invalid datetime separator, expected `T`, `t`, `_` or space [type=datetime_parsing, input_value='2024-04-01', input_type=str]
    """
```

##### <font color="#46aa63">Fields</font>

Функция `Field` используется для настройки и добавления метаданных в поля моделей. Параметр `default` используется для определения значения по умолчанию для поля:

```python
from pydantic import BaseModel, Field


class User(BaseModel):
    name: str = Field(default='John Doe')


user = User()
print(user)
#> name='John Doe'
```

Мы также можем использовать `default_factory` для определения вызываемого объекта, который будет вызываться для генерации значения по умолчанию:

```python
from uuid import uuid4

from pydantic import BaseModel, Field


class User(BaseModel):
    id: str = Field(default_factory=lambda: uuid4().hex)
```

##### <font color="#46aa63">Field aliases</font>

Для проверки и сериализации можно определить псевдоним для поля.

Существует три способа определения псевдонима:

- `Field(..., alias='foo')`
- `Field(..., validation_alias='foo')`
- `Field(..., serialization_alias='foo')`

Параметр `alias`используется как для проверки, так и для сериализации. Если вы хотите использовать разные псевдонимы для проверки и сериализации соответственно, вы можете использовать параметры `validation_alias` и `serialization_alias`, которые будут применяться только в соответствующих случаях использования.

Вот пример использования `alias`параметра:

```python
from pydantic import BaseModel, Field


class User(BaseModel):
    name: str = Field(..., alias='username')


user = User(username='johndoe')  
print(user)
#> name='johndoe'
print(user.model_dump(by_alias=True))  
#> {'username': 'johndoe'}
```

Если мы хотим использовать псевдоним только для проверки, мы можем использовать `validation_alias`параметр:

```python
from pydantic import BaseModel, Field


class User(BaseModel):
    name: str = Field(..., validation_alias='username')


user = User(username='johndoe')  
print(user)
#> name='johndoe'
print(user.model_dump(by_alias=True))  
#> {'name': 'johndoe'}
```

Если мы хотим определить псевдоним только для сериализации , мы можем использовать `serialization_alias`параметр:

```python
from pydantic import BaseModel, Field


class User(BaseModel):
    name: str = Field(..., serialization_alias='username')


user = User(name='johndoe')  
print(user)
#> name='johndoe'
print(user.model_dump(by_alias=True))  
#> {'username': 'johndoe'}
```

В случае использования `alias`вместе с `validation_alias`или `serialization_alias`одновременно, `validation_alias`будет иметь приоритет над `alias`для проверки и `serialization_alias` будет иметь приоритет дан `alias`для сериализации.

Если мы используем `alias_generator`в [Model Config](https://docs.pydantic.dev/latest/api/config/#pydantic.config.ConfigDict.alias_generator) , мы можем контролировать порядок приоритета для указанного поля по сравнению с сгенерированными псевдонимами через `alias_priority`настройку. Больше прочитать о приоритете псевдонимов можно [здесь](https://docs.pydantic.dev/latest/concepts/alias/#alias-precedence) .

##### <font color="#46aa63">Числовые ограничения</font>

Существуют некоторые ключевые аргументы, которые можно использовать для ограничения числовых значений:

- `gt`- больше чем
- `lt`- меньше, чем
- `ge`- больше или равно
- `le`- меньше или равно
- `multiple_of`- кратное данному числу
- `allow_inf_nan`- разрешить `'inf'`, `'-inf'`, `'nan'`значения

Вот пример:

```python
from pydantic import BaseModel, Field


class Foo(BaseModel):
    positive: int = Field(gt=0)
    non_negative: int = Field(ge=0)
    negative: int = Field(lt=0)
    non_positive: int = Field(le=0)
    even: int = Field(multiple_of=2)
    love_for_pydantic: float = Field(allow_inf_nan=True)


foo = Foo(
    positive=1,
    non_negative=0,
    negative=-1,
    non_positive=0,
    even=2,
    love_for_pydantic=float('inf'),
)
print(foo)
"""
positive=1 non_negative=0 negative=-1 non_positive=0 even=2 love_for_pydantic=inf
"""
```

В случае использования ограничений полей с составными типами в некоторых случаях может возникнуть ошибка. Чтобы избежать потенциальных проблем, можно использовать `Annotated`:

```python
from typing import Optional

from typing_extensions import Annotated

from pydantic import BaseModel, Field


class Foo(BaseModel):
    positive: Optional[Annotated[int, Field(gt=0)]]
    # Can error in some cases, not recommended:
    non_negative: Optional[int] = Field(ge=0)
```

##### <font color="#46aa63">Ограничения строк</font>

Существуют поля, которые можно использовать для ограничения строк:

- `min_length`: Минимальная длина строки.
- `max_length`: Максимальная длина строки.
- `pattern`: Регулярное выражение, которому должна соответствовать строка.

Вот пример:

```python
from pydantic import BaseModel, Field


class Foo(BaseModel):
    short: str = Field(min_length=3)
    long: str = Field(max_length=10)
    regex: str = Field(pattern=r'^\d*$')  


foo = Foo(short='foo', long='foobarbaz', regex='123')
print(foo)
#> short='foo' long='foobarbaz' regex='123'
```

##### <font color="#46aa63">Десятичные ограничения</font>

Существуют поля, которые можно использовать для ограничения десятичных знаков:

- `max_digits`: Максимальное количество цифр в `Decimal`. Оно не включает ноль перед десятичной точкой или конечные десятичные нули.
- `decimal_places`: Максимально допустимое количество десятичных знаков. Не включает конечные десятичные нули.

Вот пример:

```python
from decimal import Decimal

from pydantic import BaseModel, Field


class Foo(BaseModel):
    precise: Decimal = Field(max_digits=5, decimal_places=2)


foo = Foo(precise=Decimal('123.45'))
print(foo)
#> precise=Decimal('123.45')
```

##### <font color="#46aa63">Exclude</font>

Параметр `exclude`можно использовать для управления тем, какие поля следует исключить из модели при ее экспорте.

Вот пример:

```python
from pydantic import BaseModel, Field


class User(BaseModel):
    name: str
    age: int = Field(exclude=True)


user = User(name='John', age=42)
print(user.model_dump())  
#> {'name': 'John'}
```

##### <font color="#46aa63">Декоратор computed_field​</font>

Декоратор [`computed_field`](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.computed_field)может использоваться для включения [`property`](https://docs.python.org/3/library/functions.html#property)или [`cached_property`](https://docs.python.org/3/library/functools.html#functools.cached_property)атрибутов при сериализации модели или класса данных. Свойство также будет учитываться в схеме JSON.

Свойства могут быть полезны для полей, которые вычисляются из других полей, или для полей, вычисление которых требует больших затрат (и, следовательно, кэшируется при использовании [`cached_property`](https://docs.python.org/3/library/functools.html#functools.cached_property)).

Однако следует отметить, что Pydantic не будет выполнять никакую дополнительную логику в отношении упакованного свойства (проверку, аннулирование кэша и т. д.).

Вот пример:

```python
from pydantic import BaseModel, computed_field


class Box(BaseModel):
    width: float
    height: float
    depth: float

    @computed_field
    @property  
    def volume(self) -> float:
        return self.width * self.height * self.depth


b = Box(width=1, height=2, depth=3)
print(b.model_dump())
#> {'width': 1.0, 'height': 2.0, 'depth': 3.0, 'volume': 6.0}
```

#### <font color="#46aa63">Union​</font>

`Union` принципиально отличаются от всех других типов, проверяемых `Pydantic`, вместо требования, чтобы все поля/элементы/значения были действительными,  `Union` требуют, чтобы действительным был только один член.
Это приводит к некоторым нюансам относительно того, как проверять `Union`:

- В отношении каких членов `Union` следует проверять данные и в каком порядке?
- Какие ошибки следует выдавать при неудачной проверке?

Проверка `Union` ощущается как добавление еще одного ортогонального измерения в процесс проверки.
Для решения этих проблем `Pydantic` поддерживает три основных подхода к проверке профсоюзов:

1. [left to right mode](https://docs.pydantic.dev/latest/concepts/unions/#left-to-right-mode) - режим слева направо, самый простой подход, каждый член объединения проверяется по порядку и возвращается первое совпадение
2. [smart mode](https://docs.pydantic.dev/latest/concepts/unions/#smart-mode) - интеллектуальный режим, аналогично режиму «слева направо», элементы проверяются по порядку; однако проверка будет продолжена после первого совпадения, чтобы попытаться найти лучшее совпадение; это режим по умолчанию для большинства проверок объединений
3. [discriminated unions](https://docs.pydantic.dev/latest/concepts/unions/#discriminated-unions) - дискриминационные, судят только одного члена `Union` на основании дискриминационного признака

В сложных случаях, если вы используете немаркированные `Union`, рекомендуется использовать , `union_mode='left_to_right'`если вам нужны гарантии относительно порядка попыток проверки членов `Union`.

Если вам нужно очень специализированное поведение, вы можете использовать [специальный валидатор](https://docs.pydantic.dev/latest/concepts/validators/#field-validators) .

##### <font color="#46aa63">Режим слева направо​</font>

Поскольку этот режим часто приводит к неожиданным результатам проверки, он не является режимом по умолчанию в Pydantic >=2, а `union_mode='smart'`является режимом по умолчанию.

При таком подходе проверка выполняется в отношении каждого члена `Union` в том порядке, в котором они определены, и первая успешная проверка принимается в качестве входных данных.

Если проверка не пройдена для всех членов, ошибка проверки включает ошибки всех членов союза.

`union_mode='left_to_right'`должен быть установлен как [`Field`](https://docs.pydantic.dev/latest/concepts/fields/)параметр в полях объединения, где вы хотите его использовать.

```python
from typing import Union

from pydantic import BaseModel, Field, ValidationError


class User(BaseModel):
    id: Union[str, int] = Field(union_mode='left_to_right')


print(User(id=123))
#> id=123
print(User(id='hello'))
#> id='hello'

try:
    User(id=[])
except ValidationError as e:
    print(e)
    """
    2 validation errors for User
    id.str
      Input should be a valid string [type=string_type, input_value=[], input_type=list]
    id.int
      Input should be a valid integer [type=int_type, input_value=[], input_type=list]
    """
```

В этом случае порядок членов очень важен, как показано в приведенном выше примере:

```python
from typing import Union

from pydantic import BaseModel, Field


class User(BaseModel):
    id: Union[int, str] = Field(union_mode='left_to_right')


print(User(id=123))  # 
#> id=123
print(User(id='456'))  # 
#> id=456
```

##### <font color="#46aa63">Smart режим​</font>

Из-за потенциально неожиданных результатов `union_mode='left_to_right'`в Pydantic >=2 режимом `Union`проверки по умолчанию является `union_mode='smart'`.

В этом режиме pydantic пытается выбрать наилучшее соответствие для ввода от членов союза. Точный алгоритм может меняться между второстепенными релизами Pydantic, чтобы обеспечить улучшения как производительности, так и точности.

```python
from typing import Union
from uuid import UUID

from pydantic import BaseModel


class User(BaseModel):
    id: Union[int, str, UUID]
    name: str


user_01 = User(id=123, name='John Doe')
print(user_01)
#> id=123 name='John Doe'
print(user_01.id)
#> 123
user_02 = User(id='1234', name='John Doe')
print(user_02)
#> id='1234' name='John Doe'
print(user_02.id)
#> 1234
user_03_uuid = UUID('cf57432e-809e-4353-adbd-9d5c0d733868')
user_03 = User(id=user_03_uuid, name='John Doe')
print(user_03)
#> id=UUID('cf57432e-809e-4353-adbd-9d5c0d733868') name='John Doe'
print(user_03.id)
#> cf57432e-809e-4353-adbd-9d5c0d733868
print(user_03_uuid.int)
#> 275603287559914445491632874575877060712
```


#### <font color="#46aa63">Alias​</font>

`Alias` - это альтернативное имя поля, используемое при сериализации и десериализации данных.

Указать псевдоним можно следующими способами:
- `alias`на[`Field`](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.Field)
    - должно быть`str`
- `validation_alias`на[`Field`](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.Field)
    - может быть экземпляром `str`, [`AliasPath`](https://docs.pydantic.dev/latest/api/aliases/#pydantic.aliases.AliasPath), или[`AliasChoices`](https://docs.pydantic.dev/latest/api/aliases/#pydantic.aliases.AliasChoices)
- `serialization_alias`на[`Field`](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.Field)
    - должно быть`str`
- `alias_generator`на[`Config`](https://docs.pydantic.dev/latest/api/config/#pydantic.config.ConfigDict.alias_generator)
    - может быть вызываемым или экземпляром[`AliasGenerator`](https://docs.pydantic.dev/latest/api/aliases/#pydantic.aliases.AliasGenerator)

Примеры использования `alias`, `validation_alias`, и `serialization_alias`можно посмотреть на сайте в  [разделе Псевдонимы полей](https://docs.pydantic.dev/latest/concepts/fields/#field-aliases) .

Для удобства использования Pydantic предоставляет два специальных типа `validation_alias`: `AliasPath`и `AliasChoices`.

Используется `AliasPath`для указания пути к полю с использованием псевдонимов. Например:

```python
from pydantic import BaseModel, Field, AliasPath


class User(BaseModel):
    first_name: str = Field(validation_alias=AliasPath('names', 0))
    last_name: str = Field(validation_alias=AliasPath('names', 1))

user = User.model_validate({'names': ['John', 'Doe']})  
print(user)
#> first_name='John' last_name='Doe'
```

В `'first_name'`поле мы используем псевдоним `'names'`и индекс `0`для указания пути к имени. В `'last_name'`поле мы используем псевдоним `'names'`и индекс `1`для указания пути к фамилии.

`AliasChoices`используется для указания выбора псевдонимов. Например:

```python
from pydantic import BaseModel, Field, AliasChoices


class User(BaseModel):
    first_name: str = Field(validation_alias=AliasChoices('first_name', 'fname'))
    last_name: str = Field(validation_alias=AliasChoices('last_name', 'lname'))

user = User.model_validate({'fname': 'John', 'lname': 'Doe'})  
print(user)
#> first_name='John' last_name='Doe'
user = User.model_validate({'first_name': 'John', 'lname': 'Doe'})  
print(user)
#> first_name='John' last_name='Doe'
```

Также можно использовать `AliasChoices`с `AliasPath`:

```python
from pydantic import BaseModel, Field, AliasPath, AliasChoices


class User(BaseModel):
    first_name: str = Field(validation_alias=AliasChoices('first_name', AliasPath('names', 0)))
    last_name: str = Field(validation_alias=AliasChoices('last_name', AliasPath('names', 1)))


user = User.model_validate({'first_name': 'John', 'last_name': 'Doe'})
print(user)
#> first_name='John' last_name='Doe'
user = User.model_validate({'names': ['John', 'Doe']})
print(user)
#> first_name='John' last_name='Doe'
user = User.model_validate({'names': ['John'], 'last_name': 'Doe'})
print(user)
#> first_name='John' last_name='Doe'
```

#### <font color="#46aa63">Serialization​</font>

Помимо прямого доступа к атрибутам модели через имена их полей (например, `model.foobar`), модели можно преобразовывать, выгружать, сериализовать и экспортировать несколькими способами.

`model.model_dump(...)` - основной способ преобразования модели в словарь. Подмодели будут рекурсивно преобразованы в словари.

Пример:

```python
from typing import Any, List, Optional

from pydantic import BaseModel, Field, Json


class BarModel(BaseModel):
    whatever: int


class FooBarModel(BaseModel):
    banana: Optional[float] = 1.1
    foo: str = Field(serialization_alias='foo_alias')
    bar: BarModel


m = FooBarModel(banana=3.14, foo='hello', bar={'whatever': 123})

# returns a dictionary:
print(m.model_dump())
#> {'banana': 3.14, 'foo': 'hello', 'bar': {'whatever': 123}}
print(m.model_dump(include={'foo', 'bar'}))
#> {'foo': 'hello', 'bar': {'whatever': 123}}
print(m.model_dump(exclude={'foo', 'bar'}))
#> {'banana': 3.14}
print(m.model_dump(by_alias=True))
#> {'banana': 3.14, 'foo_alias': 'hello', 'bar': {'whatever': 123}}
print(
    FooBarModel(foo='hello', bar={'whatever': 123}).model_dump(
        exclude_unset=True
    )
)
#> {'foo': 'hello', 'bar': {'whatever': 123}}
print(
    FooBarModel(banana=1.1, foo='hello', bar={'whatever': 123}).model_dump(
        exclude_defaults=True
    )
)
#> {'foo': 'hello', 'bar': {'whatever': 123}}
print(
    FooBarModel(foo='hello', bar={'whatever': 123}).model_dump(
        exclude_defaults=True
    )
)
#> {'foo': 'hello', 'bar': {'whatever': 123}}
print(
    FooBarModel(banana=None, foo='hello', bar={'whatever': 123}).model_dump(
        exclude_none=True
    )
)
#> {'foo': 'hello', 'bar': {'whatever': 123}}


class Model(BaseModel):
    x: List[Json[Any]]


print(Model(x=['{"a": 1}', '[1, 2]']).model_dump())
#> {'x': [{'a': 1}, [1, 2]]}
print(Model(x=['{"a": 1}', '[1, 2]']).model_dump(round_trip=True))
#> {'x': ['{"a":1}', '[1,2]']}
```

Метод `.model_dump_json()`сериализует модель непосредственно в строку в формате JSON, которая эквивалентна результату, полученному с помощью [`.model_dump()`](https://docs.pydantic.dev/latest/concepts/serialization/#modelmodel_dump).

```python
from datetime import datetime

from pydantic import BaseModel


class BarModel(BaseModel):
    whatever: int


class FooBarModel(BaseModel):
    foo: datetime
    bar: BarModel


m = FooBarModel(foo=datetime(2032, 6, 1, 12, 13, 14), bar={'whatever': 123})
print(m.model_dump_json())
#> {"foo":"2032-06-01T12:13:14","bar":{"whatever":123}}
print(m.model_dump_json(indent=2))
"""
{
  "foo": "2032-06-01T12:13:14",
  "bar": {
    "whatever": 123
  }
}
"""
```

Модели Pydantic также можно преобразовать в словари с помощью `dict(model)`, а также можно выполнить итерацию по полям модели с помощью `for field_name, field_value in model:`. При таком подходе возвращаются необработанные значения полей, поэтому подмодели не будут преобразованы в словари.

Пример:

```python
from pydantic import BaseModel


class BarModel(BaseModel):
    whatever: int


class FooBarModel(BaseModel):
    banana: float
    foo: str
    bar: BarModel


m = FooBarModel(banana=3.14, foo='hello', bar={'whatever': 123})

print(dict(m))
#> {'banana': 3.14, 'foo': 'hello', 'bar': BarModel(whatever=123)}
for name, value in m:
    print(f'{name}: {value}')
    #> banana: 3.14
    #> foo: hello
    #> bar: whatever=123
```

#### <font color="#46aa63">Field Validator​</font>

Для того чтобы прикрепить валидатор к определенному полю модели, мы можем использовать `@field_validator`декоратор.

```python
from pydantic import (
    BaseModel,
    ValidationError,
    ValidationInfo,
    field_validator,
)


class UserModel(BaseModel):
    name: str
    id: int

    @field_validator('name')
    @classmethod
    def name_must_contain_space(cls, v: str) -> str:
        if ' ' not in v:
            raise ValueError('must contain a space')
        return v.title()

    # you can select multiple fields, or use '*' to select all fields
    @field_validator('name')
    @classmethod
    def check_alphanumeric(cls, v: str, info: ValidationInfo) -> str:
        if isinstance(v, str):
            # info.field_name is the name of the field being validated
            is_alphanumeric = v.replace(' ', '').isalnum()
            assert is_alphanumeric, f'{info.field_name} must be alphanumeric'
        return v


print(UserModel(name='John Doe', id=1))
#> name='John Doe' id=1

try:
    UserModel(name='samuel', id=1)
except ValidationError as e:
    print(e)
    """
    1 validation error for UserModel
    name
      Value error, must contain a space [type=value_error, input_value='samuel', input_type=str]
    """

try:
    UserModel(name='John Doe', id='abc')
except ValidationError as e:
    print(e)
    """
    1 validation error for UserModel
    id
      Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='abc', input_type=str]
    """

try:
    UserModel(name='John Doe!', id=1)
except ValidationError as e:
    print(e)
    """
    1 validation error for UserModel
    name
      Assertion failed, name must be alphanumeric
    assert False [type=assertion_error, input_value='John Doe!', input_type=str]
    """
```

### <font color="#46aa63">Ответы в FastAPI</font>

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


##### Задание 1: Основы Pydantic

Создайте простую модель Pydantic для представления пользователя с полями 
- **id** (целое число)
- **name** (строка)
- **email** (строка)

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel, EmailStr

class User(BaseModel):
    id: int
    name: str
    email: EmailStr
```

Пример использования:

```python
user = User(id=1, name="John Doe", email="john.doe@example.com")
print(user)
```

</details>

##### Задание 2: Валидация типов данных

Создайте модель Pydantic для представления продукта с полями 
- **name** (строка)
- **price** (положительное число)
- **in_stock** (булево значение). 

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel, PositiveFloat

class Product(BaseModel):
    name: str
    price: PositiveFloat
    in_stock: bool
```

Пример использования:

```python
product = Product(name="Laptop", price=999.99, in_stock=True)
print(product)
```

</details>

##### Задание 3: Использование вложенных моделей

Создайте модель Pydantic для представления заказа, который содержит информацию о пользователе (вложенная модель) и список продуктов (вложенные модели).

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel
from typing import List

class User(BaseModel):
    id: int
    name: str
    email: str

class Product(BaseModel):
    name: str
    price: float

class Order(BaseModel):
    user: User
    products: List[Product]

Пример использования:

```python
user = User(id=1, name="John Doe", email="john.doe@example.com")
products = [Product(name="Laptop", price=999.99), Product(name="Mouse", price=25.50)]
order = Order(user=user, products=products)
print(order)
```

</details>


##### Задание 4: Установка значений по умолчанию

Создайте модель Pydantic для представления статьи с полями 
- **title** (строка)
- **content** (строка)
- **published** (булево значение, по умолчанию False).

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel

class Article(BaseModel):
    title: str
    content: str
    published: bool = False
```

Пример использования:

```python
article = Article(title="My First Article", content="This is the content of the article.")
print(article)
```

</details>

##### Задание 5: Основной alias

Создайте модель `Person`, которая имеет поля `first_name` и `last_name`, используя alias `first` и `last`. Проверьте, что данные, переданные с `alias`, правильно преобразуются в поля модели.

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel, Field 
class Person(BaseModel): 
	first_name: str = Field(..., alias='first')
	last_name: str = Field(..., alias='last')
```

Пример использования:

```python
data = {'first': 'John', 'last': 'Doe'} 
person = Person(**data) 
print(person)
```

</details>

##### Задание 6: Кастомные валидаторы

Создайте модель Pydantic для представления пользователя с полем **password** (строка). Добавьте кастомный валидатор, который проверяет, что пароль содержит как минимум одну цифру и одну заглавную букву.

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel, validator

class User(BaseModel):
    password: str

    @validator('password')
    def password_must_contain_digit_and_uppercase(cls, v):
        if not any(char.isdigit() for char in v):
            raise ValueError('Password must contain at least one digit')
        if not any(char.isupper() for char in v):
            raise ValueError('Password must contain at least one uppercase letter')
        return v
```

Пример использования:

```python
user = User(password="Password123")
print(user)
```

</details>

##### Задание 7: Исключение простых полей

Создайте модель `Person` с полями `first_name`, `last_name` и `age`. Проверьте, как использовать `exclude` для исключения поля `age` при сериализации.

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel
class Person(BaseModel):
	first_name: str 
	last_name: str age: int
```

Пример использования:

```python
person = Person(first_name="Alice", last_name="Smith", age=30)
person_dict = person.dict(exclude={"age"}) 
print(person_dict)
```

</details>


##### Задание 8: Исключение вложенных полей

Создайте модели `Address` и `User`, где `User` содержит `Address`. Используйте `exclude` для исключения вложенного поля `city`.

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel

class Address(BaseModel):
    street: str
    city: str

class User(BaseModel):
    username: str
    address: Address
```

Пример использования:

```python
user = User(username="johndoe", address=Address(street="123 Main St", city="Anytown"))

# Исключение вложенного поля 'city'
user_dict = user.dict(exclude={"address": {"city"}})
print(user_dict)
```

</details>

##### Задание 9: Работа с перечислениями (enums)

Создайте модель Pydantic для представления задачи с полями 
- **title** (строка) 
- **status** (перечисление с возможными значениями `pending`, `in_progress`, `completed`). 

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel
from enum import Enum

class Status(str, Enum):
    pending = 'pending'
    in_progress = 'in_progress'
    completed = 'completed'

class Task(BaseModel):
    title: str
    status: Status
```

Пример использования:

```python
task = Task(title="Write report", status=Status.pending)
print(task)
```

</details>

##### Задание 10: Валидация списков

Создайте модель Pydantic для представления группы пользователей с полем **users** (список моделей пользователей). Добавьте валидатор, который проверяет, что список не пустой.

<details>

<summary>Решение</summary>

```python
from pydantic import BaseModel, validator
from typing import List

class User(BaseModel):
    id: int
    name: str

class UserGroup(BaseModel):
    users: List[User]

    @validator('users')
    def users_must_not_be_empty(cls, v):
        if len(v) == 0:
            raise ValueError('users list must not be empty')
        return v
```

Пример использования:

```python
users = [User(id=1, name="John Doe"), User(id=2, name="Jane Doe")]
user_group = UserGroup(users=users)
print(user_group)
```

</details>


##### Задание 11: API для управления заказами

**Определение модели заказа:**

Создайте модель `Order` с полями:
- `order_id` (int, уникальный идентификатор заказа)
- `customer_name` (str, имя клиента)
- `items` (list[str], список товаров)
- `total_amount` (float, общая сумма заказа)

**Создание эндпоинтов:**
1. **Создание заказа** (POST `/orders/`):
    - Принимает данные заказа и возвращает созданный заказ с `order_id`.
2. **Получение заказа по ID** (GET `/orders/{order_id}`):
    - Возвращает информацию о заказе по заданному `order_id`.
3. **Обновление заказа** (PATCH `/orders/{order_id}`):
    - Позволяет обновлять данные заказа по `order_id`.
4. **Удаление заказа** (DELETE `/orders/{order_id}`):
    - Удаляет заказ по `order_id` и возвращает сообщение об успешном удалении.

<details>

<summary>Решение</summary>

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List

app = FastAPI()

class Order(BaseModel):
    order_id: int
    customer_name: str
    items: List[str]
    total_amount: float

orders_db = {}

@app.post("/orders/", response_model=Order, status_code=201)
async def create_order(order: Order):
    orders_db[order.order_id] = order
    return order

@app.get("/orders/{order_id}", response_model=Order)
async def get_order(order_id: int):
    order = orders_db.get(order_id)
    if not order:
        raise HTTPException(status_code=404, detail="Order not found")
    return order

@app.patch("/orders/{order_id}", response_model=Order)
async def update_order(order_id: int, order: Order):
    if order_id not in orders_db:
        raise HTTPException(status_code=404, detail="Order not found")
    orders_db[order_id] = order
    return order

@app.delete("/orders/{order_id}")
async def delete_order(order_id: int):
    if order_id not in orders_db:
        raise HTTPException(status_code=404, detail="Order not found")
    del orders_db[order_id]
    return {"detail": "Order deleted"}

```

</details>