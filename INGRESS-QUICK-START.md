# Ingress Hızlı Başlangıç - Pratik Rehber

## 🚀 5 Dakikada Ingress Kurulumu

### Adım 1: Ingress Controller Kur (Bir Kez)

```bash
# Nginx Ingress Controller kur
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Kontrol et (2-3 dakika bekle)
kubectl get pods -n ingress-nginx
# Çıktı: ingress-nginx-controller-xxx  1/1   Running

# Service'i kontrol et
kubectl get svc -n ingress-nginx
# Çıktı: ingress-nginx-controller   LoadBalancer/NodePort
```

### Adım 2: Service'i NodePort Yap (Local için)

```bash
# Service'i düzenle
kubectl edit svc ingress-nginx-controller -n ingress-nginx

# type: LoadBalancer → type: NodePort
# nodePort: 3000 ekle (ports bölümüne)
```

Veya manifest ile:

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

### Adım 3: Ingress Resource Oluştur

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

### Adım 4: ArgoCD ile Deploy

```bash
# Root application'ı uygula
kubectl apply -f root-application.yaml

# ArgoCD otomatik olarak:
# 1. Gitea'yı deploy eder
# 2. Service'i oluşturur
# 3. Ingress'i oluşturur
```

### Adım 5: Test Et

```bash
# Ingress durumunu kontrol et
kubectl get ingress -n gitea

# Tarayıcıda aç
# http://localhost:3000
```

---

## 🔍 Sorun Giderme

### Problem: Ingress oluştu ama erişilemiyor

```bash
# 1. Ingress Controller çalışıyor mu?
kubectl get pods -n ingress-nginx

# 2. Ingress resource doğru mu?
kubectl describe ingress gitea-ingress -n gitea

# 3. Service var mı?
kubectl get svc gitea -n gitea

# 4. Pod'lar çalışıyor mu?
kubectl get pods -n gitea

# 5. Ingress Controller loglarını kontrol et
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### Problem: 404 Not Found

- Service adı doğru mu? (`kubectl get svc -n gitea`)
- Port numarası doğru mu? (Service port = Ingress backend port)
- Namespace doğru mu? (Ingress ve Service aynı namespace'de)

### Problem: Connection Refused

- Ingress Controller service'i NodePort mu? (`kubectl get svc -n ingress-nginx`)
- NodePort numarası doğru mu? (localhost:3000)
- Firewall port'u açık mı?

---

## 📋 Kontrol Listesi

- [ ] Ingress Controller kurulu ve çalışıyor
- [ ] Ingress Controller service'i NodePort (veya LoadBalancer)
- [ ] Application service'i ClusterIP
- [ ] Ingress resource oluşturuldu
- [ ] Ingress'teki service adı doğru
- [ ] Ingress'teki port numarası doğru
- [ ] Namespace'ler eşleşiyor
- [ ] Pod'lar çalışıyor

---

## 🎯 Özet Komutlar

```bash
# Tüm ingress'leri listele
kubectl get ingress --all-namespaces

# Belirli bir ingress detayı
kubectl describe ingress gitea-ingress -n gitea

# Ingress Controller logları
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Ingress Controller service
kubectl get svc -n ingress-nginx

# Application service
kubectl get svc -n gitea
```

---

## ✅ Başarı Kriterleri

1. `kubectl get ingress -n gitea` → Ingress görünüyor
2. `kubectl describe ingress -n gitea` → Backend service doğru
3. Tarayıcıda `http://localhost:3000` → Gitea açılıyor

