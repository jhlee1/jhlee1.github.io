# Set up Elasticsearch

## 1. Linux VM으로 Elasticsearch 설치

- The lecture recommends to install Elasticsearch on Linux Server

  - Download Oracle VM VitualBox 
  - Download ubuntu server image 
  - Install Ubuntu on the Virtual Box
  - Windows에서 VM을 사용해서 Ubuntu를 사용하는 경우
    - Hyper-V가 켜져있는지 확인
    - 백신 프로그램인 avast랑 같이 키면 충돌난다

- Elasticsearch 다운받기

- ```bash
  $ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
  $ sudo apt-get install apt-transport-https
  $ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
  $ sudo apt-get update && sudo apt-get install elasticsearch
  ```

- Set up Elasticsearch 

  - Enter the setting 

  ```
  $ sudo vi /etc/elasticsearch/elasticsearch.yml
  ```

  - Change the node name in the file

  ```
  node.name: node-1
  ```

  - Network

  ```
  network.host:0.0.0.0
  ```

  - Discovery

  ```
  discovery.seed_hosts: ["127.0.0.1"]
  cluster.initial_master_nodes: ["node-1"]
  ```

  - Apply the changes & Set it to boot up automatically when we start the OS

  ```
  $ sudo /bin/systemctl daemon-reload
  $ sudo /bin/systemctl enable elasticsearch.service
  $ sudo /bin/systemctl start elasticsearch.service
  ```

  

## 2. Install on Docker

- Create a network for ELK stack

- ```bash
  docker network create elknetwork
  ```

- Create and start the Elasticsearch docker image setting port for 9200, 9300 as well as setting up to use a single node

  - I am using 7.3.1 version (which is the lastest by 08/30/2019)
  - You can set up the configs with environment variables

```bash
docker run -d --name elasticsearch --net elknetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "node.name=node-1" elasticsearch:7.3.1
```



## 3. Check if it's running

- Send curl GET and check if it returns well

```bash
$ curl -XGET 127.0.0.1:9200
{
  "name" : "node-1",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "OLJygowASAGBSDmglE3bPg",
  "version" : {
    "number" : "7.3.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "4749ba6",
    "build_date" : "2019-08-19T20:19:25.651794Z",
    "build_snapshot" : false,
    "lucene_version" : "8.1.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



##  4. Download example schema (Shakespeare's literature)

- Download schema

```bash
$ wget http://media.sundog-soft.com/es7/shakes-mapping.json
```

- Submit the data type which is downloaded above

```bash
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json

{"acknowledged":true,"shards_acknowledged":true,"index":"shakespeare"}%
```

- Download the works of Shakespeare

```bash
$ wget http://media.sundog-soft.com/es7/shakespeare_7.0.json
```

- Make index of Elasticsearch with the schema that we created above

```bash
$ curl -H "Content-Type: application/json" -XPOST '127.0.0.1:9200/shakespeare/_bulk' --data-binary @shakespeare_7.0.json

...
"_shards":{"total":2,"successful":1,"failed":0},"_seq_no":111395,"_primary_term":1,"status":201}
```

- Search to see if it works well
  - I am sending a search query to the **index** called `shakespeare` to find a work containing "to be or not to be"

```bash
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '{"query" : {"match_phrase" : {"text_entry" : "to be or not to be"}}}'

{
  "took" : 35,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 13.889601,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_type" : "_doc",
        "_id" : "34229",
        "_score" : 13.889601,
        "_source" : {
          "type" : "line",
          "line_id" : 34230,
          "play_name" : "Hamlet",
          "speech_number" : 19,
          "line_number" : "3.1.64",
          "speaker" : "HAMLET",
          "text_entry" : "To be, or not to be: that is the question:"
        }
      }
    ]
  }
}
```



