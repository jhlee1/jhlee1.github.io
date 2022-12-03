# Mapping & Indexing Data

## 1. Introduction to movie data set

- Movielens
  - a free dataset of movie ratings gathered from movielens.org
  - contians user ratings, movie metadata, and user metadata
	- How to download it?
		- Go to grouplens.org
			- has movielens data
			- free for resaerch purpose
    - Enter datasets category
    - Download <u>ml-latest-small.zip</u> under **recommended for education and development** section
      - Check README as well to see the information of data

## 2. Creating Mappings

- Mapping is a schema definitions
- Elasticsearch has reasonable defaults
  - It can figure out if you're trying to store strings or floats, integers
-  but sometimes you need to give it a little bit of a hint.
- Ex) we're going to import some movie lines data and we want the release date to be explicitly interpreted as a date type field.

```
curl -XPUT 127.0.0.1:9200/movies -d '
{
	"mappings" : { 
		"properties" : {
			"year" : {
				"type" : date"
			}
		}
	}
}'
```

- From Elasticsearch 6, you need to set header to show the body is json with  `-h "Content-Type:application/json"` when you send a request

## 3. Common Mappings

- Field types

  - sting, byte, short, integer, long, float, double, boolean, date

  Ex)

  ```json
  "properties": {
  	"user_id": {
  		"type": "long"
  	}
  }
  ```

- Field index

  - Do you want this field indexed for full-text search? analyzed / not_analyzed / no

  Ex) It is not searched for full-text search, but it will show up on the result

  ```json
  "properties": {
  	"genre": {
  		"index": "not_analyzed"
  	}
  }
  ```

- Field analyzer

  - define your tokenizer and token filter. standard / whitespace / simple /english etc.

  Ex)

  ```
  "properties": {
  	"description": {
  		"analyzer": "english"
  	}
  }
  ```

## 4. More about analyzer

- character filters
  - remove HTML encoding
  - convert & to and
- tokenizer
  - Define how to chomp stings into token
  - Several options by
    - whitespace
    - punctuation 
    - non-letters
- token filter
  - lowercasing
  - stemming - Let you search by the root word ex) boxing, boxes, boxed -> box
  - synonyms - Let you search by synonyms ex) happy & pleasure
  - stopwords - remove the words that you don't want to be indexed ex) and, or, the
    - It sometimes has side effects, so, be careful when you use it

## 5. choices for analyzer

- standard
  - Splits on word boundaries, remove punctuation, lowercase
  - Good choice if language is unknown
- simple
  - splits on anything that isn't a letter, and lowercases
- whitespace
  - splits on whitespace but does not lowercase
- language(ex. english)
  - accounts for language-specific stopwords and stemming
  - You can put different languaed data into one index by using different analyzers ex) Put English data with **english** analyzer & Put French Data with **french** analyzer

