# 🔐 DevSecOps Pipeline Framework

**5 Security Gates I Build Into Every CI/CD Pipeline**

A production-ready framework for building secure, automated CI/CD pipelines on AWS with Kubernetes (EKS). This repo demonstrates the DevSecOps practices I use to ship fast while maintaining zero-compromise security.

---

## The Problem

> A single over-permissioned IAM role almost exposed an entire production environment.

During a routine audit, I found an IAM role with full S3 and RDS access attached to a non-critical service. It had been there for months — no alerts, no flags. That moment changed how I build pipelines.

**Security can't be a review step at the end. It has to be automatic, or it doesn't happen.**

---

## The 5-Gate Framework

| Gate | What | Tool | Impact |
|------|------|------|--------|
| **Gate 1** | Code-Level Scanning | SonarQube | Catch vulnerabilities in the PR, not staging |
| **Gate 2** | Container Image Scanning | Trivy + ECR | Block CVEs before they touch EKS |
| **Gate 3** | IAM Least-Privilege | Access Analyzer + Terraform | Every role scoped to minimum permissions |
| **Gate 4** | Policy-as-Code | OPA + Calico | Auto-reject non-compliant K8s deployments |
| **Gate 5** | Zero Hardcoded Secrets | KMS + Secrets Manager | Auto-rotation + pre-commit hooks |

### Results
- **0** security incidents in production over the past year
- **0%** added friction — deployment frequency stayed the same
- **60%** faster audit prep — compliance evidence generated automatically

---

## Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Developer  │────▶│   GitLab CI  │────▶│   Argo CD    │
│   Git Push   │     │   Pipeline   │     │   GitOps     │
└──────────────┘     └──────┬───────┘     └──────┬───────┘
                            │                     │
                    ┌───────▼───────┐     ┌───────▼───────┐
                    │  Gate 1: SAST │     │  Gate 4: OPA  │
                    │  SonarQube    │     │  Admission    │
                    ├───────────────┤     │  Controllers  │
                    │  Gate 2: Trivy│     ├───────────────┤
                    │  Image Scan   │     │  Gate 4:      │
                    ├───────────────┤     │  Calico Net   │
                    │  Gate 5:      │     │  Policies     │
                    │  Secret Check │     └───────┬───────┘
                    └───────┬───────┘             │
                            │              ┌──────▼───────┐
                    ┌───────▼───────┐      │  Amazon EKS  │
                    │  Gate 3: IAM  │      │  Production  │
                    │  Validation   │      │  Cluster     │
                    └───────────────┘      └──────────────┘
```

---

## Repo Structure

```
.
├── terraform/
│   ├── modules/
│   │   ├── vpc/              # VPC, subnets, security groups
│   │   ├── eks/              # EKS cluster, node groups, Karpenter
│   │   └── iam/              # Least-privilege roles + policies
│   └── environments/
│       └── dev/              # Dev environment configuration
├── kubernetes/
│   ├── policies/             # OPA Gatekeeper constraint templates
│   └── network-policies/     # Calico network policies
├── ci-cd/
│   ├── gitlab-ci/            # GitLab CI pipeline with all 5 gates
│   └── github-actions/       # GitHub Actions equivalent
├── scripts/
│   ├── iam-audit.sh          # IAM least-privilege audit script
│   └── secret-scanner.sh     # Pre-commit secret detection
└── docs/
    └── gate-details.md       # Deep dive on each security gate
```

---

## Quick Start

### Prerequisites
- AWS CLI configured with appropriate credentials
- Terraform >= 1.5
- kubectl
- Helm 3

### Deploy Infrastructure
```bash
cd terraform/environments/dev
terraform init
terraform plan
terraform apply
```

### Apply Kubernetes Security Policies
```bash
# Install OPA Gatekeeper
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace

# Apply constraint templates and constraints
kubectl apply -f kubernetes/policies/

# Apply Calico network policies
kubectl apply -f kubernetes/network-policies/
```

### Set Up Pre-Commit Hooks
```bash
cp scripts/secret-scanner.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

---

## Tech Stack

| Category | Tools |
|----------|-------|
| **Cloud** | AWS (EKS, EC2, S3, RDS, IAM, KMS, VPC, Route 53) |
| **IaC** | Terraform |
| **Container Orchestration** | Kubernetes (EKS), Helm, Karpenter |
| **CI/CD** | GitLab CI, GitHub Actions, Argo CD |
| **Security Scanning** | SonarQube, Trivy, AWS Access Analyzer |
| **Policy Engine** | OPA Gatekeeper, Calico |
| **Secrets Management** | AWS KMS, AWS Secrets Manager |
| **Observability** | Prometheus, Grafana, CloudWatch, Splunk |

---

## About

Built by **Parveen Shaik** — Cloud & DevOps Engineer with 7+ years of experience building secure, scalable infrastructure on AWS.

Open to: **Cloud Engineer | DevOps Engineer | SRE | Platform Engineer**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://www.linkedin.com/in/parveenshaik07/)

---

## License

MIT License — feel free to use and adapt for your own pipelines.
