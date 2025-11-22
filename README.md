# App of Apps - ArgoCD

This project implements the ArgoCD App of Apps pattern to centrally manage multiple Kubernetes applications.

## 📋 Table of Contents

- [Project Structure](#project-structure)
- [Architecture](#architecture)
- [Installation](#installation)
- [Usage](#usage)
- [Applications](#applications)
- [Adding New Applications](#adding-new-applications)
- [Verification](#verification)
- [Contributing](#contributing)

## 📁 Project Structure

```
app-of-apps-argocd/
├── root-application.yaml          # Root (parent) application - manually applied to ArgoCD
├── apps/
│   ├── applications/              # Child application YAML files (ArgoCD Application objects)
│   │   ├── nginx-app.yml         # Nginx child application
│   │   ├── redis-app.yml         # Redis child application
│   │   └── guestbook-app.yml     # Guestbook child application
│   │
│   └── manifests/                 # Manifest folders for all applications
│       ├── nginx/                 # Nginx manifests
│       │   ├── deployment.yml
│       │   └── service.yml
│       ├── redis/                 # Redis manifests
│       │   ├── deployment.yml
│       │   └── service.yml
│       └── guestbook/             # Guestbook manifests
│           ├── deployment.yml
│           └── service.yml
```

## 🏗️ Architecture

### App of Apps Pattern

This project uses the **App of Apps** pattern:

1. **Root Application** (`root-application.yaml`)
   - Manually applied to ArgoCD
   - Manages all child applications in the `apps/applications/` folder
   - Each child application file creates an ArgoCD Application object

2. **Child Applications** (`apps/applications/*.yml`)
   - Each file defines a child Application object
   - Points to its own manifest folder
   - Deploys the actual applications

3. **Manifests** (`apps/manifests/*/`)
   - Kubernetes manifest files for each application
   - Deployment, Service, ConfigMap, etc.

### Namespace Structure

- **nginx** namespace → Nginx web server
- **redis** namespace → Redis cache (shared, can be used by multiple applications)
- **guestbook** namespace → Guestbook application (connects to Redis)

### Dependencies

- **Guestbook** → Connects to Redis (`redis.redis.svc.cluster.local:6379`)
- Uses cross-namespace access (different namespaces)

## 🚀 Installation

### Prerequisites

- Kubernetes cluster
- ArgoCD installed and running
- Git repository access

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
   - Read all child applications from the `apps/applications/` folder
   - Deploy each one
   - Read manifest files and start the applications

## 📱 Applications

### Nginx
- **Namespace:** `nginx`
- **Port:** 80
- **Type:** Web server
- **Image:** `nginx:alpine`

### Redis
- **Namespace:** `redis` (shared)
- **Port:** 6379
- **Type:** Cache/Database
- **Image:** `redis:alpine`
- **Usage:** Can be used by multiple applications

### Guestbook
- **Namespace:** `guestbook`
- **Port:** 80
- **Type:** Web application
- **Image:** `gcr.io/google-samples/gb-frontend:v5`
- **Dependency:** Redis (`redis.redis.svc.cluster.local:6379`)

## 🔧 Adding New Applications

1. **Create a child application file:**
   ```yaml
   # apps/applications/my-app.yml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/onurglr/app-of-apps-argocd.git
       targetRevision: HEAD
       path: apps/manifests/my-app
     destination:
       server: https://kubernetes.default.svc
       namespace: my-app
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
         - CreateNamespace=true
   ```

2. **Create the manifest folder:**
   ```bash
   mkdir -p apps/manifests/my-app
   ```

3. **Add manifest files:**
   - `apps/manifests/my-app/deployment.yml`
   - `apps/manifests/my-app/service.yml`

4. **Commit to Git:**
   ```bash
   git add .
   git commit -m "Add my-app"
   git push
   ```

5. **ArgoCD will automatically deploy the new application!**

## 🔍 Verification

### Via ArgoCD UI
- Navigate to ArgoCD UI
- View the root application
- Check child applications and their status

### Via kubectl
```bash
# List all applications
kubectl get applications -n argocd

# Check a specific application
kubectl get application nginx-app -n argocd

# Check pods
kubectl get pods -n nginx
kubectl get pods -n redis
kubectl get pods -n guestbook
```

## 📚 Learning Resources

This project was created for educational purposes. Detailed explanations can be found in each file.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is for educational purposes.
