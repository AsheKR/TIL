# 로깅 HOW TO



### 로깅을 사용할 때

로깅을 사용할 때를 결정하려면 아래의 표를 참고하자.



| 수행하려는 작업                                              | 작업을 위한 최상의 도구                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 일반적인콘솔 출력                                            | **print()**                                                  |
| 프로그램의 정상 작동 중 발생하는 이벤트 보고 (상태 모니터링이나 결함 조사) | **logging.info()** 또는 진단의 목적이 자주 자세한 출력의 경우 **logging.debug()** |
| 특정 실행시간 이벤트와 관련하여 경고를 발행                  | 라이브러리 코드에서 **warnings.warn()**: 문제를 피할 수 있고 경고를 제거하기위해 클라이언트 응용 프로그램이 수정되어야 하는 경우<br /> **logging.warning()**: 클라이언트 응용 프로그램이 할 수 있는 일이 없는 상황이지만 이벤트를 계속 주목해야하는 경우 |
| 특정 실행시간 이벤트와 관련한 에러를 보고                    | 예외를 일으킨다.                                             |
| 예외를 발생시키지 않고 에러의 억제를 보고 (장기 실행 서버 프로세스의 에러 처리기) | 구체적인 에러와 응용 프로그램 영역에 적절한 **logging.error()**, **logging.exception()**, **logging.critical()** |

로깅 함수는 추적되어 이벤트의 수준 또는 심각도를 따라 명명된다. 표준 수준과 그 용도는 아래 표와같다.

| 수준       | 사용할 때                                                    |
| ---------- | ------------------------------------------------------------ |
| `DEBUG`    | 상세한 정보. 보통 문제를 진단할 때 필요하다.                 |
| `INFO`     | 예상대로 작동하는지에 대한 확인                              |
| `WARNING`  | 예상치 못한 일이 발생했거나 가까운 미래에 발생할 문제에 대한 표시. 소프트웨어는 여전히 예상대로 작동한다. |
| `ERROR`    | 더욱 심각한 문제로 인해, 소프트웨어가 일부 기능을 수행하지 못했다. |
| `CRITICAL` | 심각한 에러. 프로그램 자체가 계속 실행되지 않을 수 있음을 나타낸다. |

기본 수준은 `WARNING`이다. logging패키지가 달리 구성되지 않는한, 이 수준 이상의 이벤트만 추적된다는 것을 의미한다.

추적되는 이벤트는 여러 방식으로 처리될 수 있다. 가장 간단한 방법은 콘솔에 인쇄한다. 또다른 일반적 방법은 파일에 기록하는 것이다.

#### 간단한 예

```python
import logging
logging.warning('Watch out!')  # console에 메세지가 출력된다.
logging.info('I told you so')  # 아무것도 출력되지 않는다.
```

```
WARNING: root: Watch out!
```

기본 수준이 `WARNING`이므로 `INFO` 메세지는 나타내지 않는다. `root`나 포매팅에 대해서는 추후 설명한다.

#### 파일에 로깅하기

매우 일반적 상황은 로깅 이벤트를 파일에 기록하는 것이다.

```python
import logging
logging.basicConfig(filename='example.log', level=logging.DEBUG)
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
```

이후 파일을 열면 다음의 메세지를 찾을 수 있다.

```text
DEBUG:root:This message should go to the log file
INFO:root:So should this
WARNING:root:And this, too
```

이 예제는 기본 레벨은 `DEBUG`로 설정했기 때문에 모든 메세지가 출력되었다.

#### 변수 데이터 로깅

```python
import logging
logging.warning('%s before you %s', 'Look', 'leap!')
```

```text
WARNING:root:Look before you leap!
```

이전 버전과의 호환성을 위해 %-Style의 문자열 포매팅을 사용한다.

#### 표시된 메세지의 포맷 변경

```python
import logging
logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)
logging.debug('This message should appear on the console')
logging.info('So should this')
logging.warning('And this, too')
```

```text
DEBUG:This message should appear on the console
INFO:So should this
WARNING:And this, too
```

`root`가 사라졌음을 주목한다. 포맷 문자열에 나타낼 수 있는 모든 것은 **LogRecord Attribute**문서를 참조한다.

#### 메세지에 날짜/시간 표시

`%(asctime)s`를 사용하여 시간을 표시할 수 있다.

```python
import logging
logging.basicConfig(format='%(asctime)s %(message)s')
logging.warning('is when this event was logged.')
```

```text
2010-12-12 11:41:42,612 is when this event was logged.
```

기본 포맷은 `ISO8601` 또는 `RFC 3399`와 같다. 날짜/시간의 포맷을 좀 더 제어해야하는 경우, `basicConfig`에 _datefmt_ 인자를 제공한다.

```python
import logging
logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%m/%d/%Y %I:%M:%S %p')
logging.warning('is when this event was logged.')
```

```text
12/12/2010 11:46:36 AM is when this event was logged.
```

_datefmt_ 인자의 형식은 **time.strftime()**에 의해 지원되는 것과 같다.



## 로거의 이름

로거의 범위는 패키지/모듈 계층을 추적할 수 있게 해주기때문에 보통 모듈 수준의 로거를 사용한다. 

```python
logger = logging.getLogger('__name__')
```

로깅 전략에 따라 객체, 클래스, 함수별 로거 이름을 생성할 수 있다.

```python
# 객체 인스턴스
def __init__(self, name)
	self.logger = logging.getLogger("{0}.{1}".format(self.__class__.qualname__, name))

# 클래스 명(__class__.__qualname__)만으로 생성한다.

# 큰 함수 내 로그를 생성할수도있다.

def main():
    logger = logging.getLogger("main") 
```



## 로거

**Logger**객체는 세 가지 작업을 한다.

- 응용 프로그램이 실행시간에 메세지를 기록할 수 있도록 여러 메서드를 응용 프로그램 코드에 노출한다.
- 로거 객체는 삼각도 또는 필터 객체에 따라 어떤 로그 메세지를 처리할지 결정한다.
- 로거 객체는 관련 로그 메세지를 관심 있는 모든 로그 처리기로 전달한다.

대표적인 메서드는 두가지 범주로 나뉜다.

1. 일반적인 구성 메서드

- Logger.setLevel()
  - 수준을 설정한다.
- Logger.addHandler(), Logger.removeHandler()
  - 로거 객체에서 처리기 객체를 추가하고 제거한다.
- Logger.addFilter(), Logger.removeFilter()
  - 로거 필터객체를 추가하고 제거한다.

2. 로거 객체가 생성된 후 로그 메세지를 생성하는 메서드

- Logger.debug(), Logger.info(), Logger.warning(), Logger.error(), Logger.critical()
- Logger.exception()
  - Logger.error()와 비슷한 로그 메세지를 생성한다. 차이점은 스택 트레이스를 덤프한다.
- Logger.log()
  - 명시적으로 로그 수준을 받아들인다.



로거에는 **실효 수준**이라는 개념이 있다. 수준이 명시적으로 설정되지 않은 경우, 부모 수준을 대신 사용한다. 부모가 명시적 수준 집합을 가지지 않으면 재귀적으로 부모의 수준을 사용한다. 루트 로거는 기본적으로 `WARNING` 을 사용하기때문에 아무것도 설정하지 않으면 루트 로거의 수준을 사용한다.



## 처리기

Logger 객체는 addHandler() 메서드를 사용하여 0개 이상의 처리기 객체를 자신에게 추가 할 수 있다.

표준 라이브러리에는 많은 처리기가 포함되어있다.

| Handler                  | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| StreamHandler            | 스트림으로 메세지를 보낸다.                                  |
| FileHandler              | 디스크 파일에 메세지를 보낸다.                               |
| BaseRotatingHandler      | 로그 파일을 회전시킨다. (새 파일을 생성함)                   |
| RotatingFileHandler      | 최대 로그 파일 크기와 로그 파일 회전을 지원한다.             |
| TimedRotatingFileHandler | 인스턴스는 디스크 파일에 메세지를 보내는데, 일정한 시간 간격으로 로그 파일을 회전시킨다 |
| SocketHandler            | TCP/IP 소켓에 메세지를 보낸다.                               |
| DatagramHandler          | UDP 소켓에 메세지를 보낸다.                                  |
| SMTPHandler              | 지정된 전자 우편 주소로 메세지를 보낸다.                     |
| SysLogHandler            | syslog 데몬에 메세지를 보낸다.                               |
| NTEventLogHandler        | NT/2000/XP 이벤트 로그에 기록을 남긴다.                      |
| MemoryHandler            | 버퍼에 메세지를 보내는데, 특정 기준이 만족 될 때마다 플러시된다. |
| HTTPHandler              | `GET`, `POST`를 사용해서 HTTP 서버에 메세지를 보낸다.        |
| WatchedFileHandler       | 파일이 변경되면 닫히고 파일이름을 사용하여 다시 연다. 유닉스 계열 시스템에서만 유용하다. |
| QueueHandler             | **queue**, **multiprocessing** 모듈에 구현된 것과 같은 큐로 메세지를 보낸다. |
| NullHandler              | 에러메세지로 아무것도 하지 않는다.                           |

처리기에는 응용 프로그램 개발자가 직접 신경써야할 메서드가 거의 없다.

- setLevel()
  - 로거에 설정된 수준은 처리기에 전달할 메세지의 수준을 판별한다. 각 처리기에 설정된 수준은 처리기가 전송할 메세지를 결정한다.
- setFormatter()
  - 처리기가 사용할 포매터를 결정한다.
- addFilter(), removeFilter()
  - 필터 객체를 구성하고 해체한다.



## 포매터

로그 메세지의 구성 및 내용을 설정한다.

```
logging.Formatter.__init__(fmt=None, datefmt=None, style='%)
```

날짜 포맷 문자열이 없다면 기본 형식은 다음과 같다.

```
%Y-%m-%d %H:%M:%S
```



`style`은 `%` 또는 `$`이다.

`%`는 `%(<dictionary key>)s` 스타일의 문자열 치환을 사용한다. `$`이면 메세지 포맷 문자열은 `string.Template.substitue()`가 기대하는 것과 일치해야한다.



#### dictionary key





| 어트리뷰트 이름 | 포맷                         | 설명                                                         |
| :-------------- | :--------------------------- | :----------------------------------------------------------- |
| args            | 직접 포맷할 필요는 없습니다. | `message` 를 생성하기 위해 `msg` 에 병합되는 인자의 튜플. 또는 (인자가 하나뿐이고 딕셔너리일 때) 병합을 위해 값이 사용되는 딕셔너리. |
| asctime         | `%(asctime)s`                | 사람이 읽을 수 있는, LogRecord 가 생성된 시간. 기본적으로 '2003-07-08 16:49:45,896' 형식입니다 (쉼표 뒤의 숫자는 밀리 초 부분입니다). |
| created         | `%(created)f`                | LogRecord 가 생성된 시간 (time.time() 이 반환하는 시간.)     |
| exc_info        | 직접 포맷할 필요는 없습니다. | 예외 튜플 (`sys.exc_info` 에서 제공) 또는, 예외가 발생하지 않았다면, `None`. |
| filename        | `%(filename)s`               | `pathname` 의 파일명 부분.                                   |
| funcName        | `%(funcName)s`               | 로깅 호출을 포함하는 함수의 이름.                            |
| levelname       | `%(levelname)s`              | 메시지의 텍스트 로깅 수준 (`'DEBUG'`, `'INFO'`, `'WARNING'`, `'ERROR'`, `'CRITICAL'`). |
| levelno         | `%(levelno)s`                | 메시지의 숫자 로깅 수준 (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`). |
| lineno          | `%(lineno)d`                 | 로깅 호출이 일어난 소스 행 번호 (사용 가능한 경우).          |
| message         | `%(message)s`                | 로그 된 메시지. `msg % args` 로 계산됩니다. Formatter.format() 이 호출 될 때 설정됩니다. |
| module          | `%(module)s`                 | 모듈 (`filename` 의 이름 부분).                              |
| msecs           | `%(msecs)d`                  | LogRecord 가 생성된 시간의 밀리 초 부분.                     |
| msg             | 직접 포맷할 필요는 없습니다. | 원래 로깅 호출에서 전달된 포맷 문자열. `args`와 병합하여 `message` 를 만듭니다. 또는 임의의 객체 |
| name            | `%(name)s`                   | 로깅 호출에 사용된 로거의 이름.                              |
| pathname        | `%(pathname)s`               | 로깅 호출이 일어난 소스 파일의 전체 경로명 (사용 가능한 경우). |
| process         | `%(process)d`                | 프로세스 ID (사용 가능한 경우).                              |
| processName     | `%(processName)s`            | 프로세스 이름 (사용 가능한 경우).                            |
| relativeCreated | `%(relativeCreated)d`        | logging 모듈이 로드된 시간을 기준으로 LogRecord가 생성된 시간 (밀리 초). |
| stack_info      | 직접 포맷할 필요는 없습니다. | 현재 스레드의 스택 바닥에서 이 레코드를 생성한 로깅 호출의 스택 프레임까지의 스택 프레임 정보 (사용 가능한 경우). |
| thread          | `%(thread)d`                 | 스레드 ID (사용 가능한 경우).                                |
| threadName      | `%(threadName)s`             | 스레드 이름 (사용 가능한 경우).                              |



## 로깅 구성 방법

1. 위에 나열된 구성 메서드를 호출하는 파이썬 코드를 사용하여 로거, 처리기 및 포매터를 명시적으로 만든다.
2. 로깅 구성 파일을 만들고, **fileConfig()** 함수를 사용하여 그것을 읽는다.
3. 구성 정보의 딕셔너리를 만들고, **dictConfig()** 함수에 전달한다.



로깅은 어떤 소프트웨어가 실행할 때 발생하는 이벤트를 추적하는 수단이다. 