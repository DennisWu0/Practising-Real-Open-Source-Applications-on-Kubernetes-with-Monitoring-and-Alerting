Higuys! In the last article, we successfully built our Kubernetes cluster. Now it’s time to make things more exciting DEPLOYING. Instead of diving deeper into theory, we’ll learn by deploying real applications and operating them with a production mindset.

This is the purpose of this article, and I’m glad you’re here to explore it with me.

In this guide, we’ll take real open-source projects, deploy them on our self-built Kubernetes cluster, and then add monitoring and alerting.

This article is split into two practical parts:

Part 1: Deploying open-source applications on Kubernetes
Part 2: Building a monitoring and alert system with Prometheus, Grafana, and Blackbox Exporter
Besides, if you want a deep understanding of Kubernetes components and how to build a cluster from scratch, I‘ve provided the link to my article below. Once understand how Kubernetes works internally, we can use Ansible or any other tools to significantly speed up the cluster setup process.

- [**Hands-On Kubernetes: Manually Build a K8s Cluster with kubeadm**](https://medium.com/@vuhuy999999/manual-building-a-k8s-cluster-via-kubeadm-71e395b8f623)
- [**Kubernetes from Zero to Automation with Ansible**](https://medium.com/@vuhuy999999/kubernetes-from-zero-to-automation-with-ansible-77eebc8fc9c3)

Here is what I set out to build:

Infrastructure Layer

Three VMs on local (one master, two workers)
Ansible for faster VM creation
Cluster Layer

Kubernetes installed via kubeadm
Calico for pod networking
MetalLB for load balancing
Platform Layer

Ingress as the ingress controller
cert-manager for TLS automation
Observability Layer

Prometheus for metrics collection
Grafana for visualization
Prerequisites
Before starting this article, make sure the following environment is ready:

MetablLB
An Ingress controller installed (I used NGINX Ingress in our lab)
A DNS like DuckDNS or any free source
Router port forwarding for HTTP (port 80) and HTTPS (port 443) (I used a router in here to easily route the packget but if u dont have one, u can use the port-forward command instead.)
In the note-metabllb.txt, the configuration steps are detailed, but the last two steps, DNS and router, need to be configured manually, but this shouldn’t cause you worry.

![1_qX6iuVL1DehStxaJr1Y3mA](https://github.com/user-attachments/assets/c0a613c6-82a8-47fa-8938-ba28fd82806c)

What Are the Goals Here?

Before we jump into writing any YAML file and building our projects, let’s stop for a moment and think about what we actually want to achieve here.

Our goal isn’t just to spin up containers and say “it works.” Instead, we want to truly understand how to deploy open-source applications properly on Kubernetes.

Open-Source Projects We’ll Work With
Many thanks to the authors of these open-source projects for their invaluable contributions. My friend, plz give them a star or share cause they are doing a wonderful thing. Okay, so this series focuses on three open sources:

- [**Linkding](https://github.com/sissbruecker/linkding)
- [**Nextcloud](https://github.com/nextcloud)
- [**Homepage](https://github.com/gethomepage/homepage):**

In order to keep things focused and practical, we’ll take a deep dive into the Linkding project. The other two projects are intentionally left as hands-on exercises for you to explore on your own way. Once you understand the workflow for one application, you’ll notice the same deployment patterns apply to most modern open-source apps. Based on this pattern, we can expand later to the new level.

Feel free to experiment with any open-source project on GitHub as part of your learning journey (just make sure it’s for non-commercial use).

I also want to give credit where it’s due. I was inspired by Mischa van den Burg, one of his videos gave me the confidence to deploy my very first Kubernetes application. 👉  [Your First Real Kubernetes App!](https://www.youtube.com/watch?v=K42Ux_KU4DM&t=65s](https://www.youtube.com/watch?v=K42Ux_KU4DM&t=65s)

Step 1: Deploying Linkding on Kubernetes
The deployment flow I follow is usually:

![1_6zU-w10cBrjItVfHxREk1w](https://github.com/user-attachments/assets/b9ce69a0-e9f5-44b6-b957-8da9d48d1593)

Storage is where things start to get interesting, and also where many real-world considerations begin. Before we move on to Deployments, we first need to make sure our application data can persist beyond the lifecycle of a Pod. After all, containers can be recreated at any time, but the data shouldn’t disappear with them.

Since the goal here is to practice with a production mindset, I didn’t want to rely solely on local PersistentVolumes. Instead, I chose to use a dedicated NFS server, which is a simple and common way to provide shared storage in lab and small-scale environments.

So how do we set this up? A great starting point is the step-by-step guide  “[How to Set Up an NFS Mount on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)”  by Brian Boucheron and Anish Singh Walia on DigitalOcean. The tutorial is clear and beginner-friendly, making NFS an easy and practical option when you’re first experimenting with shared storage.

Or you can check my note-nfs-server.txt

petops@master:~/projects_backup_from_201$ cat note-nfs-server.txt
1. Install NFS server on the separate VM
sudo apt update
sudo apt install nfs-kernel-server -y
2. Create a shared directory
sudo mkdir -p /data/nfs
sudo chown -R nobody:nogroup /data/nfs
sudo chmod -R 777 /data/nfs
3. Export this directory
sudo vim /etc/exports
#Add this line
/data/nfs  *(rw,sync,no_subtree_check,no_root_squash)
#Then apply it:
sudo exportfs -rav
4. Install NFS client on all Kubernetes nodes
sudo apt install nfs-common -y
After the NFS export is ready, Kubernetes can reference it via a PersistentVolume. And then we can apply this command kubectl apply -f storage.yaml to create the PV and PVC, then data is stored in 192.168.0.201 NFS server.

petops@master:~/projects_backup_from_201/linkding$ cat storage.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/linkding
    server: 192.168.0.201
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
Although Kubernetes allows you to run applications using raw Pod manifests, this approach is only suitable for quick experiments or debugging. A Deployment wraps Pods with declarative lifecycle management, ensuring the application is automatically restarted, updated, and kept desired. My personal loves Deployment, so in this Deployment I use a single-replica Linkding using a stable container image sissbruecker/linkding:1.44.1-plus-alpine, exposes the service on port 9090, and persists application data by mounting a PersistentVolumeClaim into the container.

petops@master:~/projects_backup_from_201/linkding$ cat depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: linkding-deployment
  labels:
    app: linkding
spec:
  replicas: 1
  selector:
    matchLabels:
      app: linkding
  template:
    metadata:
      labels:
        app: linkding
    spec:
      containers:
      - name: linkding
        image: sissbruecker/linkding:1.44.1-plus-alpine
        ports:
        - containerPort: 9090
        volumeMounts:
        - mountPath: /etc/linkding/data
          name: mypd
      volumes:
      - name: mypd
        persistentVolumeClaim:
          claimName: mypvc
At this point, we can see that the PV, PVC, and Pod have all been created successfully.

![1_N1I7aPcGYIUvnnuNbjD1sQ](https://github.com/user-attachments/assets/97966e6b-8473-49d4-905f-ca2dabdd5e67)

Next, we will move to the Service. Since Linkding listens on port 9090 internally, we map it to port 80 inside the cluster. I intentionally use ClusterIP, because I don’t want external traffic bypassing Ingress and hitting the service directly. If you want to check on the private nextwork, you can change it to NodePort and access the private IP address to see if the project works or not.

petops@master:~/projects_backup_from_201/linkding$ cat linkding-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: linkding-service
spec:y
  selector:
    app: linkding
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9090
  type: ClusterIP
The service is created:
![1_cYPHRHBapkEXOanI6yn_Mw](https://github.com/user-attachments/assets/dc4fe6c0-05e3-45f4-af3d-2db451a7b0bc)


Besides, to get the new IP address for our Ingress, we still need this Yaml file to define the range of Ingress IP.
![1_ezzfwawA7H-3eaYbFNfqsg](https://github.com/user-attachments/assets/544aee3a-0ab7-462d-8386-175a1a1a30d7)


So the result of this step is a new IP Ingress address. When users access the DNS, they will meet ingress first, then routing to the internal linking structure.
![1_j5cpzajJh9p6YNtY3zjIzA](https://github.com/user-attachments/assets/1e21ed75-567e-412f-aac2-0e74752c6829)

Once everything is wired correctly, traffic flows like this:

User → Publish IP → Router → Ingress → Service → Pod

Adding TLS with Let’s Encrypt
When we first access the site, the browser will show a warning about an insecure connection.
![1_UD9Iwf4DxSeO4eXhQzUGcw](https://github.com/user-attachments/assets/c2bfa075-c234-49e4-912b-20a9446ed51e)


In order to fix this, we integrate Let’s Encrypt using Kubernetes certificates. The configuration lives in the ClusterIssuer YAML. After a few minutes (don’t rush this), the certificate is issued.

petops@master:~/projects_backup_from_201/linkding$ cat cluster.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: <https://acme-v02.api.letsencrypt.org/directory>
    email: [your own gmail]
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
One important real-world lesson here: Because I’m using a home router, I had to forward port 80 to my VM. Otherwise, the HTTP challenge would stay pending forever. automatically.

Now we have HTTPS for free.
![1_o-4JN7b2rl2YmAriDrpXFw](https://github.com/user-attachments/assets/7090b94c-3e13-43f4-8b38-3a457cb4b5b8)

Final Touch: Creating the Superuser
Since Linkding doesn’t create a superuser automatically, we need to do this manually. To start, we can exec into the running Pod:

kubectl exec -it <pod-name> -- sh
From these steps, we create the admin user manually.
![1__5n6oChDi2iwfAe8sK8jwQ](https://github.com/user-attachments/assets/a065624c-8a21-46aa-b0af-82f55e6c8ceb)

🎉 I know the config process can be boring, but if you’ve made it this far Congratulations! You’ve now deployed a complete, production-style application on Kubernetes.

![1_lH2SPdP4FBsjwkzYKTs7PA](https://github.com/user-attachments/assets/62518fbe-9ad3-4b17-9b54-fcc05a93c831)

Let’s try it and give me some feedback if it benefits you. Each small step brings us closer to mastering K8s 💪.
