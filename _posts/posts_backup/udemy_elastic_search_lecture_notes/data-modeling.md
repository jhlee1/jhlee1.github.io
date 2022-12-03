# Data Modeling

## 1. Normalized Data

- you can minimize storage space
- make it easy to change titles
- However, it requires two queries ans storage is cheap

ex) When you want to get ratings of a movie, you have to look up Rating first and you also have to look up Movie to get its title 

Look up rating -> Rating (userID, movieID, rating) -> Look up title -> Movie (MovieID, title, genres)

- You can minimize storage. 
  - If you put them in a single entity, the fields in the movie will be duplicated a lot because ther will be way more rating records than movie records
- you don't have to use query twice
  - Currently you are sending query twice. One for Rating, Another for Movie

## 2. Denormalized Data

- titles are duplicated, but only one query
  - You have to copy movie title for every single rating. It wastes disks
  - To change the title, you have to update all ratings which would be a lot
  - However,
    - disks are cheap these days
    - Movie titles are not changed unless you made a typo when you first create the record

Ex) Rating has all of the information you need

Look up rating -> Rating(userID, rating, title)

## 3. Parent / Child Relationship

- You can create a relationship

Parent - Star Wards

Children - A New Hope, Empire Strikes back, Return of the Jedi, The Force Awakens

Ex)

1. Create a new index

```bash
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/series -d '
{
	"mappings" : {
		"properties" : {
			"film_to_franchise" : {
				"type" : "join",
				"relations" : {
					"franchise" : "film" 
				}
			}
		}
	}
}
' 
```

2. Download data and check it

```bash
$ wget http://media.sundog-soft.com/es7/series.json
$ less series.json
```

- The first one is franchise titled "Star Wars"
- The other ones are children of "Star Wars"
  - You can see `"parent": "1"`. The document is a child of a document having id `"1"`

```json
{ "create" : { "_index" : "series", "_id" : "1", "routing" : 1} }
{ "id": "1", "film_to_franchise": {"name": "franchise"}, "title" : "Star Wars" }
{ "create" : { "_index" : "series", "_id" : "260", "routing" : 1} }
{ "id": "260", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode IV - A New Hope", "year":"1977" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "1196", "routing" : 1} }
{ "id": "1196", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode V - The Empire Strikes Back", "year":"1980" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "1210", "routing" : 1} }
{ "id": "1210", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode VI - Return of the Jedi", "year":"1983" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "2628", "routing" : 1} }
{ "id": "2628", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode I - The Phantom Menace", "year":"1999" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series",  "_id" : "5378", "routing" : 1} }
{ "id": "5378", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode II - Attack of the Clones", "year":"2002" , "genre":["Action", "Adventure", "Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "series", "_id" : "33493", "routing" : 1} }
{ "id": "33493", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode III - Revenge of the Sith", "year":"2005" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "122886", "routing" : 1} }
{ "id": "122886", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode VII - The Force Awakens", "year":"2015" , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
```

3. Put the document into the index "series"

```bash
$ curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk\?pretty --data-binary @series.json 
```

4. search all films associated with Star Wars franchise

```bash
$ curl -H "Content-Type: application/json" -XGET localhost:9200/series/_search\?pretty -d '
{
	"query" : {
		"has_parent" : {
			"parent_type" : "franchise",
			"query" : {
				"match" : {
					"title" : "Star Wars"
				}
			}
		}
	}
}
'
```

5. Search by `"has_child"`

```bash
$ curl -H "Content-Type: application/json" -XGET localhost:9200/series/_search\?pretty -d '
{
	"query" : {
		"has_child" : {
			"type" : "film",
			"query" : {
				"match" : {
					"title" : "The Force Awakens"
				}
			}
		}
	}
}
'
```







