# Python Innards: Objects 101

> [Python’s Innards: Objects 101](https://tech.blog.aknin.name/2010/05/12/pythons-innards-objects-101/)을 정리한 글, 원글에 따라 이 글도 [CC BY-NC-SA 3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/) 라이센스를 따른다.

파이선 3.x의 **objects** 구현의 관련한 것을 설명하는 포스트이다.

파이썬에서는 정수에서 dictonary, 사용자 정의 클래스에서 빌트인 클래스, 스택 프레임에서 code object에 이르기까지 대부분의 객체가 Object이다. 주어진 메모리 조각에 대한 포인터가 주어지면 포인터를 Object로 취급하기 위해 **./Include/object.h: PyObject**라는 C 구조체에 정의된 두개의 필드를 가진다.

```c
typedef struct _object {
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
}PyObject;
```

많은 객체가 이 구조를 확장하여 객체의 값을 나타내는데 필요한 다른 변수를 사용한다. 하지만 이 두 필드는 항상 존재해야한다. **reference count**와 **type**

reference count는 객체가 참조된 횟수를 계산하는 정수이다.

```
a = b = c = object()
```

a, b, c 이 이름들은 object가 한번만 할당되었음에도 불구하고 각각 다른 참조를 만든다. object를 다른 이름으로 바인딩하거나 목록에 추가하면 다른 참조가 만들어진다. 하지만 다른 객체를 만들지는 않는다.

reference count는 객체 시스템의 중심은 덜하고 가비지 컬렉션과 더 관련이 있다. 이 주제는 나중에 따로 글로 설명한다. 이 주제를 끝내기전에, 이전에 사용된 `./Include/object.h: Py_DECREF` 매크로를 더 잘 이해할 수 있게 되었다. 단순히 **ob_refcnt**를 감소시키고 0이되면 할당을 해제한다.

**ob_type**은 객체 타입에 대한 포인터로 파이썬의 객체 모델의 핵심이다. (Python3에서는 타입과 클래스는 효과적으로 동일한 것을 의미하고, 역사적인 historic reasons는 문맥에 따라 하나의 이름을 사용하게 한다.) 모든 객체는 객체의 수명동안 절대로 변하지 않는 하나의 타입을 가진다. 아마 가장 중요한 것은 객체의 타입에 따라 객체로 수행할 수 있는 작업이 결정되는것이다.

이전 글에서 인터프리터가 Subtraction opCode를 평가할 때, 그 피연산자가 정수인지, 실수인지 또는 무의미한지에 관계없이 단일 C 함수 **PyNumber_Subtract**를 호출한다.

```python
class Foo(object):
    "I don't have __call__, so I can't be called"
    
class Bar(object):
    __call__ = lambda *a, **kw: 42
    
foo = Foo()
bar = Bar()

foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'Foo' object is not callable

bar()
42

foo.__call__ = lambda *a, **kw: 42
foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'Foo' object is not callable
    
Foo.__call__ = lambda *a, **kw: 42
foo()
42
```

어떻게 C 함수에서 넘어온 모든 종류의 객체를 처리할 수 있는가? 우선 **void \*** 포인터를 사용하여 받을 수 있다. (실제로는 **PyObject \*** 포인터를 받는다.) 그러나 주어진 객체를 조작하는 방법을 어떻게 알 수 있는가? 객체의 타입에 답이 있다. 타입 자체는 파이썬 객체이다. (reference count와 type 자체 type도 있지만, 거의 모든 타입은 타입의 타입이다.) 하지만 refcount, 타입의 타입에 더하여, 더 많은 필드가 type objects를 설명하는 C 구조가 있다.

[이](https://docs.python.org/3/c-api/typeobj.html#type-objects) 페이지에 타입에 대한 정보와 구조체의 정의를 가지고 있다. 이 정보는 **./Include/object.h: PyTypeObject**에서도 찾을 수 있다. 이 글을 읽을 때 가끔씩 정의를 참고하는것이 좋다. 타입 객체가 가지고 있는 많은 필드는 슬롯이라고 불리며 함수를 가리킨다 (또는 관련된 함수들을 가리키는 구조들). 이 함수는 파이썬 C-API 함수가 호출되어 해당 유형에서 인스턴스화 된 객체에서 작동할 때 실제로 호출될것이다. **PyNumber_Subtract**를 사용할 때 실제로 피연산자의 타입을 역참조하고 **'subtraction'** 슬롯의 type-specific subtraction 함수가 사용된다. 그래서 우리는 C-API 함수가 일반적이지 않고, 오히려 세부사항을 추상화하여 타입에 의존하여 모든 작업을 수행 할 수 있는 것처럼 보인다. (정상 작동은 **TypeError**를 발생시키는 것이다.)



**PyNumber_Subtract**를 자세히 살펴보자. 일반적인 two-argument 함수  `./Object/abstract.c: binary_op`호출하고 숫자형 슬롯 **nb_subtract**에서 작동하도록 지시한다. (숫자형 슬롯 **nb_negative**, 시퀀스형 슬롯 **sq_length**와 같은 다른 함수에도 유사한 슬롯이 존재한다.) **binary_op**는 **binary_op1**을 둘러싼 오류 검사 래퍼이다. `./Objects/abstract.c: binary_op1`은 **BINARY_SUBTRACT**의 피연산자 v, w를 받을 다음 `v->ob_type->tp_as_number`라는 v가 숫자로 사용될 수 있는 방법을 나타내는 숫자 슬롯을 역참조하려 시도한다. **binary_op1**은 **tp_as_number -> nb_subtract**에서 빼기를 수행하거나 특 수 값인 **Py_NotImplemented**를 반환하여 이들 피연산자가 서로 관련하여 **insubtracticable**임을 알린다. (이로 인해 TypeError 예외가 발생한다.)



객체 동작 방식을 변경하려면 코드에서 **PyObjectType** 구조를 정적으로 정의하고 적절하게 슬롯을 채우는 확장을 C로 작성할 수 있다. 그러나 파이썬에서 자신의 타입을 생성할 때(`class Foo(list): pass`처럼 새 타입을 생성할 때), 수동으로 C 구조체를 할당하지 않고 슬롯을 채우지 않는다. 이러한 타입은 built-in 타입과같이 어떻게 작동하는가? 답은 상속이다. 타입화가 중요한 역할을 한다. Python은 **list**나 **dict**같은 built-in 타입을 가지고 있다. 앞서 말했듯이 이러한 타입에는 슬롯을 채우는 특정 함수집합이 있으므로 변수의 인스턴스가 변경될 수 있는 값 시퀀스 또는 키와 값의 매핑과 같은 특정 방식으로 동작한다. 파이썬에서 새로운 타입을 정의할 때, 그 타입에 대한 새로운 C 구조체는 다른 객체와 마찬가지로 힙에 동적으로 할당되고, 슬롯은 그것이 상속되는 타입(베이스라고 함)으로부터 채워진다. 슬롯이 복사되기 때문에 새로 생성된 하위 유형은 기본적으로 베이스와 동일하다. 파이썬은 또한 **object**(**PyBaseObject_Type** in C)라는 특징없는 기본 객체 유형을 가지고 있다. 이 객체 유형은 대부분 **null** 슬롯을 가지며 특정 기능을 상속받지 않고 확장할 수 있다.



그래서 결코 순수 파이썬 타입을 생성하지 않는다. 반드시 하나를 상속받는다. (명시적으로 상속하지 않고 클래스를 정의하면 암시적으로 객체를 상속받는다.) 물론 모든것을 상속받을 필요는 없다. 앞부분의 코드에서 이야기했듯 순수 Python으로 작성된 타입의 동작을 변형시킬 수 있다. Bar 클래스의 `__call__`을 설정함으로서, 그 클래스의 인스턴스를 호출 가능하도록 만들었다. 클래스가 생성되는 동안 **tp_call** 슬롯이 `__call__` 메서드와 연결한다는 것을 알았다.

`./Objects/typeobject.c: type_new`이 기능이 정교하고 중심적인 기능이다. 이후에 다시 이 함수를 볼것이지만, 지금은 마지막 부분에 있는 **fixup_slot_dispatchers(type)**을 본다. 이 함수는 새로 생성된 타입에 대해 정의된 올바르게 명명된 메서드를 반복하고 특정 이름을 기준으로 올바른 슬롯에 배치한다. (이 메서드들은 나중에!)



세세한 부분까지 해답을 찾을 수 없든 또다른 문제가 남아있다. 이미 생성된 타입에 `__call__`메서드를 설정하면 해당 타입의 인스턴스에서 호출 가능하게된다. (이미 해당 타입으로 인스턴스화된 객체조차) 타입이 객체이고 타입의 타입이 타입이라는 것을 떠올리자. 그래서 클래스에서 작업할 때, 클래스를 호출하거나 빼거나, 실제로 클래스의 속성을 설정하는 등, 클래스의 `ob_type` 속성을 역참조하여 클래스의 타입이 타입임을 알 수 있다.  그런 다음 `type->tp_setattro` 슬롯을 사용하여 실제 속성 설정을 수행한다. 따라서 클래스는 자체적인 속성을 가질 수 있다. 그리고 자체 속성을 가지는 기능( `./Object/typeobject.c: type_setattro` )은 **fixup_slot_dispatchers**가 실제로 수정작업할 때 사용하는 (**update_one_slot**)를 호출한다. 클래스에 새 속성을 설정한 후, 다른 세부사항이 발견했다.