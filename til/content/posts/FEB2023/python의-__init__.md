---
title: "Python의 __Init__"
date: 2023-02-10T21:34:36+09:00
draft: false
author: redjen
---

파이썬에서도 클래스를 활용한 객체 지향 프로그래밍을 할 수 있다.
그런데 `__init__`함수가 계속 눈에 보인다. 이 함수는 어떤 역할을 할까?

https://stackoverflow.com/questions/625083/what-do-init-and-self-do-in-python

## `__init__` 함수의 역할

파이썬은 언어 명세이다. CPython으로 대표되는 실제 언어의 구현체는 따로 있는 것이 파이썬의 특징이다.

`__init__`은 파이썬 OOP에서 클래스 생성자에 해당한다.
C++의 `this`처럼 `self`  파라미터는 객체의 인스턴스 자신을 가르킨다.


## 파이썬 프로그래밍에서 `self`가 왜 그렇게 많이 사용되는가? / 왜 파이썬 클래스 메서드들은 `self` 파라미터를 필요로 할까?

### 파이썬은 특별해

https://stackoverflow.com/questions/2709821/what-is-the-purpose-of-the-self-parameter-why-is-it-needed

파이썬은 인스턴스 어프리뷰트들에 대한 레퍼런스를 가르키기 위한 특별한 문법을 지원하지 않는다. 

- 파이썬에서 메서드가 속한 인스턴스의 전달은 자동으로 이루어진다.
- 하지만 메서드가 속한 인스턴스를 받을 때에는 자동으로 이루어지지 않는다.

메서드의 첫번째 파라미터를 메서드가 불러일으켜진 인스턴스로 설정함으로써 함수의 역할을 수행하게 하는 것이다. (사실 `self`라는 이름 자체는 컨벤션이고, 이름 자체는 다른 어떤 것을 사용해도 상관 없다)

파이썬은
- 모든 것을 명시적으로 만들어서
- 무엇이 무엇인지 명확하게 하기 위함이다.
- 인스턴스 어트리뷰트에 할당할 때에는 어떤 할당의 대상이 되는 인스턴스가 어떤 것인지 알아야 하기 때문에 `self`가 필요하다.

```python
def some_method(self, arg1, arg2):
	# do something
```

위와 같은 함수가 있다고 가정했을 때, `test_object.some_method(arg1, arg2)`와 같은 메서드 호출은 파이썬 내부적으로 `TestClass.some_method(test_object, arg1, arg2)`로 변환된다.

### 다른 언어에서는..

다른 언어 (Java, C++) 등에서는 객체 내부에 정의된 메서드들에 대해서 파라미터를 '숨겨서' 전달한다.
파이썬에서는 이 행위가 일어나지 않기 때문에 명시적으로 '자기 자신'을 가르키는 레퍼런스를 전달해주어야 객체의 올바른 메서드로 동작하게 되는 것이다.

## 다시 돌아와서..

```python
class MyClass(object):
    i = 123
    def __init__(self):
        self.i = 345
     
a = MyClass()
print(a.i)
print(MyClass.i)
```

위 코드는 345, 123을 출력한다.
`MyClass()` 생성자인 `__init__`에 의해 생성된 인스턴스는 내부에 i 값으로 345를 가지게 된다.

반면 `MyClass.i` 값은 static처럼 동작하여 123을 출력하게 된다. 이 값은 인스턴스의 생성 없이 활용될 수 있는 값이라는 점이 차이가 있다.

> 객체가 내부적으로 가지는 private한 값을 할당해서 관리하기 위한 목적을 가진다면, 생성자인 `__init__` 내부에서 `self`에 대한 값을 명시적으로 초기화 해줘야 할 필요가 있다.
