# Docker Architecture: System-Wide Dependencies & Per-User Container Management

---

## Part 1: How System-Wide Dependencies Work (Zero Code Duplication)

### The Problem in Traditional Setups
If you have 10 users and create 10 separate folders, each folder contains:
* Node.js / Python packages (`node_modules`) ~600 MB
* Chromium browser binaries ~350 MB
* Project code & assets ~100 MB
* **Total Disk Space**: 10 users × 1.05 GB = **10.5 GB duplicated disk space!**

---

### The Docker Solution: Image Layering

Docker uses **Copy-on-Write (Layer Sharing)**. You build **ONE Base Docker Image** system-wide.

```mermaid
flowchart TD
    subgraph Master_Base_Image [Single System-Wide Docker Base Image - Built ONCE ~800 MB]
        OS[Linux Ubuntu OS Layer]
        NODE[Node.js Runtime Layer]
        PW[Playwright & Chromium Binaries Layer]
        DEPS[node_modules & Automation Engine Layer]
        OS --> NODE --> PW --> DEPS
    end

    subgraph User_Containers [Lightweight Per-User Containers - Read-Only Layer]
        DEPS ==>|Shared Memory Layer| U1[User 001 Container\nOnly mounts: user_001.json\nRAM: 80MB | Disk: 0 MB]
        DEPS ==>|Shared Memory Layer| U2[User 002 Container\nOnly mounts: user_002.json\nRAM: 80MB | Disk: 0 MB]
        DEPS ==>|Shared Memory Layer| U3[User 003 Container\nOnly mounts: user_003.json\nRAM: 80MB | Disk: 0 MB]
    end
```

### How Layer Sharing Works under the Hood:
1. When you run `docker build -t linkedin-engine:latest .` on your Linux server, Docker installs Node.js, Chromium, and all dependencies **ONCE** into a single image on disk (~800 MB).
2. When you start 10 or 100 user containers from `linkedin-engine:latest`, Docker **does NOT copy the files**.
3. All containers read from the **same single set of files on the Linux server**.
4. Each user container only takes a few kilobytes of memory for their specific environment variables (`USER_ID=user_001`) and session cookie file!

---

## Part 2: How Individual Users Are Added and Managed in Docker

Adding a new user does **NOT** require creating new code files or building new images. It is a 3-step process:

---

### Step 1: Add User Entry in Google Sheet
Add the new client to your master Google Sheet:

| User ID | Client Name | Status | Target Keywords | Cookie File Path |
| :--- | :--- | :--- | :--- | :--- |
| `user_001` | Client Alpha | `ACTIVE` | Software Founders | `sessions/user_001.json` |
| `user_002` | Client Beta | `ACTIVE` | Sales VP | `sessions/user_002.json` |

---

### Step 2: Upload the User's Session Cookie File
When onboarding a user, save their logged-in LinkedIn cookies to a JSON file in your server's `sessions/` directory:
```
/opt/linkedin-automation/
├── sessions/
│   ├── user_001.json   (Lightweight ~4 KB file)
│   ├── user_002.json   (Lightweight ~4 KB file)
```

---

### Step 3: Launching Per-User Docker Containers

You have two simple options for triggering containers per user:

#### Method A: On-Demand Container Spawning (Recommended for Best Memory Usage)

When it is time to run User 001's daily task, your server executes a 1-line Docker command:

```bash
docker run --rm \
  --name container_user_001 \
  -e USER_ID=user_001 \
  -e GOOGLE_SHEET_ID=your_sheet_id \
  -v /opt/linkedin-automation/sessions/user_001.json:/app/session.json \
  linkedin-engine:latest
```

**What happens step-by-step**:
1. Docker instantly boots container `container_user_001` using the shared base image (< 1 second boot time).
2. It passes `USER_ID=user_001` and mounts `user_001.json`.
3. The engine fetches `user_001`'s settings from the Google Sheet.
4. It performs outreach on LinkedIn in headless mode (using ~80 MB RAM).
5. It writes outreach results directly back into the Google Sheet.
6. The `--rm` flag tells Docker to **automatically delete the container and free 100% of RAM** as soon as the task completes.

---

#### Method B: Docker Compose Per-User Services

If you want fixed service definitions in `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Base Shared Service Config
  user_001:
    image: linkedin-engine:latest
    container_name: linkedin_user_001
    environment:
      - USER_ID=user_001
    volumes:
      - ./sessions/user_001.json:/app/session.json

  user_002:
    image: linkedin-engine:latest
    container_name: linkedin_user_002
    environment:
      - USER_ID=user_002
    volumes:
      - ./sessions/user_002.json:/app/session.json
```

To run a specific user:
```bash
docker-compose run --rm user_001
```

---

## Summary of Benefits

1. **System-Wide Dependencies**: Installed **ONCE** in the Docker image. 100 users share 1 single set of Chromium/Node binaries.
2. **Adding a User**: Simply add a row in Google Sheet + save 1 session cookie file (`user_XXX.json`).
3. **Execution**: Run `docker run -e USER_ID=user_XXX linkedin-engine:latest`.
4. **Memory Cleanliness**: Container automatically self-destructs after task completion, leaving zero leftover memory clutter.
