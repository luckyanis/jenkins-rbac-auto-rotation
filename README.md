# Automated Jenkins RBAC Rotation System

[![Jenkins](https://img.shields.io/badge/Jenkins-REST%20API-red?style=flat&logo=jenkins)]()
[![Python](https://img.shields.io/badge/Python-3.x-blue?style=flat&logo=python)]()
[![Google Sheets](https://img.shields.io/badge/Google%20Sheets-API-green?style=flat&logo=google-sheets)]()

## 📌 Overview / The Problem
In a fast-paced CI/CD environment, managing access controls (RBAC) manually is error-prone and time-consuming. Previously, the team had to manually toggle user roles in Jenkins every week to determine who had deployment (Uploader) privileges for the Singapore Cruise Centre (CFOS) project. This manual process violated the principle of least privilege and introduced deployment risks.

## 💡 The Solution
I engineered a fully automated RBAC rotation system. By leveraging Python, Bash, and the Jenkins REST API, this system reads the weekly schedule from a Google Sheets document and automatically synchronizes user roles (granting `uploader` access while revoking `developer` access, and vice versa) with zero human intervention.

## 🏗️ System Architecture
1. **Google Sheets (Source of Truth):** Maintains the weekly schedule of the designated PIC (Person In Charge).
2. **Google Cloud (Service Account):** Authenticates securely via JSON key to read the spreadsheet using the `gspread` library.
3. **Python Script (`rotate-role.py`):** 
   - Calculates the current ISO week.
   - Fetches the PIC's username and maps it to the Jenkins user database.
   - Hits the Jenkins API to toggle the roles (`scc-uploader` vs `scc-dev`).
   - Includes a `--dry-run` flag for safe testing.
4. **Bash Wrapper (`run_rotation.sh`):** Executes the Python script, formats the output with timestamps using `awk`, and logs it for auditing.
5. **Cronjob:** Triggers the Bash script automatically every Monday morning.

## 🛠️ Tech Stack
- **Languages:** Python 3, Bash
- **APIs:** Jenkins REST API, Google Sheets API (`gspread`)
- **Cloud:** Google Cloud Platform (Service Accounts)
- **Automation:** Cron, Linux Systemd (optional)

## 🚀 Key Features
- **Zero-Touch Automation:** Roles are rotated automatically without manual login.
- **Safety First:** Includes a `--dry-run` simulation mode to preview changes before applying them.
- **Manual Override:** Supports `--week <number>` parameter for backfilling or testing specific schedules.
- **Audit Logging:** All executions and API responses are logged with precise timestamps for compliance.

## ⚙️ How It Works (Quick Start)
```bash
# 1. Clone the repository
git clone https://github.com/luckyanis/jenkins-rbac-auto-rotation.git
cd jenkins-rbac-auto-rotation

# 2. Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows use `venv\Scripts\activate`

# 3. Install dependencies
pip install -r requirements.txt

# 4. Make the bash script executable
chmod +x run_rotation.sh

# 5. Run a dry-run test (Safe mode)
python3 rotate-role.py --week 48 --dry-run

# 6. Execute actual rotation (Requires valid credentials)
./run_rotation.sh
