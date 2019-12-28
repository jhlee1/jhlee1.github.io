---
layout: post
title: "AWS Cloudwatch Logs insight에서 에러로그 찾기"
date: 2019-10-02 13:09:29 +0900
tags: ["devops", "aws"]
categories: [devops]
---

## I. 사용 이유
- 기존에 log groups에서 log stream에서 기본 필터링(Filter Events)에 입력한 단어가 포함된 행을 찾는 방식의 경우 너무 많은 데이터가 검색됨 => 정확한 검색의 필요성
- Query를 이용한 검색을 통해 에러 로그를 찾을 수 있음
- 로그에 대한 통계 데이터 등 다양한 값을 얻어올 수 있지만 일단 쉽게 에러 찾는 방법으로 사용하기

## II. Fields
- AWS에서 많은 field를 제공하지만 에러 로그를 찾기 위해선 아래 사항만 알면된다.
	- @ingestionTime = 해당 이벤트 발생 후 cloudwatch에서 log를 받은시간
	- @logStream = 어떤 로그 그룹에서 나온 것인지 알려준다. 클릭하면 해당 로그 그룹으로 이동
	- @message = 실제 로그 메세지
	- @timestamp = 실제 이벤트 발생 시간

## III. Filter
- like 문 
	- SQL의 LIKE문과 같다
	- Regex 사용 가능
	- 아래 문장은 case-insensitive로 @message에 error가 포함된 로그를 찾기
	- filter @message like /(?i)(error)/
- 기본 비교 (=, !=, <, <=, >, >=) 
- 기본 연산 ( x + y, x - y, x * y, x / y, x ^ y, x % y)
- 기타 연산자 및 함수들은 아래 링크 참조
	- https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html

## IV. limit
- SQL의 Limit과 똑같음
- limit 20
	- 매칭되는 데이터 중 20개 가져오기
- 거의 쓸 일 없음

## V. sort
- 이것도 마찬가지로 SQL과 똑같음
- sort @timestamp desc
	- timestamp 기준으로 내림차순 정리

## VI. Examples 및 사용법
1. 서버는 market-prod를 고른다
2. 서버를 고르고 날짜를 설정
3. 쿼리 작성( 각 명령은 | 로  구분)
4. Run Query를 누른다

EX) 
```
fields @timestamp, @message # @timestamp와 @message를 가져옴
| sort @timestamp desc #timestamp 기준으로 내림차순 정렬
| limit 200 # 최대 200개만 뽑아오기
| filter @message like /(?i)error/ # 대소문자 구분 없이 message에 error라는 단어가 포함된 것만 필터링
```

## VII. 참고 링크
- https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html


