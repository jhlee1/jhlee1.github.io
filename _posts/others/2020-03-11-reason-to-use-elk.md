---
layout: post
title: "ELK stack을 써야하는 이유"
date: 2020-03-11 11:33:54 +09:00
tags: ["kibana", "logstash", "elasticsearch", "apm"]
categories: [others]
---

## I. Performance

- 지난 일주일간 에러로그 조회 방법 비교 
  - 클라우드 워치
    - 클라우드 워치의 경우 조회 범위가 넒어질수록 속도가 많이 느려짐
    - 현재 MSA구조에 모든 log가 하나의 그룹에 쌓임
    - EX) 지난 5일간 file 서버 로그 전체 조회
      - Indexing이 잘 처리 안되있어서 엄청 오래걸림 43초쯤에서 꺼버렸음
    ```
    fields @logStream, @timestamp, @message
    | sort @timestamp desc
    | filter @logStream like 'file
    ```
  
  - ElasticSearch에서 처리

    - 5초내로 처리
    - 한번 조회하면 캐싱처리로 더 빠름

## II. Visualization & Filtering


- 단순히 줄 글로 있는거보다 훨씬 눈에 보기 쉬움

- [Kibana visualize 활용](http://apm.market.ogq.me/app/kibana#/visualize?_g=())

- 각 태그별 필터링 가능 EX) 서버별, log level 별 등등


  - Cloudwatch의 경우 에러 확인하려면 매번 쿼리를 복붙해야되고 한번에 볼 수있는 에러 범위가 좁다

      - Elaticsearch의 경우 필터링으로 클릭 몇번이면 거를 수 있음
      - apm에선 에러만 모아서 비슷한 유형을 정리해서 보여줌

```
fields @logStream, @timestamp, @message
| sort @timestamp desc
| filter @message like /(?i)exception/
| limit 200
```

## III. Monitering

- apm 사용으로 쿼리를 몇번 보내는지, 각 쿼리별 시간 측정 + Rest 통신별 시간 측정 가능

## IV. Cost

- 아직 로그 데이터가 얼마 쌓이지 않아서 필요한 인스턴스 크기를 예측할 수 없지만, 아마 더 쌀수도? 

- m5d.large의 On demand 기준 시간당 \$0.139, 한달에 ​\$100 정도 / RI로 처리하면 더 쌀듯
- 지난달 CloudWatch 비용 - $273.58

## V. ETC


- 그럼에도 갈아탈 수 없는 이유? 

  - 아직 어색하기 때문에
  - 그냥 단순하게 쭉 나열된게 더 편할때가 있을수도 있음
- 안정성 
  - Instance가 죽는 경우를 고려해야함
    - 아직 상용서버에서 사용할 경우 어느정도 instance 크기가 필요한지 측정이 안됬음
  - AWS 자체에서 제공하는 CloudWatch를 따라갈 수 없음
  - AWS ES를 쓰면 안정적이겠지만 너무 비쌈 ㅠ
