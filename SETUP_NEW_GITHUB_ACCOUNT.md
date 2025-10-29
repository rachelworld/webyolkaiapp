# Complete Guide: Setting Up New GitHub Account and GitHub Pages

## Overview
This guide will walk you through:
1. Creating a new GitHub account
2. Setting up SSH keys for multiple GitHub accounts
3. Configuring git to use different accounts per repository
4. Creating a new repository and pushing your code
5. Setting up GitHub Pages
6. Managing both accounts going forward

---

## Step 1: Create a New GitHub Account

1. Go to [github.com](https://github.com) and click "Sign up"
2. Choose a username (this will be your GitHub Pages URL: `username.github.io`)
3. Enter your email address (use a different email from your existing account if possible)
4. Create a password and verify your account
5. Choose the free plan (includes GitHub Pages)

**Important Notes:**
- Your GitHub Pages site will be available at: `https://NEWUSERNAME.github.io`
- Keep your username and password secure

---

## Step 2: Generate SSH Key for New Account

You'll need separate SSH keys for each GitHub account to avoid conflicts.

### 2.1 Generate a New SSH Key

Open Terminal and run:

```bash
# Generate SSH key with a unique name for your new account
ssh-keygen -t ed25519 -C "your-new-email@example.com" -f ~/.ssh/id_ed25519_newaccount

# If ed25519 is not available, use RSA:
# ssh-keygen -t rsa -b 4096 -C "your-new-email@example.com" -f ~/.ssh/id_rsa_newaccount
```

When prompted:
- **Enter passphrase**: Press Enter for no passphrase, or set one (recommended)
- This creates: `~/.ssh/id_ed25519_newaccount` (private key) and `~/.ssh/id_ed25519_newaccount.pub` (public key)

### 2.2 Add SSH Key to SSH Agent

```bash
# Start the ssh-agent
eval "$(ssh-agent -s)"

# Add your new SSH private key
ssh-add ~/.ssh/id_ed25519_newaccount

# If you have an existing key, add it too
# ssh-add ~/.ssh/id_ed25519  # or whatever your old key is named
```

### 2.3 Add SSH Key to New GitHub Account

1. Display your public key:
```bash
cat ~/.ssh/id_ed25519_newaccount.pub
```

2. Copy the entire output (starts with `ssh-ed25519` or `ssh-rsa`)

3. On GitHub.com (logged into your NEW account):
   - Click your profile picture ? Settings
   - Click "SSH and GPG keys" in the left sidebar
   - Click "New SSH key"
   - Title: `MacBook - New Account` (or descriptive name)
   - Paste your public key
   - Click "Add SSH key"

---

## Step 3: Configure SSH to Use Different Keys for Different Accounts

Create/edit `~/.ssh/config` to tell SSH which key to use for which GitHub account:

```bash
# Open the config file
nano ~/.ssh/config
```

Add this configuration:

```
# GitHub - Old Account (use your existing host pattern)
Host github.com-old
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# GitHub - New Account
Host github.com-new
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_newaccount
    IdentitiesOnly yes

# Default (use old account)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

**Alternative approach (better for per-repo control):**
Use different remote URLs per repository instead. See Step 4.

---

## Step 4: Create New Repository on GitHub (New Account)

1. Log in to GitHub with your NEW account
2. Click the "+" icon in the top right ? "New repository"
3. Repository name: Choose one of:
   - `NEWUSERNAME.github.io` (for GitHub Pages at root URL)
   - OR any name (will be at `NEWUSERNAME.github.io/reponame`)
4. Description: (optional)
5. Visibility: **Public** (required for free GitHub Pages)
6. **DO NOT** initialize with README, .gitignore, or license
7. Click "Create repository"

---

## Step 5: Initialize Git Repository and Push Code

Navigate to your project directory and run:

```bash
cd /Users/benkim/code/webyolkaiapp

# Initialize git repository
git init

# Configure git to use your NEW account for this repo
git config user.name "New Account Name"
git config user.email "new-email@example.com"

# Add all files
git add .

# Make initial commit
git commit -m "Initial commit"

# Add remote (using the special hostname from SSH config)
# Replace NEWUSERNAME with your actual new GitHub username
git remote add origin git@github.com-new:NEWUSERNAME/REPONAME.git

# Or if using the alternative approach, you can use regular github.com
# but make sure to set the remote URL properly

# Push to GitHub
git branch -M main
git push -u origin main
```

**If using custom SSH hostname approach:**
- For NEW account repos: Use `git@github.com-new:username/repo.git`
- For OLD account repos: Use `git@github.com-old:username/repo.git` OR `git@github.com:username/repo.git`

---

## Step 6: Alternative Method - Use Different Remote URLs

Instead of custom SSH hostnames, you can use standard URLs but ensure each repo uses the correct account:

### For New Account Repos:
```bash
# In the new repo directory
git config user.name "New Account Name"
git config user.email "new-email@example.com"

# Set remote with explicit SSH key
git remote set-url origin git@github.com-new:NEWUSERNAME/REPONAME.git
```

### For Old Account Repos:
```bash
# In old repo directories
git config user.name "Old Account Name"
git config user.email "old-email@example.com"

# Use standard GitHub URL (will use default key)
git remote set-url origin git@github.com:OLDUSERNAME/REPONAME.git
```

---

## Step 7: Enable GitHub Pages

1. On GitHub (logged into your NEW account), go to your repository
2. Click "Settings" tab
3. Scroll to "Pages" in the left sidebar
4. Under "Source":
   - Branch: `main`
   - Folder: `/ (root)`
5. Click "Save"
6. Your site will be available at:
   - If repo is `username.github.io`: `https://username.github.io`
   - Otherwise: `https://username.github.io/reponame`

**Note:** It may take a few minutes for the site to deploy. You'll see a green checkmark when it's ready.

---

## Step 8: Managing Multiple GitHub Accounts Going Forward

### For Each Repository:

**Set local git config per repository:**
```bash
# In new account repos
cd /path/to/new-account/repo
git config user.name "New Account Name"
git config user.email "new-email@example.com"

# In old account repos  
cd /path/to/old-account/repo
git config user.name "Old Account Name"
git config user.email "old-email@example.com"
```

### Verify Which Account You're Using:

```bash
# Check current repo's git config
git config user.name
git config user.email

# Check which SSH key is being used
git remote -v
```

### When Pushing/Pulling:

- **New account repos**: Use `git@github.com-new:username/repo.git` remote URLs
- **Old account repos**: Use `git@github.com-old:username/repo.git` or `git@github.com:username/repo.git`

### Quick Check Script:

Create this script to verify your setup:

```bash
# Save as: ~/check-git-config.sh
#!/bin/bash
echo "=== Current Directory Git Config ==="
echo "User: $(git config user.name)"
echo "Email: $(git config user.email)"
echo ""
echo "=== Remote URLs ==="
git remote -v
echo ""
echo "=== SSH Keys Loaded ==="
ssh-add -l
```

---

## Step 9: Troubleshooting

### Issue: "Permission denied (publickey)"
**Solution:** Make sure you're using the correct SSH key:
```bash
# Add the key to ssh-agent
ssh-add ~/.ssh/id_ed25519_newaccount

# Test SSH connection
ssh -T git@github.com-new
```

### Issue: Wrong account committing
**Solution:** Set local git config:
```bash
git config user.name "Correct Name"
git config user.email "correct@email.com"
```

### Issue: Need to switch remote URL
**Solution:**
```bash
# For new account
git remote set-url origin git@github.com-new:username/repo.git

# For old account
git remote set-url origin git@github.com-old:username/repo.git
```

### Issue: SSH agent not persisting
**Solution:** Add to `~/.zshrc` or `~/.bashrc`:
```bash
# Add SSH keys to agent on startup
eval "$(ssh-agent -s)" > /dev/null
ssh-add ~/.ssh/id_ed25519_newaccount 2>/dev/null
ssh-add ~/.ssh/id_ed25519 2>/dev/null  # or your old key
```

---

## Step 10: Testing Your Setup

1. **Test SSH connection to new account:**
```bash
ssh -T git@github.com-new
# Should say: "Hi NEWUSERNAME! You've successfully authenticated..."
```

2. **Test SSH connection to old account:**
```bash
ssh -T git@github.com-old
# or
ssh -T git@github.com
# Should say: "Hi OLDUSERNAME! You've successfully authenticated..."
```

3. **Test commit and push:**
```bash
# In your new repo
echo "# Test" >> test.txt
git add test.txt
git commit -m "Test commit"
git push
# Should work without errors
```

---

## Quick Reference Commands

### Initialize new repo with new account:
```bash
git init
git config user.name "New Account Name"
git config user.email "new@email.com"
git add .
git commit -m "Initial commit"
git remote add origin git@github.com-new:NEWUSERNAME/REPONAME.git
git branch -M main
git push -u origin main
```

### Clone a repo with new account:
```bash
git clone git@github.com-new:NEWUSERNAME/REPONAME.git
cd REPONAME
git config user.name "New Account Name"
git config user.email "new@email.com"
```

### Clone a repo with old account:
```bash
git clone git@github.com:OLDUSERNAME/REPONAME.git
cd REPONAME
git config user.name "Old Account Name"
git config user.email "old@email.com"
```

---

## Summary Checklist

- [ ] Created new GitHub account
- [ ] Generated SSH key for new account (`id_ed25519_newaccount`)
- [ ] Added SSH key to new GitHub account
- [ ] Updated `~/.ssh/config` with both accounts
- [ ] Created new repository on GitHub (new account)
- [ ] Initialized git repo locally
- [ ] Set local git config (name & email)
- [ ] Added remote with correct SSH hostname
- [ ] Pushed code to GitHub
- [ ] Enabled GitHub Pages
- [ ] Verified site is live
- [ ] Tested SSH connections for both accounts

---

**Need Help?** If you run into issues, check:
1. SSH key is added to ssh-agent (`ssh-add -l`)
2. Git config is set per repo (`git config user.name`)
3. Remote URL uses correct SSH hostname (`git remote -v`)
4. SSH key is added to correct GitHub account (Settings ? SSH keys)