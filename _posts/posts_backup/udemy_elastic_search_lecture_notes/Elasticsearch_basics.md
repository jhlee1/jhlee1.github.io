# Elasticsearch Basics: Logical Concepts

## 1. documents

- Documents are the things you are searching for
- They can be more than text - any structured JSON data works
- Every document has a unique ID, and a type
  - You can set the ID or let the Elasticsearch set it for you

## 2. Indices

- Highest level entity you can query against an Elasticsearch
- An index powers search into all documents into a collection of types
- They contain 
  - **inverted indices** that let you search across everything within them at once
  - **mappings** that define schemas for the data within
- The schema belongs to the index

## 3. inverted index

- It stores splitted words with which document it is in 

Ex)

### Document 1:

Space: The final frontier. There are the voyages

### Document2:

He's bad, he's number one. He's the space cowboy with the laser gun

### Inverted index

| Lowercased & Splitted word | Index |
| :------------------------- | ----- |
| space                      | 1, 2  |
| the                        | 1, 2  |
| final                      | 1     |
| frontier                   | 1     |
| he                         | 2     |
| ...                        | ...   |



## 4. TF-IDF (Term Frequency Times Inverse Document Frequency)

- It is an indicator for relevance
- It means Term Frequency * Inverse Document Frequency
- **Term Frequency** - How often a term appears in a given document
- Document Frequency - How oftern a term appears in all documents
- Term Frequency / Document Frequency measures the relevance of a term in document

ex) 

- **"the"** is a pretty common word so it will has low relevance
- **"space"** will have much higher rank

## 5. Using indeces

- There are three ways to use 
  - RESTful API
  - Client API's - libraries in each language 
  - Analytic Tools - Kibana

## 6. What's new in Elasticseach 7?

- The concept of document types is deprecated
  - In API call, we use "_doc" or skip it 
- Elasticsearch SQL is "production ready"
  - SQL seems Lingua franca that's tying together all sorts of big data technologies
- Lots of changes to defaults
  - default shards is 1 instead of 5
  - How replication works in production setting
- Lucene 8 under the hood
- Several X-pack plugins now included with Elasticsearch itself
- Bundled Java runtime
  - You can run Elasticsearch without external JDK
  - However, Kibana still requires external JDK
- Cross-cluster replication is "production ready"
  - Replication across cluster is now possible in Elasticsearch7
- Index Lifecycle Management
- High-level REST client in Java(HLRC)
- Performace improvements
- (countless little breaking changes)[https://www.elastic.co/guide/en/elasticsearch/reference/7.0/breaking-changes-7.0.html]

## 7. How elasticsearch scales (How to work in a cluster)

- Documents are hashed to a particular shards
- Every shard is a self-contained instance of Lucene index of its own
- Each shard may be on a different node in a cluster
- When you search a document, it quickly finds which shard it has and it directs you to the shard
- You can easily add more machines to the cluster

## 8. Primary and replica shards

- For resilience to the failure

EX) This index has two primary shards and two replicas. Your application should round-robin requests among nodes

- **Write** requests are routed to the primary shard, then replicated
- **Read** requests are routed to the primary or any replica

| Node 1                  | Node 2                  | Node 3                  |
| ----------------------- | ----------------------- | ----------------------- |
| (Primary 1) (Replica 0) | (Replica 0) (Replica 1) | (Primary 0) (Replica 1) |

- The number of primary shards cannot be changed later
  - Not as bad as it sounds - you can add more replica shards for more read throughput
  - Worst case you can re-index your data
  - The number of shards can be set up front via a PUT command via REST / HTTP

```
PUT /testindex
{
	"settings": {
		"number_of_shards": 3,
		"number_of_replicas": 1
	}
}
```

- This will create 6 shards
  - 3 shards for primary and 1 replica for each primary shard