---
title: "Python의 Array와 List"
date: 2023-02-09T22:50:28+09:00
draft: false
author: redjen
---

어제는 Javascript의 배열에 대해 알아봤었는데, 파이썬의 List도 여러 타입의 데이터가 들어갈 수 있었던 것 같아 추가적으로 찾아보다 보니 재밌는 점이 있었다.

> List와 Array가 다른 자료 구조다?

https://www.geeksforgeeks.org/difference-between-list-and-array-in-python/

## 파이썬의 List

- 서로 다른 데이터 타입의 원소들을 포함할 수 있다.
- 선언하기 위해서 모듈을 명시적으로 import 할 필요 없다.
- arithmetic한 연산을 직접적으로 다룰 수 없다.
- 보다 짧고 간단한 데이터를 담기 위해 선호된다.
- 유연하기 때문에 데이터를 쉽게 추가하고, 삭제할 수 있다.
- 전체 원소가 명시적인 반복문 없이 `print` 될 수 있다.
- 쉽게 원소들을 추가하는 대신 더 많은 메모리를 사용한다.
 
## 파이썬의 Array

- 동일한 데이터 타입의 원소들만 포함할 수 있다.
- 선언하기 위해서 모듈을 명시적으로 import해야 한다. (`import array`)
- arithmetic한 연산을 직접적으로 다룰 수 있다.
- 보다 긴 시퀀스의 데이터를 담기 위해 선호된다.
- 덜 유연하기 때문에 원소의 추가와 삭제는 신중히 이뤄져야 한다.
- 전체 원소는 `print` 되기 위해 명시적인 반복문을 사용해야 한다.
- 사용하는 메모리 크기가 보다 적다.

그런데.. 나는 파이썬 객체들이 실제로 어떻게 메모리를 사용하는지 알고 썼는가? 하는 의문에 또 찾아봤다. 

## 여러 객체들은 파이썬에서 어떻게 메모리에 저장될까..그런데 CPython을 곁들인

https://stackoverflow.com/questions/4062752/in-what-structure-is-a-python-object-stored-in-memory

파이썬은 엄밀히 말하면 언어가 아니라 언어 명세이다.
때문에 구현하는 구현체 기준으로 메모리 구조가 다르다.

가장 보편적으로 사용하는 구현체인 CPython에서는 모든 객체들이 `PyObject`라고 부르는 C 구조체로 표현된다. CPython에서 객체를 저장하는 모든 것은 실제로 `PyObject*`를 저장하는 셈이다.

`PyObject`는 
- 객체의 타입 (다른 `PyObject`로의 참조)
- 참조 횟수를 저장한다. (악명 높은 파이썬의 GC에 사용되는 녀석)

이렇게 구조체 내에서 C로 정의된 타입들은 객체 자체에 저장해야 하는 추가 정보로 구조체를 확장하고, 가끔 추가적인 데이터를 별도로 할당한다. 

### Tuple

CPython에서 파이썬의 Tuple은 `PyObject` 구조체의 연장선에 있는 `PyTupleObject`로 구현된다. 

`PyTupleObject` 구조체는 아래의 정보를 포함한다.
- 자신의 길이
- 구조체 안에 가지고 있는 `PyObject` 포인터들

이 구조체는 정의로는 길이 1인 배열을 포함하지만, 실제 구현 시에는 `PyTupleObject` 구조체와 tuple이 들고 있어야 하는 모든 아이템들의 메모리 블록을 할당한다.

### String

CPython에서 String은 `PyStringObject`로 구현된다.

`PyStringObject`는
- 자신의 길이
- 캐시된 해시 값
- 일부 문자열에 대한 캐시
- 실제 데이터에 해당하는 `char*`

때문에 Tuple과 String은 CPython에서 연속된 메모리에 할당된다.

### List

CPython에서 List는 `PyListObject`로 구현된다.

`PyListObject`는 
- 자신의 길이
- 저장하고 있는 데이터의 `PyObject**` 
- 데이터에 대해 할당한 메모리를 트래킹하는 `ssize_t`

파이썬은 `PyObject` 포인터들을 아무 곳에나 저장하기 때문에, `PyObject` 구조체를 할당된 이후에는 변경할 수 없다. 한번 할당된 후 변경 작업에는 해당 구조체가 사용되는 모든 포인터를 찾아서 변경해줘야 하기 때문이다.

리스트는 하지만 가변 길이의 자료구조이다. 때문에 데이터를 `PyObject` 구조체와는 별개로 저장해야 할 필요가 있었다. (Tuple과 String은 가변 속성을 띄지 않기 때문에 그럴 필요가 없다)

### Array

Array는 별도의 모듈 import를 통해 사용할 수 있는 특별한 자료구조이다.

CPython 기준 Array 명세는 다음 링크에서 찾아볼 수 있다.

https://github.com/python/cpython/blob/main/Doc/library/array.rst

CPython의 Array 구현은 다음 링크에서 찾아볼 수 있었다.

https://github.com/python/cpython/blob/main/Modules/arraymodule.c

알아본 바를 간략하게만 정리하자면,
- Array 모듈은 '기본 값들에 대한 배열을 간단히 표현할 수 있는' 객체 타입을 정의한다.
- 리스트와 유사하지만 저장되는 객체의 타입에 대한 제약이 존재하고, 이 타입은 객체 생성 시에 특정된다.
- Array 객체들은 인덱싱, 슬라이싱, concat, multiplication을 지원한다.

## 정리하자면

CPython 기준 Array와 List 모두 해당 객체의 포인터, 즉 레퍼런스를 담는 자료구조이다.

- 레퍼런스를 담기 때문에 배열 안에 들어가 있는 원소들에 대한 GC도 아무래도 조심스럽게 이뤄질 것이다.
- 배열 바깥에서 객체를 변경해도 해당 객체가 존재하는 모든 곳에 변경이 반영될 것이다. 
- 여래저래 '효율적'으로 구현하기 위해서는 신경써야 할 것들이 너무 많은 언어이다.

테스트를 해볼까.
```python
class Student(object):
        name = "name"
        age = 0
        major = "major"


me = Student()
me.name = "redjen"
me.age = 26
me.major = "CSE"

arr = []
arr.append(me)
print("name: {0}, age: {1}, major: {2}".format(arr[0].name, arr[0].age, arr[0].major))

me.name = "redjen8"
print("name: {0}, age: {1}, major: {2}".format(arr[0].name, arr[0].age, arr[0].major))
```

해당 코드를 실행하면 아래와 같이 나온다.
```bash
python3 object_test.py
name: redjen, age: 26, major: CSE
name: redjen8, age: 26, major: CSE
```

이를 통해 파이썬의 리스트가 레퍼런스를 저장하는 것을 알 수 있었다.