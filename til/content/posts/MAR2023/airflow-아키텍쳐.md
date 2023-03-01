---
title: "Airflow 아키텍쳐"
date: 2023-03-01T20:42:04+09:00
draft: false
author: redjen
---

## Airflow란

Apache에서 만든 workflow를 빌드하고 실행할 수 있는 플랫폼이다.

https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html

- 각각의 workflow들은 DAG(Directed Acyclic Graph, 유향 비순환 그래프)로 표현된다.
- 각각의 workflow들은 Tasks 라고 부르는 개별 작업들을 포함한다.
- 각각의 workflow들은 의존성과 데이터 플로우를 가진채로 구성될 수 있다.

DAG는 Task들 간 의존성을 구체화시킨다.
1. 어떤 것을 실행할지에 대한 순서
2. 재시도를 실행하는 지에 대한 여부

Task들은 자기 자신이 어떤 일을 할지 서술하고, 데이터를 가져오고, 분석을 실행하고, 다른 시스템을 트리거하는 등의 여러 일을 수행할 수 있다.

## Airflow의 구성 요소들

- Scheduler: 스케쥴된 workflow들을 트리거하거나, task들을 executor에 보내서 실행하는 일을 수행한다. 
- Executor: task들을 수행하는 것이 주된 역할이다. 기본 Airflow 설치에서는 스케쥴러 안의 모든 것을 실행하지만 대부분의 프로덕션 환경에서는 task들을 worker들에 위임하는 방식으로 실행한다.
- Webserver: DAG들과 task들의 행동을 감시, 트리거, 디버깅 하기 편리한 사용자 인터페이스를 제공한다.
- DAG file folder: DAG들이 모여 있는 곳으로 scheduler와 executor에 의해 읽힌다.
- metadata database: executor와 webserver가 상태를 저장하고 scheduler에 의해 사용되는 곳이다.

## Workload

DAG는 tasks의 시리즈로 실행된다. 그리고 Task에는 흔히 3가지 타입이 존재한다.

- Operator: DAG들의 대부분을 빠르게 빌드하기 위해 사전에 정의된 Task이다.
- Sensor : 외부 이벤트가 발생하기를 기다리기만 하는, 특별한 operator의 한 종류이다.
- TaskFlow : `@task`로 데코레이팅 되어 있는 task로 패키징된 커스텀 파이썬 함수이다.

## 제어 흐름

DAG들은 여러 번 실행될 수 있고, 다수의 DAG 실행은 병렬적으로 이뤄질 수 있다.

DAG들은 '실행되는 인터벌'을 항상 포함하는 파라미터화되어 있지만, 다른 부가적인 파라미터들도 사용할 수 있다.

Task들은 서로 선언된 의존성을 가지고 있고, `>>`와 `<<` 연산자를 사용해 상호간의 의존성을 표현한다.

```python
first_task >> [second_task, third_task]
fourth_task << third_task
```

또는 `set_upstream`과 `set_downstream` 메서드를 사용해서 상기 코드와 동일한 효과를 낼 수 있다.

```python
first_task.set_downstream([second_task, third_task])
fourth_task.set_upstream(third_task)
```

이러한 의존성들은 그래프의 간선을 만드는 행위이고, Airflow가 Task들을 어떤 순서로 어떻게 실행할지를 설정하는 것이다.

기본적으로는 task는 upstream task들이 성공할 때까지 기다리지만 `Branching`, `LatestOnly`, `Trigger Rules` 등의 기능을 사용해서 커스텀될 수 있다.

task들 간에 데이터를 전달하려면 
1. XComs (Cross-Communication): task들에 작은 크기의 메타데이터를 push 하고 pull 할 수 있는 시스템
2. Storage 서비스 사용: 큰 파일들을 전달해야 한다면, 클라우드 서비스에 업로드 및 다운로드 해서 처리 가능
3. TaskFlow API: TaskFlow API는 자동으로 task들 간 데이터를 암묵적인 XComs 사용을 통해 전달

