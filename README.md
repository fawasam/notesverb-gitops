# NotesVerb GitOps Repository

This repository contains the GitOps configuration for the NotesVerb microservices platform using Kustomize and ArgoCD.

## 📁 Repository Structure

```
notesverb-gitops/
├── services/
│   ├── api-gateway/
│   ├── auth-service/
│   ├── notes-service/
│   ├── tags-service/
│   └── user-service/
└── argo/
    ├── apps-dev.yaml
    ├── apps-staging.yaml
    └── apps-prod.yaml
```

## 🚀 Microservices

| Service | Port | Health Endpoint | Description |
|---------|------|----------------|-------------|
| **api-gateway** | 8080 | `/health` | API Gateway for routing requests |
| **auth-service** | 3001 | `/health` | Authentication and authorization |
| **user-service** | 3002 | `/health` | User management |
| **notes-service** | 3003 | `/health` | Notes CRUD operations |
| **tags-service** | 3004 | `/health` | Tags management |

## 🏗️ Service Structure

Each service follows the same Kustomize-based structure:

```
service-name/
├── base/
│   ├── deployment.yaml      # Base deployment configuration
│   ├── service.yaml          # Kubernetes service definition
│   └── kustomization.yaml    # Base kustomization
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── staging/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

## 🌍 Environments

### Development
- **Namespace**: `{service}-dev`
- **Replicas**: 1
- **Auto-sync**: ✅ Enabled
- **Self-heal**: ✅ Enabled

### Staging
- **Namespace**: `{service}-staging`
- **Replicas**: 1
- **Auto-sync**: ✅ Enabled
- **Self-heal**: ✅ Enabled

### Production
- **Namespace**: `{service}-prod`
- **Replicas**: 2
- **Auto-sync**: ❌ Disabled (manual approval required)
- **Self-heal**: ❌ Disabled

## 🔧 Prerequisites

- Kubernetes cluster (v1.20+)
- ArgoCD installed and configured
- kubectl CLI
- kustomize CLI (optional, for local testing)

## 📦 Deployment

### 1. Deploy with ArgoCD (Recommended)

Apply the app-of-apps manifests to deploy all services for an environment:

```bash
# Deploy to development
kubectl apply -f argo/apps-dev.yaml

# Deploy to staging
kubectl apply -f argo/apps-staging.yaml

# Deploy to production
kubectl apply -f argo/apps-prod.yaml
```

### 2. Manual Deployment with Kustomize

For testing or manual deployment:

```bash
# Build and apply dev environment
kustomize build services/api-gateway/overlays/dev | kubectl apply -f -

# Build and apply staging environment
kustomize build services/auth-service/overlays/staging | kubectl apply -f -

# Build and apply prod environment
kustomize build services/notes-service/overlays/prod | kubectl apply -f -
```

## 🧪 Local Testing

Test kustomize builds locally before pushing:

```bash
# Validate dev overlay
kustomize build services/api-gateway/overlays/dev

# Validate staging overlay
kustomize build services/user-service/overlays/staging

# Validate prod overlay
kustomize build services/tags-service/overlays/prod
```

## 🔐 Configuration

### Update Container Images

Edit the base deployment files to update container images:

```yaml
# services/{service}/base/deployment.yaml
spec:
  template:
    spec:
      containers:
      - name: {service}
        image: your-registry/{service}:v1.0.0
```

### Add Environment Variables

Add environment-specific variables in overlay kustomization files:

```yaml
# services/{service}/overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

namespace: {service}-dev

configMapGenerator:
- name: {service}-config
  literals:
  - ENV=development
  - LOG_LEVEL=debug
```

### Add Secrets

Create sealed secrets or use external secret management:

```bash
# Example using kubectl
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=secret \
  -n {service}-dev
```

## 📊 Monitoring

Each service includes:
- **Liveness Probe**: Checks if the service is running
- **Readiness Probe**: Checks if the service is ready to accept traffic
- **Resource Limits**: CPU and memory limits defined

## 🔄 GitOps Workflow

1. **Make changes** to manifests in this repository
2. **Commit and push** to the main branch
3. **ArgoCD detects** changes automatically
4. **Dev/Staging**: Auto-syncs and deploys
5. **Production**: Requires manual sync approval in ArgoCD UI

## 📝 Best Practices

- ✅ Always test changes in dev environment first
- ✅ Use semantic versioning for container images
- ✅ Never commit secrets directly (use sealed secrets or external secret management)
- ✅ Review changes in staging before promoting to production
- ✅ Use pull requests for production changes

## 🛠️ Troubleshooting

### Check ArgoCD Application Status

```bash
# List all applications
kubectl get applications -n argocd

# Describe specific application
kubectl describe application api-gateway-dev -n argocd

# View application logs
kubectl logs -n argocd deployment/argocd-application-controller
```

### Check Service Health

```bash
# Check pod status
kubectl get pods -n api-gateway-dev

# Check service logs
kubectl logs -n api-gateway-dev deployment/api-gateway-dev

# Port forward to test locally
kubectl port-forward -n api-gateway-dev svc/api-gateway-dev 8080:80
```

### Sync Issues

```bash
# Force sync an application
argocd app sync api-gateway-dev

# Refresh application
argocd app get api-gateway-dev --refresh
```

## 🤝 Contributing

1. Create a feature branch
2. Make your changes
3. Test with `kustomize build`
4. Submit a pull request
5. Wait for review and approval

## 📄 License

[Add your license here]

## 📧 Contact

[Add contact information here]
