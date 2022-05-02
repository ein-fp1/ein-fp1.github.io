---
title: "Architecture: 특정 API에 대한 대규모 트래픽 분산 처리"
categories:
  - Architecture
tags:
  - aws
  - application load balancer
  - scale-out
  - aws ecs
  - aws elb
---

## 사례
![Architecture_01](/assets/images/20220502_01.png)
* 프로덕트 환경에서 간헐적으로 서버 오류를 발생
* 운영자는 약속된 시간대에 Registration API를 이용하여 대규모 트래픽을 발생시키고 있고, 프로덕트에서 일반 사용자들도 해당 API를 사용중 
* Registration API 호출 시 파트너사와의 데이터 동기화를 위해 파트너사의 API를 호출
* 양사에서 10~100MB 크기에 달하는 디지털 콘텐츠를 가공하여 보관하기 위해 메시징 서비스를 이용하여 비동기로 처리중
* 파트너사측으로 부터 디지털 콘텐츠 처리에 대한 성공 여부를 회신받기 위해 콜백 API를 제공

## 환경 및 기술
* Spring Boot 2.x
* Netflix Zuul
* AWS ECS
* AWS ELB
* Bitbucket Pipeline
* Jenkins

## 설계 및 구현
### 내부 프로덕트 설정
![Architecture_02](/assets/images/20220502_02.png)
* 임시 방편으로 스케일 아웃으로 대응하였고, 운영자가 발생시키는 트래픽은 망을 분리하여 처리
* 운영자가 발생시키는 트래픽은 API요청 시 쿼리 스트링 추가
  * 예시) api.domain?bulk_request=Y
### 파트너사와의 API 호출 규칙 협의
![Architecture_03](/assets/images/20220502_03.png)
* 파트너사측에서도 망을 분리하여 운영하기 위해 API 호출 규칙 협의
  * 규칙 정의: bulk_request=Y
* 운영자가 발생시키는 트래픽은 쿼리 스트링으로 구분하여 요청
  * 예시) api.partner_service?bulk_request=Y
* 파트너사 시스템에서도 약속된 쿼리 스트링이 전달될 경우 콜백 API에도 망을 구분을 위한 약속된 쿼리 스트링을 추가
  * 예시) api.partner.domain?bulk_request=Y
### AWS 애플리케이션 로드밸런서 설정 예시
![Architecture_04](/assets/images/20220502_04.png)
* AWS 애플리케이션 로드밸런서에서는 위와 같이 설정
### 도커 이미지 동기화
![Architecture_05](/assets/images/20220502_05.png)
* 빌드 배포는 Bitbucket Pipeline으로 이루어지는 상태
* 서비스 변경사항 발생 시 Bulk Group에 반영하기 위해 Jenkins를 이용하여 Bulk Group 동시 빌드 및 배포

## 결과
* 일반 사용자 서비스 안정성 강화
* 인프라 관리 및 대규모 트래픽에 대한 모니터링 편의성 강화

