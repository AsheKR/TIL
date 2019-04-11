# Python's Innards: Introduction

>  [Python's Innards: Introduction](https://tech.blog.aknin.name/2010/04/02/pythons-innards-introduction/)을 정리한 글, 원글에 따라 이 글도 [CC BY-NC-SA 3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/) 라이센스를 따른다.



### Curriculum

- CPython
- py3k
- bytecode evaluation

따로 언급하지 않는 한 **CPython**이며, **POSIX** 또는 **LINUX**를 가정한다.



### 기반 상식

- C 지식
- OS 이론
- assembly
- Python

- UNIX ( 설치할 수 있는 정도? )



## Introduction

Python의 경우 (다른 인터프리터 언어와 마찬가지로) **가상머신** 위에서 동작한다.



------

#### 프로세스 가상 머신이란?

운영체제 안에서 일반 응용 프로그램을 돌리고 단일 프로세스를 지원한다.

어떤 플랫폼에서나 같은 방식으로 실행하는 프로그램(플랫폼 독립성)을 허용하고 기초가되는 하드웨어나 운영 체제의 상세한 부분을 가져오는 독립 프로그래밍 환경을 제공하기 위함이다.

------



CPU는 입력값(기계어, 데이터)을 받고 상태값(레지스터)을 가지며 입력 값과 상태 값에 따라 출력값을 보내는 복잡한 전자기계이다.

CPython은 상태값과 명령 처리방법을 가지고 있는 소프트웨어로 만들어진 기계이다. 이 소프트웨어 기계는 Python 인터프리터를 호스팅하는 프로세서에서 작동한다.



즉, `$ python -c 'print("Hello, world!")'` 같은 일을 할 때 일어나는 개요를 보자.

Python 바이너리가 실행되고 표준 C 라이브러리 초기화가 거의 모든 프로세스가 실행되며, main 함수가 실행된다. (`./Modules/python.c: main`이 곧 `./Modules/main.c: Py_Main`을 호출한다.) 평범한 초기화 작업 (argument를 파싱하고, 환경변수가 동작에 영향을 주는지 확인하고, 표준 스트림에 접근하고 그에 따라 행동하는 등)을 마치고나면 `./Python/pythonrun.c: Py_Initialize`가 호출된다. 여러가지 방법으로 이 함수는 CPython을 실행하는데 필요한 부품을 **빌드**하고 모아서 **프로세스**를 **Python 인터프리터가 있는 프로세스**로 만든다. 무엇보다도 두가지 중요한 **인터프리터 상태**와 **스레드 상태**에 관한 Python 데이터 구조를 생성한다. 또한 내장 모듈 _sys_와 모든 내장 함수모듈을 호스팅한다.

이 다음 Python은 쉘에 입력한 실행 옵션에 따라 실행한다. 현재는 **-c**를 사용해 하나의 문자열을 실행하는 경우를 따르고 있다. 이 문자열을 실행하기 위해 `./Python/pythonrun.c: PyRun_SimpleStringFlags`가 호출된다. 이 함수는 `__main__` 네임스페이스를 생성한다. 이 네임스페이스에서 데이터가 저장되고, 실행될것이다. 이전에 문자열을 기계가 작업가능한 형태로 변환해야한다.

그래서 `PyRun_SimpleStringFlags`의 파서/컴파일러 단계는 크게 다음과 같다. 소스 코드를 토큰화하고 구체 구문 트리(Concrete Syntax Tree, CST)를 만들고 CST를 추상 구문 트리(Abstract Syntax Tree, AST)로 변환하고 마지막으로 `./Python/ast.c: PyAST_FromNode.`를 사용하여 AST를 **Code Object**로 컴파일한다. 지금은 **Code Object**를 Python의 VM 기계가 작동할 수 있는 기계어 이진 문자열이라 생각하자. 이제 해석할 준비가 되었다.

현재 `__main__`에는 Code Object를 가지고있고 이제 그것을 **평가**해야한다. 

`Python/pythonrun.c: run_mod, v = PyEval_EvalCode(co, golobals, locals);`

이는 **Code Object**와 **global**, **local** 네임스페이스를 인자로 받는다. (현재의 경우 둘 다 새로 생성된 `__main__` 네임스페이스가 된다) 그리고 이것들로부터 **frame object**를 작성해  실행한다. 위에서 말했듯 `Py_Initialize`가 스레드 상태를 생성한다. 각 Python 스레드는 현재 실행중인 프레임의 스택을 가리키는 자체 스레드 상태로 표현된다. frame object가 생성되고 스레드 상태 스택의 맨 위에 놓인 후, `./Python/ceval.c: PyEval_EvalFrameEx` 코드에 의해 opCode로 평가된다.

`PyEval_EvalFrameEx`는 frame을 가져와 opCode를 생성하고, opCode와 일치하는 짧은 C코드를 실행한다. 컴파일 된 Python 코드를 분해하여 이러한 "opCode"가 어떻게 생겼는지 살펴본다.

```python
>>> from dis import dis # 편리한 역어셈블리 함수!
>>> co = compile("spam = eggs - 1", "<string>", "exec")
>>> dis(co)
  1           0 LOAD_NAME                0 (eggs)
              3 LOAD_CONST               0 (1)
              6 BINARY_SUBTRACT
              7 STORE_NAME               1 (spam)
             10 LOAD_CONST               1 (None)
             13 RETURN_VALUE
>>>
```

`eggs`를 **로드**했고, 상수 값 **1**을 로드한다. 이 후 **Binary Subtract**를 실행한다. 앞에 살펴본 **global**, **local** 네임스페이스에 변수가 **로드**되고 이들은 피연산자 스택(실행중인 프레임 스택과 혼동해서는 안된다.)에 로드 된다. **Binary Subtract**는 그들을 빼내고, 그 중 하나를 뺀다음 그 결과를 다시 피연산자 스택에 넣는다.

`./Python/ceval.c`에서 `PyEval_EvalFrameEx`를 살펴볼 수 있으며, **BINARY_SUBTRACT** opCode가 발견되었을 때 실행되는 코드는 다음과 같다.

```c
TARGET(BINARY_SUBTRACT)
    w = POP();
    v = TOP();
    x = PyNumber_Subtract(v, w);
    Py_DECREF(v);
    Py_DECREF(w);
    SET_TOP(x);
    if (x != NULL) DISPATCH();
    break;
```

어떤 값을 가져오고, 피 연산자 스택의 맨 위를 가져오고, C 함수 `PyNumber_Subtract()`를 호출하고, 우리가 여전히 이해하지 못하는 `Py_DECREF`를 양쪽 모두에게 호출하고, 스택의 상단을 뺄셈의 결과로 설정하고 (이전의 상단을 덮어씀) 마지막으로 결과가 null이 아니면 **DISPATCH**한다.

frame이 실행되고 `pyRun_SimpleStringFlags`가 반환된 후, 메인함수는 청소를 한다. (**Py_Finalize**) 표준 C 라이브러리 초기화 해제(deinitialization)작업이 완료되고, 프로세스가 종료된다.
