# Elastic APM Setup

## I. 구조

![elastic-apm-setup](/Users/joohanlee/Downloads/elastic-apm-setup.png)

## II. ElasticSearch, Kibana, APM Server Docker 이미지 올리기

-  docker-compose.yml 파일

  - 
  - `docker-compose up` - 이미지 생성 & 컨테이너 시작
  - `docker-compose stop` - 멈추기
  - `docker-compose down` - 멈추고 삭제


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

## III. APM agent

- Agent로 jar파일 실행해서 데이터를 apm-server로 보내는 역할
- APM Agent 다운받기 ("https://search.maven.org/search?q=a:elastic-apm-agent")
- 현재까지(2019년 12월 26일) agent의 경우 1.12.0버전이 최신
- apm 서버의 경우 8200이 기본 포트라서 위에 docker-compose.yml에서도 8200으로 맵핑
- 현재 spring을 사용중이기 때문에 `-Dspring.profiles.active=prod`로 프로필 설정
  - jar파일 뒤에 쓰면 적용안되므로 앞에 써야한다 

```cmd
$ java -javaagent:elastic-apm-agent-1.12.0.jar \
     -Delastic.apm.service_name=your-service-name \
     -Delastic.apm.server_url=http://localhost:8200 \
     -Delastic.apm.secret_token= \
     -Delastic.apm.application_packages=your.base.backage \
     -jar -Dspring.profiles.active=prod your-application.jar
```

## IV. Kibana 설정

- Add APM

  - 아래쪽에 `Check agent status` 버튼을 클릭해서 agent와 잘 연결되었는지 확인
  - 바로 아래 `Load Kibana objects` 를 클릭해서 APM에서 kibana 데이터로 불러오기


## V. References

- https://www.elastic.co/kr/blog/monitoring-applications-with-elasticsearch-and-elastic-apm

- https://jistol.github.io/docker/2019/03/27/docker-compose-elasticsearch-cluster/

- https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html#%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EB%91%98%EB%9F%AC%EB%B3%B4%EA%B8%B0

