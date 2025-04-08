# 204815 - KIPROTICH IAN MARK - CAT 2 SUBMISSION.
# Information Retrieval System using Elasticsearch & Kibana: A case study of Sports-related documents

## Overview

This technical manual details how to set up a traditional Information Retrieval (IR) system using Elasticsearch and Kibana via Docker Compose. The document corpus consists of sports-related documents.

---

## 🧰 Tools & Technologies

- **Elasticsearch 8.12.0**: Full-text search and analytics engine.
- **Kibana 8.12.0**: Data visualization for Elasticsearch.
- **Docker & Docker Compose**: Containerization and orchestration.
- **Corpus**: Sports-related PDF documents stored in a volume.

---

## 🛠️ Prerequisites

- Docker installed: https://docs.docker.com/get-docker/
- Docker Compose installed: https://docs.docker.com/compose/install/
- Docker Compose yaml file
- Folder structure:
``` ir-project/
├── corpus/
│   └── sports
          └── sports-docs.pdf
          └── sports_bulk.json
          └── sports_bulk_ngram.json

├── es_data/
└── .env
└── docker-compose.yml

```

## Step 1: Setup a Single-node cluster 


### Step 1a: Run Elastic-Search in docker (Single-node Cluster)


#### Create a new docker network.


```
docker network create elastic
```
#### Pull the elastic docker image

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.17.4
```

#### Optional: Install Cosign for your environment. Then use Cosign to verify the Elasticsearch image’s signature.
```
wget https://artifacts.elastic.co/cosign.pub
cosign verify --key cosign.pub docker.elastic.co/elasticsearch/elasticsearch:8.17.4
```
if/incase ```wget``` gives an error use ```curl```
#### Start an Elasticsearch container:
```
docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.17.4
```
The command prints the elastic user password and an enrollment token for Kibana.

#### Make a REST API call to Elasticsearch to ensure the Elasticsearch container is running
```
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

### Step 1b: Run Kibana in docker


#### Pull the Kibana Docker image.
```
docker pull docker.elastic.co/kibana/kibana:8.17.4
```
#### Optional: Verify the Kibana image’s signature.
```
wget https://artifacts.elastic.co/cosign.pub
cosign verify --key cosign.pub docker.elastic.co/kibana/kibana:8.17.4
```
if/incase ```wget``` gives an error use ```curl```

#### Start a Kibana container:
```
docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.17.4
```
The command prints the elastic user password and an enrollment token for Kibana.

#### Lets remove containers now that we have tested elasticsearch and kibana
To remove container run:
```
# Remove the Elastic network
docker network rm elastic

# Remove Elasticsearch containers
docker rm es01
docker rm es02

# Remove the Kibana container
docker rm kib01
```

## Step 2: Setup Multi-node cluster

### Create a ```docker-compose.yml``` and add the following 
```

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.12.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    volumes:
      - ./es_data:/usr/share/elasticsearch/data
      - ./corpus:/usr/share/elasticsearch/corpus
    ports:
      - "9200:9200"
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:8.12.0
    container_name: kibana
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```

Note: Port for Elasticsearch is set to ```9200``` as shown and for Kibana as ```5601``` which will be used as access point to perform operations.

### Add ```.env``` file
This comes is really important in hiding your elastic and kibana password, Elastic stack version and the Port to expose Elasticsearch HTTP API to the host
```
# Password for the 'elastic' user (at least 6 characters)
ELASTIC_PASSWORD=changeme

# Password for the 'kibana_system' user (at least 6 characters)
KIBANA_PASSWORD=changeme

...
# Version of Elastic products
STACK_VERSION=8.17.4
...
# Port to expose Elasticsearch HTTP API to the host
#ES_PORT=9200
ES_PORT=127.0.0.1:9200
...

```

### Run/Compose up to start the cluster
Simply right click on the ```docker-compose.yml``` and select ```compose - select services``` and choose both ```elastic-search``` and ```kibana```

the output on the terminal will be:
```
 *  Executing task: docker compose -f 'docker.compose.yml' up -d --build 'elasticsearch' 'kibana' 

[+] Running 2/0
 ✔ Container elasticsearch  Running                                                          0.0s 
 ✔ Container kibana         Running                                                          0.0s 
 *  Terminal will be reused by tasks, press any key to close it.
```
Now we can load data and query/retrieve information from it
### Step 3: Load Document Corpus
The choice of topic was around Sports-related PDF documents stored in a volume.

A preloaded ```json``` and ```pdf``` file were loaded in to a folder named ```/corpus/sports/``` both of which contain the data that will be ingested by elasticsearch
 #### 1. Open a terminal
 cd
 ```
cd corpus/sports/
```

#### 2. Having the information/corpus in ```json``` format in the directory, we can use curl to Bulk Index to upload to elasticsearch for indexing
why? You cannot upload a file directly in Kibana, so use curl instead for bulk upload.
In terminal:
```
curl -X POST "localhost:9200/_bulk" \
  -H "Content-Type: application/json" \
  --data-binary @sports_bulk.json
```

#### 3. Access Kibana 
Simply right click on the ```docker-compose.yml``` and select ```compose - select services``` and choose both ```elastic-search``` and ```kibana```

the output on the terminal will be:
```
 *  Executing task: docker compose -f 'docker.compose.yml' up -d --build 'elasticsearch' 'kibana' 

[+] Running 2/0
 ✔ Container elasticsearch  Running                                                          0.0s 
 ✔ Container kibana         Running                                                          0.0s 
 *  Terminal will be reused by tasks, press any key to close it.
```
go to ```http://localhost:5601/```

#### 4. Index the Extracted Content into Elasticsearch(at Kibana - devtools)
In json
```
PUT /sports
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" }
    }
  }
}

```

#### 5. Perform a search on Indexed Content
```
GET /sports/_search
{
  "query": {
    "match": {
      "content": "NBA"
    }
  }
}

```
output:
```
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 3.4788775,
    "hits": [
      {
        "_index": "sports",
        "_id": "R5KGEpYBo7CWKCtLGzdu",
        "_score": 3.4788775,
        "_source": {
          "title": "significance in modern basketball.",
          "content": "NBA Evolution (22)"
        }
      },
      {
        "_index": "sports",
        "_id": "PJKGEpYBo7CWKCtLGzdu",
        "_score": 2.7833369,
        "_source": {
          "title": "Famous NBA Players (14)",
          "content": "Famous NBA Players (14) - This section provides an overview of famous nba players (14) and its"
        }
      },
      {
        "_index": "sports",
        "_id": "U5KGEpYBo7CWKCtLGzdu",
        "_score": 2.7833369,
        "_source": {
          "title": "Famous NBA Players (30)",
          "content": "Famous NBA Players (30) - This section provides an overview of famous nba players (30) and its"
        }
      },
      {
        "_index": "sports",
        "_id": "WpKGEpYBo7CWKCtLGzdu",
        "_score": 2.7833369,
        "_source": {
          "title": "Famous NBA Players (35)",
          "content": "Famous NBA Players (35) - This section provides an overview of famous nba players (35) and its"
        }
      },
      {
        "_index": "sports",
        "_id": "YJKGEpYBo7CWKCtLGzdu",
        "_score": 2.7833369,
        "_source": {
          "title": "Famous NBA Players (39)",
          "content": "Famous NBA Players (39) - This section provides an overview of famous nba players (39) and its"
        }
      }
    ]
  }
}
```

## Bonus: Search Optimization with Partial Word Matching

To enhance the **search experience** and support **partial word queries** (e.g., typing `"dunk"` to find `"dunking"`), I implemented a custom **N-Gram Analyzer** in Elasticsearch.

### Optimization steps:

- **Custom N-Gram Analyzer**:
  - Configured to tokenize text into substrings (n-grams) of 3 to 10 characters.
  - This allows for **matching on partial inputs**, enabling autocomplete-like behavior.

- **Index-Level Settings**:
  - We extended the `index.max_ngram_diff` setting to support a wider range between `min_gram` and `max_gram`.
  - Applied the analyzer on the `content` field during indexing for targeted improvements.

- **Why it Matters**:
  - Users don’t need to know full terms.
  - Ideal for scenarios involving:
    - Typo tolerance
    - Autocomplete
    - Substring search
   
    
#### step 1: Delete current sports content on kibana
```
DELETE /sports
```
#### step 2: Create the New Index with ngram Analyzer on kibana
Configuration is done to tokenize text into substrings (n-grams) of 3 to 10 characters.
This allows for **matching on partial inputs**, enabling autocomplete-like behavior.

```
PUT /sports
{
  "settings": {
    "index": {
      "max_ngram_diff": 10
    },
    "analysis": {
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 10,
          "token_chars": [ "letter", "digit" ]
        }
      },
      "analyzer": {
        "ngram_analyzer": {
          "type": "custom",
          "tokenizer": "ngram_tokenizer",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "content": {
        "type": "text",
        "analyzer": "ngram_analyzer",
        "search_analyzer": "standard"
      }
    }
  }
}
```

#### step 4: Upload Bulk Data via Curl


```
cd corpus/sports

curl -X POST "localhost:9200/_bulk" \
  -H "Content-Type: application/json" \
  --data-binary "@sports_bulk_ngram.json"


```

#### step 5: Create a Data View in Kibana

choose the sports index and add an (*) asterick 
```
sports*
```

### DONE! Perform searches on kibana discover or devtools

### For Example
A search for:

```json
{
  "query": {
    "match": {
      "content": "dunk"
    }
  }
}

```

## References

- [Elasticsearch Official Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/index.html)
- [Kibana Official Documentation](https://www.elastic.co/guide/en/kibana/index.html)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Bulk Upload Format in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)
- [Custom Analyzers in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html)
- [N-Gram Token Filter in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenfilter.html)
- [Kibana DevTools Console](https://www.elastic.co/guide/en/kibana/current/console-kibana.html)
- [Creating Index Patterns in Kibana](https://www.elastic.co/guide/en/kibana/current/index-patterns.html)
