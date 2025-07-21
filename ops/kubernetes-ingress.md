# 🚪 Kubernetes Ingress + SSL Setup Guide

This guide helps you install the NGINX Ingress Controller and configure SSL using a certificate.

---

## 📥 Install Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

> ⚠️ If you're not a cluster admin, request cluster-admin permissions first.

Grant admin access:

```bash
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
```

### 🔍 Get External IP

Once installed, fetch the external IP:

```bash
kubectl get svc -n ingress-nginx
```

Wait for the LoadBalancer IP to be assigned.

---

## 🔒 Add TLS Certificate

### Step 1: Prepare Your Certificate

Ensure you have:

- Your domain certificate (e.g. `your_cert.crt`)
- Private key (e.g. `your_private_key.key`)
- GoDaddy intermediate bundle (e.g. `gd_bundle.crt`)

Combine the cert + bundle:

```bash
cat your_cert.crt gd_bundle.crt > full_chain.pem
```
### Step 2: Create a TLS Secret in Kubernetes

```bash
kubectl create secret tls <tls-secret-name> \
  --namespace <namespace> \
  --cert=full_chain.pem \
  --key=your_private_key.key
```
### Step 3: Reference TLS Secret in Ingress

Sample Ingress resource with TLS:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <ingress-resource-name>
  namespace: <namespace>
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - <your-domain>
      secretName: <tls-secret-name>
  rules:
    - host: <your-domain>
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

---

### Step 4: Apply the Ingress File

```bash
kubectl apply -f <ingress-resources>.yaml
```

After a few seconds, your domain should be secured with SSL!

---

> ✅ **Pro Tip:** You can verify the TLS setup with:

```bash
kubectl describe ingress <ingress-resource-name> -n <namespace>
```
