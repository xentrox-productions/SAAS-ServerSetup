# Google Sheets + Docker Infrastructure & Deployment Blueprint
## Lightweight Linux Cloud Server Setup for Multi-User LinkedIn Automation

---

## 1. Overview: Google Sheets as the Control Center

Since you are using **Google Sheets as your storage and database**, your architecture becomes much simpler and cheaper.

* **No SQL Database Needed**: Google Sheets acts as your CRM, User Management Panel, and Master Control Switch.
* **Linux Cloud Server (Ubuntu 22.04 LTS)**: Runs a single lightweight Linux VM (Virtual Machine on Google Cloud / Hetzner / DigitalOcean) using ~200 MB RAM for the entire operating system.
* **Shared Docker Engine**: System-wide dependencies (Node.js, Playwright, Chromium) are packaged into **ONE Docker image**. Each user requires **0 MB disk space duplication**.

---

## 2. Architecture Diagram (Google Sheets + Docker)

```mermaid
flowchart TD
    subgraph Control_Center [Google Sheets Master Control]
        GS[(Google Sheet CRM & User List\nColumns: User ID | Status | Keywords | Cookie Link | Logs)]
        ADMIN[You / Admin\nChange Status: ACTIVE / PAUSED] -->|Edit Cell| GS
    end

    subgraph Linux_Server [Linux Cloud VPS Server - Ubuntu 22.04]
        subgraph Docker_Host [Shared Docker Engine]
            CRON[Scheduler / Cron Engine\nReads Google Sheet every 15 mins] <-->|Google Sheets API| GS
            
            CRON -->|Check User Status| CHK{Status == ACTIVE?}
            CHK -->|No / PAUSED| SKIP[Skip User - 0% RAM]
            CHK -->|Yes| W1[Docker Worker Task\nLoads User Cookie & Settings]
        end

        subgraph Session_Storage [Local Encrypted Storage]
            S3_BUCKET[(Local / GCS Bucket\nuser_1_cookies.json\nuser_2_cookies.json)] <--> W1
        end
    end

    subgraph LinkedIn_Web [Target Layer]
        W1 -->|Headless Chrome\nStripped Assets| LINKEDIN[LinkedIn Web]
        W1 -->|Write New Leads & Logs| GS
    end
```

---

## 3. How Google Sheets Handles User Activation & Management

Your Google Sheet serves as both the **UI Admin Panel** and **User Database**:

### Master Google Sheet Columns

| A: User ID | B: Account Name | C: Status | D: Target Search URL | E: Cookie File Link | F: Daily Limit | G: Last Run | H: Log Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `usr_001` | Client A | **ACTIVE** | `linkedin.com/search/...` | `sessions/usr_001.json` | 15 | 2026-09-12 09:00 | Completed (5 sent) |
| `usr_002` | Client B | **PAUSED** | `linkedin.com/search/...` | `sessions/usr_002.json` | 10 | 2026-09-11 14:00 | Paused by Admin |
| `usr_003` | Client C | **ACTIVE** | `linkedin.com/search/...` | `sessions/usr_003.json` | 20 | 2026-09-12 10:15 | Completed (8 sent) |

### How Remote Activation Works:
1. **To Activate a User**: Set Column C to `ACTIVE`.
2. **To Deactivate a User**: Set Column C to `PAUSED` or `DEACTIVATED`.
3. When the Linux server runs its scheduled check, it fetches the Sheet via the **Google Sheets API**. If Column C is `PAUSED`, it instantly skips the user—spending **0 RAM and 0 CPU**.

---

## 4. Docker Infrastructure Setup (System-Wide Shared Dependencies)

Instead of giving each user a separate heavy folder with `node_modules` (~1 GB), we use **Docker**.

### A. The Master Dockerfile (`Dockerfile`)
Installs Linux OS packages, Node.js, and Playwright Chromium **ONCE system-wide**:

```dockerfile
# Base lightweight Linux image
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

# Set working directory
WORKDIR /app

# Install dependencies once system-wide
COPY package*.json ./
RUN npm ci --only=production

# Copy master automation code
COPY . .

# Set environment to headless production
ENV NODE_ENV=production
ENV HEADLESS=true

# Run the Google Sheet master worker scheduler
CMD ["node", "scripts/master_sheets_worker.js"]
```

---

### B. The Docker Compose Setup (`docker-compose.yml`)
Deploys the automation engine seamlessly on your Linux server:

```yaml
version: '3.8'

services:
  linkedin-automation-engine:
    build: .
    container_name: linkedin_master_engine
    restart: always
    environment:
      - GOOGLE_SHEETS_ID=your_google_sheet_id_here
      - GOOGLE_SERVICE_ACCOUNT_EMAIL=your-service-account@iam.gserviceaccount.com
      - CONCURRENCY_LIMIT=3
    volumes:
      - ./sessions:/app/sessions       # Mount session cookie files
      - ./google-credentials.json:/app/google-credentials.json
    resources:
      limits:
        memory: 2048M                  # Hard cap RAM at 2GB for the whole server container
```

---

## 5. Step-by-Step Server Deployment Guide (How to Deploy on Linux)

### Step 1: Get a Cheap Linux VPS (Virtual Private Server)
* **Recommended OS**: Ubuntu 22.04 LTS (x86_64).
* **Provider**: Google Cloud Compute Engine (e2-standard-2 instance: 2 vCPU, 4GB RAM) or Hetzner / DigitalOcean ($10 - $15/month).

### Step 2: Install Docker on the Linux Server
Run these 2 simple commands inside the Linux terminal:
```bash
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```

### Step 3: Upload Code & Google Credentials to Server
Upload your project files and your Google Service Account key (`google-credentials.json`) to `/opt/linkedin-automation/` on the server.

### Step 4: Launch the Container
Run this single command on the Linux server:
```bash
docker-compose up -d --build
```
The server will now run 24/7 in the background.

---

## 6. What Next? Additional Low-Cost Enhancements

1. **Automated Error Alerts via Telegram / Slack Bot**:
   * Add a simple Telegram Bot webhook. If a user's LinkedIn cookie expires or gets a security challenge, the server sends a message to your Telegram:
   > ⚠️ *Alert: Client B (usr_002) session cookie expired. Please re-login in Google Sheet.*

2. **Session Cookie Manager Script**:
   * Create a 1-click helper script (`npm run login --user=usr_001`) that opens a browser locally on your PC once, lets you log into LinkedIn, saves the session file `sessions/usr_001.json`, and uploads it automatically.

3. **Auto-Restart & Health Monitoring**:
   * Use Docker health checks so if Chrome ever hangs, Docker reboots the worker container automatically in < 5 seconds without losing data.
