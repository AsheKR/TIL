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

------

#### annotate()

**annotate(\*args, \*\*kwargs)**

query expressions로 쿼리에 새 요소를 추가한다. expression은 단순 값, 모델의 필드에 대한 참조(또는 모든 관련 모델), 또는 집계식(평균, 합계), **Queryset**의 객체와 관련된 객체에 대해 계산된 값이다.

**QuerySet**의 각 객체에 새 요소가 추가되어 반환된다.

------

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

------

#### reverse()

**reverse()**

쿼리셋을 반대순서로 만든다.

마지막 5개를 가져오고 싶다면 다음을 사용한다.

```
my_queryset.reverse()[:5]
```

파이썬의 마지막에서부터 가져오는 슬라이싱인 **seq[-5:]**는 SQL에서 불가능하기때문에 지원하지 않는다.

또한 보통 ordering이 정의되어있는 **QuerySet**에서 호출해야한다. ordering이 정의되지 않은 **QuerySet**에 **reverse()**를 하게 되면 실제 효과는 없다.

------

