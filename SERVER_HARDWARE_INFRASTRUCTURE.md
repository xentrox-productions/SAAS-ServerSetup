# Server Hardware Infrastructure & Linux Setup Guide
## Complete Guide to Setting Up Your Physical/Cloud Server for Docker Multi-User Automation

---

## 1. Recommended Server Hardware & OS Specifications

### Recommended Linux Distribution
* **Operating System**: **Ubuntu 22.04 LTS Server (64-bit)**
  * *Why Ubuntu 22.04?* It is the industry gold-standard for Docker. It uses under **200 MB RAM** for the base OS, has native Playwright/Chromium driver support, and receives security updates until 2027.

### Hardware Tiering Guide

| Scale Target | CPU Cores | System RAM | Storage | Estimated Cost |
| :--- | :--- | :--- | :--- | :--- |
| **1 – 50 Active Users** | 2 Cores | **4 GB RAM** | 25 GB SSD | ~$10 - $15 / month |
| **50 – 200 Active Users** | 4 Cores | **8 GB RAM** | 50 GB SSD | ~$20 - $30 / month |
| **200 – 1000 Active Users** | 8 Cores | **16 GB RAM** | 100 GB NVMe | ~$50 - $70 / month |

---

## 2. Step-by-Step Linux Server Setup (Copy-Paste Terminal Commands)

Connect to your server terminal via SSH: `ssh root@your_server_ip`

### Step 1: Update Linux Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git ufw htop unzip ca-certificates gnupg lsb-release
```

---

### Step 2: Install Docker CE (Community Edition) & Docker Compose
Run these official commands to install the latest Docker engine:

```bash
# Add Docker's official GPG key:
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and Docker Compose Plugin:
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Enable Docker to start automatically on system boot:
sudo systemctl enable --now docker
```

---

### Step 3: Configure Linux Swap Memory (Prevents OOM Crashes)
Setting up a 4 GB Swap File guarantees your server won't crash if memory spikes unexpectedly:

```bash
# Create a 4GB swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make swap permanent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### Step 4: Create the Server Project Directory Structure

```bash
# Create project folder on server
mkdir -p /opt/linkedin-automation/sessions

# Move into project directory
cd /opt/linkedin-automation
```

Your server folder structure will look like this:
```
/opt/linkedin-automation/
├── Dockerfile
├── docker-compose.yml
├── google-credentials.json
├── package.json
├── src/
└── sessions/               <-- Stores client session cookies (user_001.json)
```

---

## 3. Server Deployment Files

### File 1: `Dockerfile` (Place inside `/opt/linkedin-automation/`)
```dockerfile
# Official Playwright image with pre-installed Linux Chromium dependencies
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

# Set working directory inside container
WORKDIR /app

# Install project dependencies system-wide ONCE
COPY package*.json ./
RUN npm ci --only=production

# Copy application source code
COPY . .

# Set environment
ENV NODE_ENV=production
ENV HEADLESS=true

# Launch the master worker scheduler
CMD ["node", "scripts/master_sheets_worker.js"]
```

---

### File 2: `docker-compose.yml` (Place inside `/opt/linkedin-automation/`)
```yaml
version: '3.8'

services:
  linkedin-automation-engine:
    build: .
    container_name: linkedin_master_engine
    restart: always
    environment:
      - GOOGLE_SHEETS_ID=your_google_sheet_id_here
      - CONCURRENCY_LIMIT=4
    volumes:
      - ./sessions:/app/sessions
      - ./google-credentials.json:/app/google-credentials.json
    resources:
      limits:
        memory: 3500M                  # Prevents container from using more than 3.5GB RAM
```

---

## 4. Running & Managing the Server

### Start the Engine:
```bash
cd /opt/linkedin-automation
docker compose up -d --build
```

### View Live Logs:
```bash
docker logs -f linkedin_master_engine
```

### Check Container RAM & CPU Usage:
```bash
docker stats
```

### Stop the Engine:
```bash
docker compose down
```

---

## 5. Security & Firewall Basics

Enable Ubuntu Firewall (UFW) to secure your Linux server:
```bash
sudo ufw allow 22/tcp                  # Allow SSH terminal access
sudo ufw allow 80/tcp                  # Allow HTTP (if running web dashboard)
sudo ufw allow 443/tcp                 # Allow HTTPS
sudo ufw enable                        # Turn on firewall
```
