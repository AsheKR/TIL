# PEP8

`코딩 컨벤션` 개인 공부
모르겠는 번역 `{*?}`

## 번역 참고

[Luavis' Dev Story](https://b.luavis.kr/python/python-convention)

[kongdols-room](https://kongdols-room.tistory.com/18)

## 코드 레이아웃

### 들여쓰기

하나의 들여쓰기 레벨은 4개의 공백을 사용한다.

여러줄에 걸쳐 코드를 작성할 경우 괄호처럼 암시적으로 여러 줄을 결합하는 기능을 사용하거나, 들여쓰기 (hanging indent)* 를 사용하여 코드의 요소들을 수직으로 정렬해야한다.

* hanging indent: 문장 중간에 들여쓰기를 사용하는 형식

```python
# 구분자를 사용하여 정렬
foo = long_function_name(var_one, var_two,
						var three, var_four)

# 괄호 아래서부터 첫 요소가 있을 경우 한 레벨 이외에 부가적인 indent는 필요하지 않는다.
foo = long_function_name(
	var_one, var_two,
	var_three, var_four)

# 안의 내용과 구별하기위해 들여쓰기
def long_function_name(
		var_one, var_two, var_three,
		var_four):
	print(var_one);
```

여러줄에 걸친 괄호/중괄호/대괄호를 닫을경우 리스트의 마지막 항목의 indent에 맞추어 다음 줄에 닫는 표시를 할 수 있다.
```python
my_list = [
	1, 2, 3,
	4, 5, 6,
	]
result = some_function_that_takes_arguments(
	'a', 'b', 'c',
	'd', 'e', 'f',
	)
```
또는 코드의 첫 문자 위치에 닫는 표시를 넣을 수 있다.
```python
my_list = [
	1, 2, 3,
	4, 5, 6,
]
result = some_function_that_takes_arguments(
	'a', 'b', 'c',
	'd', 'e', 'f',
)
```

### 탭을 쓰냐 공백을 쓰냐!
공백이 더 선호된다.

탭 문자와 공백문자를 섞어쓰지 말자. 위 두개가 섞은 코드는 반드시 공백문자만 사용한 코드로 변환해야한다.
`-t` 옵션을 사용하여 파이썬 인터프리터를 호출하면 탭 문자와 공백 문자가 섞인 코드가 있을 경우 인터프리터가 경고 메세지를 표시한다. `-tt` 옵션을 호출하면 경고 메세지가 에러 메세지로 표시된다. 위 옵션들의 사용을 권장한다.

### 한줄에 들어갈 수 있는 최대 문자 개수
모든 행을 **79**개의 문자로 제한한다.

`주석`이나 `docstring`의 경우 72자로 제한한다.

긴 줄을 여러줄로 나누는 가장 좋은 방법은 괄호, 대괄호 및 중괄호 안에 Python의 암묵적 줄 연속 사용을 사용하는것이다. 긴 줄은 괄호 안에 표현식을 묶어 여러 줄로 나눌 수 있다. 지속적인 행을 위해 백슬래시를 사용하는 것보다 우선 사용되어야한다.

괄호문은 여러개의 `with`, `assert`문이 암묵적 줄 연속을 사용할 수 없으므로 백슬래시가 적절히 사용될 수 있다.
```python
with open('/path/to/some/file/you/want/to/read') as file_1, \
	open('/path/to/some/file/being/written', 'w') as file_2:
	file_2.write(file_1.read())
```

### 이항연산자 이후에 줄바꿈
과거에는 `첫 피연산자 + 연산자` 후 줄바꿈을 추천했다. 이는 두가지 이유로 가독성을 해친다.
- 연산자는 스크린상 한열이 아니라 여러 열로 흩어지는 경향이 있다.
- 각 연산자는 피연산자 이후에 배치된다.
```python
incom = (gross_wages +
		taxable_interest +
		(dividends - qualified_dividends) -
		ira_deduction -
		student_loan_interest)
```


이를 해결하기위해 과거와 반대인 표기법을 사용한다.
첫 단락에 `첫 피연산자` 이후 줄바꿈되고 `연산자 + 피연산자`가 오게된다.
```python
income = (gross_wages
		+ texable_interest
		+ (dividends - qualified_dividends)
		- ira_deduction
		- student_loan_interest)
```

### 빈 줄

`최상위 함수`와 `클래스 정의`는 위에 두 개의 빈줄로 구분한다.
클래스 내 메서드 정의는 단일 공백으로 구분한다.

서로 연관이 있는 함수들의 묶음을 구분하기 위해 빈 줄을 추가로 사용할 수 있다. 반대로 한 줄짜리 구현 코드가 뭉쳐있을 경우 이러한 빈 줄을 생략할 수 있다. (예: 미구현, 임시 코드)

함수 내 논리적 단위를 위해 빈 줄을 사용하되 적당히 사용하자.

파이썬은 `컨트롤-L`(`^L`)을 공백으로 취급한다. 많은 프로그램이 이 문자를 페이지 구분자로 취급하며, 파일 내 서로 연관성 있는 단락을 페이지별로 구분하는데 이 문자를 사용할 수 있다. 몇 에디터와 웹 기반 코드 뷰어는 `컨트롤-L` 문자를 폼피드로 인식하지 못하고 다른 형태의 특수문자로 보여준다.

### 소스파일 인코딩
파이썬 배포본 내 코드는 `UTF-8` 혹은 `Python2의 ASCII`를 사용해야한다. 

`ASCII(Python2)` 또는 `UTF-8(Python3)`를 사용하는 파일에는 인코딩 선언이 없어야한다.

표준 라이브러리에서 기본이 아닌 인코딩은 `테스트 목적`으로 사용하거나 `주석` 또는 `docstring`에 작성자 이름을 언급해야 하는 경우에만 사용해야한다. 그 외에 `비 ASCII` 문자열을 포함하려면 `\x, \u, \U, \N`의 이스케이프 문자를 사용하는게 선호된다.

파이썬 3.0 혹은 그 이후 버전에서는, 표준 라이브러리에 대해 아래 정책이 적용된다. (`PEP3131` 참조)
- 모든 식별자는 반드시 `ASCII` 식별자만 사용하여야하며, 실현가능한 곳에서는 영어만 사용하도록 한다. (많은 경우 영어가 아닌 약어와 기술 용어가 사용된다.) 
- `String` 문자열과 `주석`은 반드시 `ASCII`를 사용하여야만 한다.
- 예외의 경우로 **(a)** `ASCII`특성이 아닌 것을 테스트할 때 **(b)** 작성자의 이름
- 라틴 알파벳 이외의 문자를 사용하는 이름을 가진 작성자는 자신의 이름에 대한 라틴어 음역을 같이 제공해야한다.

### Import

- `Import`는 행으로 구분되어 사용되어진다.

```python
# 좋은 예
import os
import sys

# 나쁜 예
import sys, os

# 이 경우는 괜찮다.
from subprocess import Popen, PIPE
```

- `Import`는 항상 **(a)**파일의 가장 위쪽에 **(b)**모듈의 주석과 닥스트링 뒤에, **(c)**모듈 전역변수나 상수 전에 위치시키도록 한다.
- import는 아래와 같이 그룹화하기도 한다.
```
1. 표준 라이브러리
2. 써드파티 라이브러리
3. 로컬 애플리케이션/자체 라이브러리
```

각 `Import` 사이에는 빈 줄을 삽입해야한다.

- 만약 `Import` 시스템이 부정확히 설정된 경우(예: 패키지 내부 디렉토리가 `sys.path`로 끝나는 경우), 위처럼 그룹화가 된다면 읽기 쉬워지며 수정이 용이해지므로 `Absolute Import`하는 것이 권장된다.
```python
import mypkg.sibling
from mypkg import sibling
from mypkg.sibling import example
```
그러나 복잡한 패키지 레이아웃을 다루고 있을 때 `Explicit Relative import`는 대안으로 사용될 수 있다.
```python
from . import sibling
from .sibling import example
```

표준라이브러리는 항상 복잡한 패키지 레이아웃을 피해야하며 `Absolute Import`를 권장한다.

- 클래스를 포함하고 있는 모듈에서 클래스를 임포트 할 때, 보통 다음과 같이 기술한다.
```python
from myclass import MyClass
from foo.bar.yourclass import YourClass
```

만약 이미 선언된 로컬 이름과 충돌할 경우, 다음과 같이 기술한다.
```python
import myclass
import foo.bar.yourclass
```
그 후 `myclass.MyClass`, `foo.bar.yourclass.YourClass` 같은 식으로 클래스를 사용하면 된다.

#### Wildcard Import
- `from <module> import *`는 네임스페이스에 존재하는 이름(변수, 함수, 클래스 등)이 명확하지 않고 코드를 읽는 사람과 자동화된 툴에게 혼동을 일으키므로 사용하지 않도록 해야한다.
- 이를 사용하여도 되는 예는 `public API`의 일부로 `internal interface`가 다시 게시(republish`{*?}`)된 경우이다. (예로 선택적 (accelerator module)로부터 정의와 인터페이스의 순수한 파이썬 구현을 덮어 쓰고 정확히 어떤 정의가 덮어 쓰여질지 미리 못하는 경우가 있다.)

이 방법으로 이름을 다시 작성할 때 `Public`과 `internal interface`와 관련된 아래의 가이드라인이 적용 가능하다.

#### Module Level Dunder Names

모듈 레벨의 `Dunders` (즉, `__`가 앞뒤로 쓰이는)는 반드시 모듈 닥스트링 뒤에 쓰여져야하고, `from __future__ import`를 제외하고 `import`문 전에 쓰여져야 한다.
파이썬에서는 `future import`는 반드시 닥스트링을 제외하곤 다른 코드 전에 쓰여지는 것을 요구한다.

```python
"""This is the example module.

This module does stuff.
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'Cardinal Biggle'

import os
import sys
```

### 따옴표
파이썬에서 문자열을 표현할 때 큰 따옴표, 작은 따옴표 아무거나 사용 가능하다.
다만 문자열 안에서 따옴표를 사용하여야할 때, 가독성을 위해 백슬래시 사용을 피하도록 적절히 따옴표를 선택해야한다.

세개의 따옴표를 사용할때는 `쌍 따옴표`를 사용한다.

### 표현식에서 공백

아래 상황에서의 공백을 피하자.
- 괄호안의 양쪽 끝
```python
Yes: spam(hame[1], {eggs: 2})
No: spam( hame[ 1 ], { eggs: 2 } )
```
- 뒤에오는 쉼표와 뒤에오는 닫는 괄호 사이
```python
Yes: foo = (0,)
No: bar = (0, )
```
- 콤마, 세미콜론, 콜론의 바로 앞
```python
Yes: if x == 4: print x, y; x, y = y, x
No: if x == 4: printx , y ; x , y = y , x
```
- 슬라이스에서의 콜론은 이진 연산자와 같이 적용되며, 양쪽에서 같은 공백을 가진다. 
- 확장된 슬라이스에서 두개의 콜론은 같은 간격을 가진다. 
- 예외적인 상황은 슬라이스 파라미터가 생략되거나 공간이 생략되는 경우
```python
Yes: 
ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
ham[lower:upper], ham[lower:upper:], ham[lower::step]
ham[lower+offset : upper+offset]
ham[: upper_fn(x) : step_fn(x)], ham[:: step_fn(x)]
ham[lower + offset : upper + offset]

No:
ham[lower + offset:upper + offset]
ham[1: 9], ham[1 :9], ham[1:9: 3]
ham[lower : : upper]
ham[ : upper]
```
- 함수 호출 괄호 앞의 공백은 존재하지 않는다.
```python
Yes: spam(1)
No: spam (1)
```
- 인덱싱, 슬라이싱 괄호 앞의 공백은 존재하지 않음
```python
Yes: dct['key'] = lst[index]
No:  dct ['key'] = lst [index]
```
- 다른 변수와의 줄맞춤을 위한 연산자 주변의 하나 이상의 공백
```python
Yes:
x = 1
y = 2
long_variable = 3

No:
x			  = 1
y			  = 2
long_variable = 3
```

### 다른 권장사항
- 어디서든 후행공백을 피한다. 후행공백은 시작적으로 보이지 않기때문에 혼란을 만들 가능성이 많다. 예로 백슬래쉬 뒤 오는 공백이나 줄바꿈은 문장이 이어지는것을 표현하는 마커 역할을 하지 못한다. 몇 에디터는 이를 허용하지 않는 `pre-commit` 훅을 가진다.
- 이진 연산자는 양쪽에 하나의 공백을 반드시 둔다. 우선순위가 다른 연산자를 사용하는 경우, 우선 순위가 가장 낮은 연산자 주위에 공백을 추가하는 것이 좋다. 그러나 둘 이상의 공백은 사용하지 말고, 항상 이진연산자 양쪽에 같은 양의 공백을 사용하자.
```python
Yes:
i = i + 1
submitted += 1
x = x*2 - 1
hypot2 = x*x + y*y
c = (a+b) * (a-b)

No:
i=i+1
submitted +=1
x = x * 2 - 1
hypot2 = x * x + y * y
c = (a + b) * (a - b)
```

- `Function Annotation`은 콜론에 대한 일반 규칙을 사용해야하며 항상 -> 화살표 주위에 공백이 있어야한다.
```python
Yes:
def munge(input: AnyStr): ...
def munge() -> AnyStr: ...

No:
def munge(input:AnyStr): ...
def munge()->PosInt: ...
```

- 함수의 파라미터로 기본값을 나타내기위해 `=` 사이에 공백을 사용하지 말도록 한다.
```python
Yes:
def complex(real, imag=0.0): ...

No:
def complex(real imag = 0.0): ...
```

그러나 인수 주석을 기본값과 결합할 때 = 기호 주위에 공백을 사용한다.

```python
Yes:
def munge(sep: AnyStr = None): ...
def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...

No:
def munge(input: AnyStr=None): ...
def munge(input: AnyStr, limit = 1000): ...
```

- 합성구문은 일반적으로 좋지 않다.
```python
Yes:
if foo == 'blah':
	do_blah_thing()
do_one()
do_two()
do_three()

Rather not:
if foo == 'blah': do_blah_thing()
do_one(); do_two(); do_three()
```

종종 `if/for/while` 구문의 같은 줄에 짧은 바디가 같이 놓이는 것은 괜찮지만, 여러 줄을 이런식으로 구성하지 않을 것을 당부한다. 또한 긴 줄을 접는것을 피한다.

```python
Rather not:
if foo == 'blah': do_blah_thing()
fox in lst: total += x
whilte t < 10: t = delay()

Definitely not:
if foo == 'blah': do_blah_thing()
else: do_non_blah_thing()

try: something()
finally: cleanup()

do_one(); do_two(); do_three(long, argument,
							list, like, this)

if foo == 'blah': one(); two(); three();
```

## 후행쉼표를 사용할 때

하나의 튜플을 하나의 요소로 만들 때를 제외하고는 후행쉼표는 선택 사항이다. 명확성을 위해 괄호로 덮는것을 추천한다.
```python
# Yes:
FILES = ('setup.cfg', )

OK, but confusing
FILES = 'setup.cfg',
```

후행콤마는 버전 컨트롤 시스템이 사용될 때 혹은 값의 리스트, 독립변수 혹은 import된 item들이 확장될 것으로 예상될 때 상당히 도움된다. 
각각의 값을 각 라인에 놓고 항상 후행콤마를 추가하여야하고 다음줄에 닫는 괄호를 놓아야한다. 
하나의 요소를 갖는 튜플 이외에 뒤에오는 콤마와 닫는 괄호와 같은 라인에 있지 않아야한다.

```python
Yes:
FILES = [
	'setup.cfg',
	'tox.ini',
	]
initialize(FILES,
			error=True,
			)

No:
FILES = ['setup.cfg', 'tox.ini',]
initialize(FILES, error=True,)
```

## 주석

코드와 모순되는 주석은 주석이 없는것보다 나쁘다. 코드 변경시 항상 최신의 주석을 유지하는것이 중요하다.

주석은 완전한 문장이여야한다. 첫 번째 단어는 소문자로 시작하는 식별자가 아닌경우 대문자로 입력해야한다. (식별자의 대/소문자를 변경하지 말자!)

블록 주석은 일반적으로 한 문장으로 끝나는 완전한 문자으로 구성된 하나 이상의 단락으로 구성된다.

여러문장의 주석은 마지막 문장을 제외하고 문장의 끝에 두개의 스페이스를 사용한다.

영어를 쓸 때는 `Strunk and White`를 따르도록한다.

영어권이 아닌 개발자는 자국가 사용자만 읽는다고 확신하지 않는 한 주석을 영어로 작성한다.

### 블록 주석
블록의 주석의 각 행은 `# 및 하나의 공백`으로 시작한다. (주석 안에 들여쓰기 된 텍스트가 아닌경우)

### 인라인 주석
코드와 동일행에서 시작하는 주석문이다.
구문으로부터 최소 2개의 스페이스로 구분되어야한다.
뻔한 내용의 인라인 주석문은 작성하지 않는다.
