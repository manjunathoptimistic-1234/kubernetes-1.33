 **EKS Upgrade Process**
1. Upgrading the **EKS Control Plane**
2. Upgrading **Node Groups / Worker Nodes / Fargate**
3. Upgrading **EKS Add-ons**



---

# **EKS Upgrade Process – Commands Guide**

This document provides commands for upgrading:

* EKS Control Plane
* Managed Node Groups / Worker Nodes / Fargate
* EKS Add-ons

---

## ## **1. Upgrade the EKS Control Plane**

You can upgrade the control plane using `eksctl`:

```bash
eksctl upgrade cluster \
  --name eksupgrade \
  --region us-east-1 \
  --approve
```

### Check cluster version:

```bash
aws eks describe-cluster \
  --name eksupgrade \
  --region us-east-1 \
  --query "cluster.version"
```

```bash
kubectl version --short


# Check all pods across all namespaces
kubectl get pods --all-namespaces

# Look for any pods not in Running/Completed state
kubectl get pods --all-namespaces | grep -v "Running\|Completed"

# Check coredns
kubectl get pods -n kube-system | grep coredns

# Check kube-proxy
kubectl get pods -n kube-system | grep kube-proxy

# Check vpc-cni
kubectl get pods -n kube-system | grep aws-node

# Check all add-on statuses via AWS CLI
aws eks list-addons --cluster-name <cluster-name>
aws eks describe-addon --cluster-name <cluster-name> --addon-name coredns

# Check componentstatus
kubectl get componentstatuses

# Check kube-system namespace
kubectl get all -n kube-system

# Look for warnings or errors
kubectl get events --all-namespaces --field-selector type=Warning

# Sort by time
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

##Check Node Health in Detail
# Describe each node to check conditions
kubectl describe nodes | grep -A5 "Conditions:"

# Check node resource usage
kubectl top nodes

# Check node resource usage
kubectl top nodes

##Check EKS Cluster Health via AWS CLI
aws eks describe-cluster \
  --name <cluster-name> \
  --query 'cluster.health'

# Check cluster insights for any issues
aws eks list-insights \
  --cluster-name <cluster-name>

# Check deployments
kubectl get deployments --all-namespaces

# Check daemonsets
kubectl get daemonsets --all-namespaces

# Check statefulsets
kubectl get statefulsets --all-namespaces
https://claude.ai/share/69b6296c-7e4b-4df2-b750-effe4bd9eeb2 - claude AI for k8s upgrade
---
## ## **2. Upgrade Node Groups / Worker Nodes / Fargate**

### List node groups:

```bash
eksctl get nodegroup --cluster eksupgrade --region us-east-1
```

```bash
kubectl get nodes -o wide
```

### Upgrade a Managed Node Group:

```bash
eksctl upgrade nodegroup \
  --cluster eksupgrade \
  --region us-east-1 \
  --name eksupgrade-ng-private \
  --kubernetes-version 1.31 \
  --approve
```

### For Self-Managed Worker Nodes (If applicable)

**Drain the node:**

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

**Upgrade AMI in Launch Template**
(Example for Amazon Linux 2)

```bash
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amazon-eks-node-1.31-*" \
  --region us-east-1
```

Then update Launch Template and perform a rolling update of the ASG.

**Uncordon the node:**

```bash
kubectl uncordon <node-name>
```

---

## ## **3. Upgrade EKS Add-ons**

### **CoreDNS**

```bash
eksctl utils update-coredns \
  --cluster eksupgrade \
  --region us-east-1 \
  --approve
```

### **kube-proxy**

```bash
eksctl utils update-kube-proxy \
  --cluster eksupgrade \
  --region us-east-1 \
  --approve
```

### **VPC CNI (aws-node)**

```bash
eksctl utils update-aws-node \
  --cluster eksupgrade \
  --region us-east-1 \
  --approve
```

### **Cluster Autoscaler (If installed via Helm)**

Get the recommended version first:

```bash
kubectl get deployment cluster-autoscaler -n kube-system -o yaml | grep image
```

Upgrade using Helm:

```bash
helm upgrade cluster-autoscaler \
  autoscaler/cluster-autoscaler-chart \
  --namespace kube-system
```

### **Metrics Server (If using Helm)**

```bash
helm upgrade metrics-server \
  metrics-server/metrics-server \
  --namespace kube-system
```

# **4. Validate Add-on Versions**

```bash
kubectl get pods -n kube-system
```

You should see:

* coredns pods recreated
* kube-proxy DaemonSet updated
* aws-node updated


# ## **5. Validate the Upgrade**

### Check node versions:

```bash
kubectl get nodes
```

### Check system pods:

```bash
kubectl get pods -n kube-system
```

### Check API server:

```bash
kubectl version --short
```
https://claude.ai/share/69b6296c-7e4b-4df2-b750-effe4bd9eeb2

# **6. Final Verification Test**

### Test cluster health:

```bash
kubectl get componentstatuses
```

### Test workload scheduling:

```bash
kubectl run nginx-test \
  --image=nginx \
  --restart=Never
```

Check pod:

```bash
kubectl get pod nginx-test -o wide
```

Delete test pod:

```bash
kubectl delete pod nginx-test
```

---

# **7. (Optional) Delete Cluster After Testing**

```bash
eksctl delete cluster \
  --name eksupgrade \
  --region us-east-1
```

---




