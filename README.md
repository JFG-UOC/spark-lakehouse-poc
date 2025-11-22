# 🚀 Spark Lakehouse PoC on Kubernetes (microk8s)
### Open‑Source Data Lakehouse Architecture — Production‑Grade PoC

This repository contains a **Proof of Concept (PoC)** for a fully open‑source **Data Lakehouse** architecture deployed on **Kubernetes (microk8s)**. It includes:

- **Apache Spark 3.5.6** (Master, Workers, Thrift Server)
- **Jupyter Notebook** with Spark 3.5.6
- **Hive Metastore 3.1.3** backed by **MariaDB 11.4**
- **MinIO S3** as the object‑storage data lake
- **Delta Lake 3.3.2** and **RAPIDS 25.10.0** support
- Full S3A Hadoop integration
- External access via LoadBalancer services

All components are deployed via declarative YAML manifests.

---

# 📦 Components & Versions

## Namespace
- `lakehouse`

## MinIO
- minio/minio:RELEASE.2025‑09‑07  
- minio/mc:RELEASE.2025‑08‑13

## MariaDB
- mariadb:11.4

## Hive Metastore
- apache/hive:3.1.3  
- mariadb‑java‑client 3.5.1  
- hadoop‑aws 3.1.0  
- aws‑java‑sdk‑bundle 1.11.271

## Spark
- apache/spark:3.5.6  
- pyspark 3.5.6  
- Delta Lake 3.3.2  
- RAPIDS 25.10.0  

## Jupyter
- jupyter/datascience‑notebook:python‑3.8  
- OpenJDK 11.0.27  
- Spark 3.5.6 (downloaded by initContainer)

---

# 🧱 Architecture

```
                    ┌──────────────────────────┐
                    │      SQL / BI Clients     │
                    └───────────────┬───────────┘
                                    ▼
                         ┌──────────────────────┐
                         │ Spark Thrift Server  │
                         └─────────────┬────────┘
                                       ▼
             ┌─────────────────────────┼──────────────────────────┐
             ▼                         │                          ▼
   ┌───────────────────┐       ┌───────────────────┐      ┌──────────────────┐
   │   Spark Master    │◄─────►│   Spark Worker    │◄────►│  Spark Worker    │
   └───────────────────┘       └───────────────────┘      └──────────────────┘
                                       ▼
                         ┌──────────────────────┐
                         │    Hive Metastore    │
                         └─────────────┬────────┘
                                       ▼
                         ┌──────────────────────┐
                         │        MinIO S3       │
                         └──────────────────────┘
```

---

# 📁 Repository Structure

```
.
├── 00_namespace.yaml
├── 01_minio.yaml
├── 02_mariadb.yaml
├── 03_hive_metastore.yaml
├── 04_spark.yaml
├── 05_spark_thrift.yaml
├── 06_jupyter.yaml
├── 99_external_access.yaml
└── versions.txt
```

---

# 🚀 Deployment

## Requirements

```
microk8s enable dns storage ingress metallb
microk8s enable metallb:192.168.1.240-192.168.1.250
```

## Apply Manifests

```
kubectl apply -f 00_namespace.yaml
kubectl apply -f 01_minio.yaml
kubectl apply -f 02_mariadb.yaml
kubectl apply -f 03_hive_metastore.yaml
kubectl apply -f 04_spark.yaml
kubectl apply -f 05_spark_thrift.yaml
kubectl apply -f 06_jupyter.yaml
kubectl apply -f 99_external_access.yaml
```

## Verify Deployment

```
kubectl get pods -n lakehouse
kubectl get svc -n lakehouse
```

---

# 🌐 External Access

| Service | Port | Description |
|---------|-------|-------------|
| `minio-lb` | 9000 / 9001 | S3 API / Console |
| `spark-master-ui-lb` | 8080 | Spark Master UI |
| `spark-master-lb` | 7077 | Spark cluster endpoint |
| `spark-master-rest-lb` | 6066 | Spark REST API |
| `spark` | 10000 | Spark Thrift Server |
| `jupyter-svc-lb` | 8888 | Jupyter Notebook |

---

# 🔧 S3A Configuration

```
fs.s3a.endpoint=http://minio-svc:9000
fs.s3a.path.style.access=true
fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem
fs.s3a.connection.ssl.enabled=false
```

Credentials come from `minio-secret`.

---

# 🧪 Quick Tests

## Create Delta Table

```sql
CREATE TABLE default.demo_delta (
  id INT,
  msg STRING
)
USING delta;
```

## Use Spark from Jupyter

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
spark.range(10).show()
```

## spark-submit Test

```bash
/opt/spark/bin/spark-submit   --master spark://spark-master-svc:7077   /opt/spark/examples/src/main/python/pi.py 1000
```

---

# 🔮 Next Steps

- Add Keycloak authentication (OIDC)
- Add open‑source Unity Catalog alternative
- Add Superset for BI
- Convert manifests into full Helm chart

---

# 👤 Author
**Jorge Florencio García**  
Data Architect
