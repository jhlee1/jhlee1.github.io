---
layout: post
title: "Elastic APM Setup 방법"
date: 2019-12-31 12:35:00 +0900
tags: ["elasticsearch", "apm"]
categories: [devops]
image: elastic-apm-setup.png
---

# Elastic APM Setup

## I. 구조

![elastic-apm-setup](/static/img/posts/devops/elastic-apm-setup.png)

## II. ElasticSearch, Kibana, APM Server Docker 이미지 올리기

- `docker-compose.yml` 파일
  - `docker-compose up`: 이미지 생성 & 컨테이너 시작
  - `docker-compose stop`: 멈추기
  - `docker-compose down`: 멈추고 삭제


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
  kibana:
    container_name: kibana_compose
    image: kibana:7.5.1
    ports:
      - 5601:5601
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
      - setup.kibana.host="kibana:5601"
      - setup.template.enabled=true
      - logging.to_files=false
    networks:
      - es_net
    depends_on:
      - elasticsearch
      - kibana
networks:
  es_net:

```

- apm-server에 username & password 또는 Token을 추가할 수도 있음
  - apm server의 environment 필드에 추가하면됨
  - apm server에 추가하는 경우 client에도 추가해줘야함

```yaml
apm-server:
  environment:
    - apm-server.secret_token="xxVpmQB2HMzCL9PgBHVrnxjNXXw5J7bd79DFm6sjBJR5HPXDhcF8MSb3vv4bpg44"
```

- terminal bash 쓰기 위해 아래 부분 추가

```yaml
stdin_open: true
tty: true
```

## III. APM agent 실행

- Agent로 jar파일 실행해서 데이터를 apm-server로 보내는 역할
- 현재까지(2019년 12월 26일) agent의 경우 1.12.0버전이 최신
- apm 서버의 경우 8200이 기본 포트라서 위에 docker-compose.yml에서도 8200으로 맵핑
- 아래의 두가지 방법중 하나를 사용하면됨

### 1. jar파일 직접 받기
   - APM Agent 다운받기 ("https://search.maven.org/search?q=a:elastic-apm-agent")
   - 현재 spring을 사용중이기 때문에 `-Dspring.profiles.active=prod`로 프로필 설정
      (**jar파일 뒤에 쓰면 적용안되므로 앞에 써야한다**)
   - 맨 끝에 apm agent를 붙일(attach) jar파일을 넣으면됨

```bash
$ java -javaagent:elastic-apm-agent-1.12.0.jar \
     -Delastic.apm.service_name=your-service-name \
     -Delastic.apm.server_url=http://localhost:8200 \
     -Delastic.apm.secret_token= \
     -Delastic.apm.application_packages=your.base.package \
     -jar -Dspring.profiles.active=prod \
     your-application.jar
```

### 2. Maven이용하기 
  - (위 방법의 경우 서비스를 빌드할 때마다 jar파일을 만들어서 agent를 통해 실행시켜야하는 불편함이 있기 때문에 maven으로 간단하게 설정)
  - Dependency 추가 (pom.xml)
    ```xml
    <dependency>
      <groupId>co.elastic.apm</groupId>
      <artifactId>apm-agent-attach</artifactId>
      <version>1.12.0</version>
    </dependency>
    ```
  - 프로그램 시작지점에 agent 실행코드 추가(아래는 Spring Boot를 사용했지만 꼭 Spring Boot를 쓸 필요는 없음)
    ```java
      @SpringBootApplication
      public class SpringMongoExampleApplication {
        public static void main(String[] args) {
          ElasticApmAttacher.attach();
          SpringApplication.run(SpringMongoExampleApplication.class, args);
        }
      }
    ```
  - elasticapm.properties 설정하기
  ```properties
    service_name=spring-boot-example
    server_url=http://localhost:8200
    application_packages=lee.joohan
  ```
  - 설정별 자세한 부분은 [Official Reference](https://www.elastic.co/guide/en/apm/agent/java/current/config-reference-properties-file.html)

  
## IV. Kibana 설정

- Add APM
  - 아래쪽에 `Check agent status` 버튼을 클릭해서 agent와 잘 연결되었는지 확인
  - 바로 아래 `Load Kibana objects` 를 클릭해서 APM에서 kibana 데이터로 불러오기


## V. References

- https://www.elastic.co/kr/blog/monitoring-applications-with-elasticsearch-and-elastic-apm

- https://jistol.github.io/docker/2019/03/27/docker-compose-elasticsearch-cluster/

- https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html#%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EB%91%98%EB%9F%AC%EB%B3%B4%EA%B8%B0
- https://www.elastic.co/guide/en/apm/agent/java/current/setup-attach-api.html
