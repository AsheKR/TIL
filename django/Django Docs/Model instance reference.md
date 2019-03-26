# Model instance reference

모델 API의 세부사항을 설명한다.



## Creating objects

모델의 새 인스턴스를 생성하기위해 다른 Python 클래스처럼 인스턴스를 생성한다.

**class Model(\*\*kwargs)**

키워드 인수는 단순히 모델에 정의한 필드의 이름이다. 모델을 인스턴스화하는 것은 결코 데이터베이스를 건드리지 않는다. 저장을 위해 **save()**를 해야한다.

> **\_\_init\_\_** 메서드를 오버라이드하여 모델을 커스터마이징할 수 있다. 그러나 모델 인스턴스의 변경사항이 저장되지 않을 수 있다. **\_\_init\_\_**을 오버라이드하는 대신 다음 방법중 하나를 사용한다.
>
> 1. 모델 클래스에 classmethod를 추가한다.
>
> ```python
> from django.db import models
> 
> class Book(models.Model):
>     title = models.CharField(max_length=100)
> 
>     @classmethod
>     def create(cls, title):
>         book = cls(title=title)
>         # do something with the book
>         return book
> 
> book = Book.create("Pride and Prejudice")
> ```
>
> 2. 맞춤 관리자에 메서드를 추가하자. 
>
> ```python
> class BookManager(models.Manager):
>     def create_book(self, title):
>         book = self.create(title=title)
>         # do something with the book
>         return book
> 
> class Book(models.Model):
>     title = models.CharField(max_length=100)
> 
>     objects = BookManager()
> 
> book = Book.objects.create_book("Pride and Prejudice")
> ```

### Customizing model loading

**classmethod Model.from_db(db, field_names, values)**

**from_db()** 메서드를 사용하여 데이터베이스에서 로드 할 때 모델 인스턴스 작성을 사용자 정의를 할 수 있다.

**db**인수는 모델이 로드 된 데이터베이스의 데이터베이스의 별명을 포함하고, **field_names**는 로드된 모든 필드의 이름을 포함하고 **values**에 **field_names**의 각 필드에 로드 된 값을 포함한다. **values**와 **field_names**의 값은 같은 순서이다. 모델의 모든 필드가 존재하면 값은 **\_\_init\_\_()**이 기대하는 순서로 보장된다. 인스턴스는 **cls(\*values)**에 의해 생성될 수 있다. 필드가 지연되면 **field_names**에 표시되지 않는다.

새 모델을 만드는 것 외에 **from_db()**메서드는 새 인스턴스의 **_state**속성에 추가 및 **db**플래그를 설정해야한다.

데이터베이스에서 로드 된 필드의 초기 값을 기록하는 방법을 보여주는 예이다.

```python
from django.db.models import DEFERRED

@classmethod
def from_db(cls, db, field_names, values):
    # Default implementation of from_db() (subject to change and could
    # be replaced with super()).
    if len(values) != len(cls._meta.concrete_fields):
        values = list(values)
        values.reverse()
        values = [
            values.pop() if f.attname in field_names else DEFERRED
            for f in cls._meta.concrete_fields
        ]
    instance = cls(*values)
    instance._state.adding = False
    instance._state.db = db
    # customization to store the original field values on the instance
    instance._loaded_values = dict(zip(field_names, values))
    return instance

def save(self, *args, **kwargs):
    # Check how the current values differ from ._loaded_values. For example,
    # prevent changing the creator_id of the model. (This example doesn't
    # support cases where 'creator_id' is deferred).
    if not self._state.adding and (
            self.creator_id != self._loaded_values['creator_id']):
        raise ValueError("Updating the value of creator isn't allowed")
    super().save(*args, **kwargs)
```

위 예제는 전체 **from_db()** 구현을 보여준다. 이 경우 **from_db()** 메서드에서 **super()** 호출만 사용하는 것이 가능하다.



## Refreshing objects from database

모델 인스턴스에서 필드를 삭제하면 다시 액세스하여 데이터베이스에서 값을 다시 로드한다.

```python
>>> obj = MyModel.objects.first()
>>> del obj.field
>>> obj.field  # Loads the field from the database
```

**Model.refresh_from_db(using=None, fields=None)**

모델 값을 데이터베이스에서 다시 로드해야하는 경우 **refresh_from_db()**메서드를 사용할 수 있습니다. 인수없이 이 메서드를 호출하면 다음 작업이 수행된다.

1. 모델의 모든 지연되지 않은 필드는 현재 데이터베이스에 있는 값으로 업데이트된다.
2. 캐시 된 관계는 다시 로드 된 인스턴스에서 지워진다.

모델의 필드만 데이터베이스에서 다시 로드된다. 주석과 같은 다른 데이터베이스 종속값은 다시 로드되지 않는다. 모든 **@cached_property**속성은 지워지지 않는다.

재로드는 인스턴스가 로드 된 데이터베이스 또는 인스턴스가 데이터베이스에서 로드되지 않은 경우 기본 데이터베이스에서 발생한다. **using**인수는 재로드에 사용되는 데이터베이스를 강제로 사용하는 데 사용될 수 있다.

**fields** 인수를 사용하여 로드 할 필드 셋을 강제 실행할 수 있다. 

예를 들어 **update()** 호출이 예상되는 업데이트를 일으킨다는 것을 테스트할 수 있다.

```python
def test_update_result(self):
    obj = MyModel.objects.create(val=1)
    MyModel.objects.filter(pk=obj.pk).update(val=F('val') + 1)
    # At this point obj.val is still 1, but the value in the database
    # was updated to 2. The object's updated value needs to be reloaded
    # from the database.
    obj.refresh_from_db()
    self.assertEqual(obj.val, 2)
```

지연된 필드에 액세스 할 때 지연된 필드의 값 로드는 메서드를 통해 발생한다. 따라서 지연로드가 발생하는 방식을 사용자 정의할 수 있다. 아래 예제는 지연 필드가 다시 로드 될 때 인스턴스의 모든 필드를 다시 로드하는 방법을 보여준다.

```python
class ExampleModel(models.Model):
    def refresh_from_db(self, using=None, fields=None, **kwargs):
        # fields contains the name of the deferred field to be
        # loaded.
        if fields is not None:
            fields = set(fields)
            deferred_fields = self.get_deferred_fields()
            # If any deferred field is going to be loaded
            if fields.intersection(deferred_fields):
                # then load all of them
                fields = fields.union(deferred_fields)
        super().refresh_from_db(using, fields, **kwargs)
```

**Model.get_deferred_fields()**

이 모델에 연기됟고있는 모든 필드의 속성 명을 포함한 세트를 반환하는 헬퍼 메서드이다.



## Validating Objects

모델 유효성 검사에는 세가지 단계가 있다.

1. 모델 필드의 유효성 검사 - **Model.clean_fields()**
2. 모델 전체 유효성 검사 - **Model.clean()**
3. 필드 고유성 검증 - **Model.validate_unique()**

모델의 **full_clean()**을 호출하면 위 세단계가 모두 수행된다.

**ModelForm**을 사용할 때 **is_valid()** 호출은 양식에 포함된 모든 필드에 대해 이러한 검증 단계를 수행한다. 검증 오류를 직접 처리하려는 경우 **full_clean()**메서드를 호출하면 된다.

**Model.full_clean(exclude=None, validate_unique=True)**

이 메서드는 **Model.clean_fields()**, **Model.clean()** 및 **Model.validate_unique()**를 이 순서로 호출하고 세 단계의 오류가 있는 **message_dict** 특성을 가진 **ValidationError**를 발생한다.

**exclude** 인수는 validation과 cleaning에서 제외할 수 있는 필드 이름 목록을 제공하는 데 사용할 수 있다. **ModelForm**은 사용자가 오류를 수정할 수 없으므로 양식에 없는 필드를 검증 대상에서 제외하기 위해 이 인수를 사용한다.

모델의 **save()**메서드를 호출할 때 **full_clean()**이 자동으로 호출되지 않는다. 수동으로 생성한 모델에 대해 원스텝 검증을 실행하려면 수동으로 호출해야한다.

```python
from django.core.exceptions import ValidationError
try:
    article.full_clean()
except ValidationError as e:
    # Do something based on the errors contained in e.message_dict.
    # Display them to a user, or handle them programmatically.
    pass
```

**full_clean()**의 첫 단계는 개별 필드를 정리한다.

**Model.clean_fields(exclude=None)**

이 메서드는 모델의 모든 필드의 유효성을 검사한다. 선택적인수 **exclude**인수를 사용하여 유효성 검증에서 제외할 필드 이름 목록을 제공할 수 있다. 유효성 검사에 실패한 필드가 있다면 **ValidationError**를 발생시킨다.

**full_clean()** 두 번째 단계는 **Model.clean()**을 호출하는 것이다. 모델에서 사용자 정의 유효성 검사를 수행하려면 이 메서드를 재정의해야한다.

**Model.clean()**

이 방법은 사용자 정의 모델 검증을 제공하고 원하는 경우 모델의 특성을 수정하는 데 사용해야 한다. 예를 들어, 필드를 사용하여 필드의 값을 자동으로 제공하거나, 한 개 이상의 필드에 액세스해야 하는 유효성 검사를 수행할 수 있다.

```python
import datetime
from django.core.exceptions import ValidationError
from django.db import models
from django.utils.translation import gettext_lazy as _

class Article(models.Model):
    ...
    def clean(self):
        # Don't allow draft entries to have a pub_date.
        if self.status == 'draft' and self.pub_date is not None:
            raise ValidationError(_('Draft entries may not have a publication date.'))
        # Set the pub_date for published items if it hasn't been set already.
        if self.status == 'published' and self.pub_date is None:
            self.pub_date = datetime.date.today()
```

그러나 **Model.full_clean()** 처럼 모델의 **save()**메서드를 호출 할 때 모델의 **clean()**메서드가 호출되지 않는다.

다음의 예에서 **Model.clean()**에서 제기한 **ValidationError** 예외는 문자열로 인스턴스화되었기 때문에 특수 오류 키인 **NON_FIELD_ERRORS**에 저장된다. 이 키는 특정 필드가 아닌 전체 모델에 연결된 오류에 사용된다.

```python
from django.core.exceptions import NON_FIELD_ERRORS, ValidationError
try:
    article.full_clean()
except ValidationError as e:
    non_field_errors = e.message_dict[NON_FIELD_ERRORS]
```

특정 필드에 예외를 지정하려면 **Dictionary**를 사용하여 **ValidationError**를 인스턴스화한다. 여기서는 키는 필드 이름이다. 이전 예를 업데이트하여 **pub_date** 필드에 오류를 할당할 수 있다.

```python
class Article(models.Model):
    ...
    def clean(self):
        # Don't allow draft entries to have a pub_date.
        if self.status == 'draft' and self.pub_date is not None:
            raise ValidationError({'pub_date': _('Draft entries may not have a publication date.')})
        ...
```

**Model.clean()**중 여러 필드에서 오류를 감지하면 사전에 필드 이름 매핑을 오류에 전달할 수도 있다.

```python
raise ValidationError({
    'title': ValidationError(_('Missing title.'), code='required'),
    'pub_date': ValidationError(_('Invalid date.'), code='invalid'),
})
```

**Model.validate_unique(exclude=None)**

**clean_fields()**와 유사하지만, 개별 필드 값 대신 모델의 모든 고유 제약 조건을 검증한다. 선택적 제외 인수를 사용하면 검증에 제외할 필드 이름 목록을 제공할 수 있다. 어떤 필드가 유효성 검사에 실패할 경우 **ValidationError** 를 발생시킨다.

**validate_unique()**에 **exclude**인수를 제공하면 사용자가 제공한 필드 중 하나와 관련된 **unique_together** 제약조건이 검사되지 않는다.



## Saving objects

객체를 데이터베이스에 저장하려면 **save()**를 호출한다.

**Model.save(force_insert=False, force_update=False, using=DEFAULT_DB_ALIAS, update_fields=None)**

사용자 정의 저장 동작을 원하면 **save()** 메서드를 대체 할 수 있다.

모델 저장 프로세스에는 또한 약간의 미묘함이 있다. 아래 섹션에 나온다.

### Auto-incrementing primary keys

모델에 **AutoField**가 있는 경우 자동 증가 값이 계산되어 **save()**를 처음 호출할 때 객체에 특성으로 저장된다.

```python
>>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b2.id     # Returns None, because b doesn't have an ID yet.
>>> b2.save()
>>> b2.id     # Returns the ID of your new object.
```

데이터베이스에서 값을 계산하기 때문에 **save()**를 호출하기 전에 **ID** 값이 무엇인지 알 수 있는 방법이 없다.

모델의 필드에서 **primary_key = True**를 명시적으로 지정하지 않으면 편의상 각 모델에는 기본적으로 **AutoField**라는 **id**가 있다. 자세한 내용은 **AutoField** 설명서를 참고한다.

#### The pk property

**Model.pk**

당신이 직접 primary-key 키 필드를 정의하든, 아니면 Django가 당신을 위해 primary-key 키 필드를 공급하게 하든지 사아관없이 각 모델은 `pk`라는 속성을 갖게 될 것이다. 이는 모델에서 일반적인 속성처럼 동작하지만 실제 모델에서 기본 키 필드가 되는 속성에 대한 별칭이다. 이는 다른 속성과 마찬가지로 읽고 설정할 수 있으며, 모델에서 올바른 필드가 업데이트 된다.

---

#### Explicity specifying auto-primary-key values

모델에 **AutoField**가 있지만 저장시 명시적으로 새 객체의 ID를 정의하려는 경우 **ID** 자동 할당에 의존하기보다는 저장하기 전 모델을 명시적으로 정의한다.

```python
>>> b3 = Blog(id=3, name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b3.id     # Returns 3.
>>> b3.save()
>>> b3.id     # Returns 3.
```

자동 기본 키 값을 수동으로 할당할 경우 이미 존재하는 기본 키 값을 사용하지 마라. 데이터베이스에 이미 존재하는 명시적인 primary-key 키 값을 가진 새로운 객체를 만들 경우, Django는 새 레코드를 만드는 것이 아니라 기존 레코드를 변경하는 것으로 가정할 것이다.

아래의 **Cheddar Talk**블로그 예제를 보면 예제는 데이터베이스의 이전 레코드를 대체한다.

```python
b4 = Blog(id=3, name='Not Cheddar', tagline='Anything but cheese.')
b4.save()  # Overrides the previous blog with ID=3!
```

기본 키 값의 업데이트를 방지하기 위해 **How Django Knows to UPDATE vs INSERT**를 확인한다.

기본 키 충돌이 발생하지 않을 것임을 확신할 때 대용량 저장 객체에 주로 유용하다.

---

### What happends when you save?

객체를 저장하면 장고는 다음과 같은 단계를 수행한다.

1. **pre-save signal**을 내보낸다. **pre_save**신호가 전송되어 해당 신호를 수신하는 모든 기능이 무언가를 수행할 수 있다.

2. 데이터를 사전 처리한다. 각 필드의 **pre_save()**메서드는 자동화 된 데이터 수정을 수행하기 위해 호출된다. 예를 들어 날짜 / 시간 필드는 **auto_now_add**및 **auto_now**를 구현하기 위해 **pre_save()**를 대체한다.

3. 데이터베이스에 대한 데이터 준비를 한다. 각 필드의 **get_db_prep_save()**메서드는 현재 값을 데이터베이스에 기록할 수 있는 데이터 유형으로 제공하도록 요청받는다.

   대부분의 필드는 데이터 준비가 필요하지 않는다. 정수 및 문자열과 같은 간단한 데이터 유형은 파이썬 객체로 "쓰기 가능"상태이다. 그러나 더 복잡한 데이터 유형은 종종 약간의 수정이 필요하다.

   예를 들어 **DateField**필드는 Python **datetime** 객체를 사용하여 데이터를 저장한다. 데이터베이스는 날짜 시간 객체를 저장하지 않으므로 필드 값을 데이터베이스에 삽입하기 위해 ISO 규격 날짜를 문자열로 반환해야한다.

4. 데이터를 데이터베이스에 삽입한다. 전처리되고 준비된 데이터는 데이터베이스에 삽입하기위한 SQL문으로 구성된다.

5. **post-save signal**을 내보낸다. **post_save**신호가 전송되어 해당 신호를 수신하는 모든 기능이 어떤 작업을 수행할 수 있다.

---

### How Django knows to UPDATE vs INSERT

Django 데이터베이스 객체가 객체 생성과 변경이 같은 **save()** 메서드를 사용한다는 것을 알 수 있다. Django는 **INSERT** 또는 **UPDATE SQL**문을 사용할 필요가 있음을 추상화한다. 특히 **save()**를 호출하면 Django는 다음 알고리즘을 따른다.

- 객체의 기본 키 속성이 True로 평가되는 값으로 설정된 경우 Django는 **UPDATE**를 실행한다.
- 객체가 기본 키 속성이 설정되지 않았거나 **UPDATE**가 아무것도 업데이트하지 않았으면 Django는 **INSERT**를 실행한다.

기본 키 값이 사용되지 않는다고 보장할 수 없는 경우 새 개체를 저장할 때 기본 키 값을 명시적으로 지정하지 않도록 주의해야한다.

1.5 이전 버전에서는 장고는 기본 키 속성이 설정되었을 때 **SELECT**를 수행한다. **SELECT**가 행을 찾은 경우 Django는 **UPDATE**를 수행하고 그렇지 않으면 **INSERT**를 수행한다. 이전 알고리즘은 **UPDATE**의 경우 하나 이상의 쿼리를 생성한다. 데이터베이스에 오브젝트의 기본 키 값에 대한 행이 포함되어 있어도 행이 갱신되었음을 데이터베이스가 보고하지 않는 경우가 드물게 있다. 예를 들어 PostgreSQL ON UPDATE 트리거는 NULL을 반환한다. 이러한 경우 **select_on_save** 옵션을 True로 설정하여 이전 알고리즘으로 되돌릴 수 있다.



#### Forcing an INSERT or UPDATE

드물기는하지만 **save()**메서드가 강제로 **SQL INSERT**를 수행할 수 있어야하며 **UPDATE**로 되돌아가지 않아야한다. 또는 반대로 가능한 경우 업데이트하되, 새 행을 삽입하지 않는다. 이 경우 **force_insert=True** 또는 **force_update=True** 매개변수를 **save()** 메서드에 전달할 수 있다. 분명히, 두 매개변수를 모두 전달하는 것은 오류다. 삽입과 업데이트를 동시에할 수 없다.

이 매개변수를 사용해야하는 경우는 매우 드물다. 이 기능은 고급 사용 목적으로만 사용된다.



### Updating attributes based on existing fields

가끔 현재 값을 증가시키거나 감소시키는 것과 같이 필드에서 간단한 산술 작업을 수행해야한다. 이를 달성하는 분명한 방법은다음과 같이 수행한다.

```python
>>> product = Product.objects.get(name='Venezuelan Beaver Cheese')
>>> product.number_sold += 1
>>> product.save()
```

데이터베이스에서 검색된 이전 **number_sold**값이 10이면 값 11이 데이터베이스에 기록된다.

이 프로세스는 새로운 값을 명시적으로 할당하는 것이 아니라 원래 필드 값에 상대적 업데이트를 표시함으로 race_condition을 방지하고 약간 더 빠르고 견고하게 만들 수 있다. Django는 이러한 종류의 상대적 업데이트를 수행하기 위한 **F Expressions**를 제공한다.

```python
>>> from django.db.models import F
>>> product = Product.objects.get(name='Venezuelan Beaver Cheese')
>>> product.number_sold = F('number_sold') + 1
>>> product.save()
```

---

### Specifying which fields to save

**save()**에 키워드 인수 **update_fields**의 필드 이름 목록이 전달되면 해당 목록에 지정된 필드만 업데이트 된다. 개체 하나 또는 소수의 필드만 업데이트하려는 경우 바람직할 수 있다. 모든 모델 필드가 데이터베이스에서 업데이트되지 않도록함으로써 약간의 성능 이점이 있습니다.

```python
product.name = 'Name changed again'
product.save(update_fields=['name'])
```

**update_fields** 인수는 문자열을 포함하는 반복 가능한 모든 인수가 될 수 있다. 빈 **update_fields iterable**은 저장을 건너뛴다. 없는 값에 대해 갱신을 수행한다.

**update_fields**를 지정하면 강제로 업데이트된다.

**only()** 또는 **defer()**를 통해 가져온 모델을 저장하면 DB에서 로드 된 필드만 업데이트된다. 실제로 이 경우 자동 **update_fields**가 있다. 지연 필드 값을 할당하거나 변경하면 필드가 업데이트 된 필드에 추가된다.

---

## Deleting objects

**Model.delete(using=DEFAULT_DB_ALIAS, keep_parents=False)**

오브젝트에 대해 **SQL DELETE**를 발행한다. 이 작업은 데이터베이스의 개체만 삭제한다. 파이썬 인스턴스는 여전히 존재할것이고 여전히 필드에 데이터를 가지고 있을것이다. 메서드는 삭제된 객체의 수와 객체 유형당 삭제 수가 있는 dictionary를 반환한다.

사용자 정의 삭제 동작을 원하면 **delete()**메서드를 대체할 수 있다. 때때로 다중 테이블 상속을 사용하면 하위 모델의 데이터만 삭제할 수 있다. **keep_parents=True**를 지정하면 상위 모델의 데이터가 유지된다.

---

## Pickling objects

모델을 **pickle**할 때 현재 상태가 저장된다. 이를 **unpikle**하면 현재 데이터베이스에 있는 데이터가 아닌 피클링된 순간의 모델 인스턴스가 포함된다.

---

## Other model instance methods

몇 메서드는 특별한 목적을 가지고 있다.

### \_\_str\_\_()

**Model.\_\_str\_\_**

메서드는 객체에서 **str()**을 호출할때마다 호출된다. 장고는 여러 장소에서 **str(obj)**를 사용한다. 특히, Django 관리 사이트에 객체를 표시하고 객체를 표시할 대 템플릿에 삽입되는 값으로 사용한다. **\_\_str\_\_()**메서드에서 항상 모델을 사람이 읽을 수 있는 형식으로 표현해야한다.

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def __str__(self):
        return '%s %s' % (self.first_name, self.last_name)
```

---

### \_\_eq\_\_()

**Model.\_\_eq\_\_()**

equal 메서드는 동일한 기본 키 값 및 동일한 클래스를 가지면 인스턴스가 동일하게 되도록 정의되어있으며 기본 키가 None인 인스턴스는 다른 인스턴스와 동일하지 않다. 

프록시 모델의 경우, 클래스는 모델의 첫번째 비 프록시 부모로 정의된다.

```python
from django.db import models

class MyModel(models.Model):
    id = models.AutoField(primary_key=True)

class MyProxyModel(MyModel):
    class Meta:
        proxy = True

class MultitableInherited(MyModel):
    pass

# Primary keys compared
MyModel(id=1) == MyModel(id=1)
MyModel(id=1) != MyModel(id=2)
# Primary keys are None
MyModel(id=None) != MyModel(id=None)
# Same instance
instance = MyModel(id=None)
instance == instance
# Proxy model
MyModel(id=1) == MyProxyModel(id=1)
# Multi-table inheritance
MyModel(id=1) != MultitableInherited(id=1)
```

---

### \_\_hash\_\_()

**Model.\_\_hash\_\_()**

**\_\_hash\_\_()**방법은 인스턴스의 기본 키 값을 기반으로 한다. 그렇기때문에 기본 키 값이 없는경우 **TypeError**가 발생한다. 

---

### get\_\_absolute\_\_url()

**get\_\_absolute\_\_url()**

Django에 객체의 표준 URL을 계산하는 방법을 알려준다. HTTP를 통해 개체를 참조하는 데 사용할 수 있는 문자열을 반환해야한다.

```python
def get_absolute_url(self):
    return "/people/%i/" % self.id
```

하드코딩보단 **reverse()**함수가 더 좋은 방법이다.

```python
def get_absolute_url(self):
    from django.urls import reverse
    return reverse('people.views.details', args=[str(self.id)])
```

장고가 **get\_\_absolute\_\_url()**를 사용하는 곳은 admin_app에서 사용된다. 

---

## Extra instance methods

**save()**, **delete()**외 모델 객체에는 다음과 같은 메서드가 있을 수 있다.

**Model.get\_FOO\_display()**

**choice**필드의 값을 사람이 읽을 수 있는 값을 가져온다.

```python
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=2, choices=SHIRT_SIZES)
```

```python
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```

**Model.get\_next\_by\_FOO()**

**Model.get\_previous\_by\_FOO()**

**null=True**가 아닌 모든 **DateField** 및 **DateTimeField**에 대해 객체에는 위 두가지 메서드가 존재한다. 날짜 필드와 관련하여 다음 및 이전 객체를 반환하고 없는 경우 **DoesNotExist**예외를 발생한다.

이 두가지 방법 모두 모델의 기본 관리자를 사용하여 쿼리를 수행한다. 사용자 지정 관리자가 사용하는 필터링을 에뮬레이트해야하거나 일회성 사용자 정의 필터링을 수행하려는 경우 두 방법 모두 필드 룩업에 설명된 형식이어야하는 선택적 키워드 인수를 허용한다.

동일 날짜 값의 경우 이러한 방법은 기본 키를 두번째 선택자로 사용한다. 이는 어떠한 행도 건너뛰지 않음을 보장한다. 또한 저장되지 않은 객체는 사용할 수 없음을 의미한다.

---

## Other attributes

### DoesNotExist

이 예외는 **Queryset.get()**에서 객체를 찾을 수 없는 경우 예외를 발생한다.

Django는 각 모델 클래스의 속성으로 **DoesNotExist**예외를 제공하여 찾을 수 없는 객체의 클래스를 식별하고 **try/except**로 특정 모델 클래스를 에러핸들링 가능하게한다.