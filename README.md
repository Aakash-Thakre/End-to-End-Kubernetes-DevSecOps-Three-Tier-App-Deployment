# 🚀 End-to-End Kubernetes DevSecOps — Three-Tier App Deployment

<div align="center">

![Kubernetes](https://img.shields.io/badge/Kubernetes-1.32-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Multi--Stage-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)

**A production-grade DevSecOps project deploying the Jerney Blog Platform on AWS EKS.**  
Security-first. Infrastructure as Code. Fully automated CI/CD.

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Infrastructure — Terraform](#-infrastructure--terraform)
- [Dockerization](#-dockerization)
- [Security](#-security)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Prerequisites](#-prerequisites)

---

## 📖 Overview

This project demonstrates a complete **DevSecOps lifecycle** for the **Jerney Blog Platform** — a three-tier web application. It covers containerizing with Docker, provisioning cloud infrastructure with Terraform, deploying on Kubernetes with production-grade manifests, and automating delivery through GitHub Actions.

Security is embedded at every layer — non-root containers, read-only filesystems, NetworkPolicies, encrypted EBS storage, and least-privilege IAM.

---

## ⚙️ CI/CD Pipeline

GitHub Actions automates build → scan → push → deploy on every push to `main`.

```
Push to main
     │
     ▼
┌──────────┐   ┌──────────────┐   ┌─────────────┐   ┌───────────┐
│ Build &  │──►│    Docker    │──►│   Trivy     │──►│ Deploy to │
│  Lint    │   │  Build+Push  │   │ Image Scan  │   │    EKS    │
└──────────┘   └──────────────┘   └─────────────┘   └───────────┘
```

## 🏗 Architecture

```
                    ┌──────────────────────────────────────────────────┐
                    │                   AWS Cloud                      │
                    │                                                  │
  User Browser      │  ┌──────────────────────────────────────────┐   │
       │            │  │           AWS EKS Cluster (v1.32)        │   │
       ▼            │  │                                          │   │
  ┌─────────┐       │  │  ┌─────────┐     ┌──────────────────┐   │   │
  │ Internet│──────►│  │  │   ALB   │────►│ jerney Namespace │   │   │
  └─────────┘       │  │  │ Ingress │     │                  │   │   │
                    │  │  └─────────┘     │  ┌────────────┐  │   │   │
                    │  │                  │  │  Frontend  │  │   │   │
                    │  │                  │  │ React+Nginx│  │   │   │
                    │  │                  │  │  x2 Pods   │  │   │   │
                    │  │                  │  └─────┬──────┘  │   │   │
                    │  │                  │        │         │   │   │
                    │  │                  │  ┌─────▼──────┐  │   │   │
                    │  │                  │  │  Backend   │  │   │   │
                    │  │                  │  │ Node.js +  │  │   │   │
                    │  │                  │  │  Express   │  │   │   │
                    │  │                  │  │  x2 Pods   │  │   │   │
                    │  │                  │  └─────┬──────┘  │   │   │
                    │  │                  │        │         │   │   │
                    │  │                  │  ┌─────▼──────┐  │   │   │
                    │  │                  │  │  Database  │  │   │   │
                    │  │                  │  │ PostgreSQL │  │   │   │
                    │  │                  │  │ 16-alpine  │  │   │   │
                    │  │                  │  └─────┬──────┘  │   │   │
                    │  │                  │        │         │   │   │
                    │  │                  │  ┌─────▼──────┐  │   │   │
                    │  │                  │  │  EBS gp3   │  │   │   │
                    │  │                  │  │ Encrypted  │  │   │   │
                    │  │                  │  │  10Gi PVC  │  │   │   │
                    │  │                  │  └────────────┘  │   │   │
                    │  │                  └──────────────────┘   │   │
                    │  └──────────────────────────────────────────┘   │
                    │  VPC (10.0.0.0/16)                               │
                    │  ├── Public Subnets  → ALB, NAT Gateway          │
                    │  └── Private Subnets → EKS Nodes                 │
                    └──────────────────────────────────────────────────┘
```

### Network Policy Flow

```
Internet ──► ALB ──► Frontend pods
                         └──► Backend pods   (port 5000)  ← NetworkPolicy
                                  └──► Database pods (port 5432)  ← NetworkPolicy
```

> No direct access to Backend or Database from outside the cluster.

---

## 🛠 Tech Stack

| Layer | Technology | Version |
|---|---|---|
| **Frontend** | React + Vite, Nginx | React 18, Nginx 1.27-alpine |
| **Backend** | Node.js, Express, pg | Node 20, Express 4 |
| **Database** | PostgreSQL | 16-alpine |
| **Containers** | Docker (multi-stage builds) | latest |
| **Orchestration** | Kubernetes on AWS EKS | 1.34 |
| **IaC** | Terraform (EKS Auto Mode) | v1.5+ |
| **CI/CD** | GitHub Actions | — |
| **Registry** | GitHub Container Registry | ghcr.io |
| **Ingress** | AWS ALB Controller | v2.11.0 |
| **Storage** | AWS EBS gp3 (encrypted) | 10Gi |
| **Autoscaling** | Karpenter (EKS Auto Mode) | — |
---

## ☁️ Infrastructure — Terraform

Provisions the full AWS stack using **EKS Auto Mode** — AWS manages node lifecycle, Karpenter handles scaling.

```
VPC (10.0.0.0/16)
├── Public Subnets  (x3 AZs)  → ALB, NAT Gateway
│     Tag: kubernetes.io/role/elb = 1
├── Private Subnets (x3 AZs)  → EKS worker nodes
└── Single NAT Gateway         → cost-optimised for dev

EKS Cluster (jerney-eks, v1.32)
├── EKS Auto Mode + Karpenter  → intelligent node provisioning
├── Node pools: general-purpose + system
├── KMS envelope encryption    → secrets encrypted at rest
├── Full control plane logging → API, audit, authenticator
└── OIDC provider              → IRSA pod-level AWS permissions
```

```bash
cd Terraform/
terraform init
terraform apply -var-file=terraform.tfvars

# Connect kubectl
aws eks update-kubeconfig --region us-east-1 --name jerney-eks
```

---

## 🐳 Dockerization

Both services use **multi-stage builds** — build tools and source never reach the final image.

### Backend Pipeline

```
node:20-alpine (build)  →  npm ci --only=production
        │
        ▼
node:20-alpine (prod)   →  non-root user UID 1000
                            dumb-init (PID 1 signal handling)
                            EXPOSE 5000
```

### Frontend Pipeline

```
node:20-alpine (build)  →  npm run build → /dist
        │
        ▼
nginx:1.27-alpine (prod) → serve /dist
                            non-root nginx UID 101
                            EXPOSE 8080
```

### Run Locally

```bash
docker compose up --build
# Frontend → http://localhost:80
# Backend  → http://localhost:5000/api/health
```


### Key Design Decisions

| Decision | Reason |
|---|---|
| `strategy: Recreate` on DB | RWO volumes attach to one node only |
| `initContainer` on Backend | Waits for DB:5432 before app starts |
| All services `ClusterIP` | ALB Ingress is the single external entry point |
| Numeric file prefixes (`01-`, `02-`) | Enforces correct apply order |
| `automountServiceAccountToken: false` | Pods don't need K8s API access |
| Karpenter tolerations | Graceful handling of node disruption |

---

## 🔒 Security

Security applied at **every layer** — container, network, storage, and IAM.

### Container Security Context

| | Frontend | Backend | Database |
|---|---|---|---|
| Non-root user | ✅ UID 101 | ✅ UID 1000 | ⚠️ root for initdb |
| Read-only filesystem | ✅ | ✅ | ❌ needs writes |
| No privilege escalation | ✅ | ✅ | ✅ |
| Drop ALL capabilities | ✅ | ✅ | — |
| No SA token automount | ✅ | ✅ | ✅ |

### Network Isolation

```
✅ DB accepts traffic ONLY from backend pods   (port 5432)
✅ Backend accepts traffic ONLY from frontend  (port 5000)
✅ Zero direct external access to backend or DB
✅ All services ClusterIP — no public exposure
```

### Storage

```
✅ EBS gp3: encrypted = "true"    → AES-256 encryption at rest
✅ ReclaimPolicy: Retain           → data persists beyond pod deletion
✅ subPath: pgdata                 → isolated PostgreSQL data directory
```

> ⚠️ **Secrets** are currently base64-encoded Kubernetes Secrets.  
> For production, upgrade to [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) or [External Secrets Operator](https://external-secrets.io).

> 📄 ALB + OIDC + IAM setup → see [`ALB Controller.md`](./ALB%20Controller.md)

---


Images are tagged with **Git commit SHA** for full traceability and rollback:

```
ghcr.io/aakash-thakre/.../jerney-backend:40df8b2
ghcr.io/aakash-thakre/.../jerney-frontend:40df8b2
```

---

## ✅ Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| `kubectl` | v1.32+ | Kubernetes CLI |
| `terraform` | v1.5+ | Infrastructure provisioning |
| `helm` | v3.x | ALB controller install |
| `eksctl` | latest | OIDC + IAM service account setup |
| `aws cli` | v2 | AWS authentication |
| `docker` | latest | Local builds and Compose |

---

<div align="center">

Built by [Aakash Thakre](https://github.com/Aakash-Thakre) &nbsp;•&nbsp;
![AWS EKS](https://img.shields.io/badge/Deployed_on-AWS_EKS-FF9900?style=flat&logo=amazonaws)
![DevSecOps](https://img.shields.io/badge/Practice-DevSecOps-brightgreen?style=flat)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat)

</div>
