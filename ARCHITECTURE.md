# TaskOps - Architecture

## Overview

This document describes the architecture of a full-stack To-Do List web application designed for production deployment on AWS with modern DevOps practices.

## System Architecture

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │    Database     │
│   (React.js)    │◄──►│  (Node.js/      │◄──►│   (MongoDB)     │
│                 │    │   Express)      │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Load Balancer │
                    │   (Kubernetes   │
                    │    Ingress)     │
                    └─────────────────┘
```

### Component Overview

#### 1. Frontend Layer
- **Technology**: React.js 17.0.2
- **UI Framework**: Material-UI 4.11.4
- **Purpose**: User interface for managing to-do tasks
- **Key Features**:
  - Add new tasks
  - Mark tasks as complete/incomplete
  - Delete tasks
  - Real-time task status updates

#### 2. Backend Layer
- **Technology**: Node.js with Express.js 4.17.1
- **Database Driver**: Mongoose 6.13.6
- **API Type**: RESTful API
- **Key Features**:
  - CRUD operations for tasks
  - Health check endpoints
  - CORS support for cross-origin requests

#### 3. Database Layer
- **Technology**: MongoDB
- **Purpose**: Persistent storage for to-do tasks
- **Schema**: Task documents with fields for content, completion status, and metadata

## Infrastructure Architecture

### AWS Cloud Infrastructure

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Cloud                            │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐  │
│  │      VPC        │  │   Internet      │  │   Route     │  │
│  │  10.0.0.0/16    │  │   Gateway       │  │   Table     │  │
│  └─────────────────┘  └─────────────────┘  └─────────────┘  │
│           │                     │                   │       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐  │
│  │  Public Subnet  │  │  Security Group │  │   ECR       │  │
│  │ 10.0.1.0/24     │  │   (Ports:       │  │ (Container  │  │
│  │                 │  │ 22,80,8080,9000)│  │  Registry)  │  │
│  └─────────────────┘  └─────────────────┘  └─────────────┘  │
│           │                                                 │
│  ┌─────────────────┐                                        │
│  │   EC2 Instance  │                                        │
│  │ (Jenkins Server)│                                        │
│  └─────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

### Kubernetes Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                       │
│                                                             │
│  ┌─────────────────┐                                        │
│  │     Ingress     │ ◄─── External Traffic                  │
│  │   Controller    │                                        │
│  └─────────────────┘                                        │
│           │                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐  │
│  │   Frontend      │  │    Backend      │  │   Database  │  │
│  │   Service       │  │    Service      │  │   Service   │  │
│  └─────────────────┘  └─────────────────┘  └─────────────┘  │
│           │                     │                   │       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐  │
│  │   Frontend      │  │    Backend      │  │   MongoDB   │  │
│  │   Deployment    │  │   Deployment    │  │ StatefulSet │  │
│  │   (Pods)        │  │    (Pods)       │  │   (Pods)    │  │
│  └─────────────────┘  └─────────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## DevOps Architecture

### CI/CD Pipeline Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   GitHub    │───►│   Jenkins   │───►│     ECR     │───►│ Kubernetes  │
│ Repository  │    │   Server    │    │ (Container  │    │   Cluster   │
│             │    │             │    │  Registry)  │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                          │
                   ┌─────────────┐
                   │ SonarQube   │
                   │ (Code       │
                   │  Quality)   │
                   └─────────────┘
```

### Security Architecture

1. **Network Security**:
   - VPC with private/public subnets
   - Security groups with minimal required ports
   - Internet Gateway for controlled external access

2. **Application Security**:
   - CORS configuration for frontend-backend communication
   - Health check endpoints for monitoring
   - Container isolation

3. **DevOps Security**:
   - IAM roles and policies for Jenkins
   - ECR for secure container image storage
   - Encrypted Terraform state in S3

## Data Flow

### User Interaction Flow

1. **User Access**: User accesses the React frontend through browser
2. **API Calls**: Frontend makes HTTP requests to Express backend
3. **Database Operations**: Backend performs CRUD operations on MongoDB
4. **Response**: Data flows back through the same path to update UI

### Deployment Flow

1. **Code Commit**: Developer pushes code to GitHub
2. **Jenkins Trigger**: Webhook triggers Jenkins pipeline
3. **Build Process**: Jenkins builds and tests the application
4. **Quality Gates**: SonarQube performs code quality analysis
5. **Containerization**: Docker images are built and pushed to ECR
6. **Deployment**: Kubernetes pulls images and deploys to cluster

## Scalability Considerations

### Horizontal Scaling
- Frontend and backend deployments can be scaled independently
- Kubernetes HPA (Horizontal Pod Autoscaler) for automatic scaling
- Database can be scaled through MongoDB replica sets

### Performance Optimization
- Container resource limits and requests
- Health checks for proper load balancing
- Ingress controller for efficient traffic routing

## Monitoring and Observability

### Health Checks
- `/healthz`: Basic server health
- `/ready`: Database connection status
- `/started`: Application startup verification

### Infrastructure Monitoring
- AWS CloudWatch for infrastructure metrics
- Kubernetes native monitoring
- Application logs through container logging
