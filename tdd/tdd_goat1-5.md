# TDD Goat



### 유용한 TDD Concepts

### User Story

유저의 관점에서 응용프로그램이 어떻게 작동하는지에 대한 설명. 기능 테스트를 구조화하는데 사용된다.

### Expected failure

우리가 예상한대로 테스트가 실패한다.

### Running the Django dev server

`python manage.py runserver`

### Running the functional tests

`python functional_tests.py`

### Running the unit tests

`python manage.py test`

### The unit-test/code cycle

1. 터미널에서 unit test를 실행
2. 에디터에서 최소한의 코드를 변경
3. 반복!

### 회귀 분석

새 코드가 작동하는데 응용 프로그램의 일부를 망칠 때

### 예기치 않은 오류

테스트에서 실수를 저지르거나, 회귀를 찾는데 도움이 되었다는 것을 의미한다. 코드에서 뭔가 고칠 필요가 있다.

### Red / Green / Refactor

TDD 프로세스를 설명하는 또다른 방법

### 삼각 측량

새로운 특정 예제가 포함된 테스트 사례 추가 구현 일반화를 정당화한다.

### 삼진과 리팩터

코드에서 중복을 제거 할 시기에 대한 경험적 규칙. 코드 두개가 매우 비슷해보이는 경우 세번째 케이스가 보이게되면 함수로 만듬

### 스크래치 패드 할 일 목록

코딩시 일어나는 일을 적어서 하고있는 일을 끝내고 할 목록들



## Install Settings



### Required

[__Firefox web browser__](Firefox web browser and Geckodriver)

[__Git version control system__](Firefox web browser and Geckodriver)

[__vertualenv with Python 3, Djanog 1.11, and Selenium 3__]()

[__Geckodriver__]()



### Firefox web browser and Geckodriver

[FireFox 링크](https://www.mozila.org/firefox/)

[geckodriver 링크](https://github.com/mozilla/geckodriver/releases)

geckodriver 다운로드 후 해당 프로그램을  `/usr/local/bin`에 넣는다.

이후 실행되는지 확인하기위하여 명령어와 버전을 체크한다.

```bash
$ geckodriver --version
```



### 가상환경 설정

pipenv를 사용하여 설정함

[자세한 설명](https://github.com/teachmesomething2580/eb-default-repo)



### Django, Selenium 설치

```bash
$ pip install "django<1.12" "selenium<4"
```

`pip list` 했을 때 Django, Selenium이 존재해야함.



## Basic of TDD and Django

첫 파트는 TDD(Test Driven Development)의 기본사항을 소개한다.

단위테스트뿐 아니라 Selenium의 기능 데스트도 살펴보고 두 테스트의 차이점을 살펴본다. 단위테스트/코드주기라고 부르는 TDD 워크 플로를 소개한다. 또한 리팩토링을 수행하여 이것이 TDD와 어울리는지 확인한다.



### 기능 테스트를 사용하여 장고 세팅하기

TDD의 첫 단계는 항상 동일하다. _테스트를 작성하라._

먼저 테스트를 작성하여 실행 후, 예상대로 실패가 있는지 확인한다.



프로젝트 코드가 있는 곳에 _function_tests.py_ 라는 새로운 Python 파일을 만들고 작은 테스트 코드를 입력해라.

`functional_tests.py`

```python
from selenium import webdriver

browser = webdriver.Firefox()
browser.get('http://localhost:8000')

assert 'Django' in browser.title
```

이는 첫번째 기능 테스트(FT)이다. 기능테스트를 통해 의미하는 바가 무엇인지, 단위 테스트와 어떻게 대비되는지에 대해 뒤에서 자세히 설명한다.

위 코드의 설명은 다음과 같다:

- Selenium "webdriver"를 시작하여 실제 Firefox 브라우저 창을 연다.
- 이를 사용하여 실행되어야할 웹 페이지를 연다.
- 페이지 제목에 "Django"라는 단어가 있는지 검사한다.

테스트 코드를 작성했으면 실행해보자.

```bash
$ python functional_tests.py
Traceback (most recent call last):
  File "functional_tests.py", line 4, in <module>
    browser.get('http://localhost:8000')
  File ".../selenium/webdriver/remote/webdriver.py", line 333, in get
    self.execute(Command.GET, {'url': url})
  File ".../selenium/webdriver/remote/webdriver.py", line 321, in execute
    self.error_handler.check_response(response)
  File ".../selenium/webdriver/remote/errorhandler.py", line 242, in
check_response
    raise exception_class(message, screen, stacktrace)
selenium.common.exceptions.WebDriverException: Message: Reached error page: abo
ut:neterror?e=connectionFailure&u=http%3A//localhost%3A8000/[...]
```

브라우저 창이 나타나고 _localhost:8000_을 열고 "연결할 수 없음" 오류 페이지를 표시한다. 다시 콘솔을 보면 Selenium이 오류 페이지에 도달(Reaced error page) 메세지가 나타난다.

현재로서는 테스트가 실패한 상태이므로 앱을 빌드할 수 없다.



### 장고 설치 및 실행

`$ django-admin.py startproject superlists`

이 명령은 프로젝트 기본 구조를 생성해준다.

```
├── functional_tests.py
├── geckodriver.log
├── manage.py
├── superlists
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── Pipenv
    ├── [...]
```

이제 `manage.py`를 통해 장고 프로젝트를 실행한 후 테스트를 실행해본다.

```bash
$ python manage.py runserver

# 새 터미널을 열고
$ python functional_tests.py
$
```

테스트를 통과하면 아무런 내용도 나오지 않는다. 처음으로 테스트를 통과하게 된것이다.



## unittest 모듈을 사용하여 기능테스트 확장

장고의 "작동 여부" 페이지를 확인하는 테스트를 수정하고 대신 사이트의 실제 페이지에서 확인하고 싶은 내용을 확인해본다.

우리는 _To Do List_ 사이트를 만들게 될것이다. 이는 마감 시간 추가, 알림, 다른 사용자와 공유 및 클라이언트 측 UI 개선과 같은 다양한 방식으로 확장될 수 있다. 요점은 웹 프로그래밍의 모든 주요 측면을 보여줄 수 있어야하고 TDD를 어떻게 적용해야하는지이다.



### 기능테스트를 사용하여 실행 가능한 최소 응용프로그램 범위 지정

Selenium을 사용하는 테스트를 통해 실제 웹 브라우저를 구동할 수 있으므로 실제 응용 프로그램이 이 사용자 관점에서 어떻게 작동하는지 확인할 수 있다. 이것이 _기능 테스트_라고 불리는 이유이다.

이는 FT가 응용 프로그램에 대한 일종의 규격이 될 수 있음을 의미한다. _사용자 스토리(User Story)_라고 부르는 것을 추적하는 경향이 있으며 사용자가 특정 기능을 어떻게 사용할 수 있는지 그리고 앱이 그들에게 어떻게 반응해야하는지에 따른다.

> ### 용어:
>
> #### 기능 테스트 == 수락 테스트 == 엔드 - 투 - 엔드 테스트
>
> 기능 테스트라 부르는 일부 사람들은 _수락 테스트_ 또는 _엔드 투 엔드 테스트_를 선호한다. 요점은 이러한 테스트가 전체 애플리케이션이 외부에서 어떻게 작동하는지 살펴보는 것이다. 또 다른 용어는 _블랙 박스 테스트_이다. 테스트에서는 테스트중인 시스템 내부에 대해 아무것도 모르기 때문이다.

FT는 인간이 읽을 수 있는 이야기를 따라야한다. 테스트 코드와 함께 제공되는 주석을 사용하여 명시적으로 지정한다. 새 FT를 만들 때 사용자 스토리의 핵심 사항을 파악하기 위해 먼저 주석을 작성한다. 사람이 읽을 수 있도록 앱의 요구 사항과 기능을 토론하는 방법으로 비 프로그래머와 공유할수도 있다.

TDD와 agile 소프트웨어 개발 방법론은 종종 함께 진행되고, 우리가 자주 이야기하는 것들 중 하나는 _실행 가능한 최소 응용 프로그램_이다.



현재 작성해야할 최소 기능은 _할 일 목록을 입력_ 하고 _다음 방문을 위해 기억_ 할 수 있도록 해야한다.

_functional_tests.py_를 열고 다음과 같은 Story를 작성한다.

`functional_tests.py`

```python
from selenium import webdriver

browser = webdriver.Firefox()

# Edith는 멋진 to-do app이 있다는 소문을 듣고
# 홈페이지를 접속한다.
browser.get('http://localhost:8000')

# 그녀는 페이지 제목과 헤더가 to-do 리스트를 언급하고 있는지 확인한다.
assert 'To-Do' in browser.title

# 그녀는 바로 To-Do 입력창을 눌렀다.

# 그녀는 텍스트 상자에 "Buy peacock feathers"를 입력한다. (Edith의 취미는 플라잉 낚시 미끼 묶기이다.)

# 그녀가 엔터를 치면 페이지가 업데이트되고 페이지가 표시된다.
# "1: Buy peacock feathers"가 To-Do List의 아이템이 된다.

# 여전히 다른 항목을 추가할 수 있는 텍스트상자가 있다.
# 그녀는 "Use peacock feathers to make a fly"를 입력한다. (Edith는 매우 조직적이다.)

# 페이지가 다시 업데이트되고, 목록에 두 항목이 모두 표시된다.

# Edith는 사이트가 그녀의 할일 목록을 기억할것인지 궁굼해한다.
# 설명란에 사이트에서 고유한 URL을 생성했음을 알린다.

# 그녀는 URL로 방문한다. 그녀의 to-do list가 여전히 존재한다.

# 만족스럽게, 그녀는 잠에 빠진다.

browser.quit()
```

`assert`로 "Django" 대신 "To-Do"라는 단어를 찾도록 업데이트 했다. 이는 우리가 시험이 실패할 것을 기대한다는 것을 의미한다. 실행시켜보자.

```bash
$ python functional_tests.py
Traceback (most recent call last):
  File "functional_tests.py", line 10, in <module>
    assert 'To-Do' in browser.title
AssertionError
```

이는 우리가 _기대하는 실패_라고 부른다. 실제로 좋은 소식이다. 통과하는 테스트만큼 좋지는 않지만, 적어도 적절한 이유로 실패하고있다. 우리는 테스트를 올바르게 작성했다고 확신한다.



### 파이썬 표준라이브러리의 unittest 모듈

사실 "AssertionError" 메세지는 별로 도움되지 않는다. 테스트에서 브라우저 제목으로 실제로 무엇이 작성되어져있는지 알려주면 좋을것이다.



첫 옵션은 `assert` 키워드의 두번째 파라미터를 사용하는 것이다.

```python
assert 'To-Do' in browser.title "Browser title was" + browser.title
```

그리고 이전 파이어폭스 창을 닫기 위해 `try/finally`를 사용할수도 있다. 하지만 이러한 종류의 문제는 테스트에서 흔히 볼 수 있으며 이는 표준 라이브러리의 모듈에 이미 준비되어있는 솔루션인 `unittest`가 있다.

`functional_tests.py`

```python
from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):
    
    def setUp(self):
        self.browser = webdriver.Firefox()
        
    def tearDown(self):
        self.browser.quit()
        
    def test_can_start_a_list_and_retrieve_it_later(self):
        # Edith는 멋진 to-do app이 있다는 소문을 듣고
        # 홈페이지를 접속한다.
        self.browser.get('http://localhost:8000')

        # 그녀는 페이지 제목과 헤더가 to-do 리스트를 언급하고 있는지 확인한다.
        self.assertIn('To-Do', self.browser.title)
        self.fail('Finish the test!')
        
        # 그녀는 바로 입력창을 선택했다.
        [...위에서 적은 남은 항목들을 생략]
        
if __name__ == '__main__':
    unittest.main(warnings='ignore')
```

여기서 몇 가지 사실을 알게된다.

1. 테스트는 클래스로 구성되며 상속된다. `unittest.TestCase`
2. 테스트 본문에 `test_can_start_a_list_and_retrieve_it_later`라는 테스트를 호출한다. `test`가 붙여진 모든 메서드는 테스트된다. 클래스에 하나 이상의 `test_` 메서드를 가질 수 있다. 테스트메서드의 이름에 좋은 설명을 붙이는것도 좋은 방법이다.
3. `setUp`, `tearDown`은 각 테스트 전 후 실행되는 특별 메서드이다. 브라우저를 시작하고 멈추기위해 사용한다. 테스트중 에러가 발생했을 때 `tearDown`이 실행된다는 점에서 `try/finally`와 비슷하다. 더이상 Firefox창이 남을 일은 없다!
4. test assertion을 위해 `assert` 대신 `self.assertIn`을 사용했다. `unittest`는 `assertEqual`, `assertTrue`, `assertFalse`와 같은 test assertion을 만들기위해 많은 헬퍼 함수를 제공한다. [unittest documentation]()에서 더 많은 정보를 찾을 수 있다.
5. `self.fail`은 오류 메세지를 생성하면서 무엇을 해도 실패한다. 시험을 마치기 위한 알림으로 사용하고있다.
6. 마지막으로 `if __name__ == '__main__'`을 가지고 있다. (다른 모듈에서 이 스크립트를 가져올때는 실행되지 않고 직접 실행되었을 때만 실행된다.) `unittest.main()`을 호출하여 unittest를 테스트를 시작한다. unittest 테스트 러너는 파일에서 테스트 클래스와 메서드를 자동으로 찾아 실행한다.

7. `warnings='ignore`는 글쓰기를 할 때 불필요한 `ResourceWarning`을 억제한다.

이제 실행해보자

```bash
$ python functional_tests.py
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 18, in
test_can_start_a_list_and_retrieve_it_later
    self.assertIn('To-Do', self.browser.title)
AssertionError: 'To-Do' not found in 'Welcome to Django'

 ---------------------------------------------------------------------
Ran 1 test in 1.747s

FAILED (failures=1)
```

좀 더 좋아보인다. 파이어 폭스창을 정리하고, 얼마나 많은 테스트가 실행되었고 얼마나 많은 테스트가 실패했는지에 대한 형식화된 보고서가 제공되며 `assertIn`은 유용한 디버깅 정보가 포함된 유용한 오류 메세지를 제공한다.



## 단위 테스트를 사용하여 간단한 홈페이지 테스트

### 첫 번째 장고 앱 및 첫 번째 단위 테스트

Django는 코드를 _앱_으로 구성 할 것을 권장한다. 하나의 프로젝트가 많은 앱을 가질 수 있고, 다른 사람들이 개발한 타사 앱을 사용할 수 잇으며, 다른 프로젝트에서 자신의 앱 중 하나를 재사용 할 수도 있다. 

To-Do List를 위한 앱을 시작한다.

```bash
$ python manage.py startapp lists
```

그러면 `manage.py`와 `superlists` 폴더 옆 `lists`라는 폴더가 생기고  그 안에모델, 뷰, 그리고 우리에게 직접적으로 관심이 있는 테스트파일 같은 것들이 생성된다.

```
.
├── db.sqlite3
├── functional_tests.py
├── lists
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── superlists
│   ├── __init__.py
│   ├── __pycache__
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── Pipenv
    ├── [...]
```



### 단위 테스트, 어떻게 기능 테스트와 다른가

우리가 많은 레이블을 붙인것과 마찬가지로, 단위 테스트와 기능 테스트 사이의 경계는 때때로 약간 흐려질 수 있다.  기본 구별은 기능 테스트가 사용자의 관점에서 외부 애플리케이션을 테스트한다는 것이다. 단위 테스트는 프로그래머의 관점에서 애플리케이션을 내부에서 테스트한다.

TDD 접근 방식은 두가지 유형의 테스트에서 응용프로그램을 다루기를 원한다. 우리의 워크 플로는 다음과 같다.

1. 먼저 사용자의 관점에서 새 기능을 설명하는 기능 테스트를 작성한다.
2. 기능 테스트가 실패하면 성공하는 코드를 작성하는 방법에 대해 생각한다. (또는 적어도 현재 실패를 지나치던가) 우리는 이제 하나 이상의 단위 테스트를 사용하여 코드가 작동하는 방식을 정의한다. 우리가 작성한 각 줄의 프로덕션 코드는 적어도 하나 이상의 단위 테스트에서 테스트되어야한다.
3. 일단 실패한 단위 테스트를 얻으면 단위 테스트를 통과하기에 충분할 정도로 작은 양의 응용 프로그램 코드를 작성한다. 기능 테스트가 조금 더 발전할 것이라고 생각할 때까지 2 ~ 3 단계를 몇번 더 반복할 수 있다.
4. 이제 기능 테스트를 다시 수행하여 테스트가 통과하는지 확인하거나 조금 더 테스트할 수 있다. 새로운 단위 테스트와 몇가지 새로운 코드 등을 쓸 수 있다.

단위 테스트는 우리가 저수준에서 하는 일을 주도하는 반면, 기능 테스트는 우리가 개발한 것을 높은 수준에서 이끌어낼 수 있는 것을 볼 수 있다.

약간 중복된 것 같나? 때로 그런식으로 느낄 때도 있지만, 기능 테스트와 유닛 테스트는 실제로 매우 다른 목표를 가지고 있으며, 대게 완전히 다르게 보일 것이다.



### Unit Testing in Django

우리가 홈페이지를 보기 위한 단위 테스트를 작성하는 방법을 살펴본다. `lists/test.py`를 열면 다음과 같은 내용이 표시된다.

`lists/tests.py`

```python
from django.test import TestCase

# Create your tests here.
```

Django는 TestCase의 특별한 버전을 사용하라고 제안했고, 제공한다. 이는 `unittest.TestCase`의 증강된 버전이며 부가적인 Django 고유의 기능을 가지고 있다.

이미 TDD 사이클에는 실패한 테스트로 시작한 다음 성공하기 위한 코드를 작성하는 것이 포함되어 있다. 음, 우리가 그정도까지 가기 전에, 우리가 쓰고 있는 단위 테스트가 무엇이던간에 우리의 자동화된 테스트 러너에 의해 실행될 것이라는 것을 알고싶다. `functional_tests.py`의 경우 직접 실행하지만 Django가 만든 파일은 좀 더 마술과 같다. 따라서 확실하게 하기 위해 고의적으로 실패 테스트를 만들어본다.

`lists/tests.py`

```python
from django.test import TestCase

class SmokeTest(TestCase):

    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
```

이제 신비한 Django 테스트 러너를 호출한다.

```bash
$ python manage.py test
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_bad_maths (lists.tests.SmokeTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 6, in test_bad_maths
    self.assertEqual(1 + 1, 3)
AssertionError: 2 != 3

 ---------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

좋다. 잘 작동하는 것 같다. 이는 좋은 커밋 포인트이다.



### Django의 MVC, URLs, 그리고View Functions

Django는 고전적인 _Model-View-Controller(MVC)_패턴으로 구성되어 있다. model은 같지만 View는 Templates이고 Controller는 View와 비슷하다.

웹 서버와 마찬가지로 Django의 주요 임무는 사용자가 사이트에서 특정 URL을 요청할 때 수행 할 작업을 결정하는 것이다. Django의 워크 플로는 다음과 같다.

1. 특정 URL에 대한 HTTP 요청이 온다.
2. Django는 몇가지 규칙을 사용하여 요청을 처리해야하는 VIEW를 결정한다. (resolving URL)
3. view 함수는 요청을 처리하고 HTTP response를 리턴한다.

그래서 우리는 두가지를 테스트하길 원한다.

- 우리가 만든 특정 View 기능에 루트 URL("/")을 사용할 수 있나?
- 이 View 함수가 기능 테스트를 통과할 일부 HTML을 반환하도록 할 수 있나?

첫번째부터 시작한다. `lists/tests.py`를 열고 테스트를 변경한다.

`lists/tests.py`

```python
from django.urls import resolve
from django.test import TestCase
from lists.views import home_page

class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)
```

무엇이 일어나는가?

1. `resolve`함수는 Django가 내부적으로 URL을 분석하고 그들이 매핑해야하는 View 함수를 찾기위해 내부적으로 사용된다. 사이트의 루트인 "/"와 함께 호출될 때  `home_page`라는 함수를 찾는다.
2. `home_page`는 무슨 함수인가? 다음에 작성하고자하는 View 함수이다. 실제로 우리가 원하는 HTML을 반환한다. 임프트를 봐서 우리가 `lists/views.py`에 저장하려고한다는 것을 알 수 있다.

다시 Test를 실행한다.

```bash
$ python manage.py test
ImportError: cannot import name 'home_page'
```

예측가능한 오류가 발생했다. 우리는 아직쓰지 않은 것을 import하려 했다. 이는 여전히 희소식이다.  TDD 목적상 예상된 예외는 예상된 실패로 간주한다.



### 마지막으로!  실제로 일부 애플리케이션 코드를 작성한다.

실패한 코드를 해결하기위해  `lists.views`의  `home_page`를 작성한다.

`lists/views.py`

```python
from django.shortcuts import render

# Create your views here.
home_page = None
```

지금 당장은 위의 코드가 혼란스러울지 몰라도, 우리의 테스트가 올바른 코드를 작성하는 데 도움이 될 수 있다.



테스트를 다시 실행한다.

```bash
$ python manage.py test
Creating test database for alias 'default'...
E
======================================================================
ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 8, in
test_root_url_resolves_to_home_page_view
    found = resolve('/')
  File ".../django/urls/base.py", line 27, in resolve
    return get_resolver(urlconf).resolve(path)
  File ".../django/urls/resolvers.py", line 394, in resolve
    raise Resolver404({'tried': tried, 'path': new_path})
django.urls.exceptions.Resolver404: {'tried': [[<RegexURLResolver
<RegexURLPattern list> (admin:admin) ^admin/>]], 'path': ''}

 ---------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (errors=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

> ### Reading Tracebacks
>
> 트레이스 백을 읽는 방법에 대해 잠깐 이야기한다. TDD에서 해야할 일이 많기 때문이다. 이를 스캔하고 관련된 단서를 찾는 방법을 배운다.
>
> ```bash
> ======================================================================
> ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)  
>  ---------------------------------------------------------------------
> Traceback (most recent call last):
>   File "...python-tdd-book/lists/tests.py", line 8, in
> test_root_url_resolves_to_home_page_view
>     found = resolve('/')  
>   File ".../django/urls/base.py", line 27, in resolve
>     return get_resolver(urlconf).resolve(path)
>   File ".../django/urls/resolvers.py", line 394, in resolve
>     raise Resolver404({'tried': tried, 'path': new_path})
> django.urls.exceptions.Resolver404: {'tried': [[<RegexURLResolver  
> <RegexURLPattern list> (admin:admin) ^admin/>]], 'path': ''}  
>  ---------------------------------------------------------------------
> [...]
> ```
>
> 1. `django.urls.exceptions.Resolver404: {'tried': [[<RegexURLResolver  
>    <RegexURLPattern list> (admin:admin) ^admin/>]], 'path': ''}  `
>
>    이는 오류 자체이다. 때로는 이게 볼 수 있는 전부인데, 이는 문제를 즉시 확인할 수 있게 해줄것이다. 하지만 때때로 이 경우처럼 자명한것은 아니다.
>
> 2. `ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)  `
>
>    다음 확인해야할것은 어떤 테스트가 실패하는지다. 우리가 기대했던 것인가? 즉, 우리가 방금 쓴 것인가? 이 경우에, 대답은 '그렇다'이다.
>
> 3. `Traceback (most recent call last):
>      File "...python-tdd-book/lists/tests.py", line 8, in
>    test_root_url_resolves_to_home_page_view
>    ​    found = resolve('/')  `
>
>    그리고나서 우리는 테스트코드에서 실패한 줄을 찾는다. 테스트 파일의 파일 이름을 찾고, 어떤 테스트 기능과 어떤 코드 라인에서 오류가 발생하는지 확인하기위해 추적 파일의 상단에서 아래로 내려간다.
>
>    이 경우 "/" URL에 대한 resolve 기능을 호출하는 라인이다.
>
> 일반적으로 네 단계가 있는데, 여기서 우리 문제와 관련이 있는 우리 자신의 응용프로그램 코드를 더 내려다보게된다. 이 경우 장고 코드이지만, 이 책 뒷부분에서 네 번째 단계에 대한 예제를 보게 될 것 이다.
>
> 모든 것을 한데 모으고, 우리는 "/"를 해결하려고 할 때 장고가 404오류를 발생시켰다고 해석한다.즉, 장고는 "/"에 대한 URL 매핑을 찾을 수 없다.



#### urls.py

우리의 테스트는 URL 매핑이 필요하다는 것을 알려준다. Django는 urls.py라는 파일을 사용하여 URL을 view 함수에 매핑한다.`superlists/superlists` 폴더에 전체 사이트에 대한 기본 urls.py가 있다.

`superlists/urls.py`

```python
from django.conf.urls import url
from lists import views

urlpatterns = [
    url(r'^$', views.home_page, name='home'),
]
```

위와같이 작성하고 다시 테스트를 실행한다.

```bash
[...]
TypeError: view must be a callable or a list/tuple in the case of include().
```

잘 동작한다. 더이상 404가 에러가 나지 않는다.

traceback은 복잡하나 마지막에 우리에게 무슨일이 일어나고 있는지 알려준다. 즉, 단위 테스트는 실제로 URL "/"과 `home_page = None`을 `lists/views.py`의 링크로 만들었다. 그리고 현재 `home_page` view를 호출할 수 없다고한다. 이는 우리가 그것을 None에서 실제 기능으로 바꾸는 것에 대한 정당성을 준다. 모든 단일 코드 변경은 테스트를 통해 이루어진다!

다시 `lists/views.py`

`lists/views.py`

```python
from django.shortcuts import render

# Create your views here.
def home_page():
    pass
```

그리고 테스트

```bash
$ python manage.py test
Creating test database for alias 'default'...
.
 ---------------------------------------------------------------------
Ran 1 test in 0.003s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

처음으로 유닛 테스트를 통과했다!



### Unit Testing a View

View에 대한 테스트를 작성하여 아무것도 하지 않는 기능이 될 수 있도록 HTML Response를 브라우저로 되돌려주는 기능을 작성한다. `lists/tests.py`를 열고 새 test method를 작성한다.

`lists/tests.py`

```python
from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest

from lists.views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)


    def test_home_page_returns_correct_html(self):
        request = HttpRequest()  
        response = home_page(request)  
        html = response.content.decode('utf8')  
        self.assertTrue(html.startswith('<html>'))  
        self.assertIn('<title>To-Do lists</title>', html)  
        self.assertTrue(html.endswith('</html>'))  
```

1. `request = HttpRequest()  `

   `HttpRequest` 객체를 생성했다. 이 객체는 사용자 브라우저 페이지를 요청할 때 Django가 볼 수 있는 객체이다.

2. `response = home_page(request)`

   `home_page` view에 전달하여 응답을 받는다. 이 객체가 `HttpResponse`라는 클래스 인스턴스가 올것이다.

3. `html = response.content.decode('utf8')`

   Response의 내용을 추출한다. 이는 원시 바이트, 사용자의 브라우저로 전송되는 1과 0이다. `.decode`를 호출하여 사용자에게 보내지는 HTML 문자열로 변환한다.

4. `self.assertTrue(html.startswith('<html>'))`, `self.assertTrue(html.endswith('</html>'))`

   `<html>`로 시작하고 `</html>`로 끝나는 내용이길 원한다.

5. 그리고 중간 어딘가에 `<title>`태그가 필요하다. 이는 기능테스트에서 지정한 것이기 때문에 "To-Do lists"라는 단어를 사용한다.

다시한번, 단위 테스트는 기능 테스트에 의해 주도되지만 실제 코드에 훨씬 가깝다. 이제 우리는 프로그래머처럼 생각한다.

다시실행했을 때 다음의 결과를 보게된다.

```
TypeError: home_page() takes 0 positional arguments but 1 was given
```

### 단위 테스트/ 코드 주기

이제 TDD 단위 테스트/ 코드 사이클에 착수 할 수 있다.

1. 터미널에서 단위 테스트를 실행하고 실패를 확인하라.
2. 편집기에서 현재 테스트 실패를 해결하기 위한 최소한의 코드 변경을 하라.

이를 반복하자.

우리가 코드를 올바르게 작성하는 것에 신경을 쓰면 작을수록 코드가 조금식 변한다. 이는 각 코드가 테스트에 의해 정당화된다는 것을 절대적으로 확신할 수 있게된다.



이제 이 사이클을 얼마나 빨리 진행할 수 있는지 보자.

- 최소 코드 변경:

`lists/views.py`

```python
def home_page(request):
    pass
```

- 테스트:

```bash
html = response.content.decode('utf8')
AttributeError: 'NoneType' object has no attribute 'content'
```

- 코드 - `django.http.HttpResponse` 를 사용한다.

`lists/views.py`

```python
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse()
```

- 다시 테스트:

```bash
    self.assertTrue(html.startswith('<html>'))
AssertionError: False is not true
```

- 다시 코드:

`lists/views.py`

```python
def home_page(request):
    return HttpResponse('<html>')
```

- 테스트:

```bash
AssertionError: '<title>To-Do lists</title>' not found in '<html>'
```

- 코드:

`lists/views.py`

```python
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title>')
```

- 테스트-- 거의다 왔슴

```bash
    self.assertTrue(html.endswith('</html>'))
AssertionError: False is not true
```

- 마지막 코드:

`lists/views.py`

```python
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')
```

- 정말?

```bash
$ python manage.py test
Creating test database for alias 'default'...
..
 ---------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

이제 다시 기능 테스트를 실행해보자. 서버를 실행한 후, 테스트를 해본다.

```bash
$ python functional_tests.py
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 19, in
test_can_start_a_list_and_retrieve_it_later
    self.fail('Finish the test!')
AssertionError: Finish the test!

 ---------------------------------------------------------------------
Ran 1 test in 1.609s

FAILED (failures=1)
```

실패가 발생했지만 이는 우리가 발생시킨 완료 신호이다.



## 이 테스트를 통해 우리는 무엇을 하고있나? (그리고 리팩토링)

[TDD에 관한 불만](https://www.obeythetestinggoat.com/book/chapter_philosophy_and_refactoring.html)

### Selenium을 사용하여 사용자 상호 작용 테스트

`functional_tests.py`

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
import unittest

class NewVisitorTest(unittest.TestCase):

    def setUp(self):
        self.browser = webdriver.Firefox()

    def tearDown(self):
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        # Edith는 멋진 to-do app이 있다는 소문을 듣고
		# 홈페이지를 접속한다.
        self.browser.get('http://localhost:8000')

        # 그녀는 페이지 제목과 헤더가 to-do 리스트를 언급하고 있는지 확인한다.
        self.assertIn('To-Do', self.browser.title)
        header_text = self.browser.find_element_by_tag_name('h1').text  
        self.assertIn('To-Do', header_text)

        # 그녀는 바로 To-Do 입력창을 눌렀다.
        inputbox = self.browser.find_element_by_id('id_new_item')  
        self.assertEqual(
            inputbox.get_attribute('placeholder'),
            'Enter a to-do item'
        )

        # 그녀는 텍스트 상자에 "Buy peacock feathers"를 입력한다. (Edith의 취미는 플라잉 낚시 미끼 묶기이다.)
        inputbox.send_keys('Buy peacock feathers')  

        # 그녀가 엔터를 치면 페이지가 업데이트되고 페이지가 표시된다.
		# "1: Buy peacock feathers"가 To-Do List의 아이템이 된다.
        inputbox.send_keys(Keys.ENTER)  
        time.sleep(1)  

        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')  
        self.assertTrue(
            any(row.text == '1: Buy peacock feathers' for row in rows)
        )

        # 여전히 다른 항목을 추가할 수 있는 텍스트상자가 있다.
		# 그녀는 "Use peacock feathers to make a fly"를 입력한다. (Edith는 매우 조직적이다.)
        self.fail('Finish the test!')

        # 페이지가 다시 업데이트되고, 목록에 두 항목이 모두 표시된다.
        [...]
```

1. Selenium을 사용하여 웹 페이지를 검사하기 위해 제공하는 방법 중 몇가지를 사용하고있다. (`find_element_by_tag_name`, `find_element_by_id`)
2. 또한 `send_keys`를 사용하여 타이핑한다.
3. `Keys` 클래스를 가져와 특수키를 보낼 수 있다.
4. Enter를 누르면 페이지가 새로고침된다. `time.sleep`은 새 페이지에 대한 검사 전 브라우저 로드가 완료되기를 확인한다. 이는 "명시적 대기"라고 한다.



`any`라는 built-in 함수를 사용하여 To-do list의 내용을 검사한다.

다시 테스트

```bash
$ python functional_tests.py
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: h1
```

`<h1>`요소를 찾을 수 없다고 나타난다. 이후 HTML에 추가할 수 있는 작업을 살펴본다.



### "상수를 테스트하지 마라" 규칙 및 구조에 대한 템플릿

단위 테스트인 `lists/tests.py`를 살펴본다. 현재 특정 HTML 문자열을 찾고있지만 HTML을 테스트하는데 특히 효율적이진 않다. 일반적으로 단위 테스트 규칙 중 하나는 상수를 테스트하지 않고 HTML을 텍스트로 테스트하는 것은 상수를 테스트하는 것과 같다.

즉 다음과 같은 코드가 있는 경우:

```python
wibble = 3
```

다음과 같은 테스트에는 별로 중요하지 않다.

```python
from myprogram import wibble
assert wibble == 3
```

단위 테스트는 실제 로직, 흐름제어 및 구성 테스트에 관한 것이다. HTML 문자열에 가지고 있는 문자의 순서가 정확히 무엇인지에 대한 테스트가 아니다.



### 템플릿을 사용하는 리팩토링

지금 하고싶은 것은 뷰 함수가 정확하게 동일한 HTML을 반환하지만 다른 프로세스를 사용하는 것이다. 리팩토링 - _기능을 변경하지 않고_ 코드를 개선하려고 할 때.

리팩토링과 동시에 새 기능을 추가할 시 문제가 발생할 확률이 높다. 리팩토링은 사실 그 자체로 분야이며, 도서도 있다.

첫 규칙은 테스트없이 리팩토링 불가하다는 것이다. 고맙게도 우리는 TDD를 하고있다.

`lists/templates`라는 폴더에 `lists/templates/home.html`을 작성한다.

`lists/templates/home.html`

```html
<html>
    <title>To-Do lists</title>
</html>
```

그리고 view를 바꾼다.

`lists/views.py`

```python
from django.shortcuts import render

def home_page(request):
    return render(request, 'home.html')
```

`HttpResponse`대신 `render`함수를 사용한다.

그 후 테스트를 확인한다.

```bash
$ python manage.py test
[...]
======================================================================
ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 17, in
test_home_page_returns_correct_html
    response = home_page(request)
  File "...python-tdd-book/lists/views.py", line 5, in home_page
    return render(request, 'home.html')
  File "/usr/local/lib/python3.6/dist-packages/django/shortcuts.py", line 48,
in render
    return HttpResponse(loader.render_to_string(*args, **kwargs),
  File "/usr/local/lib/python3.6/dist-packages/django/template/loader.py", line
170, in render_to_string
    t = get_template(template_name, dirs)
  File "/usr/local/lib/python3.6/dist-packages/django/template/loader.py", line
144, in get_template
    template, origin = find_template(template_name, dirs)
  File "/usr/local/lib/python3.6/dist-packages/django/template/loader.py", line
136, in find_template
    raise TemplateDoesNotExist(name)
django.template.base.TemplateDoesNotExist: home.html

 ---------------------------------------------------------------------
Ran 2 tests in 0.004s
```

traceback을 분석할 수 있는 또다른 기회:

1. `django.template.base.TemplateDoesNotExist : home.html`

   먼저 템플릿을 찾을 수 없습니다.

2. `ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest)`

   어떤 테스트가 실패했는지 확인

3. `Traceback (most recent call last):
     File "...python-tdd-book/lists/tests.py", line 17, in
   test_home_page_returns_correct_html
   ​    response = home_page(request)`

   실패의 원인이 되는 행을 찾음

4. 마지막으로 실패의 원인이되는 자체 애플리케이션 코드의 일부를 찾음, `render` 우리가 호출하려할 때이다.



이 때 장고 템플릿을 찾을 수 없는 이유는 아직 Django에 앱을 _공식적으로_ 등록하지 않았기 때문이다.`settings.py`의 `INSTALLED_APPS`에 추가해야 Template을 찾을 수 있다.

`superlists/settings.py`

```python
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'lists',
]
```

되는 사람도 있겠지만 안될 수도 있다. 에디터가 파일 끝 부분에 줄 바꿈을 추가할지 여부에 따라 오류가 발생할 수 있다.

```bash
$ python manage.py test 
    [...] 
    self.assertTrue (html.endswith ( '</ html>')) 
AssertionError : False가 맞지 않습니다.
```

실패했다면 디버깅을 해보고 마지막 줄바꿈이 있는 것을 확인하고 줄바꿈을 지운다.

`lists/tests.py`

```python
self.assertTrue(html.strip().endswith('</html>'))
```

```bash
$ python manage.py test 
[...] 
OK
```

코드의 리팩터링이 완료되었으며 테스트를 통해 동작이 유지된다는 사실에 만족한다는 것을 의미한다. 더 이상 상수를 테스트하지 않도록 테스트를 변경할 수 있다. 대신, 올바른 템플릿을 렌더링하고 있는지 확인해야한다.



### 장고 테스트 클라이언트

테스트 방법중 수동으로 템플릿을 테스트에 렌더링한다음 이 뷰가 반환하는 것을 비교하는 것이다. 장고에서 `render_to_string`은 위 파이썬에서 html을 string으로 가져와준다.

`lists/test.py`

```python
from django.template.loader import render_to_string
[...]

    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        html = response.content.decode('utf8')
        expected_html = render_to_string('home.html')
        self.assertEqual(html, expected_html)
```

위의 내용은 너무 산만해서 Django는 [Django Test Client]()라는 도구를 제공한다.

`lists/tests.py`

```python
    def test_home_page_returns_correct_html(self):
        response = self.client.get('/')  

        html = response.content.decode('utf8')  
        self.assertTrue(html.startswith('<html>'))
        self.assertIn('<title>To-Do lists</title>', html)
        self.assertTrue(html.strip().endswith('</html>'))

        self.assertTemplateUsed(response, 'home.html')  
```

1. 수동으로 `HttpRequest` 객체를 만들고 뷰 함수를 직접 호출하는 대신, `self.client.get` 테스트할 URL을 호출한다.
2. 모든 테스트가 제대로 작동하는지를 위해 이전 테스트를 그대로 두었다.
3. `assertTemplateUsed`는 Django TestCase 클래스가 제공하는 테스트 메서드이다. 이를 통해 응답을 렌더링하는데 사용된 템플릿이 확인 가능하다. (주의: 테스트 클라이언트가 검색한 응답에 대해서만 작동한다.)

오류를 보기위해 `wrong.html`으로 확인해보자.

`lists/tests.py`

```python
        self.assertTemplateUsed(response, 'wrong.html')
```

```bash
AssertionError: False is not true : Template 'wrong.html' was not a template
used to render the response. Actual template(s) used: home.html
```

다시 정상적으로 바꾸어주고 `test_root_url_resolves`의 테스트는 현재 테스트 검사하는것과 겹치기 때문에 두개를 하나로 바꿔준다.

```python
from django.test import TestCase

class HomePageTest(TestCase):

    def test_uses_home_template(self):
        response = self.client.get('/')
        self.assertTemplateUsed(response, 'home.html')
```



### 프론트 페이지를 조금 더

그동안 우리의 기능 테스트는 여전히 실패한다. 이제 코드를 변경하여 통과시켜보자. HTML이 이제 테스트에 포함되어있으므로 추가 단위 테스트를 작성할 필요 없이 자유로이 템플릿을 변경할 수 있다.



`lists/templates/home.html`

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
    </body>
</html>
```

테스트

```bash
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_new_item"]
```

수정...

`lists/templates/home.html`

```html
    [...]
        <h1>Your To-Do list</h1>
        <input id="id_new_item" />
    </body>
    [...]
```

그리고 나면?

```bash
AssertionError: '' != 'Enter a to-do item'
```

placeholder 텍스트도 추가하자..

`lists/templates/home.html`

```html
    <input id="id_new_item" placeholder="Enter a to-do item" />
```

결과는?

```bash
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_list_table"]
```

이제 테이블을 만들자.

`lists/templates/home.html`

```html
	<input id="id_new_item" placeholder="Enter a to-do item" />
    <table id="id_list_table">
    </table>
</body>
```

이제 FT가 뭐라 말하나?

```bash
  File "functional_tests.py", line 43, in
test_can_start_a_list_and_retrieve_it_later
    any(row.text == '1: Buy peacock feathers' for row in rows)
AssertionError: False is not true
```

이를 추적하기위해 번호를 사용한다. 이 오류는 `any`이전 `assertTrue`에 관한 오류였으며, 사용자 정의 오류 메세지를 인수로 전달해서 더 확실히 한다.

`functional_tests.py`

```python
    self.assertTrue(
        any(row.text == '1: Buy peacock feathers' for row in rows),
        "New to-do item did not appear in table"
    )
```

FT를 다시 실행하면 다음 메세지가 표시된다.

```bash
AssertionError: False is not true : New to-do item did not appear in table
```

이제 이 문제를 해결하기위해 사용자 form을 실제로 처리해야한다.



### 요약: TDD 프로세스

이제 실제로 TDD 프로세스의 모든 주요 측면을 살펴보았다.

- 기능 테스트
- 단위 테스트
- 단위 테스트 / 코드 주기
- 리팩토링

테스트를 실행하고 실패한 것을 본다. 이를 해결하기위해 최소한의 코드를 작성한다. 모든 테스트를 통과할 때 까지 이를 반복한다. 다음으로 선택적으로 코드 리팩터링할 때 테스트를 사용하여 기능 무결성을 지킬 수 있다.

기능 테스트와 단위 테스트가 있을 때 테스트 우선순위는 어떻게 될까? 우선 기능 테스트를 작성하고 실패했는지 확인한다. 그 후 하나 이상의 단위 테스트를 작성하고 단위 테스트가 통과할 때까지 단위 테스트 / 코드 사이클을 진행한다. 그 다음 FT로 돌아가 조금 더 나아갔는지 확인하고 더 많은 단위 테스트를 사용하여 응용 프로그램을 작성해나간다.



## 사용자 입력 저장: 데이터베이스 테스트

이제 입력받은 데이터를 저장하고 테스트하는 방법을 알아본다.

### POST 요청을 보내기 위해 HTML FORM 작성

브라우저에서 POST 요청을 보내기 위해 두가지 작업을 수행해야한다.

1. 주고받는 `<input>` 요소에 `name=` 속성
2. `<form>` 태그로 포장 후 `method=POST`

`lists/templates/home.html`

```html
<h1>Your To-Do list</h1>
<form method="POST">
    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
</form>

<table id="id_list_table">
```

이제 FT를 실행하면 예기치 않은 오류가 발생한다. 이런상황에서의 디버깅은 아래 방법으로 실행한다.

- 예를 들어, 현재 페이지가 텍스트가 무엇인지 보여주는 명령문을 추가한다.
- _오류 메세지_를 개선하여 현재 상태에 대한 자세한 정보를 표시한다.
- 수동으로 직접 사이트를 방문한다.
- 실행 중 `time.sleep`으로 테스트를 일시 중지하는데 사용한다.

FT를 조금더 확장한다.

`functional_tests.py`

```python
    # When she hits enter, the page updates, and now the page lists
    # "1: Buy peacock feathers" as an item in a to-do list table
    inputbox.send_keys(Keys.ENTER)
    time.sleep(10)

    table = self.browser.find_element_by_id('id_list_table')
```

이후 FT를 재실행하면 403 Forbidden 페이지가 나타난다. Django는 기본적으로 CSRF 보호를 위해 CSRF 토큰을 사용해야한다. 이를 해결하려면 폼안에  `{% csrf_token %}`를 사용해야한다.

`lists/templates/home.html`

```html
<form method="POST">
    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
    {% csrf_token %}
</form>
```

다시 테스트

```bash
AssertionError: False is not true : New to-do item did not appear in table
```

디버깅을 끝냈으니 디버깅 코드인 `time.sleep(10)`을 원래대로 돌려놓는다.



### 서버에서 POST 처리

`action=` 속성을 지정하지 않았으므로 기본적으로 렌더링 된 url(즉, `/`)과 동일한 URL로 제출한다. URL은 `home_page`함수에서 처리한다. 

POST를 처리하기 위해 `home_page` 뷰에 대한 새로운 단위테스트를 작성한다.

`lists/tests.py`

```python
def test_uses_home_template(self):
    response = self.client.get('/')
    self.assertTemplateUsed(response, 'home.html')


def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})
    self.assertIn('A new list item', response.content.decode())
```

POST를 수행하기 위해 `self.client.post`를 호출하고 `data`는 보내려는 양식 데이터가 들어있는 인수를 취한다. 다음 POST 요청의 텍스트가 렌더링 된 HTML이 있는지 확인한다.

```bash
$ python manage.py test
[...]
AssertionError: 'A new list item' not found in '<html>\n    <head>\n
<title>To-Do lists</title>\n    </head>\n    <body>\n        <h1>Your To-Do
list</h1>\n        <form method="POST">\n            <input name="item_text"
[...]
</body>\n</html>\n'
```

이제 이 오류를 해결하기 위해 코드를 작성한다.

`lists/views.py`

```python
from django.http import HttpResponse
from django.shortcuts import render

def home_page(request):
    if request.method == 'POST':
        return HttpResponse(request.POST['item_text'])
    return render(request, 'home.html')
```

이는 그저 단위 테스트를 통과하게 만든다. 우리가 원하는 동작은 홈페이지 템플릿 테이블에 추가되는 것이다.



### 템플릿에서 렌더링 할 파이썬 변수 전달하기

파이썬 객체를 템플릿에 포함시켜보자.

`lists/templates/home.html`

```html
<body>
    <h1>Your To-Do list</h1>
    <form method="POST">
        <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
        {% csrf_token %}
    </form>

    <table id="id_list_table">
        <tr><td>{{ new_item_text }}</td></tr>  
    </table>
</body>
```

이후 단위 테스트를 조정하여 템플릿을 아직 사용하고 있는지 확인한다.

`lists/tests.py`

```python
    def test_can_save_a_POST_request(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})
        self.assertIn('A new list item', response.content.decode())
        self.assertTemplateUsed(response, 'home.html')
```

예상대로 실패한다.

```bash
AssertionError : 응답을 렌더링하는 데 템플릿이 사용되지 않습니다.
```

이제 이를 해결하기위해 `render`함수를 사용하여 템플릿, 데이터를 내려준다.

`lists/views.py`

```python
def home_page(request):
    return render(request, 'home.html', {
        'new_item_text': request.POST['item_text'],
    })
```



### 예기치 않은 오류

위처럼 작성하면 잘되야 정상이지만, 빈 값을 전달했을 때 dict는 오류를 발생한다.

`lists/views.py`

```python
def home_page(request):
    return render(request, 'home.html', {
        'new_item_text': request.POST.get('item_text', ''),
    })
```

이제 단위 테스트는 통과한다. 하지만 기능 테스트에서 실패한다.

```bash
AssertionError: False is not true : New to-do item did not appear in table
```

도움이 되는 에러는 아니다. 더 도움되는 디버깅 에러를 만들기위해 다음과 같이 작성한다.

`functional_tests.py`

```python
self.assertTrue(
    any(row.text == '1: Buy peacock feathers' for row in rows),
    f"New to-do item did not appear in table. Contents were:\n{table.text}"  
)
```

"f-string"을 사용하여 해당 줄의 실제 내용을 가져온다.

```bash
AssertionError: False is not true : New to-do item did not appear in table.
Contents were:
Buy peacock feathers
```

이보다 더 좋은 방법은 4줄을 한줄로 간단히 구현 가능하다.

```python
    self.assertIn('1: Buy peacock feathers', [row.text for row in rows])
```

이를 이용하면 다음과 같은 오류가 발생한다.

```bash
    self.assertIn('1: Buy peacock feathers', [row.text for row in rows])
AssertionError: '1: Buy peacock feathers' not found in ['Buy peacock feathers']
```

해당 에러를 통과시키는 가장 간단한 코드를 짜보자.

`lists/templates/home.html`

```html
    <tr><td>1: {{ new_item_text }}</td></tr>
```

이제 test를 하면 `self.fail('Finish the test!')`에 도착하게된다. 하지만 FT를 확장하면 이 해결책이 도움이 되지 않은 것을 알 수 있다.

`functional_tests.py`

```python
    # There is still a text box inviting her to add another item. She
    # enters "Use peacock feathers to make a fly" (Edith is very
    # methodical)
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Use peacock feathers to make a fly')
    inputbox.send_keys(Keys.ENTER)
    time.sleep(1)

    # The page updates again, and now shows both items on her list
    table = self.browser.find_element_by_id('id_list_table')
    rows = table.find_elements_by_tag_name('tr')
    self.assertIn('1: Buy peacock feathers', [row.text for row in rows])
    self.assertIn(
        '2: Use peacock feathers to make a fly',
         [row.text for row in rows]
    )

    # Edith wonders whether the site will remember her list. Then she sees
    # that the site has generated a unique URL for her -- there is some
    # explanatory text to that effect.
    self.fail('Finish the test!')

    # She visits that URL - her to-do list is still there.
```

물론 기능 테스트는 오류를 반환한다.

```bash
AssertionError: '1: Buy peacock feathers' not found in ['1: Use peacock
feathers to make a fly']
```



### 삼진과 리팩터

코드를 반복하지 마라 (DRY)원칙이 존재한다. 코드를 복사하여 붙여 넣을 수 있지만, 3번 발생하면 복제본을 제거해야한다.

리팩터링 전 항상 커밋을 수행하도록하자.

기능 테스트로 돌아가서, 헬퍼 함수를 작성하여 테스트 코드에 포함되지 않고 중복을 처리한다.

`functional_tests.py`

```python
    def tearDown(self):
        self.browser.quit()


    def check_for_row_in_list_table(self, row_text):
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertIn(row_text, [row.text for row in rows])


    def test_can_start_a_list_and_retrieve_it_later(self):
        [...]
```

헬퍼함수는 `tearDown`과 첫 번째 테스트 함수 사이에 두는 것을 권장한다.

`functional_tests.py`

```python
    # When she hits enter, the page updates, and now the page lists
    # "1: Buy peacock feathers" as an item in a to-do list table
    inputbox.send_keys(Keys.ENTER)
    time.sleep(1)
    self.check_for_row_in_list_table('1: Buy peacock feathers')

    # There is still a text box inviting her to add another item. She
    # enters "Use peacock feathers to make a fly" (Edith is very
    # methodical)
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Use peacock feathers to make a fly')
    inputbox.send_keys(Keys.ENTER)
    time.sleep(1)

    # The page updates again, and now shows both items on her list
    self.check_for_row_in_list_table('1: Buy peacock feathers')
    self.check_for_row_in_list_table('2: Use peacock feathers to make a fly')

    # Edith wonders whether the site will remember her list. Then she sees
    [...]
```

다시 테스트를 실행한다.

```bash
AssertionError: '1: Buy peacock feathers' not found in ['1: Use peacock
feathers to make a fly']
```

리팩터링 후 커밋한다.



### 장고 ORM과 모델

Django는 우수한 ORM을 제공하며, 이를 사용하는 단위 테스트를 작성하는 것은 실제로 원하는 방식으로 코드를 실행되게 하는것을 배우기 위한 훌륭한 방법이다.

`lists/tests.py`

```python
from lists.models import Item
[...]

class ItemModelTest(TestCase):

    def test_saving_and_retrieving_items(self):
        first_item = Item()
        first_item.text = 'The first (ever) list item'
        first_item.save()

        second_item = Item()
        second_item.text = 'Item the second'
        second_item.save()

        saved_items = Item.objects.all()
        self.assertEqual(saved_items.count(), 2)

        first_saved_item = saved_items[0]
        second_saved_item = saved_items[1]
        self.assertEqual(first_saved_item.text, 'The first (ever) list item')
        self.assertEqual(second_saved_item.text, 'Item the second')
```

단위 테스트를 실행해보자.

```bash
ImportError: cannot import name 'Item'
```

이제 models.py에서 가져올 Item 모델을 생성한다.

`lists/models.py`

```python
from django.db import models

class Item(object):
    pass
```

이는 다음과 같은 에러를 발생한다.

```bash
    first_item.save () 
AttributeError : 'Item'객체에 'save'속성이 없습니다.
```

Item 클래스에 `save`메서드를 제공하고 실제 장고 모델로 만들려면 `Model`클래스를 상속받아야한다.

`lists/models.py`

```python
from django.db import models

class Item(models.Model):
    pass
```



### 데이터베이스 마이그레이션

다음으로 일어나는 일은 데이터베이스 오류이다.

```bash
django.db.utils.OperationalError: no such table: lists_item
```

테이블 생성을 위해 마이그레이션을 생성한다.

```bash
$ python manage.py makemigrations
Migrations for 'lists':
  lists/migrations/0001_initial.py
    - Create model Item
$ ls lists/migrations
0001_initial.py  __init__.py  __pycache__
```



### The Test Gets Surprisingly Far

```bash
$ python manage.py test lists
[...]
    self.assertEqual(first_saved_item.text, 'The first (ever) list item')
AttributeError: 'Item' object has no attribute 'text'
```

마지막으로 실패한 곳보다 많이 멀리 왔다. 우리는 두개의 객체를 저장하는 과정을 거쳤고, 데이터베이스에 저장되어 있는지 확인했다. 그러나 장고는 `.text`속성을 가져오지 못했다.

컬럼을 추가하기위해 models.py를 다시 수정한다.

`lists/models.py`

```python
class Item(models.Model):
    text = models.TextField()
```



### 새 필드는 새 마이그레이션을 의미한다.

다시 테스트하면 오류가 발생한다.

```bash
django.db.utils.OperationalError: no such column: lists_item.text
```

데이터베이스에 새 필드를 추가했기 때문에 마이그레이션해야한다.

```bash
$ python manage.py makemigrations
You are trying to add a non-nullable field 'text' to item without a default; we
can't do that (the database needs something to populate existing rows).
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null
value for this column)
 2) Quit, and let me add a default in models.py
Select an option:2
```

기본 값없이 열을 추가할 수 없기 때문에 기본값을 지정해준다.

`lists/models.py`

```python
class Item(models.Model):
    text = models.TextField(default='')
```

이제 마이그레이션이 완료된다.



### 데이터베이스에 POST 저장

홈페이지 POST 요청에 대한 테스트를 조정하고 View가 응답하는 전달 대신 새 항목을 데이터베이스에 저장한다 가정한다. 기존 테스트의 3 라인을 추가하여 이를 수행할 수 있다.

`lists/tests.py`

```python
def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})

    self.assertEqual(Item.objects.count(), 1)  
    new_item = Item.objects.first()  
    self.assertEqual(new_item.text, 'A new list item')  

    self.assertIn('A new list item', response.content.decode())
    self.assertTemplateUsed(response, 'home.html')
```

1. 새 `Item` 데이터베이스에 저장되었는지 확인한다.
2. 저장된 Item object를 꺼내온다.
3. 항목의 텍스트가 일치하는지 확인한다.

테스트가 길다면 적어놓자. 긴 단위 테스트는 복잡하다는 표시일 수 있다.

```bash
    self.assertEqual (Item.objects.count (), 1) 
AssertionError : 0! = 1
```

이를 해결하기 위해 View를 수정하자.

`lists/views.py`

```python
from django.shortcuts import render
from lists.models import Item

def home_page(request):
    item = Item()
    item.text = request.POST.get('item_text', '')
    item.save()

    return render(request, 'home.html', {
        'new_item_text': request.POST.get('item_text', ''),
    })
```

여기서 문제는 POST로 들어온 item_text만 new_item_text로 전달된다는 점이다. 테스트 중에 다른 문제를 해결하는것보다 현재 문제를 해결하고 다른 문제를 해결하는 것이 좋다. 혹은 현재 변경내역을 담아놓고 다른 문제를 해결하여도 좋다.

이제 테스트는 잘 통과한다.

이제 다른 문제들을 해결해보자.

- 빈 항목의 요청은 저장하지 않는다.
- Code Smell: POST 테스트가 김
- 표에 여러 항목 표시
- 하나 이상의 목록을 지원하자!

첫번째 작업부터 순차적으로 진행한다.

`lists/tests.py`

```python
class HomePageTest(TestCase):
    [...]

    def test_only_saves_items_when_necessary(self):
        self.client.get('/')
        self.assertEqual(Item.objects.count(), 0)
```

이는 우리에게 `1 != 0` 실패를 준다. 이를 해결하자.

`lists/views.py`

```python
def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']  
        Item.objects.create(text=new_item_text)  
    else:
        new_item_text = ''  

    return render(request, 'home.html', {
        'new_item_text': new_item_text,  
    })
```

1. `new_item_text`는 POST의 내용이나 빈 문자열을 포함하는 변수를 사용한다.
2. `Item.objects.create`를 사용하여 빠르게 생성할 수 있다.

이제 모두 작동한다.



### POST 후 리다이렉션

하지만 `new_item_text=''`는 고쳐야할 부분이다. View 기능에는 사용자 입력 처리와 적절한 응답 반환이라는 두 가지 작업이 있다. 첫 번째 사용자 입력을 데이터베이스에 저장하는 첫 번째 부분을 처리 했으므로 두 번째 부분을 살펴보자.

POST 후 항상 리다이렉션되므로 그렇게한다. 다시 한번 POST 요청을 저장하기 위한 단위 테스트를 변경하여 항목에 응답을 렌더링 하는 대신 홈페이지로 리다이렉션해야한다고 말한다.

`lists/tests.py`

```python
    def test_can_save_a_POST_request(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```

더이상 `.content` 템플릿에 의한 렌더링을 기대하지 않으므로 이를 삭제하고, 리다이렉션 코드인 302, 도착지점인 '/'를 비교한다.

테스트하면 `200 != 302`를 반환한다. 이제 해결해보자.

`lists/views.py`

```python
from django.shortcuts import redirect, render
from lists.models import Item

def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/')

    return render(request, 'home.html')
```

테스트가 통과하여야한다.



### 더 나은 단위 테스트 실습: 각 테스트는 한가지 테스트를 해야함

기존 `test_can_save_a_POST_request`는 POST 후 리다이렉션을 실행한다. 

이보다 더 나은 테스트코드를 작성하기위해 각 테스트가 한가지만 테스트하도록 작성한다. 그 이유는 버그 트래킹이 쉽기 때문이다. 테스트에서 여러개의 어설션을 사용하면 초기 어설션에서 테스트가 실패하면 이후 어설션의 상태를 모르게된다.

첫 시도에서 단 하나의 어설션으로 완벽한 단위 테스트를 작성하는 것은 아니지만 지금은 문제를 해결하지 좋을 때라고 생각한다.

`lists/tests.py`

```python
    def test_can_save_a_POST_request(self):
        self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')


    def test_redirects_after_POST(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})
        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```



### 템플릿 항목 렌더링

표에 여러 항목 표시할 수 있는 새로운 단위 테스트를 작성한다.

`lists/tests.py`

```python
class HomePageTest(TestCase):
    [...]

    def test_displays_all_list_items(self):
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')

        response = self.client.get('/')

        self.assertIn('itemey 1', response.content.decode())
        self.assertIn('itemey 2', response.content.decode())
```

예상대로 실패한다.

```bash
AssertionError: 'itemey 1' not found in '<html>\n    <head>\n [...]
```

장고 템플릿 구문을 사용하여 list를 반복하는 태그를 작성한다.

`lists/templates/home.html`

```html
<table id="id_list_table">
    {% for item in items %}
        <tr><td>1: {{ item.text }}</td></tr>
    {% endfor %}
</table>
```

바꿔도 테스트는 통과하지 않는다.

`lists/views.py`

```python
def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/')

    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

이렇게 작성하면 단위 테스트를 통과하게 된다.

그 후 기능 테스트를 실행하면 또다른 예외를 발생한다.

```bash
$ python functional_tests.py
[...]
AssertionError: 'To-Do' not found in 'OperationalError at /'
```

단위 테스트 데이터베이스는 테스트시에 자동으로 생성되지만 기능 테스트는 실제 동작하고있는 데이터베이스를 사용하여 테스트한다.



### 마이그레이션으로 데이터베이스 생성

실제 데이터베이스를 만들어주려면 `settings.py`에서 데이터베이스를 세팅한다.

`superlists/settings.py`

```python
[...]
# Database
# https://docs.djangoproject.com/en/1.11/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

그 후 `python manage.py migrate`를 실행하여 데이터베이스를 생성한다.

다시 테스트를 돌려보면:

```bash
AssertionError: '2: Use peacock feathers to make a fly' not found in ['1: Buy
peacock feathers', '1: Use peacock feathers to make a fly']
```

거의 다 왔다! 번호 매김만 올바르게 작성하면 된다.

`lists/templates/home.html`

```django
    {% for item in items %}
        <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
```

Django Template 문법을 사용하여 작성해서 테스트를 무사히 넘겼다.

하지만 FT가 실행될 때마다 데이터가 쌓여가는 것을 볼 수 있다. 이는 나중에 해결한다.



### 해당 챕터에서 한 것

- POST를 사용하여 새 항목을 추가
- 목록 항목을 저장하기위해 데이터베이스 모델을 설정
- 테스트 데이터베이스와 실제 데이터베이스에 대해 마이그레이션 만드는 방법을 배움
- Django 템플릿 `{% csrf_token %}`과 `{% for .. endfor %}`루프를 사용함
- 적어도 세 가지 다른 FT 디버깅 기술을 사용했다. `inline print`, `time sleep`, `오류 메세지 개선`

하지만 여전히 할 일은 남아있다.

- ~~빈 항목의 요청은 저장하지 않는다.~~
- ~~Code Smell: POST 테스트가 김~~
- ~~표에 여러 항목 표시~~
- FT가 실행된 후 정리
- 하나 이상의 목록을 지원하자!