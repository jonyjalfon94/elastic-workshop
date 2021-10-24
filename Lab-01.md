# Lab 01: Deploy Elasticsearch using Docker Compose

## Tasks

 - Configure your workspace
 - Create Elasticsearch configuration file
 - Deploy Elasticsearch using Docker Compose
 - Explore Elasticsearch

## Configure your workspace

1. During the tutorial we will create several files that will be pased as volumes to the different applications. To make it easier let's create a new directory to be used as our workspace (we will use "~/logging-lab" but you can use a different one if you prefer)

```
mkdir ~/logging-lab
```

2. Create a dedicated folder for your elasticsearch files

```
mkdir ~/logging-lab/elasticsearch
```

## Create Elasticsearch configuration file

1. Now let's create a basic configuration file for Elasticsearch (I will use vim, you can use a different editor)

```
vim ~/logging-lab/elasticsearch/elasticsearch.yml
```

2. The content of the file should be the below:

```
---
cluster.name: "bootcamp-elk-workshop"
network.host: 0.0.0.0
```

## Deploy Elasticsearch using Docker Compose

1. Let's create a docker-compose file in the workspace root

```
vim ~/logging-lab/docker-compose.yml
```

2. The content of the docker-compose file should be the following:

```
version: '3.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
    volumes:
      - ./elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
      - elasticsearch:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      ELASTIC_PASSWORD: changeme
      discovery.type: single-node
    networks:
      - elk
networks:
  elk:
volumes:
  elasticsearch:
```

3. Deploy Elasticsearch using docker compose

```
cd ~/logging-lab
docker-compose up -d
```

## Explore Elasticsearch

1. Elasticsearch itself does not have a UI but we are going to do a couple of requests to it's API. The first request will be to the / endpoint to see if it was installed succesfully.
  ```
  curl http://elastic:changeme@localhost:9200/
  ```

2. We can check the health of the cluster by sending a GET request to the health endpoint. We will see that the health of the cluster is yellow and this is because we are using a single node cluster.
  ```
    curl http://elastic:changeme@localhost:9200/_cat/health
  ```

3. We are going to create two indices where we will uploading our documents.
  ```
  curl -XPUT "http://elastic:changeme@localhost:9200/app-logs-auth"

  curl -XPUT "http://elastic:changeme@localhost:9200/app-logs-actions"
  ```
4. Now we are going to upload a document into each of the indices that we created.
  ```
  curl -XPOST 'elastic:changeme@localhost:9200/app-logs-auth/my_app' -H 'Content-Type: application/json' -d'
  {
  	"timestamp": "2020-01-24 12:34:56",
  	"message": "User logged in",
    "username": "mickeymouse",
  	"user_id": 4,
  	"admin": false
  }
  '
  
  curl -XPOST 'elastic:changeme@localhost:9200/app-logs-actions/my_app' -H 'Content-Type: application/json' -d'
  {
  	"timestamp": "2020-01-24 12:34:56",
  	"message": "User performed action",
    "username": "mickeymouse",  
    "action": "Buy",
    "ammount": "37",
  	"user_id": 999,
  	"admin": false
  }
  '
  ```
5. Indexed documents are available for search in near real-time. To search data, we are going to use the _search API. The following request will get all the documents in all the indexes that match the app-logs-* pattern.
  ```
  curl -XGET 'elastic:changeme@localhost:9200/app-logs-*/_search?pretty'
  ```

6. The following request uses the same _search api but this time we are sending a query that matches documents that contain the string "User logged in" in the field "message"
  ```
  curl -XGET 'elastic:changeme@localhost:9200/app-logs-*/_search?pretty' -H 'Content-Type: application/json' -d'
  {
    "query": {
      "match_phrase": {
        "message": "User logged in"
      }
    }
  }
  '
  ```






