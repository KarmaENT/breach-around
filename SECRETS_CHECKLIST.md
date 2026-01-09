# Secrets Configuration Checklist

This document helps verify that all required secrets are properly configured for the automated sync and release workflows.

## 🔐 Required Secrets

### Public Repository (`Fused-Gaming/breach-around`)

| Secret Name | Required For | Status |
|-------------|--------------|--------|
| `PRIVATE_REPO_ACCESS_TOKEN` | Syncing code from private repo | ⚠️ **NEEDS SETUP** |
| `PYPI_API_TOKEN` | PyPI publishing (if not using OIDC) | ℹ️ Optional (OIDC preferred) |

### Private Repository (`Fused-Gaming/breach-checker`)

| Secret Name | Required For | Status |
|-------------|--------------|--------|
| `PUBLIC_REPO_DISPATCH_TOKEN` | Triggering public repo sync | ⚠️ **NEEDS SETUP** |

---

## ✅ How to Verify Secrets Are Configured

### Method 1: Check via GitHub Web UI

**For Public Repo:**
1. Go to https://github.com/Fused-Gaming/breach-around/settings/secrets/actions
2. Look for `PRIVATE_REPO_ACCESS_TOKEN` in the list
3. You won't see the value, but you'll see the name if it exists

**For Private Repo:**
1. Go to https://github.com/Fused-Gaming/breach-checker/settings/secrets/actions
2. Look for `PUBLIC_REPO_DISPATCH_TOKEN` in the list

### Method 2: Test the Workflows

**Test Sync Workflow (No Secrets Required):**
```bash
# This test workflow doesn't need secrets
1. Go to Actions → Test Sync Workflow
2. Click "Run workflow"
3. Enter tag: v0.0.1-test
4. Click "Run workflow"
5. Should complete successfully (green checkmark)
```

**Test Full Sync Workflow (Requires PRIVATE_REPO_ACCESS_TOKEN):**
```bash
1. Go to Actions → Sync from Private Repository
2. Click "Run workflow"
3. Enter tag: v0.0.1-test
4. Click "Run workflow"
5. Watch for errors:
   - "Input required and not supplied: token" = SECRET MISSING ❌
   - Completes successfully = SECRET CONFIGURED ✅
```

---

## 📋 Step-by-Step Secret Creation

### Step 1: Create Personal Access Token (PAT)

1. Go to https://github.com/settings/tokens
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. Configure:
   - **Name**: `Breach Around Automation`
   - **Expiration**: 1 year (or your preference)
   - **Scopes**: Check these:
     - ✅ `repo` - Full control of private repositories
     - ✅ `workflow` - Update GitHub Action workflows
4. Click **"Generate token"**
5. **⚠️ COPY THE TOKEN IMMEDIATELY** (starts with `ghp_...`)

### Step 2: Add Secret to Public Repository

1. Go to https://github.com/Fused-Gaming/breach-around/settings/secrets/actions
2. Click **"New repository secret"**
3. Enter:
   - **Name**: `PRIVATE_REPO_ACCESS_TOKEN`
   - **Secret**: Paste the PAT from Step 1
4. Click **"Add secret"**

### Step 3: Add Secret to Private Repository

1. Go to https://github.com/Fused-Gaming/breach-checker/settings/secrets/actions
2. Click **"New repository secret"**
3. Enter:
   - **Name**: `PUBLIC_REPO_DISPATCH_TOKEN`
   - **Secret**: Paste the same PAT (or create a new one)
4. Click **"Add secret"**

---

## 🧪 Testing After Setup

### Test 1: Manual Sync from Public Repo

```bash
# This tests if PRIVATE_REPO_ACCESS_TOKEN works
1. Go to https://github.com/Fused-Gaming/breach-around/actions
2. Select "Sync from Private Repository"
3. Click "Run workflow"
4. Fill in:
   - Branch: claude/public-repo-setup-Sb6tf
   - Tag name: v0.0.1-test
5. Click "Run workflow"

Expected Result: ✅ Workflow completes successfully
Error if secret missing: "Input required and not supplied: token"
```

### Test 2: Automatic Trigger from Private Repo

```bash
# This tests if PUBLIC_REPO_DISPATCH_TOKEN works
1. In breach-checker repo, ensure the trigger workflow is installed
2. Create a test release:
   - Go to Releases → Draft a new release
   - Tag: v0.0.1-test
   - Title: Test Release
   - Click "Publish release"
3. Check Actions in BOTH repos:
   - Private: "Trigger Public Repo Sync" should run
   - Public: "Sync from Private Repository" should run

Expected Result: ✅ Both workflows complete successfully
```

### Test 3: PyPI Publishing (Optional)

```bash
# Only needed if you want PyPI publishing
Option A: OIDC (Recommended - No secrets!)
1. Go to https://pypi.org/manage/project/breach-around/
2. Navigate to "Publishing" tab
3. Add publisher:
   - Repository: breach-around
   - Owner: Fused-Gaming
   - Workflow: release.yml
4. Test by creating a real tag (v0.1.0) - will auto-publish

Option B: API Token
1. Get API token from https://pypi.org/manage/account/token/
2. Add secret PYPI_API_TOKEN to public repo
3. Modify release.yml to use the token
```

---

## 🔍 Current Status Check

### Quick Status Commands

Run these in the repository to check configuration:

```bash
# Check if workflows exist
ls -la .github/workflows/
# Should show: release.yml, sync-from-private.yml, test-sync.yml

# Check workflow syntax
cat .github/workflows/sync-from-private.yml | grep "PRIVATE_REPO_ACCESS_TOKEN"
# Should show the secret reference

# Check recent workflow runs
# Visit: https://github.com/Fused-Gaming/breach-around/actions
```

---

## ❌ Common Errors and Solutions

### Error: "Input required and not supplied: token"

**Problem**: `PRIVATE_REPO_ACCESS_TOKEN` is not configured

**Solution**:
1. Create PAT with `repo` scope
2. Add as secret `PRIVATE_REPO_ACCESS_TOKEN` in public repo
3. Retry the workflow

### Error: "Resource not accessible by integration"

**Problem**: PAT doesn't have correct scopes

**Solution**:
1. Go to https://github.com/settings/tokens
2. Edit the PAT
3. Ensure `repo` and `workflow` scopes are checked
4. Update the secret with regenerated token

### Error: "API rate limit exceeded"

**Problem**: Too many API calls without authentication

**Solution**:
1. Verify the secret is set correctly
2. Wait a few minutes and retry
3. Check PAT isn't expired

### Error: "Repository not found"

**Problem**: PAT doesn't have access to private repo

**Solution**:
1. Verify the PAT was created by user with access to breach-checker
2. Ensure the token has `repo` scope
3. Check the private repo name is correct in workflow file

---

## 📊 Secret Audit Log

Keep track of when secrets were created and updated:

| Secret Name | Created Date | Last Updated | Expires | Notes |
|-------------|--------------|--------------|---------|-------|
| `PRIVATE_REPO_ACCESS_TOKEN` | YYYY-MM-DD | YYYY-MM-DD | YYYY-MM-DD | [Status] |
| `PUBLIC_REPO_DISPATCH_TOKEN` | YYYY-MM-DD | YYYY-MM-DD | YYYY-MM-DD | [Status] |
| `PYPI_API_TOKEN` | N/A | N/A | N/A | Using OIDC instead |

---

## ⏰ Maintenance Schedule

- **Every 90 days**: Rotate PATs for security
- **Before expiration**: Renew PATs (check expiration dates)
- **After team changes**: Review and rotate if team members leave
- **Quarterly**: Audit secret usage in workflow runs

---

## 🎯 Success Criteria

Your secrets are properly configured when:

- [x] `PRIVATE_REPO_ACCESS_TOKEN` exists in public repo secrets
- [x] `PUBLIC_REPO_DISPATCH_TOKEN` exists in private repo secrets
- [x] Test sync workflow runs successfully
- [x] Full sync workflow completes without "token" errors
- [x] PyPI publishing configured (OIDC or token)

---

## 📞 Help

If you're still having issues:

1. Double-check secret names match exactly (case-sensitive)
2. Verify PAT scopes include `repo` and `workflow`
3. Ensure PAT hasn't expired
4. Check workflow logs for specific error messages
5. See SETUP.md for detailed troubleshooting

---

**Last Updated**: 2026-01-09
