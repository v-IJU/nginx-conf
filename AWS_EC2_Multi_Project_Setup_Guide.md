# 🏗️ AWS EC2 Multi-Project Laravel Setup Guide

## 📘 Overview
This document explains how to host **multiple Laravel projects** on a **single AWS EC2 instance (Ubuntu + Nginx)** efficiently.

Each project has:
- Its own codebase and `.env`
- Its own MySQL database
- Independent **queue workers**, **cron jobs**, and **S3 backups**

Later, this setup can evolve into a **multi-tenant Laravel architecture** using a single codebase with multiple databases.

---

## ⚙️ Server Configuration

### 🔹 EC2 Instance
| Component | Details |
|------------|----------|
| Instance Type | `t3.large` (2 vCPU, 8 GB RAM) |
| OS | Ubuntu 22.04 LTS |
| Web Server | Nginx |
| PHP | PHP 8.2 + PHP-FPM |
| Queue Manager | Supervisor |
| Scheduler | Cron |
| Backup | AWS S3 using `spatie/laravel-backup` |

---

## 🗂 Folder Structure
```bash
/var/www/
 ├── abc_project/       # Project 1 → abc.example.com
 │   ├── .env
 │   └── public/
 └── cde_project/       # Project 2 → cde.example.com
     ├── .env
     └── public/
```

---

## 🌐 Nginx Configuration

### Create virtual hosts:
#### `/etc/nginx/sites-available/abc.example.com`
```nginx
server {
    listen 80;
    server_name abc.example.com;
    root /var/www/abc_project/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    access_log /var/log/nginx/abc_access.log;
    error_log  /var/log/nginx/abc_error.log;
}
```

#### `/etc/nginx/sites-available/cde.example.com`
(similar structure, different root path)

Enable both:
```bash
sudo ln -s /etc/nginx/sites-available/abc.example.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/cde.example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 SSL Setup (Let’s Encrypt)
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d abc.example.com -d cde.example.com
```

---

## 🗃️ Database Setup (MySQL)
```sql
CREATE DATABASE abc_db;
CREATE DATABASE cde_db;

CREATE USER 'abc_user'@'localhost' IDENTIFIED BY 'abc_pass';
CREATE USER 'cde_user'@'localhost' IDENTIFIED BY 'cde_pass';

GRANT ALL PRIVILEGES ON abc_db.* TO 'abc_user'@'localhost';
GRANT ALL PRIVILEGES ON cde_db.* TO 'cde_user'@'localhost';
FLUSH PRIVILEGES;
```

Update each `.env`:
```bash
DB_DATABASE=abc_db
DB_USERNAME=abc_user
DB_PASSWORD=abc_pass
```

---

## 🚀 Deployment Steps
```bash
cd /var/www/abc_project
git pull origin main
composer install --no-dev
php artisan migrate --force
php artisan optimize:clear

cd /var/www/cde_project
git pull origin main
composer install --no-dev
php artisan migrate --force
php artisan optimize:clear
```

---

## 🤖 Supervisor (Queue Workers)

### Install
```bash
sudo apt install supervisor -y
sudo systemctl enable supervisor
```

### Configuration

#### `/etc/supervisor/conf.d/abc_queue.conf`
```ini
[program:abc_queue]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/abc_project/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/abc_project/storage/logs/queue.log
user=www-data
```

#### `/etc/supervisor/conf.d/cde_queue.conf`
(similar, change path to `/var/www/cde_project`)

Reload Supervisor:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start all
sudo supervisorctl status
```

---

## ⏰ Cron (Laravel Scheduler)
Add both projects to root crontab:
```bash
sudo crontab -e
```

Add lines:
```bash
* * * * * cd /var/www/abc_project && php artisan schedule:run >> /dev/null 2>&1
* * * * * cd /var/www/cde_project && php artisan schedule:run >> /dev/null 2>&1
```

---

## ☁️ S3 Backup Setup

Install:
```bash
composer require spatie/laravel-backup
```

Configure `.env`:
```bash
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_DEFAULT_REGION=ap-south-1
AWS_BUCKET=your-backup-bucket
```

Publish config:
```bash
php artisan vendor:publish --provider="Spatie\\Backup\\BackupServiceProvider"
```

Schedule backup:
```php
// app/Console/Kernel.php
$schedule->command('backup:run')->dailyAt('01:00');
```

---

## 🧩 Optional Automation: Bash Deploy Script

`/home/ubuntu/deploy-all.sh`
```bash
#!/bin/bash
echo "Deploying abc_project..."
cd /var/www/abc_project && git pull origin main && composer install --no-dev && php artisan migrate --force && php artisan optimize:clear

echo "Deploying cde_project..."
cd /var/www/cde_project && git pull origin main && composer install --no-dev && php artisan migrate --force && php artisan optimize:clear

sudo systemctl reload nginx
echo "✅ All deployments completed!"
```

Run:
```bash
bash /home/ubuntu/deploy-all.sh
```

---

## 💰 Cost Overview

| Setup | Components | Approx. Monthly Cost |
|--------|-------------|----------------------|
| EC2 (t3.large) | 2 Projects | $35–45 |
| RDS (Optional) | Shared MySQL | +$15–25 |
| S3 Backups | 10 GB | ~$0.25 |
| Total | --- | ~$45–70 |

---

## 🔮 Future Plan – Multi-Tenancy

Later, migrate to a **single Laravel codebase** with multiple databases using [`stancl/tenancy`](https://tenancyforlaravel.com/).

Benefits:
- Easier updates (one codebase)
- Lower maintenance
- Shared queue/scheduler
- Automatic tenant switching via domain/subdomain

---

## ✅ Summary
- One EC2, multiple Laravel apps.
- Nginx handles domain routing.
- Supervisor manages queues independently.
- Cron runs schedulers per project.
- Backups stored in S3.
- Scalable to a multi-tenant setup later.

---
**Author:** DevOps/Laravel Setup by VIJU JK  
**Date:** November 2025  
**Environment:** AWS EC2 (Ubuntu 22.04 LTS)
