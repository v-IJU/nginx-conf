# AWS Laravel Migration Guide (EC2 + MySQL)

**Date:** August 8, 2025

## Overview

This guide outlines the process of migrating a Laravel application
running on a single EC2 instance (with MySQL installed) to a new
architecture where the App server and DB server are separated in a new
AWS account.

------------------------------------------------------------------------

## Step-by-Step Migration

### ec2 to local machine current directory

    ```
    scp -i /path/to/key.pem ec2-user@<EC2_PUBLIC_IP_OR_DNS>:/remote/path/to/file .
    
    ```

### 1. Create an AMI from the Old EC2

-   Go to EC2 dashboard \> Instances \> Select instance \> Actions \>
    Create Image.
-   Share AMI with new AWS account.
-   In the new account, launch a new EC2 instance from this shared AMI.

### 2. Backup MySQL from Old EC2

``` bash
mysqldump -u root -p your_database > backup.sql
```

### 3. Transfer Backup to New DB Server

``` bash
scp backup.sql ec2-user@<new-db-private-ip>:/tmp/
```

### 4. Restore Backup on New DB Server

``` bash
mysql -u root -p your_database < /tmp/backup.sql
```

### 5. Update Laravel .env

``` env
DB_HOST=<private-ip-of-new-db>
DB_DATABASE=your_database
DB_USERNAME=your_user
DB_PASSWORD=your_password
```

### 6. Test the Application

-   Ensure DB connection is working.
-   Check all pages and functionalities.

### 7. Remove MySQL from App Server (Optional)

#### Ubuntu/Debian:

``` bash
sudo apt purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*
sudo apt autoremove
sudo apt autoclean
```

#### Amazon Linux / RHEL:

``` bash
sudo yum remove mysql mysql-server
```

------------------------------------------------------------------------

## Best Practices

-   Limit DB access via Security Groups to app server only.
-   Enable automatic backups on DB server.
-   Harden the EC2 server (fail2ban, ufw/firewalld, etc.).
-   Ensure environment variables and storage are correctly configured.

------------------------------------------------------------------------

Migration complete. Ensure all services (queues, caches, scheduled jobs)
are also reviewed.
