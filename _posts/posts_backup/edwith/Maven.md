---
title: "Maven"
date: 2019-02-24T11:12:08+09:00
archives: "2019"
tags: ["MAVEN", "edwith"]
author: Joohan Lee
---

## Maven

### 1. 기존 외부 Libray이용 방식의 문제점

- 직접 jar 파일을 받아서 /lib 속에 넣어줌
  - 프로젝트가 복잡해질수록 사용하는 Library의 양이 많아지고 컴파일과 배포가 어려워짐
  - 여러 사람이 함께 작업할 경우 관리가 어려움
- Maven을 사용하면 <dependency>를 추가해줌으로써 직접 다운로드 받거나 하는 것을 하지 않아도 라이브러리를 사용할 수 있다.
- Maven에 설정한 대로 모든 개발자가 일관된 방식으로 빌드 -> 빌드 방법에 대해 따로 가이드를 만들 필요 없음
- Maven의 다양한 플러그인을 이용하여 많은 일들을 자동화

### 2. Maven이란

- Application 개발을 위해 반복적으로 진행해왔던 작업들을 지원하기 위하여 등장한 도구
- 빌드(Build), 패키징, 문서화, 테스트와 테스트 리포팅, git, 의존성관리, svn등과 같은 형상관리서버와 연동(SCMs), 배포 등의 작업을 손쉽게 할 수 있음
- CoC(Convention over Configuration) 기반
  - 프로그램의 소스파일은 어떤 위치에 있어야 하고, 소스가 컴파일된 파일들은 어떤 위치에 있어야 하고 등이 미리 정해짐
  - 익숙한 사용자는 쉽게 사용할 수 있는데, 익숙하지 않다면 이러한 제약사항에 대해서 거부감을 느낄 수 있음
  - Maven을 사용한다는 것은 어쩌면 이러한 CoC를 배워가는 것

### 3. Maven 기본

- Archetype을 이용하여 Maven 기반 프로젝트를 생성할 경우 생성된 프로젝트 하위에 pom.xml이 생성됨

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>kr.or.connect</groupId>
    <artifactId>examples</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>mysample</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

- 각 태그의 의미
  - project: pom.xml파일의 Root Element
  - modelVersion: POM model의 버전
  - groupId: 프로젝트를 생성하는 조직의 고유 아이디 (일반적으로 도메인 이름을 거꾸로)
  - artifactId: 해당 프로젝트에 의하여 생성되는 artifact의 고유 아이디를 결정
    - 빌드할 경우 다음과 같은 규칙으로 artifact가 생성 artifactid-version.packaging
    - 위 예의 경우 빌드할 경우 mysamexamples-1.0-SNAPSHOT.jar
  - packaging: 해당 프로젝트를 어떤 형태로 packaging 할 것인지 결정 ex) jar, war, ear 등
  - version : 프로젝트의 현재 버전
    - 프로젝트가 개발 중일 때는 SNAPSHOT을 접미사로 사용
  - name : 프로젝트의 이름
  - url : 프로젝트 사이트가 있다면 사이트 URL을 등록
  - dependencies: 필요한 Library를 넣으면 해당 Library를 사용하기 위해 필요한 depencencies까지 알아서 받아줌
- 실습: https://www.edwith.org/boostcourse-web/lecture/16724/
- 기본 Maven Web Proejct 구조
- project
  ├─src
  │  └─main
  │      ├─java : Java package 폴더와 source code가 있음
  │      ├─resources: *.properties, *.xml 등 설정 파일
  │      └─webapp :웹 관련 리소스들
  │          └─WEB-INF
  ├─target: Compile, Packaging된 결과물
  │    ├─classes
  │    ├─m2e-wtp
  │    │  └─web-resources
  │    │      └─META-INF
  │    │          └─maven
  │    │              └─lee.joohan
  │    │                  └─mavenweb
  │    └─test-classes: 테스트 관련 Java package, source code, 설정관련 파일 등이 존재
  └─pom.xml: Maven 설정 파일
- dependency의 scope
  - **compile** : 컴파일 할 때 필요. 테스트 및 런타임에도 클래스 패스에 포함. scope 을 설정하지 않는 경우 기본값입니다.
  - **runtime** : 컴파일 시에는 필요하지 않지만, 실행 시에 필요한 경우 ex) JDBC
  - **provided** : 컴파일 시에 필요하지만, 실제 런타임 때에는 컨테이너 같은 것에서 제공되는 모듈(배포 시 제외 됨) ex) servlet, jsp api
  - **test** : 테스트 코드를 컴파일 할 때 필요. 테스트 시 클래스 패스에 포함되며, 배포 시 제외

### 4. 실습

원본 강의: https://www.edwith.org/boostcourse-web
