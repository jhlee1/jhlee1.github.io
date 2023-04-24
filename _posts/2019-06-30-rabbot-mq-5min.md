---
title: "Rabbit MQ 5분내로 이해하기"
author: joohan
date: 2019-02-24T10:39:07+09:00
archives: "2019"
tags: ["K-MOOC", "rabbit_mq", "msa"]
categories: [msa]
---

## I. 핵심 개념
1. Producer - Exchange에게 Message를 보냄
2. Consumer - Queue로부터 Message를 받음
3. Exchange는 Routing Key와 Binding Key를 비교해 어떤 Queue와 연결할지 결정
4. Message는 exchange type에 따라 분배됨

## II. Exchange Type
1. Direct: Producer는 Message를 Routing Key와 함께 Exchange로 보내고 Exchange는 Routing Key를 Binding Key가 일치하는 Queue에게 Message를 넣어줌
2. Fanout - Routing Key와 상관없이 모든 Queue에게 Message를 보냄
3. Topic - Regex를 이용해 Queue에게 Message 전달
ex) message with “red.*” -> Binding Key “red.green”
4. Header - Routing Key 대신 Header를 사용
5. Default (nameless) - Routing Key와 Queue의 이름이 같으면 보냄
ex) routing key가 “B”이면 “A” Queue, “B” Queue, “C” Queue 중에 “B” Queue에게 보냄

## III. Channel 개념
- lightweight connections that share a single TCP connection
- Connection안에 존재
- Connection이 사라질 경우 해당 Connection 안에 존재하는 Channel들은 모두 사라진다.
- Connection안에 여러개의 Channel을 구분하기 위해 Channel ID (Channel number를 사용)
- 독립된 Connection을 제공하기 위해 존재한다.
- 어플리케이션이 여러 개의 연결을 요구할 경우 매번 Connection을 생성하기에는 자원의 낭비가 크기 때문에 Channel을 이용한다. 

## IV. Round-robin dispatching
- Queue에선 Consumer가 여러명일 경우 Round-robin 방식으로 Consumer에게 Message를 전달
- Round-robin 방식이란 Message를 순차적으로 하나씩 나눠주는 것
ex) 
```
Task 1 ~ 4 & Worker 1 ~ 2 인 경우, 
Task 1, 3 -> Worker1 
Task 2, 4 -> Worker 2
```
문제점: Task1과 Task3이 오래걸리는 작업인 경우 Worker1은 계속 작업중이고 Worker2는 계속 놀게 된다.

## V. Fair dispatch
- Queue는 Consumer에게 unacknowledged message(처리 완료 안된 메세지)가 몇개인지 확인하지 않고 있음
- 처리가 오래 걸리는 Message가 한 Consumer에게만 보내질 경우 해당 Consumer에게만 계속 쌓이게 되는 문제가 생김
- 해결 방법: basicQos를 사용해서 한번에 1개 이상의 Message를 Consumer가 들고있지 않도록 해서 해당 Consumer가 Message를 처리하지 않은 경우 다음 Consumer에게 넘김
```
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

## VI. Message acknowledgment
- Consumer가 항상 받은 모든 Message를 처리할 수 있진 않음
	- Consumer가 Message를 정상적으로 받지 못한 경우
	- Message 처리 중에 Consumer가 죽는 경우 
- Queue가 확인하기 위해 Consumer가 정상적으로 Message를 받고 처리한 후 Queue에게 acknowledgement를 보냄 
- Consumer가 acknowledgement를 보내지 않고 죽는 경우 unacknowledged Message는 다시 Queue속에 들어감

## VII. Message Durability
- RabbitMQ가 재시작되거나 Crash 발생할 경우 현재 실행중인 프로세스 내의 Message가 사라짐 -> 프로세스가 비정상적으로 종료되는 경우를 대비해 Disk에 저장
```
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```
- But, Message를 받고 Disk에 쓰여지기 전에 약간의 시간이 존재하므로 프로세스가 갑자기 중단될 경우 위의 방법이 무조건 Message의 지속성(Persistence) 을 보장해주진 않음
	- 사실 Java에서 Write 처리를 해도 Disk에 쓰이기 전에 OS단에서는 memory에 cache해놓고 나중에 쓰기 때문에
- [If you need a stronger guarantee, you can use publisher confirms](https://www.rabbitmq.com/confirms.html)

### 참고 링크:
1. https://www.rabbitmq.com/getstarted.html - 공식 가이드
2. https://www.youtube.com/watch?v=deG25y_r6OY - 개념정리
