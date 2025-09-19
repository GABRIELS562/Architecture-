You're right to double-check. Let me verify the exact network ports and services based on our actual conversation:

# Architecture Overview

## JAG DevOps Infrastructure - Complete Production Architecture

This repository documents the complete architecture of a production DevOps infrastructure supporting forensic LIMS, financial trading, and pharmaceutical compliance applications.

---

## 🏗️ Complete Architecture Diagram

```mermaid
graph TB
    subgraph "Public Access"
        Internet[Internet Users]
        CF[Cloudflare Tunnel]
        AWS[AWS EC2<br/>13.218.244.32]
    end

    subgraph "Production Server 1 - 192.168.50.100"
        K3S[K3s Cluster]
        subgraph "K3s Applications - NodePorts"
            LIMS[LIMS DNA System<br/>NodePort 30007]
            FIN[Finance Trading<br/>NodePort 30003]
            PHARMA[Pharma Dashboard<br/>NodePort 30002]
            DASH[Dashboard<br/>NodePort 30004]
        end
        subgraph "Docker Containers"
            JENKINS[Jenkins<br/>Port 30080]
            PROM_DOCKER[Prometheus<br/>Port 9090]
            REG[Docker Registry<br/>Port 5000]
        end
        ARGO[ArgoCD in K3s<br/>Port 30443]
        PG1[PostgreSQL in K3s]
        PROMTAIL[Promtail in K3s]
    end

    subgraph "Monitoring Server 2 - 192.168.50.74"
        GRAF[Grafana<br/>Port 3000]
        LOKI[Loki<br/>Port 3100]
        PROM2[Prometheus<br/>Port 9090]
    end

    subgraph "AWS EC2 Instance"
        WEB[Portfolio Website<br/>Port 80]
        FORENSIC[Forensic Collector<br/>Port 9999]
        NODE[Node Exporter<br/>Port 9100]
    end

    subgraph "Networking"
        TAIL[Tailscale VPN<br/>100.x.x.x Network]
    end

    Internet -->|HTTPS| CF
    CF -->|localhost:30xxx| K3S
    Internet -->|HTTP| AWS
    
    K3S --> LIMS
    K3S --> FIN
    K3S --> PHARMA
    K3S --> DASH
    
    PROMTAIL -.->|Collect Logs| K3S
    PROMTAIL -.->|Send to| LOKI
    
    LIMS -.->|Metrics| PROM_DOCKER
    FIN -.->|Metrics| PROM_DOCKER
    PHARMA -.->|Metrics| PROM_DOCKER
    
    FORENSIC -.->|Compliance Metrics<br/>Port 9999| PROM2
    
    PROM2 --> GRAF
    LOKI --> GRAF
    
    JENKINS -->|Build & Push| REG
    REG -->|Pull Images| K3S
    ARGO -->|GitOps Sync| K3S
    
    FORENSIC -.->|Monitor| LIMS
    FORENSIC -.->|Monitor| FIN
    FORENSIC -.->|Monitor| PHARMA
    
    TAIL -.->|VPN| K3S
    TAIL -.->|VPN| GRAF
    TAIL -.->|VPN| AWS

    classDef production fill:#2E7D32,stroke:#1B5E20,color:#fff
    classDef monitoring fill:#1565C0,stroke:#0D47A1,color:#fff
    classDef devops fill:#F57C00,stroke:#E65100,color:#fff
    classDef aws fill:#FF9800,stroke:#F57C00,color:#fff
    
    class LIMS,FIN,PHARMA,DASH production
    class GRAF,LOKI,PROM2 monitoring
    class JENKINS,ARGO,REG,PROM_DOCKER devops
    class AWS,WEB,FORENSIC aws
```

---

## 🖥️ Server Details

### **Server 1: Production Kubernetes (192.168.50.100)**
- **K3s Cluster Services (NodePorts)**:
  - LIMS: 30007
  - Finance: 30003
  - Pharma: 30002
  - Dashboard: 30004
  - ArgoCD: 30443
- **Docker Containers**:
  - Jenkins: 30080
  - Prometheus: 9090 (Docker, not K3s)
  - Docker Registry: 5000
- **K3s Internal Services**:
  - PostgreSQL databases
  - Promtail (log collector)
  - ArgoCD

### **Server 2: Monitoring Hub (192.168.50.74)**
- **Services**:
  - Grafana: 3000
  - Prometheus: 9090
  - Loki: 3100

### **EC2 Instance: Public & Forensics (13.218.244.32)**
- **Services**:
  - Portfolio website: 80
  - Forensic collector: 9999
  - Node exporter: 9100

---

## 🌐 Network Flow

```mermaid
graph LR
    subgraph "Cloudflare Tunnel Config"
        CF[~/.cloudflared/config.yml<br/>on Server1]
        CF --> |lims.jagdevops.co.za| L[localhost:30007]
        CF --> |finance.jagdevops.co.za| F[localhost:30003]
        CF --> |pharma.jagdevops.co.za| P[localhost:30002]
        CF --> |dashboard.jagdevops.co.za| D[localhost:30004]
    end
    
    subgraph "Direct Access"
        JAG[jagdevops.com] --> EC2[EC2:80]
    end
    
    subgraph "Tailscale IPs"
        S1[Server1: 100.89.26.128]
        S2[Server2: 100.101.151.6]
        EC[EC2: 100.101.99.93]
    end
```

---

## 📊 Monitoring Architecture

```mermaid
sequenceDiagram
    participant K3s as K3s Apps (Server1)
    participant Promtail as Promtail (Server1)
    participant PromD as Prometheus Docker (Server1)
    participant Forensic as Forensic (EC2:9999)
    participant Prom2 as Prometheus (Server2)
    participant Loki as Loki (Server2)
    participant Grafana as Grafana (Server2)
    
    K3s->>Promtail: Collect pod logs
    Promtail->>Loki: Ship logs
    K3s->>PromD: Expose metrics
    Forensic->>Forensic: Monitor compliance
    Forensic->>Prom2: Export metrics (9999)
    PromD-->>Prom2: Could federate
    Loki->>Grafana: Query logs
    Prom2->>Grafana: Query metrics
```

---

## 📦 Current Status

| Application | URL | Port | Location | Method |
|------------|-----|------|----------|---------|
| LIMS | https://lims.jagdevops.co.za | 30007 | K3s Server1 | Jenkins |
| Finance | https://finance.jagdevops.co.za | 30003 | K3s Server1 | ArgoCD |
| Pharma | https://pharma.jagdevops.co.za | 30002 | K3s Server1 | ArgoCD |
| Dashboard | https://dashboard.jagdevops.co.za | 30004 | K3s Server1 | Direct |
| Grafana | http://192.168.50.74:3000 | 3000 | Server2 | Direct |
| Forensic | Metrics endpoint | 9999 | EC2 | Python |
| Portfolio | https://jagdevops.com | 80 | EC2 | Nginx |

---

## 🔄 Data Flows

1. **Logs**: K3s Pods → Promtail → Loki (Server2) → Grafana
2. **Metrics**: Apps → Prometheus (Server1 Docker) & Forensic (EC2) → Prometheus (Server2) → Grafana
3. **Deployments**: GitHub → Jenkins/ArgoCD → Docker Registry → K3s

---

## 📂 Related Repositories

- [LIMS Application](https://github.com/GABRIELS562/JAG-LABSCIENTIFIC-DNA)
- [Zero-Downtime Pipeline](https://github.com/GABRIELS562/zero-downtime-pipeline)
- [Digital Evidence Pipeline](https://github.com/GABRIELS562/digital-evidence-pipeline)

---

*This architecture represents a production-grade DevOps infrastructure with forensic-grade monitoring.*
