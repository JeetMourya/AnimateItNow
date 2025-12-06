# Setting Up GitHub Secrets for Token Injection

This project uses GitHub Actions to inject the GitHub Personal Access Token from repository secrets into a `config.js` file **during deployment only**. The token is **never committed** to the repository.

## ⚠️ Important Security Note

**The token will still be visible in the deployed website** since it's in client-side JavaScript. For truly secure token usage, consider:
- Using a backend/proxy server
- Using serverless functions
- Using environment variables in your deployment platform

## Setup Instructions

### Step 1: Add GitHub Secret

1. Go to your repository on GitHub
2. Click on **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `GH_PAT` (⚠️ **Important**: Cannot start with `GITHUB_`)
5. Value: Paste your GitHub Personal Access Token
6. Click **Add secret**

### Step 2: Update Deployment Workflow

The workflow generates `config.js` during deployment. You need to add your deployment steps to the workflow file (`.github/workflows/inject-token.yml`).

### Step 3: How It Works

- ✅ Token stored in GitHub Secrets (secure)
- ✅ `config.js` generated during deployment (not committed)
- ✅ Token injected into deployed files only
- ⚠️ Token still visible in deployed website (client-side limitation)

### For Different Deployment Platforms

#### GitHub Pages
Add this to your workflow:
```yaml
- name: Deploy to GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./
```

#### Netlify
1. Add `GH_PAT` as an environment variable in Netlify dashboard
2. In Netlify build settings, add:
```bash
echo "window.GITHUB_TOKEN = '$GH_PAT';" > config.js
```

#### Vercel
1. Add `GH_PAT` as an environment variable in Vercel dashboard
2. In `vercel.json`, add a build command:
```json
{
  "buildCommand": "echo \"window.GITHUB_TOKEN = '$GH_PAT';\" > config.js"
}
```

### Current Implementation

The code now:
- Loads `config.js` in both `contributors.html` and `leaderboard.html`
- Uses `window.GITHUB_TOKEN` if available
- Falls back gracefully if token is not set

### Testing Locally

For local development, create a `config.js` file manually:
```javascript
window.GITHUB_TOKEN = 'your-token-here';
```

**Remember**: Add `config.js` to `.gitignore` so you don't accidentally commit it!

### ⚠️ Secret Naming Rules

GitHub does **NOT** allow secret names that start with `GITHUB_`. Use names like:
- ✅ `GH_PAT`
- ✅ `GITHUB_TOKEN_SECRET`
- ✅ `ANIMATEITNOW_GITHUB_TOKEN`
- ❌ `GITHUB_PAT` (not allowed)
- ❌ `GITHUB_TOKEN` (not allowed)
