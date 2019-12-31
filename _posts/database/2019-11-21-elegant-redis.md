---
layout: post
title: "우아한 Redis"
date: 2019-11-21 19:30:00 +0900
tags: ["redis", "database"]
categories: [database]
---

## I. Redis 소개

- In-memory Data Structure Store

  - vs. EHCache - Local Cache
    - vs. MemCache - 제한된 자료구조

- Support data structures
  - Strings, set, sorted-set, hashes, list
  - Hyperloglog, bitmap, geospatial index
  - Stream
  
- Only 1 Committer

- Cache
  - 빨리 사용하기 위해 나중에 요청할 결과를 미리 저장해두는 것
  - ex) Factorial - 9! 를 미리 저장해놓으면 10! 은 손쉽게 가능
  - 어디서 사용할까?
  - 일반적인 구조 = Client - Web Server - DB
    
    - 파레토의 법칙 - 전체 요청의 80%는 20%의 사용자
  - Cache를 사용한 구조
    - Look aside Cache - cache를 조회 후 없으면 db에 조회
      1. Web Server는 데이터가 존재하는지 Cache를 먼저 확인
      2. Cache에 데이터가 있으면 Cache에서 가져온다
      3. Cache에 데이터가 없으면 DB에서 읽어온다
      4. 읽어온 데이터를 Cache에 다시 저장
    - Write Back - Cache에 일단 저장해놓고 나중에 몰아서 DB 반영
      - Cache에서 DB에 쓰여지기 전에 시스템이 다운되는 경우 데이터 손실 발생
      1. Web Server는 모든 데이터를 Cache에만 저장
      2. Cache에 특정시간동안의 데이터가 저장
      3. Cache에 있는 데이터를 DB에 저장하고 DB에 저장된 데이터를 Cache에서 삭제

## II. 왜 Collection이 중요한가

- Redis의 강점이다. 사실 Collection이 필요없으면 굳이 Redis 안쓰고 MemCache 써도됨

- 개발의 편의성
  - 이미 만들어진 것을 쓸 수 있음
  - Ex) 랭킹 서버를 직접 구현한다면?
    - 가장 간단한 방법
      - DB에 유저의 Score를 저장하고 Score로 Order By로 구현
      - 개수가 많아지는 경우 IO에 부하가 생김
    - Redis의 Sorted Set을 이용하면 랭킹을 구현하기 쉽다
  
- 개발의 난이도
  - Race Condition
    - EX) 친구 리스트 관리
    - 시나리오: Social:Friends를 key로 저장해서 친구 A를 저장 -> B를 A 뒤에 붙이고 -> C를 B 뒤에 붙인다
    
      | Order | Task1(B를 추가)         | Task2(C를 추가)         | Result |
      | ----- | ----------------------- | ----------------------- | ------ |
      | 1     | Social:Friends 읽어오기 |                         | A      |
      | 2     |                         | Social:Friends 읽어오기 | A      |
      | 3     | 친구 B 추가             |                         | A      |
      | 4     |                         | 친구 C 추가             | A      |
      | 5     | Social:Friends에 쓰기   |                         | A, B   |
      | 6     |                         | Social:Friends에 쓰기   | A, C   |
  
  
  
  - Redis의 경우 자료구조가 Atomic하기 때문에, 해당 Race Condition을 피할 수 있음
    - 그래도 잘못짜면 발생함 (Ex. 따닥)
      - 따닥: 빠르게 같은 요청을 두번 보내는 경우를 뜻하는 전문용어임. 앞단에서 같은 요청을 빠르게 연속해서 보내지 않도록 잘 처리해야됨
  - 외부 Collection을 잘 이용하는 것으로 여러가지 개발 시간 단축, 여러 문제 해결
  
- 사용처
  - Remote Data Store
    - A서버, B서버, C서버에서 데이터를 공유하고 싶을 때 
  - 한대에서만 필요한다면, 전역 변수를 쓰면 되지 않을까?
    - Redis 자체가 Atomic을 보장해준다 (싱글 스레드라 ...)
  - 많이 쓰는 곳들
    - 인증 토큰 저장 (String or Hash)
    - Ranking 보드 사용 (Sorted Set)
    - 유저 API Limit
    - 잡 큐(list)

## III. Redis Collections

- 가장 많이 쓰이는 것은 key 와 sorted set

- O(N) 걸리는 자료구조 또는 메소드를 조심해야됨

- Strings 

  - key에 prefix를 넣어두면 여러 곳에서 쓸 때 유용하다
  - 단일 Key
    - 기본 사용법
      - `Set <key> <value>`
      - `Get <key>`
      - EX)
        - `Set Example:1 example1`
        - `Get Example:1`
  - 멀티 Key
    - 기본 사용법
      - `mset <key1> <value1> <key2> <value2> ... <keyN> <valueN>`
      - `mget <key1> <key2> ... <keyN>`
      - EX)
        - `mset Example:1 example1 Example:2 example2`
        - `mget Example:1 Example:2`
  - 간단한 SQL 대체하기
    - `INSERT INTO USERS(name, email) VALUES('john', 'john.doe@example.com')`
    - Using Set
      - `Set name:john john`
      - `Set email:john john.doe@example.com`
    - Using mset
      - `Mset email name:john john email:john john.doe@example.com`

- List - Queue의 느낌으로 접근

  - `Lpush <key> <A>`
    - Key: (A)
  - `Rpush <key> <B>`
    - Key: (A, B)
  - Pop (ex. A B D C)
  - `LPOP <key>`
    - Pop A, Key: (B D C)
  - `RPOP <key>`
    - Pop A, Key: (B D)
  - BLPOP, BRPOP
    - Ex. Key: ()
    - `LPOP <key>`
      - No Data
    - `BLPOP <key>`
      - 데이터 push되기 전까지 대기

- Set

  - `SADD <key> <value>`
    - Value가 이미 Key에 있으면 추가 안됨
  - `SMEMBERS <Key>`
    - 모든 Value를 돌려줌
    - O(N) 이기 때문에 조심해야됨
  - `SISMEMBER <key> <value>`
    - Value가 존재하면 1, 없으면 0
  - Ex) 특정 유저를 Follow하는 목록을 저장할 때

- SortedSet

  - Set과의 차이점 = score를 주어서 정렬할 수 있음
  - 랭킹에 따라 순서가 바뀜
  - `ZADD <key> <score> <value>`
    - Value가 이미 Key에 있으면 해당 score로 변경
  - `ZRANGE <key> <startIndex> <endIndex>`
    - index 범위 안에 값을 가져옴
    - `ZRANGE Example:Key 0 -1` 모든 범위 가져오기
  - score는 int가 아니라 double이다 -> 값이 정확하지 않을 수 있다.
    - 실수로 정확히 표현할 수 없는 숫자들이 존재한다 0.00000000001 => 0.00000000이 되어버릴 수 있음

  - zrevrange
    - Descending 처리

  - zrangebyscore
    - Score 기준으로 뽑기
  - Ex1) zrange rank 50 70
    - SELECT * FROM rank ORDER BY score LIMIT 50, 20;
  - Ex2) zrevrange
    - SELECT * FROM rank ORDER BY score DESC LIMIT 50, 20;
  - EX3) zrangebyscore rank 70 100
    - SELECT * FROM rank WHERE 70 <= score AND score <= 100;
  - EX4) zrangebyscore rank (70 +inf
    - SELECT * FROM rank WHERE 70 < score 
    - 해당 값을 제외하려면 앞에 (를 붙임

- Hash: Key 밑에 sub key가 존재

  - 기본 사용법
    - `Hmset <key> <subkey1> <value1> <subkey2> <value2>`
    - `Hgetall <key>`
      - key에 해당하는 모든 subkey와 value를 가져옴
    - `Hget <key> <subkey>`
    - `Hmget <key> <subkey1> <subkey2> ... <subKeyN>`
    - Ex) `hmset johndoe name John email john.doe@example.com`
      - `INSERT INTO users(name, email) VALUES ('John', 'john.doe@example.com');`

- Collection 주의 사항
  - 하나의 컬렉션에 너무 많은 아이템을 담으면 좋지 않음
    - 10000개 이하 몇천개 수준으로 유지하는게 좋음
  - Expire는 Collection의 item 개별로 걸리지 않고 전체 Collection에 대해서만 걸림

## IV. Redis 운영

- 메모리 관리를 잘하자

  - Physical Memory 이상을 사용하면 문제가 발생

    - Swap이 있다면 Swap 사용으로 해당 Memory Page 접근시 마다 늦어짐
    - Swqp이 없다면?
- Maxmemory를 설정하더라도 이보다 더 사용할 가능성이 큼
  
  - Redis는 jemalloc을 Memory Allocator로 사용하고 있다
    - Paging으로 인해 메모리 파편화
      - 1 Byte를 요청하면 실제 1Byte만 사용하는 것이 아니다. 더 많은 양을 할당해줌
      - 1Byte를 3번 요청하면 3Byte가 아니라 12Byte를 사용하게될 수도 있음
      - 자기가 아는 memory 사용량보다 더 많이 씀
- 따라서, RSS 값을 모니터링 해야함
  - 많은 업체가 현재 메모리를 사용해서 swap이 일어나고 있는것을 모르고 있음
- 큰 메모리를 사용하는 instance 하나보다는 적은 메모리를 사용하는 instance 여러개가 안전함
  - CPU 4 Core, 32GB Memory 의 경우 8GB instance 3대를 갖도록 하는게 좋음
- 특정 액션들 (Migration 등등) 수행할 경우 현재 할당된 것보다 메모리를 훨씬 많이 사용할 수 있음
  - 4.x 대부터 메모리 파편화를 줄이도록 jemalloc에 힌트를 준다. 하지만 무조건 믿을 순 없음
  - 3.x대 버전의 경우 실제 used memory는 2GB라고 하지만 실제로 11GB의 RSS를 사용하고 있는 경우를 자주 본다
  - 다양한 사이즈를 가지는 데이터보다 유사한 사이즈를 가진 데이터를 바꾸는게 파편화가 덜 일어남
- 메모리가 부족할 때?
  
  - Cache is cash!!
      - 좀 더 메모리 많은 장비로 Migration
    - 메모리가 빡빡하면 Migration 중에 문제가 발생
    - 불필요한 data 없애기
      - 확인해서 없애기
      - 이미 swap을 사용중이라면 Process를 재시작해야함
  - 기본적으로 Colleciton들은 다음과 같은 자료구조를 사용
    - Hash -> HashTable을 사용 
	  - Sorted Set -> Skiplist와 HashTable을 이용
  	- Set -> HashTable 사용
	  - 해당 자료구조들은 메모리를 많이 사용
	- Ziplist를 이용하자 
  - 속도를 좀 더 느리지만 메모리 사용량이 적음
  - In-Memory 특성상, 적은 개수라면 선형 탐색을 하더라도 빠르다
      - 하지만 데이터가 엄청 커지면 다시 위에 Collection을 사용해야함
  - List, Hash, sorted set 등을 ziplist로 대체해서 처리하는 설정이 존재
  
    - Hash-max-ziplist-entries, hash-max-ziplist-value
  
- O(N) 관련 명령어는 주의하자
  - Redis 는 Single Threaded
    - 그러면 Redis가 동시에 여러 개의 명령을 처리할 수 있을까? No, 1개만 가능
      - 단순한 get/set의 경우, 초당 10만 TPS 이상 처리 가능
        - CPU 속도에 영향을 받음 => Single Core 성능이 중요
    - Packet으로 하나의 Command가 완성되면 processCommand에서 실제로 실행됨
      - ProcessInputBuffer 내부에 processCommand가 packet을 합쳐서 처리
    - 10만 TPS의 경우 하나가 1초 걸리면 99999개가 Timeout 발생함
      - 하나의 긴 명령을 수행하면 망한다!
  - 쓰면 안되는 명령어들 O(N)
    - KEYS
    - FLUSHALL, FLUSHDB
    - Delete Collections
    - Get All Collections
  - 대표적인 실수 사례
    - Key가 백만개 이상인데 확인하기 위해 KEYS 명령을 사용하는 경우
      - 모니터링 스크립트가 1초에 한번씩 KEYS를 호출하는 경우도 있음
    - 아이템이 몇만개 든 hash, sorted set, set에서 모든 데이터를 가져오는 경우
    - 예전의 Spring Security Oauth Redis TokenStore
      - Access Token의 저장을 List(O(N)) 자료구조를 통해서 이루어짐
        - 검색, 삭제 시에 모든 item을 매번 찾아봐야 함
        - 100만개 이상 데이터가 쌓일 경우 속도가 느려짐
        - 현재는 Set(O(1))으로 바꿔서 처리하도록 함
      - 최신 버전은 괜찮다
        - 강대명님이 고쳐서 커밋 올리셨다는 ....
  - KEYS 대체하기
    - scan 명령어로 하나의 긴 명령어 대신 여러개로 나눠서 처리할 수 있음
    - EX) scan 0, scan 17
  - Collection의 모든 item을 가져와야할 때?
    - Collection의 일부만 가져오거나
      - Sorted Set
    - Collection을 나눠서 작은 단위로 저장
      - UserRanks -> UserRank1, UserRank2, UserRank3
      - 하나당 n천개 안쪽으로 저장하는게 좋음 
- Replication
  - Async Replication
    - Replication Lag이 발생할 수 있음
      - A에 데이터가 쓰여지고 B에 동기화 되기전에 읽는 경우 아직 없을 수 있음 
    - 'Relicaof'(>=5.0.0) or 'slaveof' 명령으로 설정 가능
      - Replicaof hostname port
    - DBMS로 보면 statement replication과 유사
  - 설정 과정
    - Secondary에 replicaof 명령을 전달
    - Secondary는 Primary에 sync 명령 전달
    - Primary는 현재 메모리 상태를 저장하기 위해 Fork
      - Fork과정에서 메모리 부족이 일어날 수 있음
    - Fork한 프로세서는 현재 메모리 정보를 disk에 dump
    - 해당 정보를 secondary에 전달
    - Fork 이후의 데이터를 secondary에 계속 전달
  - Redis-cli--rdb 명령은 현재 상태의 메모리 스냅샷을 가져오므로 같은 부하를 줌
  	- AWS나 Google Cloud의 경우 Redis를 다른 방식으로 구현해서 좀 더 안정적
  - 많은 대수의 Redis 서버가 Replica를 가지고 있는 경우
    - Nerwork 이슈나 사람의 작업으로 동시에 replication이 재시도 되도록하면 문제가 발생할 수 있음
      - Ex) 같은 Network안에서 30GB를 쓰는 Redis Master 100대 정도가 Replication을 동시에 재시작하면?
      - 네트워크 사용량이 급증됨. 대역폭을 버티지 못하는 경우 => 서버가 계속 Down & 재시작 반복
- redis.conf 권장설정
  - Maxclient 설정 50000
  - RDB/AOF 설정 off
    - 마스터는 무조건 꺼놓고 Replica만 필요하면 Secondary로 켜놓는다
  - 특정 commands disable
    - KEYS
    - AWS의 ElasticCache는 이미 하고 있음
  - 전체 장애의 90% 이상이 KEYS와 SAVE 설정을 사용해서 발생
  - 적절한 ziplist 설정

## V. Redis 데이터 분산

- 데이터의 특성에 따라서 선택할 수 있는 방법이 달라진다
  - Cache일 때 우아한 Redis
  - Persistent 해야하면 Open the Hellgate
- 분산 방법
  - Application
    - Consistent Hashing
      - twemproxy를 사용하는 방법으로 쉽게 사용 가능
      - 기존 Modular 방식
        - Hash % (서버 개수) 에서 나오는 인덱스에 있는 서버에 저장
        - 서버가 추가되거나 사라지면? 기존 서버에 있던 데이터들을 새롭게 Modular 계산해서 재분산 시켜야함
      - Hash Function을 이용해서 분배
        - Data의 Hash 값보다 가장 가까운 높은 Hash 값을 가진 서버에 저장. 서버가 가진 값보다 높은 Hash 값을 가진 데이터가 존재할 경우 첫번째 서버에 저장
        - N / K 만큼의 데이터만 재분산하면됨
          - N: Data size
          - K: Servers
      - 서버 주소를 Hashing하는 것보다 Nickname으로 해싱하는 것이 변동이 더 적음
    - Sharding (데이터를 어떻게 나눌 것인가? <=> 데이터를 어떻게 찾을 것인가?)
      - 하나의 data를 모든 서버에서 찾아야하면?
      - 상황마다 sharding 전략이 달라짐
      - Range
        - 그냥 특정 Range를 정의하고 해당 Range에 속하면 거기에 저장
        - EX) 1 ~ 1000 / 1001 ~ 2000 / 2001 ~ 3000 인 경우 500은 어디에 저장되어야 할까?
        - 한 Range에 데이터가 몰리는 경우 서버 상황에 따라 놀고 있는 서버와 일하고 있는 서버의 차이가 심해진다
      - Modular
        - N % K로 서버의 데이터를 결정
        - Range보다 균등하게 분배됨
        - 서버가 2배씩 늘어나는 경우 재분배해야할 데이터 개수를 예상할 수 있음
      - Indexed
        - Index서버를 따로 두고 어디로 가야할지 알려줌
          - 모든 정보를 index 서버가 관리하기 때문에 index서버가 죽는 경우 서비스 작동 불가
  - Redis Cluster
    - Hash 기반으로 Slot 16384로 구분
      - Hash 알고리즘 CRC 164을 사용
      - Slot = crc16(key) % 16384
      - Key가 Key{hashkey} 패턴이면 실제 crc16에 hashkey가 사용된다
      - 특정 Redis 서버는 이 slot range를 가지고 있고, 데이터 migration은 이 slot 단위의 데이터를 다른 서버로 전달하게 된다. (migrateCommand 이용)
    - 동작 방식
      - 여러 대의 Primary , Secondary로 이루어짐
      - Primary가 죽으면 Secondary가 Primary 됨
    - 장점
      - 자체적인 Primary, Secondary Failover
      - Slot 단위의 데이터 관리
    - 단점
      - 메모리 사용량이 더 많음
      - Migration 자체는 관리자가 시점을 결정해야 함
      - Library 구현이 필요함

## VI. Redis Failover

- Coordinator 기반
  - Zookeeper, etcd, consul 등의 Coordinator 사용
  - 동일한 방식으로 관리가 가능
  - Redis #1과 
- VIP/DNS 기반
  - 클라이언트에 추가적인 구현이 필요없음
  - VIP 기반은 외부로 서비스를 제공해야 하는 서비스 업체에 유리
  - DNS Cache TTL을 관리해야 함
    - 사용하는 언어별 DNS 캐싱 정책을 잘 알아야함
    - 툴에 따라서 한번 가져온 DNS 정보를 다시 호출하지 않는 경우도 존재
- Monitoring
  - Factor
    - Redis Info를 통한 정보
      - RSS - 실제 피지컬 메모리를 얼마나 쓰고 있느냐
      - Used Memory
      - Connection 수
        - Connection을 Transaction마다 생성 끊는 경우 single thread 기반이기 때문에 퍼포먼스가 느려짐
        - DB 작업 말고 연결에도 꽤 많은 부하가 있음
      - 초당 처리 요청 수
    - System
      - CPU
      - Disk
      - Network rx / tx
  - CPU가 100%를 칠 경우
    - 처리량이 많다면
      - 좀 더 좋은 CPU 성능을 가진 서버로 이전
      - 실제 CPU 성능에 영향을 많이 받음 Single thread니까 하나의 core 성능 좋은거 쓰면됨
    - O(N) 계열의 특정 명령이 많은 경우
      - Monitor 명령을 통해 특정 패턴을 파악하는것이 필요
      - Monitor 잘못쓰면 부하로 해당 서버에 더 큰 문제를 일으킬 수 있음 (짧게 쓰는게 좋음)

## VII. 결론

- 기본적으로 Redis는 매우 좋은 툴
- 하지만, 메모리 장비를 빡빡하게 쓸 경우 성능이 느려짐
- Redis as Cache
  - Cache 일 경우 문제가 적게 발생
    - Redis 
    - Consistent Hashing도 실제 부하를 아주 균등하게 나누지는 않음 => Adaptive Consistent Hashing 을 고려해볼 수 있음
- Redis as Persistent Store
  - Persistent Store의 경우
    - Cache로 쓸때보다 훨씬 더 많은 공수가 필요하다
    - 무조건 Primary / Secondary 구조로 구성이 필요함
    - 메모리를 절대로 빡빡하게 사용하면 안됨
      - 정기적인 migration이 필요
      - 가능하면 자동화 툴을 만들어서 이용
    - RDB / AOF가 필요하다면 Secondary에서만 구동



## VII. 다루지 않는 것

- Redis Persistence
- Redis Pub/Sub
- Redis Stream
- 확률적 자료구조
  - Hyperlog
- Redis Module







