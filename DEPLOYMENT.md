# Simple EC2 Deployment

This workflow deploys the Flask S3 Gallery app directly to EC2 servers using SSH.

## Required GitHub Secrets

Only **6 secrets** needed:

### Staging Environment
- `STAGING_HOST` - Staging EC2 server IP address
- `STAGING_USER` - SSH username (usually `ubuntu` or `ec2-user`)
- `STAGING_SSH_KEY` - Private SSH key for staging server

### Production Environment
- `PRODUCTION_HOST` - Production EC2 server IP address  
- `PRODUCTION_USER` - SSH username (usually `ubuntu` or `ec2-user`)
- `PRODUCTION_SSH_KEY` - Private SSH key for production server

## How It Works

1. **Push to `staging` branch** → Deploys to staging server
2. **Push to `main` branch** → Deploys to production server
3. **Manual trigger** → Choose which environment to deploy

## Server Setup

Each EC2 server needs:

```bash
# Install Docker
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Create app directory
sudo mkdir -p /opt/flask-app
sudo chown $USER:$USER /opt/flask-app

# Create environment file (copy from .env.example)
cd /opt/flask-app
# The workflow will copy .env.example, then rename it:
# cp .env.example .env
# nano .env  # Add your AWS credentials
```

**No Git required!** The GitHub Action copies files directly from the repository.

## SSH Key Setup

1. Generate SSH key pair:
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/deploy_key
   ```

2. Copy public key to server:
   ```bash
   ssh-copy-id -i ~/.ssh/deploy_key.pub user@server-ip
   ```

3. Add private key to GitHub secrets:
   ```bash
   cat ~/.ssh/deploy_key
   ```

## What the Workflow Does

1. ✅ Checks out code from GitHub
2. ✅ Connects to server via SSH  
3. ✅ Stops existing Docker container
4. ✅ **Copies files directly** from GitHub Actions to server
5. ✅ Builds new Docker image on server
6. ✅ Runs new container
7. ✅ Performs health check

**No Git needed on servers!** Files are transferred directly from the GitHub Actions runner.