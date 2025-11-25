# Ingress Manifests

Bu klasör, app-of-apps pattern ile yönetilen tüm Ingress resource'larını içerir.

## 📁 Yapı

```
apps/manifests/ingress/
├── gitea-ingress.yml          # Gitea için Ingress
└── ingress-controller-service.yml  # Ingress Controller Service (opsiyonel)
```

## 🚀 Kullanım

### 1. Ingress Controller Kurulumu (Bir Kez)

Ingress Controller'ı manuel olarak kurmanız gerekir:

```bash
# Nginx Ingress Controller kur
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Kontrol et
kubectl get pods -n ingress-nginx
```

### 2. Ingress Controller Service (Opsiyonel)

Eğer Ingress Controller'ı NodePort ile kullanmak istiyorsanız:

```bash
# Service'i düzenle
kubectl edit svc ingress-nginx-controller -n ingress-nginx

# veya ingress-controller-service.yml'i uygula
# (Not: Bu dosya sadece örnek, gerçek service'i günceller)
```

### 3. Ingress Resource'ları

ArgoCD otomatik olarak `ingress-app` application'ı ile bu klasördeki tüm Ingress resource'larını deploy eder.

## 📋 Mevcut Ingress'ler

- **gitea-ingress**: Gitea uygulaması için Ingress (localhost:3000)

## ➕ Yeni Ingress Ekleme

1. Bu klasöre yeni bir `*-ingress.yml` dosyası ekleyin
2. ArgoCD otomatik olarak deploy edecektir

Örnek:
```yaml
# apps/manifests/ingress/api-ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: api
spec:
  ingressClassName: nginx
  rules:
  - host: api.localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

