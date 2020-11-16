# Dealing with Concurrency

## 1. The problem

- When two clients read the data from Elasticseach and update the data, it works weird

EX) It should be **12**. When two users try to update the page view count, it only updates for one request.

Get view count for page -> 10 -> Increment view count for page -> 11

Get view count for page -> 10 -> Increment view count for page -> 11



## 2. Optimistic Concurrency Control

- It uses Sequence number and Primary term to solve this problem
- In the same situation, the second one fails because the value for the sequence number has been changed to 10, not 9 anymore

Get view count for page -> 10 (_seq_no: 9, _primary_term: 1) -> Increment view count for page  (_seq_no: 9, _primary_term: 1) -> 11

Get view count for page -> 10 (_seq_no: 9, _primary_term: 1) -> Increment view count for page  (_seq_no: 9, _primary_term: 1) -> Error! Try again

- The second user will request and get a new document information with new sequence number and update to 12
  - You don't have to do it manually
  - Use retry_on_conflicts = N to automatically retry

## 3. Real Example

1. Get the document
   1. Check sequence number and primary term

```bash
curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_doc/109487\?pretty

{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "109487",
  "_version" : 5,
  "_seq_no" : 9,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "genre" : [
      "IMAX",
      "Sci-Fi"
    ],
    "title" : "Interstellar 3",
    "year" : 2014
  }
}
```

2. Update the document specifying sequence number that we found above
   1. It should work well

```bash
$ curl -XPUT -H "Content-Type: application/json" localhost:9200/movies/_doc/109487\?if_seq_no=9\&if_primary_term=1 -d '
{
	"genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar 3",
  "year": 2014
}'

{"_index":"movies","_type":"_doc","_id":"109487","_version":6,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":10,"_primary_term":1}%


curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_doc/109487\?pretty
```

3. Update with the same sequence number
   1. It should conflict with sequence number 9 which we used above

```bash
$ curl -XPUT -H "Content-Type: application/json" localhost:9200/movies/_doc/109487\?if_seq_no=9\&if_primary_term=1 -d '
{
	"genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar 3",
  "year": 2014
}'

{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[109487]: version conflict, required seqNo [9], primary term [1]. current document has seqNo [10] and primary term [1]",
        "index_uuid": "UE9kbO2JSfCUvX_-gRbgKA",
        "shard": "0",
        "index": "movies"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[109487]: version conflict, required seqNo [9], primary term [1]. current document has seqNo [10] and primary term [1]",
    "index_uuid": "UE9kbO2JSfCUvX_-gRbgKA",
    "shard": "0",
    "index": "movies"
  },
  "status": 409
}
```

## 4. Use retry on conflicts

- It automatically retry on conflicts up to 5 times

```bash
$ curl -XPOST -H "Content-Type: application/json" localhost:9200/movies/_doc/109487/_update\?retry_on_conflict=5 -d '
{
	"doc" : {
		"title" : "Interstellar"
	}
}'
```
