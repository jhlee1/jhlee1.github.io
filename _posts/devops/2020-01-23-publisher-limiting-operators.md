---
layout: post
title: "통지 데이터(Publisher)를 제한하는 연산자 정리"
date: 2020-01-23 10:50:00 +0900
tags: ["java", "rx_java", "stream", "reactive"]
categories: [java]
---

## I. filter

- Predicate를 이용해서 true인 것만 통지

```java
filter(Predicate predicate)
```

EX) 짝수만 통지

```java
private static void filterExample() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .filter(data -> data % 2 == 0);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력:

```java
RxComputationThreadPool-1: 0
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 4
RxComputationThreadPool-1: 6
RxComputationThreadPool-1: 8
```

## II. distinct

- 이미 통지한 데이터와 같은 데이터를 제외하고 통지
- Parameter
  - distinct()
  - distinct(Function<? Super T,K> keySelector)
    - keySelector - 받은 데이터와 비교할 데이터를 생성

EX) distinct()

```java
private static void distinctExample() throws InterruptedException {
  Flowable<String> flowable = Flowable.just("A", "a", "B", "b", "A", "a", "B", "b")
      .distinct();

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력:

```java
main: A
main: a
main: B
main: b
main: 완료
```

EX) distinct(keySelector)

```java
private static void distinctWithKeySelectorExample() throws InterruptedException {
  Flowable<String> flowable = Flowable.just("A", "a", "B", "b", "A", "a", "B", "b")
      .distinct(String::toLowerCase);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력:

```
main: A
main: B
main: 완료
```

## III. distinctUntilChanged

- 같은 데이터를 연속으로 받을 때 이 데이터를 제외
- 통지한 데이터와 같아도 **연속되지 않으면** 제외하지 않음
- Parameters
  - keySelector - 받은 데이터와 비교할 데이터를 생성
  - comparer - 바로 앞 데이터와 현재 데이터가 같은지 판단하는 함수 생성

EX) distinctUntilChanged()

```java
private static void distinctUntilChangedExample() {
  Flowable<String> flowable = Flowable.just("A", "B", "A", "A", "B", "B")
      .distinctUntilChanged();

  flowable.subscribe(new DebugSubcriber<>());
}
```

출력:

```
main: A
main: B
main: A
main: B
main: 완료
```

EX) distinctUntilChanged(Function<? super T, K> keySelector)

```java
private static void distinctUntilChangedWithKeySelectorExample() {
  Flowable<String> flowable = Flowable.just("A", "a", "B", "b", "A", "a", "B", "b")
      .distinctUntilChanged(data -> data.toLowerCase());

  flowable.subscribe(new DebugSubcriber<>());
}
```

출력:

```
main: A
main: B
main: A
main: B
main: 완료
```

EX) distinctUntilChanged(BiPredicate<? super T, ? super T> comparer)

```java
private static void distinctUntilChangedWithComparerExample() {
  Flowable<String> flowable = Flowable.just("A", "a", "B", "b", "A", "a", "B", "b")
      .distinctUntilChanged((data1, data2) -> data1.equalsIgnoreCase(data2));

  flowable.subscribe(new DebugSubcriber<>());
}
```

출력:

```
main: A
main: B
main: A
main: B
main: 완료
```

## IV. take

- 지정한 개수, 기간까지만 데이터 통지

EX) take()

```java
private static void takeExample() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(1000L, TimeUnit.MILLISECONDS)
      .take(3);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(4000L);
}
```

출력:

```java
RxComputationThreadPool-1: 0
RxComputationThreadPool-1: 1
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 완료
```

## V. takeUntil

- 특정 조건이 될 때까지 데이터를 통지
  - Parameters에 따라 통지방식이 나뉨
    - stopPredicate  - 해당 조건이 true가 될 때까지 통지 (Predicate)
      - true가 될 때 받은 데이터도 완료와 같이 통지
    - other - 해당 Flowable이 데이터 통지를 시작할 때까지 통지 (Publisher/ObservableSource)

EX) takeUntil(stopPredicate)

```
private static void takeUntilWithStopPredicateExample() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .takeUntil(data -> data == 3);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(4000L);
}
```

출력:

```
RxComputationThreadPool-1: 0
RxComputationThreadPool-1: 1
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 3
RxComputationThreadPool-1: 완료
```

EX) takeUntil(other)

```java
private static void takeUntilWithOtherExample() throws InterruptedException {
    Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
        .takeUntil(Flowable.timer(1000L, TimeUnit.MILLISECONDS));

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(4000L);
}
```

출력:

```
RxComputationThreadPool-2: 0
RxComputationThreadPool-2: 1
RxComputationThreadPool-2: 2
RxComputationThreadPool-1: 완료
```

## VI. takeWhile

- Predicate로 받는 조건이 true면 계속 통지

EX) takeWhile(predicate)

```java
  private static void takeWhileWithPredicateExample() throws InterruptedException {
    Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
        .takeWhile(data -> data != 3);

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(2000L);
  }
```

출력:

```
RxComputationThreadPool-1: 0
RxComputationThreadPool-1: 1
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 완료
```

## VII. takeLast

- 완료시점에서 마지막 데이터 부터 지정한 개수나 기간의 데이터를 통지
- 완료 시점을 통지하지 않는 Flowable/ObservableSource에선 사용 불가
- 조건에 맞는 데이터를 확인하는 시점은 데이터 통지시점이 아니라 완료 시점
- Parameters
  - count - 끝에서부터 세는 데이터 개수 (int)
  - time - 통지할 데이터를 결정할 대상 기간 (long)
  - unit - 위에 기간의 단위 (TimeUnit)

EX) takeLast(count)

```java
private static void takeLastWithCountExample() throws InterruptedException {
	Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
		.take(5)
		.takeLast(2);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(2000L);
}
```

출력:

```
RxComputationThreadPool-1: 3
RxComputationThreadPool-1: 4
RxComputationThreadPool-1: 완료
```

- takeLast(count, time, unit)
  - 마지막 통지 시점부터 지정한 기간(time)까지의 데이터를 얻고, 그 중에 끝에서 count만큼 데이터를 통지

EX) takeLast(count, time, unit)

```java
private static void takeLastWithTimeAndUnitExample() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(10)
      .takeLast(3, 300L, TimeUnit.MILLISECONDS);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(4000L);
}
```

출력:

```
RxComputationThreadPool-1: 8
RxComputationThreadPool-1: 9
RxComputationThreadPool-1: 완료
```

## VIII. skip

- 앞에서부터 지정한 만큼 건너뛰기
- 데이터 개수나 경과 시간으로 지정 가능
- Parameters
  - count - 건너뛸 개수
  - time - 건너뛸 시간
  - unit - 건너뛸 시간 단위

EX) skip()

```java
private static void skip() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .skip(2);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(1000L);
}
```

출력:

```
RxComputationThreadPool-1: 2
```

## IX. skipUntil

- Parameter로 받는 Flowable/Observable이 데이터를 통지할때까지 건너뛴다

EX)  skipUntil(Publisher/ObservableSource other)

```java
private static void skipUntilWithOther() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .skipUntil(Flowable.timer(1000L, TimeUnit.MILLISECONDS));

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(2000L);
}
```

출력:

```
RxComputationThreadPool-2: 3
RxComputationThreadPool-2: 4
RxComputationThreadPool-2: 5
```

## X. SkipWhile

- 조건이 true가 될때까지 건너뛰기

EX) skipWhile(predicate)

```java
private static void skipWhileWithPredicate() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .skipWhile(data -> data != 3);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(2000L);
}
```

출력:

```
RxComputationThreadPool-1: 3
RxComputationThreadPool-1: 4
RxComputationThreadPool-1: 5
```



## XI. skipLast

- 끝에서부터 지정한 범위의 시간, 개수만큼 데이터 건너뛰어서 통지
- 건너뛰는만큼 늦게 결과 통지 시작

EX) skipLast(count)

```java
private static void skipLastWithCount() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .skipLast(2);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(2000L);
}
```

출력:

```
RxComputationThreadPool-1: 0
RxComputationThreadPool-1: 1
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 3
```



## XII. throttleFirst

- 데이터 통지 후 지정 시간 동안 데이터를 통지하지 않음
- 처리가 완료될 때까지 반복됨
- 단기간에 들어오는 데이터가 모두 필요하지 않은 경우 사용해서 쳐낼 수 있다
- Parameters
  - Time - 데이터를 파기하는 시간
  - Unit - 시간 단위

EX)

```java
private static void throttleFirstWithTimeAndUnit() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(10)
      .throttleFirst(1000L, TimeUnit.MILLISECONDS);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력

```
RxComputationThreadPool-1: 0
RxComputationThreadPool-1: 4
RxComputationThreadPool-1: 8
RxComputationThreadPool-1: 완료
```

## XIII. throttleLast

- 지정한 시간마다 가장 마지막에 통지되는 데이터만 통지
- throttleLast와 sample은 이름만 다를뿐 같은 Parameter를 넣으면 똑같은 작업
- Parameters
  - Time - 간격으로 지정한 시간 (long)
  - Unit - 시간 단위 (TimeUnit)

EX) throttleLast(long time, TimeUnit unit)

```java
private static void throttleLastWithTimeAndUnit() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(9)
      .throttleLast(1000L, TimeUnit.MILLISECONDS); // 1000 Milliseconds 동안 받은 데이터중 가장 마지막꺼만 통지

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력

```
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 5
RxComputationThreadPool-2: 완료
```

- throttleLast와 sample은 이름만 다를뿐 같은 Parameter를 넣으면 똑같은 작업
  - `sample(time, timeunit)`
- Parameter
  - Time - 간격으로 지정한 시간 (long)
  - Unit - 시간 단위 (TimeUnit)
  - sampler - 데이터 통지 간격을 정하는 Flowable / Observable (Publisher / ObservableSource)

EX) sample(sampler)

```java
private static void sampleWithSampler() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(9)
      .sample(Flowable.interval(1000L, TimeUnit.MILLISECONDS));

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력

```
RxComputationThreadPool-1: 2
RxComputationThreadPool-1: 5
RxComputationThreadPool-2: 완료
```

## XIV. throttleWithTimeout

- 데이터를 받은 후 일정 기간동안 다음 데이터를 받지 못하면 현재 데이터를 통지
- Parameters
  - time - 지정하려는 시간
  - unit - 시간 단위
  - debounceIndicator - 건너뛸 시간을 정하는 Flowable/Observable

```java
private static void throttleWithTimeout() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(9)
      .throttleWithTimeout(500L, TimeUnit.MILLISECONDS);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력

```
RxComputationThreadPool-1: 8
RxComputationThreadPool-1: 완료
```

- Debounce는 throttleWithTimeout과 parameter가 같으면 동일한 처리

EX) debounce(time, unit)

```java
private static void debounce() throws InterruptedException {
  Flowable<Long> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(9)
      .debounce(500L, TimeUnit.MILLISECONDS);

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(3000L);
}
```

출력

```java
RxComputationThreadPool-1: 8
RxComputationThreadPool-1: 완료
```

## XV. elementAt / elementAtOrError

- 지정한 위치의 데이터만을 통지하는 연산자
- Single / Maybe를 return
- methods
  - `Maybe<T> elementAt(long index)` - 데이터가 없으면 완료를 통지하는 Maybe
  - `Single<T> elementAt(long index, T defaultItem)` - 데이터가 없으면 defaultItem을 통지
  - `Single<T> elementAtOrError(long index)` - 데이터가 없으면 에러 통지

EX) elementAt(long)

```java
private static void elementAtExample() throws InterruptedException {
  Maybe<Long> maybe = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .elementAt(3);

  maybe.subscribe(new DebugMaybeObserver());

  Thread.sleep(3000L);
}
```

출력

```
RxComputationThreadPool-1: 3
```
