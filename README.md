# elasticsearch-autocomplete

## Running with Docker Compose

To make it easier to set up and run Elasticsearch and Kibana, you can use Docker Compose.

### Requirements

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/)

### Steps

1. Clone this repository and navigate to the project directory.
2. Run `docker-compose up` to start Elasticsearch and Kibana.
3. Open [http://localhost:9200](http://localhost:9200) for Elasticsearch and [http://localhost:5601](http://localhost:5601) for Kibana.


## 1. Create the Index

Once Elasticsearch is running, create the index with the required settings. Run the following commands using the Dev Tools in Kibana or any API client like `curl` or `Postman`.

### Create the Index with Settings

PUT http://localhost:9200/autocomplete_index
```json
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        },
        "autocomplete_search_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "autocomplete_analyzer",
        "search_analyzer": "autocomplete_search_analyzer"
      }
    }
  }
}
```


## 2. Populate the Index
Add some documents to the index for testing purposes.

POST http://localhost:9200/autocomplete_index/_doc/1
```json
{
  "text": "apple"
}
```

POST http://localhost:9200/autocomplete_index/_doc/2
```json
{
  "text": "application"
}
```

POST http://localhost:9200/autocomplete_index/_doc/3
```json
{
  "text": "apply"
}
```

POST http://localhost:9200/autocomplete_index/_doc/4
```json
{
  "text": "appoint"
}
```

## 3. Test the Search Query
Use the following query to test the autocomplete with typo tolerance:

GET http://localhost:9200/autocomplete_index/_search
```json
{
  "query": {
    "match": {
      "text": {
        "query": "applcation",
        "fuzziness": "AUTO",
        "prefix_length": 2,
        "max_expansions": 50
      }
    }
  }
}
```
