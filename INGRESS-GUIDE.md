# Ingress Kullanım Kılavuzu - Kubernetes & ArgoCD

## 📚 İçindekiler
1. [Ingress Nedir?](#ingress-nedir)
2. [Ingress Controller Nedir?](#ingress-controller-nedir)
3. [Kurulum](#kurulum)
4. [Nasıl Çalışır?](#nasıl-çalışır)
5. [ArgoCD ile Kullanım](#argocd-ile-kullanım)
6. [Standart Kullanım Örnekleri](#standart-kullanım-örnekleri)

---

## 🎯 Ingress Nedir?

**Ingress**, Kubernetes'te HTTP/HTTPS trafiğini cluster içindeki servislere yönlendiren bir API kaynağıdır.

### Basit Analoji:
```
Internet → Ingress (Kapı) → Service (Adres) → Pod (Ev)
```

### Neden Gerekli?
- **Service (ClusterIP)**: Sadece cluster içinden erişilebilir
- **Service (NodePort)**: Her node'da port açmak gerekir (güvenlik riski)
- **Ingress**: Tek bir entry point, temiz URL'ler, SSL yönetimi

---

## 🔧 Ingress Controller Nedir?

**Ingress Controller**, Ingress resource'larını okuyan ve trafiği yönlendiren bir pod'dur.

### Popüler Ingress Controller'lar:
1. **Nginx Ingress Controller** (En yaygın)
2. **Traefik**
3. **HAProxy**
4. **Istio Gateway**

### Ingress vs Ingress Controller:
- **Ingress Resource**: Yapılandırma (ne yapılacağını söyler)
- **Ingress Controller**: Uygulama (gerçekten yönlendirme yapar)

---

## 📦 Kurulum

### Adım 1: Ingress Controller Kurulumu

#### Nginx Ingress Controller (Önerilen):

```bash
# Kubernetes 1.18+
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# veya Helm ile
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
```

#### Kontrol:
```bash
# Ingress Controller pod'unu kontrol et
kubectl get pods -n ingress-nginx

# Service'i kontrol et
kubectl get svc -n ingress-nginx
```

### Adım 2: Ingress Controller Service Yapılandırması

Ingress Controller varsayılan olarak **LoadBalancer** veya **NodePort** ile çalışır.

#### NodePort ile (Local/Demo):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 3000  # localhost:3000
  selector:
    app.kubernetes.io/name: ingress-nginx
```

#### LoadBalancer ile (Cloud):
```yaml
type: LoadBalancer  # Cloud provider otomatik IP verir
```

---

## ⚙️ Nasıl Çalışır?

### 1. Trafik Akışı:

```
┌─────────────────────────────────────────┐
│  1. Kullanıcı: http://localhost:3000   │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  2. Ingress Controller (Port 3000)     │
│     - Ingress resource'ları okur        │
│     - Routing kurallarını uygular       │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  3. Ingress Resource                    │
│     - Host: localhost                   │
│     - Path: /                           │
│     - Backend: gitea service            │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  4. Service (gitea)                     │
│     - ClusterIP: gitea.gitea.svc        │
│     - Port: 3000                        │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  5. Pod (gitea-xxx)                     │
│     - Container Port: 3000              │
└─────────────────────────────────────────┘
```

### 2. Ingress Resource Yapısı:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitea-ingress
  namespace: gitea
spec:
  ingressClassName: nginx  # Hangi controller kullanılacak
  rules:
  - host: localhost        # Domain adı
    http:
      paths:
      - path: /            # URL path
        pathType: Prefix
        backend:
          service:
            name: gitea     # Service adı
            port:
              number: 3000  # Service port
```

---

## 🚀 ArgoCD ile Kullanım

### Yapı 1: App-of-Apps Pattern (Önerilen)

```
apps/
├── applications/
│   ├── gitea-app.yml
│   └── postgresql-app.yml
└── manifests/
    ├── gitea/
    │   ├── deployment.yml
    │   ├── service.yml
    │   └── ingress.yml  ← Gitea ile birlikte
    └── postgresql/
        ├── deployment.yml
        └── service.yml
```

#### Gitea Application:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gitea-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/user/repo.git
    path: apps/manifests/gitea  # Ingress de burada
  destination:
    namespace: gitea
```

#### Ingress Resource:
```yaml
# apps/manifests/gitea/ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitea-ingress
  namespace: gitea
spec:
  ingressClassName: nginx
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: gitea
            port:
              number: 3000
```

### Yapı 2: Ayrı Ingress Application

```
apps/
├── applications/
│   ├── gitea-app.yml
│   └── ingress-app.yml  ← Ayrı application
└── manifests/
    ├── gitea/
    │   ├── deployment.yml
    │   └── service.yml
    └── ingress/
        └── ingress.yml
```

---

## 📝 Standart Kullanım Örnekleri

### Örnek 1: Basit Ingress (Tek Host)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ingress
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

### Örnek 2: Path-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 3000
```

### Örnek 3: Multiple Hosts

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: gitea.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: gitea
            port:
              number: 3000
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8080
```

### Örnek 4: SSL/TLS (HTTPS)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

---

## 🔍 Kontrol ve Debug

### Ingress Durumunu Kontrol:

```bash
# Tüm ingress'leri listele
kubectl get ingress --all-namespaces

# Belirli bir ingress'i detaylı gör
kubectl describe ingress gitea-ingress -n gitea

# Ingress Controller loglarını gör
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### Yaygın Sorunlar:

1. **Ingress Controller yok:**
   ```bash
   kubectl get pods -n ingress-nginx
   # Eğer yoksa: Yukarıdaki kurulum adımlarını takip et
   ```

2. **Service bulunamıyor:**
   ```bash
   kubectl get svc -n gitea
   # Ingress'teki service adının doğru olduğundan emin ol
   ```

3. **Port uyumsuzluğu:**
   ```bash
   kubectl get svc gitea -n gitea -o yaml
   # Ingress'teki port ile service port'unun eşleştiğinden emin ol
   ```

---

## ✅ Özet

1. **Ingress Controller kur** (bir kez)
2. **Ingress Resource oluştur** (her uygulama için)
3. **Service'in ClusterIP olduğundan emin ol**
4. **ArgoCD ile deploy et**

### Minimal Kurulum:
```bash
# 1. Ingress Controller kur
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# 2. Ingress Resource oluştur (yukarıdaki örneklerden biri)

# 3. ArgoCD ile sync et
```

---

## 📚 Ek Kaynaklar

- [Kubernetes Ingress Docs](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [ArgoCD Ingress Examples](https://argo-cd.readthedocs.io/en/stable/user-guide/ingress/)

