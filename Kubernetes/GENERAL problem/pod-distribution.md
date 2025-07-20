# Kubernetes Pod Distribution Optimization Guide

This guide explains how to ensure a balanced distribution of pods across Kubernetes cluster nodes **without causing instability** due to inaccurate resource definitions.

## 🚨 The Problem

If one node in your cluster has significantly higher **memory usage** than others, it's likely that:

- Pods are **not evenly distributed** across nodes.
- Scheduler doesn't have enough information to balance them.
- Some pods **lack resource requests/limits**, causing over-scheduling.

## ✅ Goals

- Distribute pods more evenly across nodes.
- Avoid pod crashes due to under-provisioning.
- Minimize operational risk while optimizing resource usage.

---

## ✅ Recommended Strategy

### 1. Define `resources.requests` only (skip `limits`)

By setting `requests` only, you help the scheduler understand how much memory/cpu the pod needs — **without limiting** how much it can consume.

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
# No limits section
```

> This prevents OOMKilled issues while still improving scheduling decisions.

---

### 2. Use **Topology Spread Constraints**

Encourage Kubernetes to spread pods evenly across nodes (or zones):

```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: "kubernetes.io/hostname"
  whenUnsatisfiable: ScheduleAnyway
  labelSelector:
    matchLabels:
      app: my-app
```

---

### 3. Avoid overly restrictive `affinity` and `nodeSelector`

Check for any affinity rules or nodeSelectors that force pods onto specific nodes. These reduce scheduling flexibility.

---

### 4. Use the **Vertical Pod Autoscaler (VPA)**

If you're unsure about actual resource consumption, VPA can recommend (or auto-set) the correct values:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Or "Off" for recommendation only
```

---

### 5. Install and Configure the **Descheduler**

To rebalance pods over time without manual intervention:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/descheduler/main/kubernetes/deployment.yaml
```

Descheduler strategies to enable:

- `RemoveDuplicates`
- `LowNodeUtilization`
- `RemovePodsViolatingTopologySpreadConstraint`

---

### 6. Monitor Usage via Prometheus + Grafana

Track real-world memory and CPU usage before adjusting requests:

```promql
avg_over_time(container_memory_usage_bytes{container!="POD",pod=~"my-app.*"}[1h])
```

This lets you optimize based on real usage instead of guesses.

---

## 🧹 Summary Table

| Step | Tool / Setting             | Purpose                                      |
| ---- | -------------------------- | -------------------------------------------- |
| 1️⃣  | `requests` only            | Helps scheduler without risking pod failures |
| 2️⃣  | TopologySpreadConstraints  | Ensures even pod distribution                |
| 3️⃣  | Remove hard affinity rules | Increases flexibility                        |
| 4️⃣  | VPA (Auto or Recommend)    | Adjusts or suggests resource settings        |
| 5️⃣  | Descheduler                | Rebalances overused nodes                    |
| 6️⃣  | Prometheus                 | Tracks actual resource usage                 |

---

## 📌 Notes

- Avoid setting `limits` unless you're confident in peak usage.
- Use `updateMode: Off` in VPA if you just want safe recommendations.
- Test changes in staging before applying in production.

---

Created with 💡 by a Kubernetes enthusiast.

