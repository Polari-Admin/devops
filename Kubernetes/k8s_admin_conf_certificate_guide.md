# 🔐 بررسی و بازسازی گواهی کاربر در Kubernetes (`admin.conf`)

این راهنما به شما نشان می‌دهد که چگونه گواهی موجود در فایل `admin.conf` را بررسی کرده و در صورت نیاز آن را بازسازی (regenerate) کنید.

---

## ✅ بخش اول: بررسی اعتبار گواهی

### 🔧 ۱. استخراج گواهی از فایل `admin.conf`

```bash
grep client-certificate-data /etc/kubernetes/admin.conf | awk '{print $2}' | base64 -d > admin.crt
```

---

### 🔧 ۲. مشاهده جزئیات گواهی با OpenSSL

```bash
openssl x509 -in admin.crt -noout -text
```

### ✅ نکات مهم در خروجی:

| بخش | توضیح |
|------|-------|
| **Issuer** | باید با CA خوشه مطابقت داشته باشد |
| **Subject** | معمولاً: `CN=kubernetes-admin` |
| **Validity** | بررسی `Not After` برای انقضای گواهی |
| **Key Usage** | باید شامل `TLS Web Client Authentication` باشد |

---

## ✅ بخش دوم: بازسازی (Regenerate) گواهی کاربر

### 🎯 پیش‌نیاز:
دسترسی به `ca.crt` و `ca.key` خوشه (معمولاً در مسیر `/etc/kubernetes/pki/`)

---

### 🔧 مراحل ساخت گواهی جدید برای `kubernetes-admin`

#### ۱. تولید کلید خصوصی

```bash
openssl genrsa -out kubernetes-admin.key 2048
```

#### ۲. ساخت CSR (Certificate Signing Request)

```bash
openssl req -new -key kubernetes-admin.key -out kubernetes-admin.csr -subj "/CN=kubernetes-admin/O=system:masters"
```

#### ۳. امضای CSR با گواهی CA

```bash
openssl x509 -req -in kubernetes-admin.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out kubernetes-admin.crt \
  -days 365
```

#### ۴. تولید فایل kubeconfig جدید

```bash
kubectl config set-credentials kubernetes-admin \
  --client-certificate=kubernetes-admin.crt \
  --client-key=kubernetes-admin.key

kubectl config set-context kubernetes-admin@kubernetes \
  --cluster=kubernetes \
  --user=kubernetes-admin

kubectl config use-context kubernetes-admin@kubernetes
```

---

### ✅ روش جایگزین: استفاده از kubeadm

اگر از `kubeadm` استفاده کرده‌اید، می‌توانید فایل را دوباره تولید کنید:

```bash
sudo kubeadm init phase kubeconfig admin
```

---

## 🛡️ امنیت

- فایل `admin.conf` شامل کلید خصوصی و گواهی است → **دسترسی محدود بدهید**
- از اشتراک‌گذاری `client-key-data` یا `ca.key` خودداری کنید

---

## 📌 خلاصه سریع

| اقدام | دستور |
|-------|--------|
| بررسی گواهی | `openssl x509 -in admin.crt -noout -text` |
| بازسازی گواهی | با `openssl` و `ca.key` |
| بازسازی admin.conf | `kubeadm init phase kubeconfig admin` |

---

تهیه‌شده با ❤️ برای مدیران خوشه‌های Kubernetes