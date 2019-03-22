# QuerySet API reference



## 언제 쿼리셋이 평가되는가

Queryset은 실제로 데이터베이스 활동이 일어나지 않을때까지 평가되지 않는다.

**Queryset**은 다음 방법으로 평가 가능하다.

- **반복.** 쿼리셋은 반복가능하며 반복을 시작하는 처음에만 데이터베이스에서 값을 가져온다.

- **자르기.** 파이썬의 배열 슬라이싱문법으로 쿼리셋을 자를 수 있다. 이는 평가되지 않는다.
  - **QuerySet**은 보통 평가되지 않는 다른 **QuerySet** 을 반환하지만 배열 슬라이싱의 "step" 파라미터를통해 평가된 쿼리셋을 가져올 수 있다.
  - 평가되지 않은  **QuerySet**을 슬라이싱하면 다른 평가되지 않은 **QuerySet** 이 반환된다. 추가적인 수정은(필터를 추가하거나 정렬하거나) SQL로 잘 변환되지 않으며 명확한 의미도 갖지 않기 때문에 허용되지 않는다.
- **Pickling/Caching.** 

- **repr().** 파이썬 인터프리터로 사용될 때 편의성을 위해 평가된다.
- **len().**  
  - 실제 개체가 필요하지 않는 경우 **count()**메서드를 제공한다.
- **list().**
- **bool().**
  - **QuerySet**의 결과가 존재하는지 확인하는 경우 **exists()**를 사용하자.



### Pickling querySets

?



## QuerySet  API

**class QuerySet(model=None, query=None, using=None)**

​	**ordered**

​		**True**면 **QuerySet**이 정렬되어있음을 나타낸다.

​	**db**

​		쿼리가 실행될 때 사용할 데이터베이스



### 새 Queryset을 반환하는 메서드



#### filter()

**filter(\*\*kwargs)**

지정된 조회 매개변수와 일치하는 객체가 포함된 새 **QuerySet**을 반환한다.

여러 매개변수는 기본 SQL문에서 AND를 통해 조인된다.

더 복잡한 쿼리 (**OR**문 같은)를 해야하는 경우 **Q objects**를 사용한다.



#### exclude()

**exclude(\*\*kwargs)**

지정된 조회 매개변수와 일치하지 않는 객체가 포함된 새 **QuerySet**을 반환한다.

filter와 같이 AND로 조인되며 모든것은 **NOT()**으로 묶인다.

더 복잡한 쿼리 (**OR**문 같은)를 해야하는 경우 **Q objects**를 사용한다.

1. 하나의 exclude문

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')
```

```
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')
```

2. 연속된 exclude문

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')
```

```
SELECT ...
WHERE NOT pub_date > '2005-1-3'
AND NOT headline = 'Hello'
```

두번째 쿼리문이 더 제한적이다.

첫번째 쿼리문은 두개 모두 일치하는것들만 제거하지만 아래는 하나의 조건을 모두 지운 후 다음 조건을 만족하는 것들을 지운다.

---

#### annotate()

**annotate(\*args, \*\*kwargs)**

query expressions로 쿼리에 새 요소를 추가한다. expression은 단순 값, 모델의 필드에 대한 참조(또는 모든 관련 모델), 또는 집계식(평균, 합계), **Queryset**의 객체와 관련된 객체에 대해 계산된 값이다.

**QuerySet**의 각 객체에 새 요소가 추가되어 반환된다.

---

#### order_by()

**order_by(\*fields)**

기본적으로 반환되는 **QuerySet**은 모델의 **Meta**옵션의 **ordering**에 의해 정렬된다.**order_by()**메서드를 사용하여 쿼리셋단위로 **order_by()**를 덮어 사용할 수 있다.

**order_by()**에 여러개가 주어졌을 경우 앞의 것을 기준으로 먼저 정렬되고 앞의 것이 같은 값을 가질때 뒤의 것으로 정렬된다.

랜덤한 정렬을 위해서는 `"?"` 인자를 사용한다. 사용중인 백엔드에 따라 비싸고 느릴 수 있다.

정렬시 대소문자 구분 여부를 지정할 방법이 없다. 이와 관련하여 다음과 같이 사용할 수 있다.

```
Entry.objects.order_by(Lower('headline').desc())
```

연쇄적인 order_by는 가장 뒤에 것이 적용된다. 그리고 각각의 호출마다 데이터베이스의 비용이 발생한다.

---

#### reverse()

**reverse()**

쿼리셋을 반대순서로 만든다.

마지막 5개를 가져오고 싶다면 다음을 사용한다.

```
my_queryset.reverse()[:5]
```

파이썬의 마지막에서부터 가져오는 슬라이싱인 **seq[-5:]**는 SQL에서 불가능하기때문에 지원하지 않는다.

또한 보통 ordering이 정의되어있는 **QuerySet**에서 호출해야한다. ordering이 정의되지 않은 **QuerySet**에 **reverse()**를 하게 되면 실제 효과는 없다.

---

#### distinct()

**distinct(\*fields)**

**SELECT DISTINCT**를 사용하는 새 **QuerySet**을 반환한다.

간단한 쿼리셋에 대해서는 중복이 보통 존재하지 않지만 쿼리가 여러 테이블에 걸쳐있는 경우 **QuerySet**을 평가할 때 중복된 결과를 얻을 수 있다. 이 때 **discinct()**가 유용하다.

> **order_by()** 호출에 사용되는 모든 필드는 SQL SELECT열에 포함된다. 이는 때때로 **distinct()**와 함께 사용할 때 예상치 못한 결과를 초래할 수 있다. **realted_model에서 정렬하려할 때** 선택된 컬럼에 관련 모델의 필드가 추가되고 이는 다른 중복행을 만들고 **distinct**가 처리된다.
>
> 이는 반환 결과에 표시되지 않으므로 때로는 중복이 제거되지 않은 결과가 반환되는것처럼 보인다.
>
> 비슷하게 **values()** 쿼리를 사용하여 선택한 열을 제한하면 모든 **order_by()**에 사용된 열이 여전히 관련되어 결과의 고유성에 영향을미친다.
>
> 교훈은 **distinct()**를 사용한다면 관련 모델에 의한 정렬을 주의해야한다는 것이다. 또한 **distinct()**와 **values()**를 함께 사용할 때 **values()**호출이 아닌 필드로 정렬할 때는 주의해야한다.

PostgreSQL에서만 **DISTINCT**를 적용할 필드의 이름을 지정하기 위해 위치 인수(**\*fields**)를 전달할 수 있다. 이는 일반적인 **distinct()**와는 다르게 지정된 필드의 값들의 중복만을 비교한다.

>  필드 이름을 지정할 때 **QuerySet**에 **order_by()**를 제공해야하며 **order_by()**의 필드는 **distinct()**의 필드로 시작해야한다.
>
> ```D
> Entry.objects.order_by('blog').distinct('blog')
> ```
>
> 위의 경우 만약 Blog의 **ordering**이 **name**인경우 작동하지 않는다.
>
> **order_by**는 **blog__name**을 가리키고 **distinct**의 SQL은 **blog__id**를 가리키기 때문이다.

---

#### values()

**values(\*fields, \*\*expressions)**

모델 인스턴스가 아닌 딕셔너리 형태의 쿼리셋을 반환한다.

**values()**는 **\*fields**인수를 사용하여 **SELECT**를 제한한다. 필드를 지정하지 않을경우 사전의 해당 테이블의 모든 필드에 대한 키와 값이 들어간다.

**\*\*expressions**를 통해 **annotate()**에 전달될 수 있다.

```django
>>> from django.db.models.functions import Lower
>>> Blog.objects.values(lower_name=Lower('name'))
<QuerySet [{'lower_name': 'beatles blog'}]>
```

내장된 custom lookups 또한 가능하다.

```django
>>> from django.db.models import CharField
>>> from django.db.models.functions import Lower
>>> CharField.register_lookup(Lower)
>>> Blog.objects.values('name__lower')
<QuerySet [{'name__lower': 'beatles blog'}]>
```

집계함수가 포함된 **values()**절은 다른 인자가 보다 먼저 적용된다. 다른 값으로 **group by**를 하고싶다면 먼저 **values()**를 사용해야한다.

```django
>>> from django.db.models import Count
>>> Blog.objects.values('entry__authors', entries=Count('entry'))
<QuerySet [{'entry__authors': 1, 'entries': 20}, {'entry__authors': 1, 'entries': 13}]>
>>> Blog.objects.values('entry__authors').annotate(entries=Count('entry'))
<QuerySet [{'entry__authors': 1, 'entries': 33}]>
```

변환과 집계를 결합하여 사용하려면 명시적으로 값에 대한 키워드 인수로 두개의 **annotate()**를 사용해야한다. 변환이 관련 필드 타입에 등록된 경우 첫 **annotate()**는 생략할 수 있다.

```django
>>> from django.db.models import CharField, Count
>>> from django.db.models.functions import Lower
>>> CharField.register_lookup(Lower)
>>> Blog.objects.values('entry__authors__name__lower').annotate(entries=Count('entry'))
<QuerySet [{'entry__authors__name__lower': 'test author', 'entries': 33}]>
>>> Blog.objects.values(
...     entry__authors__name__lower=Lower('entry__authors__name')
... ).annotate(entries=Count('entry'))
<QuerySet [{'entry__authors__name__lower': 'test author', 'entries': 33}]>
>>> Blog.objects.annotate(
...     entry__authors__name__lower=Lower('entry__authors__name')
... ).values('entry__authors__name__lower').annotate(entries=Count('entry'))
<QuerySet [{'entry__authors__name__lower': 'test author', 'entries': 33}]>
```

이는 필드에서 값만 필요하고 모델 인스턴스의 객체 기능이 필요하지 않을 때 유용하다.

Django를 만든 사람은 SQL에 영향을 주는 모든 메서드를 호출하고 선택적으로 **values()**같은 출력에 영향을 미치는 메서드를 사용한다.

> ManyToMany 의 역관계에는 여러 관련 행이 존재할 수 있으므로 결과 집합에 대해 모든 조합이 반환된다.

---

#### values_list()

**values_list(\*fields, flat=False, named=False)**

**values()**와 비슷하지만 사전대신 튜플을 반환한다.

단일 필드라면 **flat** 매개변수를 전달할 수 있다. True이면 반환된 결과가 튜플이아니라 단일 값임을 의미한다.

```django
>>> Entry.objects.values_list('id').order_by('id')
<QuerySet[(1,), (2,), (3,), ...]>

>>> Entry.objects.values_list('id', flat=True).order_by('id')
<QuerySet [1, 2, 3, ...]>
```

**named=True**를 사용하면 **namedtuple()**을 반환한다.

```django
>>> Entry.objects.values_list('id', 'headline', named=True)
<QuerySet [Row(id=1, headline='First entry'), ...]>
```

**values()**와 **values_list()**는 특정 사용 사례에 대한 최적화, 즉 모델 인스턴스 생성에 대한 오버헤드 없이 데이터의 부분집합을 검색하기 위한 것이다. ?

---

#### dates()

**dates(field, kind, order='ASC')**

**QuerySet**의 내용 내 날짜로 표시가능한 것을 **datetime.date** 객체의 목록으로 평가되는 **QuerySet**을 반환한다.

**field**는 모델의 **DateField**이름이여야한다. **kind**는 **year**, **month**, **week**, **day**이여야한다.

---

#### datetimes()

**dateimtes(field_name, kin, order='ASC', tzinfo=None)**

**QuerySet**의 내용 내 날짜로 표시가능한 것을 **datetime.datetime** 객체의 목록으로 평가되는 **QuerySet**을 반환한다.

**field**는 모델의 **DateTimeField**이름이여야한다. **kind**는 **year**, **month**, **week**, **day**, **hour**, **minute**, **second** 이여야한다.

---

#### none()

**none()**

**EmptyQuerySet**을 반환한다.

```django
>>> Entry.objects.none()
<QuerySet []>
>>> from django.db.models.query import EmptyQuerySet
>>> isinstance(Entry.objects.none(), EmptyQuerySet)
True
```

---

#### all()

**all()**

현재 **QuerySet**을 복사하여 반환한다.

**Queryset**이 평가되면 일반적으로 결과를 캐시한다. 평가 이후 이전 평가된 **QuerySet**에서 **all()**을 사용하여 업데이트 된 결과를 가져올 수 있다.

---

#### union()

#### intersection()

#### difference()

---

#### select_related()

**select_related(\*fields)**

관련 객체의 데이터를 기존 쿼리에 추가하여 가져온다. 이는 복잡한 SQL문을 만들지만 성능향상에 도움이된다.

**ForeignKey** 또는 **OneToOneField** 관계의 필드가 **select_related()**에 사용될 수 있다.

**select_related**를 많은 관련 객체로 호출하고자 하는 상황이나 모든 관계를 알지 못할 때 인자없이 **select_related**를 사용하여 모든 관계를 가져올 수 있다. 이는 기본 쿼리를 복하고 실제 필요 데이터보다 많은 데이터를 반환하기 때문에 대부분의 경우 권장하지 않는다.

이전 **select_related**데이터를 지우고싶다면 **None**파라미터를 사용한다.

```django
>>> without_relations = queryset.select_related(None)
```

---

#### prefetch_related()

**prefetch_related(\*lookups)**

각 관계에 대해 별도의 조회를 수행하고 Python에서 'joining'을 수행한다. 이를 통해 **many-to-many** 및 **many-to-one** 객체를 프리패치할 수 있게 한다.

**prefetch_related()**는 기본 쿼리가 평가된 후 평가된다.

> **QuerySets**에서는 항상 다른 데이터베이스 쿼리를 암시하는 후속 체인 메서드가 이전에 캐시된 결과를 무시하고 새 데이터베이스 쿼리를 사용하여 데이터를 검색한다.

**Prefetch**객체를 이용하여 더 세밀하게 prefetch를 컨트롤할 수 있다.

**queryset**인자를 통해 사용자 정의 querysest을 제공할 수 있다. 이는 queryset의 기본 순서를 변경하는데 사용할 수 있다.

```django
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas__toppings', queryset=Toppings.objects.order_by('name')))
```

또는 **select_related()**를 호출하여 쿼리 수를 더 줄일 수  있다.

```django
>>> Pizza.objects.prefetch_related(
...     Prefetch('restaurants', queryset=Restaurant.objects.select_related('best_pizza')))
```

**to_attr**옵션을 사용하여 prefetch된 결과를 할당할 수 있다. 결과는 리스트에 직접 저장된다.

이렇게 하면 다른 **QuerySet**으로 동일한 관계를 여러번 프리패치가능하다.

```django
>>> vegetarian_pizzas = Pizza.objects.filter(vegetarian=True)
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', to_attr='menu'),
...     Prefetch('pizzas', queryset=vegetarian_pizzas, to_attr='vegetarian_menu'))
```

사용자 정의 **to_attr**로 작성된 것은 다른 조회처럼 탐색될 수 있다.

```django
>>> vegetarian_pizzas = Pizza.objects.filter(vegetarian=True)
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=vegetarian_pizzas, to_attr='vegetarian_menu'),
...     'vegetarian_menu__toppings')
```

prefetch의 결과의 모호성이 발생할 수 있으므로 **to_attr**을 사용하는 것을 권장한다.

```django
>>> queryset = Pizza.objects.filter(vegetarian=True)
>>>
>>> # Recommended:
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=queryset, to_attr='vegetarian_pizzas'))
>>> vegetarian_pizzas = restaurants[0].vegetarian_pizzas
>>>
>>> # Not recommended:
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=queryset))
>>> vegetarian_pizzas = restaurants[0].pizzas.all()
```

사용자 지정 프리패치는 **ForeignKey** 또는 **OneToOneField**같은 단일 관계에도 작동한다. 일반적으로 **select_related**를 사용하지만 사용자 지정 프리패치가 유용할 경우도 있다.

- 관련 모델에 추가 프리패치를 수행하는 **QuerySet**을 사용할 때
- 관련  객체의 서브셋만 프리패치하려할 때
- **deferred fields**같은 성능 최적화 기능을 사용하려 할 때

>  조회하려는 필드의 순서도 중요하다.
>
> ```django
> >>> prefetch_related('pizzas__toppings', Prefetch('pizzas', queryset=Pizza.objects.all()))
> ```
>
> 이전에 조회한 쿼리세트를 다시 정의하려는 경우 **ValueError**가 발생한다.
>
> ```django
> >>> prefetch_related('pizza_list__toppings', Prefetch('pizzas', to_attr='pizza_list'))
> ```
>
> **pizza_list_toppings**가 처리중일 때 **pizza_list**가 존재하지 않기 때문에 **AttributeError**가 발생한다.

---

#### extra()

---

#### defer()

**defer(\*fields)**

복잡한 모델링 상황에 모델에 많은 필드가 포함될 수 있거나 필드를 파이썬 개체로 변환하는데 많은 비용이 소요되는 필가 있을 수 있다. 처음 데이터를 가져올 때 특정 필드가 필요할지 모르는 상황에서 쿼리 세트의 결과를 사용하는 경우 Django가 데이터베이스에서 검색하지 않도록 지시할 수 있다.

deffered가 있는 쿼리세트는 여전히 모델인스턴스를 반환한다. 해당 필드에 액세스하면 지연된 각 필드가 데이터베이스에서 검색된다. **defer()**는 여러번 호출 가능하다. 각 호출은 지연 필드에 새 필드를 추가한다.

**select_related()**를 통해 로드되는 경우 관련 모델의 필드로드를 지연시킬 수 있다.

```django
Blog.objects.select_related().defer("entry__headline", "entry__body")
```

**defer()**를 취소하고싶다면 **None**을 파라미터로 작성한다.

기본키의 **defer()**는 불가하다. **select_related()**를 사용하여 관련 모델을 검색하는 경우 관련 모델로 연결하는 필드의 로드를 지연시켜야한다. 그렇지 않으면 오류가 발생한다.

---

#### only()

**only(\*fields)**

**defer()**의 반대. 연기되서는 안되는 모델의 필드를 호출한다. 거의 모든 필드가 지연될 필요가 있는 모델을 가지고 있다면 보완적 필드 셋을 지정하기 위해 **only()**를 사용하면 코드가 더 간단해질 수 있다.

**only()**가 호출될때마다 즉시 로드할 필드 집합을 바꾼다. 즉, 최종 **only()**필드만 고려한다.

**defer()**와 **only()**의 체인은 논리적으로 동작할 수 있다.

```django
# Final result is that everything except "headline" is deferred.
Entry.objects.only("headline", "body").defer("body")

# Final result loads headline and body immediately (only() replaces any
# existing set of fields).
Entry.objects.defer("body").only("headline", "body")
```

---

#### using()

**using(alias)**

이 방법은 둘 이상의 데이터베이스를 사용하는 경우 **QuerySet**을 평가할 데이터베이스를 제어하는데 사용한다.

이 메서드는 **DATABASES**의 인자만 취한다.

```django
# queries the database with the 'default' alias.
>>> Entry.objects.all()

# queries the database with the 'backup' alias
>>> Entry.objects.using('backup')
```

---

#### select_for_update()

**select_for_update(nowait=False, skip_locked=False, of=())**

트랜잭션이 끝날때까지 행을 잠그고 지원되는 데이터베이스에서 **SELECT ... FOR UPDATE**문을 생성하는 쿼리 집합을 반환한다.

```django
from django.db import transaction

entries = Entry.objects.select_for_update().filter(author=request.user)
with transaction.atomic():
    for entry in entries:
        ...
```

queryset이 평가되면 (**for entry in entries**) 모든 일치 항목은 트랜잭션 블록이 끝날때까지 잠겨진다. 즉, 다른 트랜잭션이 변경되거나 잠금을 획득할 수 없게 된다.

일반적으로 다른 트랜잭션이 선택된 행 중 하나에서 잠금을 이미 획득한 경우 잠금이 해제될 때까지 쿼리가 차단된다. **nowait=True**의 경우 비 차단된다. 충돌하는 잠금이 다른 트랜잭션에 의해 이미 획득된 경우 쿼리 집합이 평가될 때 **DatabaseError**가 발생한다.

**select_for_update(skip_locked=True)**를 대신 사용하여 잠긴 행을 무시할수도 있다. **nowait**및 **skip_locked**는 모두 상호 배타적이며 두 옵션을 모두 사용하는 **select_for_update()**를 호출하면 **ValueError**가 발생한다.

**select_for_update()**는 쿼리에서 선택한 모든 행을 잠근다. 예를 들어 **select_related()**에 지정된 관련 객체의 행의 쿼리셋의 모델은 추가로 잠겨있다. 이를 원하지 않는 경우 **select_related()**와 동일한 필드 구문을 사용하여 **select_for_update(of=())**에 잠글 관련 객체를 지정한다. **queryset**의 모델을 참조하려면 **self**를 사용한다.

nullable 관계에서는 **select_for_update()**를 사용할 수 없다.

---

#### raw()

**raw(raw_query, params=None, translations=None)**

---

### Operators that return new QuerySets

결합 된 쿼리 세트는 동일한 모델을 사용한다.

#### AND (&)

#### OR(|)



### Methods that do not return QuerySets

이는 호출시에 캐시하지 않는다.

#### get()

**get(\*\*kwargs)**

**MultipleObjectsReturned**와 **DoesNotExist** 에러를 조심하자.

쿼리셋이 한 행을 반환할것을 예상할 수 있다면 **get()**에 파라미터를 없이 사용하여 하나의 행을 가져올 수 있다.

---

#### create()

**create(\*\*kwargs)**

객체를 한번에 저장할 수 있는 메서드이다. (`save()`를 호출하지 않아도 됨)

___

#### get_or_create()

**get_or_create(defaults=None, \*\*kwargs)**

```python
try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
except Person.DoesNotExist:
    obj = Person(first_name='John', last_name='Lennon', birthday=date(1940, 10, 9))
    obj.save()
```

위의 문장은 주로 사용되기때문에 간결하게 사용하기 위해 메서드로 만들었다.

```python
obj, created = Person.objects.get_or_create(
    first_name='John',
    last_name='Lennon',
    defaults={'birthday': date(1940, 10, 9)},
)
```

**defulats** 옵션은 get을하지 못했을 때 생성시 사용되는 기본값을 나타낸다.

---

#### update_or_create()

**update_or_create(defaults=None, \*\*kwargs)**

