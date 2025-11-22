# App of Apps - ArgoCD

This project implements the ArgoCD App of Apps pattern to manage multiple Kubernetes applications with shared dependencies.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Applications](#applications)
- [Dependencies](#dependencies)
- [Installation](#installation)
- [Usage](#usage)
- [Adding New Applications](#adding-new-applications)
- [Verification](#verification)
- [Contributing](#contributing)

## 🎯 Overview

This project uses the **App of Apps** pattern to centrally manage:
- **Main Applications**: Ghost (blog platform) and Gitea (git hosting)
- **Shared Dependencies**: PostgreSQL (database) and Redis (cache)

All applications are deployed using Docker Hub images and managed through GitOps principles.

## 🏗️ Architecture

### App of Apps Pattern

1. **Root Application** (`root-application.yaml`)
   - Manually applied to ArgoCD
   - Manages all child applications in `apps/applications/` folder
   - Each child application file creates an ArgoCD Application object

2. **Child Applications** (`apps/applications/*.yml`)
   - Each file defines a child Application object
   - Points to its own manifest folder
   - Deploys the actual applications

3. **Manifests** (`apps/manifests/*/`)
   - Kubernetes manifest files for each application
   - Deployment, Service, ConfigMap, etc.

### System Architecture

```
┌─────────────┐     ┌─────────────┐
│   Ghost     │     │   Gitea     │
│  (Blog)     │     │ (Git Host)  │
│ Port: 30080 │     │ Port: 30081 │
└──────┬──────┘     └──────┬──────┘
       │                   │
       ├───────────┬───────┤
       │           │       │
┌──────▼──────┐ ┌─▼───────▼──┐
│ PostgreSQL  │ │   Redis     │
│ (Database)  │ │  (Cache)    │
│ Port: 5432  │ │ Port: 6379  │
└─────────────┘ └─────────────┘
```

## 📁 Project Structure

```
app-of-apps-argocd/
├── root-application.yaml          # Root (parent) application - manually applied to ArgoCD
├── apps/
│   ├── applications/              # Child application YAML files (ArgoCD Application objects)
│   │   ├── ghost-app.yml         # Ghost blog platform
│   │   ├── gitea-app.yml         # Gitea git hosting
│   │   ├── postgresql-app.yml    # PostgreSQL database (shared)
│   │   └── redis-app.yml         # Redis cache (shared)
│   │
│   └── manifests/                 # Manifest folders for all applications
│       ├── ghost/                 # Ghost manifests
│       │   ├── deployment.yml
│       │   └── service.yml
│       ├── gitea/                 # Gitea manifests
│       │   ├── deployment.yml
│       │   └── service.yml
│       ├── postgresql/            # PostgreSQL manifests
│       │   ├── deployment.yml
│       │   ├── service.yml
│       │   └── configmap.yml     # Init script for multiple databases
│       └── redis/                 # Redis manifests
│           ├── deployment.yml
│           └── service.yml
```

## 📱 Applications

### Ghost (Blog Platform)

- **Namespace:** `ghost`
- **Port:** 2368 (NodePort: 30080)
- **Image:** `ghost:latest`
- **Access:** `http://localhost:30080`
- **Dependencies:**
  - PostgreSQL (database: `ghost`)
  - Redis (cache)

### Gitea (Git Hosting)

- **Namespace:** `gitea`
- **Port:** 3000 (NodePort: 30081)
- **Image:** `gitea/gitea:latest`
- **Access:** `http://localhost:30081`
- **Dependencies:**
  - PostgreSQL (database: `gitea`)
  - Redis (cache)

## 🔗 Dependencies

### PostgreSQL (Shared Database)

- **Namespace:** `postgres`
- **Port:** 5432
- **Image:** `postgres:alpine`
- **Databases:**
  - `ghost` - Used by Ghost
  - `gitea` - Used by Gitea
- **Configuration:**
  - Init script in ConfigMap creates all databases automatically
  - Access: `postgres.postgres.svc.cluster.local:5432`

### Redis (Shared Cache)

- **Namespace:** `redis`
- **Port:** 6379
- **Image:** `redis:alpine`
- **Used by:**
  - Ghost (cache)
  - Gitea (cache)
- **Access:** `redis.redis.svc.cluster.local:6379`

## 🚀 Installation

### Prerequisites

- Kubernetes cluster
- ArgoCD installed and running
- Git repository access
- `kubectl` configured

### Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/onurglr/app-of-apps-argocd.git
   cd app-of-apps-argocd
   ```

2. **Apply the root application to ArgoCD:**
   ```bash
   kubectl apply -f root-application.yaml
   ```

3. **ArgoCD will automatically:**
   - Read all child applications from `apps/applications/` folder
   - Deploy each one
   - Read manifest files and start the applications
   - Create namespaces if needed

## 📖 Usage

### Access Applications

- **Ghost:** `http://localhost:30080`
- **Gitea:** `http://localhost:30081`

### Check Application Status

```bash
# List all applications
kubectl get applications -n argocd

# Check specific application
kubectl get application ghost-app -n argocd
kubectl get application gitea-app -n argocd

# Check pods
kubectl get pods -n ghost
kubectl get pods -n gitea
kubectl get pods -n postgres
kubectl get pods -n redis
```

### View Logs

```bash
# Ghost logs
kubectl logs -n ghost -l app=ghost

# Gitea logs
kubectl logs -n gitea -l app=gitea

# PostgreSQL logs
kubectl logs -n postgres -l app=postgres

# Redis logs
kubectl logs -n redis -l app=redis
```

## ➕ Adding New Applications

To add a new application that uses PostgreSQL:

1. **Add database to ConfigMap:**
   ```yaml
   # apps/manifests/postgresql/configmap.yml
   CREATE DATABASE new-app;
   ```

2. **Create child application:**
   ```yaml
   # apps/applications/new-app.yml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: new-app
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/onurglr/app-of-apps-argocd.git
       targetRevision: HEAD
       path: apps/manifests/new-app
     destination:
       server: https://kubernetes.default.svc
       namespace: new-app
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
         - CreateNamespace=true
   ```

3. **Create manifest files:**
   - `apps/manifests/new-app/deployment.yml`
   - `apps/manifests/new-app/service.yml`

4. **Add PostgreSQL connection in deployment:**
   ```yaml
   env:
   - name: DATABASE_HOST
     value: postgres.postgres.svc.cluster.local
   - name: DATABASE_PORT
     value: "5432"
   - name: DATABASE_NAME
     value: "new-app"
   ```

5. **Commit and push:**
   ```bash
   git add .
   git commit -m "Add new-app"
   git push
   ```

6. **ArgoCD will automatically deploy!**

## 🔍 Verification

### Via ArgoCD UI

1. Navigate to ArgoCD UI
2. View the root application (`root-app`)
3. Check child applications and their status
4. Verify all applications are "Healthy" and "Synced"

### Via kubectl

```bash
# Check all applications
kubectl get applications -n argocd

# Check application details
kubectl describe application ghost-app -n argocd

# Check services
kubectl get svc -A

# Check all pods
kubectl get pods -A
```

## 🔧 Troubleshooting

### Application not syncing

```bash
# Check application status
kubectl get application <app-name> -n argocd

# Manually sync
argocd app sync <app-name>

# Check logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

### Database connection issues

```bash
# Check PostgreSQL pod
kubectl get pods -n postgres

# Check PostgreSQL logs
kubectl logs -n postgres -l app=postgres

# Test connection from Ghost pod
kubectl exec -it -n ghost <ghost-pod-name> -- sh
# Inside pod: ping postgres.postgres.svc.cluster.local
```

### Service not accessible

```bash
# Check service
kubectl get svc -n <namespace>

# Check NodePort
kubectl get svc <service-name> -n <namespace> -o yaml

# Check pods
kubectl get pods -n <namespace>
```

## 📚 Learning Resources

This project was created for educational purposes. Each file contains detailed comments explaining:
- What each field does
- Why specific values were chosen
- Alternatives and when to use different values

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is for educational purposes.

## 🔗 Links

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Ghost Documentation](https://ghost.org/docs/)
- [Gitea Documentation](https://docs.gitea.io/)
