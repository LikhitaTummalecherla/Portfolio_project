# DevOps Node.js Application

A comprehensive DevOps demonstration project showcasing modern CI/CD practices with Docker, Jenkins, and Kubernetes.

## 🚀 Features

- **Node.js Express API** with RESTful endpoints
- **Multi-stage Docker builds** for optimized production images
- **Jenkins CI/CD pipeline** with automated testing and deployment
- **Kubernetes manifests** for container orchestration
- **Docker Compose** for local development
- **Security scanning** with Trivy and npm audit
- **Health checks** and monitoring endpoints
- **Nginx reverse proxy** for load balancing

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Developer     │    │     Jenkins     │    │   Kubernetes    │
│   Local Dev     │───▶│   CI/CD Server  │───▶│    Cluster      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Docker Compose  │    │ Docker Registry │    │  Load Balancer  │
│ Local Services  │    │  Image Storage  │    │     Nginx       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 🛠️ Prerequisites

- **Docker** (v20.10+)
- **Docker Compose** (v2.0+)
- **Node.js** (v18+)
- **Jenkins** (v2.400+)
- **Kubernetes** (v1.24+)

## 🚀 Quick Start

### Local Development

1. **Clone and setup:**
   ```bash
   git clone <repository-url>
   cd devops-node-app
   chmod +x scripts/local-setup.sh
   ./scripts/local-setup.sh
   ```

2. **Access the application:**
   - Main app: http://localhost:3000
   - Health check: http://localhost:3000/health
   - API endpoints: http://localhost:3000/api/users

### Development Commands

```bash
# Start development server
npm run dev

# Run tests
npm test
npm run test:coverage

# Lint code
npm run lint
npm run lint:fix

# Docker operations
npm run docker:build
npm run docker:run
npm run docker:compose:up
npm run docker:compose:down
```

## 🐳 Docker

### Multi-stage Build

The Dockerfile uses multi-stage builds for:
- **Development**: Full dev dependencies with hot reload
- **Build**: Production dependencies only
- **Production**: Minimal runtime image with security hardening

### Docker Compose Services

- **app**: Development server with hot reload
- **app-prod**: Production build
- **redis**: Caching layer
- **nginx**: Reverse proxy and load balancer

## 🔄 CI/CD Pipeline

### Jenkins Pipeline Stages

1. **Checkout**: Get source code and commit info
2. **Install Dependencies**: npm clean-install
3. **Parallel Quality Checks**:
   - Code linting (ESLint)
   - Unit tests with coverage
   - Security audit (npm audit)
4. **Build Docker Image**: Multi-stage production build
5. **Security Scan**: Container vulnerability scanning (Trivy)
6. **Integration Tests**: Full application testing
7. **Deploy to Staging**: Automatic deployment for develop branch
8. **Production Approval**: Manual approval gate
9. **Deploy to Production**: Blue-green deployment with rollback

### Environment Promotion

```
develop branch → Staging Environment
     ↓
main branch → Production Approval → Production Environment
```

## ☸️ Kubernetes Deployment

### Manifests

- **Deployment**: 3 replicas with resource limits
- **Service**: LoadBalancer for external access
- **Health Probes**: Liveness and readiness checks

### Deployment Script

```bash
# Deploy to staging
./scripts/deploy.sh staging v1.0.0

# Deploy to production
./scripts/deploy.sh production v1.0.0
```

## 🔒 Security Features

- **Container Security**: Non-root user, minimal base image
- **Dependency Scanning**: npm audit in pipeline
- **Container Scanning**: Trivy vulnerability scanning
- **Security Headers**: Helmet.js middleware
- **Environment Isolation**: Separate configs per environment

## 📊 Monitoring & Health Checks

### Endpoints

- `GET /health`: Application health status
- `GET /`: Application info and version

### Health Check Response

```json
{
  "status": "healthy",
  "uptime": 3600,
  "timestamp": "2025-01-07T10:00:00.000Z",
  "environment": "production"
}
```

## 🧪 Testing

### Test Types

- **Unit Tests**: Jest with Supertest
- **Integration Tests**: Docker Compose test environment
- **Security Tests**: npm audit, Trivy scanning
- **Health Checks**: Kubernetes probes

### Coverage Reports

Generated in `coverage/` directory with HTML and XML formats.

## 📝 Configuration

### Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
NODE_ENV=development
PORT=3000
DB_HOST=localhost
JWT_SECRET=your-secret-key
```

### Jenkins Configuration

Required plugins:
- Docker Pipeline
- Kubernetes CLI
- Slack Notification
- Test Results Analyzer

## 🚨 Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 3000, 6379, and 80 are available
2. **Docker permissions**: Add user to docker group
3. **Memory issues**: Increase Docker memory allocation
4. **Health check failures**: Check application logs

### Debug Commands

```bash
# View application logs
docker-compose logs app

# Check container health
docker inspect --format='{{.State.Health.Status}}' <container-id>

# Kubernetes debugging
kubectl describe pods -l app=devops-app
kubectl logs -l app=devops-app
```

## 📚 API Documentation

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | / | Application info |
| GET | /health | Health check |
| GET | /api/users | List users |
| POST | /api/users | Create user |

### Example Requests

```bash
# Get users
curl http://localhost:3000/api/users

# Create user
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","role":"developer"}'
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Make changes and test locally
4. Commit changes: `git commit -am 'Add new feature'`
5. Push to branch: `git push origin feature/new-feature`
6. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 Additional Resources

- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Kubernetes Deployment Guide](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/)