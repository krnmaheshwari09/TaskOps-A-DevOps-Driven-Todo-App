# TaskOps - Workflow Documentation

## Overview

This document outlines the complete workflow for developing, building, testing, and deploying the To-Do List web application. It covers both development workflows and CI/CD automation processes.

## Development Workflow

### 1. Local Development Setup

#### Prerequisites
```bash
# Install Node.js 14.x or higher
node --version
npm --version

# Install Docker
docker --version
docker-compose --version

# Install Git
git --version
```

#### Project Setup
```bash
# Clone the repository
git clone https://github.com/krnmaheshwari09/To-do-list-webApp.git
cd To-do-list-webApp

# Setup Frontend
cd Application-Code/frontend
npm install
npm start  # Runs on http://localhost:3000

# Setup Backend (in another terminal)
cd Application-Code/backend
npm install
node index.js  # Runs on http://localhost:3500

# Setup MongoDB (local development)
# Option 1: Local MongoDB installation
mongod --port 27017

# Option 2: Docker MongoDB
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

#### Environment Configuration
```bash
# Frontend (.env)
REACT_APP_API_URL=http://localhost:3500

# Backend (.env)
PORT=3500
MONGODB_URI=mongodb://localhost:27017/todoapp
NODE_ENV=development
```

### 2. Feature Development Workflow

#### Branch Strategy
```bash
# Create feature branch
git checkout -b feature/new-feature-name

# Work on feature
# Make changes to code
# Test locally

# Commit changes
git add .
git commit -m "feat: implement new feature"

# Push feature branch
git push origin feature/new-feature-name

# Create Pull Request on GitHub
# Request code review
# Address review feedback
# Merge to main branch
```

#### Code Quality Standards
```bash
# Frontend linting
cd Application-Code/frontend
npm run test  # Run tests
npx eslint src/  # Lint code

# Backend testing
cd Application-Code/backend
npm test  # Run tests (if available)
```

### 3. Testing Workflow

#### Frontend Testing
```bash
cd Application-Code/frontend

# Unit tests
npm test

# Component testing
npm test -- --coverage

# Build verification
npm run build
```

#### Backend Testing
```bash
cd Application-Code/backend

# API testing (manual with curl)
curl -X GET http://localhost:3500/healthz
curl -X GET http://localhost:3500/api/tasks
curl -X POST http://localhost:3500/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"task": "Test task"}'
```

#### Integration Testing
```bash
# Start all services with Docker Compose
docker-compose up -d

# Test frontend-backend integration
# Access http://localhost:3000
# Create, update, delete tasks
# Verify database persistence

# Clean up
docker-compose down
```

## CI/CD Workflow

### 1. Continuous Integration Pipeline

#### Trigger Events
- **Push to main branch**: Full CI/CD pipeline
- **Pull Request**: CI pipeline only (build and test)
- **Manual trigger**: On-demand deployment

#### Jenkins Pipeline Stages

##### Frontend Pipeline (`Jenkinsfile-Frontend`)
```groovy
pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/krnmaheshwari09/To-do-list-webApp.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                dir('Application-Code/frontend') {
                    sh 'npm install'
                }
            }
        }
        stage('Unit Tests') {
            steps {
                dir('Application-Code/frontend') {
                    sh 'npm test -- --coverage --watchAll=false'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'sonar-scanner'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Application') {
            steps {
                dir('Application-Code/frontend') {
                    sh 'npm run build'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                dir('Application-Code/frontend') {
                    sh 'docker build -t frontend-app .'
                }
            }
        }
        stage('Push to ECR') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                sh 'docker tag frontend-app:latest ${REPOSITORY_URI}frontend-app:latest'
                sh 'docker push ${REPOSITORY_URI}frontend-app:latest'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifests-file/Frontend/'
                sh 'kubectl rollout restart deployment/frontend-deployment'
            }
        }
    }
}
```

##### Backend Pipeline (`Jenkinsfile-Backend`)
```groovy
pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/krnmaheshwari09/To-do-list-webApp.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                dir('Application-Code/backend') {
                    sh 'npm install'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'sonar-scanner'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                dir('Application-Code/backend') {
                    sh 'docker build -t backend-app .'
                }
            }
        }
        stage('Push to ECR') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                sh 'docker tag backend-app:latest ${REPOSITORY_URI}backend-app:latest'
                sh 'docker push ${REPOSITORY_URI}backend-app:latest'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifests-file/Backend/'
                sh 'kubectl rollout restart deployment/backend-deployment'
            }
        }
    }
}
```

### 2. Infrastructure Provisioning Workflow

#### Terraform Workflow
```bash
# Navigate to Terraform directory
cd Jenkins-Server-TF

# Initialize Terraform
terraform init

# Plan infrastructure changes
terraform plan -var-file="variables.tfvars"

# Apply infrastructure
terraform apply -var-file="variables.tfvars" -auto-approve

# Verify infrastructure
aws ec2 describe-instances --region us-east-1
```

#### Infrastructure Components
1. **VPC**: Virtual Private Cloud with public subnet
2. **Security Group**: Firewall rules for required ports
3. **EC2 Instance**: Jenkins server with IAM role
4. **S3 Backend**: Terraform state storage
5. **DynamoDB**: State locking

### 3. Kubernetes Deployment Workflow

#### Database Deployment
```bash
# Deploy MongoDB
kubectl apply -f Kubernetes-Manifests-file/Database/

# Verify database deployment
kubectl get pods -l app=mongodb
kubectl get svc mongodb-service
```

#### Backend Deployment
```bash
# Deploy backend application
kubectl apply -f Kubernetes-Manifests-file/Backend/

# Verify backend deployment
kubectl get deployment backend-deployment
kubectl get pods -l app=backend
kubectl get svc backend-service
```

#### Frontend Deployment
```bash
# Deploy frontend application
kubectl apply -f Kubernetes-Manifests-file/Frontend/

# Verify frontend deployment
kubectl get deployment frontend-deployment
kubectl get pods -l app=frontend
kubectl get svc frontend-service
```

#### Ingress Configuration
```bash
# Deploy ingress controller
kubectl apply -f Kubernetes-Manifests-file/ingress.yaml

# Verify ingress
kubectl get ingress
kubectl describe ingress todo-app-ingress
```

### 4. Monitoring and Verification Workflow

#### Health Check Workflow
```bash
# Check application health
curl -X GET http://your-domain.com/healthz
curl -X GET http://your-domain.com/ready
curl -X GET http://your-domain.com/started

# Check Kubernetes health
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get ingress
```

#### Application Testing Workflow
```bash
# Test API endpoints
curl -X GET http://your-domain.com/api/tasks
curl -X POST http://your-domain.com/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"task": "Production test task"}'

# Test frontend
# Open browser to http://your-domain.com
# Verify UI functionality
```

#### Log Monitoring Workflow
```bash
# View application logs
kubectl logs -f deployment/frontend-deployment
kubectl logs -f deployment/backend-deployment
kubectl logs -f deployment/mongodb-deployment

# View Jenkins logs
docker logs jenkins-container

# View infrastructure logs
aws logs describe-log-groups --region us-east-1
```

## Rollback Workflow

### Application Rollback
```bash
# Rollback Kubernetes deployment
kubectl rollout undo deployment/frontend-deployment
kubectl rollout undo deployment/backend-deployment

# Verify rollback
kubectl rollout status deployment/frontend-deployment
kubectl rollout status deployment/backend-deployment
```

### Infrastructure Rollback
```bash
# Terraform rollback
cd Jenkins-Server-TF
terraform plan -destroy -var-file="variables.tfvars"
terraform destroy -var-file="variables.tfvars" -auto-approve
```

## Troubleshooting Workflow

### Application Issues
```bash
# Check pod status
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>

# Check service connectivity
kubectl exec -it <pod-name> -- /bin/bash
curl http://backend-service:3500/healthz

# Check database connectivity
kubectl exec -it <mongodb-pod> -- mongo
```

### Infrastructure Issues
```bash
# Check AWS resources
aws ec2 describe-instances --region us-east-1
aws ecr describe-repositories --region us-east-1

# Check Jenkins connectivity
ssh -i your-key.pem ec2-user@jenkins-server-ip

# Check Terraform state
terraform show
terraform refresh
```

### Pipeline Issues
```bash
# Check Jenkins job logs
# Access Jenkins UI at http://jenkins-server:8080
# View console output for failed builds

# Check SonarQube analysis
# Access SonarQube UI at http://jenkins-server:9000
# Review code quality reports
```

## Security Workflow

### Secret Management
```bash
# Create Kubernetes secrets
kubectl create secret generic mongodb-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpassword

# Update application configuration
kubectl apply -f Kubernetes-Manifests-file/
```

### Security Scanning
```bash
# Container security scanning
docker scan frontend-app:latest
docker scan backend-app:latest

# Dependency vulnerability scanning
npm audit --audit-level moderate
```

## Performance Optimization Workflow

### Resource Optimization
```bash
# Monitor resource usage
kubectl top pods
kubectl top nodes

# Scale deployments
kubectl scale deployment frontend-deployment --replicas=3
kubectl scale deployment backend-deployment --replicas=2

# Configure HPA
kubectl autoscale deployment frontend-deployment --cpu-percent=50 --min=1 --max=10
```

### Database Optimization
```bash
# Monitor database performance
kubectl exec -it <mongodb-pod> -- mongo --eval "db.stats()"

# Optimize queries
# Add database indexes
# Monitor slow queries
```

This workflow documentation provides a comprehensive guide for all aspects of developing, deploying, and maintaining the To-Do List web application. Follow these workflows to ensure consistent and reliable application delivery.
