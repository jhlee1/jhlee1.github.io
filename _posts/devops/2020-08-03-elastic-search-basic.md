---
layout: post
title: "Elastic search basic concepts"
date: 2020-08-03 11:08:00 +0900
tags: ["elasticsearch"]
categories: [devops]

---

입사초기에 정리했던 내용
![search_engine_diagram](/static/img/posts/devops/search_engine_diagram.png)

## I. 검색 엔진 작동 방식

- Crawler가 웹에 있는 웹페이지들을 방문하여 해당 페이지 내 + 링크된 페이지 + 링크된 페이지의 페이지 + 링크된 페이지의 페이지의 …. 의 모든 내용을 읽어와서 가공 후 저장
  - Webpage를 저장하는 DB에 저장
  - 페이지 내용, 중요도 등을 기준으로 가공하여 Index를 만들어 Index DB에 저장
- 사용자가 입력한 정보를 Index로 바꿔서 Index DB에서 검색
- Index DB에서 찾은 데이터를 Webpage를 저장해놓은 DB에서 가져오기
- 기존 RDBMS에 대용량의 정보를 가지고 있는 경우 검색에 많은 시간이 필요하다. 한줄 한줄 필드값을 비교 & 확인해야하기 때문에

## II. 설치 및 실행방법 (5.5.x && 6.6.x 사용)

- homebrew로 설치하면 최신 버전이 설치되기 때문에 직접 받아서 압축을 풀어야한다.
- [공식문서](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/getting-started-install.html)의 Installation example with tar를 참고하여 설치 (6.6 버전 기준이지만 5.5도 파일만 받아서 똑같은 방식으로 하면됨)
- 압축을 푼 폴더에서 bin/elasticsearch를 실행하면된다.
- config/elasticsearch.yml 파일에는 보안, 메모리 설정, 분산환경(클러스터/노드) 설정 등을 포함하고 있다.
- 두가지 버전을 사용하므로 path에는 등록하지 않는 것이 낫다.

## III. 기본 개념

- Lucene(루씬) 기반으로 개발된 오픈소스 분산 검색엔진
  - Lucene은 Apache에서 개발한 오픈소스 프로젝트
  - 확장 가능한 고성능 정보검색(IR, Information Retrieval) 라이브러리
  - Index와 Search를 간단하게 추가할 수 있음
- JSON을 데이터 모델로 사용
  - 타입을 자동으로 매핑
  - NoSQL처럼 사용 가능
- Index를 이용해서 빠르게 검색할 수 있음
- Cluster로 이루어짐
  - Cluster란 여러개의 노드(서버) 집합 
- REST API를 지원
  - Documenrt 추가, 검색, 삭제, 업데이트 API를 HTTP Call을 통해서 부른다
- RDBMS와 ElasticSearch 비교

| RDBMS    | ElasticSearch         |
| -------- | --------------------- |
| Database | Index                 |
| Table    | Type                  |
| Row      | Document              |
| Column   | Field                 |
| Schema   | Mapping               |
| Index    | Everything is indexed |
| SQL      | Query DSL             |

## IV. 참고링크

- <https://bakyeono.net/post/2016-06-03-start-elasticsearch.html>
- <https://www.popit.kr/%EB%9D%BC%EB%9D%BC%EB%B2%A8-%EC%97%98%EB%9D%BC%EC%8A%A4%ED%8B%B1%EC%84%9C%EC%B9%98-%EC%82%AC%EC%9A%A9-%EA%B2%BD%ED%97%98%EA%B8%B0-%EC%B4%88%EA%B8%B0%EC%9E%91%EC%97%85/>
- <https://12bme.tistory.com/171>