# Import a single movie via JSON / REST



## 1. Insert a single record

- Apply mapping first

```bash
$ curl -XPUT -H "Content-Type: application/json" localhost:9200/movies -d '
{
	"mappings" : {
		"properties" : {
			"year" : {
				"type" : "date"
			}
		}
	}
}
'

{"acknowledged":true,"shards_acknowledged":true,"index":"movies"}
```

- Check if the mapping is applied

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_mapping
```

- Now add a  document

```bash
$ curl -XPOST -H "Content-Type: application/json" localhost:9200/movies/_doc/109487 -d '
{
	"genre": ["IMAX", "Sci-Fi"],
	"title": "Interstellar",
	"year": 2014
}
'

{"_index":"movies","_type":"_doc","_id":"109487","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```

- Check if the document is added correctly

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty
```



## 2. How to skip putting  Header

- Stop adding header when your send the request
  - Let the curl do this for you
- Create a curl file in your user bin directory

```bash
$ cd ~
$ mkdir bin
$ vim curl
```

- In the curl file, add the following

```bash
#! /bin/bash
/usr/bin/curl -H "Content-Type: application/json" "$@"
```

- activate curl for the session

```bash
$ chmod a+x ~/bin/curl
$ source ~/.profile
$ which curl
/home/student/bin/curl
```



## 3. Import many documents

- It's not a huge whole chunk of a single json
  - Each record consists of two lines
  - Elasticseach reads line by line because it needs to figure out which shards to go
    - Read the first line and pass the second line to the right shard

```bash
curl -XPUT -H "Content-Type: application/json" 127.0.0.1:9200/_bulk -d '
{"create" : {"_index" : "movies", "_id" : "135569"} }
{"id" : "135569, "title" : "Start Trek Beyond", "year" : 2016, "genre" : ["Action", "Adventure", "Sci-Fi"] }
{"create" : {"_index" : "movies", "_id" : "122886"} }
{"id" : "122886, "title" : "Star Wars: Episode VII - The Force Awakens", "year" : 2015, "genre" : ["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
```

- Let's apply data using json file

  - Download the json file

  ```bash
  $ wget http://media.sundog-soft.com/es7/movies.json
  ```

  - check if you got the right file

  ```bash
  $ cat movies.json
  ```

  - Apply the file
    - It works out except an error saying the document that I just created above is conflicting

  ```bash
  $ curl -XPUT -H "Content-Type: application/json" localhost:9200/_bulk\?pretty --data-binary @movies.json
  
  {
    "took": 47,
    "errors": true,
    "items": [
      {
        "create": {
          "_index": "movies",
          "_type": "_doc",
          "_id": "135569",
          "_version": 1,
          "result": "created",
          "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
          },
          "_seq_no": 1,
          "_primary_term": 1,
          "status": 201
        }
      },
      {
        "create": {
          "_index": "movies",
          "_type": "_doc",
          "_id": "122886",
          "_version": 1,
          "result": "created",
          "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
          },
          "_seq_no": 2,
          "_primary_term": 1,
          "status": 201
        }
      },
      {
        "create": {
          "_index": "movies",
          "_type": "_doc",
          "_id": "109487",
          "status": 409,
          "error": {
            "type": "version_conflict_engine_exception",
            "reason": "[109487]: version conflict, document already exists (current version [1])",
            "index_uuid": "UE9kbO2JSfCUvX_-gRbgKA",
            "shard": "0",
            "index": "movies"
          }
        }
      },
      {
        "create": {
          "_index": "movies",
          "_type": "_doc",
          "_id": "58559",
          "_version": 1,
          "result": "created",
          "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
          },
          "_seq_no": 3,
          "_primary_term": 1,
          "status": 201
        }
      },
      {
        "create": {
          "_index": "movies",
          "_type": "_doc",
          "_id": "1924",
          "_version": 1,
          "result": "created",
          "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
          },
          "_seq_no": 4,
          "_primary_term": 1,
          "status": 201
        }
      }
    ]
  }
  ```

  - Check if all those documents are created correctly

  ```bash
  $ curl -XGET -H "Content-Type: application/json" localhost:9200/_search\?pretty
  ```

## 4. Update Documents

- Elasticsearch documents are immutable
  - You can't actually edit
- Every document has a _version field
- When you update an existing document
  - A new document is created with an increment _versioin
  - The old document is marked for deletion
    -  Elasticsearch will actually delete it later when it needs to do so
- Ex) You are updating the document as if you create a new one
  - You can see the version is changed

```bash
$ curl -XPUT -H "Content-Type: application/json" localhost:9200/movies/_doc/109487\?pretty -d '
{
	"genres" : ["IMAX", "Sci-Fi"],
	"title" : "Interstellar",
	"year" : 2014
}'

{
        "genres" : ["IMAX", "Sci-Fi"],
        "title" : "Interstellar",
        "year" : 2014
}'
{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "109487",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}

#Check if the version has been changed
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_doc/109487\?pretty

{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "109487",
  "_version" : 2,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "genres" : [
      "IMAX",
      "Sci-Fi"
    ],
    "title" : "Interstellar",
    "year" : 2014
  }
}	
```

- Ex 2) Partial update - Updating only title

```bash

$ curl -XPOST -H "Content-Type: application/json" localhost:9200/movies/_doc/109487/_update -d '
{
	"doc" : {
		"title" : "Interstellar2"
	}
}'

{
	"_index":"movies",
	"_type":"_doc",
	"_id":"109487",
	"_version":3, #The version is updated!
	"result":"updated",
	"_shards":{
		"total":2,
		"successful":1,
		"failed":0
	},
	"_seq_no":6,
	"_primary_term":1
}
```

## 5. Delete documents

- It's simple. Let's delete Dark Knight from the movies
  - Search Dark because I don't know its id

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?q=Dark\&pretty
{
  "took" : 2,
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
    "max_score" : 1.5169398,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "58559",
        "_score" : 1.5169398,
        "_source" : {
          "id" : "58559",
          "title" : "Dark Knight, The",
          "year" : 2008,
          "genre" : [
            "Action",
            "Crime",
            "Drama",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

- Delete the document Dark Knight

```bash
$ curl -XDELETE localhost:9200/movies/_doc/58559

{"_index":"movies","_type":"_doc","_id":"58559","_version":2,"result":"deleted","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":7,"_primary_term":1}
```

- Check if it still exits

```bash
{
  "took" : 58,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

