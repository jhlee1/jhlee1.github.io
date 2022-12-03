# Elastic Search 기본 설정

## 1. config/jvm.options

- 현재는 Heap Size가 1GB로 설정되어 있음

```
-Xms1g
-Xmx1g
```

- 위의 숫자를 바꿔주면됨 (2GB로 설정하기)

```
-Xms2g
-Xmx2g
```

## 2. config/elasticsearch.yml

- 외부에서 접속할 수 있도록 설정

```yaml
network.host: 192.168.0.1 (외부에서 접속할 ip 입력)
http.port: 9200 (elasticsearch를 위한 Port 번호 입력)
```