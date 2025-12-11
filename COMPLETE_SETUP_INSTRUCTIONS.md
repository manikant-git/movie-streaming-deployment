# COMPLETE SETUP INSTRUCTIONS - Movie Streaming Microservices

## STEP 1: PREREQUISITES & ENVIRONMENT SETUP

### Required Software

```bash
# Windows/Linux/Mac - Install these tools
1. Git (v2.40+)
   Download from: https://git-scm.com/
   
2. Docker Desktop (v4.20+)
   Windows/Mac: https://www.docker.com/products/docker-desktop
   Linux: sudo apt-get install docker.io docker-compose
   
3. Docker Compose (usually comes with Docker Desktop)
   Verify: docker-compose --version
   
4. kubectl (for Kubernetes deployments)
   https://kubernetes.io/docs/tasks/tools/
   
5. Go 1.21+ (for building services locally)
   https://golang.org/dl/
```

### Verify Installations

```bash
# Run these commands to verify everything is installed
git --version
docker --version
docker-compose --version
kubectl version --client
go version
```

---

## STEP 2: CLONE ALL REPOSITORIES

### Create Project Directory

```bash
# Create a main project directory
mkdir movie-streaming-app
cd movie-streaming-app

# Initialize as git repository (optional)
git init
```

### Clone All 6 Repositories

```bash
# Clone deployment configuration repo
git clone https://github.com/manikant-git/movie-streaming-deployment.git

# Clone all 5 microservice repositories
git clone https://github.com/manikant-git/movie-streaming-api-gateway.git
git clone https://github.com/manikant-git/movie-streaming-movie-service.git
git clone https://github.com/manikant-git/movie-streaming-user-service.git
git clone https://github.com/manikant-git/movie-streaming-streaming-service.git
git clone https://github.com/manikant-git/movie-streaming-search-service.git
```

### Verify Structure

```bash
# You should see this directory structure:
ls -la

output:
drwxr-xr-x  movie-streaming-deployment
drwxr-xr-x  movie-streaming-api-gateway
drwxr-xr-x  movie-streaming-movie-service
drwxr-xr-x  movie-streaming-user-service
drwxr-xr-x  movie-streaming-streaming-service
drwxr-xr-x  movie-streaming-search-service
```

---

## STEP 3: CONFIGURE DOCKER COMPOSE

### Copy docker-compose.yml

```bash
# Copy docker-compose file to root directory
cp movie-streaming-deployment/docker-compose.yml ./docker-compose.yml

# Verify it exists
ls -l docker-compose.yml
```

### Create .env File

```bash
# Create environment configuration file
cat > .env << 'EOF'
# API Gateway Configuration
API_GATEWAY_PORT=8080
API_GATEWAY_ENV=development

# Movie Service Configuration
MOVIE_SERVICE_PORT=8081
MOVIE_DB_HOST=localhost
MOVIE_DB_PORT=5432

# User Service Configuration
USER_SERVICE_PORT=8082
USER_DB_HOST=localhost
USER_DB_PORT=5432

# Streaming Service Configuration
STREAMING_SERVICE_PORT=8083
STREAMING_TEMP_DIR=/tmp/streams

# Search Service Configuration
SEARCH_SERVICE_PORT=8084
ELASTIC_SEARCH_HOST=elasticsearch
ELASTIC_SEARCH_PORT=9200

# Common
DOCKER_NETWORK=movie-streaming-net
LOG_LEVEL=INFO
EOF

cat .env  # Verify content
```

### Update docker-compose.yml (if needed)

If any port conflicts, edit docker-compose.yml:

```yaml
# Example modification for different ports
api-gateway:
  ports:
    - "8080:8080"  # Change first number to avoid conflicts
```

---

## STEP 4: BUILD DOCKER IMAGES

### Build All Services

```bash
# From root directory containing docker-compose.yml

# Build all images
docker-compose build

# Verbose output for debugging
docker-compose build --progress=plain

# Build specific service
docker-compose build api-gateway
```

### Verify Images Built

```bash
# List all Docker images
docker images | grep movie

# Expected output:
movie-streaming-api-gateway       latest
movie-streaming-movie-service     latest
movie-streaming-user-service      latest
movie-streaming-streaming-service latest
movie-streaming-search-service    latest
```

---

## STEP 5: START ALL SERVICES

### Start in Foreground (For Testing)

```bash
# Start all services and see logs
docker-compose up

# You should see output like:
# Creating network "movie-streaming-deployment_movie-streaming-net" with driver "bridge"
# Creating movie-streaming-api-gateway   ... done
# Creating movie-streaming-movie         ... done
# Creating movie-streaming-user          ... done
# Creating movie-streaming-stream        ... done
# Creating movie-streaming-search        ... done
```

### Start in Background (Recommended)

```bash
# Start services in detached mode
docker-compose up -d

# Check if all containers are running
docker-compose ps

# Expected output:
NAME                                STATUS
movie-streaming-api-gateway         Up 2 minutes
movie-streaming-movie               Up 2 minutes
movie-streaming-user                Up 2 minutes
movie-streaming-stream              Up 2 minutes
movie-streaming-search              Up 2 minutes
```

### View Service Logs

```bash
# View all service logs
docker-compose logs

# View specific service logs
docker-compose logs api-gateway

# Follow logs in real-time
docker-compose logs -f movie-service

# View last 50 lines
docker-compose logs --tail=50 api-gateway
```

---

## STEP 6: TEST THE SERVICES

### Test API Gateway (Main Entry Point)

```bash
# Test 1: Health check without authentication (should fail)
curl -v http://localhost:8080/api/v1/movies

# Expected: 401 Unauthorized (no auth token)

# Test 2: With authentication token
curl -H "Authorization: Bearer test-token" http://localhost:8080/api/v1/movies

# Expected: Should proxy to movie service
```

### Test Movie Service Directly

```bash
# Get all movies
curl http://localhost:8081/api/v1/movies

# Expected JSON response with movie list:
# [{"id":"1","title":"Inception","genre":"Sci-Fi",...},...]

# Get specific movie by ID
curl "http://localhost:8081/api/v1/movies/id?id=1"

# Expected: Single movie JSON object
```

### Test User Service

```bash
# Test user endpoints (if implemented)
curl http://localhost:8082/health

# or
curl http://localhost:8082/api/v1/users
```

### Test Search Service

```bash
# Test search functionality
curl "http://localhost:8084/search?q=inception"
```

### Test Streaming Service

```bash
# Test streaming endpoint
curl http://localhost:8083/health
```

---

## STEP 7: MANAGE SERVICES

### Stop Services

```bash
# Stop all running services (keeps containers)
docker-compose stop

# Stop specific service
docker-compose stop api-gateway

# Verify services stopped
docker-compose ps
```

### Restart Services

```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart movie-service

# Fast restart
docker-compose down && docker-compose up -d
```

### Remove All Containers

```bash
# Remove containers (not images)
docker-compose down

# Remove containers AND volumes
docker-compose down -v

# Remove everything including images
docker-compose down --rmi all
```

---

## STEP 8: DOCKER TROUBLESHOOTING

### Check Container Status

```bash
# See detailed container info
docker-compose ps --all

# Inspect specific container
docker inspect movie-streaming-api-gateway

# Check container logs
docker logs movie-streaming-api-gateway
```

### Port Already in Use

```bash
# Find what's using a port
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows

# Solution 1: Use different port in docker-compose.yml
# Change 8080:8080 to 9080:8080

# Solution 2: Kill the process
kill -9 <PID>  # macOS/Linux
taskkill /PID <PID> /F  # Windows
```

### Rebuild Due to Code Changes

```bash
# If you modified any service code:

# Option 1: Rebuild and restart
docker-compose down
docker-compose build
docker-compose up -d

# Option 2: Quick rebuild specific service
docker-compose build --no-cache api-gateway
docker-compose up -d api-gateway
```

### Clear Docker Cache

```bash
# Remove unused images
docker image prune

# Remove unused containers
docker container prune

# Full cleanup
docker system prune -a
```

---

## STEP 9: KUBERNETES DEPLOYMENT (OPTIONAL)

### Prerequisites

```bash
# Install kubectl
https://kubernetes.io/docs/tasks/tools/

# Have a Kubernetes cluster running
# Options:
# - Minikube (local): https://minikube.sigs.k8s.io/
# - Docker Desktop built-in Kubernetes
# - AWS EKS, GCP GKE, Azure AKS
```

### Deploy to Kubernetes

```bash
# 1. Create namespace
kubectl create namespace movie-streaming

# 2. Create deployment manifests (not yet created - you need to add these)
# For now, reference the SETUP_GUIDE.md for detailed K8s instructions

# 3. Deploy services
kubectl apply -f k8s/ -n movie-streaming

# 4. Check deployment status
kubectl get deployments -n movie-streaming
kubectl get services -n movie-streaming
kubectl get pods -n movie-streaming

# 5. Access services
kubectl port-forward svc/api-gateway 8080:8080 -n movie-streaming
```

---

## STEP 10: PRODUCTION DEPLOYMENT CHECKLIST

### Before Going Live

- [ ] Add environment-specific configurations
- [ ] Set up proper logging (ELK Stack/CloudWatch)
- [ ] Configure database connections
- [ ] Set up monitoring (Prometheus + Grafana)
- [ ] Configure health checks
- [ ] Set resource limits
- [ ] Enable network policies
- [ ] Set up auto-scaling
- [ ] Configure backup strategy
- [ ] Test disaster recovery
- [ ] Security scanning (Trivy, SonarQube)
- [ ] Load testing
- [ ] Capacity planning

---

## QUICK REFERENCE COMMANDS

```bash
# Start everything
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Stop everything
docker-compose stop

# Remove everything
docker-compose down -v

# Rebuild on changes
docker-compose build --no-cache && docker-compose up -d

# Test API
curl -H "Authorization: Bearer test" http://localhost:8080/api/v1/movies
```

---

## SUPPORT & DOCUMENTATION

- **API Gateway**: https://github.com/manikant-git/movie-streaming-api-gateway
- **Movie Service**: https://github.com/manikant-git/movie-streaming-movie-service
- **User Service**: https://github.com/manikant-git/movie-streaming-user-service
- **Streaming Service**: https://github.com/manikant-git/movie-streaming-streaming-service
- **Search Service**: https://github.com/manikant-git/movie-streaming-search-service
- **Deployment**: https://github.com/manikant-git/movie-streaming-deployment

## TROUBLESHOOTING GUIDE

For issues, check:
1. Docker Desktop is running
2. Ports are not in use
3. All containers are running: `docker-compose ps`
4. Check logs: `docker-compose logs <service-name>`
5. Verify environment: `cat .env`
