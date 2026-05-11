# 🏗️ Repo 21 — `devops-infra`

> **Status:** Planned · **Flag:** — · **Difficulty:** ⭐⭐⭐⭐ · **Build Priority:** 20 (Final Phase)

---

## Purpose

The **infrastructure-as-code repository that deploys all 21 GlucoVision services**. Contains every Dockerfile, Helm chart, Terraform configuration, and GitHub Actions CI/CD pipeline for the entire platform. Exists as a separate repo because infrastructure code deploys all other repos — it must have its own audit trail, access controls, and rollback capability independent of any application service. A bug in a Terraform config must be patchable without touching patient data services.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `docker/` | Production-optimised multi-stage Dockerfiles for all 21 services |
| `kubernetes/` | Helm charts, K8s namespaces, ingress rules, resource quotas, NetworkPolicy |
| `terraform/` | Cloud infrastructure provisioning (VPC, EKS/GKE, RDS, S3, InfluxDB, Redis) |
| `ci-cd/` | GitHub Actions workflow per service — build, test, scan, push, deploy |
| `secrets/` | HashiCorp Vault configuration, env templates, secret rotation policies |
| `monitoring/` | Kubernetes-level Prometheus stack, Grafana operator, alert rules |
| `scripts/` | Bootstrap scripts (local dev setup, seed data, DB migrations) |
| `helm-values/` | Per-environment (dev / staging / prod) Helm value overrides |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Application Docker images | All 21 service CI pipelines | Docker image tags (GHCR registry) |
| Cloud credentials | HashiCorp Vault / GitHub OIDC | Encrypted / keyless |
| Helm chart values | Per-environment YAML overrides | YAML |
| Terraform variables | `terraform.tfvars` per environment | HCL |
| GitHub PR trigger | Developer code push | GitHub webhook |
| Manual deploy trigger | DevOps engineer | GitHub Actions `workflow_dispatch` |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Running K8s pods | All 21 services | K8s Pod spec |
| Cloud infrastructure | All services (VPC, RDS, S3, InfluxDB) | Terraform state |
| Deployment status | Dev team | GitHub Actions CI/CD logs |
| TLS certificates | All services (cert-manager) | K8s TLS secret |
| Secrets | All services (Vault agent sidecar) | Env injection |
| Monitoring stack | Ops team | Prometheus + Grafana (K8s deployed) |

---

## Dependencies

| Tool / Service | Role |
|---|---|
| GitHub Actions | CI/CD runner |
| GitHub Container Registry (GHCR) | Container image registry |
| HashiCorp Vault | Secret management |
| Helm 3 | Kubernetes package manager |
| Terraform | Cloud infrastructure provisioning |
| AWS EKS / GCP GKE | Managed Kubernetes cluster |
| AWS RDS / Cloud SQL | Managed PostgreSQL |
| InfluxDB Cloud / Self-hosted | Time-series database |
| Redis (ElastiCache / Memorystore) | Managed cache |
| MinIO / S3 | Object storage |
| cert-manager | Automatic TLS via Let's Encrypt |
| Traefik Ingress Controller | API gateway in K8s |
| NVIDIA device plugin | GPU scheduling for AI services |

---

## Communication

| Protocol | Used For |
|---|---|
| Kubernetes API | Helm deploys → K8s API server |
| Terraform AWS/GCP API | Infrastructure provisioning |
| GitHub Actions OIDC | Keyless auth to AWS/GCP for CI/CD |
| Vault Agent sidecar | Secret injection into pods |
| HTTPS | Docker image registry pulls |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Containerization | Docker (multi-stage builds) |
| Orchestration | Kubernetes (EKS / GKE) |
| Package Management | Helm 3 |
| Infrastructure as Code | Terraform |
| CI/CD | GitHub Actions |
| Secrets | HashiCorp Vault + Vault Agent Injector |
| TLS | cert-manager + Let's Encrypt |
| Image Registry | GitHub Container Registry (GHCR) |
| GitOps | ArgoCD (optional) |
| Service Mesh | Istio (optional, mTLS between services) |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| GPU node pools | K8s node pool with NVIDIA GPU instances for `09`, `10`, `11`, `12` |
| NVIDIA device plugin | Deploy K8s device plugin for GPU pod scheduling |
| Model storage | MinIO / S3 lifecycle policies for model artifact retention |
| MLflow deployment | Helm chart for MLflow tracking server + artifact store |
| FL server deployment | Isolated K8s namespace for `20` federated learning |

---

## Data Knowledge

| Artifact | Format | Location |
|---|---|---|
| Terraform state | `.tfstate` | S3 backend (encrypted, versioned) |
| Helm release history | Helm release records | K8s cluster |
| Docker image tags | Semver `v1.2.3` | GHCR |
| Secrets | AES-256 encrypted | HashiCorp Vault |

**Repository structure:**
```
devops-infra/
├── docker/
│   ├── 05-auth-service/Dockerfile
│   ├── 09-food-recognition/Dockerfile.gpu
│   ├── 17-esp32-cam-streaming/Dockerfile
│   ├── 18-wearable-sync/Dockerfile
│   └── ... (one per service, 21 total)
├── kubernetes/
│   ├── namespaces/
│   │   ├── glucovision-backend.yaml
│   │   ├── glucovision-ai.yaml
│   │   ├── glucovision-health.yaml
│   │   ├── glucovision-iot.yaml
│   │   └── glucovision-research.yaml
│   ├── helm-charts/
│   │   ├── auth-service/
│   │   ├── food-recognition/
│   │   ├── esp32-cam-streaming/
│   │   ├── wearable-sync/
│   │   └── ... (21 charts)
│   └── ingress/
├── terraform/
│   ├── modules/
│   │   ├── eks-cluster/
│   │   ├── rds-postgres/
│   │   ├── influxdb/
│   │   ├── gpu-node-pool/
│   │   └── vpc/
│   └── environments/
│       ├── dev/
│       ├── staging/
│       └── prod/
├── ci-cd/
│   └── .github/workflows/
│       ├── 05-auth-service.yml
│       ├── 09-food-recognition.yml
│       ├── 17-esp32-cam-streaming.yml
│       ├── 18-wearable-sync.yml
│       └── ... (21 pipelines)
└── secrets/
    ├── vault-policies/
    └── env-templates/
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Multi-environment | dev / staging / prod — Terraform workspaces + Helm value overrides |
| Namespace isolation | Backend, AI, Health, IoT, Research — each in own K8s namespace |
| Medical service isolation | `12` + `13` in dedicated namespace; NetworkPolicy restricts cross-namespace traffic |
| IoT namespace | `17` + `18` in `glucovision-iot` namespace — separate from AI inference |
| GPU scheduling | `09`, `10`, `11`, `12` — GPU resource requests; scheduled on GPU node pool only |
| HPA | Horizontal Pod Autoscaler for `09`, `12`, `17` (stream relay) |
| Rollout strategy | Blue-green for `12`, `13` (medical critical); rolling for all others |
| Terraform state | Remote state in S3 + DynamoDB lock table |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| OIDC auth | GitHub Actions → AWS/GCP via OIDC — no long-lived cloud keys |
| Vault Agent Injector | Secrets injected as env vars at pod startup — never in images or git |
| Network policies | Each service only accepts traffic from declared upstream services |
| K8s RBAC | Service accounts with minimum required permissions |
| Image scanning | Trivy in CI — block push if critical CVE detected |
| Image signing | cosign image signing for all 21 service images |
| Audit logs | K8s audit log + Vault audit log for all infrastructure operations |
| Secrets rotation | Vault dynamic DB credentials — auto-rotate every 24hr |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| HPA thresholds | CPU > 70% → scale `09` food-recognition, `12` glucose-prediction |
| GPU autoscaling | KEDA + GPU metrics for AI inference pod scaling |
| Resource limits | Every pod has CPU + memory requests and limits |
| Pod disruption budget | PDB for `12`, `13` — min 1 pod always running during K8s upgrades |
| CD speed | GitHub Actions: build + push + deploy in < 5 min per service |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Terraform Tests | `terraform plan` in CI — diff review before `apply` |
| Helm Tests | `helm lint` + `helm template` + `helm test` post-deploy smoke test |
| CI Tests | Each pipeline: unit test → build → scan → push → deploy-to-staging |
| Infrastructure Tests | Terratest (Go) for Terraform module integration tests |
| Smoke Tests | Post-deploy: `kubectl rollout status` + `/health` endpoint check |
| Rollback Tests | `helm rollback` restores previous version within 2 min |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | K8s node metrics (CPU, memory, GPU utilisation), pod restart counts |
| Alerts | Pod CrashLoopBackOff → PagerDuty; GPU OOM → scale node |
| Deployment tracking | GitHub Actions summary + Slack notifications |
| Audit | All `kubectl` commands logged to K8s audit log |
| Cost monitoring | AWS/GCP billing alerts — GPU instances are expensive |

---

## Research Complexity

**🟠 Advanced** — Kubernetes production deployment, GPU scheduling, multi-environment Terraform, Vault secret management, GitOps CI/CD pipelines for 21 services.

---

## Portfolio Value

Kubernetes production deployment (EKS / GKE) · Helm chart authoring for 21 microservices · Terraform infrastructure-as-code (multi-environment) · GitHub Actions CI/CD per service · HashiCorp Vault secret management · GPU node pool scheduling for AI services · Medical-grade blue-green deployment strategy · K8s NetworkPolicy inter-service security isolation · GitOps with ArgoCD · Container image signing (cosign) + scanning (Trivy)
