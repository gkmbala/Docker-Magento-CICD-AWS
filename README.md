# Magento 2.4.x Development Setup

## Windows 10 / 11 + WSL2 + Docker 


* WSL2 (Ubuntu)
* Docker Desktop
* Docker Magento (markshust)
* Magento Open Source 2.4.x

This setup is stable, production-like, and recommended for Magento developers.

---

## 📋 System Requirements

* Windows 11 (Updated)
* Minimum 16 GB RAM recommended (8 GB minimum)
* 30–50 GB free disk space
* Admin access on Windows
* Internet connection

---

## 🧠 Architecture Overview

```
Windows 11
   ↓
WSL2 (Ubuntu Linux)
   ↓
Docker Desktop
   ↓
Magento Containers
   ├── PHP
   ├── Nginx
   ├── MySQL
   ├── Redis
   ├── OpenSearch
   └── Mailhog
```

Magento runs fully inside Docker containers.

---

# STEP 1 — Install WSL2 + Ubuntu

Open **PowerShell as Administrator**:

```powershell
wsl --install
```

Restart Windows after installation.

---

### Open Ubuntu

1. Open Start Menu
2. Search **Ubuntu**
3. Launch it
4. Create Linux username & password

Update packages:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# STEP 2 — Install Docker Desktop

Download:

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

During installation:

✅ Enable **WSL2 backend**
✅ Enable **Ubuntu integration**

Restart Windows.

---

### Verify Docker inside Ubuntu

```bash
docker --version
docker compose version
```

If versions appear → Docker is ready.

---

# STEP 3 — Install Required Tools in WSL

```bash
sudo apt install git curl unzip -y
```

(Optional but recommended)

```bash
sudo apt install make -y
```

---

# STEP 4 — Clone Docker Magento Repository

Go to home directory:

```bash
cd ~
```

Clone repository:

```bash
git clone https://github.com/markshust/docker-magento.git
cd docker-magento
```

Verify structure:

```bash
ls
```

You should see:

```
compose  lib  images  README.md
```

---

# STEP 5 — Configure Environment (.env)

⚠️ New repository structure uses:

```
compose/env/
```

Go to env directory:

```bash
cd compose/env
```

Create environment file:

```bash
cp .env.dist .env
```

---

### Edit configuration

```bash
nano .env
```

Update only these values:

```env
COMPOSE_PROJECT_NAME=magento
PHP_VERSION=8.2
MAGENTO_VERSION=2.4.7
VHOST=magento.test
```

Save:

```
CTRL + O → ENTER → CTRL + X
```

---

# STEP 6 — Start Docker Containers

Return to project root:

```bash
cd ~/docker-magento
```

Start environment:

```bash
bin/start
```

First run may take **5–10 minutes** (images download).

---

# STEP 7 — Magento Marketplace Keys

Create access keys:

[https://marketplace.magento.com/customer/accessKeys/](https://marketplace.magento.com/customer/accessKeys/)

You will get:

* Public Key
* Private Key

---

# STEP 8 — Download Magento

```bash
bin/download magento/project-community-edition
```

Enter marketplace keys when prompted.

---

# STEP 9 — Install Magento

```bash
bin/setup
```

After installation completes, you will see:

```
Frontend: https://magento.test
Admin: https://magento.test/admin_xxxxx
```

---

# STEP 10 — Configure Local Domain

Edit Windows hosts file (Run Notepad as Admin):

```
C:\Windows\System32\drivers\etc\hosts
```

Add:

```
127.0.0.1 magento.test
```

Save file.

---

# STEP 11 — Access Magento

Frontend:

```
https://magento.test
```

Admin Panel:

```
https://magento.test/admin_xxxxx
```

Default credentials:

```
Username: admin
Password: password123
```

(You may see SSL warning — safe for local development.)

---

# Useful Magento Docker Commands

### Magento CLI

```bash
bin/magento cache:flush
bin/magento setup:upgrade
bin/magento indexer:reindex
```

### Container Control

```bash
bin/start      # Start containers
bin/stop       # Stop containers
bin/restart    # Restart containers
```

### Access Containers

```bash
bin/bash       # PHP container shell
bin/mysql      # MySQL access
```

---

# Recommended Development Workflow

1. Develop locally
2. Commit changes to Git
3. Push to repository
4. Deploy to AWS/Linux server

Never develop directly on production.

---

# Important Notes

✅ Do NOT store Magento inside `/mnt/c`
Always work inside WSL home directory:

```
/home/username/
```

---

✅ Do NOT commit these folders to Git:

```
vendor/
generated/
var/
pub/static/
pub/media/
app/etc/env.php
```

---

# Common Issues & Fixes

### Docker not detected

Restart Docker Desktop and WSL:

```powershell
wsl --shutdown
```

---

### Magento slow on Windows

Ensure project is inside WSL filesystem, not Windows drive.

---

### SSL Warning

Normal for local development.

---

# Disk Usage

Approximate usage:

| Component       | Size    |
| --------------- | ------- |
| Docker Images   | 3–5 GB  |
| Magento Project | 2–3 GB  |
| Total           | ~6–8 GB |

---

# Why Use Docker for Magento?

* Exact PHP/MySQL versions
* No system conflicts
* Production-like environment
* Easy reset and rebuild
* Team consistency

---

# ✅ Setup Complete

You now have a fully functional Magento 2 development environment running on:

**Windows → WSL2 → Docker → Magento**
-----------------------------------------------------------------------------------------------------------


# 🚀 Deployment, Branch Strategy & CI/CD

-----------------------------------------------------------------------------------------------------------

**Magento 2.4.x — WSL + Docker Development Workflow**

This document explains how development moves from **local Windows (WSL + Docker)** → **GitHub** → **AWS Server** using a clean and safe workflow.

---

# 🧭 Overall Workflow (Real Production Flow)

```
Developer (Windows + WSL + Docker)
            ↓
        GitHub Repository
            ↓
        CI/CD Pipeline
            ↓
      AWS Magento Server
            ↓
        Live Website
```

---

# 🌿 Branch Strategy (Recommended for Magento)

Magento projects must remain stable because deployments affect database + cache + indexing.

We use a **simplified Git Flow**.

## Branch Structure

```
main        → Production (LIVE SERVER)
develop     → Integration / staging
feature/*   → New features
hotfix/*    → Urgent production fixes
release/*   → Pre-production testing
```

---

## ✅ Branch Purpose

| Branch      | Usage                  | Deploys To |
| ----------- | ---------------------- | ---------- |
| `main`      | Stable production code | Live AWS   |
| `develop`   | Ongoing development    | Staging    |
| `feature/*` | New module or change   | Local only |
| `hotfix/*`  | Emergency fixes        | Production |
| `release/*` | Final testing          | Staging    |

---

## Example Workflow

### 1️⃣ Create Feature

```bash
git checkout develop
git pull

git checkout -b feature/product-badge
```

Work locally in Docker Magento.

---

### 2️⃣ Push Feature

```bash
git add .
git commit -m "Add product badge feature"
git push origin feature/product-badge
```

Create Pull Request → `develop`.

---

### 3️⃣ Release Preparation

```bash
git checkout develop
git checkout -b release/v1.2.0
```

QA testing happens here.

---

### 4️⃣ Production Deployment

Merge:

```
release/v1.2.0 → main
```

Production deploy triggers automatically.

---

# ☁️ AWS Deployment Strategy (Magento Best Practice)

Magento should **NOT** deploy by FTP or manual uploads.

Use Git-based deployment.

---

## Server Structure (Recommended)

```
/var/www/magento/
    ├── current -> symlink
    ├── releases/
    │      ├── 2026_02_20_001/
    │      ├── 2026_02_25_002/
    └── shared/
           ├── pub/media
           ├── var
           └── app/etc/env.php
```

### Why?

* Zero downtime deployment
* Easy rollback
* Safe upgrades

---

# ⚙️ Magento Deployment Commands (Production)

CI/CD will run:

```bash
composer install --no-dev
php bin/magento maintenance:enable

php bin/magento setup:upgrade
php bin/magento setup:di:compile
php bin/magento setup:static-content:deploy -f

php bin/magento cache:flush
php bin/magento maintenance:disable
```

---

# 🔄 CI/CD Pipeline (Simple & Practical)

You can use:

✅ GitHub Actions (recommended)
✅ GitLab CI
✅ Bitbucket Pipelines

We’ll use **GitHub Actions example**.

---

## Repository Structure

```
.github/
   workflows/
       deploy.yml
```

---

## Example CI/CD Workflow

Create:

```
.github/workflows/deploy.yml
```

```yaml
name: Magento Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Deploy to AWS Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.AWS_HOST }}
          username: ${{ secrets.AWS_USER }}
          key: ${{ secrets.AWS_SSH_KEY }}
          script: |
            cd /var/www/magento/current
            git pull origin main

            composer install --no-dev

            php bin/magento maintenance:enable
            php bin/magento setup:upgrade
            php bin/magento setup:di:compile
            php bin/magento cache:flush
            php bin/magento maintenance:disable
```

---

# 🔐 GitHub Secrets Required

Add inside:

```
GitHub → Settings → Secrets → Actions
```

| Secret        | Description     |
| ------------- | --------------- |
| `AWS_HOST`    | Server IP       |
| `AWS_USER`    | SSH user        |
| `AWS_SSH_KEY` | Private SSH key |

---

# 🧱 Environment Separation

| Environment | Purpose                  |
| ----------- | ------------------------ |
| Local       | WSL + Docker development |
| Staging     | Testing server           |
| Production  | Live AWS                 |

---

## Important Magento Files (DO NOT COMMIT)

Add to `.gitignore`:

```
/vendor
/pub/static
/var
/generated
/app/etc/env.php
```

---

# 📦 Deployment Best Practices (Magento)

✅ Always deploy from CI/CD
✅ Never edit files directly on server
✅ Keep media outside releases
✅ Use maintenance mode during deploy
✅ Backup database before release

---

# 🔁 Rollback Strategy

If deployment fails:

```bash
cd /var/www/magento
ln -sfn releases/PREVIOUS_RELEASE current
```

Flush cache:

```bash
php bin/magento cache:flush
```

Site restored instantly.

---

# Real Developer Workflow (Daily Use)

```
1. Start Docker Magento locally
2. Develop feature
3. Push feature branch
4. PR → develop
5. Test
6. Merge → main
7. CI/CD auto deploys to AWS
```
----------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------
