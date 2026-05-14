# Sentinel GitOps Platform 🚀

Production-grade multi-cluster Kubernetes platform built on AWS using Terraform, ArgoCD, GitOps, and progressive delivery.

Designed and engineered by Ogaji Igwe Samuel.

---

# What This Project Is

Sentinel GitOps Platform is a real-world DevOps and platform engineering project built entirely on live AWS infrastructure.

The platform demonstrates how modern engineering teams automate infrastructure provisioning, application delivery, environment management, secrets handling, and progressive deployments across multiple Kubernetes clusters using GitOps principles.

Every component was provisioned, configured, debugged, and destroyed manually during a multi-phase self-directed engineering program.

No guided labs. No sandbox environments. Real infrastructure. Real troubleshooting.

---

# Core Features

✅ Multi-cluster Amazon EKS architecture  
✅ Infrastructure as Code with Terraform  
✅ Remote Terraform state with S3 + DynamoDB locking  
✅ GitOps delivery with ArgoCD  
✅ Kustomize multi-environment overlays  
✅ Progressive delivery with Argo Rollouts  
✅ Canary deployments (20% → 50% → 100%)  
✅ Cross-cluster application deployment  
✅ Secrets management with ESO + AWS Secrets Manager  
✅ Self-healing Kubernetes workloads  
✅ Fully automated deployment workflows  

---

# Platform Architecture
<img width="1536" height="1024" alt="GitOps architecture overview infographic" src="https://github.com/user-attachments/assets/4ab3514c-c36d-4798-8990-119419a696e3" />



# What This Demonstrates

| Capability | Implementation |
|---|---|
| Infrastructure as Code | Terraform-provisioned EKS clusters and VPCs |
| GitOps | ArgoCD-driven deployments from Git |
| Multi-environment delivery | Dev, staging, prod via Kustomize overlays |
| Multi-cluster operations | Management cluster controlling workload cluster |
| Progressive delivery | Canary rollout strategy with Argo Rollouts |
| Secrets management | ESO syncing from AWS Secrets Manager |
| Remote Terraform state | S3 backend with DynamoDB locking |
| Kubernetes operations | Automated reconciliation and self-healing |
| Troubleshooting | Real AWS, Terraform, Kubernetes recovery scenarios |

---

# Repository Structure

```text
sentinel-gitops-platform/
│
├── apps/
│   ├── appset.yaml
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── namespace.yaml
│   │   └── kustomization.yaml
│   │
│   ├── overlays/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   │
│   └── rollouts/
│       └── workload/
│
├── bootstrap/
│   ├── argocd-install.sh
│   └── register-clusters.sh
│
└── README.md
```

---

# Deployment Workflow

```text
git push
   ↓
ArgoCD detects repository change
   ↓
Repo Server clones repository
   ↓
Kustomize renders manifests
   ↓
Application Controller compares desired vs live state
   ↓
Changes applied to target cluster
   ↓
Argo Rollouts performs canary deployment
   ↓
20% → 50% → 100% traffic shift
   ↓
Healthy + Synced
```

Average deployment time:

```text
30–60 seconds from commit to running pods
```

---

# Canary Delivery Strategy

```yaml
strategy:
  canary:
    steps:
      - setWeight: 20
      - pause: {duration: 30s}

      - setWeight: 50
      - pause: {duration: 30s}

      - setWeight: 100
```

Deployments can be promoted, paused, or rolled back instantly.

---

# Secrets Management

```text
Git Repository
      ↓
ExternalSecret Resource
      ↓
External Secrets Operator
      ↓
AWS Secrets Manager
      ↓
Kubernetes Secret
```

Sensitive values never exist inside Git repositories.

---

# Infrastructure Specifications

| Resource | Management Cluster | Workload Cluster |
|---|---|---|
| Platform | Amazon EKS | Amazon EKS |
| Kubernetes Version | v1.31 | v1.31 |
| VPC CIDR | 10.0.0.0/16 | 10.1.0.0/16 |
| Nodes | 2 × t3.medium | 2 × t3.medium |
| Purpose | ArgoCD + environments | Production workloads |

---

# Operational Capabilities Proven

| Operation | Result |
|---|---|
| Git push deployment | Fully automated delivery |
| Self-healing | ArgoCD restored manual changes |
| Rolling updates | Zero-downtime deployments |
| Rollbacks | Git revert restored previous version |
| Canary deployments | Progressive traffic shifting |
| Cross-cluster deployment | Remote workload synchronization |
| Secret synchronization | ESO created Kubernetes secrets automatically |

---

# Troubleshooting Experience

This platform was built and debugged on live AWS infrastructure.

Real issues resolved include:

- Terraform state lock recovery
- Corrupted Terraform state repair
- Cross-cluster connectivity issues
- ArgoCD synchronization failures
- Kustomize rendering conflicts
- ESO API version incompatibilities
- Kubernetes deployment recovery
- ApplicationSet CRD annotation limits
- AWS backend cleanup and recovery

---

# Key Commands

## Terraform

```bash
terraform init
terraform plan
terraform apply
terraform state list
terraform force-unlock <lock-id>
```

## Kubernetes

```bash
kubectl config get-contexts
kubectl config use-context management
kubectl config use-context workload
```

## ArgoCD

```bash
argocd app list
argocd app get myapp-dev
argocd app rollback myapp-dev <id>
```

## Argo Rollouts

```bash
kubectl argo rollouts get rollout myapp
kubectl argo rollouts promote myapp
kubectl argo rollouts abort myapp
```

---

# Cost Awareness

| Resource | Approximate Cost |
|---|---|
| EKS control plane | $0.10/hour |
| t3.medium node | $0.0416/hour |
| Two-cluster environment | ~$8–10/day |

All infrastructure was destroyed after each session to avoid unnecessary AWS charges.

---

# Engineering Journey

## Phase 1 — Terraform & Infrastructure as Code
- Remote state
- EKS provisioning
- Multi-environment infrastructure

## Phase 2 — GitOps & Kubernetes Operations
- ArgoCD
- Kustomize
- ApplicationSets
- Self-healing deployments

## Phase 3 — Multi-Cluster Delivery
- Cross-cluster GitOps
- Argo Rollouts
- Progressive delivery
- Canary deployments

## Capstone Platform
All systems integrated into one production-style delivery platform.

---

# Key Lessons Learned

- GitOps simplifies operational consistency
- Infrastructure automation reduces deployment risk
- Progressive delivery improves deployment safety
- Multi-cluster systems require operational discipline
- Troubleshooting is one of the most valuable engineering skills

---

# Author

## 👨‍💻 Ogaji Igwe Samuel

GitHub: https://github.com/samklin92

LinkedIn: https://linkedin.com/in/samklin92

Repository:
https://github.com/samklin92/sentinel-gitops-platform

---

# Final Note

This project represents a complete production-style GitOps platform engineered from the ground up using AWS, Kubernetes, Terraform, ArgoCD, and progressive delivery workflows.

Built on real infrastructure. Debugged through real failures. Operated like a real platform.
