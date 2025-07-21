# 🐾 Datadog Learning Guide for Beginners: Monitoring with Docker on Windows

**Date:** July 21, 2025  
**Local Setup:** Windows + Docker Desktop  
**Goal:** Set up Datadog Agent using Docker, monitor an Nginx server, visualize data, and learn observability fundamentals.

---

## 🧠 What is Datadog?

Datadog is a powerful cloud-based monitoring and observability platform for servers, applications, containers, and services. It provides:

- **Metrics** (e.g., CPU, memory, requests/second)
- **Logs** (e.g., system/application events)
- **Traces** (e.g., journey of a request through microservices)

---

## 🎯 Why Use Datadog?

- **Unified Dashboard:** View logs, metrics, traces in one place.
- **Faster Troubleshooting:** Pinpoint issues in real-time.
- **Performance Insights:** Analyze behavior over time.
- **Alerting:** Be notified when systems fail.

---

## 🧩 Tools & Prerequisites

- ✅ Windows 10/11 with **Docker Desktop** installed and running.
- ✅ [Datadog Free Trial](https://www.datadoghq.com/free-trial/)
- ✅ Your API Key (e.g., `your_DD_api_key`)
- ✅ Your Datadog Site: `datadoghq.com`

---

## 🛠️ Step 1: Prepare Configuration Files

### 📁 Create Folder Structure

```powershell
mkdir C:\datadog-config
mkdir C:\datadog-config\conf.d\nginx.d
````

### 📝 `datadog.yaml`

`C:\datadog-config\datadog.yaml`:

```yaml
# Datadog Agent config
# Leave api_key blank as it's passed via Docker env vars
```

> 🔍 **Ensure it's saved as `.yaml`, not `.txt`.**

---

### 📝 `nginx_custom.conf`

`C:\datadog-config\nginx_custom.conf`:

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }

    location /nginx_status {
        stub_status on;
        allow 127.0.0.1;
        allow 172.16.0.0/12;
        deny all;
    }
}
```

---

### 📝 `conf.yaml` (Nginx Integration)

`C:\datadog-config\conf.d\nginx.d\conf.yaml`:

```yaml
init_config:

instances:
  - nginx_status_url: http://my-nginx/nginx_status
```

---

## ⚙️ Step 2: Setup Docker Environment

### 🚮 Cleanup Previous Containers

```bash
docker stop datadog-agent-local my-nginx 2>nul
docker rm datadog-agent-local my-nginx 2>nul
```

### 🌐 Create Docker Network

```bash
docker network create datadog-net
```

---

## 🧾 Step 3: Run Containers

### 🌍 Nginx Web Server

```bash
docker run -d --name my-nginx --network datadog-net -p 8080:80 `
-v C:/datadog-config/nginx_custom.conf:/etc/nginx/conf.d/datadog_nginx.conf:ro nginx
```

### 🐶 Datadog Agent

```bash
docker run -d --name datadog-agent-local --network datadog-net `
-v C:/datadog-config/:/etc/datadog-agent/ `
-e DD_API_KEY="your_DD_API_key" `
-e DD_SITE="datadoghq.com" `
-e DD_HOSTNAME="my-windows-docker-host" `
-e DD_APM_ENABLED=true `
-e DD_LOGS_ENABLED=true `
-e DD_LOGS_CONFIG_CONTAINER_COLLECT_ALL=true `
-e DD_PROCESS_AGENT_ENABLED=true `
-e DD_CONTAINER_EXCLUDE="name:datadog-agent-local" datadog/agent:latest
```

---

## 🔍 Step 4: Verify Setup

### ✅ Container Status

```bash
docker ps
```

### 📜 Agent Logs

```bash
docker logs datadog-agent-local
```

### 📊 Check Agent Status

```bash
docker exec datadog-agent-local /opt/datadog-agent/bin/agent/agent status
```

Look for `nginx` under **Running Checks** with `[OK]`.

---

## 🧑‍💻 Step 5: Explore Datadog Dashboard

* 🔗 Go to [Datadog App](https://app.datadoghq.com/)
* Navigate to:

  * **Infrastructure > Infrastructure List**
  * **Integrations > Nginx**
  * **Dashboards > New Dashboard**

### 📈 Add Custom Widgets

* Add Timeseries for:

  * `nginx.net.requests_per_second`
  * `nginx.connections.active`

---

## 🔁 Step 6: Add More Integrations (Optional)

### Redis Container

```bash
docker run -d --name my-redis --network datadog-net redis
```

Create `conf.yaml` at `C:\datadog-config\conf.d\redis.d\`:

```yaml
instances:
  - host: my-redis
    port: 6379
```

Check Redis metrics in Datadog dashboard.

---

## 🧪 Step 7: Send Custom Metrics (Optional)

### 🐍 Python Example

```python
# send_metrics.py
from datadog import statsd
statsd.socket = ("127.0.0.1", 8125)
statsd.gauge('my_app.users_online', 42, tags=['env:dev'])
```

Install & Run:

```bash
pip install datadog
python send_metrics.py
```

---

## 🚨 Step 8: Set Alerts (Monitor)

* Go to **Monitors > New Monitor**
* Choose **Metric**
* Set threshold (e.g., `nginx.net.requests_per_second < 1`)
* Configure notification (email)

---

## 🧹 Step 9: Cleanup

### 🛑 Stop Containers

```bash
docker stop datadog-agent-local my-nginx my-redis
```

### 🗑️ Remove Containers

```bash
docker rm datadog-agent-local my-nginx my-redis
```

### 🧼 Remove Network

```bash
docker network rm datadog-net
```

### 🧽 (Optional) Remove Images

```bash
docker rmi datadog/agent:latest nginx:latest redis:latest
```

---

## ✅ You're Done!

You’ve successfully installed and configured Datadog locally, monitored a web server, visualized metrics, and explored logs and traces. Keep practicing by adding more services, alerts, and dashboards!

Happy Monitoring! 🚀

