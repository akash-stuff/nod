# Deploying Nod on Render

This guide explains how to deploy the Nod server on Render.

## ⚠️ Important Limitations

**Browser and Workspace features require Docker-in-Docker**, which Render does not support. This means:

| Feature | Status on Render |
|---------|------------------|
| Main chat/assistant | ✅ Works |
| Model connections (OpenAI, Anthropic, etc.) | ✅ Works |
| PostgreSQL database | ✅ Works (managed) |
| File uploads/attachments | ✅ Works |
| **Browser automation** | ❌ **Disabled** |
| **Code execution workspace** | ❌ **Disabled** |
| Skills requiring browser/workspace | ❌ **Limited** |
| Telegram/WhatsApp webhooks | ✅ Works (with public URL) |

If you need browser/workspace features, consider deploying on a VPS (DigitalOcean, Hetzner, AWS, etc.) using the [official self-host guide](https://github.com/n8n-io/nod/blob/main/docs/src/content/docs/deploy/self-host.mdx).

---

## Prerequisites

1. **GitHub account** - Repository must be on GitHub
2. **Render account** - Sign up at [render.com](https://render.com)
3. **Model API key** - At least one: OpenAI, Anthropic, Gemini, etc.

---

## Quick Deploy (Using render.yaml)

### 1. Push to GitHub

```bash
# Clone the repository
git clone https://github.com/n8n-io/nod.git
cd nod

# Add the Render deployment files (created in this guide)
# Copy Dockerfile.render, render.yaml, and RENDER_DEPLOYMENT.md to the repo

# Commit and push
git add .
git commit -m "Add Render deployment configuration"
git push origin main
```

### 2. Deploy on Render

1. Go to [Render Dashboard](https://dashboard.render.com)
2. Click **"New +"** → **"Blueprint"**
3. Connect your GitHub repository
4. Render will detect `render.yaml` and show the services to create
5. Click **"Apply"** to deploy

### 3. Configure Environment Variables

After deployment starts, go to the **nod-server** service → **Environment** tab and add:

| Variable | Value | Required |
|----------|-------|----------|
| `OPENAI_API_KEY` | Your OpenAI API key | At least one model provider |
| `ANTHROPIC_API_KEY` | Your Anthropic API key | At least one model provider |
| `GEMINI_API_KEY` | Your Google Gemini API key | At least one model provider |
| `NOD_PUBLIC_URL` | `https://your-service.onrender.com` | **Yes - Update after deploy!** |

### 4. Get Your Public URL

After deployment completes:
1. Go to the **nod-server** service
2. Copy the URL (e.g., `https://nod-server-abcd.onrender.com`)
3. Update `NOD_PUBLIC_URL` environment variable with this exact URL
4. Trigger a new deploy (Render will auto-redeploy on env var change)

### 5. Access Nod

1. Open your `NOD_PUBLIC_URL` in browser
2. Enter the `NOD_AUTH_TOKEN` (shown in Render environment variables)
3. Complete onboarding and connect a model

---

## Manual Deploy (Without render.yaml)

If you prefer manual setup:

### 1. Create PostgreSQL Database

1. Dashboard → **New +** → **PostgreSQL**
2. Name: `nod-db`
3. Database: `nod`
4. User: `nod`
5. Plan: `Starter` (or higher)
6. Region: Same as your web service
7. Create database

### 2. Create Web Service

1. Dashboard → **New +** → **Web Service**
2. Connect GitHub repository
3. Settings:
   - **Name**: `nod-server`
   - **Runtime**: `Docker`
   - **Dockerfile Path**: `./Dockerfile.render`
   - **Docker Context**: `.`
   - **Plan**: `Standard` (minimum for browser/workspace memory requirements)
   - **Region**: Same as database

### 3. Add Environment Variables

In the web service **Environment** tab:

```
NOD_AUTH_TOKEN=<generate-secure-random-string>
NOD_VAULT_KEY=<generate-secure-random-string>
DATABASE_URL=<copy-from-postgresql-dashboard>
NOD_HOST=0.0.0.0
PORT=4310
NOD_DATA_ROOT=/var/lib/nod
NOD_ENVIRONMENT=production
NOD_PUBLIC_URL=https://your-service.onrender.com
NOD_DESKTOP=false
LLM_PROVIDER=openai
OPENAI_API_KEY=your-key-here
```

### 4. Add Persistent Disk

1. Service → **Disks** → **Add Disk**
2. Name: `nod-data`
3. Mount Path: `/var/lib/nod`
4. Size: `10 GB` (minimum)

### 5. Deploy

Click **"Create Web Service"** and wait for deployment.

---

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `NOD_AUTH_TOKEN` | Secret token for authentication (generate with `openssl rand -base64 32`) | `abc123...` |
| `NOD_VAULT_KEY` | Encryption key for credentials (generate with `openssl rand -base64 32`) | `xyz789...` |
| `DATABASE_URL` | PostgreSQL connection string (from Render PostgreSQL) | `postgresql://...` |
| `NOD_PUBLIC_URL` | Your public HTTPS URL (UPDATE AFTER DEPLOY) | `https://nod-xyz.onrender.com` |
| `LLM_PROVIDER` | Model provider: `openai`, `anthropic`, `gemini`, etc. | `openai` |
| `OPENAI_API_KEY` | Your OpenAI API key (or other provider key) | `sk-...` |

---

## Optional Environment Variables

```bash
# Model Configuration
LLM_MODEL=gpt-4o                    # Specific model (optional)
ANTHROPIC_API_KEY=sk-ant-...        # For Anthropic
GEMINI_API_KEY=...                  # For Google Gemini
OPENROUTER_API_KEY=...              # For OpenRouter

# Integrations (Composio)
NOD_COMPOSIO_PROXY_URL=https://composio.n8n.cloud

# Analytics
NOD_ANALYTICS_ENABLED=true
NOD_ANALYTICS_ID=<auto-generated>

# Telegram (requires public HTTPS URL)
TELEGRAM_ENABLED=true
TELEGRAM_MODE=polling
TELEGRAM_BOT_TOKEN=your-bot-token

# WhatsApp (requires public HTTPS URL)
WHATSAPP_ENABLED=true
WHATSAPP_MODE=cloud
WHATSAPP_ACCESS_TOKEN=...
WHATSAPP_PHONE_NUMBER_ID=...
WHATSAPP_VERIFY_TOKEN=...
WHATSAPP_ALLOWED_NUMBER=...
```

---

## Generating Secure Keys

```bash
# Generate NOD_AUTH_TOKEN
openssl rand -base64 32

# Generate NOD_VAULT_KEY
openssl rand -base64 32
```

**Save these securely!** They cannot be recovered.

---

## Updating Nod

1. Pull latest changes:
   ```bash
   git pull origin main
   ```
2. Push to GitHub - Render auto-deploys on push
3. Or manually trigger deploy in Render dashboard

---

## Troubleshooting

### Service won't start
- Check **Logs** tab in Render dashboard
- Verify all required environment variables are set
- Ensure `DATABASE_URL` is correct

### Database connection failed
- Verify PostgreSQL is in same region
- Check `DATABASE_URL` format
- Ensure database is running

### "Invalid NOD_PUBLIC_URL"
- Must be exact HTTPS URL (e.g., `https://nod-abc.onrender.com`)
- No trailing slash
- Must match the URL you access in browser

### Out of memory
- Upgrade to higher plan (Standard → Pro)
- Browser/workspace features disabled but still allocate memory

### Health check failing
- Server takes ~60s to start first time
- Check logs for actual errors
- Ensure PORT=4310 matches EXPOSE in Dockerfile

---

## Cost Estimate (USD/month)

| Resource | Plan | Cost |
|----------|------|------|
| Web Service | Standard (2 CPU, 4GB RAM) | $25 |
| PostgreSQL | Starter (1GB RAM, 10GB) | $7 |
| Disk (10GB) | Included in service | $0 |
| **Total** | | **~$32/month** |

*Note: Browser/workspace would need Pro plan ($85/month) but don't work on Render anyway.*

---

## Alternative: Deploy on VPS

For full features (browser, workspace, etc.), use a VPS:

- **DigitalOcean Droplet** - $24/month (4GB RAM, 2 CPU, 80GB SSD)
- **Hetzner Cloud** - ~€15/month (4GB RAM, 2 CPU, 80GB SSD)
- **AWS EC2** - t3.medium ~$30/month

Follow the [official self-host guide](https://github.com/n8n-io/nod/blob/main/docs/src/content/docs/deploy/self-host.mdx) for VPS deployment.

---

## Support

- **Documentation**: https://github.com/n8n-io/nod/tree/main/docs
- **Issues**: https://github.com/n8n-io/nod/issues
- **Render Docs**: https://render.com/docs