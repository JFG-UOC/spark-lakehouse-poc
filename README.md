# 🚀 Spark Lakehouse PoC on Kubernetes (microk8s)

This repository contains a **Proof of Concept (PoC)** demonstrating an **open‑source Data Lakehouse architecture** deployed on **Kubernetes (microk8s)**.

The goal is to build a fully functioning analytical environment composed of:

- **Apache Spark 4.0.1**
- **Hive Metastore**
- **S3‑compatible object storage (MinIO)**
- **Spark Thrift Server with JDBC/ODBC**
- **External access via LoadBalancer services**
- **Support for Iceberg and Delta Lake tables**

All components are deployed through reproducible YAML manifests.

---

## 📁 Repository Structure

```
.
├── 00_namespace.yaml
├── 01_minio.yaml
├── 02_mariadb.yaml
├── 03_hive_metastore.yaml
├── 04_spark.yaml
├── 05_spark_thrift.yaml
└── 99_external_access.yaml
```

### File Overview

| File | Description |
|------|-------------|
| **00_namespace.yaml** | Creates the `lakehouse` namespace. |
| **01_minio.yaml** | Deploys MinIO + root credentials + internal S3 service. |
| **02_mariadb.yaml** | MariaDB database backend for Hive Metastore. |
| **03_hive_metastore.yaml** | Hive Metastore with S3A configuration and dependencies. |
| **04_spark.yaml** | Spark Master + Workers, S3A configuration, external jars. |
| **05_spark_thrift.yaml** | Spark Thrift Server with JDBC/ODBC access. |
| **99_external_access.yaml** | LoadBalancer services for external access. |

---

## 🧱 Architecture

```
                        ┌────────────────────────────┐
                        │        SQL Clients          │
                        │ (DBeaver, JDBC Apps, BI...) │
                        └──────────────┬──────────────┘
                                       │ JDBC/ODBC
                                       ▼
                        ┌────────────────────────────┐
                        │     Spark Thrift Server    │
                        │          (cluster)         │
                        └──────────────┬──────────────┘
                                       │ Spark Jobs
                                       ▼
 ┌────────────────────────────┐   ┌─────────────────────────────┐
 │       Spark Master         │   │        Spark Workers         │
 │ (REST, Scheduler, History) │◄──►   (Executors, S3A access)   │
 └──────────────┬─────────────┘   └─────────────────────────────┘
                │
                ▼
        ┌─────────────────────────────┐
        │        Hive Metastore       │
        │ (Iceberg / Delta catalog)   │
        └──────────────┬──────────────┘
                       │
                       ▼
        ┌─────────────────────────────┐
        │            MinIO            │
        │  S3A: warehouse / tables    │
        └─────────────────────────────┘
```

---

## 🔧 Requirements

- **microk8s 1.30+** with:
  ```
  microk8s enable dns storage ingress metallb
  ```
- `kubectl` configured for your cluster
- At least **4 vCPUs** and **6–8 GB RAM**
- A MetalLB IP range, e.g.:
  ```
  microk8s enable metallb:192.168.1.240-192.168.1.250
  ```

---

## 🚀 Deployment

Clone the repository:

```
git clone https://github.com/<your_user>/<repo>.git
cd <repo>
```

Apply manifests in order:

```bash
kubectl apply -f 00_namespace.yaml
kubectl apply -f 01_minio.yaml
kubectl apply -f 02_mariadb.yaml
kubectl apply -f 03_hive_metastore.yaml
kubectl apply -f 04_spark.yaml
kubectl apply -f 05_spark_thrift.yaml
kubectl apply -f 99_external_access.yaml
```

Check everything is running:

```
kubectl get pods -n lakehouse
kubectl get svc -n lakehouse
```

---

## 🌐 External Access (LoadBalancer)

| Service | Port | Description |
|---------|-------|-------------|
| `spark-master-lb` | **8080** | Spark Master UI |
| `spark-thrift-lb` | **10000** | JDBC/ODBC for BI clients |

Example connection:

```
http://<LB_IP>:8080
jdbc:hive2://<LB_IP>:10000/
```

---

## 📦 S3A Storage

MinIO is used as the S3-compatible object store.

Spark configuration:

```
spark.hadoop.fs.s3a.endpoint = http://minio-svc:9000
spark.hadoop.fs.s3a.access.key = minioadmin
spark.hadoop.fs.s3a.secret.key = minioadmin123
spark.hadoop.fs.s3a.path.style.access = true
```

---

## 🧪 Quick Tests

### Create a Delta/Iceberg table

```sql
CREATE SCHEMA IF NOT EXISTS default;

CREATE TABLE default.sales_delta (
  id INT,
  date DATE,
  product STRING,
  qty INT,
  unit_price DECIMAL(10,2)
)
USING delta;
```

### Read data via spark-submit

```bash
/opt/spark/bin/spark-submit   --master spark://spark-master-svc:7077   /opt/spark/work-dir/read_demo.py
```

---

## 📝 PoC Goals

- Validate a fully open‑source Lakehouse architecture  
- Test compatibility with **S3A**, **Iceberg**, **Delta Lake**, and **Spark 4.0**
- Deploy reproducible components on Kubernetes
- Provide SQL access through the **Spark Thrift Server**

---

## 📚 Next Steps

- Add open‑source Unity Catalog
- Add Superset for visualization
- Integrate Keycloak authentication
- Convert manifests into a full Helm chart

---

## 🧑‍💻 Author

**Jorge Florencio**  
Data Architect
