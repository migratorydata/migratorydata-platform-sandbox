# MigratoryData Platform — Docker Deployment

A self-contained Docker Compose setup that runs the MigratoryData real-time messaging server, the MigratoryData Portal (admin UI), and a suite of live-data demo applications.

---

## Directory Tree

```
docker/
├── docker-compose.yaml          # Service definitions for the full stack
├── run.sh                       # Pull images and start all services
├── stop.sh                      # Stop all running services
├── README.md                    # This file
├── data/
│   └── db/                      # Persistent SQLite database volume (portal)
├── migratorydata/
│   └── migratorydata.conf       # MigratoryData server configuration
└── migratorydata-portal/
    └── migratorydata-portal.conf # Portal configuration
```

---

## Core Modules

### `migratorydata-server`
The MigratoryData real-time push server (`migratorydata-test:latest`). Handles WebSocket/HTTP connections from clients and pushes live updates to subscribers.

| Setting | Value |
|---|---|
| Exposed port | `8800` |
| Protocol | TCP (WebSocket / HTTP) |
| Memory allocation | 128 MB |
| Entitlement mode | Portal |
| REST API | Enabled |
| Stats log interval | 5 s |

Configuration file: [`migratorydata/migratorydata.conf`](migratorydata/migratorydata.conf)

---

### `migratorydata-portal`
Web-based administration portal (`portal-test:latest`). Manages cluster configuration, client token revocation, and provides the demo dashboard.

| Setting | Value |
|---|---|
| Exposed port | `8080` |
| Memory allocation | 512 MB |
| Database | SQLite (persisted under `data/db/`) |
| Default admin email | `admin@admin.com` |
| Default admin password | `password` |
| Auth signature | HMAC |
| TLS | Disabled (demo mode) |

Configuration file: [`migratorydata-portal/migratorydata-portal.conf`](migratorydata-portal/migratorydata-portal.conf)

---

### Demo Data Publishers

Six demo services continuously publish live data to the MigratoryData server, providing ready-made subjects for demonstration purposes.

---

## Prerequisites

- [Docker Engine](https://docs.docker.com/engine/install/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) v2 (bundled with Docker Desktop)

Verify your installation:

```bash
docker --version
docker compose version
```

---

## Setup Instructions

### 1. Clone the repository (if not already done)

```bash
git clone <repository-url>
cd migratorydata-platforma-deployment-demo/docker
```

### 2. (Optional) Review configuration files

Before starting, inspect and adjust configuration as needed:

```bash
# MigratoryData server
cat migratorydata/migratorydata.conf

# Portal
cat migratorydata-portal/migratorydata-portal.conf
```

> **Security note:** The files ship with default credentials and keys intended for local demo use only. Change `portal.admin.password`, `portal.gateway.access.password`, and the HMAC secret before deploying in any non-local environment.

### 3. Start the stack

To run in detached (background) mode:

```bash
docker compose up -d
```

### 4. Stop the stack

Stop and remove containers:

```bash
docker compose down
```

---

## Getting Started

### Open the Portal

Once the stack is running, open your browser and navigate to:

```
http://127.0.0.1:8080
```

Log in with the default credentials:

| Field | Value |
|---|---|
| Email | `admin@admin.com` |
| Password | `password` |

The portal dashboard displays live charts for all six demo data streams.

---

### Inspect Running Services

```bash
# List all running containers
docker compose ps

# Tail logs from the server
docker compose logs -f migratorydata-server

# Tail logs from the portal
docker compose logs -f migratorydata-portal
```

---

## Network Architecture

All services communicate on an internal Docker bridge network (`local-network`). Only the following ports are published to the host:

| Port | Service |
|---|---|
| `8080` | MigratoryData Portal (admin UI) |
| `8800` | MigratoryData Server (client connections) |
