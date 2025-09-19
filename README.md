# Architecture Overview

## JAG DevOps Infrastructure - Complete Production Architecture

This repository documents the complete architecture of a production DevOps infrastructure supporting forensic LIMS, financial trading, and pharmaceutical compliance applications.

---

## 📋 Infrastructure Summary

A 3-server hybrid infrastructure combining on-premise servers with AWS EC2, running production applications with comprehensive monitoring and forensic-grade compliance tracking.

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
        subgraph "Applications"
            LIMS[LIMS DNA System<br/>30007]
            FIN[Finance Trading<br/>30003]
            PHARMA[Pharma Dashboard<br/>30002]
        end
        subgraph "DevOps Tools"
            JENKINS[Jenkins<br/>30080]
            ARGO[ArgoCD<br/>30443]
            REG[Docker Registry<br/>5000]
        end
        PG1[PostgreSQL]
        PROM1[Prometheus Docker]
    end

    subgraph "Monitoring Server 2 - 192.168.50.74"
        GRAF[Grafana<br/>3000]
        LOKI[Loki]
        PROM2[Prometheus]
        CLIENT[Client LIMS Demo]
    end

    subgraph "AWS EC2 Instance"
        WEB[Portfolio Website<br/>Port 80]
        FORENSIC[Forensic Collector<br/>Port 9999]
        NODE[Node Exporter<br/>9100]
    end

    subgraph "Networking"
        TAIL[Tailscale VPN<br/>100.x.x.x Network]
    end

    Internet -->|HTTPS| CF
    CF -->|Tunnel| K3S
    Internet -->|HTTP| AWS
    
    K3S --> LIMS
    K3S --> FIN
    K3S --> PHARMA
    
    LIMS -.->|Logs| LOKI
    FIN -.->|Logs| LOKI
    PHARMA -.->|Logs| LOKI
    
    LIMS -.->|Metrics| PROM1
    FIN -.->|Metrics| PROM1
    PHARMA -.->|Metrics| PROM1
    
    PROM1 -.->|Federate| PROM2
    FORENSIC -.->|Compliance Metrics| PROM2
    
    PROM2 --> GRAF
    LOKI --> GRAF
    
    JENKINS -->|Build| REG
    REG -->|Deploy| K3S
    ARGO -->|GitOps| K3S
    
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
    
    class LIMS,FIN,PHARMA production
    class GRAF,LOKI,PROM2,PROM1 monitoring
    class JENKINS,ARGO,REG devops
    class AWS,WEB,FORENSIC aws
```

---

## 🖥️ Server Details

### **Server 1: Production Kubernetes (192.168.50.100)**
- **Role**: Main production environment
- **Services**:
  - K3s Kubernetes cluster
  - LIMS application (DNA forensic tracking)
  - Finance application (Trading platform)
  - Pharma application (Inventory management)
  - Jenkins CI/CD
  - ArgoCD GitOps
  - Docker Registry
  - PostgreSQL databases
  - Prometheus (Docker container)

### **Server 2: Monitoring Hub (192.168.50.74)**
- **Role**: Centralized monitoring and observability
- **Services**:
  - Grafana (visualization)
  - Prometheus (metrics aggregation)
  - Loki (log aggregation)
  - Client LIMS demo instance

### **EC2 Instance: Public Gateway & Forensics (13.218.244.32)**
- **Role**: Public-facing services and forensic monitoring
- **Services**:
  - Portfolio website (jagdevops.com)
  - Forensic evidence collector
  - Node exporter
  - Compliance monitoring

---

## 🌐 Network Architecture

```mermaid
graph LR
    subgraph "Public Domain Routing"
        A[lims.jagdevops.co.za] --> CF1[Cloudflare]
        B[finance.jagdevops.co.za] --> CF2[Cloudflare]
        C[pharma.jagdevops.co.za] --> CF3[Cloudflare]
        D[dashboard.jagdevops.co.za] --> CF4[Cloudflare]
    end
    
    CF1 --> P30007[Server1:30007]
    CF2 --> P30003[Server1:30003]
    CF3 --> P30002[Server1:30002]
    CF4 --> P30004[Server1:30004]
    
    E[jagdevops.com] --> EC2[EC2:80]
    
    subgraph "Internal Network"
        TAIL[Tailscale VPN]
        S1[100.89.26.128]
        S2[100.101.151.6]
        EC[100.101.99.93]
    end
    
    TAIL --- S1
    TAIL --- S2
    TAIL --- EC
```

---

## 🔄 Deployment Pipelines

### **LIMS Pipeline**
```
GitHub → Jenkins → Docker Build → Registry → K3s Deployment
```

### **Finance/Pharma Pipeline**
```
GitHub → ArgoCD → Auto-sync → K3s Rolling Update
```

---

## 📊 Monitoring Flow

```mermaid
sequenceDiagram
    participant App as Applications
    participant Prom1 as Prometheus (Server1)
    participant Forensic as Forensic Collector
    participant Prom2 as Prometheus (Server2)
    participant Grafana as Grafana
    
    App->>Prom1: Expose metrics
    App->>Loki: Send logs
    Forensic->>App: Monitor compliance
    Forensic->>Prom2: Export compliance metrics
    Prom1->>Prom2: Federate metrics
    Prom2->>Grafana: Provide data
    Loki->>Grafana: Provide logs
```

---

## 🔐 Security & Access

- **Public Access**: Via Cloudflare tunnels (HTTPS)
- **Internal Communication**: Tailscale VPN mesh
- **Service Mesh**: K3s internal networking
- **Authentication**: Grafana login, Jenkins auth, ArgoCD RBAC

---

## 📦 Applications Status

| Application | URL | Deployment Method | Replicas | Status |
|------------|-----|-------------------|----------|---------|
| LIMS | https://lims.jagdevops.co.za | Jenkins CI/CD | 1 | ✅ Operational |
| Finance | https://finance.jagdevops.co.za | ArgoCD GitOps | 2 | ✅ Operational |
| Pharma | https://pharma.jagdevops.co.za | ArgoCD GitOps | 2 | ✅ Operational |
| Dashboard | https://dashboard.jagdevops.co.za | Direct | 1 | ✅ Operational |
| Portfolio | https://jagdevops.com | Nginx | 1 | ✅ Operational |

---

## 🎯 Key Features

- **Zero-Downtime Deployments**: Rolling updates with health checks
- **GitOps**: Declarative infrastructure with ArgoCD
- **Forensic Monitoring**: DNA lab principles applied to DevOps
- **Multi-Environment**: Production, monitoring, and public-facing tiers
- **Compliance Tracking**: FDA 21 CFR Part 11, SOX, GMP standards

---

## 📂 Related Repositories

- [LIMS Application](https://github.com/GABRIELS562/JAG-LABSCIENTIFIC-DNA)
- [Zero-Downtime Pipeline](https://github.com/GABRIELS562/zero-downtime-pipeline)
- [Digital Evidence Pipeline](https://github.com/GABRIELS562/digital-evidence-pipeline)

---

## 👤 Author

**Gabriel S.**  
*DevOps Engineer | 15 Years Forensic Science Background*

---

*This architecture represents a production-grade DevOps infrastructure combining modern cloud-native practices with forensic-grade monitoring and compliance.*
