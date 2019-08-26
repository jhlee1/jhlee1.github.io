---
title: "Biz 민첩성과 아키텍처 요건"
date: 2019-02-24T09:49:59+09:00
archives: "2019"
tags: ["K-MOOC", "MicroService"]
author: Joohan Lee
---

## Biz 민첩성과 아키텍처 요건

### 1. 11.6초

- Amazon이 Application을 배포하는 주기
- 일반 온라인 기업
  서비스 기획 -> 서비스 지원용 어플리케이션 개발 -> (개선 사항 발생) -> 수정 후 배포
- vs. 국내 쇼핑몰 배포 주기 - 3일
- 최근 빠른 비지니스의 변화로 빠르게 대응할 능력이 필요함

### 2. 개발 문화의 변화 (DevOps)

- "The traditional model is that you take your software to the wall that separates development and operations, and throw it over and then forget about it. Not at Amazon. You build it, you run it." - Werner Vogels (아마존 CTO)
- 기존 - 개발조직과 운영 조직 분리 -> 최근 - 개발과 운영을 함께 진행
- DevOps 조직의 특징
  - 개발 시 빠른속도
  - 전체적 운영의 책임감
  - 문제 발생시 쉬운 개선

### 3. 개발 환경이 이루는 물리적인 하드웨어 인프라

- 기존 - 인프라 구축을 위한 많은 시간이 필요함 (N주 ~ N달)
  - 컴퓨터 장비 설치
  - 서버 장비 구입
  - 네트워크 연결 및 운영체제 설치
- 최근 - 클라우드 환경의 등장

### 4. 클라우드 인프라

- 필요에 따라 쉽고 편한 서비스 형태로 사용
  - 사용자 측면 - 사용한 만큼 과금(Application)
  - 개발자 측면 - 필요한 만큼 과금(Application이 올라가는 Infra)
- ex) AWS, Google Cloud, Azure, Bluemix, 클라우드Z

### 5. 어플리케이션 측면에서의 필요성 (ex 타임 세일)

- Problem: 이벤트 참가용 사용량 증가 -> 전체 서비스 성능 저하
- 해결 방법
  - Scale-up: 사용량 증가에 따른 물리적 인프라 사전 준비 또는 높은 용량의 장비로 대체
  - Scale-out: 전체 시스템을 복사하여 증설 -> 클라우드에서 많이 사용하는 방식
- 마이크로 서비스의 필요성
  - 일부 서비스만 확장이 필요한데 전체 서비스를 복사하는 것은 자원의 낭비
  - 각 서비스를 독립적으로 분리하여 필요한 부분만 확장 ex) 타임 서비스