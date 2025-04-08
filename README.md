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
<img src="./screenshots/file-structure.png" alt="Description" width="200"/>

## Step 1: Run Elastic-Search in docker


### Create a new docker network.


```
docker network create elastic
```
### Pull the elastic docker image

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.17.4
```

### Optional: Install Cosign for your environment. Then use Cosign to verify the Elasticsearch image’s signature.
```
wget https://artifacts.elastic.co/cosign.pub
cosign verify --key cosign.pub docker.elastic.co/elasticsearch/elasticsearch:8.17.4
```
### Start an Elasticsearch container:
```
docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.17.4
```
The command prints the elastic user password and an enrollment token for Kibana.

output: 
