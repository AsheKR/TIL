# Web API performance: profiling Django REST framework

[원본](https://www.dabapps.com/blog/api-performance-profiling-django-rest-framework/)

## Profiling our views

간단한 Django REST 프레임 워크 API를 프로파일링하여 일반 Django 뷰를 사용하여 비교하는 방법을 살펴보고 성능 향상이 가장 큰 부분 파악 할 수 있는 방법을 알아본다.

여기 사용된 프로파일링 접근법은 포괄적인 성능 척도로 사용되는 것이 아니라 API의 다양한 구성 요소의 상대적인 성능에 대한 높은 수준의 개요를 제공하기 위한 것이다.

테스트 케이스의 경우 간단한 API list를 프로파일리앟ㄴ다.

```python
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('id', 'username', 'email')

class UserListView(APIView):
    def get(self, request):
        users = User.objects.all()
        serializer = UserSerializer(users)
        return Response(serializer.data)
```

우리가 관심있게봐야하는 측정요소는 다음과 같다.

- **Database lookup.** 
  - Django ORM부터 원시 데이터베이스 액세스까지 모든 것을 다룬다. 직렬화와 독립적으로 작업을 수행하기 위해 queryset 호출을 평가하기위해 **list()**에 래핑한다.

- **Django request/response cycle**
  - View 메서드가 실행되기 전이나 후 실행되는 모든 것
  - 기본 미들웨어, 요청 라우팅 및 각 요청시 발생하는 기타 핵심 매커니즘이 포함된다.
  - `request_started` 및 `request_finished` 신호에 연결하여 시간 측정이 가능하다.
- **Serialization**
  - 모델 인스턴스를 간단한 원시 파이썬 데이터 구조로 직렬화하는 데 걸리는 시간
  - 이를 serializer 인스턴스 화 및 `.data` 액세스에서 이루어지는 모든 작업으로 처리할 수 있다.
- **View code**
  - `APIView`가 호출되면 실행되는 모든 것
  - 여기에 REST 프레임워크의 인증, 사용 권한, 스로틀링, 컨텐츠 협상, 요청 / 응답 처리의 메커니즘이 포함된다.
- **Response rendering**
  - REST 프레임워크의 `Response` 객체는 `TemplateResponse`의 한 유형으로, 뷰가 응답을 반환 한 후 렌더링 프로세스가 수행됨을 의미한다. 이를 반환하기 전 응답을 렌더링하도록 강제하는 수퍼클래스에 `APIView.dispatch`를 래핑할 수 있다.

파이썬의 프로파일링 모듈을 사용하는 대신 간단한것들을 유지하고 코드의 관련 부분을 타이밍 블록 안에 래핑한다.

데이터베이스 룩업과 관련하여 직렬화의 시간을 재려면 뷰에서 `.get()` 메서드를 약간 수정해야한다.

```python
def get(self, request):
    global serializer_time
    global db_time

    db_start = time.time()
    users = list(User.objects.all())
    db_time = time.time() - db_start

    serializer_start = time.time()
    serializer = UserSerializer(users)
    data = serializer.data
    serializer_time = time.time() - serializer_start

    return Response(data)
```

REST 프레임워크 내 발생하는 모든 것을 시간에 맞추기 위해 뷰가 입력되지 마자 호출되는 `.dispatch()` 메서드를 오버라이드해야한다. 이를 통해 응답 렌더링뿐만 아니라 `APIView` 클래스의 매커니즘을 시간 측정할 수 있다.

```python
def dispatch(self, request, *args, **kwargs):
    global dispatch_time
    global render_time

    dispatch_start = time.time()
    ret = super(WebAPIView, self).dispatch(request, *args, **kwargs)

    render_start = time.time()
    ret.render()
    render_time = time.time() - render_start

    dispatch_time = time.time() - dispatch_start

    return ret
```

마지막으로 Django의 request_started 및 request_finished 신호에 연결하여 나머지 request/response 응답 주기를 측정한다.

```python
def started(sender, **kwargs):
    global started
    started = time.time()

def finished(sender, **kwargs):
    total = time.time() - started
    api_view_time = dispatch_time - (render_time + serializer_time + db_time)
    request_response_time = total - dispatch_time

    print ("Database lookup               | %.4fs" % db_time)
    print ("Serialization                 | %.4fs" % serializer_time)
    print ("Django request/response       | %.4fs" % request_response_time)
    print ("API view                      | %.4fs" % api_view_time)
    print ("Response rendering            | %.4fs" % render_time)

request_started.connect(started)
request_finished.connect(finished)
```

이제 타이밍 테스트를 실행할 준비가 되었으므로 데이터베이스에 10명의 사용자를 추가한다. API 호출은 다음과 같이 curl을 사용하여 수행된다.

```shell
curl http://127.0.0.1:8000
```

| Action                  | Time(s)    | Percentage |
| ----------------------- | ---------- | ---------- |
| Database lookup         | 0.0090     | 65.7%      |
| Serialization           | 0.0025     | 18.2%      |
| Django request/response | 0.0015     | 10.9%      |
| API View                | 0.0005     | 3.6%       |
| Response render         | 0.0002     | 1.5%       |
| **Total**               | **0.0137** |            |

## Removing serialization

현재는 모델인스턴스를 반환하고 이를 직렬화한다. 간단한 작업의 경우 불필요하여 ORM을 직접 내려주도록한다.

```python
class UserListView(APIView):
    def get(self, request):
        data = User.objects.values('id', 'username', 'email')
        return Response(data)
```

Serializer는 출력 표현을 위한 표현 인터페이스를 제공하고, 입력 유효성 검사를 처리하며, 하이퍼링크된 표현과 같은 사례를 쉽게 처리하는데 유용하지만, 이와 같은 단순한 경우 항상 필요한것은 아니다.

| Action                  | Time(s)    | Percentage |
| ----------------------- | ---------- | ---------- |
| Database lookup         | 0.0090     | 80.4%      |
| Django request/response | 0.0015     | 13.4%      |
| API View                | 0.0005     | 4.5%       |
| Response render         | 0.0002     | 1.8%       |
| **Total**               | **0.0112** |            |

이는 개선되었다. 그러나 데이터베이스 룩업과 관련된 가장 큰 문제는 처리하지 않았다.

## Cache lookups

데이터베이스 조회는 여전히 시스템에서 가장 시간이 많이 소요되는 부분이다. 중요한 성능 향상을 이루려면 각 요청에 대해 데이터베이스 읽기를 수행하는 대신 대부분의 조회가 캐시에서 이루어지도록 해야한다.

캐시를 사용하기 위해 View를 변경한다.

```python
class UserListView(APIView):
    def get(self, request):
        data = cache.get('users')
        return Response(data)
```

| Action                  | Time(s)    | Percentage |
| ----------------------- | ---------- | ---------- |
| Django request/response | 0.0015     | 60.0%      |
| API View                | 0.0005     | 20.0%      |
| Redis lookup            | 0.0003     | 12.0%      |
| Response render         | 0.0002     | 5.0%       |
| **Total**               | **0.0025** |            |

성능차이가 꽤 있다.  평균 요청시간이 원래 버전보다 80% 이상 적다. Redis나 Memcached와 같은 캐시의 검색은 믿을 수 없을정도로 빠르며, 우리는 현재 요청으로 인해 걸리는 시간의 대다수가 뷰 코드에 있지않고 대신 Django의 요청/응답 사이클에 있는 시점에 와 있다.

## Slimming the view

REST 프레임워크의 VIEW의 기본 설정에는 세션 및 기본 인증과 탐색 가능한 API 및 일반 JSON용 렌더러가 모두 포함된다.

마지막으로 가능한 성능을 모두 압축해야한다면 불필요한 설정을 일부 삭제할 수 있다. **IgnoreClientContentNegotiation** 클래스를 사용하여 적절한 내용 협상을 사용하지 않고 뷰를 수정하고 탐색가능한 API 렌더러를 제거한 후 뷰에 대한 인증 또는 사용 권한을 해제한다.

```python
class UserListView(APIView):
    permission_classes = []
    authentication_classes = []
    renderer_classes = [JSONRenderer]
    content_negotiation_class = IgnoreClientContentNegotiation

    def get(self, request):
        data = cache.get('users')
        return Response(data)
```

이 시점에서 일부 기능이 손실되는 점에 유의하자. Browsable API는 더이상 불가하며 이 View는 인증이나 사용권한없이 공개적으로 액세스가 가능하다.

| Action                  | Time(s)    | Percentage |
| ----------------------- | ---------- | ---------- |
| Django request/response | 0.0015     | 71.4%      |
| Redis lookup            | 0.0003     | 14.3%      |
| API view                | 0.0002     | 9.5%       |
| Response render         | 0.0001     | 4.8%       |
| **Total**               | **0.0021** |            |

## Dropping middleware

이 시점에 성능 향상을 위한 가장 큰 잠재적 목표는 REST 프레임워크와 관련이 없으며 Django의 표준 request/response cycle에 있다. 이 시간 중 상당량이 기본 미들웨어를 처리하는데 소비된다. 극단적으로 모든 미들웨어를 설정에서 삭제하고 View가 어떻게 작동하는지 확인한다.

| Action                  | Time(s)    | Percentage |
| ----------------------- | ---------- | ---------- |
| Django request/response | 0.0003     | 33.3%      |
| Redis lookup            | 0.0003     | 33.3%      |
| API view                | 0.0002     | 22.2%      |
| Response render         | 0.0001     | 11.1%      |
| **Total**               | **0.0009** |            |

거의 대부분 경우 성능 향상을 위해 Django의 기본 미들웨어를 폐기할 수 있을 것 같지 않지만, 중요한 것은 일단 완전히 Middleware가 존재하지 않는 API View를 사용하게 되면 그것이 절감의 가장 큰 잠재적 포인트가 된다는 것이다.

## Returning regular HttpResponse

여전히 몇 퍼센트의 퍼포먼스 포인트를 필요로한다면 `REST Response`를 반환하는 대신 장고의 표준 `HttpResponse`를 리턴한다. 그렇게 하면 전체 응답 렌더링 프로세스를 실행할 필요가 없기 때문에 약간의 시간을 절약할 수 있다.표준 JSON 렌더러도 datetime 포맷과 같은 다양한 케이스를 적절하게 처리하는 맞춤형 인코더를 사용하는데, 이 경우 필요하지 않는다.

```python
class UserListView(APIView):
    permission_classes = []
    authentication_classes = []
    renderer_classes = [JSONRenderer]
    content_negotiation_class = IgnoreClientContentNegotiation

    def get(self, request):
        data = cache.get('users')
        return HttpResponse(json.dumps(data), content_type='application/json; charset=utf-8')
```

| Action                  | Time(s)    | Percentage |
| ----------------------- | ---------- | ---------- |
| Django request/response | 0.0003     | 37.5%      |
| Redis lookup            | 0.0003     | 37.5%      |
| API view                | 0.0002     | 25%        |
| **Total**               | **0.0008** |            |

마지막 단계는 REST 프레임워크 사용을 완전히 중단하고 평범한 Django View를 사용하는것이다.

## Comparing our results

다음은 각기 다른 사례의 전체 결과이다.

핑크/레드 영역은 REST 프레임워크를 나타내고 Django는 파랑, 녹색은 데이터베이스 또는 Redis조회이다.

![comparing_our_results](https://dabapps-website-media.s3-eu-west-1.amazonaws.com/images/profiling-rest-framework_LNGD2eJ.original.png)

이러한 수치는 실제로 상대적 타이밍에 대한 대략적인 관찰로 의도된것이다. 보다 복잡한 데이터베이스 검색, 계층화된 결과 또는 쓰기 작업이 있는 View는 매우 다른 특성을 가질것이다. The important thing to point out is that *the database lookup is the limiting factor*.

최적화되지 않은 기본적인 경우 어플리케이션은 다른 간단한 Django View 와 거의 같은 속도로 실해오딜것이다. 이러한 설정에 ApacheBench를 실행한것은 아니지만, 적절한 선에서 초당 수백건의 요청 비율을 달성할 수 있을 것으로 예상할 수 있다. 캐시된 케이스의 경우 초당 수천건의 요청을 더 가까이 보고있을 수 있다.

위 차트에서 봐야할것은 왼쪽에서 세번째 막대이다. 데이터베이스 액세스를 최적화하는 것이 가장 중요한 것으로, 그 후 프레임워크의 핵심 부분을 제거하여 성능을 높인다.

## The next step: ETgas and network level caching

성능이 API에 심각한 문제가 되는경우 웹 서버 최적화에 집중하지 말고 HTTP 캐싱을 올바르게 수행하는데 집중하자.

HTTP 캐싱의 사용 가능성은 사용 패턴 및 공용 API의 공개 용량에 따라 API마다 크게 달라지며 공유 캐시에 캐싱될 수 있다.

**Varnish**와 같은 서버측 캐시 뒤 API를 배치한다. 사용자 목록이 몇 초 지연을 허용할 경우 캐시가 만료된지 얼마 안된 후 서버에서 응답을 다시 검증하도록 적절한 캐시 헤더를 설정할 수 있다.

HTTP 캐싱을 올바르게 수행하고 공유 서버 측 캐시 뒤 API를 제공하면 대다수 요청이 캐시에 의해 처리되므로 성능이 크게 향상될 수 있다. 이는 공개와 같은 캐시 가능 속성을 나타내는 API와 대부분 읽기 요청을 처리하는 것에 달려 있지만 가능한 경우 이익이 커질 수 있다. Varnish와 같은 캐시는 초당 수십만건의 요청을 처리할 수 있다.

## 개선 범위

모든 소프트웨어와 마찬가지로 항상 개선의 여지가 있다.

데이터베이스 조회가 대부분 REST 프레임워크 API의 주요 성능 저하일지라도 Serialization 속도가 향상될 수 있다. REST 프레임워크의 핵심 부분에서 약간의 개선을 위해 조정할 수 있는 다른 영역은 content negotiation이다.



## Summary

### 1. ORM 조회

데이터베이스 조회가 View에서 가장 느린부분이기때문에 ORM 조회를 올바르게 처리하는 것이 중요하다. 필요한 경우 일반 View의 **.queryset** 속성에 **.select_related()** 및 **.prefetch_related()**를 사용하자. 모델 인스턴스가 큰 경우 **defer()** 또는 **only()**를 사용하여 모델 인스턴스를 부분적으로 채울 수도 있다.

### 2. 데이터베이스 병목현상

API가 요청의 수를 처리하는 데 어려움을 겪고 있다면 모델 인스턴스 전체를 적절히 캐시해야한다.

Serializer 레벨에서 캐싱을 동작하게 설정할 수 있다.

```python
class UserListView(APIView):
    """
    API View that returns results from the cache when possible.

    Note that cache invalidation would also be required.

    Invalidation should hook into the model save signal,
    or in the model `.save()` method.
    """
    def get(self, request):
        data = cache.get('users')
        if data is None:
            users = User.objects.all()
            serializer = UserSerializer(users)
            data = serializer.data
            cache.set('users', data, 86400)
        return Response(data)
```

### 3. 성능 향상을 위해 선택적으로 작업하자.

조기 최적화는 모든 악의 근원임을 기악하라. API 클라이언트가 표시하는 사용 특성을 프로파일링할 수 있는 위치에 있을 때까지 API 성능을 향상시키는것을 시작하지 말자. 그런 다음 가장 중요한 엔드포인트를 먼저 대상으로 선택적으로 view를 최적화한다.

REST 프레임워크에 좋은점 중하나는 일반적 Django View를 사용하기 때문에 일반 Django를 사용하는 대신 올바른 API 프레임워크로 작업하는 이점을 얻는 동시 개별 View를 최적화할 수 있다.

### 4. 항상 Serializers가 필요하지 않다.

성능이 중요한 View의 경우 Serializer을 완전히 삭제하고 데이터베이스 쿼리에서 **.values()**를 사용할 수 있다. 하이퍼 링크 표현 또는 기타 복잡한 필드를 사용하는 경우 데이터 구조에서 사후 처리를 수행한 후 응답으로 반환해야 할 수도 있다. REST 프레임워크는 이를 쉽게 만들도록 설계되었다.

## Final words

고성능 웹 API를 구축하려면 올바른것에 집중하자. REST 프레임워크와 완벽하게 기능하는 웹 API 프레임워크를 사용하고 프레임워크 코드의 세부사항이 아닌 API의 아키텍처를 개선하는데 힘쓰자. 직렬화된 모델 인스턴스의 적절한 캐싱과 HTTP 캐싱 및 서버측 캐시의 최적 사용은 사용중인 서버 프레임워크를 줄이려고하는 데 소요되는 시간과 비교하여 측정했을 때 큰 성능향상을 이룰 수 있다.