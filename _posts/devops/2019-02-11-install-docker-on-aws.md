



-----

layout: post
title: "AWS에서 Docker로 Elastic APM 설정"
date: 2020-02-11 10:27:29 +0900
tags: ["aws", "docker", "elastic_apm", "elastic_search"]
categories: [devops]

-----

## I. AWS에 docker 설치 (Amazon Liux 2 기준) 

```bash
$ amazon-linux-extras install docker
$ sudo amazon-linux-extras install docker
$ sudo systemctl start docker
$ sudo usermod -aG docker ec2-user
$ sudo exit
$ docker ps
$ sudo chkconfig docker on
$ sudo curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ docker-compose --version
$ docker-compose up -d
```

## II. CTOP 설치하기

- docker monitoring tool

```bash
$ sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.3/ctop-0.7.3-linux-amd64 -O /usr/local/bin/ctop
$ sudo chmod +x /usr/local/bin/ctop
```

## III. 추가 SSD 사용하기

- SSD가 존재하는지 확인

```
$ lsblk

NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0 69.9G  0 disk
nvme0n1       259:1    0    8G  0 disk
|-nvme0n1p1   259:2    0    8G  0 part /
`-nvme0n1p128 259:3    0    1M  0 part
```

- 사용할 수 있도록 포맷시키기

```bash
$ sudo mkfs.xfs /dev/nvme1n1
```

- Link걸기

```bash
$ sudo mount /dev/nvme1n1 /data
```

- Link 확인하기

```
$ df /data
Filesystem     1K-blocks   Used Available Use% Mounted on
/dev/nvme1n1    73206424 105984  73100440   1% /data
```

- docker에서 volume설정으로 쓸 수 있도록 설정하기

```bash
$ sudo chmod 777 /data
```

## IV. Docker-compose.yml 작성

- `vim docker-compose.yml` 명령어로 docker-compose 파일 작성하기
  - 실행은 `docker-compose up -d`

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
      - apm-server.secret_token="xxVpmQB2HMzCL9PgBHVrnxjNXXw5J7bd79DFm6sjBJR5HPXDhcF8MSb3vv4bpg44"
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



## IV. logstash pipeline config파일 작성하기

- `vim config/logstash/pipeline/logstash.conf`
  - 위에 docker-compose.yml 설정한 logstash pipeline 설정이 연결된 폴더에 작성하기

```
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

## V. 실행하기

- `docker-compose up -d`
- `ctop` 실행해서 `l` 명령어로 확인하고 싶은 instance내부에 로그를 확인해서 정상적으로 실행했다고 나오면 됨
  - Logstash 내부에서 file_beats 뭐라고 나오면 pipeline config이 설정 안된 것이므로 volume이나 conf 파일 설정을 확인

