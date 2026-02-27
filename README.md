# Task 1: Kubernetes Cluster Setup & Microservices Deployment

## Objective

The goal of this task is to provision a Kubernetes cluster and deploy a three-tier microservices application demonstrating core orchestration principles:

* **Container scheduling**
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


# 🚀 Phase 1 — Infrastructure Setup (Amazon EKS)

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
Purpose:
     * **Store database credentials securely**
     * **Avoid hardcoding sensitive values**
```bash
kubectl apply -f postgres-secret.yaml
```






