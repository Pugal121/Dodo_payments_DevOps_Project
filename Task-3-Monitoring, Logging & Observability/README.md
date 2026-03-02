# Task 3: Monitoring, Logging & Observability

## Objective

The goal of this task is to implement a comprehensive observability stack for a microservices application deployed on Amazon EKS.

This implementation demonstrates:
* *Kubernetes cluster provisioning (EKS)*
* *Application deployment with Ingress (ALB)*
* *Metrics collection using Prometheus*
* *Visualization using Grafana*
* *Application-level monitoring via ServiceMonitor*
* *Foundation for centralized logging and distributed tracing*


# 🏗 Architecture Overview

User
  ↓
AWS ALB (via Ingress Controller)
  ↓
Frontend Service (ClusterIP)
  ↓
Backend Service (ClusterIP)
  ↓
Kubernetes Pods
  ↓
Prometheus (Metrics Collection)
  ↓
Grafana (Visualization)


Namespaces used:
* *production → Application workloads*
* *monitoring → Observability stack*


# Environment Setup
EC2 Instance
    * *Instance Type: t3.medium*
    * *OS: Ubuntu*
    * *Storage: 25 GB*
    * *Region: ap-south-1*

# EKS Cluster Setup

Tools Installed
    * *AWS CLI v2*
    * *kubectl*
    * *eksctl*
    * *Helm*

Create EKS Cluster
```bash
eksctl create cluster \
  --name observability-cluster \
  --region ap-south-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed
```


Cluster validation:
```bash
kubectl get nodes -o wide
```


## Application Deployment

# Namespace Creation
```bash
kubectl create namespace production
```

# Backend Deployment
Backend uses:
```bash
ghcr.io/stefanprodan/podinfo:6.4.0
```

Exposes:
/
/metrics

# Deployment
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: production
spec:
  replicas: 2
```

Service
```bash
type: ClusterIP
port: 9898
```

# Frontend Deployment
Image:Nginx
Exposed internally via ClusterIP.


## Ingress Setup (Production-Grade)

Instead of exposing services via LoadBalancer directly, we implemented:
* *AWS Load Balancer Controller*
* *ALB Ingress*

Ingress Resource
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: alb
```

ALB DNS Example:
```bash
k8s-producti-appingre-xxxx.ap-south-1.elb.amazonaws.com
```


## Monitoring Stack

Monitoring Stack
```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring
```

This deployed:
* *Prometheus*
* *Grafana*
* *Alertmanager*
* *kube-state-metrics*
* *node-exporter*


## Application Metrics Collection

ServiceMonitor Created
```bash
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: backend-monitor
  namespace: monitoring
```

This allows Prometheus to scrape:
```bash
/metrics
```
from backend pods.


## Grafana Dashboard

Custom dashboard created:

# Production Backend Observability

Panels include:
1. Request Rate (RPS)
```bash
sum(rate(http_requests_total[1m])) by (pod)
```
2. Error Rate
```bash
sum(rate(http_requests_total{status="500"}[1m])) by (pod)
```

## Resource Usage Snapshot

Cluster Capacity:
     * *2 x t3.medium nodes*
     * *Total 4 vCPU*
     * *Total 8 GB RAM*

Observed Usage After Monitoring Stack:
     * *CPU: ~2–4%*
     * *Memory: ~25%*
System stable and healthy.


## Alerting Foundation

PrometheusRule created for:
     * *High Error Rate*
     * *High CPU Usage*

## Conclusion
This task successfully implemented a production-style observability stack on Amazon EKS, covering:
* *Infrastructure monitoring*
* *Application metrics*
* *Visualization*
* *Alerting foundation*

