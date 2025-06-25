# ✅ Install NVM (Node Version Manager) on Ubuntu

After installing Composer, PHP, and NGINX, follow these steps to install NVM (Node Version Manager) and Node.js.

---

## 🧱 Step 1: Update Your System

First, update your package list and ensure `curl` is installed:

```bash
sudo apt update
sudo apt install curl -y
```

---

## 📥 Step 2: Download and Install NVM

Use the official NVM installation script from the GitHub repository:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Once installed, load NVM into your shell by adding the following to your `.bashrc`:

```bash
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc
```

Then apply the changes:

```bash
source ~/.bashrc
```

> 💡 If you're using Zsh instead of Bash, replace `.bashrc` with `.zshrc`.

---

## ✅ Step 3: Verify NVM Installation

To check that NVM was installed correctly:

```bash
nvm --version
```

---

## 🚀 Step 4: Install Node.js with NVM

### Install the Latest LTS Version

```bash
nvm install --lts
```

### Install a Specific Version (e.g., v18)

```bash
nvm install 18
```

### Set a Default Node Version

```bash
nvm alias default 18
```

You can now use different versions of Node.js easily with NVM.

---

## 📌 Useful NVM Commands

- List installed versions:
  ```bash
  nvm ls
  ```

- Use a specific version:
  ```bash
  nvm use 18
  ```

- List available remote versions:
  ```bash
  nvm ls-remote
  ```

---

✅ You’re all set to run Node.js projects using NVM on your Ubuntu server!
