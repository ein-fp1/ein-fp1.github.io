---
title: "Architecture: 레거시 환경에서 대규모 트래픽 처리를 위한 메시징 서비스 적용"
categories:
  - Architecture
tags:
  - back-end
  - legacy system
  - rabbitmq
  - kafka
---

## 사례
![Architecture_01](/assets/images/20220430_01.png)
* 파트너 서비스와 실시간으로 양사간의 데이터를 동기화하기 위한 데이터를 주고받는 상태
* 파트너사측에 제공한 콜백 API에서 **Read Timeout** 또는 **Connection Timeout** 이 **간헐적**으로 발생하고 있다는 이슈 접수
* **MSA**와 **DDD**로 구성한 환경이지만 **Layered Architecture**의 고질적인 문제와 A Service의 도메인을 모호하게 정의함으로 인해 A Service 가 **과도한 책임**을 보유
* 지속적인 기능 업데이트로 인해 B Service와 C Service의 부하가 증가하여 A Service와 두 서비스간의 **응답지연시간**이 대폭 증가
* 파트너사측 담당자와 접수된 이슈를 **7일 내 조치**하기로 협의

## 사용된 기술
* Spring Boot 2.x
* Netflix Zuul
* RabbitMQ
* AWS ECS

## 대응 방안
![Architecture_02](/assets/images/20220430_02.png)
* 스케일 아웃으로 임시 조치
* 파트너사의 요청에 대한 책임을 내부 시스템이 가질 수 있도록 시스템 구성 변경
* 사내에 구축한 **메시징 서비스**를 이용하여 프로듀서와 컨슈머 추가
* 시스템 구성 변경으로 인한 **리스크 최소화**
  * Netflix Zuul을 이용하여 구축한 External Gateway 에 프로듀서 역할 추가
  * 컨슈머 역할은 A Service가 가져가도록 처리

## 결과
* Read Timeout 및 Connection Timeout 이슈 해결

## 향후 과제
* A Service **책임 분할**
* 컨슈머 및 프로듀서 역할을 각각의 모듈에서 분할
* Layered Architecture의 문제점 개선
  * 향후 **클린 아키텍처** 도입