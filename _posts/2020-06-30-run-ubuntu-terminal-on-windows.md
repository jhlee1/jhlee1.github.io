---
layout: post
title: "Running ubuntu terminal on windows"
date: 2020-06-30 23:08:00 +0900
tags: ["rds", "ubuntu", "window"]
categories: [devops]

---


## I. 윈도우에서 Terminal 사용하기
- Linux feature 허용하기
```
    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
- 기능 설치
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
- 재부팅 후 wsl2를 기본으로 설정하기
```
wsl --set-default-version 2
```

- [윈도우 스토어](https://aka.ms/wslstore)에서 [Ubuntu20.04](https://www.microsoft.com/ko-kr/p/ubuntu-2004-lts/9n6svws3rx71?rtc=1&activetab=pivot:overviewtab) 설치하기


## 2. Docker 설치하기
- 여기서 설치하는건 docker client이므로 사전에 Windows에 Docker가 설치되어야하고 `Expose daemon on tcp://localhost:2375 without TLS` 설정에 체크되어 있어야한다.
- Docker 공식문서를 참고해서 설치하면 되지만 WSL을 사용하면서 생기는 문제점들이 있다.
- 아래 명령어를 실행할 때 에러가 발생
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
```
- gpg를 지우고 gnupg1을 사용하도록 변경
```
$ sudo apt remove gpg
$ sudo apt-get update -y
$ sudo apt-get install -y gnupg1
```
- 다시 명령어 공식문서대로 설치
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
$ apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
- 아래 명령어로 docker를 실행해보면 `Cannot connect to the Docker daemon at tcp://localhost:2375. Is the docker daemon running?` 에러가 발생한다
```
docker ps
```
- docker client한테 host의 위치를 알려줘야한다. 환경변수를 설정해주자
```
$ echo "export DOCKER_HOST=localhost:2375" >> ~/.bash_profile
```

## 3. 참고링크
https://medium.com/@sebagomez/installing-the-docker-client-on-ubuntus-windows-subsystem-for-linux-612b392a44c4
