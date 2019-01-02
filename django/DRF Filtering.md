# DRF Filtering

기본적인 필터기능 이외에 __(a)__ 복수 필드를 필터링, __(b)__ 콤마로 분리되어 온 요청에 대해 모두 필터링하는 기능을 작성한다.



[완성 코드는 여기](https://github.com/teachmesomething2580/DRF-FILTERING-TUTORIAL)



## 프로젝트 생성

### 가상환경 설정

```bash
$ mkdir filtering_test
$ pipenv --python 3.6.6
$ pipenv shell
```



### 패키지 설치

```bash
$ pipenv install django
$ pipenv install djangorestframework
$ pipenv install django-filter
```



### 장고 패키지 생성

```bash
$ django-admin startproject config
$ mv config app
$ cd app
$ django-admin startapp post
```



## 프로젝트 환경설정

`config/settings.py`

```python
INSTALLED_APPS = [
    'post',

    ...

    'django_filters',
    'rest_framework',
]
```



`config/urls.py`

API와 관련된 설정은 app의 `apis` 폴더 하위에 둔다.

```python
urlpatterns = [
    ...
    path('apis/', include('config.apis.urls')),
]
```



`config/apis/urls.py`

```python
urlpatterns = [
    path('post/', include('post.apis.urls')),
]
```



## Post 설정

### Post App API 폴더 구조

```wiki
post/
	# API 폴더만 생성 후 하위 파일을 생성해주면 된다.
	+ apis/
	|	+-- filters.py
	|	+-- serializers.py
	|	+-- urls.py
	|	+-- views.py
	+ migrations/
	+ __init__.py
	+ admin.py
	+ apps.py
	+ models.py
	+ tests.py
	+ views.py
```





### Model 설정

`post/models.py`

```python
class Post(models.Model):
    title = models.CharField(
    	max_length=50,
    )
    content = models.TextField()
```



### URL 설정

`post/apis/urls.py`

```python
urlpatterns = [
    path('', PostListCreateView.as_view()),
]
```



### serializers 설정

`post/apis/views.py`

```python
class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__'
```



### view 설정

`post/apis/views.py`

```python
class PostListCreateView(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    filter_backends = (DjangoFilterBackend, )  # 필터기능을 사용하기위해 반드시 불러와야한다.
    filter_class = PostFilter  # 추후 생성되는 필터 클래스
```



### Filter 설정

`post/apis/filters.py`

```python
class PostFilterList(django_filters.Filter):
    """콤마로 들어온 요청에 대해 필터링하는 클래스
    
    name: 필터링하고싶은 필드의 이름을 가진다.
    lookup_type: 필터링 방법을 가진다.
    """
    def __init__(self, name, lookup_type, *args, **kwargs):
        self.name = name
        self.lookup_type = lookup_type
        super().__init__(*args, **kwargs)

    def filter(self, qs, value):
        if value not in (None, ''):
            values = [v for v in value.split(',')]
            q = Q()
            for separated_value in values:
                q |= Q(**{'%s__%s' % (self.name, self.lookup_type): separated_value})
            return qs.filter(q)
        return qs


class PostFilter(django_filters.FilterSet):
    """필터링 가능한 필드를 작성하는 클래스
    
    filter_content_and_title: 검색어가 content와 title둘 중 하나에 포함되어있는 것을 필터링한다.
    filter_title_with_comma_separated: Comma로 분리된 여러 검색어를 받았을 때 title에 하나라도 포함되어있는 것을 필터링한다.
    """
    filter_content_and_title = django_filters.CharFilter(method='get_filter_content_and_title', label='filter_content_and_title', )
    filter_title_with_comma_separated = PostFilterList(name='title', lookup_type='icontains', label='filter_title_with_comma_separated', )

    class Meta:
        model = Post
        fields = {
            'filter_content_and_title': ['exact', ],
            'filter_title_with_comma_separated': ['exact', ],
        }

    def get_filter_content_and_title(self, qs, name, value):
        return qs.filter(
            Q(title__icontains=value) | Q(content__icontains=value)
        )
```



#### filter_content_and_title

검색어가 들어왔을 때 `title` 필드와 `content` 필드에 포함되어있는 Post만 뽑는 필터링이다.

1. 기존 모델의 필드에 없으므로 커스텀 필드를 정의한다. 

   `filter_content_and_title = django_filters.CharFilter()`

2. 해당 필드를 처리하는 방법을 정의하기위해 `method` 인수를 사용하여 주어진 메서드로 처리하게한다.

   `... = django_filters.CharFilter(method='get_filter_content_and_title')`

> 해당 예제에서 label 인자는 DRF 테스트용도로 사용되었다. HTML으로 보여줄 때 나타나는 form field의 label이다.



```python
def get_filter_content_and_title(self, qs, name, value):
    return qs.filter(
        Q(title__icontains=value) | Q(content__icontains=value)
    )
```

간단히 `Q()` 오브젝트를 사용하여 `OR` 연산을 처리했다.



#### filter_title_with_comma_separated

컴마로 분리된 값이 왔을 때 모든 값을 검사하여 title에 포함되어있는 Post만 뽑는 필터링이다.

위에서는 메서드로 필터링 처리방법을 설정했다면 클래스를 사용하여 처리할 수도 있다.



```python
class PostFilterList(django_filters.Filter):
    def __init__(self, name, lookup_type, *args, **kwargs):
        self.name = name
        self.lookup_type = lookup_type
        super().__init__(*args, **kwargs)

    def filter(self, qs, value):
        if value not in (None, ''):
            values = [v for v in value.split(',')]
            q = Q()
            for separated_value in values:
                q |= Q(**{'%s__%s' % (self.name, self.lookup_type): separated_value})
            return qs.filter(q)
        return qs
```



1. 기존 모델 필드를 사용하여 필터링할 수 없으므로 필터링 클래스를 정의한다.

   `class PostFilterList(django_filters.Filter):`

2. 인자로 넘겨받은 `lookup_type, name`이 자동으로 채워지지 않기 때문에 `__init__`에서 직접 삽입한다.
3. `filter` 함수를 재정의한다.

```python
def filter(self, qs, value):
    if value not in (None, ''):  # 아무 값을 받지 않았다면 모든 쿼리셋을 그대로 리턴한다.
        values = [v for v in value.split(',')]  # 컴마로 분리된 값을 리스트로 변환한다.
        q = Q()  # 빈 Q() 인스턴스를 생성한다.
        for separated_value in values:  # 리스트로 변환한 검색어를 OR 연산을 위해 Q() 오브젝트로 채워넣는다.
            q |= Q(**{'%s__%s' % (self.name, self.lookup_type): separated_value})
        return qs.filter(q)
    return qs
```



#### 공통

반드시 커스텀필드를 작성하였으면 FilterSet 클래스의 Meta에 `fields`란에 기입해주어야한다.



 ## 테스트

`runserver`이후 `127.0.0.1:8000/apis/post/`로 들어가게되면 DRF 전용 HTML이 보이게된다.

하단의 `HTML form`을 통해 생성이 가능하며

우측 상단의 `Filters`를 통해 필터링을 테스트해볼 수 있다.