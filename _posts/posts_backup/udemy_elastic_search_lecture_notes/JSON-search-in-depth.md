# JSON Search In-Depth

## 1. Request Body Search

- query DSL is in the request body as JSON
  - a GET request can have a body. It's not usual in Web Request but it's not illegal

```
$ curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
	"query" : {
		"match" : {
			"title" : "star"
		}
	}
}'
```

## 2. Queries and Filters

- filters - ask a yes / no question of your data
- queries - return data in terms of relevance
- Use filters when you can - they are faster and cacheable

Ex) boolean query with a filter

```bash
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
	"query" : {
		"bool" : {
			"must" : {
				"term" : {
					"title" : "trek"
				}
			},
			"filter" : {
				"year" : {
					"gte" : 2010
				}
			}
		}
	}
}
'
```

## 3. Some types of filters

- term: filter by exact values

  ```json
  {"term" : {"year" : 2014}}
  ```

- terms: match if any exact values in a list match

  ```json
  {"terms" : {"genre" : ["Sci-Fi", "Adventure"]}}
  ```

- range: find numbers or dates in a given range (gt, gte, lt, lte)

  ```json
  {"range" : {"year" : {"gte" : 2010}}}
  ```

- exists: Find documents where a field exits

  ```json
  {"exits" : {"field" : "tags"}}
  ```

- bool: Combine filters with Boolean logic (must, must_not, should)

  - must - AND
  - must_not - NOT
  - should - OR

## 4. Some types of queries

- match_all: return all documents and is the default. Normally used with a filter

  ```bash
  {"match_all": {}}
  ```

- match: searches analyzed results, such as full text search

  ```json 
  {"match" : {"title" : "star"}}
  ```

- multi_match: run the same quey on multiple fields

  ```json
  {"multi_match" : {"query" : "star", "fields" : ["title", "synopsis"]}}
  ```

- bool: Works like a bool filter, but results are scored by relevance

## 5. Syntax Reminder

- queries are wrapped in a "query": {} block
- filters are wrapped in a "filter": {} block
- You can combine filters inside queries, or queries inside filters too
- Check out the first example