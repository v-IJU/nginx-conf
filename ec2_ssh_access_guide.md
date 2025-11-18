# AWS EC2: Add SSH Access for Another Developer Without Sharing `.pem`

You can add another developer to your EC2 instance **without sharing
your `.pem` key** by creating a **new Linux user** and adding their
**own SSH public key**.

This allows easy **access management** and **safe revocation** later.

------------------------------------------------------------------------

## ✅ Step 1: Get Developer's SSH Public Key

Ask the developer to generate a key pair:

### Mac / Linux / Windows (Git Bash)

``` bash
ssh-keygen -t rsa -b 4096
```

They send you only the file:

    ~/.ssh/id_rsa.pub

This file is safe to share.

------------------------------------------------------------------------

## ✅ Step 2: SSH into EC2 Using Your `.pem`

``` bash
ssh -i mykey.pem ec2-user@YOUR_EC2_PUBLIC_IP
```

------------------------------------------------------------------------

## ✅ Step 3: Create a New User

Example user: `devuser`

``` bash
sudo adduser devuser
```

------------------------------------------------------------------------

## ✅ Step 4: Add Their Public Key

``` bash
sudo mkdir /home/devuser/.ssh
sudo nano /home/devuser/.ssh/authorized_keys
```

Paste the developer's public key inside.

Set permissions:

``` bash
sudo chmod 700 /home/devuser/.ssh
sudo chmod 600 /home/devuser/.ssh/authorized_keys
sudo chown -R devuser:devuser /home/devuser/.ssh
```

------------------------------------------------------------------------

## ✅ Step 5: Developer SSH Login

The developer connects using **their private key**:

``` bash
ssh -i id_rsa devuser@YOUR_EC2_PUBLIC_IP
```

------------------------------------------------------------------------

# 🔥 Revoking Access (Anytime)

### **Option 1: Delete the user completely**

``` bash
sudo deluser --remove-home devuser
```

### **Option 2: Remove their key only**

``` bash
sudo nano /home/devuser/.ssh/authorized_keys
```

Delete their key → access instantly blocked.

------------------------------------------------------------------------

# 🛑 Notes

-   Never share your `.pem` file.
-   Each developer must use **their own** SSH key.
-   Best practice: one user per developer.

------------------------------------------------------------------------

# ⭐ Summary

  Task             Method
  ---------------- ------------------------------
  Add dev access   Create user + add public key
  Revoke access    Delete user or remove key
  Secure           Yes, per-user access
  No need          To recreate EC2
