# FastAPI + React Application Deployment on AWS EC2

A comprehensive guide to deploy a full-stack FastAPI (backend) + React (frontend) application on AWS EC2 with automated CI/CD using GitHub Actions and Docker.

## Table of Contents

1. [Launch EC2 Instance](#1-launch-ec2-instance)
2. [SSH to EC2](#2-ssh-to-ec2)
3. [Install Docker & Docker Compose](#3-install-docker--docker-compose)
4. [Create Deploy SSH Keypair](#4-create-deploy-ssh-keypair)
5. [Upload Public Key to EC2](#5-upload-public-key-to-ec2)
6. [Clone Repository on EC2](#6-clone-repository-on-ec2)
7. [Configure Environment Variables](#7-configure-environment-variables)
8. [Prepare Dockerfile](#8-prepare-dockerfile)
9. [Configure GitHub Secrets](#9-configure-github-secrets)
10. [Setup GitHub Actions Workflow](#10-setup-github-actions-workflow)
11. [Deploy and Monitor](#11-deploy-and-monitor)
12. [Troubleshooting Commands](#12-troubleshooting-commands)
13. [Security Checklist](#13-security-checklist)

---

## 1. Launch EC2 Instance

### Steps:
1. Go to **AWS Console** → **EC2** → **Launch instance**
2. Configure the following settings:

| Setting | Value |
|---------|-------|
| **AMI** | Ubuntu Server 22.04 LTS (recommended) |
| **Instance type** | t3.micro or t2.micro (free-tier eligible) |
| **Key pair** | Create new or use existing `.pem` file (keep secure!) |
| **Network** | Default VPC is fine |
| **Auto-assign Public IP** | Enable |
| **Storage** | 8 GiB (default) |

### Security Group Inbound Rules (Important):

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | 22 | Your IP (use "My IP") |
| Custom TCP | TCP | 8000 | 0.0.0.0/0 (or Your IP) |
| HTTP (Optional) | TCP | 80 | 0.0.0.0/0 |

3. **Launch** the instance
4. **Note down** the **Public IPv4 address** from the instance details

---

## 2. SSH to EC2

### From Your Local Machine:

```bash
# Set correct permissions on your AWS key
chmod 400 ec2-key.pem

# SSH to instance (replace <EC2_PUBLIC_IP> with your instance IP)
ssh -i ec2-key.pem ubuntu@<EC2_PUBLIC_IP>
```

✅ If successful, you'll see the Ubuntu prompt on your EC2 instance.

---

## 3. Install Docker & Docker Compose

### Run these commands on the EC2 instance:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine, CLI, and Compose plugin
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Allow ubuntu user to run Docker without sudo
sudo usermod -aG docker $USER

# Apply group changes immediately
newgrp docker
```

### Verify Installation:

```bash
docker --version
docker compose version
```

---

## 4. Create Deploy SSH Keypair

### On Your Local Machine (NOT on EC2):

This keypair is for GitHub Actions automation.

```bash
# Generate SSH key without passphrase
ssh-keygen -t ed25519 -f github_deploy_key -C "github-actions-deploy" -N ""
```

**Files created:**
- `github_deploy_key` (private key) — for GitHub Secrets
- `github_deploy_key.pub` (public key) — for EC2

⚠️ **Important:** No passphrase (`-N ""`) because GitHub Actions needs non-interactive access.

---

## 5. Upload Public Key to EC2

### Copy Public Key to EC2:

```bash
# From your local machine (in the directory with both keys)
scp -i ec2-key.pem github_deploy_key.pub ubuntu@<EC2_PUBLIC_IP>:/home/ubuntu/
```

### Add to Authorized Keys on EC2:

```bash
# SSH into EC2
ssh -i ec2-key.pem ubuntu@<EC2_PUBLIC_IP>

# On EC2, run:
mkdir -p ~/.ssh
cat ~/github_deploy_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm ~/github_deploy_key.pub
```

### Verify:

```bash
cat ~/.ssh/authorized_keys
# You should see a line starting with: ssh-ed25519 AAAA...
```

✅ This allows the deploy key to authenticate via SSH.

---

## 6. Clone Repository on EC2

### Option A: Public Repository (HTTPS)

```bash
cd ~
git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO.git app
cd app
```

### Option B: Private Repository (SSH with Deploy Key)

```bash
# Create SSH config for GitHub
cat > ~/.ssh/config <<'EOF'
Host github.com
  HostName github.com
  IdentityFile ~/.ssh/github_deploy_key
  IdentitiesOnly yes
  User git
EOF
chmod 600 ~/.ssh/config

# Copy private deploy key from local to EC2
# From local machine:
scp -i ec2-key.pem github_deploy_key ubuntu@<EC2_PUBLIC_IP>:/home/ubuntu/.ssh/github_deploy_key

# On EC2:
chmod 600 ~/.ssh/github_deploy_key

# Clone repository
git clone git@github.com:YOUR_GITHUB_USERNAME/YOUR_REPO.git app
cd app
```

---

## 7. Configure Environment Variables

### Create `.env` File on EC2:

```bash
cd ~/app
cat > .env <<'EOF'
DB_HOST=localhost
DB_PORT=3306
DB_NAME=mydb
DB_USER=root
DB_PASS=secret

SECRET_KEY=change_this_to_secret
ENCODE_ALGORITHM=HS256
ACCESS_TOKEN_LIFE_MINUIT=30
EOF
```

⚠️ **Important:** 
- Adjust variables according to your application
- Keep secrets out of Git
- Never commit `.env` to version control

---

## 8. Prepare Dockerfile

### Create Multi-Stage Dockerfile at Repository Root:

```dockerfile
# Multi-stage build for dashboard service (Frontend + Backend)

# Stage 1: Build Frontend with Node.js
FROM node:20-alpine AS frontend-builder

# Set working directory for frontend
WORKDIR /app/frontend

# Copy frontend package files
COPY frontend/package.json frontend/package-lock.json ./

# Install frontend dependencies (including dev dependencies for build)
RUN npm ci

# Copy frontend source code
COPY frontend/ ./

# Set environment variable for build (passed as build arg)
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

# Build the frontend for production
RUN npm run build

# Stage 2: Python Backend with built frontend
FROM python:3.12-slim AS backend

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy backend requirements and install Python dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy backend source code
COPY backend/ ./backend/

# Copy built frontend from Stage 1
COPY --from=frontend-builder /app/frontend/dist ./frontend/dist

# Set environment variables
ENV PYTHONPATH=/app
ENV PYTHONUNBUFFERED=1

# Expose port
EXPOSE 8000

# Command to run the FastAPI application
CMD ["uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 9. Configure GitHub Secrets

### Add Repository Secrets:

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**

Add the following secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `SSH_PRIVATE_KEY` | Content of `github_deploy_key` file | Full private key including `-----BEGIN OPENSSH PRIVATE KEY-----` |
| `HOST_IP` | `x.x.x.x` | Your EC2 public IPv4 address |
| `HOST_USER` | `ubuntu` | EC2 username |
| `VITE_API_URL` | `http://<EC2_IP>:8000` | Frontend API URL (optional) |

⚠️ **Important:** Copy the entire private key content, including header and footer lines.

---

## 10. Setup GitHub Actions Workflow

### Create Workflow File:

Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Deploy FastAPI to EC2

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

  deploy:
    runs-on: ubuntu-latest
    needs: [test]

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Deploy over SSH
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.HOST_IP }}
          username: ${{ secrets.HOST_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            set -e

            cd /home/ubuntu/app

            git fetch --all
            git reset --hard origin/main

            # STOP & REMOVE OLD CONTAINER
            docker stop fastapi-app || true
            docker rm fastapi-app || true

            # BUILD IMAGE
            docker build --build-arg VITE_API_URL=${{ secrets.VITE_API_URL }} -t fastapi-app:latest .

            # RUN CONTAINER
            docker run -d --name fastapi-app --env-file .env -p 8000:8000 fastapi-app:latest

            # CLEANUP
            docker image prune -f
```

⚠️ **Note:** Update `cd /home/ubuntu/app` to match your repository directory name.

---

## 11. Deploy and Monitor

### Trigger Deployment:

```bash
git add .
git commit -m "Add deployment workflow"
git push origin main
```

### Monitor Deployment:

1. Go to **GitHub** → **Actions** tab
2. Click on the latest workflow run
3. Watch the logs in real-time

### Common Deployment Issues:

| Error | Cause | Solution |
|-------|-------|----------|
| `Permission denied (publickey)` | Deploy key mismatch or not added to EC2 | Verify public key in `~/.ssh/authorized_keys` |
| `Connection timed out` | Wrong IP or SSH port blocked | Check `HOST_IP` and Security Group rules |
| `unexpected EOF` | Unbalanced quotes in workflow script | Check YAML syntax in workflow file |

✅ **Success:** GitHub Actions shows green checkmark ✅

### Verify Deployment:

```bash
# SSH to EC2
ssh -i ec2-key.pem ubuntu@<EC2_PUBLIC_IP>

# Check running containers
docker ps

# View logs
docker logs -f fastapi-app
```

### Access Your Application:

Open browser: `http://<EC2_PUBLIC_IP>:8000`

---

## 12. Troubleshooting Commands

### On EC2 Instance:

```bash
# Check container status
docker ps -a

# View container logs
docker logs -f fastapi-app

# Check port listening
sudo ss -tulpn | grep 8000

# Check firewall status
sudo ufw status

# Check disk space
df -h

# Remove dangling Docker images to free space
docker image prune -af

# Restart container
docker restart fastapi-app

# Rebuild and restart
docker stop fastapi-app
docker rm fastapi-app
docker build -t fastapi-app:latest .
docker run -d --name fastapi-app --env-file .env -p 8000:8000 fastapi-app:latest

# Check Docker service status
sudo systemctl status docker
```

---

## 13. Security Checklist

### ✅ Must-Do Security Steps:

- [ ] **Restrict SSH access:** Change Security Group SSH source to **Your IP only** (not `0.0.0.0/0`)
- [ ] **Protect secrets:** Never store sensitive data in Git — use GitHub Secrets or AWS Secrets Manager
- [ ] **Secure AWS key:** Keep `.pem` file locally and never expose it publicly
- [ ] **Rotate keys:** Regularly rotate SSH keys and deploy keys
- [ ] **Use HTTPS:** Set up SSL/TLS certificates (Let's Encrypt) for production
- [ ] **Enable firewall:** Configure `ufw` to only allow necessary ports
- [ ] **Regular updates:** Keep EC2 instance and Docker images updated
- [ ] **Backup data:** Regular backups of database and application data
- [ ] **Monitor logs:** Set up CloudWatch or other logging solutions
- [ ] **Use IAM roles:** For production, use AWS IAM roles instead of SSH keys

### Future Improvements:

- Use **AWS ECR** for Docker image registry
- Deploy with **AWS ECS/EKS** for better scalability
- Implement **OIDC** for GitHub Actions authentication
- Add **Application Load Balancer** for high availability
- Set up **Auto Scaling Groups** for handling traffic spikes

---

## Additional Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)

---

## License

[Your License Here]

## Contributing

[Your Contributing Guidelines Here]

---

**Happy Deploying! 🚀**