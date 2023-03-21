---
title: "Copy and Paste"
date: 2023-03-21T20:11:08+09:00
draft: false
author: redjen
---

우리가 컴퓨터를 다룰 때 가장 많이 사용하는 유틸리티는 무엇일까?

사내 메신저도, 인터넷 브라우저도 아닌 바로 클립보드라고 생각한다.

Copy and Paste는 OS 레벨에서 어떻게 동작할까?

https://en.wikipedia.org/wiki/Clipboard_(computing)

## 개요

클립보드는 OS가 제공하는 어플리케이션 프로그램 간 전달되는 단기 스토리지이다.

클립보드는 대부분 임시의 성격이 강하고, 익명이며, 컴퓨터의 RAM 안에 내용물을 보관한다.

클립보드는 또 다음의 특징을 가진다.
- cut, copy, paste 연산을 구체화 할 수 있는 API를 제공한다.
- 프로그램이 이 연산들을 구현하는 것은 프로그램의 몫이다.
  - 키 바인딩이나 메뉴 선택과 같은 다른 작업을 대신 수행할 수 있다.
- 어플리케이션 프로그램은 운영체제가 제공하는 클립보드 기능을 확장시켜서 사용할 수 있다.
- 클립보드 관리자는 클립보드에 대한 추가 제어를 가능하게 한다.
- 클립보드는 운영체제 간 구현이 다를 수 있으며, 심지어 같은 프로그램의 다른 버전에서도 서로 다를 수 있다.

## 역사

클립보드는 작은 텍스트 스니펫을 위한 작은 버퍼로 Pentti Kanerva에 의해 고안되었다.
- 초기에는 삭제한 텍스트들을 복구하기 위한 용도로 사용했다고 한다.
- 이후 Larry Tesler에 의해 연산들에 cut, copy, paste와 같은 이름을, 버퍼에는 clipboard라는 이름을 붙인다.

## 어떻게 저장할까?

어플리케이션은 직렬화된 객체 표현을 통해서 소통하거나, 보다 큰 객체의 경우 일종의 프로미스를 통해 전달한다.

그런데 어떤 상황에서는 공통된 형식의 데이터 전송이 추상 팩토리로 한번 감싸져서 이뤄질 수 있다.
- 예시로 MacOS에서 사용하는 NSImage 클래스는 클립보드에 저장된 이미지 데이터에 접근할 수 있게 한다.
  - 하지만 실제 이미지 객체에 대한 데이터는 숨겨져 있다.
- 클립보드 데이터를 송수신하는 어플리케이션들은 종종 허용 가능한 형식 변환을 제공하는 GUI 위젯을 사용하여, 그 사이에서 전송에 사용될 형식을 negotiate한다.
  - OS와 GUI 툴킷은 간단한 변환을 제공할 수도 있다. (rich text -> plain text)
- 최신 운영체제에서는 데이터 전송을 위한 다양한 타입 식별자를 제공한다.
  - MIME이나 Uniform Type Identifier와 같은 허용 가능한 매핑을 자동으로~

## 윈도우에서의 클립보드

윈도우에서는 클립보드에 저장되는 데이터에 대해 다양한 포맷을 지원한다.
1. standard formats (CF_BITMAP 이나 CF_UNICODETEXT)
2. registered formats (CF_HTML)
3. private formats (외부 제공이 아닌 용도)

윈도우 XP부터는 ClipBook Viewer라고 부르는 어플리케이션을 통해 클립보드 안의 데이터 접근이 가능해졌다.
보다 새로운 윈도우 버전에서는 Clipboard Managers라고 부르는 어플리케이션을 통해 접근한다.

윈도우에서는 `clip` 명령을 통해서 CLI에서 데이터를 클립보드에 저장할 수 있다. (powershell에서도 사용할 수 있다)

## MacOS에서의 클립보드

윈도우와 마찬가지로 MacOS에서도 클립보드를 위한 다양한 데이터 포맷을 지원한다.

클립보드에 저장되는 내용물은 Finder의 편집 메뉴를 통해 접근이 가능하다. 

실제로 저장되는 raw 데이터와 포맷들은 ClipboardViewer라고 부르는 어플리케이션을 통해 접근이 가능하다.

CLI에서는 `pbcopy` 명령을 통해 클립보드에 복사를, `pbpaste`를 통해 복사된 데이터를 붙여넣기 할 수 있다.

## Unix / Linux 에서의 클립보드

Unix와 Linux에서는 X Window System이라고 부르는 것을 주로 사용한다.

그리고 이 X Window System에서는 크게 3가지 종류의 클립보드를 지원한다.
1. PRIMARY
2. SECONDARY
3. CLIPBOARD

GNOME이나 KDE 같은 현대의 다양한 툴킷에서는 freedesktop.org 명세서에 적혀진 컨벤션을 따른다.

- CLIPBOARD는 전통적인 클립보드 문법을 따른다. 윈도우와 동일한 shortcut을 사용한다.
- PRIMARY는 X-11 특정된 메커니즘이다. 데이터가 하이라이팅을 통해 즉시 복사되고 마우스의 가운데 버튼을 통해 붙여넣기 된다.
  - 이렇게 복사된 데이터는 CLIPBOARD 와는 분리되고 내용물은 변경되지 않는다.
- SECONDARY는 PRIMARY를 대체하기 위해 고안되었으나 현재는 일관되지 않게 사용된다.

CLI에서는 `xclip`, `xsel` 두 가지 툴을 사용해서 클립보드에 접근할 수 있다.
```bash
$ # to paste standard output to the clipboard using xclip
$ echo text | xclip -in -selection clipboard
$ # to paste standard output to the clipboard using xsel
$ echo text | xsel --clipboard
```

OSX와 윈도우와의 결정적인 차이점은 CLIPBOARD 클립보드에는 실제로 데이터가 저장되는 것이 아니라 복사되거나 잘려진 데이터의 참조값이 저장된다는 점이다.

어플리케이션이 CLIPBOARD 선택에 대한 책임을 가지고, X 서버에 대한 통신을 하게 되는 구조이다.

이렇게 복사된 데이터를 붙여넣기 할 때에는 CLIPBOARD를 가지는 어플리케이션이 데이터와 제공 가능한 데이터 포맷들을 요청한다.

## 자바스크립트에서의 클립보드

인터넷 서핑을 하다 보면 어떤 컨텐츠는 복사 붙여넣기 할 수 없는 경우가 있다.

바로 자바스크립트에도 클립보드 데이터의 변화를 감지하는 클래스가 있기 때문에 가능한 일이다. (`ClipboardEvent`)

자바스크립트는 또한 클립보드의 내용물을 변경하거나 (`clipboardData.setData()`) 저장된 내용을 읽어오는 (`clipboardData.getData()`) 연산을 수행할 수 있지만, 보안 문제 때문에 모든 브라우저에서 사용한 것은 아니다.