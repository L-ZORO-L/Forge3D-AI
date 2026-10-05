# Forge3D AI - Web Deployment Guide

This guide explains how to deploy Forge3D to a free hosting service.

## Option 1: Render.com (Recommended)

### Backend Setup

1. Create a Render account at https://render.com

2. Create a new Web Service:
   - Connect your GitHub repo (or upload manually)
   - Select Python 3.10+
   - Build command: `pip install -r backend/requirements.txt`
   - Start command: `uvicorn backend.app:app --host 0.0.0.0 --port 10000`
   - Environment variables:
     ```
     TRIPOSR_PATH=/opt/render/project/src/engines/TripoSR
     ```

3. Deploy the backend

### Frontend Setup

1. In Render, create a new Static Site:
   - Connect your repo
   - Build command: `cd frontend && npm install && npm run build`
   - Publish directory: `frontend/dist`

2. Update frontend API URL in `frontend/src/App.jsx`:
   ```javascript
   const BACKEND_URL = 'https://YOUR_RENDER_BACKEND_URL';
   ```

3. Deploy the frontend

## Option 2: Railway.app

### Backend Setup

1. Create account at https://railway.app

2. Create new project:
   - Connect GitHub repo
   - Select Python environment
   - Add environment variables (same as above)
   - Railway will auto-detect and run `python backend/app.py`

3. Get your backend URL from Railway dashboard

### Frontend Setup

1. Update API URL in frontend code
2. Deploy frontend to Vercel (see below)

## Option 3: Fly.io

### Backend Setup

1. Install Fly CLI: https://fly.io/docs/getting-started/installing-flyctl/

2. Create app:
   ```bash
   fly auth login
   fly launch --name forge3d-backend
   ```

3. Update `fly.toml`:
   ```toml
   [env]
   TRIPOSR_PATH = "/app/engines/TripoSR"
   ```

4. Deploy:
   ```bash
   fly deploy
   ```

### Frontend Setup

Deploy to Vercel (see below)

---

## Frontend Deployment on Vercel (All Options)

1. Create Vercel account at https://vercel.com

2. Connect your GitHub repo

3. Configure:
   - Framework: Vite
   - Root directory: `frontend`
   - Build command: `npm run build`
   - Output directory: `dist`

4. Environment variables:
   ```
   VITE_BACKEND_URL=https://YOUR_BACKEND_URL
   ```

5. Deploy

---

## Important Notes

### TripoSR on Hosted Servers

The official TripoSR model is large (~2GB). When deploying:

1. **Download model weights on server startup**
2. **Use CPU inference** (most free tiers don't have GPU)
3. **Expect slow generation** (5-15 minutes per image)
4. **Set request timeouts** to 15+ minutes

### Updated Backend for Web

Replace `backend/app.py` with `backend/app_web.py`:

```python
import asyncio
import os
from pathlib import Path

# Download model weights on startup
async def startup_event():
    engine = TripoSRGenerator()
    # This will download ~2GB model on first run
    engine.system_info()

app.add_event_handler("startup", startup_event)
```

### Cost Considerations

- **Render.com free tier**: 750 hours/month (CPU only, slow)
- **Railway.app free tier**: $5 credit/month
- **Fly.io free tier**: 3 shared-cpu-1x 256MB VMs
- **Vercel free tier**: Unlimited frontend deployments

**Recommendation**: Use free tier only for testing. Production needs:
- GPU inference (faster)
- More memory (TripoSR needs 8GB+)
- Longer request timeouts

---

## Local vs Web Trade-offs

| Aspect | Local | Web |
|--------|-------|-----|
| Speed | Very fast (GPU) | Slow (CPU only) |
| Privacy | 100% private | Data on server |
| Cost | Free (your PC) | Free tier or paid |
| Setup | Easy (local) | Moderate |
| Availability | Always on your PC | Depends on hosting |

---

## Quick Deploy Script

Save as `deploy.sh`:

```bash
#!/bin/bash

# Set your backend URL
BACKEND_URL="https://YOUR_BACKEND_URL"

# Update frontend env
sed -i "s|const BACKEND_URL = '.*'|const BACKEND_URL = '${BACKEND_URL}'|" frontend/src/App.jsx

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Deploy instructions will appear
echo "Frontend built in frontend/dist"
echo "Deploy to Vercel now"
```

---

## Testing Web Version Locally

Before deploying, test locally with separate processes:

Terminal 1 (Backend):
```bash
python backend/app.py
```

Terminal 2 (Frontend):
```bash
cd frontend
npm run dev
```

Frontend will run at `http://localhost:4173`
Backend at `http://127.0.0.1:8000`

Update frontend to use `http://127.0.0.1:8000` for testing.

---

## Next Steps

1. Choose a hosting platform
2. Follow its setup guide above
3. Update `frontend/src/App.jsx` with your backend URL
4. Deploy
5. Test with an image upload

For help with any platform, ask me for platform-specific configuration.
