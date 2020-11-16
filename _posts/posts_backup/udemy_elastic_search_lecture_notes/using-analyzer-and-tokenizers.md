# Using Analyzers and Tokenizers

## 1. Controlling full-text search

- Sometimes text fields should be exact-match
  - Use keyword mapping instead of text

- Or, search on analyzed text fields will return anyting remotely relevant
  - Depending on the analyzer, results will be case-insensitive, stemmed, stopwords removed, synonyms applied, etc...
  - Searches with multiple terms needed not match them all

EX) Search with "Star Trek"

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty -d '
{
	"query" : {
		"match" : {
			"title" : "Star Trek"
		}
	}
}
'
```

- Documents containing "star" is searched because it is partially matched
  - As you can see "Star Wars" has lower score

```bash
{
  "took" : 906,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.8908832,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "135569",
        "_score" : 2.8908832,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "122886",
        "_score" : 0.7743672,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

EX) Partial search with "genre" with match_phrase

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"genre" : "sci"
		}
	}
}
'
```

- It shows all documents having "genre" : "Sci-Fi"
  - The "genre" field is analyzed & normalized
    - case-insensitive, distinct
    - Split by dash into two distinct search terms which are independently searched 

## 2. I want to make my search more tight (exact-match)

- Delete index

```bash
$ curl -XDELETE -H "Content-Type: application/json" localhost:9200/movies
```

- Create a new mapping
  - I am setting "genre" as "keyword" It means the genre will not accept partial match

```bash
$ curl -XPUT -H "Content-Type: application/json" localhost:9200/movies -d '
{
	"mappings" : {
		"properties" : {
			 "id" : {"type" : "integer"},
			 "year" : {"type" : "date"},
			 "genre" : {"type" : "keyword"},
			 "title" : {"type" : "text", "analyzer" : "english"}
		}
	}
}'
```

- Reindex the data with the new mapping

```bash
$ curl -XPUT -H "Content-Type: application/json" localhost:9200/_bulk\?pretty --data-binary @movies.json
```

- Search again for genre
  - It has no hit
  - Searching for "sci-fi" does not work either
  - To search with **keyword**, It matches only with  **"Sci-Fi"**

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"genre" : "sci"
		}
	}
}
'
```

