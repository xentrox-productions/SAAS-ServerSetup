# Xentrox SaaS Credentials & Google API Setup Guide

---

## 1. Overview

This folder stores the configuration files, Google Service Account credentials (`google-credentials.json`), and API integration parameters for the Xentrox LinkedIn Sales Automation SaaS Engine.

---

## 2. Setting Up Google API Credentials (`google-credentials.json`)

To allow the server to read user lists and write outreach leads directly to your Google Sheet:

1. **Download Service Account Key**:
   * In your Google Cloud Console ➡️ Go to **APIs and Services ➡️ Credentials**.
   * Click your Service Account (e.g. `508767919720-compute@developer.gserviceaccount.com`).
   * Go to **Keys** tab ➡️ Click **Add Key** ➡️ **Create new key** ➡️ Select **JSON**.
   * Save the downloaded `.json` file as `google-credentials.json`.

2. **Upload to Server**:
   Upload `google-credentials.json` to `/opt/linkedin-automation/google-credentials.json` on your Linux server.

3. **Share Google Sheet**:
   Share your master Google Sheet with your Service Account Email and grant **Editor** access.
