# Complete Docker Guide - From Basics to Production

## What is Docker?

**Docker** is a platform that allows you to package your application and all its dependencies into a standardized unit called a **container**. Think of it as a lightweight, portable box that contains everything your application needs to run - code, runtime, system tools, libraries, and settings.

Unlike traditional virtual machines, Docker containers share the host system's kernel, making them incredibly lightweight and fast to start (seconds instead of minutes).

### The Shipping Container Analogy

Imagine shipping physical goods:

**Before Containers (Traditional Shipping)**:
- Different goods needed different handling
- Loading/unloading was slow and complex
- Breakage and loss were common
- Each port needed different equipment

**With Shipping Containers**:
- ✅ Standardized size and shape
- ✅ Can be moved by any crane
- ✅ Stack efficiently on ships, trucks, trains
- ✅ Protected contents during transport

**Docker does the same for software**:
- Your app runs the same everywhere
- Easy to move between development, testing, and production
- Protected from environment differences
- Standardized deployment process

---

## Why Should You Use Docker?

### The Classic Developer Problem: "It Works On My Machine" 😤

**The Nightmare Scenario**:

```
Developer: "The app works perfectly on my laptop!"
QA Team: "It crashes on the testing server..."
DevOps: "It won't even start in production..."
```

**Why this happens**:
- Different operating systems (Windows dev, Linux production)
- Different versions of dependencies (Python 3.9 vs 3.11)
- Missing environment variables
- Different system libraries
- Conflicting software versions

**Docker's Solution**: "If it works in a Docker container on your machine, it works everywhere."

---

## Problems Docker Solves

### 1. Dependency Hell 🔥

**Without Docker**:
```bash
# Your server needs:
- Python 3.11
- Node.js 18
- MySQL 8.0
- Redis 7.0
- Nginx
- Specific system libraries

# Installing all this:
- Takes hours
- Version conflicts
- Different package managers
- OS-specific issues
- Manual configuration
```

**With Docker**:
```bash
docker-compose up
# Everything starts in 30 seconds ✨
```

### 2. Environment Consistency

**Without Docker**:
- Development: Windows with Python 3.11
- Staging: Ubuntu 20.04 with Python 3.9
- Production: Ubuntu 22.04 with Python 3.10
- Result: Bugs that only appear in production 😱

**With Docker**:
- All environments use the exact same container
- If it works locally, it works in production
- No more "but it worked on my machine"

### 3. Onboarding New Developers

**Without Docker**:
```
Day 1: Install Python, Node, MySQL, Redis
Day 2: Fix version conflicts
Day 3: Configure environment variables
Day 4: Debug "missing library" errors
Day 5: Still not running...
```

**With Docker**:
```
Day 1: git clone, docker-compose up
Day 1: Start coding ✅
```

### 4. Resource Efficiency

**Virtual Machines**:
- Each VM needs full OS (GBs of RAM)
- Boot time: 1-2 minutes
- Heavy resource usage
- Can run ~10 VMs per server

**Docker Containers**:
- Share host OS kernel (MBs of RAM)
- Start time: 1-2 seconds
- Lightweight resource usage
- Can run 100+ containers per server

### 5. Microservices Architecture

**Without Docker**:
- Hard to manage multiple services
- Version conflicts between services
- Complex deployment orchestration
- Scaling is painful

**With Docker**:
- Each service in its own container
- Independent scaling
- Easy orchestration with Docker Compose/Kubernetes
- Clean separation of concerns

---

## Docker vs Traditional Deployment

| Aspect | Traditional | Docker |
|--------|-------------|--------|
| **Setup Time** | Hours to days | Minutes |
| **Portability** | OS-dependent | Runs anywhere |
| **Isolation** | Minimal | Complete |
| **Resource Usage** | Heavy (VMs) | Lightweight |
| **Scaling** | Complex | Simple |
| **Rollback** | Difficult | Instant |
| **Version Control** | Manual docs | Dockerfile (code) |
| **Consistency** | "Works on my machine" | Works everywhere |

---

## Key Docker Concepts

### 1. Images 📦

**What**: A blueprint/template for containers. Like a class in programming.

**Characteristics**:
- Read-only template
- Built from a Dockerfile
- Stored in registries (Docker Hub, AWS ECR)
- Versioned with tags

**Example**:
```bash
python:3.11        # Official Python image
nginx:alpine       # Lightweight Nginx
mysql:8.0          # MySQL database
```

### 2. Containers 🏃

**What**: A running instance of an image. Like an object in programming.

**Characteristics**:
- Isolated process
- Has its own filesystem
- Can be started, stopped, deleted
- Lightweight and fast

**Example**:
```bash
docker run python:3.11
# Creates and runs a container from the Python image
```

### 3. Dockerfile 📝

**What**: A text file with instructions to build an image.

**Think of it as**: A recipe that tells Docker how to prepare your application.

### 4. Docker Compose 🎼

**What**: A tool to define and run multi-container applications.

**Think of it as**: A conductor orchestrating multiple containers working together.

### 5. Volumes 💾

**What**: Persistent storage that survives container restarts.

**Why**: Container data is deleted when the container stops. Volumes preserve it.

### 6. Networks 🌐

**What**: Allows containers to communicate with each other.

**Why**: Your backend needs to talk to your database container.

---

## Installing Docker

### Windows

1. Download **Docker Desktop** from [docker.com](https://www.docker.com/products/docker-desktop)
2. Run installer
3. Restart computer
4. Docker Desktop starts automatically
5. Verify installation:
```bash
docker --version
docker run hello-world
```

### macOS

1. Download **Docker Desktop** from [docker.com](https://www.docker.com/products/docker-desktop)
2. Drag to Applications folder
3. Open Docker from Applications
4. Verify installation:
```bash
docker --version
docker run hello-world
```

### Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt-get update

# Install dependencies
sudo apt-get install ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verify installation
sudo docker --version
sudo docker run hello-world

# Optional: Run Docker without sudo
sudo usermod -aG docker $USER
newgrp docker
```

---

## Understanding the Dockerfile

A Dockerfile is a text document containing commands to assemble an image. Let's break down each instruction:

### Essential Dockerfile Instructions

#### `FROM` - Choose Base Image
```dockerfile
FROM python:3.11-slim
```
- **What**: Specifies the parent image to build from
- **Why**: Start with a pre-built image instead of from scratch
- **Tip**: Use slim or alpine variants for smaller images

#### `WORKDIR` - Set Working Directory
```dockerfile
WORKDIR /app
```
- **What**: Sets the working directory for subsequent instructions
- **Why**: Organizes your files, like `cd /app`
- **Tip**: Creates the directory if it doesn't exist

#### `COPY` - Copy Files
```dockerfile
COPY requirements.txt .
COPY . .
```
- **What**: Copies files from host to container
- **Syntax**: `COPY <source> <destination>`
- **Tip**: Copy dependencies file first for better caching

#### `RUN` - Execute Commands
```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```
- **What**: Executes commands during image build
- **Use for**: Installing packages, compiling code
- **Tip**: Chain commands with `&&` to reduce layers

#### `ENV` - Environment Variables
```dockerfile
ENV PORT=8000
ENV PYTHONUNBUFFERED=1
```
- **What**: Sets environment variables
- **Why**: Configure application behavior
- **Tip**: Can be overridden at runtime

#### `EXPOSE` - Document Ports
```dockerfile
EXPOSE 8000
```
- **What**: Declares which port the app listens on
- **Note**: Documentary only, doesn't actually publish ports
- **Why**: Helps other developers understand the app

#### `CMD` - Default Command
```dockerfile
CMD ["python", "app.py"]
```
- **What**: Specifies the default command to run
- **Format**: JSON array format is preferred
- **Tip**: Only one CMD per Dockerfile (last one wins)

#### `ENTRYPOINT` - Fixed Command
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```
- **What**: Command that always runs (not overridable)
- **Use with CMD**: ENTRYPOINT is the program, CMD is default args
- **Tip**: Use when you want users to provide arguments

---

## Writing Dockerfiles for Different Tech Stacks

### 1. Python Flask/FastAPI Application

**Project Structure**:
```
my-python-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

**Dockerfile**:
```dockerfile
# Use official Python runtime as base image
FROM python:3.11-slim

# Set working directory in container
WORKDIR /app

# Copy requirements file
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port the app runs on
EXPOSE 5000

# Set environment variables
ENV FLASK_APP=app.py
ENV PYTHONUNBUFFERED=1

# Command to run the application
CMD ["python", "app.py"]
```

**requirements.txt**:
```txt
flask==3.0.0
gunicorn==21.2.0
pymysql==1.1.0
boto3==1.34.0
python-dotenv==1.0.0
```

**.dockerignore**:
```
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.env
.git
.gitignore
*.md
```

**Build and Run**:
```bash
# Build image
docker build -t my-python-app .

# Run container
docker run -p 5000:5000 my-python-app

# Run with environment variables
docker run -p 5000:5000 -e DATABASE_URL=mysql://user:pass@host/db my-python-app
```

---

### 2. Node.js Express Application

**Project Structure**:
```
my-node-app/
├── server.js
├── package.json
├── package-lock.json
├── Dockerfile
└── .dockerignore
```

**Dockerfile**:
```dockerfile
# Use official Node.js runtime
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set environment to production
ENV NODE_ENV=production

# Command to run the app
CMD ["node", "server.js"]
```

**.dockerignore**:
```
node_modules
npm-debug.log
.git
.env
.DS_Store
*.md
```

**Build and Run**:
```bash
# Build
docker build -t my-node-app .

# Run
docker run -p 3000:3000 my-node-app
```

---

### 3. React Frontend Application

**Multi-Stage Build** (Smaller final image):

**Dockerfile**:
```dockerfile
# Stage 1: Build the React app
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source code and build
COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

# Copy built files from stage 1
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx config (optional)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Nginx starts automatically
CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf** (for React Router):
```nginx
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

**Build and Run**:
```bash
# Build
docker build -t my-react-app .

# Run
docker run -p 80:80 my-react-app
```

---

### 4. Full-Stack Application (Python + React + MySQL)

**Project Structure**:
```
fullstack-app/
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   ├── package.json
│   └── Dockerfile
└── docker-compose.yml
```

**backend/Dockerfile**:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**frontend/Dockerfile**:
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  # MySQL Database
  database:
    image: mysql:8.0
    container_name: app-database
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp_db
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  backend:
    build: ./backend
    container_name: app-backend
    ports:
      - "5000:5000"
    environment:
      DATABASE_HOST: database
      DATABASE_USER: appuser
      DATABASE_PASSWORD: apppassword
      DATABASE_NAME: myapp_db
    depends_on:
      database:
        condition: service_healthy
    networks:
      - app-network
    volumes:
      - ./backend:/app
    command: gunicorn --bind 0.0.0.0:5000 --reload app:app

  # Frontend
  frontend:
    build: ./frontend
    container_name: app-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - app-network
    environment:
      REACT_APP_API_URL: http://localhost:5000

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge
```

**Run the entire stack**:
```bash
# Start all services
docker-compose up

# Start in detached mode (background)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Stop and remove volumes (delete database data)
docker-compose down -v
```

---

### 5. Django Application

**Dockerfile**:
```dockerfile
FROM python:3.11-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Collect static files
RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

---

### 6. Next.js Application

**Dockerfile**:
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000

CMD ["node", "server.js"]
```

**next.config.js** (required for standalone):
```javascript
module.exports = {
  output: 'standalone',
}
```

---

## Essential Docker Commands

### Image Commands

```bash
# Build an image
docker build -t myapp:1.0 .

# Build with custom Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# List images
docker images

# Remove an image
docker rmi myapp:1.0

# Remove unused images
docker image prune

# Pull image from registry
docker pull nginx:alpine

# Push image to registry
docker push username/myapp:1.0

# Tag an image
docker tag myapp:1.0 myapp:latest

# Inspect image details
docker image inspect myapp:1.0
```

### Container Commands

```bash
# Run a container
docker run myapp

# Run in detached mode (background)
docker run -d myapp

# Run with port mapping
docker run -p 8000:8000 myapp

# Run with environment variables
docker run -e DB_HOST=localhost -e DB_PORT=5432 myapp

# Run with volume mount
docker run -v /host/path:/container/path myapp

# Run with custom name
docker run --name my-container myapp

# Run interactively
docker run -it myapp /bin/bash

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop my-container

# Start a stopped container
docker start my-container

# Restart a container
docker restart my-container

# Remove a container
docker rm my-container

# Remove a running container (force)
docker rm -f my-container

# View container logs
docker logs my-container

# Follow logs in real-time
docker logs -f my-container

# Execute command in running container
docker exec my-container ls /app

# Open shell in running container
docker exec -it my-container /bin/bash

# View container resource usage
docker stats

# Inspect container details
docker inspect my-container

# Copy files from container
docker cp my-container:/app/file.txt ./

# Copy files to container
docker cp ./file.txt my-container:/app/
```

### Docker Compose Commands

```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# View logs for specific service
docker-compose logs backend

# Rebuild images
docker-compose build

# Rebuild and start
docker-compose up --build

# List running services
docker-compose ps

# Execute command in service
docker-compose exec backend python manage.py migrate

# Scale a service
docker-compose up -d --scale backend=3

# Restart services
docker-compose restart

# Pause services
docker-compose pause

# Unpause services
docker-compose unpause
```

### Cleanup Commands

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# Remove everything unused
docker system prune

# Remove everything (including volumes)
docker system prune -a --volumes
```

---

## Docker Best Practices

### 1. Use .dockerignore

**Always create a .dockerignore** to exclude unnecessary files:

```
# Version control
.git
.gitignore

# Dependencies
node_modules
__pycache__
*.pyc
venv/
env/

# Environment files
.env
.env.local
*.env

# IDE files
.vscode
.idea
*.swp

# OS files
.DS_Store
Thumbs.db

# Documentation
README.md
docs/

# Tests
tests/
*.test.js

# Build files
dist/
build/
*.log
```

### 2. Optimize Layer Caching

**Bad** (Dependencies reinstall on every code change):
```dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**Good** (Dependencies cached until requirements.txt changes):
```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### 3. Use Multi-Stage Builds

Reduces final image size dramatically:

```dockerfile
# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
```

### 4. Use Slim/Alpine Base Images

```dockerfile
# Heavy: ~900 MB
FROM python:3.11

# Better: ~150 MB
FROM python:3.11-slim

# Smallest: ~50 MB (but may need extra setup)
FROM python:3.11-alpine
```

### 5. Run as Non-Root User

**Security best practice**:

```dockerfile
FROM python:3.11-slim

# Create user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy and install as root
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy app
COPY . .

# Change ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

CMD ["python", "app.py"]
```

### 6. Use Environment Variables

```dockerfile
# In Dockerfile
ENV DATABASE_HOST=localhost
ENV PORT=8000

# Override at runtime
docker run -e DATABASE_HOST=prod-db -e PORT=9000 myapp
```

### 7. Health Checks

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["python", "app.py"]
```

### 8. Keep Images Small

```dockerfile
# Install and clean in single layer
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Use --no-cache-dir for pip
RUN pip install --no-cache-dir -r requirements.txt

# Remove unnecessary files
RUN rm -rf /tmp/* /var/tmp/*
```

---

## Troubleshooting Common Issues

### "docker: command not found"

**Problem**: Docker not installed or not in PATH

**Solution**:
```bash
# Verify installation
which docker

# If not found, reinstall Docker Desktop

# Linux: Add to PATH
export PATH=$PATH:/usr/bin/docker
```

### "permission denied while trying to connect to Docker daemon"

**Problem**: User doesn't have permission to access Docker

**Solution** (Linux):
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### "Cannot connect to the Docker daemon"

**Problem**: Docker Desktop not running

**Solution**:
- Start Docker Desktop application
- Check system tray for Docker icon

### "Container exits immediately"

**Problem**: Application crashes or nothing to keep it running

**Solutions**:
```bash
# Check logs
docker logs container-name

# Run interactively to debug
docker run -it myapp /bin/bash

# Check if CMD is correct
docker inspect myapp
```

### "Port already in use"

**Problem**: Another container or process using the port

**Solutions**:
```bash
# Check what's using the port (Linux/Mac)
lsof -i :8000

# Windows
netstat -ano | findstr :8000

# Use different port
docker run -p 8001:8000 myapp

# Stop conflicting container
docker stop $(docker ps -q --filter "publish=8000")
```

### "Image build fails"

**Problem**: Various build errors

**Solutions**:
```bash
# Build with no cache
docker build --no-cache -t myapp .

# Check Dockerfile syntax
docker build --check -t myapp .

# View detailed build output
docker build --progress=plain -t myapp .
```

### "Container can't connect to another container"

**Problem**: Networking issue

**Solutions**:
```bash
# Use Docker Compose (creates network automatically)
docker-compose up

# Or create custom network
docker network create mynetwork
docker run --network mynetwork --name db mysql
docker run --network mynetwork myapp

# Use container name as hostname
# In app: DATABASE_HOST=db (not localhost)
```

### "Volume data not persisting"

**Problem**: Data lost when container stops

**Solution**:
```bash
# Create named volume
docker volume create mydata

# Use named volume
docker run -v mydata:/app/data myapp

# Or use bind mount (development)
docker run -v $(pwd)/data:/app/data myapp
```

---

## Production Deployment Checklist

### ✅ Before Deploying to Production:

**Security**:
- [ ] Run as non-root user
- [ ] Don't expose sensitive ports publicly
- [ ] Use secrets management (not env vars for passwords)
- [ ] Scan images for vulnerabilities: `docker scan myapp`
- [ ] Use trusted base images only
- [ ] Keep images updated

**Performance**:
- [ ] Use multi-stage builds
- [ ] Optimize layer caching
- [ ] Use .dockerignore
- [ ] Use slim/alpine base images
- [ ] Set resource limits:
```bash
docker run --memory="512m" --cpus="0.5" myapp
```

**Reliability**:
- [ ] Add health checks
- [ ] Set restart policies: `--restart=always`
- [ ] Use logging drivers
- [ ] Monitor with Prometheus/Grafana
- [ ] Set up alerts

**Best Practices**:
- [ ] Tag images with versions (not just `latest`)
- [ ] Use environment variables for configuration
- [ ] Document in README how to build/run
- [ ] Test in staging environment first
- [ ] Have rollback plan ready

---

## Docker Compose Production Example

**docker-compose.prod.yml**:
```yaml
version: '3.8'

services:
  backend:
    image: myregistry/backend:1.0.5
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    environment:
      NODE_ENV: production
      DATABASE_HOST: database
    depends_on:
      - database
    networks:
      - app-network
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  database:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
      MYSQL_DATABASE: myapp
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network
    secrets:
      - db_root_password

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/ssl:ro
    depends_on:
      - backend
    networks:
      - app-network

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge

secrets:
  db_root_password:
    file: ./secrets/db_root_password.txt
```

---

## Advanced Topics

### Container Orchestration

When you have many containers across multiple servers:

**Docker Swarm** (Built-in, simpler):
```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# Scale service
docker service scale myapp_backend=5
```

**Kubernetes** (Industry standard, complex but powerful):
- Used by Google, Amazon, Microsoft
- Automatic scaling and healing
- Advanced networking and storage
- Steeper learning curve

### CI/CD Integration

**GitHub Actions Example**:
```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push myapp:${{ github.sha }}
```

### Monitoring

**Prometheus + Grafana Stack**:
```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
```

---

## Summary

You've learned:

- ✅ What Docker is and why it's revolutionary
- ✅ How Docker solves real development problems
- ✅ Writing Dockerfiles for any tech stack
- ✅ Using Docker Compose for multi-container apps
- ✅ Essential Docker commands
- ✅ Best practices and optimization techniques
- ✅ Troubleshooting common issues
- ✅ Production deployment strategies

**Next Steps**:

1. 🚀 Install Docker Desktop on your machine
2. 📝 Create a Dockerfile for your current project
3. 🏗️ Build your first Docker image
4. 🎯 Run your application in a container
5. 🎼 Use Docker Compose for multi-service apps
6. 📚 Explore Docker Hub for useful images
7. 🌟 Share your images with your team

---

## Quick Reference Cheat Sheet

### Most Common Commands

```bash
# Build
docker build -t myapp .

# Run
docker run -d -p 8000:8000 --name mycontainer myapp

# Stop
docker stop mycontainer

# View logs
docker logs -f mycontainer

# Shell access
docker exec -it mycontainer /bin/bash

# Remove container
docker rm mycontainer

# Remove image
docker rmi myapp

# Compose
docker-compose up -d
docker-compose down
docker-compose logs -f
```

### Dockerfile Template (Copy & Customize)

```dockerfile
# Choose appropriate base image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy dependency files first (for caching)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Set environment variables
ENV PYTHONUNBUFFERED=1

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8000/health || exit 1

# Run as non-root user (production)
RUN useradd -m appuser
USER appuser

# Command to run
CMD ["python", "app.py"]
```

### docker-compose.yml Template

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_HOST=db
      - DATABASE_PORT=5432
    depends_on:
      - db
    networks:
      - app-network
    volumes:
      - ./app:/app
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: unless-stopped

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

---

## Additional Resources

### Official Documentation
- [Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)

### Best Practices Guides
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Image Building Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### Learning Resources
- [Play with Docker](https://labs.play-with-docker.com/) - Free browser-based Docker playground
- [Docker Getting Started Tutorial](https://docs.docker.com/get-started/)
- [Docker Curriculum](https://docker-curriculum.com/) - Comprehensive tutorial

### Community
- [Docker Community Forums](https://forums.docker.com/)
- [Docker Reddit](https://reddit.com/r/docker)
- [Docker Slack](https://dockercommunity.slack.com/)

---

## Conclusion

Docker has transformed the way we build, ship, and run applications. What once took hours of setup now takes seconds. What once caused endless "works on my machine" problems now runs consistently everywhere.

**Remember**:
- Docker isn't magic - it's just good engineering
- Start simple, add complexity only when needed
- Use Docker Compose for local development
- Follow best practices for production
- Keep learning and experimenting

**The Docker Philosophy**:
> "Build once, run anywhere"

Now go forth and containerize everything! 🐳✨