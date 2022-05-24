---
title: "Infra Structure: Full-Text Search 환경에서 Multi Match 쿼리 최신순 정렬 최적화"
categories:
  - Infra Structure
tags:
  - elasticsearch
  - newest-sort
---

## 개요
Full-Text Search 가 필요할 경우 Elasticsearch 와 같은 Full-Text Search Engine 을 사용하게 된다.
경우에 따라 최신순 정렬 시 일반적으로 Sort 쿼리를 활용하게 된다. 하지만 원하는 품질의 결과를 낼 수 없는 상황이 발생하게 된다.
본 문서에서는 Elasticsearch 에서 제공하는 함수를 이용하여 사례에서 발생하는 문제점을 개선한다.  

## 문제점

|    ID | Description    | Google Label |    Date    |
|------:|----------------|--------------|:----------:|
|     1 | 귀여운 고양이        | 고양이          | 2019-01-01 |
|     2 | 도도한 고양이        | 고양이          | 2019-01-02 |
| ===== | ============== | ==========   | =========  |
|   100 | 사랑스러운 강아지      | 강아지          | 2021-12-28 |
|   101 | 사랑스러운 고양이      | 고양이          | 2021-12-28 |
|   102 | 캣그라스(고양이풀)     | 식물           | 2021-12-29 |
|   103 | 동물 조각상         | 고양이, 조각상     | 2021-12-30 |
|   104 | 고양이 자세를 한 사람   | 사람           | 2021-12-31 |


* Elasticsearch 에 위와 같은 데이터가 저장되어있다고 가정
* Multi Match & Sort 쿼리를 이용하여 "고양이" 라는 키워드를 검색할 경우 104-103-102-101-...-2-1 순으로 정렬
* Description 필드보다 Google Label 필드의 부스팅값이 조금 더 높지만 Sort 쿼리에 Date 필드를 대입하게 되면 스코어와 관계없이 Date 필드로 우선정렬 

기계적인 방법으로 "고양이"라는 키워드의 검색 결과를 도출해냈지만 서비스를 이용하는 유저가 원하는 결과는 분명 아닐 것이다.
그러나 경우에 따라 Description 필드에 저장된 Document 도 추출해야할 수도 있다. 너무 오래된 Document 는 결과에서 제외해야할 상황도 발생한다.


## 환경 및 기술
* Elasticsearch 6.x
* Kibana 6.x

## 해결 방법
* Elasticsearch 에서 제공하는 Function Score Query 의 [Decay Function](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-function-score-query.html#function-decay) 활용
* Decay Function 이 가진 특성
![Graph1](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/images/decay_2d.png)
* Sort 쿼리 대신 Decay Function 을 활용하면 스코어가 반영된 최신순 정렬 가능
* 오래된 데이터는 결과에서 제외 가능
* 예시) 30일 이내(2022-01-01 기준) 등록된 "고양이" 키워드 검색 결과
  * 101-100-103-104 순으로 정렬 

## 결과
* 서비스에서 더 나은 최신순 검색 결과 제공

