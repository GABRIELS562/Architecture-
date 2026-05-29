# Infrastructure Architecture

Production-grade DevOps infrastructure showcasing Kubernetes orchestration, GitOps deployment, and microservices architecture.

[![Kubernetes](https://img.shields.io/badge/Kubernetes-K3s-326CE5?logo=kubernetes&logoColor=white)](https://k3s.io/)
[![GitHub Actions](https://img.shields.io/badge/CI-GitHub_Actions-2088FF?logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![ArgoCD](https://img.shields.io/badge/CD-ArgoCD-EF7B4D?logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![Docker](https://img.shields.io/badge/Containers-Docker-2496ED?logo=docker&logoColor=white)](https://docker.com/)

---

## Architecture Overview

```mermaid
flowchart TB
    subgraph Internet["☁️ Public Access"]
        Users(("Users"))
    end

    subgraph CDN["Cloudflare Tunnels"]
        CF["Zero-Trust Access"]
    end

    subgraph Infra["Production Infrastructure"]
        subgraph S1["Kubernetes Cluster"]
            direction TB
            subgraph Apps["Applications"]
                LIMS["LIMS<br/>Lab Management"]
                ESHOP["eShop<br/>Microservices"]
            end
            subgraph Platform["Platform"]
                ARGO["ArgoCD"]
                TRAEFIK["Traefik"]
                ESO["External Secrets"]
            end
        end

        subgraph S2["Docker Host"]
            direction TB
            FORENSIC["Forensic Collector"]
            DASHBOARD["Command Center"]
            VAULT["HashiCorp Vault"]
        end
    end

    Users --> CF --> Infra
    ESO -.->|secrets| VAULT
    ARGO -->|GitOps| Apps
```

---

## CI/CD Pipeline

Full GitOps workflow using **GitHub Actions** for CI and **ArgoCD** for CD.

```mermaid
flowchart LR
    subgraph Developer
        CODE["Code Push"]
    end

    subgraph GHA["GitHub Actions (CI)"]
        BUILD["Build & Test"]
        PUSH["Push to GHCR"]
        VALIDATE["Validate Helm"]
    end

    subgraph Registry["Container Registry"]
        GHCR["ghcr.io"]
    end

    subgraph ARGO["ArgoCD (CD)"]
        SYNC["Sync Application"]
        HEALTH["Health Check"]
    end

    subgraph K8s["Kubernetes"]
        DEPLOY["Deploy Pods"]
    end

    CODE --> BUILD --> PUSH --> GHCR
    BUILD --> VALIDATE
    GHCR --> SYNC --> HEALTH --> DEPLOY
    VALIDATE -.->|trigger| SYNC
```

### Pipeline by Project

| Project | CI (GitHub Actions) | CD (ArgoCD) | Environments |
|---------|---------------------|-------------|--------------|
| **LIMS** | Build, Push to GHCR, Smoke Tests | Helm sync, Health checks | Production, Test, Develop |
| **eShop** | Validate Helm, PR checks | Sync 17 applications | Production |
| **Forensic** | — | — | Docker (manual) |

---

## Projects

### 1. LIMS - Laboratory Information Management System

Full-stack DNA sample tracking system with multi-environment deployment.

```mermaid
flowchart LR
    subgraph CI["GitHub Actions"]
        BUILD["ci.yml<br/>Build & Test"]
        DEPLOY_P["deploy-production.yml"]
        DEPLOY_T["deploy-test.yml"]
        DEPLOY_D["deploy-develop.yml"]
    end

    subgraph Registry
        GHCR["ghcr.io/gabriels562"]
    end

    subgraph CD["ArgoCD"]
        APP["lims application<br/>Helm + values"]
    end

    subgraph K8s["Kubernetes Namespaces"]
        PROD["production"]
        TEST["test"]
        DEV["develop"]
    end

    BUILD -->|push| GHCR
    DEPLOY_P & DEPLOY_T & DEPLOY_D -->|trigger| APP
    APP -->|sync| PROD & TEST & DEV
```

**Workflows (6):**
- `ci.yml` — Build, test, push images to GHCR
- `deploy-production.yml` — Production deployment + smoke tests
- `deploy-test.yml` — Test environment deployment
- `deploy-develop.yml` — Development environment deployment
- `cleanup.yml` — Resource cleanup
- `load-test.yml` — Locust load testing

| Component | Technology |
|-----------|------------|
| Frontend | React 18, Vite |
| Backend | Node.js, Express |
| Database | PostgreSQL |
| Deployment | Helm, ArgoCD |

**Live:** [lims.jagdevops.co.za](https://lims.jagdevops.co.za)

**Repository:** [JAG-LABSCIENTIFIC-DNA](https://github.com/GABRIELS562/JAG-LABSCIENTIFIC-DNA)

---

### 2. eShop - Cloud-Native Microservices

Polyglot microservices e-commerce platform with 17 ArgoCD-managed applications.

```mermaid
flowchart LR
    subgraph CI["GitHub Actions"]
        VAL["pr-validation.yml<br/>Helm lint"]
        SVC["services-ci.yml"]
        PROD["production-deploy.yml"]
    end

    subgraph CD["ArgoCD (17 Apps)"]
        direction TB
        FE["frontend"]
        CART["cartservice"]
        CHECK["checkoutservice"]
        MORE["...14 more"]
    end

    subgraph K8s["eshop namespace"]
        PODS["Microservices"]
        DATA["Redis + RabbitMQ + PostgreSQL"]
    end

    VAL & SVC --> PROD -->|sync all| CD --> K8s
```

**Workflows (5):**
- `pr-validation.yml` — Helm lint and template validation
- `services-ci.yml` — Service-level CI
- `production-deploy.yml` — ArgoCD sync trigger
- `develop-ci.yml` — Development CI
- `reusable-build.yml` — Shared build workflow

**ArgoCD Applications (17):**
| Services | Infrastructure |
|----------|----------------|
| frontend, cartservice, checkoutservice | postgresql, redis, redis-cart |
| productcatalogservice, currencyservice | rabbitmq, seq |
| shippingservice, paymentservice | otel-collector, loadgenerator |
| emailservice, recommendationservice, adservice | |

| Layer | Technologies |
|-------|--------------|
| Services | Go, C#, Node.js, Python, Java |
| Communication | gRPC, Protocol Buffers |
| Data | Redis, RabbitMQ, PostgreSQL |
| Observability | OpenTelemetry, Prometheus |

**Live:** [eshop.jagdevops.co.za](https://eshop.jagdevops.co.za)

**Repository:** [eshop-platform-infra](https://github.com/GABRIELS562/eshop-platform-infra)

---

### 3. Forensic Evidence Collector

Compliance automation platform with tamper-evident audit trails. Deployed via Docker on Server 2.

```mermaid
flowchart LR
    subgraph Server2["Docker Host"]
        API["Python API"]
        HASH["SHA-256<br/>Hash Chain"]
        DB["SQLite"]
        DASH["nginx Dashboard"]
    end

    API --> HASH --> DB
    DASH -.->|visualize| API
```

| Component | Technology |
|-----------|------------|
| Collector | Python 3.11 |
| Integrity | SHA-256 hash chains |
| Dashboard | Chart.js, nginx |
| Metrics | Prometheus-compatible |

**Live:** [dashboards.jagdevops.co.za](https://dashboards.jagdevops.co.za)

**Repository:** [forensic-evidence-collector](https://github.com/GABRIELS562/forensic-evidence-collector)

---

## Technology Stack

### CI/CD
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?logo=githubactions&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?logo=argo&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=white)

### Orchestration
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Traefik](https://img.shields.io/badge/Traefik-24A1C1?logo=traefikproxy&logoColor=white)

### Infrastructure & Security
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?logo=cloudflare&logoColor=white)
![Vault](https://img.shields.io/badge/Vault-FFEC6E?logo=vault&logoColor=black)

### Observability
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?logo=opentelemetry&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white)

### Languages
![Go](https://img.shields.io/badge/Go-00ADD8?logo=go&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?logo=react&logoColor=black)
![C#](https://img.shields.io/badge/C%23-512BD4?logo=csharp&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?logo=openjdk&logoColor=white)

### Data & Messaging
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?logo=rabbitmq&logoColor=white)

---

## DevOps Practices

| Practice | Implementation |
|----------|----------------|
| **CI** | GitHub Actions — build, test, push images |
| **CD** | ArgoCD — GitOps sync with self-healing |
| **Secrets** | HashiCorp Vault + External Secrets Operator |
| **Networking** | Cloudflare Tunnels (zero-trust) |
| **Multi-Environment** | Production, Test, Develop namespaces |
| **Observability** | OpenTelemetry + Prometheus |
| **Load Testing** | Locust distributed testing |

---

## Monitoring & Observability

```
┌─────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY STACK                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   eShop     │    │    LIMS     │    │  Forensic   │         │
│  │ Services    │    │   App       │    │  Collector  │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         ▼                  ▼                  ▼                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │              OpenTelemetry Collector                │       │
│  │         (traces, metrics, logs)                     │       │
│  └──────────────────────┬──────────────────────────────┘       │
│                         │                                       │
│         ┌───────────────┼───────────────┐                      │
│         ▼               ▼               ▼                      │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐                │
│  │Prometheus │   │   Seq     │   │   Loki    │                │
│  │ (metrics) │   │  (logs)   │   │  (logs)   │                │
│  └───────────┘   └───────────┘   └───────────┘                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

| Component | Purpose | Project |
|-----------|---------|---------|
| **OpenTelemetry** | Distributed tracing & metrics | eShop |
| **Prometheus** | Metrics collection & alerting | All |
| **Seq** | Structured log aggregation | eShop |
| **Locust** | Load testing with exporters | LIMS |
| **Health Checks** | Kubernetes probes | All |

---

## Repositories

| Project | Description | Link |
|---------|-------------|------|
| **LIMS** | Lab management system | [View →](https://github.com/GABRIELS562/JAG-LABSCIENTIFIC-DNA) |
| **eShop Infrastructure** | Kubernetes manifests | [View →](https://github.com/GABRIELS562/eshop-platform-infra) |
| **Forensic Collector** | Evidence automation | [View →](https://github.com/GABRIELS562/forensic-evidence-collector) |
| **Portfolio** | Personal website | [View →](https://github.com/GABRIELS562/JAIME-GABRIELS-PORTFOLIO-WEBSITE) |

---

## Author

**Jaime Gabriels** — DevOps Engineer

[![Portfolio](https://img.shields.io/badge/Portfolio-jagdevops.co.za-000000?style=for-the-badge)](https://jagdevops.co.za)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/jaime-gabriels-643132386)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github)](https://github.com/GABRIELS562)
