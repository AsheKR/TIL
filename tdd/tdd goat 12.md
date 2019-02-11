# 테스트를 여러 파일로 나누고 Generic Wait Helper 작성

몇 사용자가 사이트를 사용하기 시작하면 우연히 빈 목록 항목을 제출하거나 두 개의 동일한 항목을 목록에 입력하는 등, 실수로 목록을 엉망으로 만드는 경우가 있다.

`functional_tests/tests.py`

```python
def test_cannot_add_empty_list_items(self):
    # Edith goes to the home page and accidentally tries to submit
    # an empty list item. She hits Enter on the empty input box

    # The home page refreshes, and there is an error message saying
    # that list items cannot be blank

    # She tries again with some text for the item, which now works

    # Perversely, she now decides to submit a second blank list item

    # She receives a similar warning on the list page

    # And she can correct it by filling some text in
    self.fail('write me!')
```

기능이 많아짐으로 기능테스트가 조금 혼잡해지기 시작했다. 한 파일에 하나의 테스트만 포함하도록 여러 파일로 나누어보자.

### 테스트 건너 뛰기

리팩토링시에는 항상 완벽하게 통과하는 테스트를 갖는것이 좋다. 방금전 고의 실패 테스트를 작성했다. 일시적으로 "skip"이라는 데코레이터를 사용하여 해당 테스트를 건너뛰자.

`fucntional_tests/tests.py`

```python
from unittest import skip
[...]

    @skip
    def test_cannot_add_empty_list_items(self):
```

테스트를 무시하도록 지시하는것이다. 테스트를 다시 실행하면 테스트가 통과한다.

```bash
$ python manage.py test functional_tests
[...]
Ran 4 tests in 11.577s
OK
```

> Skip 데코레이션을 사용하는 것은 위험하므로 저장소 적용전 제거해야한다. 이때문에 git diff를 보는것을 추천한다.



### 기능 테스트를 여러 파일로 분할

자신의 메서드를 갖게하는 테스트 클래스로 분할한다.

`functional_tests/tests.py`

```python
class FunctionalTest(StaticLiveServerTestCase):

    def setUp(self):
        [...]
    def tearDown(self):
        [...]
    def wait_for_row_in_list_table(self, row_text):
        [...]


class NewVisitorTest(FunctionalTest):

    def test_can_start_a_list_for_one_user(self):
        [...]
    def test_multiple_users_can_start_lists_at_different_urls(self):
        [...]


class LayoutAndStylingTest(FunctionalTest):

    def test_layout_and_styling(self):
        [...]



class ItemValidationTest(FunctionalTest):

    @skip
    def test_cannot_add_empty_list_items(self):
        [...]
```

이후 다시 테스트를 실행하면 여전히 모든 테스트가 작동하는것을 볼 수 있다.

```bash
Ran 4 tests in 11.577s

OK
```

이제 각각의 클래스를 파일에 옮기고 사용한다.

```bash
$ git mv functional_tests / tests.py functional_tests / base.py 
$ cp functional_tests / base.py functional_tests / test_simple_list_creation.py 
$ cp functional_tests / base.py functional_tests / test_layout_and_styling.py 
$ cp functional_tests / base.py functional_tests / test_list_item_validation.py
```

`FunctionalTest`클래스의 중복을 없애기위해 `base.py`를 사용했다.

`functional_tests/base.py`

```python
import os
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium import webdriver
from selenium.common.exceptions import WebDriverException
import time

MAX_WAIT = 10



class FunctionalTest(StaticLiveServerTestCase):

    def setUp(self):
        [...]
    def tearDown(self):
        [...]
    def wait_for_row_in_list_table(self, row_text):
        [...]
```

> 헬퍼메서드를 기본 `FunctionalTest`클래스에 보관하면 FT 중복을 방지할 수 있다. 나중에 "페이지 패턴"을 사용할것이지만 상속에 대한 구성을 선호한다.

`functional_tests/test_simple_list_creation.py`

```python
from .base import FunctionalTest
from selenium import webdriver
from selenium.webdriver.common.keys import Keys


class NewVisitorTest(FunctionalTest):

    def test_can_start_a_list_for_one_user(self):
        [...]
    def test_multiple_users_can_start_lists_at_different_urls(self):
        [...]
```

테스트가 상속받을 `base.py`가 같은 위치에 있음을 알기에 상대 경로인 `from .base`를 사용했다.

`functional_tests/test_layout_and_styling.py`

```python
from selenium.webdriver.common.keys import Keys
from .base import FunctionalTest


class LayoutAndStylingTest(FunctionalTest):
        [...]
```

`functional_tests/tests_list_item_validation.py`

```python
from selenium.webdriver.common.keys import Keys
from unittest import skip
from .base import FunctionalTest


class ItemValidationTest(FunctionalTest):

    @skip
    def test_cannot_add_empty_list_items(self):
        [...]
```

다시 테스트를 통해 정상적으로 작동하는지 확인한다.

```bash
Ran 4 tests in 11.577s

OK
```

그리고 skip 데코레이션을 삭제한다.

`functional_tests/test_list_item_validation.py`

```python
class ItemValidationTest(FunctionalTest):

    def test_cannot_add_empty_list_items(self):
        [...]
```



### 하나의 테스트 파일 실행하기

보너스로, 다음과 같이 개별 테스트 파일을 실행할 수 있다.

```python
$ python manage.py test functional_tests.test_list_item_validation
[...]
AssertionError: write me!
```



### 새 기능 테스트 툴: Generic Explicit Wait Helper

먼저, 테스트 코드를 작성한다.

`functional_tests/test_list_item_validation.py`

```python
def test_cannot_add_empty_list_items(self):
    # Edith goes to the home page and accidentally tries to submit
    # an empty list item. She hits Enter on the empty input box
    self.browser.get(self.live_server_url)
    self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

    # The home page refreshes, and there is an error message saying
    # that list items cannot be blank
    self.assertEqual(
        self.browser.find_element_by_css_selector('.has-error').text,  
        "You can't have an empty list item"  
    )

    # She tries again with some text for the item, which now works
    self.fail('finish this test!')
    [...]
```

다음은 테스트가 검사하는 방법이다.

1. `.has-error` 오류 텍스트를 표시하기위해 호출되는 CSS 클래스를 사용하도록 지정한다.
2. 오류가 우리가 원하는 메세지를 표시하는지 확인한다.

여기서 발생할 수 있는 오류는, 엔터키를 눌렀을 때 페이지가 새로고침되고 화면이 로딩되기를 기다려야하는데 코드는 진행되어 새로고침전 `assertEqual`문이 실행하게되어 항상 실패하게된다.

이를 기다리기위해 명시적 대기는 도우미 메서드로 작성한다. 

`functional_tests/test_list_item_validation.py`

```python
[...]
    # The home page refreshes, and there is an error message saying
    # that list items cannot be blank
    self.wait_for(lambda: self.assertEqual(  
        self.browser.find_element_by_css_selector('.has-error').text,
        "You can't have an empty list item"
    ))
```

assert문을 직접 실행하는 대신 람다 함수로 래핑하고 도우미 메서드로 전달한다.

`functional_tests/base.py`

```python
    def wait_for(self, fn):
        start_time = time.time()
        while True:
            try:
                return fn()  
            except (AssertionError, WebDriverException) as e:
                if time.time() - start_time > MAX_WAIT:
                    raise e
                time.sleep(0.5)
```

기존에 사용하던 `wait_for_row_int_list_table`의 응용버전이다.

그리고 터스트를 실행해본다.

```bash
$ python manage.py test functional_tests.test_list_item_validation
[...]
======================================================================
ERROR: test_cannot_add_empty_list_items
(functional_tests.test_list_item_validation.ItemValidationTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/functional_tests/test_list_item_validation.py", line
15, in test_cannot_add_empty_list_items
    self.wait_for(lambda: self.assertEqual(  
  File "...python-tdd-book/functional_tests/base.py", line 37, in wait_for
    raise e  
  File "...python-tdd-book/functional_tests/base.py", line 34, in wait_for
    return fn()  
  File "...python-tdd-book/functional_tests/test_list_item_validation.py", line
16, in <lambda>  
    self.browser.find_element_by_css_selector('.has-error').text,  
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: .has-error
 ---------------------------------------------------------------------
Ran 1 test in 10.575s

FAILED (errors=1)
```

### FT 종료

`functional_tests/test_list_item_validation.py`

```python
    # The home page refreshes, and there is an error message saying
    # that list items cannot be blank
    self.wait_for(lambda: self.assertEqual(
        self.browser.find_element_by_css_selector('.has-error').text,
        "You can't have an empty list item"
    ))

    # She tries again with some text for the item, which now works
    self.browser.find_element_by_id('id_new_item').send_keys('Buy milk')
    self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy milk')

    # Perversely, she now decides to submit a second blank list item
    self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

    # She receives a similar warning on the list page
    self.wait_for(lambda: self.assertEqual(
        self.browser.find_element_by_css_selector('.has-error').text,
        "You can't have an empty list item"
    ))

    # And she can correct it by filling some text in
    self.browser.find_element_by_id('id_new_item').send_keys('Make tea')
    self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy milk')
    self.wait_for_row_in_list_table('2: Make tea')
```



### 단위테스트를 여러파일로 분리

```bash
$ mkdir lists/tests
$ touch lists/tests/__init__.py
$ git mv lists/tests.py lists/tests/test_all.py
$ git status
$ git add lists/tests
$ python manage.py test lists
[...]
Ran 9 tests in 0.034s

OK
$ git commit -m "Move unit tests into a folder with single file"
```

```bash
$ git mv lists/tests/test_all.py lists/tests/test_views.py
$ cp lists/tests/test_views.py lists/tests/test_models.py
```

그 후 각 테스트파일 주제에 맞는 클래스만 남겨놓는다.

다시 테스트했을 때 같은 수의 테스트가 테스트되어야한다.

```bash
$ python manage.py test lists
[...]
Ran 9 tests in 0.040s

OK
```

