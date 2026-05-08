# How to Push Your Project to GitHub

## Step 1: Initialize Git Repository (if not already done)

Open PowerShell in your project directory and run:

```bash
git init
```

## Step 2: Add All Files to Git

```bash
git add .
```

This will add all files except those in `.gitignore` (like `node_modules`, `__pycache__`, etc.)

## Step 3: Create Your First Commit

```bash
git commit -m "Initial commit: Radar Coverage Analysis System - Microservices Architecture"
```

## Step 4: Create a GitHub Repository

1. Go to [GitHub.com](https://github.com) and sign in
2. Click the **"+"** icon in the top right → **"New repository"**
3. Fill in the details:
   - **Repository name**: `radar-coverage-analysis-system` (or your preferred name)
   - **Description**: "Microservices-based radar coverage analysis system with React, Node.js, Python, PostgreSQL, and Kafka"
   - **Visibility**: Choose **Public** or **Private**
   - **DO NOT** initialize with README, .gitignore, or license (we already have these)
4. Click **"Create repository"**

## Step 5: Connect Local Repository to GitHub

After creating the repository, GitHub will show you commands. Use these:

```bash
# Add the remote repository (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/radar-coverage-analysis-system.git

# Rename the default branch to main (if needed)
git branch -M main

# Push your code to GitHub
git push -u origin main
```

## Step 6: Verify

Go to your GitHub repository page and refresh. You should see all your files!

---

## Alternative: Using GitHub CLI (if installed)

If you have GitHub CLI installed:

```bash
gh repo create radar-coverage-analysis-system --public --source=. --remote=origin --push
```

---

## Quick Command Summary

```bash
# 1. Initialize (if needed)
git init

# 2. Add files
git add .

# 3. Commit
git commit -m "Initial commit: Radar Coverage Analysis System"

# 4. Add remote (replace YOUR_USERNAME and REPO_NAME)
git remote add origin https://github.com/YOUR_USERNAME/REPO_NAME.git

# 5. Push
git branch -M main
git push -u origin main
```

---

## Future Updates

When you make changes and want to push updates:

```bash
git add .
git commit -m "Description of your changes"
git push
```

---

## Troubleshooting

**If you get authentication errors:**
- Use GitHub Personal Access Token instead of password
- Or use SSH: `git remote set-url origin git@github.com:USERNAME/REPO.git`

**If repository already exists:**
```bash
git remote remove origin
git remote add origin https://github.com/YOUR_USERNAME/REPO_NAME.git
```

