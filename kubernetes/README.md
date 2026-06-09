## MigratoryData Platform — Kubernetes Deployment

Deploys the complete MigratoryData real-time messaging stack — Portal, MigratoryData Server, and live-data demo publishers — on Kubernetes using a `migratorydata` namespace.

> Reference: [Deploy on Kubernetes — MigratoryData Docs](https://migratorydata.com/docs/portal/deploy/kubernetes/)

---

### Directory Tree

```
kubernetes/
├── 01-portal.yaml          # MigratoryData Portal (Services, PVC, ConfigMap, Deployment)
├── 02-migratorydata.yaml   # MigratoryData Server (Service, ConfigMap, Deployment)
├── 03-demos.yaml           # Demo data publishers (stocks, traffic, crypto, parking, seismic)
└── README.md               # This file
```

---

### Architecture Overview

```
                    ┌─────────────────────────────────┐
                    │        migratorydata namespace   │
                    │                                  │
  Browser / SDK ───►│  migratorydata-portal-external   │
       :8080        │  (LoadBalancer)                  │
                    │            │                     │
                    │    migratorydata-portal           │
                    │    (Deployment + PVC 5Gi)         │
                    │            │ internal             │
                    │  migratorydata-portal-internal   │
                    │  (ClusterIP)                     │
                    │            │                     │
  Client SDK ──────►│  migratorydata-server-external   │
       :8800        │  (LoadBalancer)                  │
                    │            │                     │
                    │    migratorydata-server           │
                    │    (Deployment)                  │
                    │                                  │
                    │    demo-* (5 Deployments)        │
                    └─────────────────────────────────┘
```

| Service | Type | Port | Purpose |
|---|---|---|---|
| `migratorydata-portal-external` | LoadBalancer | `8080` | Portal web UI — browser access |
| `migratorydata-portal-internal` | ClusterIP | `8080` | Portal — internal cluster communication |
| `migratorydata-server-external` | LoadBalancer | `8800` | MigratoryData Server — client SDK connections |

---

### Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured for your target cluster
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) for local development

Verify your setup:

```bash
kubectl version --client
minikube version
```

---

### Setup Instructions

#### 1. Start Minikube (local development only)

```bash
minikube start
```

Optionally, open the Kubernetes dashboard:

```bash
minikube dashboard
```

#### 2. Create the namespace

All resources are deployed into the `migratorydata` namespace:

```bash
kubectl create namespace migratorydata
```

Switch your active context to the new namespace so subsequent commands default to it:

```bash
kubectl config set-context --current --namespace=migratorydata
```

#### 3. Deploy the Portal

Deploys the MigratoryData Portal with its external and internal Services, a 5 Gi PersistentVolumeClaim for the SQLite database, a ConfigMap for configuration, and the Portal Deployment.

```bash
kubectl apply -f 01-portal.yaml
```

> **Note:** The `storageClassName: local-path` in the PVC spec may need to be adjusted to match the storage classes available in your cluster. Run `kubectl get storageclass` to list available classes.

#### 4. Deploy the MigratoryData Server

Deploys the MigratoryData Server with a LoadBalancer Service and a ConfigMap-backed configuration:

```bash
kubectl apply -f 02-migratorydata.yaml
```

#### 5. Deploy Demo Applications (optional)

Deploys five live-data publishers that continuously stream real-time data to the MigratoryData Server for demonstration purposes:

```bash
kubectl apply -f 03-demos.yaml
```

#### 6. Expose LoadBalancer Services (Minikube only)

Minikube does not automatically assign external IPs to LoadBalancer services. Run the tunnel in a separate terminal to enable access:

```bash
minikube tunnel
```

---

### Verify Installation

Check that all pods reach `Running` status:

```bash
kubectl get pods
```

Expected output:

```
NAME                                     READY   STATUS    RESTARTS   AGE
migratorydata-portal-5c94794f9-gzxbs     1/1     Running   0          3m
migratorydata-server-bd4c75658-5rhx8     1/1     Running   0          2m
demo-stocks-7bdd96c4fc-6lps5             1/1     Running   0          1m
demo-traffic-6d4d5d4868-vkrv9            1/1     Running   0          1m
demo-cryptocurrency-846789989b-wctzl     1/1     Running   0          1m
demo-parking-8679cc64fb-xtl2h            1/1     Running   0          1m
demo-seismic-76479b5d55-8m9j6            1/1     Running   0          1m
```

Verify Services and their external IPs:

```bash
kubectl get svc
```

Expected output:

```
NAME                              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
migratorydata-portal-external     LoadBalancer   10.43.158.123   127.0.0.1     8080:30856/TCP   3m
migratorydata-portal-internal     ClusterIP      10.43.199.88    <none>        8080/TCP         3m
migratorydata-server-external     LoadBalancer   10.43.237.196   127.0.0.1     8800:31735/TCP   2m
```

---

### Getting Started

#### Open the Portal

Navigate to the Portal in your browser:

```
http://127.0.0.1:8080
```

Log in with the default credentials defined in the ConfigMap inside `01-portal.yaml`:

| Field | Value |
|---|---|
| Email | `admin@admin.com` |
| Password | `password` |

> **Security:** Change the default admin password and all shared secrets before any non-local deployment.


---

### Useful Commands

#### View logs

```bash
# Portal logs
kubectl logs -f deployment/migratorydata-portal

# MigratoryData Server logs
kubectl logs -f deployment/migratorydata-server
```

#### Inspect a pod

```bash
kubectl describe pod <pod-name>
```

#### Open a shell inside a pod

```bash
kubectl exec -it <pod-name> -- /bin/bash
```

#### Update configuration

Configuration is stored in ConfigMaps inside each manifest. To apply a config change:

1. Edit the relevant `ConfigMap` section in the YAML file.
2. Re-apply the manifest:

```bash
kubectl apply -f 01-portal.yaml   # for portal config changes
kubectl apply -f 02-migratorydata.yaml  # for server config changes
```

3. Restart the affected deployment to pick up the new ConfigMap:

```bash
kubectl rollout restart deployment/migratorydata-portal
kubectl rollout restart deployment/migratorydata-server
```

---

### Uninstall

Remove all resources by deleting the namespace:

```bash
kubectl delete namespace migratorydata
```

Then restore the default namespace context:

```bash
kubectl config set-context --current --namespace=default
```
