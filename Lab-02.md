# Lab 02: Deploy Kibana using Docker Compose

## Tasks

 - Create Kibana configuration file
 - Deploy Kibana using Docker Compose
 - Explore Kibana

## Create Kibana configuration file

1. Create a dedicated folder for your Kibana files in the workspace that we configured in the previous lab

```
mkdir ~/logging-lab/Kibana
```


2. Now let's create a basic configuration file for Kibana (I will use vim, you can use a different editor)

```
vim ~/logging-lab/Kibana/kibana.yml
```

3. The content of the file should be the below:

```
---
server.name: kibana
server.host: 0.0.0.0
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
```

## Deploy Kibana using Docker Compose

1. Let's add to the docker-compose file that we created in the previous lab a new service for kibana

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
      - type: bind
        source: ./elasticsearch/elasticsearch.yml
        target: /usr/share/elasticsearch/config/elasticsearch.yml
        read_only: true
      - type: volume
        source: elasticsearch
        target: /usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      ELASTIC_PASSWORD: changeme
      discovery.type: single-node
    networks:
      - elk
  kibana:
    image: docker.elastic.co/kibana/kibana:7.12.1
    volumes:
      - type: bind
        source: ./kibana/kibana.yml
        target: /usr/share/kibana/config/kibana.yml
        read_only: true
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch   
networks:
  elk:
volumes:
  elasticsearch:
```

3. Deploy Kibana using docker compose

```
cd ~/logging-lab
docker-compose up -d
```

## Explore Kibana

1. Browse to your kibana instance from any browser
  ```
  http://localhost:5601
  ```

2. Log In with the following credentials
```
username: elastic
password: changeme
```

3. Go to Management > Stack Management > Kibana > Index patterns

  ![index-pattern-1](/images/index-pattern-1.png)

4. Let's create a Kibana index pattern to visualize all the indexes that match the pattern. Click on "Create index pattern" 

  ![index-pattern-2](/images/index-pattern-2.png)

5. You can use the "app-logs-*" pattern to match the indexes that we created in the previous lab.

  ![index-pattern-3](/images/index-pattern-3.png)

6. Now let's go Analitycs > Discover to visualize all the documents that are present in the indices that match the index pattern that we created.

  ![discover](/images/discover.png)

7. Kibana comes with built in console that can help us write requests to elasticsearch in a more confortable way. To find it go to Management > Dev Tools

  ![dev-tools](/images/dev-tools.png)

8. Lets use the _bulk API to upload some more data. With the _bulk API you can perform many operations in a single request. We will use it to upload many documents at once to our app-logs-actions. Copy and paste the following into the console.

```
POST _bulk
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "mickeymouse", "action": "Buy", "ammount": 43, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "jhon", "action": "Buy", "ammount": 67, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "mickeymouse", "action": "Buy", "ammount": 12, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "jhon", "action": "Buy", "ammount": 54, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "mickeymouse", "action": "Buy", "ammount": 79, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "fernando", "action": "Buy", "ammount": 34, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "fernando", "action": "Buy", "ammount": 767, "user_id": 999, "admin": false}
{ "index":{ "_index": "app-logs-actions", "_type": "my_app" } }
{ "timestamp": "2020-01-24 12:34:56", "message": "User performed action", "username": "fernando", "action": "Buy", "ammount": 23, "user_id": 999, "admin": false}
```

9. Click on the play button to perform the request and get the response.

  ![dev-tools](/images/dev-tools-2.png)

10. Now that we have some data lets create our first dashboard. Go to Analytics > Dashboard and click the "Create Dashboard" Button

  ![dashboard-1](/images/dashboard-1.png)

11. Click on the create panel button

  ![dashboard-2](/images/dashboard-2.png)

12. Chose the Lens option

  ![dashboard-3](/images/dashboard-3.png)

13. Chose the "Bar" chart type

14. From the menu on the left drag the field "username.keyword" and drop it to the Horizontal axis

15. From the menu on the left drag the field "ammount" and drop it to the Vertical axis then select Sum

17. It should look like the following picture:

  ![dashboard-4](/images/dashboard-4.png)

17. Click the "Save and Return" button to save the new panel.

18. Click the "Save" button to save the dashboard and give it a name.