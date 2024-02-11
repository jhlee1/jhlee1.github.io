---
layout: post
title: "Docker 기본 명령어"
date: 2019-08-09 18:55:29 +0900
tags: ["devops", "docker"]
categories: [devops]
---

## 1. docker -v

- docker version

## 2. docker run -i -t ubuntu:14.04

- -i 상호 입출력
- -t tty를 활성화 해서 bash 사용가능하게함

## 3. docker에서 빠져나오기 

- Ctrl + D | exit
  - 해당 컨테이너 종료 후 나오기
- Ctrl + P, Q
  - 해당 컨테이너 종료하지 않고 빠져나오기

## 4. docker images

- 가지고 있는 Docker image 리스트

## 5. docker create -i -t —name mycentos centos:7

- mycentos라는 이름으로 centos:7의 컨테이너 생성
- 결과로 출력되는 16진수 해시값은 container의 고유 ID
- run과 create의 차이
  - run: docker pull -> docker create -> docker start -> docker attach (-i -t 옵션을 사용했을 때)
  - create: docker pull -> docker create

## 6. docker start :containerName

- ex) `docker start mycentos`
- 해당 컨테이너 실행

## 7. docker attach :containerId

- ex) `docker attach 5440394aa2d3`
- 실행중인 container안에 접속하기
- 앞의 세자리만 입력해도되지만 다른 컨테이너와 중복될 경우 에러 발생 ex) docker start 5440 / docker attach 5440

## 8. docker ps -a

- docker ps => 현재 실행중인 모든 container 리스트를 가져온다
- -a => 정지된 container까지 가져오기

| CONTAINER ID | IMAGE    | COMMAND     | CREATED    | STATUS       | PORTS | NAMES    |
| ------------ | -------- | ----------- | ---------- | ------------ | ----- | -------- |
| 5edca2abfb96 | centos:7 | "/bin/bash" | 6 days ago | Up 7 minutes |       | mycentos |

- COMMAND 
  - 컨테이너 생성후 처음 실행할 명령어
  - 여기선 `/bin/bash`를 통해서 shell을 생성해줌
  - `docker run -i -t mycentos echo hello world!`
    - `echo hello world!`만 실행하고 shell이 생성되지 않아 컨테이너가 종료됨
- NAMES
  - 컨테이너 이름
  - rename 명령어로 변경 가능
    - ex) `docker rename mycentos mycentos2`
- 출력 형식 변경하거나 원하는 필드만 출력 가능
  {% raw %}
    `docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Image}}\t{{.Name}}"`
  {% endraw %}
## 9. docker rm :container_name

- 해당 이름의container 삭제
- container의 name 대신 id도 사용 가능
- 현재 실행중인 container를 삭제하려는 경우 에러 발생
  - `docker rm -f :container_name` 처럼 -f를 사용해서 강제로 지우거나
  - `docker stop :container_name`으로 먼저 container를 중단시키고 지워야한다
- image 삭제는 `docker rmi :image_name`

## 10. docker container prune

- 현재 실행중이지 않은 모든 컨테이너 삭제

## 11. 실행중이나 정지 상관없이 모든 container 삭제 방법

- `docker ps -a -q`
  - -a 모든 컨테이너의 -q id만 출력
- 아래의 방식으로 모두 멈추고 삭제
  - `docker stop ${docker ps -a -q}`
  - `docker rm ${docker ps -a -q}`

## 12. docker run -p (host_port):(contaner_port)

- ex) `docker run -i -t -p 80:80 --name myapacheserver centos:7`
  - 호스트에 80번 포트로 요청이 올 경우 해당 컨테이너의 80번 포트로 넘김
  - 80:80 말고 그냥 80만 쓰는 경우 현재 호스트에서 사용가능한 포트와 container의 80과 연결해줌
- 여러개의 port를 연결하는 경우
  - `docker run -p 8801:8801 -p 192.168.0.10:8888:80` 처럼 -p를 여러개 추가하면 됨
- docker에 Apache web server를 올려서 검사해보기
  - `docker run -i -t -p 80:80 --name myapache ubuntu`
  - (docker contaner 내부) `apt-get update`
  - (docker contaner 내부) `apt-get install apache2 -y`
  - (docker contaner 내부) `apt-get apache2 start`
  - Ctrl P, Q로 docker 빠져나오기
  - 브라우저에서 localhost:80으로 접속하거나 `curl localhost:80` 보내보기

## 13. DB와 Web Server 분리하기

- `docker run -d --name wordpressdb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress mysql:5.7`

  - wordpress용 mysql database container 생성

- `docker run -d  --name wordpress --link wordpressdb:mysql -e WORDPRESS_DB_PASSWORD=password -p 80 wordpress`

  - wordpress cointainer 생성 후 위에서 생성한 mysql container와 연결

- wordpress의 포트 확인 

  - `docker port wordpress`
  - output: `80/tcp -> 0.0.0.0:32768`
    - 0.0.0.0은 host의 활용가능한 모든 네트워크 인터페이스를 바인딩함

- localhost:32768로 접속 후 워드 프레스 페이지가 나오는지 확인

- -d: Detached 모드로 실행

  - -i -t로 attach 모드로 실행해서 shell을 사용할 수 있었지만 Detach모드의 경우 container가 background에서 동작하도록 설정
  - container 내부적으로 terminal 대신 mysql이 foreground로 동작하기 때문에 attach로 접속 불가
  - foreground로 동작하던 프로그램(이 경우엔 mysql)이 종료되면 컨테이너가 꺼짐
  - -d 대신에 -i -t를 사용하여 컨테이너를 실행한 경우 해당 container에 접속되어 mysql의 실행상태를 확인할 순 있지만 여전히 mysql이 foreground로 실행되어 terminal 사용이 불가능하다. 그 이유는 MySQL 이미지의 설정이 container가 시작할 때 bash가 아니라 mysqld를 실행하도록 되어있기 때문이다.
  - mysql에서 shell을 사용하려면 실행중인 container에서 `docker exec -i -t wordpressdb /bin/bash`로 terminal을 실행하도록 하면됨

- -e: 환경변수 설정

  - 잘 설정되었는지는 실행중인 container를 docker exec로 접속해서 확인
    - `docker -i -t wordpressdb /bin/bash`
    - `env`로 전체 환경변수에서 확인하거나 `echo $MYSQL_ROOT_PASSWORD`로 확인
      - `mysql -u root -p` 로 db 접속 가능한지 도 확인 가능 

- --link: 내부 IP를 알 필요 없이 별명(alias)으로 접근할 수 있도록 설정

  - 일반적으로 container A에서 container B로 접속하는 방법은 NAT로 할당 받은 내부 IP를 사용하는 것이지만 container가 시작할 때마다 172.17.0.x값이 할당되기 때문에 매번 변경되기 때문에 IP값을 가지고 container를 찾기 힘듬

  - ex) `--link wordpressdb:mysql`

    - wordpressdb라는 container를 mysql이라는 hostname으로 접근 가능
    - `docker exec -i -t wordpress /bin/bash`로 접속후 ping을 설치하고 ping mysql을 실행하거나
    - 해당 container에 ping이 이미 설치되어있는 경우 `docker exec wordpress ping -c 2 mysql`을 이용해 테스트 해볼 수 있음

  - link되어있는 container가 실행중인 상태가 아닌경우 에러가 발생한다. container간의 연결 뿐만 아니라 의존성도 설정

    - 위의 wordpress container의 경우 link 걸어놓은 wordpressdb가 stop상태라면 해당 wordpress container를 실행할 때 에러 발생

      - `docker stop wordpress wordpressdb`
      - `docker start wordpress wordpressdb`

      ```
      Error response from daemon: Cannot link to a non running container: /wordpressdb AS /wordpress/mysql
      wordpressdb
      Error: failed to start containers: wordpress
      ```

      - `docker start wordpressdb wordpress`로 실행하면 잘됨

## 14. 하나의 container에 여러개의 session이 접속하면 어떻게 될까

- ex) 
  - `docker run -i -t --name multi_attach_test ubuntu`로 하나의 세션에서 container 접속
  - `docker attach multi_attach_test`로 두번째 세션에서 container 접속
- 모두 같은 terminal을 공유함
- 여러개로 접속한 경우 한개의 session에서 입력한 데이터를 나머지도 공유
- 컨테이너 하나당 하나의 모니터를 사용한다고 생각하면됨