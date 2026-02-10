# Netlify Deployment Fix - node_modules Error

## Problem
Netlify deployment is failing with: `ENOTDIR: not a directory, mkdir '/opt/build/repo/node_modules'`

This means a `node_modules` **file** (not directory) is committed to the GitHub repository, preventing pnpm from creating the actual `node_modules` directory during the build.

## Solution

Run these commands in your local repository to fix:

```bash
# Remove node_modules from git tracking if it exists as a file
git rm --cached node_modules 2>/dev/null || true

# Ensure .gitignore has node_modules
if ! grep -q "^node_modules/$" .gitignore; then
  echo "node_modules/" >> .gitignore
fi

# Commit and push
git add .gitignore
git commit -m "Remove mistakenly committed node_modules and update gitignore"
git push origin main
```

## After Pushing

1. Go to Netlify Dashboard
2. Click "Deploys"
3. Click "Clear cache and deploy site"
4. Wait for the build to complete (should succeed now)

## Why This Happened
- A `node_modules` file or directory was accidentally committed to git
- When Netlify clones the repo, it tries to create `node_modules` as a directory
- But since a file named `node_modules` already exists, mkdir fails with ENOTDIR error

## Prevention
- Always add `node_modules/` to `.gitignore` before committing
- Run `git rm --cached node_modules -r` to remove it if already committed
