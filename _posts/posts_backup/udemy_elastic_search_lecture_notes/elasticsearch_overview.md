# Elasticsearch Overview

## 1. Elasticsearch

- Started off as scalable Lucene
  - Lucene is an open source project
- Horizontally scalable search engine
- Each "shard" is an inverted index of documents
- But just for full text search
  - It's not just simply used for searching
  - Can handle structured data, and can aggregate data quickly
  - Often a faster solution than Hadoop/Spark/Flink/etc.

- It's a simply commuincate in json format

## 2. Kibana

- Web UI for searching and **visualizing**
- Complex aggregations, graphs, charts
- Oftern used for log analysis

## 3. Logstash / Beats

- Way to feed data into Elasticsearch in real time
- FileBeat can monitor log files, parse them and import into Elasticsearch in near-real-time
- Logstash also pushes data into Elasticsearch from many machines
- Not just log files
  - You can tie two different systems and publish data to another system
  - You can collect data from many systems (such as kafka, Amazon S3)

## 4. X-Pack

- Paid add-on
- Security
- Alerting
- Monitoring
- Reporting
- Machine Learning
- Graph Exploration
- Some part of it is free such as Monitoring cluster

