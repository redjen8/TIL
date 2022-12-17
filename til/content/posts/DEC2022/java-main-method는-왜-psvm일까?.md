---
title: "Java Main Method는 왜 Psvm일까?"
date: 2022-12-17T21:01:43+09:00
draft: false
author : redjen
---

# Java Main Method는 왜 public static void main 일까?

[원문 링크](https://www.geeksforgeeks.org/java-main-method-public-static-void-main-string-args/)

## 1. main 메서드가 public이어야 하는 이유

JVM이 클래스 바깥에서 호출할 수 있도록 해야 하기 때문

## 2. main 메서드가 static이어야 하는 이유

static 키워드는 무엇인가? - static 키워드의 특성.
static 키워드를 붙이면 자바 클래스의 인스턴스 생성 필요 없이 바로 호출 가능
이는 main 메서드를 JVM이 실행할 때 굳이 인스턴스를 생성할 필요 없이 불필요한 메모리 낭비를 줄이기 위함임

## 3. main 메서드가 void여야 하는 이유

C/C++ 의 경우 main 함수를 관례상 int를 리턴하는 함수로도 많이 사용한다
하지만 Java에서는 좀 다르다 - 이는 JVM 위에서 굴러가는 자바 언어의 특성 때문
C/C++의 경우 컴파일 한 결과물을 실행 파일의 형태로 바로 실행할 수 있다
이렇게 실행된 실행 파일은 프로세스로써 운영체제의 커널단에서 관리된다.
하지만 자바는 어떤가?
자바는 JVM위에서 동작하는 언어... 
javac로 클래스 파일 생성한 후 java 명령어 통해 컴파일 된 클래스 파일을 실행한다
JVM은 main 메서드를 main 쓰레드로써 돌리는데, 이는 운영체제 레벨에서 어떤 자바 파일이 실행되고 있는지 모르게 한다. 자바 프로그램은 운영체제의 프로세스로 관리되는 것이 아니기 때문이다
자바 프로그램에 리소스를 직접적으로 할당하지도 않고, 프로세스 테이블에도 들어가 있지 않는다.

자바 main 메서드가 리턴 값을 가진다면 누구에게 리턴 값을 전달할 것인가?
JVM에게 전달할 필요가 있나? 어차피 JVM은 자바 프로그램을 main 메소드가 끝나는 대로 종료할 텐데 -> 불필요하다.
하지만 JVM은 운영체제의 프로세스로 동작한다. 운영체제는 JVM 프로세스에게 리소스를 할당하고, 관리한다
따라서 JVM은 종료될 때 운영체제에게 알려줘야 할 의무가 있고, 이게 JVM이 exit status를 가지는 이유
exit status를 int형으로 리턴한다.. java.lang.Runtime.exit 이나 System.exit 등으로 운영체제야 나 이렇게 끝났어 하고 알려준다 

## 4. main 메서드가 main 이름이어야 하는 이유

관례. JVM이 메인 쓰레드로 돌리기 위해 찾아야 할 단 하나의 메서드 이름을 main으로 정하기로 했기 때문

## 5. main 메서드가 String\[\] args 를 인자로 받는 이유

CLI에서 받는 입력을 문자열로 받기 때문
CLI에서 문자열로 받는 파라미터 이름을 args로 한다는 의미
public static void main (String... choonsik) 도 됨.. 파라미터 니 이름은 이제부터 춘식이여

## 6. 번외 : main 메서드 없이도 실행이 될까?

```
class Test {

        static {

                System.out.println("Test");

                System.exit(0);

        }

}
```

Java 11에서 테스트 해봤는데 안된다.. 
Test 클래스에서 기본 메소드를 찾을 수 없습니다. 다음 형식으로 기본 메소드를 정의하십시오~
Java 7에서부터는 main 메서드를 기본으로 스캔하기 때문에 안된다!


