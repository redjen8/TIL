---
title: "Github Api File Size Trouble Shooting"
date: 2023-03-09T19:23:37+09:00
draft: false
author: redjen
---

Github API를 사용하다 파일 컨텐츠를 불러올 수 없는 문제를 만났던 내용을 공유해보려 한다.

## 문제

[Github 클라이언트 API](https://github-api.kohsuke.org/)는 Github API를 좀 더 간편하게 사용할 수 있게 해주는 라이브러리이다.

기본적으로 Github API들은 REST API인데, 우리가 흔하게 얘기하는 REST가 아니라 HATEOAS를 얹은 진짜 정말 찐 REST API인 점이 인상 깊었다. 

[DEVIEW 2017 - 그런 REST API로 괜찮은가](https://deview.kr/2017/schedule/212)

하여튼, Github API를 잘 사용하다가.. 레포지토리 내부의 파일 컨텐츠를 가져오는 과정에서 문제가 생겼다.

> Unrecognized Encoding : None

## 원인을 찾기 위해서..

에러 코드에 `Encoding`이 있었고, 최근에 변경한 비즈니스 로직도 하필 변경 내용에 포함되어 있어서 맨 처음에는 파일 쓸 때의 인코딩이 문제인가? 하고 찾아봤었다.

하지만 변경 전 후와 비교해서 파일 쓰는 로직은 변경되지 않았었고, 문제는 다시 원점으로 돌아갔다.

그리고 [Github API 공식 문서](https://doc.github.com)를 보게 된다.

## 원인

https://docs.github.com/en/rest/repos/contents?apiVersion=2022-11-28#custom-media-types-for-repository-contents

Github API는 레포지토리 내 작은 크기의 파일의 경우 `application/json` MediaType을 사용해서 접근할 수 있게 허용한다. 

하지만 1MB가 넘는 파일의 경우 그렇지 않다. 반드시 특정 커스텀 MediaType을 지정한 API 호출을 통해서 파일 컨텐츠를 접근할 수 있게 한다.

`vnd.github.raw` MediaType으로 다시 파일 컨텐츠 요청을 보냈을 때에는 정상적으로 파일 컨텐츠를 받아올 수 있었다.

```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/OWNER/REPO/contents/PATH
```

## 왜 파일 크기에 따른 MediaType 제약 사항을 둘까?

사실 `application/json`은 간단한 요청에 대한 응답을 보내기엔 더 없이 좋은 포맷이지만,
크기가 큰 파일을 전송하는데에는 적합하지 않은 MediaType이다.

작은 크기의 파일은 가장 많이 사용하는 MIME type인 `application/json`으로 encoding된 파일 내용을 보내도 상관이 없지만

크기가 큰 파일을 전송하는데에는 File I/O 비용과, 막대한 네트워크 비용이 들기 때문에 별도의 MediaType을 두어 파일 전송에 특화된 방법을 사용하는 것이라 추측할 수 있었다.

- 클라이언트에서 어떤 파일을 서버로 전송할 때에는 `Multipart` 파일 업로드를 통해 수행한다.
- 서버에서는 `InputStream` 이나 `ByteArrayResource`를 리턴타입을 클라이언트에 전달함으로써 파일을 줄 수 있다.

https://stackoverflow.com/questions/35680932/download-a-file-from-spring-boot-rest-service

`vnd`는 찾아보니 vendor-specific한 MIME 타입을 나타내는 prefix라고 한다.

https://stackoverflow.com/questions/5351093/what-is-the-meaning-of-vnd-in-mime-types

`prs`는 personal한 MIME 타입을 나타내는 prefix라고 한다!