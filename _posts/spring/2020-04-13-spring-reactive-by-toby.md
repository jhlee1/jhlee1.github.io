---
layout: post
title: "토비의 봄 8회 스프링 리액티브 프로그래밍 (4) 자바와 스프링의 비동기 기술 강의 노트"
date: 2020-04-13 23:00:00 +0900
tags: ["spring", "java", "rxjava", "reactive"]
categories: [spring]
---

## I. Intro
- RxJava가 스프링에서 어떻게 구현되었는지 알아보자


## II. FutureTask 이용하기
- 기존 Java에선 다른 Thread에서 비동기 처리를 위해 ThreadPool을 지원함 => ExecutorService
- Future는 JS에서 Promise처럼 아직 값이 들어있진 않지만 get을 부를 경우 비동기로 실행중인 Thread의 작업이 끝날 때까지 현재 Thread를 멈추고 대기하다가 값이 나오면 이어서 처리한다
- ExecutorService를 이용해서 비동기 작업처리하기
- FutureTask를 정의해서 ExecutorService에서 처리할 내용을 미리 정의해두기
- 문제점: 매 작업마다 Exception이 발생할 경우 try / catch를 걸어줘야하고 get으로 결과값을 받아서 처리하는동안 현재 Thread가 blocking됨

EX1)
```
package lee.twoweeks.tobyreactivespringexample.rxjava.future;


import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class FutureExample {

  /**
   * Created by Joohan Lee on 2020/04/16
   */

  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService es = Executors.newCachedThreadPool();

//    es.execute(() -> { // Runnable
//      try {
//        Thread.sleep(2000);
//      } catch (InterruptedException e) {}
//      log.info("Async");
//    });

    Future<String> f = es.submit(() -> { //Callable - Runnable과 다르게 return 할 수 있고 Exception을 throw한다.
      Thread.sleep(2000);
      log.info("Async");
      return "Hello";
    });

    log.info("" + f.isDone()); // false
    Thread.sleep(2100);
    log.info("" + f.isDone()); // true
    log.info(f.get()); // Async 작업이 끝날 때까지 Blocking 으로 기다림
    log.info("Exit");

    FutureTask<String> futureTask = new FutureTask<String>(() -> {// Future로 받기 전 처리할 작업을 객체로 선언
      Thread.sleep(2000);
      log.info("Async");
      return "Hello";
    });

    es.execute(futureTask); // 위에 es.execute (() -> ...) 으로 처리한 것과 같은 결과임

    FutureTask<String> futureTask2 = new FutureTask<String>(() -> {// Future로 받기 전 처리할 작업을 객체로 선언
      Thread.sleep(2000);
      log.info("Async");
      return "Hello";
    }) {
      @Override
      protected void done() {
        try {
          System.out.println(get());
        } catch (InterruptedException e) {
          e.printStackTrace();
        } catch (ExecutionException e) {
          e.printStackTrace();
        }
      }
    };

    es.execute(futureTask2);
    es.shutdown(); // shutdown하지 않으면 ExecutorService가 계속 떠있어서 종료되지 않음
  }
}

```

## III. Callback으로 처리하기

- 위 방식의 문제점: 
  - Future값을 가져올 때 blocking해서 결과를 받을 때까지 대기하도록 함
  - 내부에서 Exception이 발생할 경우 처리를 위해 try catch문이 필요하다.
- 추가적으로, FutureTask 생성 작업을 좀 더 간결하게 해보자
- Callback으로 처리하기 
  - 위에 Future와 비슷하지만 exception를 callback으로 처리함
- Callback - 위에 Future와 비슷하지만 exception를 callback으로 처리함
  // Ex. AsynchronousByteChannel의 read
  
EX)

```
package lee.twoweeks.tobyreactivespringexample.rxjava.future;

/**
 * Created by Joohan Lee on 2020/04/16
 */

import java.util.Objects;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class FutureCallbackExample {
  interface SuccessCallback {
    void onSuccess(String result);
  }

  interface ExceptionCallback {
    void onError(Throwable t);
  }

  public static class CallbackFutureTask extends FutureTask<String> {
    SuccessCallback successCallback;
    ExceptionCallback exceptionCallback;

    public CallbackFutureTask(Callable<String> callable, SuccessCallback successCallback, ExceptionCallback exceptionCallback) {
      super(callable);
      this.successCallback = Objects.requireNonNull(successCallback); // Null인 경우 NullPointerException을 발생
      this.exceptionCallback = Objects.requireNonNull(exceptionCallback);
    }

    @Override
    protected void done() {
      try {
        successCallback.onSuccess(get());
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      } catch (ExecutionException e) {
        exceptionCallback.onError(e.getCause());
      }
    }
  }
  public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();

    CallbackFutureTask callbackFutureTask = new CallbackFutureTask(() -> {
      Thread.sleep(2000);
      log.info("Async");
      return "Hello";
    },
        result -> log.info("Result:" + result),
        e -> log.info("Error: " + e.getMessage())
    );

    executorService.execute(callbackFutureTask);
    executorService.shutdown();

  }
}
```

- 하지만, 여기까지 코드의 문제점
  - 비지니스 로직과 기술적인 코드가 혼재되어 있음. ExecutorService와 FutureTask를 생성하는 기술적인 코드를 따로 분리할 필요가 있음
  - Spring에서 제공하는 기능을 써보자
