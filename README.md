<img width="768" height="512" alt="12" src="https://github.com/user-attachments/assets/c766a19b-9d84-463c-b88d-9c3b1f21e91f" />

# Running Real Open-Source Applications on Kubernetes 🚀
I’ve already written an article about building these open-source projects, feel free to check it out on Medium for a deeper look and some extra insights!
[Running Real Open-Source Applications on Kubernetes with Monitoring and Alerting](https://medium.com/@vuhuy999999/running-real-open-source-applications-on-kubernetes-with-monitoring-and-alerting-part-1-930daba016b0) <br>

## 🎯 Goal
Deploy real open-source apps on a self-built Kubernetes cluster with a production mindset.

Before we jump into writing any YAML file and building our projects, let’s stop for a moment and think about what we actually want to achieve here.

Our goal here isn’t just to spin up containers and say “it works.” Instead, we want to truly understand how to deploy open-source applications properly on Kubernetes.<br>

## 🏗 Architecture
- 3 VMs (1 control plane, 2 workers)
- kubeadm cluster
- Calico networking
- MetalLB load balancer
- NGINX Ingress
- cert-manager for TLS
<br>
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
<br>
### Monitoring & Alerting (coming soon)
👉 `docs/monitoring.md`

- Prometheus
- Grafana
- Blackbox Exporter
<br>
---
<br>
## 🚀 Quick Start
```bash
kubectl apply -f /linkding -R
kubectl apply -f /nextcloud -R
kubectl apply -f /homepage -R
```
<br>
<br>
📝 **Note**

This guide focuses on deploying applications on Kubernetes.
If you don’t have a cluster yet, you can follow my step-by-step guide here:
👉 [Building a Kubernetes Cluster from Scratch (kubeadm)](https://medium.com/@vuhuy999999/kubernetes-from-zero-to-automation-with-ansible-77eebc8fc9c3)

Once your cluster is ready, please go ahead and return to the above deployment steps.
