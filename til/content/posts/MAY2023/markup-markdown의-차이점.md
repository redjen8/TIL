---
title: "Markup Markdown의 차이점"
date: 2023-05-28T21:30:00+09:00
draft: false
author: redjen
---

개발을 하면서 노션, 깃허브 등에서 흔히 사용하는 마크다운 문법이라는게 존재한다.
반면 FE 개발 시에 마크업 개발이라는 용어도 접할 수 있었다.

두 단어는 어떻게 다르고, 어떤 유사성이 있을까?

https://samsara-ku.dev/common_sense/difference-between-markup-and-markdown/

## 마크업 언어

HTML은 '하이퍼 텍스트 마크업 언어'이다. 

https://ko.wikipedia.org/wiki/%EB%A7%88%ED%81%AC%EC%97%85_%EC%96%B8%EC%96%B4

> 마크업 언어는 태그 등을 사용해서 문서나 데이터의 구조를 명기하는 언어의 한 가지이다.

마크업 언어는 즉 문서의 구조, 서식, 각 부분 간의 관계를 제어하기 위해 특별한 기호를 삽입할 수 있는 시스템이라고 보면 된다.

대표적으로 잘 알려져 있는 마크업 언어의 종류로는 HTML, XML, SGML 등이 있다.

## 마크다운 언어

마크다운 언어는 마크업 언어의 한 종류이다. 마크업의 정 반대이기 때문에 붙여진 이름이 아니다. 

https://daringfireball.net/projects/markdown/

- 마크다운은 텍스트를 HTML로 바꾸는 변환 툴이다.
- 마크다운을 사용하면 읽기 쉽고 쓰기 쉬운 일반 텍스트 형식을 사용해 작성한 다음 구조적으로 유효한 HTML / XHTML로 변환할 수 있다.

마크다운은 두 가지로 구성되어 있다.
1. 평문 텍스트 포매팅 문법 그 자체
2. Perl로 작성된 소프트웨어 툴 (평문 텍스트를 HTML로 변환하는)