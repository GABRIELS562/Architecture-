Here's a cleaner, more professional architecture diagram using a combination of approaches that DevOps engineers commonly use:

# Architecture Overview

## JAG DevOps Infrastructure

### High-Level Architecture

```yaml
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET USERS                           │
└─────────────────────────────────────────────────────────────────┘
                    │                          │
                    ▼                          ▼
         ┌──────────────────┐       ┌──────────────────┐
         │  Cloudflare CDN  │       │   AWS EC2        │
         │  *.jagdevops.co.za│       │  jagdevops.com   │
         └──────────────────┘       └──────────────────┘
                    │                          │
         ┌──────────┴──────────┐              │
         │  Cloudflare Tunnel  │              │
         └─────────────────────┘              │
                    │                          │
    ┌───────────────▼───────────────┐         ▼
    │   SERVER 1 - PRODUCTION       │    EC2 INSTANCE
    │   192.168.50.100              │    13.218.244.32
    │                               │    ┌──────────────┐
    │   K3s Cluster:                │    │ • Website:80 │
    │   • LIMS      :30007          │    │ • Forensic:9999│
    │   • Finance   :30003          │    │ • Node Exp:9100│
    │   • Pharma    :30002          │    └──────────────┘
    │   • Dashboard :30004          │            │
    │   • ArgoCD    :30443          │            │
    │                               │            │
    │   Docker:                     │            │
    │   • Jenkins   :30080          │            │
    │   • Prometheus:9090           │            │
    │   • Registry  :5000           │            │
    └───────────────────────────────┘            │
                    │                             │
                    │ Logs & Metrics              │ Metrics
                    ▼                             ▼
    ┌───────────────────────────────┐    ┌──────────────┐
    │   SERVER 2 - MONITORING       │◄───│  Prometheus  │
    │   192.168.50.74               │    │   Scrape     │
    │                               │    └──────────────┘
    │   • Grafana   :3000           │
    │   • Prometheus:9090           │
    │   • Loki      :3100           │
    └───────────────────────────────┘
```

### Network Topology

```
External Access:
━━━━━━━━━━━━━━━
lims.jagdevops.co.za     ─┐
finance.jagdevops.co.za  ─┼─→ Cloudflare → Server1 (localhost:30xxx)
pharma.jagdevops.co.za   ─┤
dashboard.jagdevops.co.za─┘

jagdevops.com ──────────────→ EC2 Instance (Port 80)

Internal Network (Tailscale VPN):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Server 1:  100.89.26.128
Server 2:  100.101.151.6  
EC2:       100.101.99.93
```

### Service Distribution

| **Server 1 - Production** | **Server 2 - Monitoring** | **EC2 - Public** |
|---------------------------|---------------------------|------------------|
| **K3s Services:**         | **Monitoring Stack:**     | **Services:**    |
| ✓ LIMS Application        | ✓ Grafana Dashboard       | ✓ Portfolio Site |
| ✓ Finance Trading         | ✓ Prometheus Server       | ✓ Forensic Monitor |
| ✓ Pharma Management       | ✓ Loki Log Aggregation    | ✓ Node Exporter |
| ✓ Dashboard UI            |                           |                  |
| ✓ ArgoCD                  |                           |                  |
| ✓ PostgreSQL DBs          |                           |                  |
| ✓ Promtail                |                           |                  |
|                           |                           |                  |
| **Docker Services:**      |                           |                  |
| ✓ Jenkins CI/CD           |                           |                  |
| ✓ Prometheus              |                           |                  |
| ✓ Docker Registry         |                           |                  |

### Data Flow Diagram

```
┌──────────────────────────────────────────────────────┐
│                  APPLICATION LAYER                    │
├──────────────────────────────────────────────────────┤
│  LIMS │ Finance │ Pharma │ Dashboard                 │
└───┬──────────┬──────────┬──────────┬────────────────┘
    │          │          │          │
    ▼          ▼          ▼          ▼
┌──────────────────────────────────────────────────────┐
│                  ORCHESTRATION LAYER                  │
├──────────────────────────────────────────────────────┤
│         K3s Cluster (Server 1)                       │
└───┬──────────────────────────────────────────────────┘
    │
    ├─────[Logs via Promtail]────────→ Loki (Server 2)
    │
    ├─────[Metrics]──────────────────→ Prometheus (Server 1)
    │                                         │
    │                                         ▼
    │                                   Prometheus (Server 2)
    │                                         │
    └─────[Compliance Metrics]────────────────┤
          (From EC2 Forensic:9999)           │
                                              ▼
                                        Grafana (Server 2)
```

### CI/CD Pipeline Flow

```
GitHub Repository
      │
      ├──[Push]──→ Jenkins (Build #85) ──→ Docker Registry ──→ LIMS
      │
      └──[Push]──→ ArgoCD (Auto-sync) ──→ K3s ──→ Finance/Pharma
```

### Port Reference

```
Production (Server 1):
  30002 - Pharma App
  30003 - Finance App  
  30004 - Dashboard
  30007 - LIMS App
  30080 - Jenkins
  30443 - ArgoCD
  5000  - Docker Registry
  9090  - Prometheus

Monitoring (Server 2):
  3000  - Grafana
  3100  - Loki
  9090  - Prometheus

EC2 Instance:
  80    - Portfolio Website
  9100  - Node Exporter
  9999  - Forensic Collector
```

---

## 📊 Key Metrics

- **Applications**: 4 production apps
- **Uptime**: 99.9%+ availability
- **Deployment Method**: GitOps (ArgoCD) + CI/CD (Jenkins)
- **Monitoring**: Complete observability stack
- **Compliance**: FDA, SOX, GMP standards tracked

---

## 🔗 Repositories

- [LIMS](https://github.com/GABRIELS562/JAG-LABSCIENTIFIC-DNA)
- [Zero-Downtime](https://github.com/GABRIELS562/zero-downtime-pipeline)
- [Forensic Pipeline](https://github.com/GABRIELS562/digital-evidence-pipeline)

---

*Clean, production-grade DevOps infrastructure with forensic-level monitoring*
