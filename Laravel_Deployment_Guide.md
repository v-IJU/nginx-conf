# Laravel Deployment Guide: Permissions & Git on EC2/Ubuntu

This document provides a **complete, production-ready setup** for fixing file permission issues, cleaning up your `storage` directory, and ensuring Git does not track unwanted runtime files.

---

## 1. Fixing File & Directory Permissions

When Laravel logs, cache, or sessions cannot be written, you may see errors like:

> **"Permission denied"**  
> **"Failed to open stream: Permission denied"**

This happens because your web server (`www-data`) does not have permission to write.

### ✅ Permanent Fix (Recommended)

Run these commands **from your Laravel project root**:

```bash
# 1. Give ownership to your user and www-data group
sudo chown -R $USER:www-data storage bootstrap/cache

# 2. Give proper read/write/execute permissions + inherit group for new files
sudo chmod -R 2775 storage bootstrap/cache
```

### 🔑 Why This Works
- `chown $USER:www-data` → Ensures web server and your user both own the folders.
- `chmod 2775` →  
  - `775` = Owner & group can read/write/execute, others can read/execute.  
  - `2` (setgid bit) = New files automatically inherit `www-data` group.

---

## 2. Proper Laravel `.gitignore` for Storage & Cache

Laravel generates runtime files in `storage/` & `bootstrap/cache/` that **should never be in Git**.

Add this to your `.gitignore` (or replace existing rules):

```gitignore
# Laravel Specific Ignores

/node_modules
/public/hot
/public/storage
/storage/*.key
/storage/app/*
/storage/framework/*
/storage/logs/*
/vendor

# Keep these folders so Laravel doesn't break
!storage/app/public/
!storage/framework/
!storage/framework/cache/
!storage/framework/sessions/
!storage/framework/views/
!storage/logs/

# Environment & IDE
.env
.phpunit.result.cache
Homestead.yaml
Homestead.json
.idea
*.sqlite
```

This is the **recommended, production-safe Laravel `.gitignore`**.

---

## 3. Clean Up Git Tracking (Once Only)

Even with `.gitignore`, Git may still track old files. Remove them safely:

```bash
git rm -r --cached storage bootstrap/cache
git add .
git commit -m "Cleanup: Stop tracking storage and cache files"
```

This will:
✅ Keep the files on disk  
✅ Stop Git from tracking them  
✅ Allow `.gitignore` to take effect

---

## 4. Ensure Empty Folders Are Committed

To make sure Laravel runs on fresh clones, create `.gitignore` files inside the required folders:

```bash
echo -e "*\n!.gitignore" > storage/framework/cache/.gitignore
echo -e "*\n!.gitignore" > storage/framework/sessions/.gitignore
echo -e "*\n!.gitignore" > storage/framework/views/.gitignore
echo -e "*\n!.gitignore" > storage/logs/.gitignore
echo -e "*\n!.gitignore" > storage/app/public/.gitignore
```

Commit these so that empty directories are always present.

---

## 5. Optional: Ignore Permission Changes Globally

If Git keeps showing permission changes, you can tell it to ignore mode changes:

```bash
git config core.fileMode false
```

---

## ✅ Final Notes

- Run the **permissions fix** after every new deployment (or automate in CI/CD).  
- Commit this `.gitignore` file and folder `.gitignore`s once — they will prevent further issues.  
- Never track logs, cache, or sessions — they are runtime data only.

