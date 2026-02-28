# End-to-End CI/CD Pipeline with Jenkins, ECR, EKS & ArgoCD (GitOps)

## Objective

The goal of this task is to design and implement a production-ready, end-to-end CI/CD pipeline that automates the complete software delivery lifecycle—from code validation to Kubernetes deployment—while following GitOps principles.

This implementation demonstrates:
* *Automated linting and static code analysis*
* *Unit testing and quality validation*
* *Container image build and versioning*
* *Security scanning using Trivy*
* *Image management with AWS ECR*
* *Automated deployment to Kubernetes (Amazon EKS)*
* *GitOps-based continuous delivery using ArgoCD*
* *Environment separation (Staging & Production)*
* *Controlled production releases with manual approval*
* *Rollback capability using Git version control*

This solution uses Jenkins as the CI engine, Amazon ECR as the container registry, Amazon EKS as the runtime environment, and ArgoCD to implement a GitOps-driven continuous deployment model.


# 🏗️ Architecture Overview
 
Developer Push
      ↓
Jenkins CI
  - Lint
  - Test
  - Build
  - Trivy Scan
  - Push to ECR
  - Update GitOps Repo
      ↓
   ArgoCD
      ↓
  EKS Cluster
    - Staging (Auto Sync)
    - Production (Manual Sync)


# Phase 1 – Infrastructure Setup

## Step 1: Launch EC2 Instance
EC2 Instance Configuration
     * *Instance Type: t3.medium*
     * *OS: Ubuntu*
     * *Storage: 25GB EBS Volume*

## Step 2: Create EKS Cluster

```bash
eksctl create cluster \
  --name my-cluster \
  --region ap-south-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed
```

Configure kubectl:
```bash
aws eks update-kubeconfig --region ap-south-1 --name my-cluster
kubectl get nodes
```


## Step 3: Create ECR Repository

Create private ECR repository with repo name "microservice-app"

# Configure Jenkins AWS Access
     * *Created IAM user for Jenkins*
     * *Attached:*
         * *AmazonEC2ContainerRegistryFullAccess*
     * *Stored credentials securely in Jenkins*
     * *Verified ECR login*


# Phase 2 – ArgoCD Setup

## Step 1: Install ArgoCD (Version Pinned)
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f \
https://raw.githubusercontent.com/argoproj/argo-cd/v2.11.3/manifests/install.yaml
```

## Step 2: Expose ArgoCD via LoadBalancer
```bash
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

## Step 3: Login
```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```


# Phase 3 – GitOps Repository Structure

Created separate repository:

microservice-gitops/
 ├── staging/
 │     ├── deployment.yaml
 │     └── service.yaml
 └── production/
       ├── deployment.yaml
       └── service.yaml

# Phase 4 – Jenkins CI Pipeline

Pipeline Stages:
1. Checkout
2. Install Dependencies
3. Lint
4. Unit Tests
5. Build Docker Image
6. Trivy Scan
7. Push to ECR
8. Update GitOps Repository

## Lint Setup

Added to package.json:
```bash
"scripts": {
  "start": "node app.js",
  "lint": "eslint .",
  "test": "jest"
}
```

Create .eslintrc.json:
```bash
{
  "env": {
    "node": true,
    "es2021": true,
    "jest": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": 12
  },
  "rules": {}
}
```

## Unit Test Setup

Create:
```bash
mkdir __tests__/
```

```bash
vi __tests__/app.test.js
```

```bash
test('basic sanity test', () => {
  expect(1 + 1).toBe(2);
});
```


# Phase 5 – GitOps Flow

## Staging Environment
     * *ArgoCD Application created*
     * *Auto Sync enabled*
     * *Self Heal enabled*
Flow:
     * *Jenkins updates image tag in staging/deployment.yaml*
     * *ArgoCD auto deploys*

Status:
```bash
Synced
Healthy
```

## Production Environment
     * *Separate namespace*
     * *Separate ArgoCD Application*
     * *Manual Sync (No auto-deploy)*
Flow:
     * *Promotion via PR*
     * *Manual Sync in ArgoCD UI*


# Rollback Strategy

## GitOps Native Rollback
```bash
git revert bad-commit
git push
```
ArgoCD auto redeploys previous version.


## Alternative Rollback Options

     * *ArgoCD UI → History → Rollback
     * *Kubernetes:
```bash
kubectl rollout undo deployment microservice-app -n production
```


# Conclusion
This implementation demonstrates a production-grade DevOps workflow using:
     * *Jenkins (CI)
     * *AWS ECR (Registry)
     * *Amazon EKS (Runtime)
     * *ArgoCD (GitOps CD)
The system ensures:
     * *Only validated code is deployed
     * *Environment separation (Staging / Production)
     * *Secure image management
     * *Fully auditable deployments
     * *Safe rollback capabilities




