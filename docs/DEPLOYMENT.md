# Deployment Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Infrastructure Setup](#infrastructure-setup)
3. [Jenkins Server Setup](#jenkins-server-setup)
4. [EKS Cluster Deployment](#eks-cluster-deployment)
5. [Application Deployment](#application-deployment)
6. [CI/CD Pipeline Configuration](#cicd-pipeline-configuration)
7. [Monitoring Setup](#monitoring-setup)
8. [Security Configuration](#security-configuration)
9. [Troubleshooting](#troubleshooting)
10. [Post-Deployment Verification](#post-deployment-verification)

## Prerequisites

### Required Tools
Ensure the following tools are installed on your local machine:

```bash
# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# kubectl
curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Terraform
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Docker
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker $USER
```

### AWS Account Requirements
- AWS Account with appropriate permissions
- IAM user with programmatic access
- ECR repositories for frontend and backend images
- Route 53 hosted zone (optional, for custom domain)

### Required AWS Services
- **EC2**: For Jenkins server and EKS worker nodes
- **EKS**: Kubernetes cluster management
- **ECR**: Container image registry
- **VPC**: Network isolation
- **IAM**: Access management
- **CloudWatch**: Logging and monitoring

## Infrastructure Setup

### 1. Configure AWS Credentials

```bash
# Configure AWS CLI
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Enter your default region (e.g., us-east-1)
# Enter your default output format (json)

# Verify configuration
aws sts get-caller-identity
```

### 2. Clone Repository

```bash
git clone https://github.com/krnmaheshwari09/To-do-list-webApp.git
cd To-do-list-webApp
```

### 3. Create S3 Backend for Terraform State

```bash
# Create S3 bucket for Terraform state
aws s3 mb s3://my-ews-baket1 --region us-east-1

# Create DynamoDB table for state locking
aws dynamodb create-table \
    --table-name Lock-Files \
    --attribute-definitions \
        AttributeName=LockID,AttributeType=S \
    --key-schema \
        AttributeName=LockID,KeyType=HASH \
    --provisioned-throughput \
        ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --region us-east-1
```

### 4. Update Terraform Variables

Edit `Jenkins-Server-TF/variables.tfvars`:

```hcl
vpc-name      = "Jenkins-vpc"
igw-name      = "Jenkins-igw"
subnet-name   = "Jenkins-subnet"
rt-name       = "Jenkins-route-table"
sg-name       = "Jenkins-sg"
instance-name = "Jenkins-server"
key-name      = "YOUR_KEY_PAIR_NAME"  # Update this
iam-role      = "Jenkins-iam-role"
```

## Jenkins Server Setup

### 1. Deploy Jenkins Infrastructure

```bash
cd Jenkins-Server-TF

# Initialize Terraform
terraform init

# Plan the deployment
terraform plan -var-file="variables.tfvars"

# Apply the infrastructure
terraform apply -var-file="variables.tfvars"

# Note the public IP of the Jenkins server
terraform output jenkins_public_ip
```

### 2. Install Tools on Jenkins Server

```bash
# SSH into Jenkins server
ssh -i ~/.ssh/YOUR_KEY_PAIR.pem ubuntu@JENKINS_PUBLIC_IP

# Make the script executable and run it
chmod +x tools-install.sh
./tools-install.sh

# Wait for installation to complete (approximately 10-15 minutes)
```

### 3. Configure Jenkins

#### Access Jenkins Web Interface
```bash
# Get Jenkins initial password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Access Jenkins at: http://JENKINS_PUBLIC_IP:8080
# Use the initial password to unlock Jenkins
```

#### Install Required Plugins
1. **Suggested plugins** (during initial setup)
2. **Additional plugins**:
   - AWS Steps
   - Docker Pipeline
   - SonarQube Scanner
   - OWASP Dependency-Check
   - Kubernetes CLI

#### Configure Global Tools
Navigate to `Manage Jenkins > Global Tool Configuration`:

**Node.js Installation**:
- Name: `nodejs`
- Version: `NodeJS 14.x`
- Install automatically: ✓

**SonarQube Scanner**:
- Name: `sonar-scanner`
- Install automatically: ✓
- Version: Latest

**OWASP Dependency-Check**:
- Name: `DP-Check`
- Install automatically: ✓
- Version: Latest

### 4. Configure Jenkins Credentials

Navigate to `Manage Jenkins > Manage Credentials > System > Global credentials`:

**AWS Account ID**:
- Kind: Secret text
- ID: `ACCOUNT_ID`
- Secret: Your AWS Account ID

**ECR Repository Names**:
- Kind: Secret text
- ID: `ECR_REPO1`
- Secret: `frontend`

- Kind: Secret text
- ID: `ECR_REPO2`
- Secret: `backend`

**GitHub Token**:
- Kind: Secret text
- ID: `GITHUB`
- Secret: Your GitHub personal access token

**SonarQube Token**:
- Kind: Secret text
- ID: `sonar-token`
- Secret: Your SonarQube token

### 5. Configure SonarQube

```bash
# SonarQube is running in a container
docker ps | grep sonar

# Access SonarQube at: http://JENKINS_PUBLIC_IP:9000
# Default credentials: admin/admin
```

**SonarQube Configuration**:
1. Change default password
2. Create a new project for backend and frontend
3. Generate authentication tokens
4. Configure quality gates

## EKS Cluster Deployment

### 1. Create ECR Repositories

```bash
# Create ECR repositories
aws ecr create-repository --repository-name backend --region us-east-1
aws ecr create-repository --repository-name frontend --region us-east-1

# Get repository URIs
aws ecr describe-repositories --region us-east-1
```

### 2. Create EKS Cluster

```bash
# Create EKS cluster using eksctl
eksctl create cluster \
    --name three-tier-cluster \
    --region us-east-1 \
    --nodegroup-name worker-nodes \
    --node-type t3.medium \
    --nodes 2 \
    --nodes-min 1 \
    --nodes-max 4 \
    --managed

# This process takes approximately 15-20 minutes
```

### 3. Update kubeconfig

```bash
# Update kubeconfig for the new cluster
aws eks update-kubeconfig --region us-east-1 --name three-tier-cluster

# Verify cluster access
kubectl get nodes
kubectl get namespaces
```

### 4. Create Namespace

```bash
# Create the three-tier namespace
kubectl create namespace three-tier

# Verify namespace creation
kubectl get namespaces
```

## Application Deployment

### 1. Build and Push Initial Images

#### Backend Image
```bash
cd Application-Code/backend

# Build Docker image
docker build -t backend .

# Tag for ECR
docker tag backend:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/backend:1

# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com

# Push image
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/backend:1
```

#### Frontend Image
```bash
cd Application-Code/frontend

# Build Docker image
docker build -t frontend .

# Tag for ECR
docker tag frontend:latest ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/frontend:1

# Push image
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/frontend:1
```

### 2. Update Kubernetes Manifests

Update image URLs in the Kubernetes manifest files:

**Backend Deployment** (`Kubernetes-Manifests-file/Backend/deployment.yaml`):
```yaml
spec:
  containers:
  - name: api
    image: ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/backend:1
```

**Frontend Deployment** (`Kubernetes-Manifests-file/Frontend/deployment.yaml`):
```yaml
spec:
  containers:
  - name: frontend
    image: ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/frontend:1
```

### 3. Create ECR Secret

```bash
# Create ECR registry secret
kubectl create secret docker-registry ecr-registry-secret \
    --docker-server=${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com \
    --docker-username=AWS \
    --docker-password=$(aws ecr get-login-password --region us-east-1) \
    --namespace=three-tier
```

### 4. Deploy Database

```bash
# Deploy MongoDB
kubectl apply -f Kubernetes-Manifests-file/Database/

# Verify database deployment
kubectl get pods -n three-tier
kubectl get services -n three-tier
```

### 5. Deploy Backend

```bash
# Deploy backend API
kubectl apply -f Kubernetes-Manifests-file/Backend/

# Verify backend deployment
kubectl get pods -n three-tier
kubectl logs deployment/api -n three-tier
```

### 6. Deploy Frontend

```bash
# Deploy frontend
kubectl apply -f Kubernetes-Manifests-file/Frontend/

# Verify frontend deployment
kubectl get pods -n three-tier
kubectl logs deployment/frontend -n three-tier
```

### 7. Configure Ingress

```bash
# Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/aws/deploy.yaml

# Wait for ingress controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# Apply ingress configuration
kubectl apply -f Kubernetes-Manifests-file/ingress.yaml
```

### 8. Get Application URL

```bash
# Get the load balancer URL
kubectl get ingress -n three-tier

# Or get the service URL directly
kubectl get service frontend-svc -n three-tier -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

## CI/CD Pipeline Configuration

### 1. Create Jenkins Pipeline Jobs

#### Backend Pipeline
1. **New Item** > **Pipeline**
2. **Name**: `todo-backend-pipeline`
3. **Pipeline script from SCM**:
   - SCM: Git
   - Repository URL: `https://github.com/krnmaheshwari09/To-do-list-webApp.git`
   - Script Path: `Jenkins-Pipeline-Code/Jenkinsfile-Backend`

#### Frontend Pipeline
1. **New Item** > **Pipeline**
2. **Name**: `todo-frontend-pipeline`
3. **Pipeline script from SCM**:
   - SCM: Git
   - Repository URL: `https://github.com/krnmaheshwari09/To-do-list-webApp.git`
   - Script Path: `Jenkins-Pipeline-Code/Jenkinsfile-Frontend`

### 2. Configure GitHub Webhooks

1. Go to your GitHub repository settings
2. **Webhooks** > **Add webhook**
3. **Payload URL**: `http://JENKINS_PUBLIC_IP:8080/github-webhook/`
4. **Content type**: `application/json`
5. **Events**: Push events
6. **Active**: ✓

### 3. Test Pipeline Execution

```bash
# Trigger pipeline manually or push code changes
git add .
git commit -m "test: trigger pipeline"
git push origin main
```

## Monitoring Setup

### 1. Configure CloudWatch Logging

```bash
# Install Fluent Bit for log collection
kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/logs-insight-on-eks/fluent-bit-daemonset.yaml

# Create CloudWatch log groups
aws logs create-log-group --log-group-name /aws/eks/three-tier-cluster/application --region us-east-1
```

### 2. Set Up CloudWatch Dashboards

Create CloudWatch dashboard for monitoring:
- CPU and Memory utilization
- Pod count and status
- Application response times
- Error rates

### 3. Configure Alerts

```bash
# Example: High CPU alert
aws cloudwatch put-metric-alarm \
    --alarm-name "EKS-High-CPU" \
    --alarm-description "Alert when CPU exceeds 80%" \
    --metric-name CPUUtilization \
    --namespace AWS/EKS \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2
```

## Security Configuration

### 1. Network Policies

```bash
# Apply network policies for pod-to-pod communication
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: three-tier-network-policy
  namespace: three-tier
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: three-tier
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: three-tier
EOF
```

### 2. RBAC Configuration

```bash
# Create service account with limited permissions
kubectl create serviceaccount three-tier-sa -n three-tier

# Create role with minimal required permissions
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: three-tier
  name: three-tier-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
EOF

# Bind role to service account
kubectl create rolebinding three-tier-binding \
    --role=three-tier-role \
    --serviceaccount=three-tier:three-tier-sa \
    -n three-tier
```

### 3. Pod Security Standards

```bash
# Apply pod security standards
kubectl label namespace three-tier pod-security.kubernetes.io/enforce=restricted
kubectl label namespace three-tier pod-security.kubernetes.io/audit=restricted
kubectl label namespace three-tier pod-security.kubernetes.io/warn=restricted
```

## Troubleshooting

### Common Issues

#### 1. ECR Authentication Issues
```bash
# Refresh ECR token
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com

# Update Kubernetes secret
kubectl delete secret ecr-registry-secret -n three-tier
kubectl create secret docker-registry ecr-registry-secret \
    --docker-server=${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com \
    --docker-username=AWS \
    --docker-password=$(aws ecr get-login-password --region us-east-1) \
    --namespace=three-tier
```

#### 2. Pod Startup Issues
```bash
# Check pod status
kubectl get pods -n three-tier

# View pod logs
kubectl logs <pod-name> -n three-tier

# Describe pod for events
kubectl describe pod <pod-name> -n three-tier

# Check resource constraints
kubectl top pods -n three-tier
```

#### 3. Service Connectivity Issues
```bash
# Test internal connectivity
kubectl exec -it deployment/frontend -n three-tier -- curl http://api-svc:3500/healthz

# Check service endpoints
kubectl get endpoints -n three-tier

# Test DNS resolution
kubectl exec -it deployment/frontend -n three-tier -- nslookup api-svc.three-tier.svc.cluster.local
```

#### 4. Jenkins Pipeline Failures
```bash
# Check Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log

# Check Docker daemon
sudo systemctl status docker

# Restart Jenkins if needed
sudo systemctl restart jenkins
```

### Debug Commands

```bash
# Cluster information
kubectl cluster-info
kubectl get nodes -o wide

# Application status
kubectl get all -n three-tier

# Resource usage
kubectl top nodes
kubectl top pods -n three-tier

# Events
kubectl get events -n three-tier --sort-by=.metadata.creationTimestamp

# Port forwarding for local testing
kubectl port-forward service/frontend-svc 3000:3000 -n three-tier
kubectl port-forward service/api-svc 3500:3500 -n three-tier
```

## Post-Deployment Verification

### 1. Application Health Checks

```bash
# Check all components are running
kubectl get pods -n three-tier

# Verify services are accessible
kubectl get services -n three-tier

# Test health endpoints
kubectl exec -it deployment/api -n three-tier -- curl localhost:3500/healthz
kubectl exec -it deployment/api -n three-tier -- curl localhost:3500/ready
```

### 2. End-to-End Testing

```bash
# Get application URL
export APP_URL=$(kubectl get ingress -n three-tier -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}')

# Test frontend accessibility
curl -I http://$APP_URL

# Test API endpoints
curl http://$APP_URL/api/tasks
```

### 3. Pipeline Testing

```bash
# Test CI/CD pipeline
git add .
git commit -m "test: verify pipeline functionality"
git push origin main

# Monitor pipeline execution in Jenkins
# Verify automatic deployment to Kubernetes
```

### 4. Security Verification

```bash
# Run security scan
trivy image ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/backend:latest
trivy image ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/frontend:latest

# Check network policies
kubectl get networkpolicies -n three-tier

# Verify RBAC
kubectl auth can-i create pods --as=system:serviceaccount:three-tier:three-tier-sa -n three-tier
```

### 5. Performance Testing

```bash
# Install and run basic load test
kubectl run -i --tty load-test --rm --image=busybox --restart=Never -- sh

# Inside the pod, test API performance
while true; do wget -q -O- http://api-svc.three-tier.svc.cluster.local:3500/api/tasks; done
```

## Cleanup

### Remove Application

```bash
# Delete Kubernetes resources
kubectl delete namespace three-tier

# Delete ingress controller (if needed)
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/aws/deploy.yaml
```

### Remove EKS Cluster

```bash
# Delete EKS cluster
eksctl delete cluster --region=us-east-1 --name=three-tier-cluster
```

### Remove Infrastructure

```bash
# Destroy Terraform infrastructure
cd Jenkins-Server-TF
terraform destroy -var-file="variables.tfvars"
```

---

This deployment guide provides step-by-step instructions for setting up the complete To-Do List Web Application infrastructure and CI/CD pipeline. Follow the sections in order for a successful deployment.