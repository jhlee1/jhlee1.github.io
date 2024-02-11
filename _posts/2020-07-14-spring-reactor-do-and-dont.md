---
layout: post
title: "Do's and Don'ts: Avoiding First-Time Reactive Programmer Mines 강의 노트"
date: 2020-04-13 23:00:00 +0900
tags: ["spring", "java", "reactor", "webflux", "reactive"]
categories: [spring]
---


## 최근 Webflux로 개발하면서 가장 크게 와닿았던 강의였다. 특히 Blocking과 Resiliency 부분이 가장 좋았던 것 같다

## I. [Do's] Learn functional programming

- Functional Progrmaming을 배우기 위해 Haskell을 배울 필요는 없지만 기본적인 함수형 프로그래밍의 지식을 쌓아라

## II. [Do's] Check various operators

- starter-pack 수준의 operator사용에서 벗어나기

## III. [Do's] Think about resiliency

- Timeout, Errors, Backpressure

## IV. [Do's] Start gradually

- Layer를 나눠서 Repository부터 조금씩 바꿔가기 (Repository -> Service -> Controller)

## V. [Do's] Prepare for day 2

- 처음엔 신나서 막 만들지만 만들다보면 수 많은 문제점이 발생한다
- 기존 MVC 모델의 경우 Request에 대한 처리를 하나의 Thread에서 모두 처리하므로 IDE에서 파란색 글로 찾아주지만 Webflux에선 여러 Thread에서 처리되므로 찾기 어렵다.
- Error Tracking 방법
  - checkpoint = 에러가 발생할 것 같은 지점 바로 뒤에 checkpoint를 붙여서 에러 발생 지점을 찾을 수 있음

```java
Flux.just("Hello", "world!")
    .single()
    .checkpoint("afterSingle")
    .subscribeOn(Schedulers.parallel())
```

- `Hooks.onOperatorDebug()` 사용하기
  - 계속 checkpoint를 붙여줘야하는 번거로움은 없지만 performance 문제가 있음

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        Hooks.onOperatorDebug(); // 이 부분에 넣어주면 끝
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

- ReactorDebugAgent 사용하기
  - dependency 추가 필요
  - newrelic같은 apm에서 영감을 받아서 만들었다고 함

```groovy
    implementation 'io.projectreactor:reator-tools:3.3.0.RELEASE'
```
  
```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        ReactorDebugAgent.init(); // 이 부분에 넣어주기
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

## VI. [Don'ts] Use it for heavy computations

- for loop 보다 5배 이상 overhead가 발생
  - stream과 마찬가지로 Queue 방식으로 처리하기 때문에
  - stream보다 느린 이유? scheduler, backpressure 등등 다양한 기능을 제공하다보니 더 느려짐
  - GraalVM을 이용할 경우 10% 이상 속도 향상이 있다고함

## VII. [Don'ts] Care about the threads

- Sync task(동기 작업)가 발생한 경우를 제외하고 직접 thread를 관리할 필요가 없음

## VIII. [Don'ts] Block non-blocking threads

- 내가 모르는 사이에 어디선가 block이 발생할 수 있다
  - 내가 사용하는 library들에서 자주 발생하는 경우
  - EX) `KafkaProducer#send(ProeucerRecord, Callback)` => 콜백이 존재하기 때문에 비동기인 것 같지만 사실 내부적으로 wait이 존재함. [이슈 티켓](https://issues.apache.org/jira/browse/KAFKA-3539)
- Block을 최대한 하지마라. 해야하는 경우 custom thread를 생성해라
- custom thread를 사용하기 위해 scheduler로 parallel이나 dynamicElastic을 사용해라

## IX. [Don'ts] Use ThreadLocals

- 안쓰는 것을 권장함
- 어쩔 수 없이 써야할 경우 reactive context를 사용

## X. [Don'ts] Be afraid of it :)

- Enjoy it!

### 원본 강의: [Do’s and Don’ts: Avoiding First-Time Reactive Programmer Mines](https://www.youtube.com/watch?v=0rnMIueRKNU)