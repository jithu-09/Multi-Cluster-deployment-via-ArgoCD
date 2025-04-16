# Multi-Cluster-deployment-via-ArgoCD

## Hub-and-Spoke GitOps Model

This project demonstrates a **GitOps-based Continuous Deployment (CD)** setup using **ArgoCD** in a **hub-and-spoke architecture** with multiple EKS clusters. The hub cluster runs a highly available ArgoCD setup which manages deployments to all the spoke clusters.

## 🚀 Why GitOps with ArgoCD?

Traditional CD tools (e.g., shell scripts, Ansible) do not offer:
- Change traceability  
- Rollbacks  
- Auditing & monitoring  
- Auto-healing  

**ArgoCD**, following the GitOps model, solves this by:
- Using Git as the **single source of truth**
- Reverting unauthorized changes in the cluster
- Automatically syncing with version-controlled manifests
- Providing visibility into deployed applications

✅ Recommended for CD pipelines, while CI is usually handled by tools like GitHub Actions, Jenkins, etc.

## 🏗️ Architecture: Hub-and-Spoke vs Standalone

### Hub-Spoke Model (Recommended):
- One highly available ArgoCD installation in a **hub cluster**
- Manages deployments to multiple **spoke clusters**
- Centralized, easier to manage, audit, and monitor

### Standalone Model:
- Each cluster has its own ArgoCD instance
- Harder to manage at scale

## 🔧 Prerequisites

- AWS CLI  
- eksctl  
- kubectl  
- ArgoCD CLI  

## 🌐 Setting Up Multiple EKS Clusters

```bash
aws configure

# Create clusters
eksctl create cluster --name multi-cluster-1 --region us-east-1
eksctl create cluster --name multi-cluster-2 --region us-west-1

# Note: After each eksctl create, kubectl points to the newly created cluster

# View all cluster contexts
kubectl config get-contexts

# Set context to hub cluster
kubectl config use-context <hub-cluster-context-name>
```

## 🧩 Install ArgoCD in Hub Cluster

```bash
# Create namespace and install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Optional: Enable HTTP (insecure) Access

```bash
kubectl edit cm argocd-cmd-params-cm -n argocd
# Add:
# data:
#   server.insecure: "true"
```

### Expose ArgoCD Server

```bash
kubectl edit svc argocd-server -n argocd
# Change type from ClusterIP to NodePort

kubectl get nodes -o wide
# Note down external IP & nodeport
```

### Update EC2 Security Group

Add an inbound rule to allow traffic on the ArgoCD NodePort for your worker nodes.

### Get Initial Admin Password

```bash
kubectl edit secret argocd-initial-admin-secret -n argocd
# Copy the value of the "password" field and decode:
echo <base64-password> | base64 --decode
```

## 💻 Install ArgoCD CLI

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

## 🔐 Login to ArgoCD

```bash
argocd login <node-external-ip>:<nodeport>
```

## ➕ Add Spoke Clusters to ArgoCD

```bash
# View all cluster contexts
kubectl config get-contexts

# Switch context to the cluster you want to add
kubectl config use-context <context-of-cluster-to-add>

# Add it to ArgoCD
argocd cluster add <context-of-cluster-to-add>
```

In the ArgoCD UI:  
**Settings → Clusters**  
- `in-cluster`: the hub cluster  
- All added spoke clusters will also appear here  
> You can **delete** clusters here, but **cannot add** clusters via UI

## 🛠️ Deploy Applications with ArgoCD

1. Open ArgoCD UI
2. Go to **Applications → + New App**
3. Fill out:
   - App Name
   - Project Name
   - Sync Policy: **Automatic**
   - Git Repo URL
   - Path to manifests
   - Destination cluster
   - Target namespace
4. Click **Create**

ArgoCD will now:
- Monitor the Git repository
- Detect changes
- Automatically deploy to the specified cluster

## 📚 Resources

- GitOps: [https://opengitops.dev](https://opengitops.dev)  
- ArgoCD: [https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)  
- eksctl: [https://eksctl.io](https://eksctl.io)

---

Enjoy a fully GitOps-powered multi-cluster deployment workflow with ArgoCD 🎯
