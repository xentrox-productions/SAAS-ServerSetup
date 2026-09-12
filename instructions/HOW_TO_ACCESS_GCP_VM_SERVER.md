# Step-by-Step Guide: Accessing Your Google Cloud Server PC

---

## Starting Point: From your current screen (APIs & Services ➡️ Credentials)

Follow these exact steps to navigate from your current screen to your Linux Server Terminal on Google Cloud:

---

### Step 1: Open the Google Cloud Navigation Menu
* Look at the very top-left corner of your screen next to the **Google Cloud** logo.
* Click the **`≡` (Hamburger Menu)** icon.

---

### Step 2: Navigate to Compute Engine
* In the left sidebar that pops out, scroll down to the **COMPUTE** section.
* Click **Compute Engine** ➡️ then click **VM instances**.

> 💡 *Shortcut Alternative*: You can also click the top search bar at the very top (`Search (/) for resources...`), type **`VM instances`**, and press **Enter**.

---

### Step 3: Check Your Instances List
* You will arrive at the **VM instances** page.
* **Case A: If a VM is already created**: You will see your server listed in the table. Skip to **Step 5**.
* **Case B: If no VM is listed**: Click the **`CREATE INSTANCE`** button at the top bar.

---

### Step 4: Configure Your Cloud PC/Server (If creating new)
1. **Name**: Type `linkedin-saas-server`.
2. **Machine Configuration**:
   * Series: Select **`E2`**.
   * Machine type: Select **`e2-standard-2`** (2 vCPUs, **8 GB RAM**).
3. **Boot Disk** (Operating System Selection):
   * Click **Change**.
   * Operating System: Select **Ubuntu**.
   * Version: Select **Ubuntu 22.04 LTS**.
   * Size: Change size to **`30 GB`**.
   * Click **Select**.
4. **Firewall**:
   * Check ✅ **Allow HTTP traffic**.
   * Check ✅ **Allow HTTPS traffic**.
5. Click **CREATE** at the bottom. Wait ~20 seconds until a green checkmark appears next to your server.

---

### Step 5: Access Your Server Terminal (The SSH Button)
* On the **VM instances** table, look at your server row.
* On the far right side of that row, click the **`SSH`** button.
* A black pop-up browser window will open automatically.
* Once loaded, you will see the Linux command prompt:
  ```bash
  username@linkedin-saas-server:~$
  ```

**Congratulations! You are now inside your Google Cloud Server PC terminal!**
