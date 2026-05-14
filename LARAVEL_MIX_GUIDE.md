# LARAVEL MIX & NPM SCRIPTS - COMPLETE GUIDE

## What is Laravel Mix?

Laravel Mix is a wrapper around **Webpack** that simplifies frontend asset compilation:

```
Your Source Files                 Webpack (Mix)           Output Files
┌─────────────────────┐          ┌──────────────┐        ┌──────────────┐
│ resources/js/*.js   │ ────────>│              │────────>│ public/js/   │
│ resources/css/*.css │          │ Laravel Mix  │        │ public/css/  │
│ resources/sass/*    │ ────────>│              │────────>│              │
└─────────────────────┘          └──────────────┘        └──────────────┘
                                 (webpack.mix.js)
```

It **compiles, minifies, and optimizes** your assets.

---

## NPM Scripts Explained

Your `package.json` scripts:

```json
{
  "scripts": {
    "dev": "npm run development",
    "development": "mix",
    "watch": "mix watch",
    "watch-poll": "mix watch -- --watch-options-poll=1000",
    "hot": "mix watch --hot",
    "prod": "npm run production",
    "production": "mix --production"
  }
}
```

### Breaking It Down:

#### 1. **`npm run dev` or `npm run development`**
```bash
npm run dev
```

**What it does:**
- Runs Mix once in **development mode**
- Compiles JS/CSS without minification (larger files, easier to debug)
- Creates source maps (helps debug in browser DevTools)
- Generates `public/mix-manifest.json` (file versioning)
- **Runs ONCE and exits**

**When to use:**
- Local development (one-time compile)
- Before pushing code (final compile)

**Output:**
```
public/
├── js/
│   ├── customapp.js          ← Compiled, NOT minified
│   └── pages.js              ← Compiled, NOT minified
├── css/
│   └── ...
└── mix-manifest.json         ← Version hashes
```

---

#### 2. **`npm run watch`**
```bash
npm run watch
```

**What it does:**
- Runs Mix in **watch mode**
- **Continuously monitors** your source files
- Auto-recompiles on any file change
- Development mode (not minified)
- **Keeps running** until you stop it (Ctrl+C)

**When to use:**
- **While actively developing** (best for local)
- Open terminal, run watch, keep it running
- Edit files → auto-compiles → refresh browser → see changes immediately

**Workflow:**
```
Terminal 1: npm run watch          ← Keep running
Terminal 2: php artisan serve      ← Keep running
Browser:    http://localhost:8000  ← Refresh to see changes
```

---

#### 3. **`npm run hot`**
```bash
npm run hot
```

**What it does:**
- Like `watch` but with **Hot Module Replacement (HMR)**
- Auto-recompiles on file changes
- **Injects changes into browser WITHOUT full refresh**
- Preserves component state (best for React/Vue)
- Much faster feedback loop

**When to use:**
- Modern SPA development (React, Vue)
- Want instant feedback without page reload
- Premium development experience

---

#### 4. **`npm run prod` or `npm run production`**
```bash
npm run production
```

**What it does:**
- Runs Mix in **production mode**
- **Minifies** JS/CSS (removes whitespace, shortens variable names)
- **Removes source maps** (smaller files, faster loading)
- Optimizes assets aggressively
- Runs **ONCE and exits**

**When to use:**
- **Before deploying to production server**
- Final build before git push
- Creates optimized files for live users

**Output:**
```
public/
├── js/
│   ├── customapp.js          ← Minified, 50% smaller
│   └── pages.js              ← Minified, optimized
├── css/
│   └── ...
└── mix-manifest.json         ← Version hashes (for cache busting)
```

---

## LOCAL DEVELOPMENT WORKFLOW ✅

### **Best Practice: Use Watch Mode**

```bash
# Step 1: Install dependencies (only once)
npm install

# Step 2: Start watching files
npm run watch

# Step 3: In ANOTHER terminal, run Laravel
php artisan serve

# Step 4: Open browser to http://localhost:8000
# Edit your JS/CSS files → Auto-compiles → Refresh browser to see changes
```

### Timeline:
```
Time 0:00  → npm run watch (starts, keeps running)
Time 0:05  → You edit resources/js/customapp.js
Time 0:06  → Mix detects change, auto-compiles
Time 0:07  → You refresh browser → See changes immediately ✓
Time 0:15  → You edit resources/css/styles.css
Time 0:16  → Mix detects change, auto-compiles
Time 0:17  → Refresh browser → See changes ✓
Time 2:00  → Done developing, press Ctrl+C to stop watch
```

---

## PRODUCTION DEPLOYMENT WORKFLOW ✅

### **Never Run `npm run dev` on Production!**

Here's the CORRECT production flow:

```bash
# LOCAL MACHINE (before pushing)
npm install              # Install dependencies
npm run production       # Create optimized build
git add public/js public/css public/mix-manifest.json
git commit -m "Build assets for production"
git push origin main

# PRODUCTION SERVER
git pull origin main
# ❌ DO NOT run: npm install or npm run dev
# ✅ Just use the compiled files that were pushed
# Assets are already in public/ folder from your local build
```

### Why NOT run npm on production?

```
❌ BAD (runs npm on server):
Server:
  git pull
  npm install              ← Installs 100MB+ of node_modules
  npm run production       ← Compiles (slow, CPU intensive)
  ✓ Server now has node_modules and compiled assets
  Problem: Wastes disk space, slow deployment, security risk

✅ GOOD (pre-compile locally):
Your Machine:
  npm run production       ← Compile locally (fast)
  git push                 ← Push compiled files
  
Server:
  git pull                 ← Just pull the ready-to-use files
  ✓ Done instantly!
  ✓ No node_modules on server
  ✓ No compilation needed
  ✓ Smaller, faster deployment
```

---

## Your Configuration Analyzed

### `webpack.mix.js`:
```javascript
mix.js("resources/js/customapp.js", "public/js").version();
```

This means:
- Input: `resources/js/customapp.js` (your source)
- Output: `public/js/customapp.js` (compiled file)
- `.version()` = adds hash to filename for cache-busting
  - File becomes: `customapp.js?id=abc123def456`
  - When you update JS, hash changes, browser downloads new version

```javascript
mix.js([...], "public/js/pages.js").version();
```

This:
- Takes 11 separate JS files (helper.js, popup-*.js, product-*.js)
- Combines them into ONE file: `public/js/pages.js`
- Minifies (if using `npm run production`)
- Adds version hash

---

## Your .gitignore Issue ❌

```ignore
/public/js       ← This line is the problem!
/public/mix-manifest.json
```

### The Problem:

Your `.gitignore` EXCLUDES `public/js` from git. So:

```
Developer's Machine:
  npm run production
  public/js/customapp.js (created locally)
  
  git add → But .gitignore says "ignore public/js"
  ✗ public/js is NOT added to git

Server:
  git pull
  public/js/ folder is EMPTY (files not in git!)
  ✗ Website breaks - no JavaScript files!
```

Then when someone manually adds CSS to `public/template_styles/css/`:
```
Server has:
  public/template_styles/css/customer_portal_styles.css (manually created)
  
Developer tries to git pull with clean public/js/:
  Conflict! Server has files that git doesn't know about
  Error: "Your local changes would be overwritten"
```

---

## CORRECT .gitignore for Mix Projects

### Option 1: Ignore node_modules Only (RECOMMENDED)

```ignore
/node_modules
/public/hot
/public/storage
/storage/*.key
/storage/logs/*.log
/storage/framework/views/*.php
/vendor
.env
.env.backup
package-lock.json
storage/debugbar
.idea
.vscode
.htaccess
npm-debug.log
yarn-error.log

# DO NOT IGNORE THESE:
# ✅ public/js/
# ✅ public/css/
# ✅ public/mix-manifest.json
```

Then your workflow is:

**Local Machine:**
```bash
npm install
npm run production        # Creates public/js, public/css, public/mix-manifest.json
git add .
git commit -m "Update assets"
git push
```

**Server:**
```bash
git pull
# Done! public/js already has compiled files
```

---

### Option 2: Ignore Compiled Files (If You Prefer)

If you DO want to ignore `public/js` in `.gitignore`:

```ignore
/public/js
/public/css
/public/mix-manifest.json
```

Then EVERY person and server must run:

**Everyone (including server):**
```bash
npm install
npm run production
```

But this is **SLOWER** and **NOT RECOMMENDED** because:
- Every deployment recompiles
- Server uses CPU/resources for compilation
- Requires node_modules on production server
- Slower deployments

---

## SOLUTION FOR YOUR PROJECT

### Step 1: Fix .gitignore

Remove these lines:
```ignore
/public/js              ← Remove this
/public/mix-manifest.json  ← Remove this
```

### Step 2: Remove node_modules and public/js from git history

```bash
# On your local machine
git rm -r --cached public/js
git rm -r --cached public/mix-manifest.json
git rm -r --cached node_modules
git commit -m "Remove build artifacts from git"
git push
```

### Step 3: Ensure public/js is COMPILED, not ignored

```bash
npm install
npm run production    # Creates public/js, public/css, public/mix-manifest.json
git add public/js public/css public/mix-manifest.json
git commit -m "Add compiled assets"
git push
```

### Step 4: Server pull

```bash
git pull    # Gets the compiled assets
# No npm needed!
```

### Step 5: For Future Changes

**Your workflow:**
```bash
# Make changes to resources/js/* or resources/css/*
npm run watch        # Auto-compiles
# Test locally
npm run production   # Final optimized build
git add .
git commit
git push

# Server:
git pull             # Gets ready-to-use files
```

---

## LOCAL vs PRODUCTION COMPARISON

| Activity | Local | Production |
|----------|-------|-----------|
| **Run npm install?** | ✅ Yes (first time) | ❌ No (not needed) |
| **Run npm run dev?** | ✅ Yes (while coding) | ❌ Never |
| **Run npm run watch?** | ✅ Yes (while coding) | ❌ No |
| **Run npm run production?** | ✅ Yes (before commit) | ❌ No (already compiled) |
| **Commit public/js?** | ✅ Yes | N/A |
| **Have public/js files?** | ✅ Yes | ✅ Yes |
| **Have node_modules?** | ✅ Yes | ❌ No |
| **Disk usage** | ~500MB | ~2MB for assets |
| **Deployment time** | N/A | ~10 seconds |

---

## Your Current Problem (Manual CSS Files)

Your developer manually created:
```
public/template_styles/css/customer_portal_styles.css
```

And committed it to git.

Then:
1. Another developer pulls
2. Runs `npm run dev` (or watch)
3. Git tries to pull but `public/` has uncommitted changes
4. **Conflict!**

### Solution:

#### **Option A: Don't manually create files (BEST)**

```bash
# Instead of manually creating CSS in public/
# Create it in resources/css/

resources/css/customer_portal_styles.css  ← Create here
↓
npm run watch  ← Automatically compiles to public/css/
```

In `webpack.mix.js`:
```javascript
mix.css('resources/css/customer_portal_styles.css', 'public/css');
```

#### **Option B: If Must Create in Public**

Only create files that don't need compilation:
```
public/images/           ← Images (static)
public/fonts/            ← Fonts (static)
public/vendor/           ← Third-party libraries

❌ NOT in public/css or public/js (these are build outputs)
```

---

## SUMMARY - DO THIS

### Local Development:
```bash
# First time setup
npm install
npm run watch  ← Keep this running while developing

# In another terminal
php artisan serve

# Edit files and refresh browser
# Watch automatically compiles changes
```

### Before Committing:
```bash
# Stop watch (Ctrl+C)
npm run production      ← Create final optimized build
git add .
git commit
git push
```

### Server Deployment:
```bash
git pull                ← That's it!
# No npm install needed
# No npm run dev needed
# Assets already compiled and ready
```

### .gitignore Should Have:
```ignore
/node_modules           ✓
/public/hot            ✓
/public/storage        ✓
# ❌ DO NOT HAVE /public/js
# ❌ DO NOT HAVE /public/mix-manifest.json
# ❌ DO NOT HAVE /public/css
```

---

## QUICK DECISION TREE

**Question: What should I do now?**

```
Are you actively coding?
  YES  → npm run watch (keep running)
  NO   → npm run production (once) → git push

Are you on production server after git pull?
  YES  → Nothing! Files are ready to use
  NO   → Don't run npm on production

Should compiled JS/CSS be in git?
  YES  → YES (most common, easier deployment)
  NO   → Everyone (including server) must run npm

Is someone manually creating files in public/?
  NO   → Good!
  YES  → Stop! Create in resources/ instead, let Mix compile
```

---

## The Right Way™ - One More Time

```bash
# LOCAL MACHINE - Development

# 1. Setup (once)
npm install

# 2. While coding (keep running in Terminal 1)
npm run watch

# 3. In Terminal 2, run Laravel
php artisan serve

# 4. Code → Save → See changes (auto-compiles via watch)

# 5. Ready to deploy?
# Stop watch (Ctrl+C)
npm run production

# 6. Commit and push
git add .
git commit -m "Update app with new features"
git push


# PRODUCTION SERVER

# 1. Pull latest code
git pull

# 2. That's it! 🎉
# No npm install
# No npm run anything
# Just serve the public/ files that are already there
```

This is the **professional way** to handle Laravel Mix deployments.
