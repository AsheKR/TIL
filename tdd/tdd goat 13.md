# 데이터베이스 계층에서 유효성 검사

현재까지의 FT는 빈 값이 입력되었을 때 `.has-error`를 찾을 수 없다.

```bash
$ python3 manage.py test functional_tests.test_list_item_validation
[...]
======================================================================
ERROR: test_cannot_add_empty_list_items
(functional_tests.test_list_item_validation.ItemValidationTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/test_list_item_validation.py", line
15, in test_cannot_add_empty_list_items
    self.wait_for(lambda: self.assertEqual(
[...]
  File "...python-tdd-book/functional_tests/test_list_item_validation.py", line
16, in <lambda>
    self.browser.find_element_by_css_selector('.has-error').text,
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: .has-error
```



## 모델 유효성 검사

웹 응용프로그램에는 클라이언트와 서버측 두 곳에서 검증할 수  있다. 하지만 누군가가 악의적으로 또는 일부 버그를 통해 클라이언트 측 검증을 우회할 수 있기 때문에 서버측이 더 안전한다.

서버측에서 Django에는 검증할 수 있는 두가지 단계가 있다. One is at the model level, and the other is higher up at the forms level. I like to use the lower level whenever possible, partially because I’m a bit too fond of databases and database integrity rules, and partially because, again, it’s safer—you can sometimes forget which form you use to validate input, but you’re always going to use the same database.



### The self.assertRaises Context Manager

모델 레이어에서 단위 테스트를 작성해보자. `ListAndItemModelTest`는 비어있는 list를 만드려할 때의 새 테스트이다. 이 테스트는 테스트중인 코드가 어떤 예외를 발생시키는지 테스트하기 때문에 흥미롭다.

`lists/tests/test_models.py`

```python
from django.core.exceptions import ValidationError
[...]

class ListAndItemModelsTest(TestCase):
    [...]

    def test_cannot_save_empty_list_items(self):
        list_ = List.objects.create()
        item = Item(list=list_, text='')
        with self.assertRaises(ValidationError):
            item.save()
```

이는 새로운 단위 테스트 기술이다. 무언가 오류를 발생하는지 확인하고싶을 때 `self.assertRaises` 컨텍스트 관리자를 사용할 수 있다. 이는 다음과 같은 말이지만, with와 함께쓰는 `assertRasies`가 더 깔끔하다.

```python
try:
    item.save()
    self.fail('The save should have raised an exception')
except ValidationError:
    pass
```

```bash
    item.save()
AssertionError: ValidationError not raised
```



### Django Quirk: 모델 저장이 유효성을 검사하지 않음

장고의 작은 단점 하나를 발견할 수 있다. 이 시험은 통과해야한다. Django 모델 필드의 문서를 보면 TextField가 `blank=True`이므로 빈 값을 허용하지 않을것이다.



그런데 왜 실패하는가? 역사적 이유로, Django 모델들은 저장하면서 검증을 실행하지 않는다. 나중에 알게되겠지만, 데이터베이스에서 실제로 구현되는 제약조건은 저장오류를 발생시키지만, SQLite는 텍스트열에 Blank에 제약조건을 적용하는것을 지원하지 않으므로 save 메서드는 자동으로 잘못된 값을 전달한다.

제약조건이 데이터베이스 수준에서 발생하는지 여부를 확인하는 방법이 있다. 데이터베이스 수준에서 제약 조건을 적용하려면 마이그레이션이 필요하다. 하지만 Django는 SQLite가 이러한 유형의 제약을 지원하지 않는다는 것을 알고 있으므로 `makemigrations`를 실행해도 아무것도 변경되지 않는다.

```bash
$ python manage.py makemigrations
No changes detected
```

Django는 `full_clean`이라는 수동으로 완전한 검증을 실행하는 방법을 가지고 있다.

`lists/tests/test_models.py`

```python
    with self.assertRaises(ValidationError):
        item.save()
        item.full_clean()
```

유닛 테스트가 성공한다.

The FTs will still fail though, because we’re not actually forcing these errors to appear in our actual app, outside of this one unit test.



### View에서 표면 모델 검증 오류

View 레이어에서 모델 유효성 검사를 시행하고 템플릿에서 가져와 사용자가 볼 수 있게 해보자. HTML에 오류를 선택적으로 표시하는 방법은 다음과 같다. 템플릿에 오류 변수가 전달되었는지 여부를 확인하고 오류 변수가 전달 된 경우 양식 옆에 표시한다.

`lists/templates/base.html`

```django
  <form method="POST" action="{% block form_action %}{% endblock %}">
    <input name="item_text" id="id_new_item"
           class="form-control input-lg"
           placeholder="Enter a to-do item" />
    {% csrf_token %}
    {% if error %}
      <div class="form-group has-error">
        <span class="help-block">{{ error }}</span>
      </div>
    {% endif %}
  </form>
```

오류를 템플릿에 전달하는 것은 뷰의 작업이다. `NewListTest`클래스의 단위테스트를 살펴보자. 여기서 두 가지 약간 다른 오류 처리 패턴을 사용하고자 한다.

첫 경우 새 목록에 대한 URL과 View가 선택적으로 홈페이지와 동일한 템플릿을 렌더링하지만 오류 메세지가 추가된다. 그 단위테스트는 다음과 같다.

`lists/tests/test_views.py`

```python
class NewListTest(TestCase):
    [...]

    def test_validation_errors_are_sent_back_to_home_page_template(self):
        response = self.client.post('/lists/new', data={'item_text': ''})
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'home.html')
        expected_error = "You can't have an empty list item"
        self.assertContains(response, expected_error)
```

이 테스트를 작성하면서 `lists/new`URL은 약간 불쾌할수도있다. 이 URL은 수동으로 문자열을 입력한다. 하드코딩된 많은 URL은 DRY원칙을 위배하고, 이를 리팩터링하는것이 좋다. 지금 당장하지는 않는다. 왜냐하면 지금 앱은 망가진 상태이기때문이다.

다시 우리 테스트로 돌아와서, 200응답 대신 302 리다이렉션을 반환하기 때문에 실패한 테스트로 돌아간다.

View에서 `full_clean()`을 호출해보자.

`lists/views.py`

```python
def new_list(request):
    list_ = List.objects.create()
    item = Item.objects.create(text=request.POST['item_text'], list=list_)
    item.full_clean()
    return redirect(f'/lists/{list_.id}/')
```

여기서도 하드코딩된 URL을 지우기위해 스크래치패드에 추가하도록 하자.

- Remove hardcoded URLs from views.py

이제 다시 테스트

```bash
[...]
  File "...python-tdd-book/lists/views.py", line 11, in new_list
    item.full_clean()
[...]
django.core.exceptions.ValidationError: {'text': ['This field cannot be
blank.']}
```

첫번째 접근방법으로 `try/except`를 사용하여 오류를 감지한다. 우선 except로 아무것도 실행하지 않는다. 테스트 결과가 다음에 코드화해야할것이 무엇인지 알려준다.

`lists/views/py`

```python
from django.core.exceptions import ValidationError
[...]

def new_list(request):
    list_ = List.objects.create()
    item = Item.objects.create(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
    except ValidationError:
        pass
    return redirect(f'/lists/{list_.id}/')
```

302 != 200 에러가 발생한다.

렌더링된 템플릿을 돌려준다. 그러면 템플릿 검사도 처리한다.

`lists/views.py`

```python
    except ValidationError:
        return render(request, 'home.html')
```

다시 테스트하면 템플릿에 help_text를 가진 요소가 없다고 나온다.

```bash
AssertionError: False is not true : Couldn't find 'You can't have an empty list
item' in response
```

템플릿에 변수를 넘겨주자.

`lists/views.py`

```python
    except ValidationError:
        error = "You can't have an empty list item"
        return render(request, 'home.html', {"error": error})
```

달라진게 없다..

출력기반 디버깅을 통해 보자.

```
AssertionError: False is not true : Couldn't find 'You can't have an empty list
item' in response
```

`lists/tests/test_views.py`

```python
expected_error = "You can't have an empty list item"
print(response.content.decode())
self.assertContains(response, expected_error)
```

Django의 `HTML-escaped`때문에 같지 않다고 나온것이다.

```bash
[...]
<span class="help-block">You can&#39;t have an empty list
item</span>
```

테스트에 다음을 비교하도록 수정한다.

```python
expected_error = "You can&#39;t have an empty list item"
```

이보다는 Django의 hepler 함수를 사용하는것이 더 좋다.

`lists/tests/test_views.py`

```python
from django.utils.html import escape
[...]

        expected_error = escape("You can't have an empty list item")
        self.assertContains(response, expected_error)
```

성공!

```bash
Ran 11 tests in 0.047s

OK
```



### 잘못된 입력이 데이터베이스에 저장되지 않았는지 확인하기

여기서 오류가 있다면 유효성 검사가 실패하더라도 현재 개체를 만드는것이다.

`lists/views.py`

```python
    item = Item.objects.create(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
    except ValidationError:
        [...]
```

비어있는 항목이 저장되지 않도록 새 단위 테스트를 작성한다.

`lists/tests/test_views.py`

```python
class NewListTest(TestCase):
    [...]

    def test_validation_errors_are_sent_back_to_home_page_template(self):
        [...]

    def test_invalid_list_items_arent_saved(self):
        self.client.post('/lists/new', data={'item_text': ''})
        self.assertEqual(List.objects.count(), 0)
        self.assertEqual(Item.objects.count(), 0)
```

```bash
[...]
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests/test_views.py", line 40, in
test_invalid_list_items_arent_saved
    self.assertEqual(List.objects.count(), 0)
AssertionError: 1 != 0
```

고쳐보자.

`lists/views.py`

```python
def new_list(request):
    list_ = List.objects.create()
    item = Item(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
        item.save()
    except ValidationError:
        list_.delete()
        error = "You can't have an empty list item"
        return render(request, 'home.html', {"error": error})
    return redirect(f'/lists/{list_.id}/')
```

단위 테스트는 통과하고, FT는 통과하나?

```bash
$ python manage.py test functional_tests.test_list_item_validation
[...]
File "...python-tdd-book/functional_tests/test_list_item_validation.py", line
29, in test_cannot_add_empty_list_items
    self.wait_for(lambda: self.assertEqual(
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: .has-error
```

여전히 통과하지 않는다. 별 진행은 없지만 시험의 첫번째 부분을 지나 두번째 항목을 제출하도록 오류를 표시하는것을 볼 수 있다.

### 장고 패턴: 폼을 렌더링할때와 동일한 View에서 POST 요청 처리하기

이는 조금 다른 접근법을 사용하지만 Django에서는 매우 일반적인 패턴이다. 동일한 VIEW를 사용하여 POST를 처리하여 폼을 렌더링하는것이다. 이는 RESTful URL 모델에 잘 맞지 않지만 동일한 URL이 이양식을 표시하고 사용자 입력 처리시 발생하는 모든 오류를 표시할 수 있다는 중요한 장점이 있다.

현재 상황은 목록을 표시하기위한 하나의 View와 URL과 해당 목록에 추가된 항목을 처리하기위한 하나의 View 및 URL이 있다는 것이다. 그것들을 하나로 결합한다.

`lists/templates/list.html`

```django
{% block form_action %}/lists/{{ list.id }}/{% endblock %}
```

덧붙여 이는 하드코딩된 URL이다. 스크래치패드에 추가하자.

- *Remove hardcoded URLs from views.py*
- *Remove hardcoded URL from forms in list.html and home.html*

`view_list` 페이지는 아직 POST 요청을 처리하는 방법을 알지 못하기 때문에 FT는 실패한다.

```bash
$ python manage.py test functional_tests
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: .has-error
[...]
AssertionError: '2: Use peacock feathers to make a fly' not found in ['1: Buy
peacock feathers']
```



### Refcator: new_item 기능을 view_list로 전송

`ListViewTest`의 기존 목록에 POST 요청을 저장하고 이동하는 모든 기존 테스트를 가져온다. 이 때 달라지는점은 `/add_item` 대신 기본 list URL을 가리키게 해야한다.

`lists/tests/test_views.py`

```python
class ListViewTest(TestCase):

    def test_uses_list_template(self):
        [...]

    def test_passes_correct_list_to_template(self):
        [...]

    def test_displays_only_items_for_that_list(self):
        [...]

    def test_can_save_a_POST_request_to_an_existing_list(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        self.client.post(
            f'/lists/{correct_list.id}/',
            data={'item_text': 'A new item for an existing list'}
        )

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new item for an existing list')
        self.assertEqual(new_item.list, correct_list)


    def test_POST_redirects_to_list_view(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        response = self.client.post(
            f'/lists/{correct_list.id}/',
            data={'item_text': 'A new item for an existing list'}
        )
        self.assertRedirects(response, f'/lists/{correct_list.id}/')
```

`NewItemTest`클래스가 완전히 사라지는것에 유의하자. 또한 리다이렉션 테스트의 이름을 변경하여 POST 요청에만 적용된다는 것을 명시했다.

이는 다음의 결과를 준다

```bash
FAIL: test_POST_redirects_to_list_view (lists.tests.test_views.ListViewTest)
AssertionError: 200 != 302 : Response didn't redirect as expected: Response
code was 200 (expected 302)
[...]
FAIL: test_can_save_a_POST_request_to_an_existing_list
(lists.tests.test_views.ListViewTest)
AssertionError: 0 != 1
```

`view_list` 은 두가지 유형의 요청을 처리하는 함수로 변경한다.

`lists/views.py`

```python
def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'], list=list_)
        return redirect(f'/lists/{list_.id}/')
    return render(request, 'list.html', {'list': list_})
```

이제 테스트를 통과한다.

```bash
Ran 12 tests in 0.047s

OK
```

이제 더 이상 필요없는 `add_item` View를 삭제한다.

```bash
[...]
AttributeError: module 'lists.views' has no attribute 'add_item'
```

View를 삭제했지만 여전히 urls.py 에서 참조되고있으므로 제거해야한다.

`lists/urls.py`

```python
urlpatterns = [
    url(r'^new$', views.new_list, name='new_list'),
    url(r'^(\d+)/$', views.view_list, name='view_list'),
]
```

그리고 단위 테스트는 성공적으로 수행된다. 그 후 FT 실행

```bash
$ python manage.py test
[...]
ERROR: test_cannot_add_empty_list_items
[...]

Ran 16 tests in 15.276s
FAILED (errors=1)
```



### view_list에서의 Model validation 실행

우리는 여전히 모델 검증 규칙을 적용을 받기 위해 기존 목록에 항목을 추가하기를 원한다. 이를 위해 새로운 단위 테스트드를 작성한다.

`lists/tests/test_views.py`

```python
class ListViewTest(TestCase):
    [...]

    def test_validation_errors_end_up_on_lists_page(self):
        list_ = List.objects.create()
        response = self.client.post(
            f'/lists/{list_.id}/',
            data={'item_text': ''}
        )
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'list.html')
        expected_error = escape("You can't have an empty list item")
        self.assertContains(response, expected_error)
```

현재 유효성을 검사하지 않고 모든 POST에 대해 리다이렉션하기 때문에 실패한다.

```bash
    self.assertEqual (response.status_code, 200) 
AssertionError : 302! = 200
```



`lists/views.py`

```python
def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    error = None

    if request.method == 'POST':
        try:
            item = Item(text=request.POST['item_text'], list=list_)
            item.full_clean()
            item.save()
            return redirect(f'/lists/{list_.id}/')
        except ValidationError:
            error = "You can't have an empty list item"

    return render(request, 'list.html', {'list': list_, 'error': error})
```

여기 중복 코드가 발생한다. `try/except`문이 두번 발생하게되므로 나중에 리팩토링을한다.

- *Remove hardcoded URLs from views.py*
- *Remove hardcoded URL from forms in list.html and home.html*
- *Remove duplication of validation logic in views*



### 하드코딩된 URL 삭제

_urls.py_에서 `name=`을 사용한 이름으로 url을 동적으로 매핑할 수 있다.



#### The {% url %} Template Tag

`lists/templates/home.html`

```python
{% block form_action %}{% url 'new_list' %}{% endblock %}
```

`lists/templates/list.html`

```python
{% block form_action %}{% url 'view_list' list.id %}{% endblock %}
```



### Using get_absolute_url for Redirects

`redirect`를 통해 단위, 기능테스트를 통과할 수 있지만 그보다 더 좋은 `get_absolute_url`을 장고에서 지원해준다. 장고의 모델 객체는 종종 특정 URL과 연결되므로 어떤 페이지에 항목이 표시되는지 나타내는 `get_absolute_url`이라는 특수 함수를 정의할 수 있다.

`lists/tests/test_models.py`

```python
    def test_get_absolute_url(self):
        list_ = List.objects.create()
        self.assertEqual(list_.get_absolute_url(), f'/lists/{list_.id}/')
```

이는 AttributeError를 발생시킨다.

```bash
AttributeError: 'List' object has no attribute 'get_absolute_url'
```

`lists/models.py`

```python
from django.core.urlresolvers import reverse


class List(models.Model):

    def get_absolute_url(self):
        return reverse('view_list', args=[self.id])
```

이제 view에서 사용가능하다. 

`lists/views.py`

```python
def new_list(request):
    [...]
    return redirect(list_)

def view_list(request, list_id):
    [...]

            item.save()
            return redirect(list_)
        except ValidationError:
            error = "You can't have an empty list item"
```

