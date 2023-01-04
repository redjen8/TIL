---
title: "Hpa란"
date: 2023-01-04T22:28:35+09:00
draft: false
author: redjen
---

장애 대응 wiki를 읽다가 HPA 가동 혹은 발동이라는 말이 많이 나와서 찾아봤습니다.

https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/ 참고

HPA란, Horizontal Pod Autoscaling의 줄임말
워크로드 크기를 수요에 맞게 자동으로 스케일링하는 쿠버네티스의 기능
근데 Horizontal 이라는 말이 붙었네? - 수평 스케일링

수평 스케일링 == 부하 증가에 대해 pod를 더 배치하는 것

수직 스케일링 == 이미 실행 중인 pod에 더 많은 리소스를 할당하는 것

크기 조절이 불가능한 DaemonSet에는 적용 불가능
kube-controller-manager의 --horizontal-pod-autoscaler-sync-period 파라미터에 의해 HPA 체크 주기를 설정

컨트롤러 매니저는 타겟 리소스를 찾고, pod를 선택하여 리소스 메트릭 API를 통해 메트릭을 수집한다.
- pod 단위 리소스 메트릭의 경우 컨트롤러는 대상으로 하는 각 pod에 대한 리소스 메트릭 API에서 메트릭 가져온다
- 그 후 목표 사용률 값 설정되면 - 각 pod의 컨테이너에 대한 동등한 자우너 요청을 퍼센트 단위로 사용률 값 계산
- 그 후 모든 대상 pod에서 사용된 사용률의 평균 또는 primitive value를 가져와서 원하는 레플리카의 개수를 스케일하는데 사용되는 비율 생성

HPA 컨트롤러는 원하는 메트릭 값과 현재 메트릭 값 사이의 비율로 HPA를 작동시킨다.

원하는 레플리카 수 = ceil \[현재 레플리카 수 * ( 현재 메트릭 값 / 원하는 메트릭 값) \]

즉, 원하는 메트릭 값에 비해 현재 메트릭 값이 높다면 레플리카 수가 증가할 것이고
반대로 현재 메트릭 값이 원하는 메트릭 값에 비해 낮다면 레플리카 수가 감소한다. 

HPA 대상은 스케일링 대상에서 리소스 사용량을 기준으로 스케일링 가능
	예) pod의 평균 사용률을 60%으로 유지하도록 설정 가능

kubectl의 HPA 지원 : kubectl create 명령어로 새 오토스케일러 생성 가능
- kubectl get hpa : 오토스케일러 목록 조회
- kubectl describe hpa : 오토스케일러 세부 사항 확인 가능
- kubectl delete hpa : 오토스케일러 삭제