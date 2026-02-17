<img width="768" height="512" alt="12" src="https://github.com/user-attachments/assets/c766a19b-9d84-463c-b88d-9c3b1f21e91f" />

# Running Real Open-Source Applications on Kubernetes 🚀

## 🎯 Goal
Deploy real open-source apps on a self-built Kubernetes cluster with a production mindset.

## 🏗 Architecture
- 3 VMs (1 control plane, 2 workers)
- kubeadm cluster
- Calico networking
- MetalLB load balancer
- NGINX Ingress
- cert-manager for TLS

### Deploy Applications
👉 `docs/app-deployment.md`

Deploy real apps using production practices:

- Persistent storage with NFS
- Deployments & Services
- Ingress routing
- TLS with Let's Encrypt

Apps:

- Linkding (exercise)
- Nextcloud
- Homepage dashboard

### Monitoring & Alerting (coming soon)
👉 `docs/monitoring.md`

- Prometheus
- Grafana
- Blackbox Exporter

---
## 🚀 Quick Start
```bash
kubectl apply -f /linkding -R
kubectl apply -f /nextcloud -R
kubectl apply -f /homepage -R


📝 **Note**

This guide focuses on deploying applications on Kubernetes.
If you don’t have a cluster yet, you can follow my step-by-step guide here:
👉 Building a Kubernetes Cluster from Scratch (kubeadm)

Once your cluster is ready, please go ahead and return to the above deployment steps.
