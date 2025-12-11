# Free Movie Streaming Microservices Application - Setup Guide

## Architecture Overview

This is a complete microservices-based movie streaming application with the following components:

### Microservices (5 Separate Repositories)
1. **API Gateway** (`movie-streaming-api-gateway`) - Port 8080
   - Entry point for all requests
   - Authentication and authorization
   - Request routing to appropriate services
   - Rate limiting

2. **Movie Service** (`movie-streaming-movie-service`) - Port 8081
   - Movie catalog management
   - Metadata handling
   - Movie availability

3. **User Service** (`movie-streaming-user-service`) - Port 8082
   - User authentication & JWT tokens
   - User profile management
   - User preferences

4. **Streaming Service** (`movie-streaming-streaming-service`) - Port 8083
   - Video streaming/delivery
   - Stream quality management
   - Playback management

5. **Search Service** (`movie-streaming-search-service`) - Port 8084
   - Full-text search functionality
   - Movie filtering & sorting
   - Advanced search capabilities

## Quick Start - Docker Compose (Local Development)

### Prerequisites
- Docker & Docker Compose installed
- Git

### Setup Steps

```bash
# Clone all repositories
git clone https://github.com/manikant-git/movie-streaming-deployment.git
cd movie-streaming-deployment

# Clone each microservice
git clone https://github.com/manikant-git/movie-streaming-api-gateway.git
git clone https://github.com/manikant-git/movie-streaming-movie-service.git
git clone https://github.com/manikant-git/movie-streaming-user-service.git
git clone https://github.com/manikant-git/movie-streaming-streaming-service.git
git clone https://github.com/manikant-git/movie-streaming-search-service.git

# Start all services using docker-compose
docker-compose up -d

# Verify all services are running
docker ps
```

### API Gateway Access
- URL: `http://localhost:8080`
- Example: `curl -H "Authorization: Bearer token" http://localhost:8080/api/v1/movies`

## Kubernetes Deployment

### Prerequisites
- Kubernetes cluster (minikube, EKS, GKE, etc.)
- kubectl configured
- Docker images pushed to container registry

### Deployment Steps

```bash
# Create namespace
kubectl create namespace movie-streaming

# Apply all Kubernetes manifests
kubectl apply -f k8s/ -n movie-streaming

# Check deployment status
kubectl get deployments -n movie-streaming
kubectl get services -n movie-streaming
kubectl get pods -n movie-streaming
```

## GitHub Actions CI/CD Pipeline

Each microservice has GitHub Actions workflow that:
- Builds Docker images on every push
- Runs tests
- Pushes images to container registry
- Deploys to Kubernetes cluster

### Workflow Structure
- **Build Stage**: Compile code, build Docker image
- **Test Stage**: Run unit tests
- **Push Stage**: Push image to Docker Hub/ECR
- **Deploy Stage**: Deploy to Kubernetes

## File Structure for Each Microservice

```
microservice-repo/
├── main.go              # Service implementation
├── go.mod              # Go module file
├── Dockerfile          # Docker build file
├── .gitignore
└── .github/
    └── workflows/
        └── docker-build.yml  # CI/CD pipeline
```

## Adding Missing Files to Services

To complete each microservice setup, add the following files:

### 1. User Service - Additional Files Needed
- `go.mod`
- `Dockerfile`
- `.github/workflows/docker-build.yml`

### 2. Streaming Service - Additional Files Needed
- `main.go` (video streaming handler)
- `go.mod`
- `Dockerfile`
- `.github/workflows/docker-build.yml`

### 3. Search Service - Additional Files Needed
- `main.go` (search implementation)
- `go.mod`
- `Dockerfile`
- `.github/workflows/docker-build.yml`

## Testing the Application

### Test API Gateway
```bash
curl -H "Authorization: Bearer test-token" \
  http://localhost:8080/api/v1/movies
```

### Test Movie Service
```bash
curl http://localhost:8081/api/v1/movies
curl "http://localhost:8081/api/v1/movies/id?id=1"
```

### Logs
```bash
# Docker Compose
docker-compose logs -f <service-name>

# Kubernetes
kubectl logs -f deployment/<service-name> -n movie-streaming
```

## Production Deployment Checklist

- [ ] Configure environment variables
- [ ] Set up database connections
- [ ] Configure logging (ELK Stack)
- [ ] Set up monitoring (Prometheus + Grafana)
- [ ] Configure ingress controller
- [ ] Set up auto-scaling policies
- [ ] Configure load balancing
- [ ] Set up CI/CD secrets
- [ ] Enable network policies
- [ ] Configure resource limits
- [ ] Set up health checks
- [ ] Configure backup strategy
