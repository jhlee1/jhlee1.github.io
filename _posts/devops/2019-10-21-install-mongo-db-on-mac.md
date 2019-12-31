---
layout: post
title: "Mac에서 MongoDB 설치하기"
date: 2019-10-21 12:09:29 +0900
tags: ["devops", "mongodb", "docker"]
categories: [devops, database]
---

## I. Docker를 이용한 MongoDB 서버 설치

1. docker image 받기
```bash
$ docker pull mongo
```

2. 이미지 실행
   - -p: mongoDB는 기본적으로 27017포트를 사용하고 있기 때문에 `-p [사용할 포트]:[해당 이미지 내부에서 사용하고 있는 포트] `를 통해서 해당 포트랑 연결시켜주기
   - --name : 컨테이너명
   - -d : 어떤 이미지를 사용할 것인지(여기서는 mongo이미지만 쓰고 뒤에 버전명을 생략해서 최신꺼로 받아옴)
     - ex) mongo:4.2

```bash
$ docker run -p 27017:27017 --name mongodb-example -d mongo
```

3. Local에 MongnDB 서버를 설정하지 않고 Docker image를 사용하는 이유
   - 개인 프로젝트 / 회사 프로젝트를 같이 사용하는 경우 설정이 깔끔하게 분리됨

## II. MongoDB Shell Client설치를 위해 Enterprise Server 설치하기 (Mac기준)

1. https://docs.mongodb.com/manual/installation/#mongodb-enterprise-edition-installation-tutorials 여기에 OS별로 설치법이 나와있으니 참고해서 설치하면됨
2. https://www.mongodb.com/download-center/enterprise?jmp=docs 여기서 원하는 버전과 OS를 다운받기
   - Community 버전을 사용하지 않는 이유
     - 모든 튜토리얼들이 Enterprise 버전을 사용
     - Enterprise라고 유료가 아니기 때문에 실제 서비스와 유사한 환경을 갖도록하기 위해 Enterprise를 사용
3. 압축풀기

```bash
$ tar -zxvf mongodb-macos-x86_64-enterprise-4.2.0.tgz
```

4.  원하는 디렉토리로 옮기기 (나는 계정명/monogdb로 옮김)

```bash
$ mv mongodb-macos-x86_64-enterprise-4.2.0/ ~/mongodb
```

5. 위에 옮긴 폴더의 PATH 추가하기
   - 아래의 명령어로 bashrc에 들어간 후 `export PATH="/Users/유저명/mongodb/bin:$PATH"` 를 추가함
   - zshell 사용중인 경우 bashrc 대신 zshrc를 수정

```bash
$ vim ~/.bashrc
```

## III. Mac에서 homebrew로 MongoDB Community 버전 설치하기

1. Brew에 MongoDB Repository 추가하기

   - https://github.com/mongodb/homebrew-brew 여기로 연결됨

   ```bash
   $ brew tap mongodb/brew
   ```

2. MongoDB서버 설치

   - 4.2버전이 최신임(2019년 9월 27일 기준)

   ```bash
   $ brew install mongodb-community@4.2
   ```

3. MongoDB 서버 실행하기

   - 서비스로 실행

   ```bash
   $ brew services start mongodb-community
   ```

   - 서비스로 실행 취소

   ```bash
   $ brew services stop mongodb-community
   ```

   - 직접 실행

   ```bash
   $ mongod --config /usr/local/etc/mongod.conf
   ```

4. 연결 확인하기

   ```bash
   $ mongo
   ```

## IV. 참고 링크

- [Mongo in Docker Hub](https://hub.docker.com/_/mongo)

