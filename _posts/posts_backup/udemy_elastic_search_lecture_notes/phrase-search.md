# Phrase Search

## 1. Concepts
- must find all terms, in the right order

```bash
$ curl -XGET localhost:9200/movies/_search?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"title" : "star wars"
		}
	}
}
'
```

## 2. Slop
- Order matters, but you're OK with some words being in between the terms:
- The **slop** represents how far you are willing to let a term move to satisfy a phrase (in either direction!)
- ex) "quick brown fox" would match "quick fox" with a slop 1
- ex) "star beyond" would match "beyond star" with a slop 1

```bash
$ curl -XGET localhost:9200/movies/_search?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"title" : {"query" : "star beyond", "slop" : 1}
		}
	}
}
'
```

## 3. proximity query
- Remember this is a query - results are sorted by relevance
- Just use a really high slop if you want to get any documents that contain the words in your phrase, but want documents that have the words closer together scored higher

```bash
$ curl -XGET localhost:9200/movies/_search?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"title" : {"query" : "star beyond", "slop" : 100}
		}
	}
}
'
```

## 4. Examples
- match vs. match_phrase

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty -d '
{
	"query" : {
		"match" : {
			"title" : "star wars"
		}
	}
}
'
# It returns two movies titled "star wars" and "star trek" because both matches "star"
```

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"title" : "star wars"
		}
	}
}
'
# It returns only the movie titled "star wars"
```

- slop

```bash
$ curl -XGET -H "Content-Type: application/json" localhost:9200/movies/_search\?pretty -d '
{
	"query" : {
		"match_phrase" : {
			"title" : {"query": "star beyond", "slop" : 1}
		}
	}
}
'
# It returns only the movie titled "star trek beyond"
```
