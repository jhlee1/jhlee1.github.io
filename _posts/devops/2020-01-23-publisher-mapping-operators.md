---
layout: post
title: "통지 데이터(Publisher) 변환 연산자 정리"
date: 2020-01-23 18:50:00 +0900
tags: ["java", "rx_java", "stream", "reactive"]
categories: [java]
---

## I. map

- 원본 데이터를 받아 변환 작업을 한 뒤 변환된 데이터를 통지
- 한개의 데이터로 여러 데이터를 생성해서 통지 불가
- 건너뛰거나 null을 리턴 불가능
  - null을 리턴해야 할 경우 map 대신 flatMap을 사용

```java
map(Function<? super T, ? extends R> mapper)
```

EX)

```java
public static void main(String[] args) {
    Flowable<String> flowable = Flowable.just("A", "B", "C", "D", "E")
      .map(String::toLowerCase);

    flowable.subscribe(new DebugSubcriber<>());
}
```

출력:

```
main: a
main: b
main: c
main: d
main: e
main: 완료
```

## II. flatMap

- 데이터들을 비동기로 처리 (<-> concatMap)
- map과 달리 Flowable / Observable을 리턴
  - 데이터 한개로 여러 데이터 통지 가능
  - 빈 Flowable / Observable을 이용해서 통지 하지 않고 건너뛸 수 있다
  - 에러 통지도 가능
- 주요 메소드

```java
flatMap(Function<? super T, ? extends Publisher/ObservableSource<? extends R>> mapper)
flatMap(Function<? super T, ? extends Publisher/ObservableSource<? extends U>> mapper, BiFunction<? super T, ? super U, ? extends R> combiner)
flatMap(Function<? super T, ? extends Publisher/ObservableSource<? extends R>> onNextMapper,
	Function<? super Throwable, ? extends Publisher/ObservableSource<? extends R>> onErrorMapper,
	Callable<? extends Publisher<? extends R>> onCompleteSupplier)

```

### 1. flatMap(mapper)

EX)

```java
public static void flatMapExample() {
    Flowable<String> flowable = Flowable.just("A", "B", "C")
        .flatMap(data -> {
          if ("".equals(data)) {
            return Flowable.empty();
          } else {
            return Flowable.just(data.toLowerCase());
          }
        });

    flowable.subscribe(new DebugSubcriber<>());
}
```

출력:

```
main: a
main: b
main: c
main: 완료
```

### 2. flatMap(mapper, combiner)

- combiner: 원본 데이터와 mapper에서 생성한 Flowable/Observable의 데이터를 받아서 새로운 데이터를 생성

EX)

```java
public static void mapperCombinerExample() throws InterruptedException {
	Flowable<String> flowable = Flowable.range(1, 3).flatMap(
    data -> {
			return Flowable.interval(100L, TimeUnit.MILLISECONDS).take(3);
  	},
		(sourceData, newData) -> String.format("sourceData: %s | newData: %s", sourceData, newData)
  );
  
	flowable.subscribe(new DebugSubcriber<>());
	
  Thread.sleep(1000L);
}
```

출력:

```
RxComputationThreadPool-2: sourceData: 2 | newData: 0
RxComputationThreadPool-1: sourceData: 1 | newData: 0
RxComputationThreadPool-1: sourceData: 3 | newData: 0
RxComputationThreadPool-1: sourceData: 1 | newData: 1
RxComputationThreadPool-1: sourceData: 2 | newData: 1
RxComputationThreadPool-3: sourceData: 3 | newData: 1
RxComputationThreadPool-2: sourceData: 2 | newData: 2
RxComputationThreadPool-2: sourceData: 1 | newData: 2
RxComputationThreadPool-2: sourceData: 3 | newData: 2
RxComputationThreadPool-2: 완료
```

### 3. flatMap(onNextMapper, onErrorMapper, onCompleteSupplier)

- onNextMapper는 기존 mapper 역할
- onErrorMapper는 에러가 통지되면 동작
- onCompleteSupplier는 완료 통지 시점에 동작

EX)

```java
public static void mapperErrorAndCompleteExample() {
    Flowable<Integer> flowable = Flowable.just(1, 2, 0, 4, 5)
        .map(data -> 10 / data)
        .flatMap(
          Flowable::just,
          error ->Flowable.just(-1),
          () -> Flowable.just(100)
        );

    flowable.subscribe(new DebugSubcriber<>());
  }
```

출력:

```
main: 10
main: 5
main: -1
main: 완료
```

## III. concatMap / concatMapDelayError

- flatMap과 비슷하지만 데이터를 받은 순서대로 하나씩 처리 (동기처리)
- concatMapDelayError를 통해 에러 발생 후 에러 통지를 완료 전까지 미룰 수 있음
  - tillTheEnd 값을 true로 주면됨
  - false로 두는 경우 바로 통지됨

```java
concatMap(Function<? super T, ? extends Publisher/ObservableSource<? extends R>> mapper)

concatMapDelayError(Function<? super T, ? extends Publisher/ObservableSource<? extends R>> mapper)
```

EX) ConcatMap

```java
private static void concatMapExample() throws InterruptedException {
    Flowable<String> flowable = Flowable.range(10, 3)
        .concatMap(sourceData ->
            Flowable.interval(500L, TimeUnit.MILLISECONDS)
                .take(2)
                .map(data -> {
                  long time = System.currentTimeMillis();
                  return String.format("%s ms | sourceData: %s | data: %s", time, sourceData, data);
                }));

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(4000L);
  }
```

출력:

```
RxComputationThreadPool-1: 1579622136663 ms | sourceData: 10 | data: 0
RxComputationThreadPool-1: 1579622137166 ms | sourceData: 10 | data: 1
RxComputationThreadPool-2: 1579622137668 ms | sourceData: 11 | data: 0
RxComputationThreadPool-2: 1579622138170 ms | sourceData: 11 | data: 1
RxComputationThreadPool-3: 1579622138676 ms | sourceData: 12 | data: 0
RxComputationThreadPool-3: 1579622139171 ms | sourceData: 12 | data: 1
RxComputationThreadPool-3: 완료
```

EX) ConcatMapDelay

```java
 private static void concatMapDelayErrorExample() throws InterruptedException {
    Flowable<String> flowable = Flowable.range(10, 3)
        .concatMapDelayError(sourceData ->
            Flowable.interval(500L, TimeUnit.MILLISECONDS)
                .take(3)
                .doOnNext(data -> {
                      if (sourceData == 11 && data == 1) {
                        throw new Exception("Exception!");
                      }
                    }
                ).map(data -> String.format("sourceData: %s | data: %s", sourceData, data)), 1, true);

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(6000L);
  }
```

출력: boolean tillTheEnd 값이 true 인 경우

```
RxComputationThreadPool-1: sourceData: 10 | data: 0
RxComputationThreadPool-1: sourceData: 10 | data: 1
RxComputationThreadPool-1: sourceData: 10 | data: 2
RxComputationThreadPool-2: sourceData: 11 | data: 0
RxComputationThreadPool-3: sourceData: 12 | data: 0
RxComputationThreadPool-3: sourceData: 12 | data: 1
RxComputationThreadPool-3: sourceData: 12 | data: 2
RxComputationThreadPool-3: java.lang.Exception: Exception!
```

출력: boolean tillTheEnd 값이 false 인 경우

```
RxComputationThreadPool-1: sourceData: 10 | data: 0
RxComputationThreadPool-1: sourceData: 10 | data: 1
RxComputationThreadPool-1: sourceData: 10 | data: 2
RxComputationThreadPool-2: sourceData: 11 | data: 0
RxComputationThreadPool-2: java.lang.Exception: Exception!
```

## IV. concatMapEager / concatMapEagerDelayError

- concatMap과 거의 같은 인터페이스
  - 내부적으로 구현 방식이 다름
  - 각 데이터를 동시에 처리 후(비동기처리) buffer에 담아뒀다가 받은 순서대로 넘겨줌

```java
concatMapEager(Function<? super T, ? extends Publisher/ObservableSource<? extends R>> mapper)

concatMapEagerDelayError(Function<? super T, ? extends Publisher/ObservableSource<? extends R>> mapper, boolean tillThEnd)
```

EX) concatMapEager

```java
  private static void concatMapEagerExample() throws InterruptedException {
    Flowable<String> flowable = Flowable.range(10, 3)
        .concatMapEager(sourceData ->
            Flowable.interval(500L, TimeUnit.MILLISECONDS)
                .take(2)
                .map(data -> {
                  long time = System.currentTimeMillis();
                  return String.format("%s ms | sourceData: %s | data: %s", time, sourceData, data);
                }));

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(4000L);
  }
```

출력:

```
RxComputationThreadPool-1: 1579623890897 ms | sourceData: 10 | data: 0
RxComputationThreadPool-3: 1579623891398 ms | sourceData: 10 | data: 1
RxComputationThreadPool-2: 1579623890897 ms | sourceData: 11 | data: 0
RxComputationThreadPool-2: 1579623891398 ms | sourceData: 11 | data: 1
RxComputationThreadPool-2: 1579623890897 ms | sourceData: 12 | data: 0
RxComputationThreadPool-2: 1579623891398 ms | sourceData: 12 | data: 1
RxComputationThreadPool-2: 완료
```

EX) DelayError

```java
private static void concatMapEagerDelayErrorExample() throws InterruptedException {
    Flowable<String> flowable = Flowable.range(10, 3)
        .concatMapEagerDelayError(sourceData ->
            Flowable.interval(500L, TimeUnit.MILLISECONDS)
                .take(3)
                .doOnNext(data -> {
                      if (sourceData == 11 && data == 1) {
                        throw new Exception("Exception!");
                      }
                    }
                ).map(data -> String.format("sourceData: %s | data: %s", sourceData, data)), true);

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(5000L);
  }
```

출력: boolean tillTheEnd 값이 true인 경우

```
RxComputationThreadPool-1: sourceData: 10 | data: 0
RxComputationThreadPool-1: sourceData: 10 | data: 1
RxComputationThreadPool-1: sourceData: 10 | data: 2
RxComputationThreadPool-1: sourceData: 11 | data: 0
RxComputationThreadPool-1: sourceData: 12 | data: 0
RxComputationThreadPool-1: sourceData: 12 | data: 1
RxComputationThreadPool-3: sourceData: 12 | data: 2
RxComputationThreadPool-3: java.lang.Exception: Exception!
```

출력: boolean tillTheEnd 값이 false인 경우

```
RxComputationThreadPool-2: sourceData: 10 | data: 0
RxComputationThreadPool-1: sourceData: 10 | data: 1
RxComputationThreadPool-3: sourceData: 10 | data: 2
RxComputationThreadPool-3: java.lang.Exception: Exception!
```

## V. buffer

- 바로 통지하지 않고 모아서 List나 Collection에 담아서 통지하는 연산자
- 한번에 모을 개수, 시간 간격을 조정할 수 있음

```java
buffer(int count) //count - 버퍼에 담을 데이터 개수
buffer(long time, TimeUnit unit) //time 시간 간격, unit - 시간 간격 단위
buffer(Publisher/ObservableSource<B> boundaryIndicator)
buffer(Callable<? extends Publisher/ObservableSource<B>> boundaryIndicatorSupplier)
buffer(Flowable/Observable<? extends TOpening openingIndicator, Function<? super TOpening, ? extends Publisher/ObservableSource<? extends TClosing>> closingIndicator)

```

EX) buffer(count)

```java
private static void bufferWithCountExample() throws InterruptedException {
    Flowable<List<Long>> flowable = Flowable.interval(100L, TimeUnit.MILLISECONDS)
        .take(10)
        .buffer(3);

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(1000L);
}
```

출력:

```java
RxComputationThreadPool-1: [0, 1, 2]
RxComputationThreadPool-1: [3, 4, 5]
RxComputationThreadPool-1: [6, 7, 8]
RxComputationThreadPool-1: [9]
RxComputationThreadPool-1: 완료
```



EX) buffer(time, unit)

```
  private static void bufferWithTimeExample() throws InterruptedException {
    Flowable<List<Long>> flowable = Flowable.interval(100L, TimeUnit.MILLISECONDS)
        .take(10)
        .buffer(300L, TimeUnit.MILLISECONDS);

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(1200L);
  }
```

출력: 비동기 작업이기 때문에 Thread별 시간에 오차가 생겨서 돌릴 때 마다 미세한 차이로 묶이는게 다르게 나옴

```
RxComputationThreadPool-1: [0, 1, 2]
RxComputationThreadPool-1: [3, 4, 5]
RxComputationThreadPool-1: [6, 7, 8]
RxComputationThreadPool-2: [9]
RxComputationThreadPool-2: 완료

RxComputationThreadPool-1: [0, 1, 2]
RxComputationThreadPool-1: [3, 4]
RxComputationThreadPool-1: [5, 6, 7]
RxComputationThreadPool-2: [8, 9]
RxComputationThreadPool-2: 완료

...
```

- buffer(boundaryIndicator)
  - Flowable/Observable이 데이터를 통지하는 간격 단위로 buffer에 데이터를 쌓는다
- buffer(boundaryIndicatorSupplier)
  - 버퍼링을 시작 할 때 Flowable/Observable을 생성하고, 생성된 Flowable/Observable이 데이터를 통지하는 시점에 버퍼링을 끝냄
- **boundaryIndicator는 interval을 사용하지만 boundaryIndicatorSupplier는 timer를 사용해야 같은 결과가 나온다**
  - 객체를 넘기기 / 함수로 구현의 차이

EX) buffer(Flowable boundaryIndicator)

```java
  private static void bufferWithBoundaryIndicator() throws InterruptedException {
    Flowable<List<Long>> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
        .take(7)
        .buffer(Flowable.interval(1000L, TimeUnit.MILLISECONDS));

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(4000L);
  }
```

출력:

```java
RxComputationThreadPool-1: [0, 1, 2]
RxComputationThreadPool-1: [3, 4, 5]
RxComputationThreadPool-2: [6]
RxComputationThreadPool-2: 완료
```

EX) buffer(Callable boundaryIndicatorSupplier)

```java
private static void bufferWithBoundaryIndicatorSupplier() throws InterruptedException {
  Flowable<List<Long>> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
      .take(7)
      .buffer(() -> Flowable.timer(1000L, TimeUnit.MILLISECONDS));

  flowable.subscribe(new DebugSubcriber<>());

  Thread.sleep(6000L);
}
```

EX)

```java
RxComputationThreadPool-3: [0, 1, 2]
RxComputationThreadPool-5: [3, 4, 5]
RxComputationThreadPool-4: [6]
RxComputationThreadPool-4: 완료
```

- buffer(openingIndicator, closingIndicator)
  - openingIndicator(Flowable/Observable)는 여러 데이터를 통지 
    - 버퍼링 시작을 통지
  - closingIndicator(Function)는 하나만 데이터를 통지
    - 버퍼링 종료를 통지
  - closingIndicator는 openingIndicator의 Flowable / Observable이 데이터를 통지할 때 호출되 처리 시작

EX)

```java
  private static void bufferWithOpeningClosingIndicator() throws InterruptedException {
    Flowable<List<Long>> flowable = Flowable.interval(300L, TimeUnit.MILLISECONDS)
        .take(7)
        .buffer(Flowable.interval(0L, 900L, TimeUnit.MILLISECONDS), opening -> Flowable.interval(1000L, TimeUnit.MILLISECONDS));

    flowable.subscribe(new DebugSubcriber<>());

    Thread.sleep(6000L);
  }
```

출력:

```java
RxComputationThreadPool-3: [0, 1, 2]
RxComputationThreadPool-4: [3, 4, 5]
RxComputationThreadPool-2: [6]
RxComputationThreadPool-2: 완료
```

## V. toList

- 완료 통지를 받은 시점에 모아뒀던 데이터를 List형태로 통지
- Buffer에 쌓이는 데이터가 너무 많은 경우 메모리가 부족하게 될 수 있으므로 주의
- List 하나만 보내기 때문에 Single을 return

```java
toList()
toList(Collable<U> collectionSupplier) // 데이터를 담는 객체를 생성하는 함수형 인터페이스
```

EX)

```java
private static void toListExample() throws InterruptedException {
  Single<List<String>> single = Flowable.just("A", "B", "C", "D", "E")
      .toList();

  single.subscribe(new DebugSingleObserver());
}
```

출력:

```java
main: [A, B, C, D, E]
```

## VI. toMap

- 받은 데이터로 key를 생성하고 이 key로 Map에 담는다
- buffer에 넣어둘 데이터 양이 커질 경우 메모리 부족이 발생할 수 있음
- List와 마찬가지로 Single을 return
- 같은 key 값에 데이터를 두 번 이상 넣을 경우 앞에 쓴 데이터가 덮어씌워짐
- Parameters
  - keySelector - 사용할 key를 생성 (Function)
  - valueSelector - 사용할value를 생성 (Funciton)
  - mapSupplier - key와 value를 담을 Map 객체를 생성 (Callable)

EX) toMap(keySelector)

```java
private static void toMapExample() {
  Single<Map<Long, String>> single = Flowable.just("1A", "2B", "3C", "1D", "2E")
      .toMap(data -> Long.valueOf(data.substring(0, 1)));

  single.subscribe(new DebugSingleObserver());
}
```

출력:

```java
main: {1=1D, 2=2E, 3=3C}
```

EX) toMap(keySelector, valueSelector)

```java
private static void toMapWithKeySelectorValueSelectorExample() {
  Single<Map<Long, String>> single = Flowable.just("1A", "2B", "3C", "1D", "2E")
      .toMap(data -> Long.valueOf(data.substring(0, 1)), data -> data.substring(1));

  single.subscribe(new DebugSingleObserver());
}
```

출력:

```
main: {1=D, 2=E, 3=C}
```

## VII. toMultiMap

- map인데 value가 collection 타입으로 같은 key값을 가진 value가 여러개
- Parameter
  - keySelector - 사용할 key를 생성 (Function)
  - valueSelector - 사용할value를 생성 (Funciton)
  - mapSupplier - 통지할 Map 객체를 생성 (Callable)
  - collectionFactory - key를 바탕으로 Map에 값으로 담을 collection 객체를 생성 (Function)

EX)

```java
private static void toMultiMapWithKeySelectorExample() throws InterruptedException {
  Single<Map<String, Collection<Long>>> single = Flowable.interval(500L, TimeUnit.MILLISECONDS)
  .take(5)
  .toMultimap(data -> {
    if (data % 2 == 0) {
      return "Even";
    } else {
      return "Odd";
    }
  });

  single.subscribe(new DebugSingleObserver());

  Thread.sleep(3000L);
}
```

출력:

```java
RxComputationThreadPool-1: {Even=[0, 2, 4], Odd=[1, 3]}
```

