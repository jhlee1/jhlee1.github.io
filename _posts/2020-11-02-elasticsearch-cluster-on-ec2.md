---
layout: post
title: "EC2로 Elastisearch Cluster 설정하기"
date: 2020-11-02 12:20:00 +0900
tags: ["aws", "ec2", "elasticsearch"]
categories: [devops]
---

## I. Elasticsearch 설치하기

- Amazozn linux 2 기준으로 설치
- Elasticsearch가 jvm위에서 실행되기 때문에 자바를 먼저 설치해줘야함

```bash
sudo yum update
sudo yum install -y java-11-amazon-corretto
```

- yum repository에 Elasticsearch 추가해주기
  - 6버전 기준 [7버전은 공식문서 참조](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/rpm.html)
  - `sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch` GPG 키 추가
  - `sudo vim /etc/yum.repos.d/elasticsearch.repo` 명령어로 repo 파일 생성 후 아래 내용 복붙

```bash
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

- elasticsearch 설치하기

  ```bash
  sudo yum install elasticsearch
  ```

- elasticsearch 서비스로 등록하기

  - `ps -p 1` 명령어로 systemd인지 initV인지 확인하기 
  - 아래처럼 나오는데 CMD가 systemd이므로 systemd 설정해주면됨 (Amazon Linux 2기준)

  ```
    PID TTY          TIME CMD
      1 ?        00:00:02 systemd
  ```

  - service 등록하기

  ```bash
  sudo /bin/systemctl daemon-reload
  sudo /bin/systemctl enable elasticsearch.service
  sudo systemctl start elasticsearch.service
  ```

- 작동되는지 확인하기

  - service 상태확인

  ```bash
  sudo service elasticsearch status
  ```

  - 로그 확인
    - 아래 `journalctl` 명령어로 확안힐 수 있지만 로그가 많이 생략되어보이기 때문에 실패할 경우 원인을 알 수 없다
      - --quite 옵션을 없애줘야함
  ```bash
  sudo journalctl --unit elasticsearch
  ```
  - GET 요청보내보기

  ```bash
  curl -X GET "localhost:9200/?pretty"
  ```

- `--quiet` 옵션 없애기

  - elasticsearch 실행할 때 --quiet 옵션을 빼줘야 제대로된 로그를 볼 수 있음
  - `sudo vim /usr/lib/systemd/system/elasticsearch.service` 명령어로 서비스 설정을 열어서 ExecStart 끝에 `--quite`빼주기
  - service 설정을 바꿔주고 난 후엔 `sudo systemctl daemon-reload` 명령어로 설정 적용을 시켜줘야함

  ```bash
  ExecStart=/usr/share/elasticsearch/bin/elasticsearch -p ${PID_DIR}/elasticsearch.pid --quiet
  ```

## II. Cluster 설정과 필수 시스템 설정

- disk swap 끄기

```bash
sudo swapoff -a
```

- Memlock 설정해주기

  - `LimitMEMLOCK=infinity` 를 service 설정에 추가해주기
  - `sudo vim /usr/lib/systemd/system/elasticsearch.service` 명령어로 service 설정파일 열기

  ```bash
  # Specifies the maximum size of virtual memory
  LimitAS=infinity
  LimitMEMLOCK=infinity
  ```

- discovery-ec2 플러그인 설치하기

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install discovery-ec2
```

- jvm.options에서 Heap size 늘려주기
  - `sudo vim /etc/elasticsearch/jvm.options`
  - 해당 머신 RAM의 절반을 권장

```
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms16g
-Xmx16g
```



- keystore 저장하기

  - AWS에서 IAM Role의 Credential에 들어가면 Access Key ID와 Secret Access Key가 있는데 아래 설정에 맞는 필드를 넣어주면됨
  - 해당 IAM role이 `ec2:DescribeInstances` 권한을 가지고 있는지 확인
  - `Access key ID` => `access_key`
  - `Secret Aceess Key` => `secret_key`

  ```
  sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add discovery.ec2.access_key
  sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add discovery.ec2.secret_key
  ```

  

- elasticsearch.yml 설정해주기
  
  - `sudo vim /etc/elasticsearch/elasticsearch.yml` 파일열고 아래 내용 추가해주기

```
# ---------------------------------- Cluster -----------------------------------
cluster.name: es-dev
# ------------------------------------ Node ------------------------------------
node.name: dev-node-1
# ----------------------------------- Memory -----------------------------------
bootstrap.memory_lock: true
# ---------------------------------- Network -----------------------------------
network.host: _ec2:privateIpv4_
network.bind_host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
# --------------------------------- Discovery ----------------------------------
discovery.zen.hosts_provider: ec2
discovery.ec2.any_group: true
discovery.ec2.host_type: private_ip

cloud.node.auto_attributes: true
cluster.routing.allocation.awareness.attributes: aws_availability_zone

discovery.ec2.tag.es_cluster: "dev"
discovery.ec2.endpoint: ec2.ap-northeast-2.amazonaws.com
discovery.zen.minimum_master_nodes: 2
discovery.ec2.groups: DEV-ES
```



## III. 참고링크

- https://medium.com/@abhinav.gupta.2406/ec2-discovery-with-elasticsearch-f9c9b0a67b79
- https://blog.francium.tech/install-aws-ec2-discovery-plugin-in-elasticsearch-5a973348cad9
- https://www.elastic.co/guide/en/elasticsearch/plugins/current/discovery-ec2-usage.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.9/rpm.html