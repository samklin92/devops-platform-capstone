# DevOps Platform Capstone
### Terraform · ArgoCD · Kubernetes · GitOps · Progressive Delivery

> **Engineer:** Ogaji Igwe Samuel 
> **Completed:** April 2026  
> **Repo:** devops-platform-capstone  

---

## What This Is

A production-grade DevOps platform built entirely from scratch on real AWS infrastructure. No sandboxes. No guided labs. Every resource provisioned, broken, debugged, and destroyed on a live AWS account across a four-phase self-directed engineering program.

This capstone project is the culmination of that program — all three phases working together as one unified, fully automated platform.

---

## What a Recruiter Should Know

This project demonstrates the following production-ready skills:

| Skill | Evidence |
|-------|---------|
| Infrastructure as Code | Two EKS clusters, two VPCs, IAM roles provisioned by one Terraform config |
| Remote state management | S3 + DynamoDB locking — state isolated per project, versioned, encrypted |
| GitOps delivery | Zero `kubectl apply` — all deployments driven by git push |
| Multi-environment config | Dev, staging, prod from one Kustomize base — DRY principle applied |
| Multi-cluster management | ArgoCD on management cluster driving workload cluster |
| Progressive delivery | Argo Rollouts canary — 20% → 50% → 100% traffic shift |
| Secrets management | ESO + AWS Secrets Manager — secrets never touch Git |
| Troubleshooting | State locks, corrupted state, broken backends, cluster misconfigs — all recovered |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Developer                                                  │
│  git push → GitHub (devops-platform-capstone)               │
└──────────────────────────┬──────────────────────────────────┘
                           │ webhook
┌──────────────────────────▼──────────────────────────────────┐
│  Management Cluster  (EKS · VPC 10.0.0.0/16)               │
│                                                             │
│  ArgoCD control plane                                       │
│  ├── App controller  (reconciliation loop)                  │
│  ├── Repo server     (clones + renders Kustomize)           │
│  ├── ArgoCD server   (UI + CLI + API)                       │
│  └── ApplicationSet  (one template → dev, staging, prod)   │
│                                                             │
│  myapp-dev      (2 pods · nginx:1.25 · Synced · Healthy)   │
│  myapp-staging  (3 pods · nginx:1.25 · Synced · Healthy)   │
│  myapp-prod     (5 pods · nginx:1.25 · Synced · Healthy)   │
└──────────────────────────┬──────────────────────────────────┘
                           │ cross-cluster GitOps
┌──────────────────────────▼──────────────────────────────────┐
│  Workload Cluster  (EKS · VPC 10.1.0.0/16)                 │
│                                                             │
│  Argo Rollouts controller                                   │
│  myapp-prod     (5 pods · deployed by ArgoCD remotely)      │
│  myapp-canary   (Rollout · canary 20%→50%→100%)            │
└─────────────────────────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│  AWS Services                                               │
│  Secrets Manager  →  ESO  →  Kubernetes Secret (auto)      │
│  S3 + DynamoDB    →  Remote state + locking                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
devops-platform-capstone/
├── README.md
├── apps/
│   ├── appset.yaml              ← ApplicationSet — 1 template, 3 apps
│   ├── base/                    ← shared Kubernetes manifests
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── namespace.yaml
│   │   └── kustomization.yaml
│   ├── overlays/
│   │   ├── dev/                 ← replicas: 2 · namespace: myapp-dev
│   │   │   ├── kustomization.yaml
│   │   │   └── external-secret.yaml
│   │   ├── staging/             ← replicas: 3 · namespace: myapp-staging
│   │   │   └── kustomization.yaml
│   │   └── prod/                ← replicas: 5 · namespace: myapp-prod
│   │       └── kustomization.yaml
│   └── rollouts/
│       └── workload/            ← Argo Rollouts canary config
│           ├── namespace.yaml
│           ├── rollout.yaml
│           ├── service-stable.yaml
│           ├── service-canary.yaml
│           └── kustomization.yaml
└── bootstrap/
    ├── argocd-install.sh
    └── register-clusters.sh
```

---

## The Full Delivery Flow

```
git push
  → ArgoCD detects change (webhook / 3min poll)
    → Repo server clones repo
      → Kustomize builds base + overlay
        → App controller compares desired vs live state
          → Applies diff to target cluster
            → Argo Rollouts manages canary traffic split
              → 20% new version → pause 30s
              → 50% new version → pause 30s
              → 100% new version → old pods terminate
                → Health: Healthy · Sync: Synced

Total time from git push to running pods: 30-60 seconds
```

---

## Kustomize — DRY Multi-Environment Config

```
apps/base/          ← written once
  deployment.yaml   ← image: nginx:1.25
  service.yaml
  namespace.yaml

apps/overlays/dev/  ← only what differs
  namespace: myapp-dev
  replicas: 2

apps/overlays/staging/
  namespace: myapp-staging
  replicas: 3

apps/overlays/prod/
  namespace: myapp-prod
  replicas: 5
```

Change the base image → all three environments update from one commit.

---

## Canary Rollout Strategy

```yaml
strategy:
  canary:
    steps:
      - setWeight: 20       # 20% traffic to new version
      - pause: {duration: 30s}
      - setWeight: 50       # 50% traffic to new version
      - pause: {duration: 30s}
      - setWeight: 100      # 100% — old pods terminate
    canaryService: myapp-canary
    stableService: myapp-stable
```

Triggered by a single `git push`. Abort at any step to roll back instantly.

---

## Secrets Management

```
Git                    ESO                    Cluster
───                    ───                    ───────
ExternalSecret    →    reads AWS SM    →    K8s Secret (auto)
(no values)            every 1 hour          password: ***
                                             username: myapp
```

The actual secret values live exclusively in AWS Secrets Manager.
The `ExternalSecret` resource in Git contains only a reference — never a value.

---

## Infrastructure Specs

| Resource | Management Cluster | Workload Cluster |
|----------|-------------------|-----------------|
| Name | capstone-prod-management | capstone-prod-workload |
| Kubernetes | v1.31.14-eks-f69f56f | v1.31.14-eks-f69f56f |
| VPC CIDR | 10.0.0.0/16 | 10.1.0.0/16 |
| Nodes | 2x t3.medium | 2x t3.medium |
| What runs here | ArgoCD, 3 app envs | myapp-prod, Argo Rollouts |

Both clusters provisioned by one Terraform config using `for_each`.

---

## Terraform State Structure

```
myapp-terraform-state-109804294707/ (S3)
├── backend/terraform.tfstate
├── capstone/terraform.tfstate
├── phase3/terraform.tfstate
├── eks/terraform.tfstate
└── environments/
    ├── dev/terraform.tfstate
    └── prod/terraform.tfstate
```

One S3 bucket. Every project isolated by key path. One DynamoDB table locks them all.

---

## Operations Proven

| Operation | Trigger | Result |
|-----------|---------|--------|
| Zero-touch deploy | git push | Pods created — no kubectl apply |
| Git-driven scaling | Change replicas in overlay | Synced in 60 seconds |
| Self-healing | kubectl scale to 1 manually | ArgoCD restored to configured count |
| Rolling update | Change image tag in Git | Zero-downtime rolling update |
| Rollback | Revert commit in Git | Instant — no special commands |
| Canary delivery | git push new image | 20%→50%→100% traffic shift |
| Cross-cluster deploy | ArgoCD app targeting workload | Pods on separate cluster, separate VPC |
| Secret sync | ExternalSecret in Git | K8s Secret auto-created from AWS SM |

---

## Troubleshooting Experience

Real errors encountered and resolved during this program:

- Recovered from stale DynamoDB state locks after system interruption
- Diagnosed Windows Application Control (AppLocker vs WDAC vs Smart App Control)
- Used `terraform state mv` to refactor live resources — zero downtime
- Emptied versioned S3 buckets via AWS CLI when Terraform destroy failed
- Fixed `yes-dev-cluster` — typed `yes` as project name at variable prompt
- Recovered from corrupted `deployment.yaml` after Kustomize content injection
- Fixed ESO `ClusterSecretStore` using `v1` API after `v1beta1` was rejected
- Resolved `<workload-cluster-endpoint>` placeholder error in ArgoCD app create
- Fixed ApplicationSet CRD with `--server-side` flag after annotation size limit

---

## Key Commands

```bash
# Terraform
terraform init && terraform plan && terraform apply
terraform state list
terraform state mv <old> <new>
terraform force-unlock <lock-id>
terraform output cluster_endpoints

# kubectl multi-cluster
kubectl config get-contexts
kubectl config use-context management
kubectl config use-context workload
kubectl get pods -n <ns> --context=workload

# ArgoCD
argocd cluster add workload --name workload-cluster
argocd app list
argocd app get myapp-canary
argocd app history myapp-dev
argocd app rollback myapp-dev <id>

# Argo Rollouts
kubectl argo rollouts get rollout myapp -n myapp-canary --context=workload --watch
kubectl argo rollouts promote myapp -n myapp-canary --context=workload
kubectl argo rollouts abort myapp -n myapp-canary --context=workload
```

---

## Cost Reference

| Resource | Cost |
|----------|------|
| EKS control plane | $0.10/hour per cluster |
| t3.medium node | $0.0416/hour |
| Two clusters total | ~$8-10/day |

**All infrastructure in this portfolio was destroyed after every session. Zero ongoing AWS charges.**

---

## Learning Journey

```
Phase 1 — Terraform IaC
  HCL syntax · variables · modules · remote state · EKS · multi-env

Phase 2 — ArgoCD & GitOps
  GitOps model · Kustomize · ApplicationSet · self-healing · ESO

Phase 3 — Multi-cluster Kubernetes
  Hub + spoke · cross-cluster GitOps · Argo Rollouts · canary delivery

Capstone — Full platform
  Everything above working together as one unified platform
```

Started unable to run `terraform.exe` due to Windows Application Control.  
Finished managing a multi-cluster Kubernetes fleet with progressive delivery.

---

*Built on real AWS infrastructure · All resources destroyed after every session*  
*GitHub: https://github.com/samklin92/devops-platform-capstone*
