# GitHub Actions + Docker Deployment Setup

This guide explains how to deploy your Next.js app to Digital Ocean using GitHub Actions and Docker.

## How It Works

```
┌─────────────────┐         ┌──────────────────┐         ┌────────────────┐
│  Push to GitHub │────────>│ GitHub Actions   │────────>│ Digital Ocean  │
│  (main branch)  │         │ (7GB RAM)        │         │ ($5/mo - 512MB)│
└─────────────────┘         │ 1. Build Docker  │         │ Runs container │
                            │ 2. Push to DO    │         └────────────────┘
                            │ 3. Deploy        │
                            └──────────────────┘
```

**Benefits:**
- Build runs on GitHub (7GB RAM) - no more memory errors
- Digital Ocean only runs the pre-built container (512MB is enough)
- Faster deployments
- Free CI/CD

## Setup Steps

### 1. Create Digital Ocean Container Registry

1. Go to https://cloud.digitalocean.com/registry
2. Click "Create" and choose a name (e.g., `masumi-docs`)
3. Save the registry name for later

### 2. Create Digital Ocean App (if not already created)

1. Go to https://cloud.digitalocean.com/apps
2. Click "Create App"
3. Choose "Docker Hub" or "DigitalOcean Container Registry"
4. Select your registry and the image name: `masumi-docs`
5. Choose $5/mo plan (512 MB RAM, 1 vCPU)
6. Click "Create Resources"
7. Save the App ID from the URL (format: `apps/{APP_ID}/...`)

### 3. Get Digital Ocean API Token

1. Go to https://cloud.digitalocean.com/account/api/tokens
2. Click "Generate New Token"
3. Name: `GitHub Actions Deploy`
4. Scopes: Select "Write" access
5. Click "Generate Token"
6. **IMPORTANT:** Copy the token immediately (you won't see it again)

### 4. Configure GitHub Secrets

Go to your GitHub repository → Settings → Secrets and variables → Actions → New repository secret

Add these secrets:

| Secret Name | Value | Where to Find |
|-------------|-------|---------------|
| `DO_API_TOKEN` | Your DO API token | From step 3 |
| `DO_REGISTRY_NAME` | Your registry name | From step 1 (e.g., `masumi-docs`) |
| `DO_APP_ID` | Your app ID | From step 2 (from URL) |

**How to add secrets:**
1. Click "New repository secret"
2. Enter the name exactly as shown
3. Paste the value
4. Click "Add secret"
5. Repeat for all three secrets

### 5. Update Digital Ocean App Configuration

Your app needs to pull from the container registry:

1. Go to your App in Digital Ocean
2. Go to Settings → App Spec
3. Update the image to point to your registry:
   ```yaml
   name: masumi-docs
   services:
   - name: web
     image:
       registry_type: DOCR
       repository: masumi-docs
       tag: latest
     http_port: 3000
   ```
4. Save

### 6. Test the Deployment

1. Push to main branch or manually trigger workflow:
   - Go to Actions tab in GitHub
   - Select "Build and Deploy to Digital Ocean"
   - Click "Run workflow"

2. Monitor the build:
   - Watch the Actions tab for progress
   - Build takes ~5-10 minutes

3. Check deployment:
   - Visit your Digital Ocean app URL
   - Logs: Digital Ocean → Apps → your app → Runtime Logs

## What Changed

### Files Added:
- `.dockerignore` - Excludes unnecessary files from Docker build
- `Dockerfile` - Multi-stage build for optimized production image
- `.github/workflows/deploy.yml` - CI/CD pipeline

### Files Modified:
- `next.config.mjs` - Added `output: 'standalone'` for Docker deployment

## Troubleshooting

### Build fails in GitHub Actions
- Check the Actions logs for specific errors
- Verify all secrets are set correctly
- Ensure DO registry exists

### Deployment fails
- Check Digital Ocean app logs
- Verify the App ID is correct
- Ensure the registry image was pushed successfully

### App won't start
- Check runtime logs in Digital Ocean
- Verify environment variables are set if needed
- Check if port 3000 is exposed correctly

## Rollback

If something goes wrong:
1. Go to Digital Ocean → Apps → your app
2. Click "Deployments" tab
3. Select a previous working deployment
4. Click "Rollback"

## Cost Breakdown

- **GitHub Actions:** Free (2,000 minutes/month for public repos)
- **Digital Ocean Container Registry:** Free (1 repository included)
- **Digital Ocean App:** $5/month (your current plan)

**Total:** Still $5/month, but builds work now!

## Next Steps

Once this is working, you can:
- Add staging environments
- Implement blue-green deployments
- Add health checks
- Set up custom domains
- Add environment-specific configs
