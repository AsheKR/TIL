# Model Meta options



## Available Meta options



### abstract

#### Options.abstract

True시 상속가능한 Base Class가 된다.

이는 실제 테이블을 생성할 수 없는 클래스이다.

---

### app_label

#### Options.app_label

모델이 **INSTALLED_APPS** 외부에 정의된 경우 해당 모델이 속한 앱을 선언해야한다.

```python
app_label = 'myapp'
```

**app_label.object_name** 또는 **app_label.model_name**형식으로 모델을 나타내려면 **model._meta_label** 또는 **model._meta.label_lower**를 사용할 수 있다.

---

### base_manager_name

#### Options.base_manager_name

**_base_mangaer**의 이름

---

### db_table

#### Options.db_table

모델에서 사용할 데이터베이스의 이름

```python
db_table = 'music_album'
```

#### Table names

시간을 절약하기위해 Django는 모델 클래스의 이름과 그를 포함하고 있는 앱에서 데이터베이스 테이블의 이름을 지정한다. 모델의 데이터베이스 테이블 이름을 모델의 "app_label"(**manage.py startapp**에서 사용한 이름)와 모델의 클래스 이름에 밑줄을 붙여 구성한다.

예로 **bookstore**(**manage.py startapp bookstore**에서 생성)앱을 사용하는 경우 **Book**클래스로 정의된 모델에서는 **bookstore_book**이라는 데이터베이스 테이블을 생성한다.

이 이름은 SQL 예약어이거나 Python 변수 이름에 허용하지 않는 문자를 포함하지 않으면 허용한다.

---

### db_tablespace

#### Options.db_tablespace

이 모델에 사용할 **database tablespace**의 이름을 지정한다. 기본값은 프로젝트의 **DEFAULT_TABLESPACE**설정이다. 백엔드가 tablespace를 지원하지 않으면 이 옵션은 무시된다.

> 테이블스페이스란?
>
> 데이터베이스 오브젝트 내 실제 데이터를 저장하는 공간이다. 이는 데이터베이스의 물리적 부분이며, 세그먼트로 관리되는 모든 DBMS에 대해 저장소(세그먼트)를 할당한다.
>
> 테이블스페이스는 단지 데이터베이스 저장소 위치를 지정할 뿐이며, 논리적 데이터베이스 구조나 스키마를 지정하지 않는다. 예를 들면, 동일한 스키마내의 다른 오브젝트는 서로 다른 테이블스페이스에 놓일 수 있다. 마찬가지로, 하나의 테이블스페이스는 여러 세그먼트들을 서비스 할 수 있다.

---

### default_manager_name

#### Options.default_manager_name

현재 모델의 **_default_manager**의 이름을 나타낸다.

---

### default_related_name

#### Options.default_related_name

역참조시 사용될 이름을 지정한다. 기본적으로 **<model_name>_set.**으로 사용된다.

이 이름은 고유해야하므로 여러곳에서 참조시 문제가 발생할 수 있다. 이 충돌을 해결하기위해 이름의 일부에 모델이 있는 응용 프로그램의 이름과 모델 이름으로 각각 대체되는 **(app_label)s** 및 **(model_name)s**을 이용할 수 있다.

---

### get_latest_by

#### Options.get_latest_by

**Manager**로 **latest()**나 **earliest()**에 사용될 기본 필드를 지정한다.

```python
# Latest by ascending order_date.
get_latest_by = "order_date"

# Latest by priority descending, order_date ascending.
get_latest_by = ['-priority', 'order_date']
```

---

### managed

#### Options.managed

기본 값은 **True**, 이 값으로 데이터베이스에 관리되는지의 여부를 정한다.

이 값이 **False**이면 데이터베이스 테이블 생성 또는 삭제 작업이 수행되지 않는다.

1. 선언하지 않은 경우 모델에 자동 키 필드 추가. 나중에 읽는사람에게 혼동하지 않게하려면 관리되지 않는 모든 모델의 열에 명시하는것이 좋다.

2. **managed=False**의 경우 **ManyToManyField**가 포함되어 있는 다대다 조인을 위한 테이블도 생성하지 않는다. 대신 하나의 관리되는 모델과 관리되지 않는 모델 사이의 중간 테이블이 생성된다.

   이 기본 동작을 변경해야하는 경우 중간 테이블을 명시적 모델로 필요에 따라 관리되는 집합으로 **ManyToManyField.through**특성을 사용하여 관계가 사용자 지정 모델을 사용하도록한다.

**managed=False** 모델을 포함하는 테스트의 경우 테스트 설정의 일부로 올바른 테이블이 만들어지도록 설정해야한다.

모델 클래스의 파이썬 레벨 동작을 변경하는데 관심이 있다면 **managed=False**를 사용하여 기존 모델의 복사본을 만들 수 있다. 그러나 이러한 상황에 대한 더 나은 접근 방법이 있다. **Proxy models**

---

### order_with_respect_to

#### Options.order_with_respect_to

**ForeignKey**에 대해 순서를 가질 수 있게 한다. 이는 부모 객체와 관련 객체를 정렬할 수 있게하는데 사용할 수 있다.

```python
from django.db import models

class Question(models.Model):
    text = models.TextField()
    # ...

class Answer(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    # ...

    class Meta:
        order_with_respect_to = 'question'
```

**order_with_respect_to**가 설정되면 관련 객체의 순서를 가져오고 설정하는 두가지 메서드가 제공된다.

**get_RELATED_order()**, **set_RELATED_order()** 여기서 RELATED는 소문자로된 모델의 이름이다. 예를 들어, **Question**객체에 여러 개의 관련 **Answer** 객체가 있다면 가정한다면 반환되는 목록에는 관련 **Answer** 객체의 기본 키가 포함된다.

```python
>>> question = Question.objects.get(id=1)
>>> question.get_answer_order()
[1, 2, 3]
```

**Quertsion**객체의 관련 **Answer**객체 순서는 **Answer** 기본 키 목록을 전달하여 설정할 수 있다.

```python
>>> question.set_answer_order([3, 1, 2])
```

관련 객체에는 **get_next_in_order()** 및 **get_previous_in_order()** 두 가지의 메서드도 있다. 이 메서드는 올바른 순서로 객체에 액세스하는데 사용될 수 있다.

```python
>>> answer = Answer.objects.get(id=2)
>>> answer.get_next_in_order()
<Answer: 3>
>>> answer.get_previous_in_order()
<Answer: 1>
```

> **order_with_respect_to**는 **ordering**과 함께 사용할 수 없다.

> **order_with_respect_to**가 새 데이터베이스 열을 추가하기 때문에 초기 마이그레이션 후 **order_with_respect_to**를 추가하거나 변경하면 적절한 마이그레이션을 만들어 적용한다.

---

### ordering

#### Options.ordering

객체의 기본순서는 객체 목록을 가져올 때 사용한다.

```python
ordering = ['-order_date']
```

튜플 혹은 문자열 및 쿼리식을 사용한다. 내림차순을 나타내는 것은 `-`접두사를 사용한다. 무작위 정렬을 위해서는 `?` 문자열을 사용한다.

```python
from django.db.models import F

ordering = [F('author').asc(nulls_last=True)]
```

기본 순서는 집계 쿼리에도 영향을 준다.

> ordering은 데이터베이스 비용을 발생시킨다. 
>
> 만약 쿼리에 ordering이 존재하지 않는다면 데이터베이스에서 명시되지않은 순서로 반환된다. 

---

### permissions

#### Options.permissions

이 개체를 만들 때 권한 테이블에 입력할 수 있는 추가 권한, 각 모델에 대해 추가, 변경, 삭제 및 보기 권한이 자동으로 생긴다.

```python
permissions = (("can_deliver_pizzas", "Can deliver pizzas"),)
```

이는 **(permission_code, human_readable_permission_name)**의튜플 형태의 목록을 가진다.

---

### default_permissions

#### Options.default_permissions

기본은 **('add', 'change', 'delete', 'view')**이다. 이 목록을 사용자 정의할 수 있다. 기본 사용권한이 필요하지 않은 경우 빈 목록을 설정하여 이 목록을 사용자지정 가능하다. 누락된 사용 권한이 생성되지 않도록 모델을 생성하기 전 모델에 지정해야한다.

---

### proxy

#### Options.proxy

다른 모델의 **proxy model**이된다.

---

### required_db_features

#### Options.required_db_features

마이그레이션 단계에서 모델을 고려하기 위해 현재 연결해야 하는 데이터베이스의 기능 목록

예를 들어 이 목록을 **['Gis_enabled']**로 설정하면 모델은 **GIS** 지원 데이터베이스에만 동기화된다. 여러 데이터베이스 백엔드로 테스트할 때 일부 모델을 건너뛰는것도 유용하다. **ORM**이 이 문제를 처리하지 않으므로 생성되거나 생성되지 않을 수 있는 모델 간의 관계를 피하도록한다.

---

### required_db_vendor

#### Options.requred_db_vendor

지원되는 데이터에비스의 벤더 이름은 이 모델에 한정된다.

현재는 **['sqlite', 'postgresql', 'mysql', 'oracle']**이 있다. 이 특성과 현재 연결된 데이터베이스가 일치하지 않은 경우 모델을 동기화하지 않는다.

---

### select_on_save

#### Options.select_on_save

장고가 1.6 이전의 **django.db.models.Model.save()** 알고리즘을 사용할지를 결정한다.

---

### indexes

#### Options.indexes

모델에 인덱스를 설정할 수 있다.

```python
from django.db import models

class Customer(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    class Meta:
        indexes = [
            models.Index(fields=['last_name', 'first_name']),
            models.Index(fields=['first_name'], name='first_name_idx'),
        ]
```

---

### unique_together

#### Options.unique_together

여러개의 **unique**필드를 지정할 수 있도록해주는 옵션이다. 이는 데이터베이스 수준에서 작동한다.

```python
unique_together = (("driver", "restaurant"),)
```

**ManyToManyField**는 **unique_together**에 포함될 수 없다. **MTM**와 관련된 고유성의 유효성을 검사해야하는 경우 **signal**이나 **through** 모델을 통하여 사용하라.

---

### index_togther

#### Options.index_togther

```python
index_together = [
    ["pub_date", "deadline"],
]
```

이 필드 목록은 함께 색이된다.

---

### verbose_name

#### Options.verbose_name

사람일 읽을 수 있는 이름을 정한다.

```python
verbose_name = 'pizza'
```

---

### verbose_name_plural

#### Options.verbose_name_plural

```python
verbose_name_plural = "stories"
```

복수형을 나타낸다. 주어지지않으면 **verbose_name + "s"**가 적용된다.

---

## Read-only **Meta** attributes

### label

#### Options.label

기본으로 객체의 표현은 **app_label.object_name**을 반환한다.

---

### label_lower

#### Options.label_lower

모델의 표현은 **app_label.model_name**을 반환한다.