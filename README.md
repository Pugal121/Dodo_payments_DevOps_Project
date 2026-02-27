# Task 1: Kubernetes Cluster Setup & Microservices Deployment

# Objective
The goal of this task is to provision a Kubernetes cluster and deploy a three-tier microservices application demonstrating core orchestration principles:
  => Container scheduling
  => Service discovery
  => Configuration management
  => Secret management
  => Health monitoring
  => Resource control
  => Horizontal autoscaling

This implementation uses Amazon EKS (Elastic Kubernetes Service) as the managed control plane.

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
