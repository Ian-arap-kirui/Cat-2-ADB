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
- Folder structure: 
<img src="./screenshots/file-structure.png" alt="Description" width="200"/>

===== Start a single-node cluster

. Install Docker. Visit https://docs.docker.com/get-docker/[Get Docker] to
install Docker for your environment.
+
If using Docker Desktop, make sure to allocate at least 4GB of memory. You can
adjust memory usage in Docker Desktop by going to **Settings > Resources**.

. Create a new docker network.
+
[source,sh]
----
docker network create elastic
----
// REVIEWED[DEC.10.24]
. Pull the {es} Docker image.
+
--
ifeval::["{release-state}"=="unreleased"]
WARNING: Version {version} has not yet been released.
No Docker image is currently available for {es} {version}.
endif::[]

[source,sh,subs="attributes"]
----
docker pull {docker-image}
----
// REVIEWED[DEC.10.24]
--