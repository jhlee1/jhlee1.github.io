# "Query Lite" interface

## 1. query-line search

- You can create query on url instead of creating json body

Ex) It's searching a document that its title contains "star" in movies

```
/movies/_search?q=title:star
```

Ex) It searches a movie, which its title contains "trek" and year is greater than **2010**

```
/movies/_search?q=+year:>2010+title:trek
```

- However, you need to encode the URL on the browser
  - It's cryptic and tough to debug
- can be a security issue if exposed to end users
- Fragile - one wrong character and you're hosed

```
/movies/_search?q=+year>2010+title:trek -> /movies/_search?q=%2Byear%3A%3E2010+%2Btitle%3Atrek
```

Real Ex)

```bash
$ curl -H "Content-Type: application/json" -XGET localhost:9200/movies/_search\?q=title:star\&pretty
```

```bash
$ curl -H "Content-Type: application/json" -XGET localhost:9200/movies/_search\?q=+year\>2010+title:trek\&pretty
```



## 2. Learn more

- It's commonly called "URI search"
- You should not use this for production. It is supposed to be used only for experimental purpose
