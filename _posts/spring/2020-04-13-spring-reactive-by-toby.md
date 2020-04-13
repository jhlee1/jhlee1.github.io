---
layout: post
title: "토비의 봄 8회 스프링 리액티브 프로그래밍 (4) 자바와 스프링의 비동기 기술 강의 노트"
date: 2020-04-13 23:00:00 +0900
tags: ["spring", "java", "rxjava", "reactive"]
categories: [spring]
---

## 1. Intro
- RxJava가 스프링에서 어떻게 구현되었는지 알아보자


EX1)
```
package me.twoweeks.thejavatestexample;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class FutureEx {

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

EX2)

```
package me.twoweeks.thejavatestexample;

import java.util.Objects;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class FutureEx2 {
  // 예제 1 방식에선 FutureTask를 계속 만들어 줘야하는 불편함 ... Callback으로 처리하기
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
        result -> log.info(result),
        e -> log.info("Error: " + e.getMessage())
    );

    executorService.execute(callbackFutureTask);
    executorService.shutdown();

  }
}




```