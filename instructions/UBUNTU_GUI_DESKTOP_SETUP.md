# Ubuntu Visual Graphical Desktop (GUI) Setup Guide
## How to View and Control Your Ubuntu Server as a Visual PC Desktop

---

## 1. Overview

By default, cloud servers run in **CLI (Terminal Command Line) mode** to save RAM and CPU. 

If you want a **Visual Desktop Interface** (with icons, Chrome browser window, taskbar, and file manager) that you can log into visually from your PC or Chrome browser, follow this guide!

---

## 2. Option A: Windows Remote Desktop Connection (xrdp) — RECOMMENDED ⭐

Connect directly to your Linux server using Windows built-in **Remote Desktop Connection** (`mstsc.exe`).

### Step 1: Install Desktop Environment & xrdp on Server
Copy and paste this command inside your black SSH Linux terminal:

```bash
sudo apt update && sudo apt install -y xfce4 xfce4-goodies xrdp && sudo systemctl enable --now xrdp && echo "xfce4-session" > ~/.xsession && sudo service xrdp restart
```

### Step 2: Set a Password for Your Linux Account
In your Linux SSH terminal, set a password for your account (`xentroxproductions`):
```bash
sudo passwd xentroxproductions
```
*(Enter a password of your choice when prompted).*

### Step 3: Allow Firewall Access in GCP Console
1. In Google Cloud Console ➡️ Go to **VPC Network ➡️ Firewall**.
2. Click **CREATE FIREWALL RULE**:
   * **Name**: `allow-rdp`
   * **Targets**: All instances in the network
   * **Source IPv4 ranges**: `0.0.0.0/0`
   * **Protocols and ports**: Specified protocols and ports ➡️ Check **TCP** ➡️ Type `3389`.
3. Click **CREATE**.

### Step 4: Open Remote Desktop on Your PC
1. Press `Windows Key + R` on your local PC ➡️ type `mstsc` ➡️ press **Enter**.
2. Paste your Google Cloud Server's **External IP Address**.
3. Click **Connect**.
4. Log in with Username `xentroxproductions` and the password you set in Step 2.

**You will now see your visual Ubuntu Desktop!** 🎉

---

## 3. Option B: Chrome Remote Desktop (Official Google Solution)

Access your Linux desktop directly inside your Chrome Browser from anywhere!

### Terminal Setup Commands:
```bash
# 1. Install XFCE Desktop
sudo apt update
sudo apt install -y xfce4 xfce4-goodies desktop-base dbus-x11 x11-xserver-utils

# 2. Download & Install Chrome Remote Desktop
wget https://dl.google.com/linux/direct/chrome-remote-desktop_current_amd64.deb
sudo dpkg -i chrome-remote-desktop_current_amd64.deb
sudo apt --fix-broken install -y
```

### Authorization Step:
1. Open [https://remotedesktop.google.com/headless](https://remotedesktop.google.com/headless) on your PC.
2. Click **Begin** ➡️ **Next** ➡️ **Authorize**.
3. Copy the Debian Linux command displayed on screen.
4. Paste it into your SSH Linux terminal and set a 6-digit PIN code.
5. Open [https://remotedesktop.google.com/access](https://remotedesktop.google.com/access) to view your visual Ubuntu Desktop in your browser!
