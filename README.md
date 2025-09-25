# Flux K3s Example (Oracle free tier)

This repo is used to illustrate a FluxCD-managed, GitOps-based Kubernetes cluster (k3s) on Oracle Cloud free tier. It attempts modern cloud-native practices including declarative infrastructure management, automated deployments, and continuous reconciliation.

## TODO

- [x] **Add UI [gimlet/capacitor-ui](https://github.com/gimlet-io/capacitor)**
- [x] **Implement [podinfo](https://github.com/stefanprodan/podinfo)** microservice application
- [ ] Add Traefik ingress 
- [x] Add requesite configuration for **Tailscale**
  - [ ] [TailScale Kubernetes operator?](https://tailscale.com/kb/1236/kubernetes-operator)
- [ ] **Multi-Environment Support**: Add staging and production overlays
- [ ] **Secrets Management**: Implement SOPS or External Secrets Operator
- [ ] **Monitoring Stack**: Deploy Prometheus, Grafana, and AlertManager
- [ ] **Image Update Automation**: Automated container image updates
- [ ] **Security Hardening**: Network policies, Pod Security Standards, admission controllers

### Longer Term...
- [ ] **Policy as Code**: [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/docs/) or [Kyverno](https://kyverno.io/) for compliance
- [ ] **Disaster Recovery**: Backup and restore procedures
- [ ] **Advanced Observability**: Distributed tracing and log aggregation
- [ ] **Progressive Delivery**: Flagger for canary deployments

## Configuration
- [`clusters/dev`](./clusters/dev/) points to the oracle cluster
- apps base configuration in [`apps/base`](./apps/base/)
- apps override configuration in [`apps/dev`](./apps/dev/)

## 🏗️ Architecture Overview

This repository implements a **GitOps workflow** where:
- **Git is the single source of truth** for infrastructure and application definitions
- **Flux CD automatically synchronizes** cluster state with repository state
- **Kustomize handles configuration management** across environments
- **OCI repositories provide artifact-based deployments** for applications

```
┌──────────────────┐     ┌──────────────────┐    ┌─────────────────────┐
│   Git Repository │──── │   Flux CD        │────│  Oracle K3s         │
│   (This Repo)    │     │   Controllers    │    │  Cluster            │
└──────────────────┘     └──────────────────┘    └─────────────────────┘
         │                       │                        │
         │                       │                        │
    ┌────▼────┐             ┌────▼────┐              ┌────▼────┐
    │ Apps    │             │ Source  │              │ Running │
    │ Config  │             │ Control │              │ Workload│
    └─────────┘             └─────────┘              └─────────┘
```

## 🚀 Quick Start

### Prerequisites

- **Oracle Cloud Account** with free tier K3s cluster provisioned
- **kubectl** configured to access your cluster
- **Flux CLI** installed ([installation guide](https://fluxcd.io/flux/installation/))
- **Git access** to this repository

### Initial Deployment

1. Fork repo 
2. **Bootstrap Flux on your cluster:**
   ```bash
   export GITHUB_USER=<github username>
   # preferably a token scoped to your repo
   export GITHUB_TOKEN=<github token>

   # Install Flux components
   flux install
   
   # Create the GitRepository source
   flux bootstrap github \
     --owner <github username>
     --repository <destination github repo>
     --private-key-file=/path/to/your/ssh/key \
     --personal 
   ```

3. **Verify deployment:**
   ```bash
   # Check Flux components
   flux get sources git
   flux get kustomizations
   
   # Monitor reconciliation
   flux logs --follow
   ```

## 📁 Repository Structure

```
flux-infra/
├── .github/
│   └── workflows/                      # CI/CD automation
│       ├── claude-code-review.yml      # AI-powered code review
│       ├── e2e.yaml                    # End-to-end testing workflow
│       ├── kustomize-verify.yml        # Validates Kustomize builds
│       └── tailscale.yml               # Tailscale integration workflow
├── .vscode/
│   └── settings.json                   # VSCode project settings
├── apps/
│   ├── base/                           # Base application configurations
│   │   ├── capacitor-ui/               # Capacitor UI application
│   │   │   ├── flux-kustomization.yaml # Flux deployment config
│   │   │   ├── kustomization.yaml      # Kustomize base config
│   │   │   └── oci-repository.yaml     # OCI source definition
│   │   └── podinfo/                    # Podinfo microservice
│   │       ├── helm-release.yaml       # Helm release definition
│   │       ├── helm-repository.yaml    # Helm repository source
│   │       └── kustomization.yaml      # Kustomize base config
│   └── dev/                            # Development environment overlays
│       ├── capacitor-ui/
│       │   └── kustomization.yaml      # Dev overlay for Capacitor UI
│       ├── podinfo/
│       │   ├── helm-release-values.yaml # Dev-specific Helm values
│       │   └── kustomization.yaml       # Dev overlay for Podinfo
│       └── kustomization.yaml           # Root dev apps config
├── clusters/
│   └── dev/                            # Development cluster configuration
│       ├── apps/                       # Application deployment configs
│       │   ├── flux-kustomization-apps.yaml # Apps Flux kustomization
│       │   └── kustomization.yaml      # Apps orchestration
│       ├── infra/                      # Infrastructure components
│       │   ├── configmap-env.yaml     # Environment configuration
│       │   ├── flux-kustomization.yaml # Infrastructure Flux config
│       │   ├── git-repository.yaml    # Git source definition
│       │   ├── gotk-components.yaml   # Flux core components
│       │   └── kustomization.yaml     # Infrastructure orchestration
│       └── kustomization.yaml          # Root cluster config
├── scripts/
│   └── validate.sh                     # Validation script
├── .gitignore                          # Git ignore patterns
├── flux-init.sh                        # Flux initialization script
├── README.md                           # This file
```

### Configuration Flow

```
clusters/dev/kustomization.yaml
├── infra/                  # Infrastructure first
│   ├── Flux controllers
│   └── Core services
└── apps/                   # Applications second
    └── capacitor-ui/       # UI application
        └── → apps/base/capacitor-ui/
```

## 🔧 Current Components

### Infrastructure Layer
- **Flux CD Controllers**: Source, Kustomize, and Image controllers
- **Git Synchronization**: Automatic sync with this repository every 10 minutes
- **Kustomize Integration**: Configuration management and environment overlays

### Application Layer
- **Capacitor UI**: Kubernetes dashboard and management interface
  - Source: OCI registry (`ghcr.io/gimlet-io/capacitor-manifests`)
  - Deployment: `flux-system` namespace
  - Updates: Semantic versioning (`>=0.1.0`)

## 🛠️ Development Workflow

### Making Changes

1. **Create a feature branch:**
   ```bash
   git checkout -b feature/add-monitoring
   ```

2. **Make your changes** to the appropriate directory:
   - Infrastructure changes: `clusters/dev/infra/`
   - Application changes: `apps/base/` or `apps/dev/`

3. **Test locally** (requires kubectl and kustomize):
   ```bash
   # Validate Kustomize builds
   kubectl kustomize ./clusters/dev
   
   # Check for any issues
   kubectl kustomize ./clusters/dev --enable-helm
   ```

4. **Commit and push:**
   ```bash
   git add .
   git commit -m "feat: add monitoring stack"
   git push origin feature/add-monitoring
   ```

5. **Create Pull Request** - CI will automatically validate your changes

### CI/CD Pipeline

**Automated Checks:**
- **Kustomize Validation**: Ensures all Kustomize configurations build successfully
- **Kubernetes Schema Validation**: Validates resources against K8s API schemas
- **Security Scanning**: Checks for security best practices (planned)

**Deployment:**
- **Automatic**: Flux monitors the `main` branch and deploys changes within 10 minutes
- **Manual**: Force sync with `flux reconcile kustomization flux-system`

## 🔍 Monitoring and Troubleshooting

### Check Flux Status
```bash
# Overall system health
flux get all

# Specific component status
flux get sources git
flux get kustomizations

# View recent events
flux events

# Follow logs
flux logs --follow
```

### Debug Deployment Issues
```bash
# Check Kustomization status
kubectl describe kustomization capacitor -n flux-system

# View application pods
kubectl get pods -n flux-system

# Check resource events
kubectl get events -n flux-system --sort-by='.lastTimestamp'
```

### Common Issues and Solutions

**Issue**: Kustomization stuck in "Progressing" state
```bash
# Solution: Check for dependency issues
kubectl describe kustomization <name> -n flux-system
```

**Issue**: OCI repository authentication failures
```bash
# Solution: Verify OCI repository accessibility
flux get sources oci
```

**Issue**: Git repository sync failures
```bash
# Solution: Check SSH key configuration
kubectl get secret flux-system -n flux-system -o yaml
```

## 🏛️ Infrastructure Design Principles

### GitOps Best Practices
- **Declarative Configuration**: Everything defined as code
- **Version Controlled**: All changes tracked in Git
- **Automated Synchronization**: Flux handles deployment reconciliation
- **Rollback Capability**: Git history enables easy rollbacks

### Kubernetes Patterns
- **Namespace Isolation**: Logical separation of components
- **Resource Quotas**: Prevent resource exhaustion (planned)
- **Network Policies**: Micro-segmentation for security (planned)
- **Health Checks**: Ensure application availability

### Oracle Cloud Considerations
- **Free Tier Limits**: Optimized for 2 OCPU, 12GB RAM constraints
- **ARM Architecture**: Compatible with ARM64 workloads
- **Regional Deployment**: Single availability domain deployment
- **Cost Optimization**: Resource requests tuned for free tier

## 🤝 Contributing

### Getting Started
1. Fork this repository
2. Set up your own Oracle Cloud K3s cluster for testing
3. Bootstrap Flux pointing to your fork
4. Make changes and test them
5. Submit a Pull Request with clear description

### Code Standards
- **Kustomize**: Use proper base/overlay patterns
- **Naming**: Follow Kubernetes naming conventions
- **Documentation**: Update README for significant changes
- **Testing**: Ensure all Kustomize builds succeed

### Review Process
- All changes require Pull Request review
- CI validation must pass
- Test in development environment first
- Consider impact on running workloads

## 📚 Resources and References

### Flux CD Documentation
- [Flux CD Official Docs](https://fluxcd.io/flux/)
- [GitOps Toolkit](https://fluxcd.io/flux/components/)
- [Kustomize Integration](https://fluxcd.io/flux/components/kustomize/)

### Oracle Cloud
- [OCI Free Tier](https://www.oracle.com/cloud/free/)
- [Container Engine for Kubernetes](https://docs.oracle.com/en-us/iaas/Content/ContEng/home.htm)

### Related Projects
- [Capacitor UI](https://github.com/gimlet-io/capacitor)
- [Podinfo](https://github.com/stefanprodan/podinfo)
- [Tailscale Operator](https://tailscale.com/kb/1236/kubernetes-operator)

## Scripts

This project uses validation scripts adapted from [FluxCD's flux2-kustomize-helm-example](https://github.com/fluxcd/flux2-kustomize-helm-example).

### Validation
Run validation before committing changes:
```bash
./scripts/validate.sh
## 📄 License

This project is provided as-is for demonstration and learning purposes.

---

**Questions or Issues?** 
- Check the [troubleshooting section](#-monitoring-and-troubleshooting)
- Review [Flux CD documentation](https://fluxcd.io/flux/)
- Open an issue in this repository
