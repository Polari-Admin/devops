# Kubernetes TLS Setup & Debug Guide (Manual TLS)

This guide helps you verify that TLS is properly configured in your Kubernetes cluster using a manually-created TLS certificate (e.g., self-signed).

---

## ✅ 1. Check for TLS Secret

Run the following command to list your secrets:

```bash
kubectl get secret
```

You should see a secret like:

```
echo-tls-secret     kubernetes.io/tls   2      10m
```

Make sure the type is `kubernetes.io/tls`.

---

## ✅ 2. Check Ingress Configuration

Inspect the Ingress resource:

```bash
kubectl describe ingress echo-ingress
```

Expected output includes:

```
TLS:
  echo.example.com terminates echo-tls-secret
Rules:
  Host               Path  Backends
  ----               ----  --------
  echo.example.com   /     echo-service:80
Annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: true
```

This confirms the Ingress is using your TLS secret.

---

## ✅ 3. Test TLS Access via `curl`

### If using self-signed certificate:

```bash
curl https://echo.example.com -k -v
```

Look for lines like:

```
* SSL connection using TLSv1.3
* Server certificate:
*  subject: CN=echo.example.com
```

### If using valid certificate (e.g., Let's Encrypt):

```bash
curl https://echo.example.com -v
```

Make sure you don’t see SSL verification errors.

---

## ✅ 4. Test in Browser

Open your browser and go to:

```
https://echo.example.com
```

- If TLS is working: you’ll see a secure connection icon (lock).
- If using self-signed: you’ll get a warning but can still proceed.

If you're testing locally, map the domain in `/etc/hosts`:

```
<Ingress_Controller_IP> echo.example.com
```

---

## ✅ 5. Check Ingress Controller Logs

First, get the pod name:

```bash
kubectl get pods -n ingress-nginx
```

Then view logs:

```bash
kubectl logs -n ingress-nginx <nginx-ingress-pod-name>
```

Look for messages like:

```
Successfully reloaded ingress with TLS cert for echo.example.com
```

Or errors such as:

```
Error loading TLS certificate for echo.example.com: no such secret
```

---

## ✅ Summary Checklist

| Step | Command | Purpose |
|------|---------|---------|
| ✅ Secret | `kubectl get secret` | Confirm TLS secret exists |
| ✅ Ingress | `kubectl describe ingress` | Verify secret is attached |
| ✅ Curl | `curl -v https://yourdomain` | Test HTTPS response |
| ✅ Browser | `https://yourdomain` | Confirm TLS in browser |
| ✅ Logs | `kubectl logs` | Debug ingress & TLS errors |

---

Happy debugging! 🛠️

