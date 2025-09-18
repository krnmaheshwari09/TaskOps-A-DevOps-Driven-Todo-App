# TaskOps-A-DevOps-Driven-Todo-App

A modern, full-stack to-do list web application with complete DevOps automation, built using React.js, Node.js, MongoDB, and deployed on AWS with Kubernetes.

![Application Demo](https://img.shields.io/badge/Status-Production%20Ready-green)
![React](https://img.shields.io/badge/React-17.0.2-blue)
![Node.js](https://img.shields.io/badge/Node.js-14.x-green)
![MongoDB](https://img.shields.io/badge/MongoDB-Latest-green)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-blue)

## 🚀 Features

- **📝 Task Management**: Create, update, delete, and mark tasks as complete
- **💾 Persistent Storage**: MongoDB database for reliable data persistence
- **🎨 Modern UI**: Clean, responsive design with Material-UI components
- **🔄 Real-time Updates**: Instant UI updates for task operations
- **🏗️ Microservices Architecture**: Separate frontend and backend services
- **🐳 Containerized**: Docker containers for consistent deployment
- **☸️ Kubernetes Ready**: Complete Kubernetes manifests for production deployment
- **🚀 CI/CD Pipeline**: Automated Jenkins pipelines for build and deployment
- **🔒 Security**: VPC, security groups, and IAM roles for secure AWS deployment
- **📊 Monitoring**: Health checks and monitoring endpoints
- **⚡ Performance**: Optimized builds and scalable infrastructure

## 🏗️ Architecture

### System Overview
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │    Database     │
│   (React.js)    │◄──►│  (Node.js/      │◄──►│   (MongoDB)     │
│   Port: 3000    │    │   Express)      │    │   Port: 27017   │
│                 │    │   Port: 3500    │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Technology Stack
- **Frontend**: React.js 17.0.2, Material-UI 4.11.4, Axios
- **Backend**: Node.js, Express.js 4.17.1, Mongoose 6.13.6
- **Database**: MongoDB
- **DevOps**: Docker, Kubernetes, Jenkins, Terraform
- **Cloud**: AWS (VPC, EC2, ECR, S3, DynamoDB)
- **Monitoring**: SonarQube, Health Check Endpoints

## 📋 Prerequisites

- Node.js 14.x or higher
- npm or yarn
- Docker and Docker Compose
- MongoDB (local or cloud)
- AWS Account (for cloud deployment)
- Kubernetes cluster (for production deployment)
- Jenkins server (for CI/CD)

## 🛠️ Quick Start

### Local Development

#### 1. Clone the Repository
```bash
git clone https://github.com/krnmaheshwari09/TaskOps-A-DevOps-Driven-Todo-App.git
cd TaskOps-A-DevOps-Driven-Todo-App
```

#### 2. Setup Backend
```bash
cd Application-Code/backend
npm install
node index.js
# Backend runs on http://localhost:3500
```

#### 3. Setup Frontend
```bash
cd Application-Code/frontend
npm install
npm start
# Frontend runs on http://localhost:3000
```

#### 4. Setup Database
```bash
# Option 1: Local MongoDB
mongod --port 27017

# Option 2: Docker MongoDB
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

### Docker Development

#### 1. Build and Run with Docker Compose
```bash
docker-compose up -d
```

#### 2. Access Application
- Frontend: http://localhost:3000
- Backend API: http://localhost:3500/api/tasks
- Health Check: http://localhost:3500/healthz

## 🚀 Production Deployment

### AWS Infrastructure Setup

#### 1. Deploy Jenkins Server
```bash
cd Jenkins-Server-TF
terraform init
terraform plan -var-file="variables.tfvars"
terraform apply -var-file="variables.tfvars"
```

#### 2. Configure Jenkins
- Access Jenkins at `http://<jenkins-server-ip>:8080`
- Install required plugins (Node.js, Docker, AWS CLI)
- Configure Jenkins pipelines using provided Jenkinsfiles

#### 3. Deploy to Kubernetes
```bash
# Deploy database
kubectl apply -f Kubernetes-Manifests-file/Database/

# Deploy backend
kubectl apply -f Kubernetes-Manifests-file/Backend/

# Deploy frontend
kubectl apply -f Kubernetes-Manifests-file/Frontend/

# Configure ingress
kubectl apply -f Kubernetes-Manifests-file/ingress.yaml
```

## 📚 API Documentation

### Base URL
```
Development: http://localhost:3500
Production: https://your-domain.com
```

### Endpoints

#### Health Checks
- `GET /healthz` - Basic health check
- `GET /ready` - Readiness check with database connection
- `GET /started` - Startup verification

#### Task Management
- `GET /api/tasks` - Get all tasks
- `POST /api/tasks` - Create new task
- `PUT /api/tasks/:id` - Update task completion status
- `DELETE /api/tasks/:id` - Delete task

### Example API Usage
```bash
# Get all tasks
curl -X GET http://localhost:3500/api/tasks

# Create a new task
curl -X POST http://localhost:3500/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"task": "Complete project documentation"}'

# Update task completion
curl -X PUT http://localhost:3500/api/tasks/TASK_ID \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'

# Delete a task
curl -X DELETE http://localhost:3500/api/tasks/TASK_ID
```

## 🧪 Testing

### Frontend Testing
```bash
cd Application-Code/frontend
npm test
npm test -- --coverage
```

### Backend Testing
```bash
cd Application-Code/backend
# Manual API testing with curl commands
# Integration testing with frontend
```

### E2E Testing
```bash
# Start all services
docker-compose up -d

# Test complete workflow
# 1. Access frontend at http://localhost:3000
# 2. Create, update, and delete tasks
# 3. Verify data persistence
```

## 📊 Monitoring

### Health Checks
The application includes comprehensive health check endpoints:
- **Liveness**: `/healthz` - Confirms the application is running
- **Readiness**: `/ready` - Confirms database connectivity
- **Startup**: `/started` - Confirms successful startup

### Kubernetes Probes
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 3500
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 3500
  initialDelaySeconds: 5
  periodSeconds: 5
```

## 🔒 Security

### Application Security
- CORS configuration for secure cross-origin requests
- Input validation and sanitization
- Error handling without sensitive information exposure

### Infrastructure Security
- VPC with private and public subnets
- Security groups with minimal required ports (22, 80, 8080, 9000, 9090)
- IAM roles and policies for AWS services
- Container security best practices

## 📈 Performance

### Optimization Features
- Optimized React build for production
- Efficient MongoDB queries with Mongoose
- Container resource limits and requests
- Kubernetes horizontal pod autoscaling
- Load balancing with ingress controller

### Scalability
- Stateless frontend and backend design
- Independent service scaling
- Database replica sets support
- Auto-scaling based on CPU/memory metrics

## 🛠️ Development Workflow

### CI/CD Pipeline
1. **Source Control**: GitHub repository with webhook triggers
2. **Build**: Jenkins pipelines for frontend and backend
3. **Quality Gates**: SonarQube code analysis
4. **Testing**: Automated test execution
5. **Package**: Docker image creation and ECR storage
6. **Deploy**: Kubernetes deployment with rolling updates
7. **Monitor**: Health checks and performance monitoring

### Branch Strategy
- `main`: Production-ready code
- `feature/*`: Feature development branches
- `hotfix/*`: Critical bug fixes

## 📁 Project Structure

```
TaskOps-A-DevOps-Driven-Todo-App/
├── Application-Code/
│   ├── frontend/          # React.js frontend application
│   │   ├── src/           # Source code
│   │   ├── public/        # Static assets
│   │   ├── package.json   # Dependencies and scripts
│   │   └── Dockerfile     # Frontend container
│   └── backend/           # Node.js backend application
│       ├── routes/        # API routes
│       ├── models/        # Database models
│       ├── package.json   # Dependencies and scripts
│       └── Dockerfile     # Backend container
├── Jenkins-Server-TF/     # Terraform infrastructure code
│   ├── vpc.tf            # VPC and networking
│   ├── backend.tf        # Terraform backend configuration
│   └── variables.tf      # Variable definitions
├── Jenkins-Pipeline-Code/ # CI/CD pipeline definitions
│   ├── Jenkinsfile-Frontend
│   └── Jenkinsfile-Backend
├── Kubernetes-Manifests-file/ # Kubernetes deployment files
│   ├── Frontend/         # Frontend K8s manifests
│   ├── Backend/          # Backend K8s manifests
│   ├── Database/         # Database K8s manifests
│   └── ingress.yaml      # Ingress configuration
├── ARCHITECTURE.md       # System architecture documentation
├── TECHNICAL_SPECIFICATION.md # Technical specifications
├── WORKFLOW.md          # Development and deployment workflows
└── README.md            # Project documentation
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines
- Follow the existing code style
- Add tests for new features
- Update documentation for significant changes
- Ensure all CI/CD checks pass

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## 🙋‍♂️ Support

For support and questions:
- Create an issue in this repository
- Check the [WORKFLOW.md](WORKFLOW.md) for troubleshooting steps
- Review the [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md) for detailed implementation

## 🏆 Acknowledgments

- React team for the amazing frontend framework
- Express.js community for the robust backend framework
- MongoDB for the flexible database solution
- AWS for reliable cloud infrastructure
- Kubernetes community for orchestration platform
- Jenkins for powerful CI/CD automation

---

**Built with ❤️ for modern web development and DevOps practices**