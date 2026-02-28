# Task 1: Kubernetes Cluster Setup & Microservices Deployment

## Objective

The goal of this task is to provision a Kubernetes cluster and deploy a three-tier microservices application demonstrating core orchestration principles:

* *Container scheduling*
* **Service discovery**
* **Configuration management**
* **Secret management**
* **Health monitoring**
* **Resource control**
* **Horizontal autoscaling**

This implementation uses Amazon EKS (Elastic Kubernetes Service) as the managed control plane.

# 🏗 Architecture Overview
The application consists of three microservices:
1. Frontend Service
     * **NGINX (acting as web frontend)**
     * **Exposed via AWS Elastic Load Balancer**
2. Backend Service
     * **API service (Echoserver used for demo)**
     * **Communicates with PostgreSQL**
     * **Configured with ConfigMap**
     * **Autoscaled using HPA**
3. Database Service
     * **PostgreSQL**
     * **Credentials stored securely using Kubernetes Secret**
     * **Exposed internally using ClusterIP**


# Phase 1 — Infrastructure Setup (Amazon EKS)

## Step 1: Launch EC2 Instance
EC2 Instance Configuration
     * **Instance Type: t3.medium**
     * **OS: Ubuntu**
     * **Storage: 25GB EBS Volume**

## Step 2: Install AWS CLI

```bash
sudo apt update
sudo apt install awscli -y
aws --version
```
AWS CLI enables:
* **Authentication with AWS**
* **IAM-based cluster creation**
* **EKS API interaction**

## Step 3: Configure AWS Credentials

```bash
aws configure
aws sts get-caller-identity
```
This ensures:
* **IAM authentication is working**
* **The correct AWS account is being used**
* **No permission errors during cluster creation**

## Step 4: Install eksctl
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
eksctl is the official CLI tool for:
* **Creating EKS clusters**
* **Managing node groups**
* **Configuring IAM roles automatically**
* **It simplifies infrastructure provisioning significantly.**

## Step 5: Create EKS Cluster
```bash
eksctl create cluster \
  --name microservices-cluster \
  --region ap-south-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed
```
What This Does
* **Creates EKS control plane (managed by AWS)**
* **Creates Managed Node Group**
* **Enables auto scaling between 2–3 nodes**

Why Managed Node Group?
* **Automatic patching**
* **Better lifecycle management**
* **Production-grade reliability**

## Step 6: Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```
Why?
kubectl is the Kubernetes API client.
It allows:
* **Resource deployment**
* **Cluster inspection**
* **Debugging**
* **Scaling operations**

Verify:
```bash
kubectl get nodes
```

# PHASE 2 — Namespace Isolation
```bash
kubectl create namespace microservices
```
Namespaces provide:
* **Logical isolation**
* **Resource organization**
* **Multi-team separation**
* **RBAC control boundaries**

Production clusters should never deploy everything in default.

# PHASE 3 — Database Layer (PostgreSQL)

## Step 1: Create Secret
Purpose:
* **Store database credentials securely**
* **Avoid hardcoding sensitive values**
```bash
kubectl apply -f postgres-secret.yaml
```

## Step 2: PostgreSQL Deployment

Resource Requests & Limits
```bash
requests:
  cpu: "200m"
  memory: "256Mi"
limits:
  cpu: "500m"
  memory: "512Mi"
```
Why?
* **Prevent resource starvation**
* **Enable scheduler to place pods properly**
* **Required for HPA calculations**

```bash
kubectl apply -f postgres-deployment.yaml
```

## Step 3: PostgreSQL Service
Type: ClusterIP
Reason:
* **Database should NOT be exposed externally**
* **Only backend communicates with it**

```bash
kubectl apply -f postgres-service.yaml
```

# PHASE 4 — Backend Layer
## Step 1: ConfigMap
Purpose:
* **Store non-sensitive configuration**
* **Inject DB_HOST and DB_PORT**

Why ConfigMap?
* **Decouples configuration from container image**
* **Enables dynamic configuration changes**
```bash
kubectl apply -f backend-config.yaml
```

## Step 2: Backend Deployment
Key Features:
* **2 replicas (High Availability)**
* **Resource control**
* **HTTP Liveness & Readiness probes**
* **ConfigMap injection**

Why 2 replicas?
* **Zero downtime**
* **Load distribution**
* **Fault tolerance**
```bash
kubectl apply -f backend-deployment.yaml
```

## Step 3: Backend Service
Type: ClusterIP
Why?
* **Backend is internal service**
* **Frontend communicates internally**
* **No public exposure required**
```bash
kubectl apply -f backend-service.yaml
```

# PHASE 5 — Frontend Layer
## Step 1: Frontend Deployment
Features:
* **2 replicas**
* **Health checks**
* **Resource management**
```bash
kubectl apply -f frontend-deployment.yaml
```

## Step 2: Frontend Service (LoadBalancer)
Type: LoadBalancer
Why?
* **Exposes service publicly**
* **AWS automatically provisions ELB**
* **Assigns public DNS**
```bash
kubectl apply -f frontend-service.yaml
```
Check:
```bash
kubectl get svc -n microservices
```
Access via ELB DNS.


# PHASE 6 — Horizontal Pod Autoscaler (HPA)
Horizontal Pod Autoscaler (HPA) automatically increases or decreases the number of pod replicas based on observed resource utilization (CPU in this case).
It scales horizontally — meaning it adjusts the number of pods, not the size of the node.

# Why HPA is Critical in Production
* **Handles unpredictable traffic**
* **Prevents outages**
* **Optimizes infrastructure cost**
* **Maintains service reliability**
* **Supports microservices architecture**

## Step 1: Install Metrics Server
EKS does NOT install it by default.

Why?
* **HPA requires CPU metrics from metrics-server.**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Verify:
```bash
kubectl get deployment metrics-server -n kube-system
```

## Step 2: Create HPA for Backend

```bash
kubectl autoscale deployment backend \
  --cpu-percent=50 \
  --min=2 \
  --max=5 \
  -n microservices
```
Why 50%?
* **Prevent overload**
* **Maintain performance buffer**
* **Scale before saturation**

Verify
```bash
kubectl get hpa -n microservices
```

# PHASE 7 — Load Testing

Generate load:
```bash
kubectl run -i --tty loadgen --rm --image=busybox -n microservices -- /bin/sh
```

Inside:
```bash
while true; do wget -q -O- http://backend; done
```
Monitor scaling:
```bash
kubectl get pods -n microservices -w
```
Expected Behavior:
* **CPU increases**
* **HPA scales backend pods**
* **New replicas created automatically**
* **Scale down after load stops**


# Conclusion
This project demonstrates:
* **Cloud-native architecture**
* **Kubernetes orchestration fundamentals**
* **Secure configuration management**
* **Scalable backend services**
* **High availability design**
* **Production-ready deployment patterns**

It reflects real-world DevOps engineering practices used in enterprise Kubernetes environments.
