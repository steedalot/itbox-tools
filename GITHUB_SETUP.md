# GitHub Setup Guide

Quick guide for publishing this repository to GitHub.

## 1. Prepare Local Repository

```bash
cd "/Users/daniel/Developer/IONOS Server"

# Initialize git (if not already done)
git init

# Add all files
git add gtwy-tool/ tnl-tool/ docs/ examples/
git add README-root.md CHANGELOG.md LICENSE VERSION .gitignore requirements.txt
git add STRUCTURE.md

# Rename root README
mv README-root.md README.md
git add README.md

# Initial commit
git commit -m "Initial commit - v1.0.0

- gtwy: Gateway manager with DNS, SSL, nginx automation
- tnl: Client tunnel manager with systemd integration
- Complete documentation and examples
"

# Tag version
git tag -a v1.0.0 -m "Release v1.0.0 - Initial Release"
```

## 2. Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `tunnel-gateway` (or `ssh-tunnel-manager`)
3. Description: `Self-hosted SSH reverse tunnel manager for IT.Box infrastructure with automated DNS, SSL, and nginx configuration`
4. Visibility: **Public** or **Private** (your choice)
5. **DO NOT** initialize with README, .gitignore, or license (we already have these)
6. Click "Create repository"

## 3. Connect and Push

```bash
# Add remote (replace USERNAME with your GitHub username)
git remote add origin https://github.com/USERNAME/tunnel-gateway.git

# Or if using SSH:
git remote add origin git@github.com:USERNAME/tunnel-gateway.git

# Push code and tags
git push -u origin main
git push --tags
```

## 4. Create GitHub Release

1. Go to your repository on GitHub
2. Click "Releases" → "Create a new release"
3. Choose tag: `v1.0.0`
4. Release title: `v1.0.0 - Initial Release`
5. Description: (copy from CHANGELOG.md)
6. **Attach files:**
   - Upload `gtwy-tool/gtwy` as `gtwy`
   - Upload `tnl-tool/tnl` as `tnl`
7. Click "Publish release"

## 5. Configure Repository

### Topics (GitHub)
Add these topics to your repository:
- `ssh-tunnel`
- `reverse-proxy`
- `nginx`
- `letsencrypt`
- `dns-automation`
- `ionos`
- `python`
- `self-hosted`
- `devops`

### About Section
```
Self-hosted SSH reverse tunnel manager with automated DNS (IONOS), SSL (Let's Encrypt), and nginx configuration. Perfect for exposing services behind NAT.
```

### Branch Protection (optional)
Settings → Branches → Add rule:
- Branch name pattern: `main`
- ✓ Require pull request reviews before merging
- ✓ Require status checks to pass before merging

## 6. Update README URLs

After creating the repository, update these URLs in README.md:

```markdown
# Change from:
https://github.com/USERNAME/tunnel-gateway

# To your actual URL:
https://github.com/youractualusername/tunnel-gateway
```

Files to update:
- `README.md` (root)
- `gtwy-tool/README.md`
- `tnl-tool/README.md`
- `CHANGELOG.md`

## 7. Test Downloads

After release, test that downloads work:

```bash
# Download gtwy
curl -LO https://github.com/USERNAME/tunnel-gateway/releases/latest/download/gtwy
chmod +x gtwy
./gtwy --help

# Download tnl
curl -LO https://github.com/USERNAME/tunnel-gateway/releases/latest/download/tnl
chmod +x tnl
./tnl --help
```

## Optional: Add CI/CD

Create `.github/workflows/test.yml`:

```yaml
name: Test

on: [push, pull_request]

jobs:
  syntax-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.8'
      - name: Install dependencies
        run: pip install PyYAML requests
      - name: Syntax check gtwy
        run: python3 -m py_compile gtwy-tool/gtwy
      - name: Syntax check tnl
        run: python3 -m py_compile tnl-tool/tnl
```

## File Checklist

Before pushing, verify these files exist:

- [x] gtwy-tool/gtwy (executable)
- [x] gtwy-tool/README.md
- [x] tnl-tool/tnl (executable)
- [x] tnl-tool/README.md
- [x] docs/TESTING.md
- [x] docs/CLAUDE.md
- [x] examples/config.yml.example
- [x] examples/nginx-template
- [x] examples/systemd/tnl-admin.service
- [x] README.md (root)
- [x] CHANGELOG.md
- [x] LICENSE
- [x] VERSION
- [x] .gitignore
- [x] requirements.txt

## Commands Summary

```bash
# Complete workflow
cd "/Users/daniel/Developer/IONOS Server"

# Rename root README
mv README-root.md README.md

# Initialize and commit
git init
git add .
git commit -m "Initial commit - v1.0.0"
git tag -a v1.0.0 -m "Release v1.0.0"

# Connect to GitHub (replace USERNAME)
git remote add origin git@github.com:USERNAME/tunnel-gateway.git
git push -u origin main
git push --tags

# Then create release on GitHub web interface
```

## After Publishing

Update the documentation with:
1. Actual GitHub repository URL
2. Links to releases
3. Installation commands with real URLs
4. Badge URLs (optional - see below)

## Optional: Add Badges to README

```markdown
[![GitHub release](https://img.shields.io/github/v/release/USERNAME/tunnel-gateway)](https://github.com/USERNAME/tunnel-gateway/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
```

---

Ready to publish! 🚀
