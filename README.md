# Task 1: Kubernetes Cluster Setup & Microservices Deployment

# Objective
Set up a production-grade Kubernetes cluster on Amazon EKS and deploy a microservices-based application demonstrating core orchestration concepts including:

=> Deployments
=> Services
=> ConfigMaps
=> Secrets
=> LoadBalancer
=> Resource Requests & Limits
=> Health Checks (Liveness & Readiness Probes)
=> Horizontal Pod Autoscaler (HPA)

# 🏗 Architecture Overview
The application consists of three microservices:
1. Frontend Service
     => NGINX (acting as web frontend)
     => Exposed via AWS Elastic Load Balancer
2. Backend Service
     => API service (Echoserver used for demo)
     => Communicates with PostgreSQL
     => Configured with ConfigMap
     => Autoscaled using HPA
3. Database Service
     => PostgreSQL
     => Credentials stored securely using Kubernetes Secret
     => Exposed internally using ClusterIP


# 🚀 Phase 1 — Infrastructure Setup (Amazon EKS)
EC2 Instance Configuration
     => Instance Type: t3.medium
     => OS: Ubuntu
     => Storage: 25GB EBS Volume

## Install AWS CLI

```bash
sudo apt update
sudo apt install awscli -y
aws --version
```
