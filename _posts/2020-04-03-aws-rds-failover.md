---
layout: post
title: "AWS RDS failover"
date: 2020-04-03 18:50:00 +0900
tags: ["rds", "failover", "aws"]
categories: [devops]

---

## I. 자고 일어나니 개발서버 Master DB가 내려갔었다

원인은 분석해봐야겠지만 내려간 시간대에 트래픽이 급증한 것으로 봐선  Batch 작업 때문인 것 같다. 

## II. RDS의 Failover 매커니즘

- Master(Read / Write) - Replica(Read Only) 구조를 가지고 있다
- Master가 내려간 경우 Read만 처리하던 Replica가 Master 대신 Read / Write 처리를 한다. Master가 자동으로 재시작된 후에는 기존 Master가 Replica를 대신해서 Read권한만 갖는다
- 이때, JVM의 DNS Caching TTL 설정이 중요하다. 대부분 기본 값을 60초 이하로 설정해 놓기 때문에 문제 없지만 혹시 더 길다면 여전히 내려간 Master DB를 향해 요청을 보내고 있을 수 있으므로 확인해보자

## III. 기존 마스터가 다시 Read Write 처리하고 Replica가 Read처리 하도록 돌려놓기

- 이 부분을 다루는 글이 없어서 방법을 찾아보니 의외로 간단했다
- 현재 Writer Role을 가지고 있는 replica를 클릭한 후 Actions의 Failover를 클릭해주면 Manual Failover처리가 되어서 기존 Master instance가 다시 Master 자리로 돌아온다

IV. 참고자료

- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html
