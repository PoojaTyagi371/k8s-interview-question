# Kubernetes & Monitoring Interview Questions & Answers

---

## Part 1: Core Concepts

### 1. Why is monitoring required in a production environment?
**Answer:**  
Monitoring is required to continuously check the health and performance of applications and infrastructure. It helps identify problems such as high CPU usage, memory issues, disk problems, network failures, application failures, and downtime before they become major outages.

**Monitoring also helps:**
* Detect issues proactively
* Reduce downtime
* Understand real-time application health
* Perform capacity planning
* Troubleshoot failures
* Identify abnormal behavior or trends

---

### 2. What is centralized monitoring?
**Answer:**  
Centralized monitoring means collecting monitoring data and logs from multiple servers, nodes, pods, or applications into a centralized monitoring system. Instead of manually checking logs across individual machines, all monitoring data is collected in one place.

**Benefits include:**
* Easier troubleshooting
* Centralized visibility
* Better security
* Reduced manual effort
* Efficient monitoring of large-scale environments
* Data retention even if an individual host goes down

---

### 3. What is Prometheus?
**Answer:**  
Prometheus is an open-source monitoring and metrics collection system widely used with Kubernetes. It collects metrics via a pull model and allows users to query them using **PromQL**.

**It collects and stores metrics such as:**
* CPU and memory usage
* Pod and node health
* Application-level metrics
* Kubernetes object states

---

### 4. What is Grafana?
**Answer:**  
Grafana is an open-source visualization and dashboarding platform. It connects to time-series data sources like Prometheus to render metrics through dynamic graphs, charts, and alert panels.

* **Default port:** Runs on port `3000` by default.
* **Role:** While Prometheus handles metrics collection and storage, Grafana serves as the visualization layer.

---

### 5. What is the difference between Prometheus and Grafana?
**Answer:**

| Feature | Prometheus | Grafana |
| :--- | :--- | :--- |
| **Primary Role** | Monitoring and metrics collection | Visualization and dashboard system |
| **Data Handling** | Collects and stores time-series data | Queries and displays metrics visually |
| **Query Engine** | Executes queries using PromQL | Forwards queries to configured data sources |
| **Architecture Role**| Metrics storage backend | Presentation and alerting layer |

**Standard Data Flow:**
$$\text{Kubernetes Cluster / Exporters} \longrightarrow \text{Prometheus} \longrightarrow \text{Grafana Dashboards}$$

---

### 6. Why is Prometheus commonly used with Kubernetes?
**Answer:**  
Prometheus has native service discovery for dynamic Kubernetes environments, making it ideal for monitoring rapidly changing workloads.

**Key resources monitored:**
* Cluster nodes and physical infrastructure
* Pods, Deployments, and ReplicaSets
* Internal Kubernetes control-plane components
* Custom application metrics via Prometheus client libraries

---

### 7. What is Helm?
**Answer:**  
Helm is the package manager for Kubernetes, functioning similarly to `apt` or `yum` in Linux. Helm packages collections of Kubernetes YAML manifests into reusable units called **Helm Charts**.

**Key capabilities:**
* Automated application installation
* Seamless version upgrades and rollbacks
* Centralized configuration management

---

### 8. What is a Helm Chart?
**Answer:**  
A Helm Chart is a bundled set of templates and YAML configuration files used to define, version, and deploy Kubernetes applications.

**Core components of a chart:**
* `Chart.yaml`: Contains metadata about the chart (name, version, description).
* `values.yaml`: Default configuration values for deployment templates.
* `templates/`: Manifest templates that render into valid Kubernetes YAML files when combined with values.

---

### 9. What is `values.yaml` in Helm?
**Answer:**  
`values.yaml` is the central configuration file in a Helm chart. It allows you to customize deployment parameters without modifying the core template files directly.

**Common overrides in `values.yaml`:**
* Container image repositories and tag versions
* CPU and memory resource requests/limits
* Replica counts and scaling parameters
* Service types (e.g., `ClusterIP`, `NodePort`, `LoadBalancer`)

---

### 10. What are Node Exporter and Kube-State-Metrics?
**Answer:**  
Both are metrics exporters used by Prometheus, but they target different layers of the infrastructure:

* **Node Exporter:** Gathers hardware- and OS-level metrics from the underlying host (CPU, memory usage, disk I/O, network stats).
* **Kube-State-Metrics:** Gathers cluster-level object state metrics (pod readiness, deployment replicas, container restart counts).

---

## Part 2: Scenario-Based Questions

### 1. Scenario: Your production application is suddenly down. How would monitoring help you troubleshoot it?
**Answer:**  
1. **Check the Grafana Dashboard:** Identify whether the failure is infrastructure-level (high node CPU/memory/disk exhaustion) or application-level (HTTP 5xx spikes, crashing pods).
2. **Inspect Pod & Deployment Health:** Look for failing health probes (`CrashLoopBackOff`, failed liveness/readiness checks).
3. **Analyze Timeline & Alerts:** Correlate the downtime with recent alerts, deployments, or traffic surges.
4. **Targeted Investigation:** Use the metrics to navigate directly to the affected node or pod logs, avoiding manual inspection across nodes.

---

### 2. Scenario: You have 1,000 Kubernetes pods. Can you manually check the logs and resource usage of every pod?
**Answer:**  
No. Manual inspection across 1,000 pods is inefficient. The standard approach is implementing a centralized observability stack:

* Use **Prometheus** to scrape metrics across all pods automatically via Kubernetes Service Discovery.
* Use **Grafana** to create aggregated dashboards with filtering by namespace, deployment, and pod labels.
* Use a centralized logging engine (such as Loki or an ELK stack) to aggregate logs into a single searchable interface.

---

### 3. Scenario: A server goes down and its local logs are no longer available. How would you investigate the incident?
**Answer:**  
Rely on the centralized monitoring and logging systems. Because metrics and logs are continuously shipped off the host in real time, the data remains available after the node crashes:

* Inspect Prometheus time-series data right up to the minute of failure (e.g., memory exhaustion or kernel panics).
* Review centralized log streams to check the last logged system and application events before disconnection.

---

### 4. Scenario: An attacker deletes logs from a compromised server. How can centralized monitoring help?
**Answer:**  
Centralized logging and monitoring act as an immutable audit trail. Even if a local log file is cleared on the host, previously shipped logs and metric streams persist on the remote monitoring server.

**Investigation steps:**
* Query the central logging system for the timeframe leading up to the breach.
* Check metrics for anomalous outbound traffic, spikes in privilege escalation attempts, or sudden process terminations.

---

### 5. Scenario: Your Kubernetes cluster has very high CPU usage. How would you investigate it?
**Answer:**  
1. **Identify the Scope:** Use Prometheus/Grafana to find whether the CPU spike is cluster-wide, localized to specific nodes, or isolated to a specific namespace.
2. **Find High-Consuming Pods:** Query top CPU-consuming pods using PromQL (`topk()` on container CPU usage).
3. **Review Resource Allocations:** Check if pods have missing or poorly configured `requests` and `limits`.
4. **Remediate:** If traffic has legitimately increased, configure a **Horizontal Pod Autoscaler (HPA)** or scale cluster nodes; if it is a runaway process, profile the application.

---

### 6. Scenario: You need to deploy Prometheus and Grafana into a Kubernetes cluster. Would you manually create every manifest?
**Answer:**  
No. The standard approach is using the `kube-prometheus-stack` Helm chart.

**Deployment steps:**
```bash
# 1. Add and update the Prometheus community Helm repository
helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
helm repo update

# 2. Deploy the monitoring stack into a dedicated namespace
helm install monitoring-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```
### 7. Scenario: Your company wants monitoring components isolated from application workloads. What would you do?

**Answer:**

Implement Kubernetes namespace isolation and node segmentation:

* **Namespaces:** Deploy all monitoring components into a dedicated namespace (e.g., `monitoring`) with strict RBAC policies.
* **Node Affinity / Taints:** Use node taints, tolerations, or `nodeSelector` so monitoring pods run on dedicated monitoring nodes, preventing resource competition with business applications.

---

### 8. Scenario: Grafana is running, but no Prometheus metrics are appearing. What would you check?

**Answer:**

1. **Verify Prometheus Pod Status:** Run `kubectl get pods -n <namespace>` to confirm the Prometheus server pod is `Running`.
2. **Check Network Connectivity:** Verify that the Prometheus Kubernetes Service is active and reachable on its target port (default `9090`).
3. **Inspect Grafana Data Source Settings:** Ensure the Data Source URL matches the internal cluster DNS (e.g., `[http://prometheus-server.monitoring.svc.cluster.local:9090](http://prometheus-server.monitoring.svc.cluster.local:9090)`).
4. **Test Query Directly:** Execute a basic PromQL query (like `up`) directly inside the Prometheus UI to confirm metrics are being scraped.

---

### 9. Scenario: Your Helm repository was added several months ago. Before installing a monitoring application, what should you do?

**Answer:**

Run `helm repo update` to sync local repository caches with the upstream chart repositories.

```bash
helm repo update

```

This retrieves the latest chart versions, dependency fixes, and security patches before running `helm install` or `helm upgrade`.

---

### 10. Scenario: You need to monitor both Kubernetes resources and the underlying nodes. Which components would you consider?

**Answer:**

Use a combined exporter architecture:

* **Node Exporter:** Deployed as a `DaemonSet` to capture node-level infrastructure metrics (disk, CPU, RAM, network interfaces).
* **Kube-State-Metrics:** Deployed as a `Deployment` to listen to the Kubernetes API server and generate metrics on cluster objects (pod phases, replica counts, resource limits).
* **Prometheus:** Scrapes both exporters and stores the unified metrics.
* **Grafana:** Provides pre-built dashboards visualizing both node-level and cluster-level states.
