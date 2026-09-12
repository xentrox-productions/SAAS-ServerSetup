# Multi-Tenant SaaS Server Setup & Infrastructure Guide
## Scalable, Low-RAM LinkedIn Automation Infrastructure

[![Docker](https://img.shields.io/badge/Docker-24.0+-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Google Cloud](https://img.shields.io/badge/Google_Cloud-GCP-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Executive Summary

This repository contains the complete architectural blueprints, memory optimization strategies, and server deployment plans to convert a single-tenant LinkedIn automation engine into a **high-density, multi-tenant SaaS platform**.

### Key Achievements:
* **Memory Reduction**: Slashes RAM usage from **1.5 GB per user down to ~80 MB per active worker tab** (95% memory savings).
* **Zero Code Duplication**: Dependencies (Node.js, Playwright, Chromium) are installed **ONCE** system-wide in a master Docker base image.
* **Mass Scaling**: Serves **100+ active SaaS client accounts** on a single **$10–$15/month Linux Cloud VPS** (2 vCPU / 4 GB RAM).
* **No SQL Database Needed**: Integrates with **Google Sheets** as a no-code CRM, user database, and remote admin control panel.
* **1-Click Remote Control**: Activating or deactivating users is as simple as toggling a cell in Google Sheets (`ACTIVE` vs `PAUSED`), instantly dropping deactivated users to **0% RAM / 0% CPU**.

---

## System Architecture Diagram

```mermaid
flowchart TD
    subgraph Control_Layer [Master Control Center]
        GS[(Google Sheet CRM & Database\nColumns: User ID | Status | Search URL | Cookie Link)]
        ADMIN[Admin / Owner\nRemote Activation Toggle] -->|Set Cell: ACTIVE / PAUSED| GS
        EXT[Client Onboarding Form / Chrome Extension] -->|Webhook API| API[Orchestrator Control Plane\n'linkedin_control_plane']
        API -->|Auto-Append Row & Save Cookie| GS
    end

    subgraph Server_Infrastructure [Linux Cloud VPS - Ubuntu 22.04 LTS]
        API -->|Mounts /var/run/docker.sock| DKR[Host Docker Daemon]
        
        subgraph Worker_Pool [Shared Memory Base Image Container Pool]
            DKR -->|Spawns On-Demand| W1[Worker Instance 1\nRAM: ~80 MB]
            DKR -->|Spawns On-Demand| W2[Worker Instance 2\nRAM: ~80 MB]
            DKR -->|Spawns On-Demand| W3[Worker Instance 3\nRAM: ~80 MB]
        end

        subgraph Interception_Layer [Resource Stripping Engine]
            W1 & W2 & W3 -->|Abort Payloads| STRIP[Blocked Asset Types:\n❌ Images .png, .jpg, .svg\n❌ Videos .mp4, .webm\n❌ Web Fonts .woff, .ttf\n❌ Telemetry & Tracking]
        end
    end

    subgraph External_Target [Target Destination]
        W1 & W2 & W3 -->|Headless Outreach| LINKEDIN[LinkedIn Web API]
        W1 & W2 & W3 -->|Write Logs & Leads| GS
    end
```

---

## Documentation Index

This repository provides step-by-step guides covering every aspect of the infrastructure:

| Guide Document | Description & Key Contents |
| :--- | :--- |
| 📄 [SAAS_PLATFORM_ARCHITECTURE.md](SAAS_PLATFORM_ARCHITECTURE.md) | High-level multi-tenant platform design, Google Cloud infrastructure tier, and backend toggle logic. |
| 📄 [SERVER_HARDWARE_INFRASTRUCTURE.md](SERVER_HARDWARE_INFRASTRUCTURE.md) | Physical server specs, Ubuntu 22.04 LTS installation, swap memory tuning, and Docker CE installation. |
| 📄 [SERVER_INFRASTRUCTURE.md](SERVER_INFRASTRUCTURE.md) | Core server pillars, per-user variables breakdown, and zero-downtime software update procedures. |
| 📄 [GOOGLE_SHEET_DOCKER_DEPLOYMENT_PLAN.md](GOOGLE_SHEET_DOCKER_DEPLOYMENT_PLAN.md) | Google Sheets CRM integration setup, Dockerfile, and `docker-compose.yml` configuration. |
| 📄 [DOCKER_USER_MANAGEMENT_AND_SHARED_DEPENDENCIES.md](DOCKER_USER_MANAGEMENT_AND_SHARED_DEPENDENCIES.md) | Docker image layer sharing (Copy-on-Write) and dynamic on-demand container spawning. |
| 📄 [MEMORY_OPTIMIZATION_AND_MASS_SCALING.md](MEMORY_OPTIMIZATION_AND_MASS_SCALING.md) | Stripped Chromium flags, network asset interception code, and task concurrency capacity math. |
| 📄 [AUTOMATED_ONBOARDING_AND_ORCHESTRATION.md](AUTOMATED_ONBOARDING_AND_ORCHESTRATION.md) | Hands-free client onboarding, Webhook API, Chrome extension cookie capture, and orchestrator placement. |

---

## Quickstart Deployment Guide

### 1. Server Prerequisites
* Operating System: **Ubuntu 22.04 LTS (64-bit)**
* Recommended Server Spec: 2 vCPU, 4 GB RAM, 25 GB SSD (e.g. Google Cloud Compute Engine, DigitalOcean, or Hetzner).

### 2. Install Docker & Prepare System
```bash
# Update Linux packages & install Docker CE
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com | sh
sudo systemctl enable --now docker

# Setup 4GB Swap file to prevent OOM memory crashes
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Create server directory
mkdir -p /opt/linkedin-automation/sessions
cd /opt/linkedin-automation
```

### 3. Deploy Engine
```bash
# Clone setup repository
git clone https://github.com/xentrox-productions/SAAS-ServerSetup.git .

# Start the master container engine
docker compose up -d --build
```

---

## Memory Optimization Comparison

| Metric | Traditional Single-Tenant Setup | Optimized SaaS Multi-Tenant Setup |
| :--- | :--- | :--- |
| **Disk Space per User** | ~1.05 GB (Duplicated folders) | **0 MB** (Shared Docker Base Image) |
| **RAM per Active Worker** | ~1.5 GB – 2.0 GB RAM | **~80 MB – 120 MB RAM** |
| **Idle RAM per User** | ~1.5 GB RAM (Always-on browser) | **0 MB RAM** (Context closes after task) |
| **100-User Server Cost** | ~$150 – $200 / month | **~$10 – $15 / month** |
| **User Status Management** | Manual file editing | 1-Click Toggle in Google Sheets (`ACTIVE`/`PAUSED`) |
