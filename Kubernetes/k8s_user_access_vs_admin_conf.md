# 👤 اضافه کردن کاربر جدید (مثلاً `k8s`) به خوشه Kubernetes + تفاوت با استفاده از `admin.conf`

این راهنما به‌طور کامل و حرفه‌ای دو حالت زیر را بررسی می‌کند:

1. ساخت یک کاربر جدید (مثلاً `k8s`) با گواهی و دسترسی اختصاصی
2. استفاده از فایل `admin.conf` برای اتصال کاربران به خوشه — و تفاوت‌های امنیتی آن

---

## ✅ سناریو ۱: ساخت کاربر جدید با گواهی اختصاصی و دسترسی محدود

### 🎯 هدف:
ساخت یک کاربر به نام `k8s` که از طریق گواهی اختصاصی به خوشه Kubernetes متصل شود و دسترسی کنترل‌شده داشته باشد.

---

### 🔧 مراحل کامل:

#### ۱. تولید کلید خصوصی

```bash
openssl genrsa -out k8s.key 2048
```

#### ۲. ساخت CSR (درخواست گواهی)

```bash
openssl req -new -key k8s.key -out k8s.csr -subj "/CN=k8s/O=dev-team"
```

> CN = نام کاربر، O = گروه (مثلاً `dev-team`)

#### ۳. امضای گواهی با CA خوشه

```bash
openssl x509 -req -in k8s.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out k8s.crt \
  -days 365
```

#### ۴. ساخت kubeconfig اختصاصی

```bash
kubectl config set-cluster my-cluster \
  --server=https://<API_SERVER_IP>:6443 \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --kubeconfig=k8s.kubeconfig

kubectl config set-credentials k8s \
  --client-certificate=k8s.crt \
  --client-key=k8s.key \
  --kubeconfig=k8s.kubeconfig

kubectl config set-context k8s-context \
  --cluster=my-cluster \
  --user=k8s \
  --kubeconfig=k8s.kubeconfig

kubectl config use-context k8s-context --kubeconfig=k8s.kubeconfig
```

---

### 🎯 ساخت Role و RoleBinding برای کنترل دسترسی

#### تعریف Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: read-only
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```

#### تعریف RoleBinding:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-only-binding
  namespace: dev
subjects:
- kind: User
  name: k8s
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: read-only
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f role-read-only.yaml
kubectl apply -f binding.yaml
```

---

## 🟡 سناریو ۲: استفاده از فایل `admin.conf` برای دسترسی

### دستور:

```bash
mkdir -p ~/.kube
cp inventory/mycluster/artifacts/admin.conf ~/.kube/config
export KUBECONFIG=~/.kube/config
```

### ✅ نتیجه:
کاربر (مثلاً `k8s`) به راحتی به خوشه وصل می‌شود.

### ❗ مشکل:
با این روش، کاربر در واقع با هویت و سطح دسترسی `kubernetes-admin` وارد می‌شود → **دسترسی کامل (Cluster Admin)**

---

## 🔐 مقایسه امنیتی دو روش

| روش | نوع دسترسی | امنیت | مناسب برای |
|------|-------------|--------|-------------|
| ساخت گواهی اختصاصی + RBAC | قابل تنظیم | ✅ بالا | محیط واقعی، کاربران زیاد |
| استفاده از `admin.conf` | کامل (Cluster Admin) | ❌ پایین | تست سریع، توسعه شخصی |

---

## ✅ نتیجه‌گیری نهایی

- اگر فقط برای تست کوتاه‌مدت است، می‌توان `admin.conf` را کپی کرد
- ولی اگر می‌خواهی کاربر مجزا با دسترسی مشخص داشته باشی (و امنیت رعایت شود)، باید گواهی و Role مخصوص به او بسازی

---

تهیه‌شده با ❤️ برای مدیران خوشه‌های Kubernetes