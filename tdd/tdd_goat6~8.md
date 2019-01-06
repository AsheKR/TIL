## 기능 테스트 개선: 격리 보장 및 Removing Voodoo Sleeps

우리의 문제를 해결하기 전 관리 항목을 살펴본다. 우선 서로 다른 테스트들이 서로 간섭하고있고, `time.sleep`은  효과적이지 못한 것 같다.

- FT가 실행된 후 정리
- time.sleeps 삭제

이 두 가지 변경 사항은 테스트를 "모범  사례" 테스트로 이동시켜 테스트를 보다 결정적이고 신뢰할 수 있게 만든다.



### 기능 테스트에서 테스트 격리 보장

마지막 챕터는 고전 테스트 문제, 즉 테스트 간 격리를 보장하는 방법으로 끝맺는다. 기능 테스트를 실행 할 때마다 목록 항목이 데이터베이스에 누락되어 다음에 테스트를 실행할 때 테스트 결과를 방해하게된다.

단위 테스트를 실행할 때 Django 테스트 러너는 새로운 테스트 데이터베이스를 자동으로 생성한다. 개별 테스트를 실행하기 전 안전하게 재설정한 다음 끝에 버린다. 그러나 현재 기능 테스트는 실제 데이터베이스인 sqlite3로 실행된다.

이 문제를 해결하는 첫 방법은 "우리 자신의" 솔루션을 롤업 하는 것이다. `functional_tests.py`에 정리를 수행할 코드를 추가한다. `setUp` 및 `tearDown` 방법은 효과적이다.

Django 1.4부터, `LiveServerTestCase` 작업을 할 수 있는 새로운 클래스가 있다. 자동으로 테스트 데이터베이스를 만들고 (단위테스트와 마찬가지로)  기능 테스트가 실행될 수 있도록 개발 서버를 시작한다. 도구로는 나중에 해결해야할 몇가지 제한사항이 있지만, 이 단계에서는 설명하지 않는다.

`ListServerTestCase` 장고 테스트 러너가 `manage.py`를 사용하여 실행할 것으로 예상된다. Django 1.6부터는 테스트러너는 test로 시작하는 파일을 찾는다. 일을 깔끔하게 유지하려면 기능 테스트를 위한 폴더를 만들어 응용 프로그램과 비슷하게 보이도록 한다.

```bash
$ mkdir functional_tests
$ touch functional_tests/__init__.py
```

그런 다음 `functional_tests.py`를 `functional_tests/tests.py`로 변경한다.

앞으로 테스트는 `python manage.py test functional_test`로  실행한다.

> 기능 테스트는 `lists` 앱 안에 존재할수도 있지만, 앱이 교차되어 실행되는 경우가 있기 때문에 별도로 유지하는것도 좋다. 자유~

이제 `tests.py`를 편집하여 `NewVisitorTest`클래스를 상속받아 사용하게 한다.

`functional_tests/tests.py`

```python
from django.test import LiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time


class NewVisitorTest(LiveServerTestCase):

    def setUp(self):
        [...]
```

그런 다음 localhost 포트 8000에 대한 방문을 하드 코딩하는 대신 다음과 `LiveServerTestCase`같은 속성을 제공한다.

`functional_tests/tests.py`

```python
    def test_can_start_a_list_and_retrieve_it_later(self):
        # Edith has heard about a cool new online to-do app. She goes
        # to check out its homepage
        self.browser.get(self.live_server_url)
        
        [...]
```



끝의 `if __name__ == '__main__'`은 Django 테스트 러너를 사용하여 FT를 실행할것이므로 원하는 경우 제거하여도 좋다.

```bash
$ python manage.py test functional_tests
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later
(functional_tests.tests.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/tests.py", line 65, in
test_can_start_a_list_and_retrieve_it_later
    self.fail('Finish the test!')
AssertionError: Finish the test!

 ---------------------------------------------------------------------
Ran 1 test in 6.578s

FAILED (failures=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

FT는 정상적으로 작동한다.



### 단위 테스트만 실행하기

이제 `manage.py test`를 하면 기능 테스트, 유닛 테스트 모두 실행한다.

```bash
$ python manage.py test
Creating test database for alias 'default'...
......F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later
[...]
AssertionError: Finish the test!

 ---------------------------------------------------------------------
Ran 7 tests in 6.732s

FAILED (failures=1)
```



단위 테스트만 실행하기위해 `lists`앱의 테스트만 실행하도록 지정할 수 있다.

```bash
$ python manage.py test lists
Creating test database for alias 'default'...
......
 ---------------------------------------------------------------------
Ran 6 tests in 0.009s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```



### 암시적, 명시적 대기시, Voodoo time.sleeps

FT의 `time.sleep`에 대해 이야기해본다.

`functional_tests/tests.py`

```python
        # When she hits enter, the page updates, and now the page lists
        # "1: Buy peacock feathers" as an item in a to-do list table
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)

        self.check_for_row_in_list_table('1: Buy peacock feathers')
```

위 `time.sleep(1)`"명시적 대기"라고 불린다. 이는 "암시적 대기"와는 대조적이다. 특정 경우 Selenium은 페이지가 로드되고 있다고 생각할 때 "자동으로" 대기하려고한다. 심지어 페이지에 없는 요소를 기다릴 시간을 제어할 수 있는`implictly_wait` 메서드가 존재한다.

하지만 테스트도 적절한 시간을 가지는게 좋고, 성능에 따라 무작위 실패인 오탐(false positives)이 발생할 수 있으므로 다른 방법으로 해결해주는것이 좋다.

`functional_tests/tests.py`

```python
from selenium.common.exceptions import WebDriverException

MAX_WAIT = 10  
[...]

    def wait_for_row_in_list_table(self, row_text):
        start_time = time.time()
        while True:  
            try:
                table = self.browser.find_element_by_id('id_list_table')  
                rows = table.find_elements_by_tag_name('tr')
                self.assertIn(row_text, [row.text for row in rows])
                return  
            except (AssertionError, WebDriverException) as e:  
                if time.time() - start_time > MAX_WAIT:  
                    raise e  
                time.sleep(0.5)  
```

1. `MAX_WAIT`은 기다릴 최대 시간을 설정한다.
2. `while True`를 사용하였기 때문에 안에서 중지하지 않는 한 계속 실행된다.
3. 기존에 있던 세 줄을 try 구문에 넣는다.
4. 통과된다면 루프를 빠져나간다.
5. 예외가 발생하면 짧은시간동안 기다렸다가 다시 시도한다. 이 때 발생하는 예외는 두가지가 있다. 페이지에서 요소를 찾을 수 없는 `AssertionError`와 페이지가 로드되지 않는 `WebDriverException`이 발생할 것이다.
6. Here’s our second escape route. If we get to this point, that means our code kept raising exceptions every time we tried it until we exceeded our timeout. So this time, we re-raise the exception and let it bubble up to our test, and most likely end up in our traceback, telling us why the test failed.

코드가 매우 못생겨보이긴하지만 나중에 리팩터링 될것이다.

이제 메서드 호출의 이름을 바꾸고  `time.sleep` Voodoo를 제거할 수 있다.

`functional_tests/tests.py`

```python
    [...]
    # When she hits enter, the page updates, and now the page lists
    # "1: Buy peacock feathers" as an item in a to-do list table
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy peacock feathers')

    # There is still a text box inviting her to add another item. She
    # enters "Use peacock feathers to make a fly" (Edith is very
    # methodical)
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Use peacock feathers to make a fly')
    inputbox.send_keys(Keys.ENTER)

    # The page updates again, and now shows both items on her list
    self.wait_for_row_in_list_table('2: Use peacock feathers to make a fly')
    self.wait_for_row_in_list_table('1: Buy peacock feathers')
    [...]
```

그리고 테스트를 다시 실행한다.

```bash
$ python manage.py test
Creating test database for alias 'default'...
......F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later
(functional_tests.tests.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/tests.py", line 73, in
test_can_start_a_list_and_retrieve_it_later
    self.fail('Finish the test!')
AssertionError: Finish the test!

 ---------------------------------------------------------------------
Ran 7 tests in 4.552s

FAILED (failures=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

같은 결과에 도달했고, 실행시간을 줄였다.

우리가 옳은 예외를 처리했는지 고의적으로 오류를 내본다.

첫번째는 `AssertionError`

`functional_tests/tests.py`

```python
        rows = table.find_elements_by_tag_name('tr')
        self.assertIn('foo', [row.text for row in rows])
        return
```

```bash
    self.assertIn('foo', [row.text for row in rows])
AssertionError: 'foo' not found in ['1: Buy peacock feathers']
```

두번째는 WebDriver Error

`functional_tests/tests.py`

```python
    try:
        table = self.browser.find_element_by_id('id_nothing')
        rows = table.find_elements_by_tag_name('tr')
        self.assertIn(row_text, [row.text for row in rows])
        return
    [...]
```

```bash
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_nothing"]
```



### 이 장에서 적용된 "모범 사례" 테스트

#### 테스트 격리 및 전역 상태 관리를 보장한다.

다른 테스트가 서로 영향을 미치지 않아야한다. 즉, 각 테스트가 끝나면 상태를 모두 재설정 해야한다. Django의 테스트 러너는 테스트 데이터베이스를 만들어 각 테스트 사이를 깨끗이 정리하는 방법을 도와준다.

#### "Voodoo"는 피한다.

우리가 무언가를 기다릴 필요가 있을때마다, `time.sleep`을 사용한다. 이는 오탐(false positive) 의 가능성이 있어 앱을 폴링하고 가능한 빨리 이동하는 재시도 루프를 선호하는게 좋다.

#### Selenium의 암시적 대기에 의존하지 마라.

이론적으로 몇가지 "암시적" 대기를 수행하지만 브라우저마다 다르다.



## 점진적으로 작업하기

### Small Design When Necessary

여러개의 목록을 지원하는 방법에 대해 생각한다. 현재 FT는 다음과 같다.

`functional_tests/tests.py`

```python
    # Edith wonders whether the site will remember her list. Then she sees
    # that the site has generated a unique URL for her -- there is some
    # explanatory text to that effect.
    self.fail('Finish the test!')

    # She visits that URL - her to-do list is still there.

    # Satisfied, she goes back to sleep
```

그러나 실제로 서로 다른 사용자가 서로의 목록을 보지 않으며 각자 자신의 저장된 목록으로 돌아가는 방법으로 자신의 URL을 얻는다는 점을 확장하려고 한다. 새로운 디자인은 어떻게 생겼나?

### Not Big Design Up Front

TDD는 소프트웨어 개발의 민첩한 움직임과 밀접한 관련이 있으며, 이는 전통적인 소프트웨어 엔지니어링 관행인 _Big Design Up Front_에 대한 반작용을 포함한다. 이 경우 오랜시간 연습한 후 소프트웨어가 문서상으로 계획되는 긴 설계가 있다. 신속한 변화를 위한 철학은 이론보다는 실제 문제를 해결함으로서 더 많은것을 배운다는 것이며, 특히 실제 사용자와 가능한 빨리 직면할 때 더욱 그러하다. 긴 선행 설계 단계 대신ㄴ에, 우리는 최소 실행 가능한 애플리케이션을 조기에 도입하고 실제 사용에서 나온 피드백에 따라 설계가 점차적으로 진화하도록 한다.

하지만 그렇다고해서 디자인에 대한 사고가 완전히 금지되는것은 아니다. 이전 장에서 어떻게 그냥 앞으로 나아가는 것이 결국 옳은 답을 얻을 수 있는지를 보았지만, 종종 디자인에 대해 조금 생각하는 것이 더 빠르게 될 수도 있다. 최소 실행가능한 목록 앱과 이 앱을 제공하는데 필요한 설계를 생각해보자.

- 우리는 각 사용자가 적어도 지금은 자신의 목록을 저장 할 수 있기를 바란다.

- 목록은 몇가지 항목으로 구성되며, 그 주요 특성은 다소 서술적인 텍스트다.

- 우리는 한 번의 방문에서 다음 방문을 위해 리스트를 저장할필요가 있다. 우선, 우리는 각 사용자에게 그들 고유의 URL을 주어야한다. 나중에 우리는 사용자를 자동으로 인식하고 그들의 목록을 표시하는 방법을 원할것이다.

현재 항목을 전달하기 위해, 우리는 목록들과 그 아이템들을 데이터베이스에 저장 할 것 같다. 각 목록은 고유한 URL을 가지며, 각 목록 항목은 특정 목록과 관련된 설명 텍스트이다.

### YAGNI!

일단 디자인에 대해 생각하기 시작하면 멈추기 어려울 수 있다. 다른 모든 생각이 떠오르게 된다. -- 각 목록에는 이름 또는 제목을 지정하고, 사용자 이름 비밀번호를 사용하여 사용자를 인식하기를 원할 수도 있고, 목록에는 짧은 설명뿐 아니라 긴 노트 필드를 추가하고싶을 대도 있고, 어떤 종류의 주문을 저장하기를 원할수도있다. 그러나 우리는 또 다른 agile 신조 "YAGNI"를 따르는데, 그것은 "너는 그것을 필요로 하지 않을것이다."라는 뜻이다. 소프트웨어 개발자로서, 우리는 무언가를 창조하는 것을 즐기고, 때로는 아이디어가 우리에게 떠올라 그것이 필요할 수 있다는 이유만으로 무언가를 건설하려는 충동을 억제하기 어렵다. 문제는 그 아이디어가 아무리 멋있어도 결국 그것을 사용하지 않는다는 것이다. 사용하지 않는 코드가 로드되어 애플리케이션의 복잡성이 가중된다. YAGNI는 우리가 지나치게 열성적인 창의적 충동에 저항하기 위해 사용되는 주문이다.

### REST (ish)

웹 브라우저를 사용하여 웹 서버와 상호작용하는 방법을 선택한다.

REST는 웹 기반 API의 설계를  안내하는데 주로 사용되는 웹 설계에 대한 접근 방식이다. REST 규칙을 엄격히 준수하는 것은 불가능하지만, 몇가지 유용한 영감을 제공한다.

REST는 목록 및 목록 항목에 데이터 구조와 일치하는 URL 구조를 가지고 있음을 암시한다. 각 목록은 고유한 URL을 가질 수 있다.

```bash
/lists/<list identifier>/
```

이는 FT에서 지정한 요구사항을 충족한다. 목록을 보려면 GET 요청을 사용한다.

새로운 목록을 만드려면 POST 요청을 받는 특수 URL이 있어야한다.

```
/lists/new
```

기존 목록에 새 항목을 추가할 때 필요한 URL도 필요하다.

```
/lists/<list identifier>/add_item
```

다시말하지만 REST의 규칙을 완전히 따르는것이 아닌, 영감을 얻기위해 REST를 사용하는것이다.

요약하자면 이 장의 스크래치 패드는 다음과 같다.

- 항목이 다른 목록과 연결되도록 모델 조정
- 각 목록에 대한 고유 URL 추가
- POST를 통해 새 목록을 만들기위한 URL 추가
- POST를 통해 기존 목록에 새 항목을 추가하기 위한 URL 추가

### TDD를 사용한 점진적 디자인 구현

최상위 단계에서는 새로운 기능 추가(새로운 FT 추가 및 새로운 애플리케이션 코드 작성)와 기존 구현의 일부를 재구성하여 사용자에게 동일 기능을 제공하지만 새로운 설계의 측면을 사용하도록 할 것이다. 기존 기능 테스트를 사용하여 이미 작동중인 기능을 해재하지 않았는지, 새 기능 테스트를 통해 새로운 기능을 구동할 수 있는지 확인한다.

단위 테스트 수준에서 새로운 테스트를 추가하거나 기존 테스트를 수정하여 우리가 원하는 변화를 테스트할 수 있으며, 우리는 우리가 만지지 않는 단위 테스트를 유사하게 사용하여 그 과정에서 어떤 것도 깨지지 않도록 할 수 있을 것이다.

### 회귀 분석 테스트 보장

스크래치 패드를 새로운 기능 테스트 방법으로 바꾸어 두번째 사용자를 생성하고 할 일 목록이 Edith와 별개인지 확인한다.

Edith는 할 일 목록을 만드는 첫 번째 항목을 추가하지만 Edith의 목록은 고유한 URL에서 살아야한다는 새로운기능을 소개한다.

`functional_tests/tests.py`

```python
def test_can_start_a_list_for_one_user(self):
    # Edith has heard about a cool new online to-do app. She goes
    [...]
    # The page updates again, and now shows both items on her list
    self.wait_for_row_in_list_table('2: Use peacock feathers to make a fly')
    self.wait_for_row_in_list_table('1: Buy peacock feathers')

    # Satisfied, she goes back to sleep


def test_multiple_users_can_start_lists_at_different_urls(self):
    # Edith starts a new to-do list
    self.browser.get(self.live_server_url)
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Buy peacock feathers')
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy peacock feathers')

    # She notices that her list has a unique URL
    edith_list_url = self.browser.current_url
    self.assertRegex(edith_list_url, '/lists/.+')  
```

`assertRegex`를 통해 문자열이 정규식과 일치하는지 확인하는 함수를 생성했다. REST 디자인이 생성되었는지 확인하기 위해 이를 사용했다.

다음으로 새로운 사용자가 오는 것을 테스트한다. 

```python
    [...]
    self.assertRegex(edith_list_url, '/lists/.+')  

    # Now a new user, Francis, comes along to the site.

    ## We use a new browser session to make sure that no information
    ## of Edith's is coming through from cookies etc
    self.browser.quit()
    self.browser = webdriver.Firefox()

    # Francis visits the home page.  There is no sign of Edith's
    # list
    self.browser.get(self.live_server_url)
    page_text = self.browser.find_element_by_tag_name('body').text
    self.assertNotIn('Buy peacock feathers', page_text)
    self.assertNotIn('make a fly', page_text)

    # Francis starts a new list by entering a new item. He
    # is less interesting than Edith...
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Buy milk')
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy milk')

    # Francis gets his own unique URL
    francis_list_url = self.browser.current_url
    self.assertRegex(francis_list_url, '/lists/.+')
    self.assertNotEqual(francis_list_url, edith_list_url)

    # Again, there is no trace of Edith's list
    page_text = self.browser.find_element_by_tag_name('body').text
    self.assertNotIn('Buy peacock feathers', page_text)
    self.assertIn('Buy milk', page_text)

    # Satisfied, they both go back to sleep
```

이중 해시 (`##`)를 통해 테스트가 작동하는 방식에 대한 주석과 사용자 스토리를 설명하는 FT의 일반 주석을 구분할 수 있게 했다. 

이후 테스트가 예상되로 에러가 발생하는지 확인한다.

```bash
$ python manage.py test functional_tests
[...]
.F
======================================================================
FAIL: test_multiple_users_can_start_lists_at_different_urls
(functional_tests.tests.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/tests.py", line 83, in
test_multiple_users_can_start_lists_at_different_urls
    self.assertRegex(edith_list_url, '/lists/.+')
AssertionError: Regex didn't match: '/lists/.+' not found in
'http://localhost:8081/'

 ---------------------------------------------------------------------
Ran 2 tests in 5.786s

FAILED (failures=1)
```

### 새로운 디자인을 향항 반복

할 일 목록에는 네가지가 있다. 그 중 발생한 에러인 Regexp가 일치하지 않는 작업을 수행하여 해결한다.

URL은  POST 후 리다이렉션으로 온다. `lists/tests.py`에서 `test_redirects_after_POST`를 찾아 예상 리다이렉션 위치를 변경한다.

`lists/tests.py`

```python
self.assertEqual(response.status_code, 302)
self.assertEqual(response['location'], '/lists/the-only-list-in-the-world/')
```

현재로선 이상해보일지 몰라도, 한번에 한가지를 바꾸는데 전념하고있다. 이는 REST 디자인으로 가는 과정이고, 나중에 여러개의 목록을 가질 때 쉽게 바뀔것이다.

테스트를 실행하면 다음의 실패가 예상된다.

```bash
$ python manage.py test lists
[...]
AssertionError: '/' != '/lists/the-only-list-in-the-world/'
```

``lists/views.py`의 `home_page`에서 리다이렉션 코드를 변경한다.

`lists/views.py`

```python
def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/lists/the-only-list-in-the-world/')

    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

물론, 우리는 이러한 URL이 없기때문에 기능 테스트를 망칠것이다.

```bash
  File "...python-tdd-book/functional_tests/tests.py", line 57, in
test_can_start_a_list_for_one_user
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_list_table"]

[...]

  File "...python-tdd-book/functional_tests/tests.py", line 79, in
test_multiple_users_can_start_lists_at_different_urls
    self.wait_for_row_in_list_table('1: Buy peacock feathers')
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_list_table"]
```

우리는 새로운 테스트를 실패했을뿐 아니라, 이전 테스트도 실패하였다. 이는 우리가 퇴보를 도입했다는 것을 말한다. 가능한 빨리 작동상태로 돌리도록 수정하자.



### 첫 번째 자체 완성 단계: 하나의 새로운 URL

`lists/tests.py`를 열고 새로운 테스트 클래스 `ListViewTest`를 추가한다.  `test_displays_all_list_items`라는 메서드를 `HomePageTest` 새 클래스에 복사하고 이름을 약간 수정한다.

`lists/tests.py`

```python
class ListViewTest(TestCase):

    def test_displays_all_items(self):
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')

        response = self.client.get('/lists/the-only-list-in-the-world/')

        self.assertContains(response, 'itemey 1')  
        self.assertContains(response, 'itemey 2')  
```

여기서 새로운 헬퍼 메서드가 존재한다. `assertin`/`response.content.decode()`대신 Django는 `assertContains`를 사용하여 응답과 해당 내용의 바이트를 처리하는 메서드를 제공한다.

테스트 실행:

```bash
    self.assertContains(response, 'itemey 1')
[...]
AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404
```

아직 URL이 존재하지 않아 테스트가 실패하고 404를 반환한다.

### 새 URL

우리 싱글톤 리스트 URL은 아직 존재하지 않는다. `superlists/urls.py`에서 수정한다.

`superlists/urls.py`

```python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]
```

테스트를 다시 실행하면 다음과 같이 표기된다.

```bash
AttributeError: module 'lists.views' has no attribute 'view_list'
```



### 새 함수

`lists/views.py`에 더미 뷰 함수를 생성한다.

`lists/views.py`

```python
def view_list(request):
    pass
```

우리는 다음의 결과를 얻는다.

```bash
ValueError: The view lists.views.view_list didn't return an HttpResponse
object. It returned None instead.

[...]
FAILED (errors=1)
```

실패를 통해 올바른 방향으로 이끌리고 있다. `home_page` 뷰에서 마지막 두줄을 복사하여 트릭이 수행되는지 확인한다.

```bash
Ran 7 tests in 0.016s
OK
```

단위테스트는 통과하지만 FT는 통과하지 못한다.

```bash
FAIL: test_can_start_a_list_for_one_user
[...]
  File "...python-tdd-book/functional_tests/tests.py", line 67, in
test_can_start_a_list_for_one_user
[...]
AssertionError: '2: Use peacock feathers to make a fly' not found in ['1: Buy
peacock feathers']

FAIL: test_multiple_users_can_start_lists_at_different_urls
[...]
AssertionError: 'Buy peacock feathers' unexpectedly found in 'Your To-Do
list\n1: Buy peacock feathers'
[...]
```

`lists/templates/home.html`의 form은 action 속성을 지정해주지 않아 기본적으로 현재 페이지로 POST 데이터를 보낸다. 즉, 첫번째 POST시에는 `/lists/`로 보내지고 `home_page`뷰를 통해 `/lists/the-only-list-in-the-world`로 리다이렉트된다. 그리고 두번재 POST시에는 `lists/the-only-list-in-the-world`로 보내지만 들어온 데이터를 처리하는 내용이 없어 삽입이 불가하다.

그러므로 삽입이 가능하도록 `/` 에 보내 `home_page`뷰가 처리하도록 해야한다.

`lists/templates/home.html`

```html
        <form method="POST" action="/">
```

그럼 기존의 에러는 해결하고 더 나은 에러로 오게된다.

```bash
FAIL: test_multiple_users_can_start_lists_at_different_urls
[...]
AssertionError: 'Buy peacock feathers' unexpectedly found in 'Your To-Do
list\n1: Buy peacock feathers'

Ran 2 tests in 8.541s
FAILED (failures=1)
```

원래 테스트는 다시 한번 통과해서 정상 상태로 돌아간다는 것을 안다. 새로운 기능은 아직 작동하지 않을 수 있지만, 기존 것은 잘 작동한다.



### Green? Refactor

조금 정리하는 시간.

_Red/Green/Refactor_에서 현재 Green에 도착했다. 그래서 리팩토링이 필요한 항목을 찾는다. 이제 홈페이지용과 개별 목록용의 두가지 보기가 존재한다. 둘다 현재 동일한 템플릿을 사용하며 현재 데이터베이스에 있는 모든 목록을 전달한다. 단위 테스트 항목을 살펴보면 우리가 바꾸고싶어하는 것을 찾을 수 있다.

```bash
$ grep -E "class|def" lists/tests.py
class HomePageTest(TestCase):
    def test_uses_home_template(self):
    def test_displays_all_list_items(self):
    def test_can_save_a_POST_request(self):
    def test_redirects_after_POST(self):
    def test_only_saves_items_when_necessary(self):
class ListViewTest(TestCase):
    def test_displays_all_items(self):
class ItemModelTest(TestCase):
    def test_saving_and_retrieving_items(self):
```

`HomePageTest`의 `test_displays_all_list_items` 메서드를 삭제할 수 있다. 더이상 필요하지 않아 테스트의 개수를 줄일 수 있다.

다음으로 더 이상 모든 목록을 표시하기위해 홈페이지 템플릿이 필요하지 않기때문에, 새 목록을 시작할 수 있는 하나의 입력상자만을 표시한다.

### 또 다른 작은 단계: 목록을 보기위한 별도의 템플릿

`home_page`와 `view_list`는 이제 상당히 다른 페이지이므로 서로 다른 HTML 템플릿을 사용해야한다. `home.html`은 단일 입력 상자를 가지는 반면, 새로운 템플릿인 `list.html`은 기존 항목의 표를 보여줄 수 있다.

새 테스트를 통해 다른 템플릿을 사용하는지 확인한다.

`lists/tests.py`

```python
class ListViewTest(TestCase):

    def test_uses_list_template(self):
        response = self.client.get('/lists/the-only-list-in-the-world/')
        self.assertTemplateUsed(response, 'list.html')


    def test_displays_all_items(self):
        [...]
```

`assertTemplateUsed` 는 장고 테스트 가 제공하는 유용한 함수이다.

테스트:

```bash
AssertionError: False is not true : Template 'list.html' was not a template
used to render the response. Actual template(s) used: home.html
```

이제 view를 바꾼다.

`lists/views.py`

```python
def view_list(request):
    items = Item.objects.all()
    return render(request, 'list.html', {'items': items})
```

당연하지만, `list.html`도 없다.

```bash
django.template.exceptions.TemplateDoesNotExist: list.html
```

`lists/templates/list.html`을 생성한다.

```bash
$ touch lists/templates/list.html
```

```bash
AssertionError: False is not true : Couldn't find 'itemey 1' in response
```

개별 목록을 위한 템플릿은 현재 `home.html`에 있는 많은 것들을 재사용할것이다.

```bash
$ cp lists/templates/home.html lists/templates/list.html
```

이제 모든 테스트가 통과하게된다. 이제 좀 더 정리를 해본다. 홈페이지는 항목을 나열할 필요가 없으며, 새 목록 입력 필드만 필요하므로 `lists/templates/home.html`에서 일부 행을 제거하고 `h1`을 "새 작업관리 목록 시작"으로 약간 조정할 수 있다.

`lists/templates/home.html`

```html
<body>
  <h1>Start a new To-Do list</h1>
  <form method="POST">
    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
    {% csrf_token %}
  </form>
</body>
```

다시 단위 테스트를 실행하였을 때 아무것도 깨지지 않았음을 확인한다.

`home_page`에 아무런 목록을 내려주지 않아도 되므로 더 최적화한다.

`lists/views.py`

```python
def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/lists/the-only-list-in-the-world/')
    return render(request, 'home.html')
```

여전히 단위 테스트는 동작하지만, 기능 테스트는 통과하지 않는다.

```bash
AssertionError: '1: Buy milk' not found in ['1: Buy peacock feathers', '2: Buy
milk']
```

나쁘지 않다. 우리는 회귀 테스트 (첫번째 FT)는 지나가고, 우리의 새로운 테스트는 조금씩 발전하고 있다. Prancis는 자신의 목록 페이지를 여전히 얻지 못하는 것을 말해준다.

### 세번째 단계: 목록 항목 추가를 위한 URL

- 항목이 다른 목록과 연결되도록 모델 조정
- 각 목록에 고유한 URL 추가
- POST를 통해 새 목록을 만들기위한 URL 추가
- POST를 통해 기존 목록에 새 항목을 추가하기위한 URL 추가

우선 새 목록 항목을 추가하기위한 새 URL을 생성한다.

###  새로운 리스트 생성을 위한 테스트 클래스

`lists/tests.py`에서 `test_can_save_a_POST_request`와 `test_redirects_after_POST`를 가져와  새 테스트케이스를 작성한다.

`lists/tests.py`

```python
class NewListTest(TestCase):

    def test_can_save_a_POST_request(self):
        self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')


    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/lists/the-only-list-in-the-world/')
```



`self.assertEqual(response ...)`를 대체하는 `assertRedirects`를 사용한다.

`lists/tests.py`

```python
    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertRedirects(response, '/lists/the-only-list-in-the-world/')
```

테스트:

```bash
    self.assertEqual(Item.objects.count(), 1)
AssertionError: 0 != 1
[...]
    self.assertRedirects(response, '/lists/the-only-list-in-the-world/')
[...]
AssertionError: 404 != 302 : Response didn't redirect as expected: Response
code was 404 (expected 302)
```

첫 번째 실패는 새 항목을 저장하지 않았다는 것을 의미하고, 두번째 실패는 302 리다이렉션 대신 404를 반환했다는 것이다. 이는 `/lists/new`에 대한 URL을 작성하지 않았기 때문이다.

### 새로운 리스트 생성을 위한 URL과 View

`superlists/urls.py`

```python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]
```

`lists/views.py`

```python
def new_list(request):
    pass
```

그 후 테스트하면 `HttpResponse`가 필요한 것을 알 수 있지만 리다이렉트가 필요한 것을 알기 때문에 `home_page`의 리다이렉트 코드를 붙여넣는다.

`lists.views.py`

```python
def new_list(request):
    return redirect('/lists/the-only-list-in-the-world/')
```

```bash
    self.assertEqual (Item.objects.count (), 1) 
AssertionError : 0! = 1
```

이제 오브젝트 생성 에러만 해결해준다.

`lists/views.py`

```python
def new_list(request):
    Item.objects.create(text=request.POST['item_text'])
    return redirect('/lists/the-only-list-in-the-world/')
```

모든 단위 테스트가 작동한다.

### 중복 제거

좋아보이지만, 더 나은 코드를 위해 단순화하길 원한다.

목록 항목 생성은 더이상 `home_page` view에서 사용되지 않으므로 `if request.method == 'POST'` 부분을 전부 없앨 수 있다.

`lists/views.py`

```python
def home_page(request):
    return render(request, 'home.html')
```

또한 `test_only_saves_items_when_neccessary`도 제거할 수 있다.

### A Regression! Pointing Our Forms at the New URL

```bash
ERROR: test_can_start_a_list_for_one_user
[...]
  File "...python-tdd-book/functional_tests/tests.py", line 57, in
test_can_start_a_list_for_one_user
    self.wait_for_row_in_list_table('1: Buy peacock feathers')
  File "...python-tdd-book/functional_tests/tests.py", line 23, in
wait_for_row_in_list_table
    table = self.browser.find_element_by_id('id_list_table')
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_list_table"]

ERROR: test_multiple_users_can_start_lists_at_different_urls
[...]
  File "...python-tdd-book/functional_tests/tests.py", line 79, in
test_multiple_users_can_start_lists_at_different_urls
    self.wait_for_row_in_list_table('1: Buy peacock feathers')
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_list_table"]
[...]

Ran 2 tests in 11.592s
FAILED (errors=2)
```

왜냐하면 여전히 URL Action은 `lists/new`가 아닌 곳을 가리키고 있기 때문이다.

`lists/templates/home.html, lists/templates/list.html`

```html
    <form method="POST" action="/lists/new">
```

테스트:

```bash
AssertionError: '1: Buy milk' not found in ['1: Buy peacock feathers', '2: Buy
milk']
[...]
FAILED (failures=1)
```

해야할 목록을 정리할 수 있다.

- 항목이 다른 목록과 연결되도록 모델 조정
- 각 목록에 고유한 URL 추가
- ~~POST를 통해 새 목록을 만들기위한 URL 추가~~
- POST를 통해 기존 목록에 새 항목을 추가하기위한 URL 추가

### Biting the Bullet: 모델 조정

이제 모델을 바꾼다.

`lists/tests.py`

```Github
@@ -1,5 +1,5 @@
 from django.test import TestCase
-from lists.models import Item
+from lists.models import Item, List


 class HomePageTest(TestCase):
@@ -44,22 +44,32 @@ class ListViewTest(TestCase):



-class ItemModelTest(TestCase):
+class ListAndItemModelsTest(TestCase):

     def test_saving_and_retrieving_items(self):
+        list_ = List()
+        list_.save()
+
         first_item = Item()
         first_item.text = 'The first (ever) list item'
+        first_item.list = list_
         first_item.save()

         second_item = Item()
         second_item.text = 'Item the second'
+        second_item.list = list_
         second_item.save()

+        saved_list = List.objects.first()
+        self.assertEqual(saved_list, list_)
+
         saved_items = Item.objects.all()
         self.assertEqual(saved_items.count(), 2)

         first_saved_item = saved_items[0]
         second_saved_item = saved_items[1]
         self.assertEqual(first_saved_item.text, 'The first (ever) list item')
+        self.assertEqual(first_saved_item.list, list_)
         self.assertEqual(second_saved_item.text, 'Item the second')
+        self.assertEqual(second_saved_item.list, list_)
```

새 목록 객체를 만들고, 각각의 항목을 `.list` 속성으로 할당한다. 목록이 제대로 저장되었는지 확인하고, 두 항목이 목록과의 관계도 저장했는지 확인한다. 또한 목록 객체를 직접 기본 키가 동일한지 확인하여 (`saved_list`, `list_`) 서로 비교한다.

다른 단위 테스트/ 코드 사이클의 시간이다.

매번 테스트를 실행할 때마다 어떤 코드를 입력해야하는지 분명히 보여주기보다는, 테스트를 실행할 때 예상되는 오류만을 보여줄 것이다. 하나씩 해결하고 각 최소 코드 변경사항이 맞는지 확인한다.

첫 에러는:

```bash
ImportError: cannot import name 'List'
```



고치고 나서 두번째 에러는:

```bash
AttributeError: 'List' object has no attribute 'save'
```

그리고 보게될 에러는:

```bash
django.db.utils.OperationalError: no such table: lists_list
```

반드시 `makemigrations`를 하도록 하자:

```bash
$ python manage.py makemigrations
Migrations for 'lists':
  lists/migrations/0003_list.py
    - Create model List
```

그리고 보게될 에러는

```bash
    self.assertEqual(first_saved_item.list, list_)
AttributeError: 'Item' object has no attribute 'list'
```



### Foreign Key 관계

`lists/models.py`

```python
from django.db import models

class List(models.Model):
    pass

class Item(models.Model):
    text = models.TextField(default='')
    list = models.ForeignKey(List, default=None)
```

`python manage.py makemigrations` 후 테스트



### Adjusting the Rest the World to Our New Models

```bash
$ python manage.py test lists
[...]
ERROR: test_displays_all_items (lists.tests.ListViewTest)
django.db.utils.IntegrityError: NOT NULL constraint failed: lists_item.list_id
[...]
ERROR: test_redirects_after_POST (lists.tests.NewListTest)
django.db.utils.IntegrityError: NOT NULL constraint failed: lists_item.list_id
[...]
ERROR: test_can_save_a_POST_request (lists.tests.NewListTest)
django.db.utils.IntegrityError: NOT NULL constraint failed: lists_item.list_id

Ran 6 tests in 0.021s

FAILED (errors=3)
```

기본적으로 NULL은 허용하지 않기때문에 객체 저장 시 모든 필드를 필요로하게된다.

`lists/tests.py`

```python
class ListViewTest(TestCase):

    def test_displays_all_items(self):
        list_ = List.objects.create()
        Item.objects.create(text='itemey 1', list=list_)
        Item.objects.create(text='itemey 2', list=list_)
```

이를 해결하고 나면 두 가지 실패한 테스트를 찾을 수 있다.

```bash
File "...python-tdd-book/lists/views.py", line 9, in new_list
Item.objects.create(text=request.POST['item_text'])
```

이것도 마찬가지로 NULL을 비허용하기때문에 `list_`를 넣어준다.

이후 테스트:

```bash
Ran 6 tests in 0.030s

OK
```

이제 다시 FT를 확인한다.

```bash
AssertionError: '1: Buy milk' not found in ['1: Buy peacock feathers', '2: Buy
milk']
[...]
```

List 추가를 해결했고 다시 스크래치 패드를 확인한다.

- ~~항목이 다른 목록과 연결되도록 모델 조정~~
- 각 목록에 고유한 URL 추가
- ~~POST를 통해 새 목록을 만들기위한 URL 추가~~
- POST를 통해 기존 목록에 새 항목을 추가하기위한 URL 추가

### 각 목록은 자신만의 URL을 소유해야한다.

목록에 대한 고유식별자로 사용할 수 있는 항목은? 아마도 현재로서 가장 간단한 것은 데이터베이스의 자동 생성된 ID 필드를 사용하는 것이다. 두 테스트가 새 url을 가리키도록 ListViewTest를 변경한다.

또한 기존 `test_displays_all_items` 테스트를 변경하고 `test_displays_only_items_for_list`로 호출하여 특정 목록에 대한 항목만 표시되는지 확인한다.

`lists/tests.py`

```python
class ListViewTest(TestCase):

    def test_uses_list_template(self):
        list_ = List.objects.create()
        response = self.client.get(f'/lists/{list_.id}/')
        self.assertTemplateUsed(response, 'list.html')


    def test_displays_only_items_for_that_list(self):
        correct_list = List.objects.create()
        Item.objects.create(text='itemey 1', list=correct_list)
        Item.objects.create(text='itemey 2', list=correct_list)
        other_list = List.objects.create()
        Item.objects.create(text='other list item 1', list=other_list)
        Item.objects.create(text='other list item 2', list=other_list)

        response = self.client.get(f'/lists/{correct_list.id}/')

        self.assertContains(response, 'itemey 1')
        self.assertContains(response, 'itemey 2')
        self.assertNotContains(response, 'other list item 1')
        self.assertNotContains(response, 'other list item 2')
```

이후 테스트는 404 에러가 발생한다.

```bash
FAIL: test_displays_only_items_for_that_list (lists.tests.ListViewTest)
AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404
(expected 200)
[...]
FAIL: test_uses_list_template (lists.tests.ListViewTest)
AssertionError: No templates used to render the response
```

### Capturing Parameters from URLs

`superlists/urls.py`

```python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/(.+)/$', views.view_list, name='view_list'),
]
```

다시 테스트

```bash
ERROR: test_displays_only_items_for_that_list (lists.tests.ListViewTest)
[...]
TypeError: view_list() takes 1 positional argument but 2 were given
[...]
ERROR: test_uses_list_template (lists.tests.ListViewTest)
[...]
TypeError: view_list() takes 1 positional argument but 2 were given
[...]
ERROR: test_redirects_after_POST (lists.tests.NewListTest)
[...]
TypeError: view_list() takes 1 positional argument but 2 were given
FAILED (errors=3)
```

이는 쉽게 더미 파라미터로 해결할 수 있다.

`lists/views.py`

```python
def view_list(request, list_id):
    [...]
```

예상처럼 실패가 발생한다.

```bash
FAIL: test_displays_only_items_for_that_list (lists.tests.ListViewTest)
[...]
AssertionError: 1 != 0 : Response should not contain 'other list item 1'
```

list의 item만 가져오도록 수정한다.

`lists/views.py`

```python
def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    items = Item.objects.filter(list=list_)
    return render(request, 'list.html', {'items': items})
```



### Adjusting new _list to the New World

이제 다른 에러가 발생한다.

```bash
ERROR: test_redirects_after_POST (lists.tests.NewListTest)
ValueError: invalid literal for int() with base 10:
'the-only-list-in-the-world'
```

`test_redirects_after_POST`는 기본적으로 `lists/the-only-list-in-the-world/`로 보내게 되어있는데 우리가 만든 URL로 보내기위해 수정한다.

`lists/tests.py`

```python
    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
        new_list = List.objects.first()
        self.assertRedirects(response, f'/lists/{new_list.id}/')
```

또한 View의 Redirect도 고쳐준다.

`lists/views.py`

```python
def new_list(request):
    list_ = List.objects.create()
    Item.objects.create(text=request.POST['item_text'], list=list_)
    return redirect(f'/lists/{list_.id}/')
```

이제 모든 테스트가 통과한다.

### The Functional Tests Detect Another Regression

```bash
F.
======================================================================
FAIL: test_can_start_a_list_for_one_user
(functional_tests.tests.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/tests.py", line 67, in
test_can_start_a_list_for_one_user
    self.wait_for_row_in_list_table('2: Use peacock feathers to make a fly')
[...]
AssertionError: '2: Use peacock feathers to make a fly' not found in ['1: Use
peacock feathers to make a fly']

 ---------------------------------------------------------------------
Ran 2 tests in 8.617s

FAILED (failures=1)
```

우리의 테스트는 실제로 통과하고 다른 사용자느 목록을 얻을 수 있지만, 이전 테스트는 경고한다. 더 이상 두번째 항목을 추가할 수 없는 것 같다. 이는 스크래치 패드의 항목을 해결하면 되는 일이다!

- ~~항목이 다른 목록과 연결되도록 모델 조정~~
- ~~각 목록에 고유한 URL 추가~~
- ~~POST를 통해 새 목록을 만들기위한 URL 추가~~
- POST를 통해 기존 목록에 새 항목을 추가하기위한 URL 추가



### 기존 목록에 항목 추가를 처리하는 View

`lists/tests.py`

```python
class NewItemTest(TestCase):

    def test_can_save_a_POST_request_to_an_existing_list(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        self.client.post(
            f'/lists/{correct_list.id}/add_item',
            data={'item_text': 'A new item for an existing list'}
        )

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new item for an existing list')
        self.assertEqual(new_item.list, correct_list)


    def test_redirects_to_list_view(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        response = self.client.post(
            f'/lists/{correct_list.id}/add_item',
            data={'item_text': 'A new item for an existing list'}
        )

        self.assertRedirects(response, f'/lists/{correct_list.id}/')
```

테스트:

```bash
AssertionError: 0 != 1
[...]
AssertionError: 301 != 302 : Response didn't redirect as expected: Response
code was 301 (expected 302)
```

### Beware of Greedy Regular Expressions!

조금 이상할수도 있다. URL을 아직 지정하지 않았으므로 예상되는 오류는 `404 != 302`이다. 왜 301을 얻고 있는가? 이는 URL의 정규식인 `lists/(.+)/`가 `1/add_item` 모두를 캡쳐하고있기 때문이다. 우리는 숫자만을 잡기위해 URL을 수정한다.

`superlists/urls.py`

```python
    url(r'^lists/(\d+)/$', views.view_list, name='view_list'),
```

테스트:

```bash
AssertionError: 0 != 1
[...]
AssertionError: 404 != 302 : Response didn't redirect as expected: Response
code was 404 (expected 302)
```

### 마지막 새 URL

`superlists/urls.py`

```python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/(\d+)/$', views.view_list, name='view_list'),
    url(r'^lists/(\d+)/add_item$', views.add_item, name='add_item'),
]
```

매우 흡사해보이는 URL이 존재한다. 이는 나중에 처리하도록 스크래치 패드에 적어놓는다.

- ~~항목이 다른 목록과 연결되도록 모델 조정~~
- ~~각 목록에 고유한 URL 추가~~
- ~~POST를 통해 새 목록을 만들기위한 URL 추가~~
- POST를 통해 기존 목록에 새 항목을 추가하기위한 URL 추가
- urls.py의 중복을 리팩토링

테스트:

```bash
AttributeError: module 'lists.views' has no attribute 'add_item'
```

### 마지막 새 View

`lists/views.py`

```python
def add_item(request):
    pass
```

```bash
TypeError: add_item() takes 1 positional argument but 2 were given
```

`lists/views.py`

```python
def add_item(request, list_id):
    pass
```

```bash
ValueError: The view lists.views.add_item didn't return an HttpResponse object.
It returned None instead.
```

`lists/views.py`

```python
def add_item(request, list_id):
    list_ = List.objects.get(id=list_id)
    return redirect(f'/lists/{list_.id}/')
```

그리고 테스트를 통과한다.

### Testing the Response Context Objects Directly

기존 목록에 항목을 추가하기위한 새로운 View와 URL이 있다. 이제 `list.html`에 실제로 사용하면 된다.

`lists/templates/list.html`

```html
    <form method="POST" action="/lists/{{ list.id }}/add_item">
```

이를 위해 View는 템플릿에 list를 전달해야한다. 이를 위해 새 단위 테스트를 작성한다.

`lists/test.py`

```python
class ListViewTest(TestCase):
    [...]
    
    def test_passes_correct_list_to_template(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()
        response = self.client.get(f'/lists/{correct_list.id}/')
        self.assertEqual(response.context['list'], correct_list)
```

테스트:

```bash
KeyError: 'list'
```

View에서 `list`를 넘겨주지 않았기 때문에 발생한다.

`lists/views.py`

```python
def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    return render(request, 'list.html', {'list': list_})
```

이는 기존 테스트를 무너뜨린다.

```bash
FAIL: test_displays_only_items_for_that_list (lists.tests.ListViewTest)
[...]
AssertionError: False is not true : Couldn't find 'itemey 1' in response
```

`list.html`의 POST action을 통해 고칠 수 있다.

`lists/templates/list.html`

```django
    <form method="POST" action="/lists/{{ list.id }}/add_item">  

      [...]

      {% for item in list.item_set.all %}  
        <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
      {% endfor %}
```

이로서 모든 테스트가 통과한다.

```bash
Ran 9 tests in 0.040s

OK
```

FT는?

```bash
$ python manage.py test functional_tests
[...]
..
 ---------------------------------------------------------------------
Ran 2 tests in 9.771s

OK
```

- ~~항목이 다른 목록과 연결되도록 모델 조정~~
- ~~각 목록에 고유한 URL 추가~~
- ~~POST를 통해 새 목록을 만들기위한 URL 추가~~
- ~~POST를 통해 기존 목록에 새 항목을 추가하기위한 URL 추가~~
- urls.py의 중복을 리팩토링



### 마지막 URL Refactor

```bash
$ cp superlists/urls.py lists/
```

`superlists/urls.py`

```python
from django.conf.urls import include, url
from lists import views as list_views  
from lists import urls as list_urls  

urlpatterns = [
    url(r'^$', list_views.home_page, name='home'),
    url(r'^lists/', include(list_urls)),  
]
```

`lists/urls.py`

```python
from django.conf.urls import url
from lists import views

urlpatterns = [
    url(r'^new$', views.new_list, name='new_list'),
    url(r'^(\d+)/$', views.view_list, name='view_list'),
    url(r'^(\d+)/add_item$', views.add_item, name='add_item'),
]
```



## Prettification: 레이아웃 및 스타일링 테스트

해당 챕터에서는 장고에서 정적 파일이 어떻게 작동하는지, 그리고 테스트를 위하여 할 일을 배운다.

### 레이아웃 및 스타일에 대해 기능적으로 테스트해야 할 대상

다음은 우리가 원하는 몇가지 사항이다.

- 새 목록과 기존 목록을 추가하기 위한 멋진 큰 입력 필드
- 집중을 위해 중심에 놓이길 원함.

TDD는 이를 어떻게 테스트할가? 이는 상수를 테스트하는것과 비슷하다.

우선 기능테스트에서 새로운 테스트 방법으로 시작한다.

`functional_tests/tests.py`

```python
class NewVisitorTest(LiveServerTestCase):
    [...]


    def test_layout_and_styling(self):
        # Edith goes to the home page
        self.browser.get(self.live_server_url)
        self.browser.set_window_size(1024, 768)

        # She notices the input box is nicely centered
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertAlmostEqual(
            inputbox.location['x'] + inputbox.size['width'] / 2,
            512,
            delta=10
        )
```

여기서 몇가지 새로운 것들. 먼저 창 크기를 고정된 크기로 설정한다. 그런 다음 입력 요소를 찾고 크기와 위치를 살펴보고 중간에 위치하는지 확인하기 위한 연산을 수행한다. `assertAlmostEqual`  10픽셀 내 스크롤바 등으로 인한 오류를 처리하는데 사용된다.

기능 테스트를 실행하면 다음과 같다.

```bash
$ python manage.py test functional_tests
[...]
.F.
======================================================================
FAIL: test_layout_and_styling (functional_tests.tests.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/tests.py", line 129, in
test_layout_and_styling
    delta=10
AssertionError: 117.0 != 512 within 10 delta

 ---------------------------------------------------------------------
Ran 3 tests in 9.188s

FAILED (failures=1)
```

이는 예상된 실패다. 하지만 이런 종류의 FT는 틀리기 쉬우므로, 입력 상자가 중앙에 위치할 때 FT도 통과하는지 확인하기위해 빠르고 더러운 "치트"솔루션을 사용한다. FT를 확인하는데 사용하는 즉시 이 코드를 삭제한다.

`lists/templates/home.html`

```django
<form method="POST" action="/lists/new">
  <p style="text-align: center;">
    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
  </p>
  {% csrf_token %}
</form>
```

FT가 작동한다. 입력 상자가 `list.html`의 페이지 중앙 정렬되도록 하기위해 다음과 같이 확장한다.

`functional_tests/tests.py`

```python
    # She starts a new list and sees the input is nicely
    # centered there too
    inputbox.send_keys('testing')
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: testing')
    inputbox = self.browser.find_element_by_id('id_new_item')
    self.assertAlmostEqual(
        inputbox.location['x'] + inputbox.size['width'] / 2,
        512,
        delta=10
    )
```

이는 다음과 같은 오류를 발생한다.

```bash
  File "...python-tdd-book/functional_tests/tests.py", line 141, in
test_layout_and_styling
    delta=10
AssertionError: 117.0 != 512 within 10 delta
```



### Prettification: CSS 프레임워크 사용하기

디자인은 어렵고 모바일, 태블릿 등을 다루어야한다. 그렇기 때문에 CSS프레임워크를 사용한다.

여기서는 bootstrap을 사용한다. [http://getbootstrap.com/](http://getbootstrap.com/)



[ TDD 아닌거 생략... ]



### Switching to StaticLiveServerTestCase

`runserver`는 자동으로 staticfile을 불러오지만 `LiveServerTestCase`는 그렇지 않다.

staticfile을 불러오기위해 `StaticLiveServerTestCase`를 사용한다.

`functional_tests/tests.py`

```github
@@ -1,14 +1,14 @@
-from django.test import LiveServerTestCase
+from django.contrib.staticfiles.testing import StaticLiveServerTestCase
 from selenium import webdriver
 from selenium.common.exceptions import WebDriverException
 from selenium.webdriver.common.keys import Keys
 import time

 MAX_WAIT = 10


-class NewVisitorTest(LiveServerTestCase):
+class NewVisitorTest(StaticLiveServerTestCase):

     def setUp(self):
```

