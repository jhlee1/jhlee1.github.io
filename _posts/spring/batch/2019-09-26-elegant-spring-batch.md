---
layout: post
title: "우아한 스프링 배치"
date: 2019-09-26 21:09:29 +0900
tags: ["spring", "batch", "java"]
categories: [spring]
---

## 1.  소개

- 스프링 배치를 한번이라도 써본 사람이 대상
  - 블로그나 인터넷 커뮤니티에 나오지 않는 내용을 주로 다룰 것임
- 기본편: 기본개념 & 오해 풀기
- 활용편: 실제 업무에서의 스프링 배치
- xml기반이 아니라 java config

## 2. 기본편

- 배치 어플리케이션이란?
  
  - 배치 처리는 컴퓨터에서 사람과 상호작용 없이 이뤄지는 작업
- VS 웹 - 사람과 상호작용이 핵심 (Interaction이 계속 발생)
    - Batch Application은 요청 한번만 보내면 알아서 처리됨
  - Web - 실시간 처리 / 상대적인 속도 / QA 용이성
  - Batch - 후속 처리 / 절대적인 속도 / QA 복잡성
    - QA분들이 직접 DB를 열어보거나 Java code를 볼 수 없음. 따라서 Batch의 경우 Test Code가 반드시 필요
- Spring Batch 와 Quartz
    - Quartz는 스케쥴링 프레임워크
    - 대부분 사람들이 Quartz를 오해하고있다. Spring Batch의 보안제 역할이지 **대체제**(대체 불가)는 아니다
        - Batch Application이 필요한 상황
            - 일정주기로 실행해야될 때
                - 그럼 Quartz써도 되는거 아님?
                    - 한달에 한번이라는 뜻은 한달동안 쌓인 데이터 (대량의 데이터)를 처리해야한다는 뜻
                    - Quartz는 대용량 데이터를 처리할 Batch Size, Paging, 등등 다양한 기능을 지원해주지 않음
            - 실시간 처리가 어려운 대량의 데이터를 처리할 때

- Job / Step / Tasklet 등등

  - Job 
    - Step
      - Tasklet
        - ChunkOriente Tasklet (위의 Tasklet을 구현체)
          - Reader
          - Processor
          - Writer

- JobScope / ParameterScope

  - JobParameter - 외부에서 Parameter를 주입받아 Batch Component에서 사용
  - StepParameter - 
  - 문제점
    - Long / String / Double /Date만 지원
    - Enum / LocalDate / LocalDateTime을 지원 안함
      - LocalDate같은 걸 사용하는 경우 매번 형 변환이 문제
  - 해결방법
    - @Value와 Setter를 이용

  ```
  @Value()
  ```

  - @JobScope와 @StepScope는 LateBinding
    - Job이 실행될 때 @Jobscope 생성
    - Step이 실행될 때 @StepScope 생성
    - Late Binding 애플리케이션 실행 후에도 동적으로 reader / processor / writer를 생성
      - JobParameter 값에 따라 Reader와 Processor를 교체
        - 사용 예) 사용하는 field에 따라 읽어오는 Table이 다를 때
          - 주문 - 주문 테이블을 읽어오는 Reader 사용
          - 광고 - 광고 테이블을 읽어오는 Reader 사용
          - 기타 - 기타 테이블을 읽어오는 Reader 사용

## 3. 활용편

- 관리도구들
  - Cron - Linux
  - Spring MVC + API Call - 권장되지 않는 방법
  - Spring Batch Admin (Deprecated)
    - Spring Data Cloud로 이전하라고 권장됨
  - Quartz + Admin
  - CI Tools (Jenkins / TeamCity 등등)
- Jenkins 사용을 추천
  - Integration(Slack, Email 등)
  - 실행 이력 / 로그 관리 / Dashboard
  - 다양한 실행 방법 (Rest API / Scheduling / 수동 실행)
  - 계정별 권한 관리
  - 파이프 라인
  - Web UI + Script 둘 다 사용 가능
  - Plugin (Ansible, Github Logentries 등)
- Jenkins에서 Spring Batch 실행

```terminal
java -jar Application.jar --job.name=job이름 jobparameter=param1
```

-job.name은 스프링 환경 변수

- Jenkins 공통 설정 관리
  - 아래처럼 Job 마다 공통된 코드가 Job마다 필요함 -> 수정이 불편하다

```
java -jar \
-----------------------
-XX:+UseG1GC \
-Dspring.profiles.active=dev \
Application.jar \
-----------------------------           모든 Batch Job의 공통 코드
--job.name=job이름 \
job파라미터이름1=job파라미터1 
```

- 해결방법

  - Jenkins의 Environment variables을 사용
  - 수정 사항이 생길 때 한 곳만 변경하면됨
- 무중단 배포

  - 기존에 잘 돌고있는 jar를 냅두고 배포할 수 있을까?

    - readlink

      - link파일을 실행하지 않고 원본 파일을 실행
- `java -jar $(readlink/.../app.jar)` -> `java -jar .../v1.jar`
      - `ln -s -f ` 을 이용해서 v2로 바꾸면 app.jar가 
- Pipeline
  - 같은 Job인데 Parameter / Schedule만 다른 경우?
    - 배민 / 배라 매출 찾기
      - Parameter만 다름
    - -> Pipeline으로 순차적으로 실행되도록 설정
  - 이전 job에서 처리한 데이터가 다음 job에 필요한 경우
  - 여러 작업이 순차적으로 실행이 필요할 때 Step으로 나누기 보다는 Pipeline을 우선 고려한다
    - 3개의 작업이 필요한 경우 Step을 3개로 만들어버림
      - 그럼 중간에 Step 하나를 생략해야하는 경우? 코드 수정이 필요함
      - Pipeline을 이용할 경우 변경이 쉬움. 여기선 그냥 Job 하나에 Step 하나로 처리
- 멱등성
  - 연산을 여러번 적용하더라도 결과가 달라지지 않는 성질
  - 어플리케이션 개발에서 멱등성이 깨지는 경우 > 제어할 수 없는 코드를 직접 생성할 때
    - ex) Scanner(System.in) 이나 Localdate.now()를 실행하는 경우
  - 코드 상에 LocalDate.now()가 들어간 경우 매번 돌릴 때 마다 기준 시간이 달라짐
  - 만약 어제 데이터를 얻어오려면 어떻게 할 수 있을까?
    - 코드를 임시 수정 & 임시 배포 하고 다시 원래 코드로 배포
  - 해결 방법 -> 원하는 날짜를 입력 받아서 실행
  - 그럼 Jenkins에서 어떻게 localdate를 쓸 수 있을까?
    - Plugin을 사용하면됨
      - Parameter 선언부에 LocalDate.now().minusDays(1);를 사용할 수 있음

- 테스트 코드
  - JobName이 환경변수로 넘어올 때만 배치 Config를 활성화한다
  - @ConditionalProperty
    - 스프링은 전체 테스트 수행시 Environment가 변경될 때마다 Spring Context를 재시작
      - Environment가 변경되는 조건
        - @MockBean / @SpyBean 사용할 때
        - @TestPropertySource로 환경변수 변경할 때
        - @ConditionalOnProperty로 테스트마다 Config가 다를 때
  - @ConditionalProperty 없애기
    - 어떻게? 모든 Config를 Loading 한 뒤, 원하는 배치 Job Bean을 실행한다.
- JPA & Spring Batch
  - JPA N + 1
    - @OneToMany 관계에서 하위 엔터티들을 Lazy Loading으로 가져올 때마다 조회 쿼리가 추가로 발생하는 이슈
    - 해결 방법 - Join Fetch
    - 하위 엔터티 2개 종류 이상에서 Join Fetch 사용시 MultipleBagFetchException
    - 해결 방법 - default_batch_fetch_size 사용
      - 하위 Entity를 Loading할 때 지정된 숫자만큼 상위 Entity ID를 WHERE IN ()에 넣어서 조회한다
    - 자식 Entity에도 default_batch_fetch_size만큼 조회한다
    - 하지만 ... 이 옵션은 JpaPagingItemReader에서 작동하지 않음
      - (HibernateCursorItemReader는 가능)
      - JpaRepository에서도 사용할 수 있는 정상 기능
  - Persist Writer
    - JpaItemWriter에서 발생하는 문제
      - 내부에서 entityManager.merge가 실행됨
        - 처음 데이터가 save될 때도 INSERT와 UPDATE가 둘 다 실행됨
      - 처음쓰이는 경우 Persist하도록 따로 Writer를 만들기
      - Batch 4.2버전부터 Persist을 쓸지 Merge를 쓸지 결정하도록 해줌 

