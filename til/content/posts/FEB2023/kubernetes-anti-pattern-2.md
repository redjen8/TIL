---
title: "Kubernetes Anti Pattern 2"
date: 2023-02-03T16:05:34+09:00
draft: false
author: redjen
---

## 6. 업데이트나 수정 사항들을 파드를 죽여서 재시작 과정에서 새 도커 이미지를 받아오도록 하는 배포 방식을 사용하는 것

`latest` 태그를 사용해서 업데이트를 하는 것과 마찬가지로, 실행 중인 파드들을 죽여서 새 업데이트를 반영하는 것을 코드에 대한 버저닝이 이뤄지지 않기 때문에 나쁘다.

> 프로덕션 환경에서 업데이트된 도커 이미지를 가져오기 위해 파드들을 죽이는 것을 하지 마라.

프로덕션 환경에서 버전이 배포된 이후에는 덮어씌워져서는 안된다. 만약 문제가 생겨서 코드를 롤백해야 할 일이 생긴다면, 언제 어디서 잘못되었는지 어떤 버전으로 돌아가야 하는지에 대한 힌트를 얻을 수 없기 때문이다.

다른 문제점으로는, 새 도커 이미지를 받기 위해 컨테이너를 재시작하는 것이 항상 동작할 것이라는 기대를 할 수 없기 때문이다.

배포의 파드 템플릿 (`.spec.template`)이 수정되는 경우에만 배포 롤 아웃이 트리거된다. (템플릿의 라벨이나 템플릿의 컨테이너 이미지가 업데이트 되는 경우) deployment 자체를 스케일링 하는 변경 사항등에서는 롤아웃이 트리거 되지 않는다. 

올바르게 deployment를 트리거하기 위해서는 `.spec.template`을 수정해야 한다.

또, 파드들을 새 도커 이미지를 받아와서 업데이트하기 위한 방법으로는 코드를 버저닝하고 deployment spec에 의미 있는 태그(`latest`가 아닌)를 붙이는 것이다.  쿠버네티스는 해당 경우 다운 타임 없이 업그레이드를 수행할 것이다.

1. 쿠버네티스가 새 이미지로 새 파드를 시작한다.
2. 헬스 체크가 통과할 때까지 기다린다.
3. 오래된 파드를 삭제한다.

## 7. 동일한 클러스터 내에 프로덕션 워크로드와 비프로덕션 워크로드를 섞는 것

쿠버네티스는 네임스페이스 기능을 지원한다.
- 네임스페이스는 사용자들에게 동일 물리 클러스터 내의 다른 환경을 효율적으로 관리할 수 있도록 한다.
- 예를 들어 동일한 클러스터 내에서 스테이징과 프로덕션 환경을 실행시키도록 설정하여 리소스와 비용을 절감할 수 있다.

하지만 쿠버네티스를 개발 환경에서 사용하는 것과 프로덕션에서 사용하는 것에는 큰 차이가 있다.

동일 클러스터 내에 프로덕션 워크로드와 비프로덕션 워크로드를 혼재해서 사용하기 위해서는 고려해야 할 점들이 많다.
- 예를 들어, 프로덕션 환경의 성능을 제대로 내기 위해서는 리소스 제한을 고려해야 한다.
- 일반적으로 프로덕션 네임스페이스에는 quota를 설정하지 않고 비프로덕션 네임스페이스에는 quota를 설정한다.

환경의 고립 또한 고려해야 한다. 개발자들은 프로덕션 환경에 대해 훨씬 더 많은 접근에 대한 권한이 필요하고 이 권한들을 잘 관리해야 한다.
네임스페이스는 서로의 존재를 모르지만 기본적으로는 완전히 고립된 존재가 아니다. 이것은 개발 네임스페이스가 테스트, 스페이징, 프로덕션에 사용될 수 있다는 말이기도 하다.

하지만 리소스 제한, 성능, 보안, 안정성 등을 철저히 테스트하는 것은 시간을 많이 필요로 하는 작업이다. 때문에 비프로덕션 워크로드와 동일한 클러스터에서 프로덕션 워크로드를 실행하는 것은 권장하지 않는 일이다.

> 대신 개발, 테스트, 프로덕션에 대한 분리된 클러스터를 사용해야 한다.

클러스터를 환경마다 분리함으로써, 고립과 보안을 둘 다 챙길 수 있다. 

## 8. 블루/그린이나 카나리 배포를 미션-크리티컬한 배포에 사용하지 않는 것

대부분의 현대 어플리케이션은 배포를 자주한다. (배포는 한 달에 몇 번 또는 하루에도 여러 번 이루어질 수 있다.) 
MSA를 사용하면 서로 다른 컴포넌트들이 서로 협력하며 작동하는 동안 서로 다른 주기로 개발, 관리, 배포될 수 있다. 그리고 이 업데이트 과정에서 어플리케이션을 24/7 가동시키는 것이 매우 중요하다.

쿠버네티스의 기본 롤링 업데이트가 항상 모든 경우에 적합하지는 않다. 업데이트를 수행하는 일반적인 전략은 기본 쿠버네티스 롤링 업데이트를 사용하는 것이다. (`.spec.strategy.type == RollingUpdate`) 기본 롤링 업데이트는  `maxUnavilable` (사용 불가능한 파드의 백분율) 과 `maxSurge` (선택) 옵션을 설정해서 세부적으로 제어될 수 있다. 제대로 설정된다면 롤링 업데이트는 다운 타임 없이 점진적인 업데이트를 수행할 것이다.

하지만 이 경우 다음 버전으로의 업데이트를 수행한 이후에는 다시 돌아가기는 쉽지 않다. 프로덕션 환경에서 어플리케이션이 정상 작동하지 않을 경우 롤백하기 위한 계획을 항상 가지고 있어야 한다. 파드가 다음 버전으로 업데이트 되었을 때에는 deployment가 새 ReplicaSet을 생성하고, 이전 ReplicaSet들을 저장하고 있는다. (기본으로는 10개의 ReplicaSet을 저장하지만 `spec.revisionHistoryLimit` 설정을 통해 변경할 수 있다. ) 

저장되는 ReplicaSet들은 `app6ff34b8374`와 같은 랜덤한 이름으로 저장이되고, deployment 어플리케이션 yaml에서는 이 ReplicaSet들에 대한 참조를 찾을 수 없다. 대신 `ReplicaSet.metatada.annotaion` 을 통해 찾을 수 있고, `kubectl get replicaset app-6ff88c4474 -o yaml`으로 리비전을 트래킹할 수 있다. 롤 아웃에 대한 히스토리는 yaml 리소스에 `- record` 플래그를 통해서 로그를 남기지 않기 때문에 복잡하긴 하다.

수십, 수백, 수천의 deployment들에 대한 업데이트를 동시에 수행할 때에는 수많은 리비전들에 대해 트래킹하기는 사실상 불가능하다는 문제점이 있다.
또 다른 문제점으로는
- 모든 어플리케이션이 여러 버전을 동시에 실행할 수는 없다.
- 업데이트 과정 중에 클러스터의 리소스가 전부 고갈되어 전체 프로세스를 중단시킬 수 있다.

때문에 다음과 같은 배포 방법을 사용해야 한다.

### 블루 / 그린 배포

블루 - 그린 배포를 사용하면 이전 버전과 새로운 버전의 인스턴스가 동시에 존재한다. 
- 블루가 이전 버전이다.
- 그린이 새롭게 배포되는 레플리카다.
그린 환경이 테스트와 검증들을 모두 통과한 이후에 로드밸런서가 트래픽들을 전부 그린으로 돌려서, 그린 환경을 블루 환경으로 만든다. 

블루 - 그린 배포는 다음과 같은 이점을 가진다.
- 두 개의 full 버전들이 관리되기 때문에 롤백이 매우 용이하다. 로드 밸런서를 스위칭하기만 하면 되기 때문이다. 
- 프로덕션 환경으로 직접 배포하지 않기 때문에 그린에서 블루로 갈 때 부하가 적다.
- 트래픽 리다이렉션은 즉시 일어나기 때문에 다운 타임이 발생하지 않는다.
- 트래픽 스위치 전에 실제 프로덕션 환경을 검증할 수 있는 테스트를 수행할 수 있다. (개발 환경은 프로덕션 환경과 매우 다르기 때문이다)

### 카나리 배포

카나리 배포를 사용하게 될 경우 전체 프로덕션 시스템에 영향을 주기 전에 잠재적인 문제들에 대해 테스트를 수행하고 핵심적인 메트릭들을 만족하는지 검사할 수 있다.

> 카나리 배포는 적은 수의 유저들에 대해서만 직접적으로 프로덕션 환경에 배포를 하여서 프로덕션 환경에서 테스트를 하는 방식이다. 

카나리 배포에서는 사용자들에 대해 퍼센트 기반, 위치 기반, 클라이언트 타입, 결제 속성에 대해 라우팅 방법을 설정할 수 있다.

작은 규모로 배포를 한 이후에는 어플리케이션 성능과 발생하는 에러에 대해 면밀히 모니터링할 필요가 있다. 이 때 수집되는 메트릭들은 어플리케이션 품질에 대한 역치를 정의한다. 어플리케이션이 예상대로 동작한다면 더 많은 트래픽을 감당하기 위해 새 버전의 인스턴스를 생성할 수 있다.

카나리 배포의 또 다른 장점으로는
1. Observability (프로덕션 환경에서의 실제 사용자에 대한)
2. 프로덕션 트래픽에 대해 테스트 할 수 있다는 점 (개발 환경에서 경험하기 쉬운 찐 프로덕션 환경에서 수행하기 때문에)
3. 더 큰 규모의 배포가 이루어지기 전에 작은 규모의 사용자들에 대해 새 버전을 배포해서 피드백을 받을 수 있다는 점
4. 빠른 실패를 통해 전체 사용자에 대한 장애까지 번지지 않게 컨트롤할 수 있다는 점

## 9. 배포가 성공했는지, 실패했는지에 대한 메트릭을 설정하지 않는 것

어플리케이션의 헬스 체크에는 어플리케이션 레벨의 지원이 필요하다.
쿠버네티스를 활용한다면 컨테이너 오케스트레이션에서 더 많은 작업을 수행할 수 있다.
- 응용 프로그램에 대한 리소스 소비 제어 (네임스페이스, CPU / 메모리) 하여 어플리케이션이 너무 많은 리소스를 사용하지 않도록 방지
- 서로 다른 어플리케이션 인스턴스 간 이뤄지는 로드밸런싱 - 리소스 부족이나 호스트에 문제가 생겼을 떄 해당 어플리케이션 인스턴스를 다른 호스트로 이동
- 자가 치유 (컨테이너가 크래시 되었을 때 재시작)
- 새 호스트가 클러스터에 추가되었을 때 추가 리소스들을 자동으로 배분
- 등등..

자동으로 해주는 것들이 워낙 많기 때문에 메트릭을 수집해야 하나? 와 같은 생각이 들 수 있다. 하지만 성공적인 배포는 운영 업무의 끝이 아니다. 예상치 못한 돌발 상황에 대비해서 선제적으로 액션을 취해야 한다.

모니터링해야 할 계층들은 수 없이 존재하고, k8s의 동적인 특성 때문에 문제를 해결하기 복잡하다. 예를 들어 사용 가능한 리소스를 주의 깊게 관찰하지 않으면 파드의 자동 리스케쥴링으로 인해 용량 문제가 발생할 수 있고, 어플리케이션이 크래시되거나 배포되지 않을 수 있다. 프로덕션 환경에서 이와 같은 일이 발생한다면 곧바로 장애로 이뤄진다.

그럼 어떤 항목에 대해서 모니터링을 해야 할까? 관찰해야 할 계층들은 너무 많고, 엔지니어들에게는 합리적으로 낮은 유지 보수에 대한 부담을 유지시켜야 한다. 쿠버네티스에서 실행 중인 어플리케이션에 장애가 발생할 경우, 특히 모놀리식이 아닌 MSA를 사용하는 경우 조사해야 할 로그, 데이터, 컴포넌트가 기하급수적으로 늘어난다.

어플리케이션의 성능을 수집하는 것은 전반적으로 어플리케이션을 개선하는데 도움이 된다. 또한 컨테이너, 파드, 서비스, 클러스터 전체에 대한 전체적인 뷰가 필요하다. 어플리케이션이 리소스를 어떻게 사용하는지 알 수 있다면 쿠버네티스를 사용해서 병목 현상을 더 잘 감지하고 제거할 수 있다. 프로메테우스, 그라파나와 같은 툴들을 사용하면 전체 어플리케이션에 대한 뷰를 쉽게 얻을 수 있다.

다음과 같은 핵심 지표들은 쿠버네티스 공식 문서에서 지속적으로 트래킹을 권고하는 사항들이다.
- 현재 실행 중인 파드들과, 그 파드들의 deployment
- CPU, 메모리 사용량, 디스크 IO와 같은 리소스 메트릭들
- 컨테이너 네이티브한 메트릭들
- 어플리케이션 메트릭들

## 10. 클라우드 벤더들의 쿠버네티스나 서버리스 컴퓨팅 서비스들에 의존하는 것

클라우드 벤더들에 의존하는 것은 서비스를 클라우드에 배포해서 얻을 수 있는 주요 가치인 컨테이너 유연성을 없앤다. 올바른 클라우드 벤더를 선택하는 것은 쉬운 일이 아니다. 각각의 벤더들은 고유한 인터페이스, Open API, 독자적인 스펙과 표준을 제공하고 있다. 추가적으로 어떤 벤더는 요구 사항이 갑자기 변화했을 때에만 더 적합한 경우가 있을 수 있다.

컨테이너는 플랫폼 불가지론적이고 portable하고, 모든 큰 벤더들은 cloud-agonistic한 쿠버네티스 기반을 가진다. 때문에 클라우드 벤더를 변경할 때에는 어플리케이션을 변경하지 않아도 된다.

벤더의 영향력을 최소화 시키기 위해서는 다음과 같은 것들이 고려될 수 있다.

### 1. 비용에 대한 트래킹 수행

entry - exit 전략을 잘 짜야 한다. 많은 벤더들은 entry가 쉽고, 스케일 업했을 때 요구되는 비용에 대해 고려할 수 있다.

자동 갱신이나, 조기 종료 비용, 그리고 공급자가 exit 했을 때 도움이 될 수 있는 과정을 제공하는지 체크해봐야 한다.

### 2. 어떤 클라우드에서도 돌아갈 수 있는 어플리케이션 설계

이미 클라우드에서 개발 중이고 클라우드 네이티브한 철학을 사용하고 있다면 아마 당신의 어플리케이션은 다른 벤더로 옮기기 쉬울 것이다. 

- 어플리케이션에서 사용 중인 서비스와 기능들이 portable 한 지 확인
- deployment와 프로비저닝 스크립트들이 클라우드 종속적인지 확인
	- 어떤 클라우드 벤더들은 다른 벤더로 마이그레이션 하기 어려운 툴을 제공하고 있기 때문
- Git과 CI/CD들과 같은 DevOps 환경이 어떠한 클라우드 환경에서도 돌아갈 수 있는지 확인
- 클라우드 벤더가 변경되었을 때 테스트 절차가 변경되어야 하는지 확인

> 또는 여러 클라우드 벤더를 사용하는 전략을 사용할 수도 있다.

여러 클라우드 벤더를 사용한다면 어플리케이션에 적합한 벤더를 더 잘 선택할 수 있다. 

