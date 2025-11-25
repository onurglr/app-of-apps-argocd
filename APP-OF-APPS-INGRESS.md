# Ingress ile App-of-Apps Pattern

## 🎯 İki Farklı Yaklaşım

### Yaklaşım 1: Ingress Uygulama ile Birlikte (Şu Anki)

```
apps/
├── applications/
│   ├── gitea-app.yml  ← Gitea + Ingress birlikte
│   └── postgresql-app.yml
└── manifests/
    └── gitea/
        ├── deployment.yml
        ├── service.yml
        └── ingress.yml  ← Gitea ile birlikte
```

**Avantajlar:**
- ✅ Gitea silinince ingress de silinir
- ✅ Her app bağımsız
- ✅ GitOps best practice
- ✅ Daha basit yapı

**Dezavantajlar:**
- ❌ Merkezi yönetim yok
- ❌ Çok app olunca dağınık

---

### Yaklaşım 2: Ayrı Ingress Application (App-of-Apps)

```
apps/
├── applications/
│   ├── gitea-app.yml
│   ├── postgresql-app.yml
│   └── ingress-app.yml  ← Ayrı application
└── manifests/
    ├── gitea/
    │   ├── deployment.yml
    │   └── service.yml
    └── ingress/
        └── gitea-ingress.yml  ← Ayrı klasör
```

**Avantajlar:**
- ✅ Merkezi yönetim
- ✅ Tüm ingress'ler tek yerde
- ✅ Çok app için organize
- ✅ App-of-apps pattern tam uyumlu

**Dezavantajlar:**
- ⚠️ Gitea silinince ingress manuel silinmeli
- ⚠️ Biraz daha karmaşık

---

## 📋 Yapı Karşılaştırması

### Root Application (Her İkisinde Aynı):

```yaml
# root-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
spec:
  source:
    path: apps/applications  # Tüm application'ları yönetir
```

### Yaklaşım 1: Gitea ile Birlikte

```yaml
# apps/applications/gitea-app.yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gitea-app
spec:
  source:
    path: apps/manifests/gitea  # Ingress de burada
```

### Yaklaşım 2: Ayrı Application

```yaml
# apps/applications/ingress-app.yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ingress-app
spec:
  source:
    path: apps/manifests/ingress  # Sadece ingress'ler
```

---

## 🚀 Kurulum

### Yaklaşım 1'i Kullanıyorsanız (Şu Anki):

```bash
# Zaten hazır, sadece root-app'i uygula
kubectl apply -f root-application.yaml
```

### Yaklaşım 2'ye Geçmek İsterseniz:

1. **Ingress'i gitea klasöründen çıkar:**
   ```bash
   # apps/manifests/gitea/ingress.yml → apps/manifests/ingress/gitea-ingress.yml
   ```

2. **Ingress application oluştur:**
   ```yaml
   # apps/applications/ingress-app.yml (yukarıda oluşturuldu)
   ```

3. **Root-app otomatik olarak yönetir:**
   ```bash
   kubectl apply -f root-application.yaml
   # Root-app → ingress-app'i bulur ve deploy eder
   ```

---

## 📊 App-of-Apps Yapısı

```
root-app (root-application.yaml)
  │
  ├── postgresql-app (apps/applications/postgresql-app.yml)
  │   └── Manifests: apps/manifests/postgresql/
  │
  ├── gitea-app (apps/applications/gitea-app.yml)
  │   └── Manifests: apps/manifests/gitea/
  │
  └── ingress-app (apps/applications/ingress-app.yml)  ← YENİ
      └── Manifests: apps/manifests/ingress/
```

---

## ✅ Hangi Yaklaşımı Seçmeli?

### Küçük Projeler (2-5 app):
**→ Yaklaşım 1 (Gitea ile birlikte)** ✅

### Büyük Projeler (10+ app):
**→ Yaklaşım 2 (Ayrı application)** ✅

### Merkezi Yönetim İstiyorsanız:
**→ Yaklaşım 2 (Ayrı application)** ✅

---

## 🎯 Öneri

**Şu anki proje için:** Yaklaşım 1 yeterli (Gitea ile birlikte)

**Gelecekte büyürse:** Yaklaşım 2'ye geçebilirsiniz (Ayrı application)

Her iki yaklaşım da app-of-apps pattern ile çalışır! 🚀

