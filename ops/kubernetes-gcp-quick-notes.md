# ☸️ Kubernetes & GCP Quick Reference

This README provides essential GCP and Kubernetes commands for working with clusters, namespaces, deployments, and services.

---

## 🔐 Connect to GKE Cluster

```bash
gcloud container clusters get-credentials <cluster_name> --project <project_name> --zone <zone_name>
```

Check current Kubernetes config:

```bash
cat ~/.kube/config
```

---

## 👤 Grant Admin Access to Current User

```bash
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
```

---

## 📦 Connect to Artifact Registry

```bash
gcloud auth configure-docker <zone_name>-docker.pkg.dev
```

---

## 🧭 Working with Namespaces

### 🔍 Get all namespaces

```bash
kubectl get ns
```

### ➕ Create a new namespace

```bash
kubectl create ns <namespace>
```

---

## 🚀 Create a Deployment (Manual)

```bash
kubectl create deployment <deployment_name> \
  --namespace <namespace> \
  --image=<zone>-docker.pkg.dev/<project-name>/<dir>/<image_name>:<image_tag>
```

---

## 🔁 Manage Kubernetes Contexts

### 🔍 Get current contexts

```bash
kubectl config get-contexts
```

### 🔄 Change context

```bash
kubectl config use-context <context_name>
```

---

## ⏪ Roll Back a Deployment

```bash
kubectl rollout undo deployment <deployment_name> -n <namespace>
```

### **How to Roll Back to a Specific Version?**

By default, `kubectl rollout undo` reverts to the **previous version**. If you want to roll back to a **specific version**, first check the rollout history:

```bash
kubectl rollout history deployment <deployment> -n <namespace>
```

You'll get an output like:

```
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Change 1
3         Change 2
```

Then, rollback to a specific revision (e.g., revision `2`):

```bash
kubectl rollout undo deployment <deployment> --to-revision=2 -n <namespace>
```

### **Checking Rollback Status**

After rollback, verify if the correct version is running:

```bash
kubectl get pods -n <namespace> kubectl rollout status deployment <deployment> -n <namespace>
```

---

## 🔍 Inspect Pod Details

```bash
kubectl describe pods -n <namespace> <pod_name>
```

---

## 🌐 Get Services (Look for External IP)

```bash
kubectl get services -n <namespace>
```
