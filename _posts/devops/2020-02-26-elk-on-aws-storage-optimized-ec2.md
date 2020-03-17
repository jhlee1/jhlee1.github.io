---
layout: post
title: "AWS에서 ELK + APM세팅하기"
date: 2020-02-26 18:50:00 +0900
tags: ["java", "elasticsearch", "aws", "logstash", "elastic_apm"]
categories: [devops]

---

## I. Instance 준비

- Log와 apm metric 정보의 양이 많기 때문에 m5d.large를 사용할 예정
- 보안을 위해서 Securitygroup은 
  - 일반 사용자에게는 80
  - 로그와 apm 정보를 받을 서버에는 elasticsearch와 logstash port를 열어두기
- docker 설치 후 서비스로 실행 및 권한 설정
  - 유저그룹생성하고 나갔다와야 권한이 바뀌어있음

```bash
$ sudo amazon-linux-extras install docker
$ sudo systemctl start docker
$ sudo usermod -aG docker ec2-user
$ exit
```

- 잘 실행되었는지 확인

```bash
$ sudo chkconfig docker on 
$ systemctl status docker.service
```

- docker compose 설치

```bash
$ sudo curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```

- 설치 확인

```bash
docker-compose --version

```

- ctop 설치

```bash
$ sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.3/ctop-0.7.3-linux-amd64 -O /usr/local/bin/ctop
$ sudo chmod +x /usr/local/bin/ctop
```

## II. EC2에서 EBS볼륨 외에 추가된 SSD 사용하기

- `$ lsblk` 명령어로 추가 ssd nvme1n1 확인하기

```
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0 69.9G  0 disk 
nvme0n1       259:1    0    8G  0 disk
|-nvme0n1p1   259:2    0    8G  0 part /
`-nvme0n1p128 259:3    0    1M  0 part
```

- 기존 EBS와 같은 xfs 형식으로 포맷하고 mount하기
  - 귀찮으니 /data 폴더에 접근 권한을 모두 열어주자
```bash
$ sudo mkfs.xfs /dev/nvme1n1
$ sudo mount /dev/nvme1n1 /data
$ sudo chmod 777 /data
```

- filesystem 확인하기 `sudo file -s /dev/nvme?n*`

```bash
/dev/nvme0n1:     x86 boot sector; partition 1: ID=0xee, starthead 0, startsector 1, 16777215 sectors, extended partition table (last)\011, code offset 0x63
/dev/nvme0n1p1:   SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
/dev/nvme0n1p128: data
/dev/nvme1n1:     SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
```

## III. docker-compose  파일 작성

```yaml
version: '3.7'
services:
  elasticsearch:
    container_name: elasticsearch_compose
    image: elasticsearch:7.5.1
    environment:
      - cluster.name=docker-compose-cluster
      - node.name=master-node
      - bootstrap.memory_lock=true
      - discovery.type=single-node
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - es_net
    stdin_open: true
    tty: true
    volumes:
      - /data:/usr/share/elasticsearch/data
  kibana:
    container_name: kibana_compose
    image: kibana:7.5.1
    ports:
      - 5601:5601
      - 80:5601
    networks:
      - es_net
    depends_on:
      - elasticsearch
    stdin_open: true
    tty: true
  apm-server:
    container_name: apm_server_compose
    image: store/elastic/apm-server:7.5.1
    ports:
      - 8200:8200
    environment:
      - output.elasticsearch.hosts=['http://elasticsearch:9200']
      - apm-server.host="0.0.0.0:8200"
      - apm-server.secret_token="YOUR_SECRET_TOKEN"
      - setup.kibana.host="kibana:5601"
      - setup.template.enabled=true
      - logging.to_files=false
    networks:
      - es_net
    depends_on:
      - elasticsearch
      - kibana
  logstash:
    container_name: logstash_compose
    image: logstash:7.5.1
    ports:
      - 9600:9600
      - 4560:4560
    networks:
      - es_net
    depends_on:
      - elasticsearch
    volumes:
      - ~/config/logstash/pipeline:/usr/share/logstash/pipeline/
networks:
  es_net:
```

## IV. logstash pipeline config파일 작성

- logstash 설정파일과 logstah pipline설정 파일이 따로 존재함
  - TCP Request를 받아서 Elasticsearch에 저장할 수 있도록 logstash pipeline 설정
    - docker compose를 보면 4560에서 TCP 요청을 받도록 열어둠
  - 위에 docker compose 내의 logstash 서비스의 volume으로 외부에 pipeline 설정과 연결하도록 설정함
  - `$ vim config/logstash/pipeline/logstash.conf`로 파일을 열고 아래 내용 집어넣기

```bash
input {
  tcp {
    codec => json_lines
    port => 4560
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "server-logs"
  }
}

```



## V. 이미지 실행

- `$ docker-compose up -d`

- 위에서 설치한 `ctop` 명령어로 인스턴스가 잘 떠있는지, `ctop` 내에서  `l` 명령어로 정상실행 로그를 확인할 수 있음 

## VI. Spring boot 프로젝트에서 logger 설정

- Springboot는 기본적으로 Slf4j를 사용
- 기존 프로젝트에 영향이 없도록 기본 스프링 설정 위에 logstash를 추가
- `resources/logback-spring.xml` 작성
  - MSA 구조에서 각 서버별 구분을 위해서 customField를 추가
  - profile별로 구분해서 log를 보내도록 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <include resource="org/springframework/boot/logging/logback/base.xml"/>

  <springProfile name="prod">
    <appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
      <destination>x.x.x.x:4560</destination>
      <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"server":"example1-prod"}</customFields>
      </encoder>
    </appender>

    <root level="INFO">
      <appender-ref ref="stash"/>
    </root>
  </springProfile>

  <springProfile name="staging">
    <appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
      <destination>x.x.x.x:4560</destination>
      <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"server":"example1-staging"}</customFields>
      </encoder>
    </appender>

    <root level="INFO">
      <appender-ref ref="stash"/>
    </root>
  </springProfile>
</configuration>

```

