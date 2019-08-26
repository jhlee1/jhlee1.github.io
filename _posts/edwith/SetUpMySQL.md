---
title: "Install & Set up My SQL"
date: 2019-02-16T21:12:24+09:00
archives: "2019"
tags: ["edwith", "MySQL", "database", "webProgramming"]
author: Joohan Lee
---

## Set up MySQL Community Server in windows

### 1. 다운로드

- https://www.mysql.com/downloads/ 여기서 Community Server의 다운로드 중 mysql-installer-community-X.0.XX.0.msi 받기

- web-community과 community 파일의 차이는 설치중에 파일을 받아서 설치하는 것 (인터넷 연결 필요)과 이미 설치파일에 들어있는 파일을 설치하는 것 (인터넷 연결 불필요). 따라서 설치 파일의 용량이 차이남

### 2. 설치

- msi파일을 실행 후 SetUp Type을 Developer Default로 설치
- root 비밀번호만 정하고 Next 누르면됨
- 윈도우가 켜질 때 실행시키려면 Windows Service 설정에서 시작시 켜지도록 설정해놓으면 됨
- 환경변수 설정 - C:Program Files\MySQL\MySQL Server X.X\bin을 path에 추가하기

원본 강의: https://www.edwith.org/boostcourse-web
