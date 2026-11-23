# Deploying Nod on Render FREE TIER

## ⚠️ CRITICAL LIMITATIONS

| Resource | Free Tier Limit | Impact on Nod |
|----------|-----------------|---------------|
| **RAM** | 512 MB | Very tight - Node.js + PostgreSQL client may OOM |
| **PostgreSQL** | 90 days only | **Database DELETED after 90 days** - no free persistent DB |
| **Persistent Disk** | ❌ Not available | All data lost on restart/deploy/spin-down |
| **Spin-down** | 15 min inactivity | 30-60s cold start; wakes on first request |
| **Build Time** | 15 minutes | May timeout on first build |
| **Custom Domain** | ❌ Not available | Only `*.onrender.com` subdomain |

**This is for TESTING/EVALUATION ONLY. Not suitable for production.**

---

## What Works on Free Tier

✅ Basic chat/assistant  
✅ Model connections (OpenAI, Anthropic, etc.)  
✅ PostgreSQL (for 90 days)  
✅ File uploads (stored in ephemeral memory - lost on restart)  

❌ Browser automation (needs Docker-in-Docker)  
❌ Code execution workspace (needs Docker-in-Docker)  
❌ Persistent data across restarts  
❌ Telegram/WhatsApp webhooks (unreliable due to spin-down)  

---

## Quick Deploy (3 Steps)

### 1. Fork & Push to GitHub

```bash
# Clone your fork (not the original - you need write access)
git clone https://github.com/YOUR_USERNAME/nod.git
cd nod

# Copy the free-tier deployment files to your repo
# (Dockerfile.render.free, render.free.yaml, FREE_DEPLOYMENT.md)

git add .
git commit -m "Add Render free tier deployment"
git push origin main
```

### 2. Deploy via Blueprint

1. Go to [Render Dashboard](https://dashboard.render.com)
2. **New +** → **Blueprint**
3. Connect your GitHub repo
4. Render detects `render.free.yaml` → **Apply**
5. Wait for build (may take 10-15 min on free tier)

### 3. Add API Keys & Test

1. Go to **nod-free** service → **Environment**
2. Add your model API key:
   ```
   OPENAI_API_KEY=sk-your-key-here
   # OR
   ANTHROPIC_API_KEY=sk-ant-...
   # OR
   GEMINI_API_KEY=...
   ```
3. **Save Changes** → triggers redeploy
4. After deploy, copy your URL: `https://nod-free-xyz.onrender.com`
5. Update `NOD_PUBLIC_URL` env var with this exact URL
6. Open URL, enter `NOD_AUTH_TOKEN` (from Environment tab)

---

## Free Tier Gotchas

### Database Expires in 90 Days
- Free PostgreSQL is **trial only** - auto-deleted after 90 days
- No way to extend on free tier
- **Export your data** before expiry if needed

### No Persistent Storage
- `/var/lib/nod` is **ephemeral** (in container memory)
- All uploads, configs, conversation history **lost on**:
  - New deploy
  - Service restart
  - Spin-down/wake-up
  - Any container recreation

### Spin-down Behavior
- Service sleeps after 15 min inactivity
- First request waits 30-60s for cold start
- **Telegram/WhatsApp webhooks will fail** during sleep
- Use polling mode for Telegram if testing

### Memory Constraints (512MB)
- Set `NODE_OPTIONS=--max-old-space-size=384` (included)
- Avoid large file uploads
- Limit concurrent conversations
- Monitor logs for OOM kills

---

## Alternative: Truly Free Options

### 1. Local Development (Recommended for Testing)
```bash
# Clone and run locally - no limits!
git clone https://github.com/n8n-io/nod.git
cd nod
pnpm install
pnpm run dev
# Access at http://localhost:4310
```
**Benefits**: Full features (browser, workspace), persistent data, no time limits

### 2. GitHub Codespaces (Free Monthly Hours)
- 60 hours/month free on personal accounts
- Full VS Code in browser
- Run `pnpm run dev` in Codespace terminal
- Port forward 4310

### 3. Oracle Cloud Free Tier (Always Free)
- 2 AMD Compute VMs (1/8 OCPU, 1GB RAM) - **forever free**
- Or 4 ARM Ampere VMs (24GB RAM total!)
- Install Docker, deploy via [self-host guide](https://github.com/n8n-io/nod/blob/main/docs/src/content/docs/deploy/self-host.mdx)
- **Full features including browser/workspace**

### 4. Fly.io (Free Allowance)
- $5/month credit = ~3 small VMs free
- Persistent volumes included
- Good for small deployments

### 5. Railway / Render / Koyeb Trials
- Most offer $5-10 trial credit
- Good for short-term testing

---

## Minimal .env for Free Tier

```bash
# Generate these once and save securely
NOD_AUTH_TOKEN=$(openssl rand -base64 32)
NOD_VAULT_KEY=$(openssl rand -base64 32)

# From Render PostgreSQL dashboard (after deploy)
DATABASE_URL=postgresql://...

# Your Render URL (update AFTER first deploy)
NOD_PUBLIC_URL=https://nod-free-abc123.onrender.com

# Required model provider
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...

# Memory optimization
NODE_OPTIONS=--max-old-space-size=384
```

---

## Monitoring Free Tier Health

Check Render dashboard for:
- **Memory usage** - should stay under 450MB
- **Restart count** - frequent restarts = OOM kills
- **Build logs** - watch for 15-min timeout
- **Database days remaining** - PostgreSQL trial countdown

---

## Upgrade Path

When free tier limits bite:

| Need | Solution | Cost |
|------|----------|------|
| Persistent DB | Render PostgreSQL Starter | $7/mo |
| Persistent Disk | Render Disk 10GB | Included in Standard |
| More RAM / No spin-down | Render Standard | $25/mo |
| Browser/Workspace | VPS (Hetzner/DO) | $4-24/mo |
| Always free, full features | Oracle Cloud ARM | $0 (forever) |

---

## Summary

| Scenario | Recommendation |
|----------|----------------|
| Quick test (1-2 weeks) | Render Free Tier ✅ |
| Extended testing | Local / Codespaces / Oracle Cloud |
| Personal use, full features | Oracle Cloud Free Tier (ARM) |
| Production | Paid Render / VPS |

**For anything beyond a quick test, use local development or Oracle Cloud Free Tier.**