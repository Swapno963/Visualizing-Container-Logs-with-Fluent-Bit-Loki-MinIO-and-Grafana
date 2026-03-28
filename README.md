# Kubernetes Observability Platform (Fluent Bit + Loki + Grafana)

## 🚀 Overview

This project implements a **full-stack observability platform** for Kubernetes, enabling **real-time log streaming, structured search, and visual insights into system behavior**.

**Problem Solved:**
Raw logs are not useful unless they are **centralized, queryable, and visualized**. In distributed systems, debugging requires:

* Fast log retrieval
* Context-aware filtering (pod, namespace, service)
* Trend analysis over time

This project transforms raw container logs into a **searchable and visual observability layer**.

**Why this architecture:**

* **Fluent Bit:** Lightweight, node-level log collection with minimal overhead
* **Loki:** Efficient log aggregation using label-based indexing (cost-optimized vs full-text search systems)
* **Grafana:** Real-time visualization, filtering, and dashboards

The system is designed to balance **performance, cost, and usability**, which is critical in production environments.

---

## 🏗️ Architecture

```text
          ┌──────────────────────────┐
          │        Grafana           │
          │ Dashboards + Log Search  │
          └──────────┬───────────────┘
                     │ Query (LogQL)
             ┌───────▼────────┐
             │     Loki       │
             │ Index + Store  │
             └───────┬────────┘
                     │
             ┌───────▼────────┐
             │  Fluent Bit    │
             │ (DaemonSet)    │
             └───────┬────────┘
                     │
        ┌────────────▼────────────┐
        │ /var/log/containers/    │
        │ (All Node Logs)         │
        └─────────────────────────┘
```

**Component Interaction:**

* Fluent Bit collects logs from all nodes
* Logs are enriched with Kubernetes metadata
* Loki indexes logs using labels (not full-text indexing)
* Grafana queries Loki and provides visualization + filtering

**Design Trade-offs:**

* **Loki vs ELK**

  * Loki: lower cost, label-based indexing, faster ingestion
  * ELK: richer full-text search but significantly higher resource usage

* **DaemonSet vs Sidecar**

  * DaemonSet chosen for efficiency and cluster-wide coverage
  * Sidecar avoided to reduce per-pod overhead

* **Label-based indexing**

  * Faster queries and lower storage cost
  * Requires careful label design to avoid high cardinality

---

## ⚙️ Tech Stack

**Observability:**

* Fluent Bit (log collection)
* Loki (log aggregation and indexing)
* Grafana (visualization and dashboards)

**Infrastructure:**

* Kubernetes (multi-node cluster)
* Docker (container runtime)

**Storage:**

* MinIO (S3-compatible object storage for Loki)

**Configuration:**

* ConfigMaps (Fluent Bit + Loki configuration)

---

## 📦 Services / Components

* **Fluent Bit (DaemonSet)**

  * Collects logs from `/var/log/containers/`
  * Enriches logs with Kubernetes metadata
  * Streams logs to Loki

* **Loki**

  * Receives logs via HTTP
  * Indexes logs using labels (namespace, pod, container)
  * Stores logs in MinIO

* **Grafana**

  * Provides:

    * Live log streaming
    * Clickable filters (labels)
    * Time-range selectors
    * Dashboards for log trends

* **MinIO**

  * Persistent storage backend for logs
  * Enables durability and scalability

---

## 🔄 Request Flow

1. Application containers write logs to stdout

2. Kubernetes stores logs in:

   ```bash
   /var/log/containers/
   ```

3. Fluent Bit:

   * Tails log files
   * Enriches logs with metadata
   * Sends logs to Loki

4. Loki:

   * Processes and indexes logs
   * Stores log chunks in MinIO

5. Grafana:

   * Queries Loki using LogQL
   * Displays:

     * Real-time log streams
     * Filtered views (by pod, namespace, labels)
     * Time-based dashboards

---

## 🛠️ Setup & Installation

```bash
# Clone repository
git clone <repo-url>
cd k8s-observability-platform

# Deploy storage layer
kubectl apply -f k8s/minio/

# Deploy Loki
kubectl apply -f k8s/loki/

# Deploy Fluent Bit
kubectl apply -f k8s/fluentbit/

# Deploy Grafana
kubectl apply -f k8s/grafana/

# Verify
kubectl get pods -A
kubectl get svc -A
```

Access Grafana:

```bash
kubectl port-forward svc/grafana 3000:3000
```

---

## 🧪 Key DevOps Concepts Demonstrated

* **Centralized Logging Architecture**

  * Aggregation of logs across all nodes

* **DaemonSet Pattern**

  * Ensures one log collector per node

* **Metadata Enrichment**

  * Adds context required for debugging distributed systems

* **Observability Layer**

  * Transition from raw logs → actionable insights

* **Decoupled System Design**

  * Collection, storage, and visualization are independent

* **Log Querying**

  * Efficient querying using label-based indexing (LogQL)

---

## 🔐 Production Considerations

* **Scalability**

  * Loki can scale horizontally (distributor/ingester pattern)
  * MinIO should run in distributed mode for HA

* **Label Strategy**

  * Avoid high cardinality (e.g., dynamic user IDs)
  * Focus on stable labels (namespace, service, pod)

* **Security**

  * Secure Grafana access (auth, RBAC)
  * Restrict Fluent Bit node access (`hostPath`)

* **Observability Maturity**

  * Logs alone are insufficient → combine with metrics and tracing

* **Failure Handling**

  * Fluent Bit buffering prevents log loss
  * Loki replication improves durability

---

## 🚧 Challenges & Learnings

* **Logs without visualization are low-value**

  * Centralization alone is not enough → need querying + dashboards

* **Label design complexity**

  * Poor label choices led to inefficient queries
  * Required balancing granularity vs performance

* **Understanding real log flow**

  * Logs originate from node filesystem, not directly from containers

* **Mental Model Shift:**

  * **Junior thinking:** “Logging is just printing logs”
  * **Production reality:**

    * Logging is part of **observability strategy**
    * Requires **collection, enrichment, indexing, and visualization**

---

## 📌 Future Improvements

* Add distributed tracing (OpenTelemetry + Tempo)
* Correlate logs with metrics (Prometheus integration)
* Implement alerting based on log queries
* Introduce multi-tenant log isolation
* Add retention policies and cost optimization strategies

---

## 📸 Diagram (Simplified)

```text
[Containers]
     ↓
[/var/log/containers]
     ↓
[Fluent Bit]
     ↓
[Loki]
     ↓
[MinIO]
     ↓
[Grafana Dashboard]
```

---

## 💡 Key Takeaway

This project demonstrates how to move from **raw logs → actionable observability**:

* Efficient log collection using Kubernetes-native patterns
* Cost-aware log aggregation with Loki
* Real-time visualization and filtering with Grafana
* Thoughtful trade-offs between performance, cost, and usability
