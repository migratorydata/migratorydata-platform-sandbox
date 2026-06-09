# MigratoryData Platform — Deployment Demo

This repository provides ready-to-use deployment configurations for the MigratoryData real-time messaging platform. Two deployment options are available: a **Docker Compose** setup for local development and a **Kubernetes** setup for cluster environments.

---

## Components

| Component | Description | Port |
|---|---|---|
| **MigratoryData Server** | Real-time push server. Handles WebSocket/HTTP client connections and delivers live messages to subscribers. | `8800` |
| **MigratoryData Portal** | Web-based administration UI. Manages cluster configuration, token lifecycle, and hosts the live demo dashboard. | `8080` |
| **Demo Publishers** | Five data publishers (stocks, traffic, cryptocurrency, parking, seismic) that continuously stream live data for demonstration. | — |

---

## Deployment Options

### Option 1 — Docker Compose

Best for: **local development, quick evaluation.**

**Prerequisites:** Docker Engine 20.10+, Docker Compose v2.

```bash
cd docker

# Start the full stack (pulls images on first run)
docker compose up -d

# Stop the stack
docker compose down
```

Once running, open the Portal at **http://127.0.0.1:8080** (`admin@admin.com` / `password`).

See [docker/README.md](docker/README.md) for the full setup guide, configuration reference, and available demo subjects.

---

### Option 2 — Kubernetes

Best for: **cluster deployments, Minikube-based local testing.**

**Prerequisites:** `kubectl`, Minikube (for local use).

```bash
# 1. Create the namespace
kubectl create namespace migratorydata
kubectl config set-context --current --namespace=migratorydata

# 2. Deploy Portal, Server, and demos
kubectl apply -f kafkorama/01-portal.yaml
kubectl apply -f kafkorama/02-migratorydata.yaml
kubectl apply -f kafkorama/03-demos.yaml

# 3. Expose LoadBalancer services (Minikube only)
minikube tunnel
```

Once running, open the Portal at **http://127.0.0.1:8080** (`admin@admin.com` / `password`).  
The MigratoryData Server is reachable at **127.0.0.1:8800**.

See [kafkorama/README.md](kafkorama/README.md) for the full setup guide, manifest descriptions, and operational commands.

---

## Security Notice

All configuration files ship with default credentials and demo keys intended for **local use only**. Before any non-local or production deployment, change the following in both `migratorydata.conf` and `migratorydata-portal.conf`:

- `portal.admin.password`
- `portal.gateway.access.password` / `Portal.Password`
