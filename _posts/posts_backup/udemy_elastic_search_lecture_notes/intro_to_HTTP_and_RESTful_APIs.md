# Intro to HTTP and RESTful API's

## 1. Anatomy of a HTTP request

- METHOD: the "verb" of the request. GET, POST, PUT, or DELETE
- PROTOCOL: what flavor of HTTP (HTTP/1.1)
- HOST: what web server you want to talk to (ex. google.com)
- URL: what resource is being requested (ex. path to the resource on the page /something/else)
- BODY: extra data needed by the server
- HEADERS: user-agent, content-type, etc

## 2. RESTful API's

- pragmatic definition: using HTTP requests to communicate with web services
  - examples:
    - GET requests retrieve information (like search results)
    - PUT requests insert or replace new information
    - DELETE requets delete information
- fancy speak: Representational State Transfer
  - 6 guiding constraints
    - Client-server architecture
    - Statelessness
      - All required information should be on the request rather than the sever keeps track of the state of different requests
    - Cacheability
    - Layered system
    - Code on demand(ex. sending Javascript)
    - Uniform interface

## 3. Why REST?

- It's language and system independent
- What language the client uses does not matter
  - It's more important to understand how to send HTTP Request and the query rather than how to use language specific client.

## 4. The curl Command

- A way to issue HTTP requests from the command line
- From code, you will use whatever library you use for HTTP / REST in the same way

```bash
curl -H "Content-Type:application/json" <URL> -d '<BODY>'
```

Ex)

```-H &quot;Content-Type:application/json&quot;
$ curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '{"query" : {"match_phrase" : {"text_entry" : "to be or not to be"}}}'
```

