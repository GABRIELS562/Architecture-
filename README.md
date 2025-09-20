Yes, absolutely add those Grafana dashboard screenshots! Looking at your screenshot, you have excellent monitoring dashboards that demonstrate production-grade infrastructure. Here's the updated README with screenshot placeholders:

# Architecture Overview

## JAG DevOps Infrastructure

### 🏗️ Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              INTERNET USERS                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                    │                                    │
                    ▼                                    ▼
    ┌───────────────────────────────┐    ┌──────────────────────────────┐
    │     CLOUDFLARE CDN/TUNNEL     │    │         AWS EC2              │
    │    *.jagdevops.co.za          │    │     jagdevops.com            │
    └───────────────────────────────┘    └──────────────────────────────┘
                    │                                    │
                    │                                    │
    ┌───────────────▼───────────────┐    ┌──────────────▼───────────────┐
    │   SERVER 1 - PRODUCTION       │    │   EC2 - PUBLIC FACING        │
    │   IP: 192.168.50.100          │    │   IP: 13.218.244.32          │
    │   Tailscale: 100.89.26.128    │    │   Tailscale: 100.101.99.93   │
    │                               │    │                              │
    │   ┌─────────────────────┐    │    │   ┌────────────────────┐    │
    │   │   K3s CLUSTER       │    │    │   │  Python Process    │    │
    │   │  ├─ LIMS:30007      │    │    │   │  └─ Forensic:9999  │    │
    │   │  ├─ Finance:30003   │    │    │   │  Nginx Container    │    │
    │   │  ├─ Pharma:30002    │    │    │   │  └─ Website:80     │    │
    │   │  ├─ Dashboard:30004 │    │    │   │  Node Exporter      │    │
    │   │  ├─ ArgoCD:30443    │    │    │   │  └─ Metrics:9100   │    │
    │   │  ├─ PostgreSQL      │    │    │   └────────────────────┘    │
    │   │  └─ Promtail        │    │    └──────────────────────────────┘
    │   └─────────────────────┘    │                    │
    │                               │                    │
    │   ┌─────────────────────┐    │                    │
    │   │  DOCKER SERVICES    │    │                    │
    │   │  ├─ Jenkins:30080   │    │                    │
    │   │  ├─ Prometheus:9090  │    │                    │
    │   │  └─ Registry:5000    │    │                    │
    │   └─────────────────────┘    │                    │
    └───────────────────────────────┘                    │
                    │                                     │
                    │ Logs & Metrics                     │ Metrics
                    ▼                                     ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                    SERVER 2 - MONITORING                        │
    │                    IP: 192.168.50.74                            │
    │                    Tailscale: 100.101.151.6                     │
    │                                                                 │
    │    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
    │    │ Grafana:3000 │  │ Prometheus   │  │ Loki:3100    │      │
    │    │              │◄─│ :9090        │◄─│              │      │
    │    └──────────────┘  └──────────────┘  └──────────────┘      │
    └─────────────────────────────────────────────────────────────────┘
```

### 📊 Monitoring Dashboards

![Infrastructure Alerts](docs/screenshots/infrastructure-alerts.png)
*Real-time infrastructure alerts and container monitoring*

![LIMS Dashboard](docs/screenshots/lims-dashboard.png)
*LIMS sample processing pipeline monitoring*

![Regulatory Compliance](docs/screenshots/regulatory-compliance.png)
*Forensic compliance scores for FDA, SOX, and GMP*

![Server Performance](docs/screenshots/server-performance.png)
*Server 1 & 2 CPU and memory usage monitoring*

![Zero Downtime Pipeline](docs/screenshots/zero-downtime-pipeline.png)
*Finance and Pharma deployment monitoring*

### 📊 Service Architecture Matrix

| **Component** | **Server 1** | **Server 2** | **EC2** | **Port** | **Access** |
|--------------|-------------|-------------|---------|----------|------------|
| LIMS | ✅ K3s | - | - | 30007 | CloudFlare |
| Finance | ✅ K3s | - | - | 30003 | CloudFlare |
| Pharma | ✅ K3s | - | - | 30002 | CloudFlare |
| Dashboard | ✅ K3s | - | - | 30004 | CloudFlare |
| Jenkins | ✅ Docker | - | - | 30080 | Internal |
| ArgoCD | ✅ K3s | - | - | 30443 | Internal |
| Prometheus | ✅ Docker | ✅ | - | 9090 | Internal |
| Grafana | - | ✅ | - | 3000 | Direct IP |
| Loki | - | ✅ | - | 3100 | Internal |
| Promtail | ✅ K3s | - | - | - | Pushes logs |
| Forensic | - | - | ✅ Python | 9999 | Tailscale |
| Portfolio | - | - | ✅ Nginx | 80 | Public |

### 🔄 Data Flow Paths

```
APPLICATION LOGS FLOW:
─────────────────────
K3s Pods (Server1) → Promtail (Server1) → Loki (Server2) → Grafana (Server2)

METRICS FLOW:
────────────
K3s Apps (Server1) → Prometheus Docker (Server1) ─┐
                                                   ├→ Prometheus (Server2) → Grafana
EC2 Forensic (9999) ──────────────────────────────┘

DEPLOYMENT FLOW:
───────────────
GitHub → Jenkins → Registry → kubectl → LIMS
GitHub → Jenkins → Registry → ArgoCD → Finance/Pharma
```

### 🌐 Network Routing

```yaml
Public Domain Routing via CloudFlare:
  lims.jagdevops.co.za      → localhost:30007
  finance.jagdevops.co.za   → localhost:30003
  pharma.jagdevops.co.za    → localhost:30002
  dashboard.jagdevops.co.za → localhost:30004

Direct Access:
  jagdevops.com → EC2:80
  Grafana       → 192.168.50.74:3000

Internal VPN (Tailscale):
  Server1: 100.89.26.128
  Server2: 100.101.151.6
  EC2:     100.101.99.93
```

### 📁 Repository Structure

```
GitHub Repositories:
├── JAG-LABSCIENTIFIC-DNA/        # LIMS application
│   ├── backend/                  # Node.js API
│   ├── frontend/                 # React UI
│   └── k8s/                      # Kubernetes manifests
│
├── zero-downtime-pipeline/       # Finance & Pharma
│   ├── k8s/                      # K8s manifests
│   ├── finance-deployment.yaml
│   └── pharma-deployment.yaml
│
├── digital-evidence-pipeline/    # Forensic monitoring
│   ├── forensic_complete.py      # Running process (EC2)
│   └── scripts/                  # Supporting scripts
│
└── architecture-overview/         # This documentation
    └── docs/screenshots/          # Grafana dashboards
```

### 🚀 CI/CD Pipelines

| App | Source | Build | Deploy | Method |
|-----|--------|-------|---------|---------|
| LIMS | GitHub | Jenkins #85 | K3s | Direct kubectl |
| Finance | GitHub | Jenkins → Registry | ArgoCD → K3s | GitOps |
| Pharma | GitHub | Jenkins → Registry | ArgoCD → K3s | GitOps |
| Dashboard | GitHub | Direct | K3s | Manual |
| Forensic | GitHub | None | EC2 | Python script |

### 📈 Monitoring Coverage

**Available Grafana Dashboards:**
- Infrastructure Alerts
- All Container Logs
- All Services Status
- Finance App Activity
- Finance Request Rate
- LIMS Dashboard
- LIMS Live Sample Pipeline Flow
- LIMS Processing Rate
- Pharma Equipment Logs
- Pharma Processing Rate
- Regulatory Compliance Scores
- Server 1 CPU Usage
- Server 1 Disk Usage
- Server 1 Memory Usage
- Server 2 CPU Usage
- Server 2 Memory Usage
- Zero-Downtime Pipeline (Pharma & Finance)

### 📊 Current Production Status

```
✅ All Systems Operational

LIMS:      https://lims.jagdevops.co.za      [1 replica]
Finance:   https://finance.jagdevops.co.za   [2 replicas]
Pharma:    https://pharma.jagdevops.co.za    [2 replicas]
Dashboard: https://dashboard.jagdevops.co.za  [1 replica]
Portfolio: https://jagdevops.com              [nginx]
Grafana:   http://192.168.50.74:3000         [18 dashboards]

Forensic Compliance Scores:
- LIMS Chain Integrity: Active
- GMP Compliance: 94%
- SOX Compliance: Monitored
- Overall Score: 92/100
```

### 🔐 Infrastructure Details

**Server 1 (Production)**
- Ubuntu 24.04 LTS
- K3s v1.27.4+k3s1
- 8GB RAM, 4 CPUs
- Hosts all production apps

**Server 2 (Monitoring)**
- Ubuntu 24.04 LTS
- Dedicated monitoring stack
- 4GB RAM, 2 CPUs
- 18 Grafana dashboards

**EC2 Instance (AWS Mumbai)**
- Ubuntu 24.04 LTS
- t2.micro (1GB RAM)
- Public-facing services
- Forensic monitoring

---

## 🔗 Repositories

- [LIMS Application](https://github.com/GABRIELS562/JAG-LABSCIENTIFIC-DNA)
- [Zero-Downtime Pipeline](https://github.com/GABRIELS562/zero-downtime-pipeline)
- [Digital Evidence Pipeline](https://github.com/GABRIELS562/digital-evidence-pipeline)

---

## Author

**Jaime Gabriels**  
*DevOps Engineer*

---

*Production-grade infrastructure with comprehensive monitoring through 18 Grafana dashboards tracking every aspect of the system*
