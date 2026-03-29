# Railway Deployment Guide for MESTRE

**Document Date:** 2026-03-28  
**Status:** Research & Documentation (No deployment yet)  
**Target App:** MESTRE Streamlit Spec-Text Takeoff Engine

---

## 1. Railway Account Setup

### Free Tier & Pricing (2026)

**Free Trial:**
- $5 credits for 30 days
- After trial: $1/month to keep the project alive (minimum)
- Compute: Up to 1 vCPU / 0.5 GB RAM per service
- Storage: 0.5 GB persistent volume storage
- ⚠️ Good for prototyping, NOT production

**Hobby Plan ($5/month):**
- Includes $5 monthly usage credit
- Up to 48 vCPU / 48 GB RAM per service
- Up to 5 GB persistent volume storage
- Single developer workspace
- ✅ **Best for a small app with 10-50 users/month**
- Estimated cost: **$5-20/month for MESTRE** (depends on usage)

**Pro Plan ($20/month):**
- Includes $20 monthly usage credit
- Up to 1,000 vCPU / 1 TB RAM per service
- Up to 1 TB persistent storage
- Unlimited workspace seats
- Recommended when you have paying customers

**Pricing Formula (if exceeding monthly credit):**
- Memory: $0.00000386 per GB/sec
- CPU: $0.00000772 per vCPU/sec
- Volume storage: $0.00000006 per GB/sec
- Egress: $0.05 per GB

### How to Sign Up
1. Go to **railway.com**
2. Click "Start Building" → Sign up with GitHub/email
3. GitHub account required for repo-based deployments
4. Add payment method (triggers $1/month commitment after trial)
5. Create first project from dashboard

---

## 2. Files Needed for Railway Deployment

Create these 4 files in `/Users/vanshsmac/MESTRE/`:

### 2.1 **Procfile** (Railway runtime configuration)

```
web: streamlit run app.py --server.port=$PORT --server.address=0.0.0.0 --server.headless=true --logger.level=info
```

**Why this works:**
- `streamlit run app.py` — starts your app
- `--server.port=$PORT` — listens on the port Railway assigns (usually 5000)
- `--server.address=0.0.0.0` — listens on all network interfaces (required for web access)
- `--server.headless=true` — no local browser window
- `--logger.level=info` — reduced logging noise

### 2.2 **runtime.txt** (Python version)

```
python-3.11
```

Ensures consistent Python version between local and Railway.

### 2.3 **requirements.txt** (Complete dependencies)

```
# Core
streamlit>=1.30.0
pymupdf>=1.23.0
anthropic>=0.40.0

# Data processing
openpyxl>=3.1.0
pandas>=2.0.0

# Vector database (local fallback)
chromadb>=0.4.0
sentence-transformers>=2.0.0

# Environment variables
python-dotenv>=1.0.0
```

**Status:** Current requirements.txt is incomplete. Update to the above.

### 2.4 **.gitignore** (Don't commit these to Railway)

```
# Sensitive
.env
*.json

# Cache
__pycache__/
*.pyc
.streamlit/

# Local data (will be rebuilt on server)
chromadb-store/
market-data/

# Large files
*.xlsx
*.pdf
```

**Why:** Keep secrets out of GitHub, reduce deployment size.

---

## 3. Environment Variables (Set in Railway Dashboard)

In Railway project canvas → Service settings → Variables:

```
ANTHROPIC_API_KEY=sk-proj-xxxxxxxxxx
MESTRE_ENV=production
LOG_LEVEL=info
```

### Required Variables

| Variable | Value | Required? | Where |
|----------|-------|-----------|-------|
| `ANTHROPIC_API_KEY` | Your Claude API key | ✅ YES | Railway Variables |
| `MESTRE_ENV` | `production` | Optional | App auto-detects |
| `LOG_LEVEL` | `info` or `debug` | Optional | Logging control |

### ⚠️ DO NOT hardcode secrets in code or requirements.txt

---

## 4. ChromaDB Deployment Issue — THE CRITICAL PROBLEM

**The core challenge:**  
ChromaDB is a **local vector database**. It stores embeddings on disk at `/Users/vanshsmac/MESTRE/chromadb-store/` (currently **21 MB**).

### Option A: Use Chroma Cloud (Recommended for production)

**What:** Managed cloud vector database hosted by Chroma Inc.

**Pros:**
- No need to manage local storage
- Auto-scales with usage
- Persists data across Railway redeploys
- Simpler code (just swap connection)

**Cons:**
- Additional cost (see pricing below)
- Requires API key setup

**Chroma Cloud Pricing:**
- Free tier: Up to 5 collections, limited queries
- Starter: $15/month for 100,000 queries
- Team: $100/month for 1M+ queries
- ✅ **Cheapest option** if you have <10K queries/month

**How to set up:**
1. Go to https://console.trychroma.com
2. Create Chroma Cloud account (free)
3. Get API key: `CHROMA_API_KEY` and `CHROMA_SERVER_HOST`
4. Update app.py code to use Chroma Cloud client instead of local

**Code change required in app.py:**
```python
# OLD (Local)
# import chromadb
# chroma_client = chromadb.PersistentClient(path=chroma_path)

# NEW (Chroma Cloud)
import chromadb
chroma_client = chromadb.HttpClient(
    host=os.getenv("CHROMA_SERVER_HOST"),
    api_key=os.getenv("CHROMA_API_KEY")
)
```

### Option B: Bundle ChromaDB data in deployment

**What:** Commit the `chromadb-store/` directory to GitHub, Railway deploys it.

**Pros:**
- Free (included in Railway)
- Works immediately
- No external dependencies

**Cons:**
- Adds 21 MB to deployment size
- ChromaDB is read-only on Railway (ephemeral filesystem)
- Cannot add new embeddings after deployment
- Gets reset on redeploy

**Implementation:**
1. Remove `chromadb-store/` from `.gitignore`
2. Commit to GitHub: `git add chromadb-store/ && git commit -m "Bundle ChromaDB data"`
3. Railway auto-deploys it

**⚠️ Problem:** If the app tries to write to ChromaDB after startup, it will fail silently (Railway's filesystem is ephemeral except for volumes).

### Option C: Use hardcoded OPSS dict + ChromaDB local-only

**What:** Switch to the hardcoded `OPSS_NOTES` dictionary in app.py for production, keep ChromaDB for local dev.

**Pros:**
- Zero infrastructure cost
- Always works offline
- Simple deployment

**Cons:**
- Limited spec coverage (only ~17 items currently hardcoded)
- No real OPSS document search capability
- Less powerful for production

**Current state:** App.py already has `get_hardcoded_opss_notes()` fallback. If ChromaDB is missing, it defaults to this.

### RECOMMENDATION

**For 10-50 users/month:**
- Start with **Option A (Chroma Cloud Starter, $15/month)** for true production capability
- This gives you real OPSS search + scalability
- Total monthly cost: $5 (Railway Hobby) + $15 (Chroma) = **$20/month**

**For MVP/testing:**
- Use **Option B (Bundle ChromaDB)**
- Zero cost, works immediately
- Document that it's read-only (add warning in UI)

---

## 5. File Size & Storage Considerations

### Current Asset Sizes

| Asset | Size | Notes |
|-------|------|-------|
| `chromadb-store/` | 21 MB | Embedded vectors + metadata |
| `docs/MTO_Estimating_Guide_2023.pdf` | 320 KB | Reference doc |
| Python dependencies | ~500 MB | Installed at build time (not in final size) |
| **Total app size** | ~25 MB | Deployable to Railway |

### Railway Free/Hobby Limits

- Ephemeral disk (temporary files): 100 GB
- Persistent volume: 5 GB (Hobby plan)
- ✅ **21 MB ChromaDB fits easily**

### Option B (Bundle Strategy)

If you bundle chromadb-store:
- Deploy size: ~25 MB (well under limits)
- No persistent volume needed
- Reset on every Railway redeploy (acceptable for prototype)

---

## 6. Hardcoded Paths to Fix Before Deployment

### Found in app.py

**Line 25:** API key loading
```python
# CURRENT
_env = dotenv_values(os.path.expanduser("~/claudbot/.env"))

# ISSUE: Assumes ~/claudbot exists on Railway (it won't)

# FIX: Use Railway env vars instead
_ANTHROPIC_API_KEY = os.getenv("ANTHROPIC_API_KEY")
if not _ANTHROPIC_API_KEY:
    st.error("❌ ANTHROPIC_API_KEY not set in Railway variables")
    st.stop()
```

**Line 55:** History file path
```python
# CURRENT
HISTORY_FILE = "/Users/vanshsmac/MESTRE/tender_history.json"

# FIX: Use relative path
HISTORY_FILE = os.path.join(os.getcwd(), "tender_history.json")
```

**Line 66:** ChromaDB path
```python
# CURRENT
chroma_path = "/Users/vanshsmac/MESTRE/chromadb-store"

# FIX: Use relative path or env var
if os.getenv("MESTRE_ENV") == "production":
    # Use Chroma Cloud (see Option A above)
    chroma_path = None  # Will use cloud
else:
    # Local development
    chroma_path = os.path.join(os.getcwd(), "chromadb-store")
```

### Other Files to Check

- `build_opss_db.py` (Line ~10): Has hardcoded `/Users/vanshsmac/MESTRE/chromadb-store`
- `test_opss_db.py`: Same issue

**Before deployment:** Run `grep -r "/Users/vanshsmac" MESTRE/` to find all hardcoded paths and replace with relative or env-based paths.

---

## 7. Deployment Checklist

### Pre-Deployment

- [ ] Update `requirements.txt` with complete dependencies (see section 2.3)
- [ ] Create `Procfile` in repo root
- [ ] Create `runtime.txt` with `python-3.11`
- [ ] Fix all `/Users/vanshsmac` paths in app.py + build_opss_db.py
- [ ] Choose ChromaDB strategy (Cloud, Bundle, or Hardcoded)
- [ ] If Chroma Cloud: Get API key from console.trychroma.com
- [ ] Create .gitignore to exclude `.env`, `*.json`, `__pycache__`, etc.
- [ ] Push to GitHub (public or private repo)
- [ ] Test locally with `streamlit run app.py` to ensure no import errors

### Deployment Steps (When Ready)

1. Go to railway.com → New Project → Deploy from GitHub
2. Connect your GitHub account (first time only)
3. Select MESTRE repository
4. Railway auto-detects Procfile and Python runtime
5. Wait for build (~2-3 minutes)
6. Add environment variables in Railway dashboard:
   - `ANTHROPIC_API_KEY=sk-proj-xxxxx`
   - (If Chroma Cloud) `CHROMA_API_KEY=xxxx` + `CHROMA_SERVER_HOST=xxxxx`
7. Click "Deploy" → wait for green checkmark
8. Click "Generate Domain" to get public URL
9. Share URL with testers

### Post-Deployment

- [ ] Test the deployed app (click "Generate Domain")
- [ ] Check logs in Railway dashboard for errors
- [ ] Verify ANTHROPIC_API_KEY is set (should see no auth errors)
- [ ] If using Chroma: Verify connection with test query
- [ ] Set up GitHub CI/CD for auto-deployment on push (optional)

---

## 8. Troubleshooting Common Issues

### App won't start
```
Error: port is already in use
```
**Fix:** Ensure Procfile has `--server.port=$PORT`

### ANTHROPIC_API_KEY error
```
Error: AuthenticationError — no API key
```
**Fix:** Set `ANTHROPIC_API_KEY` in Railway Variables (not in code)

### ChromaDB "collection not found"
**If using Bundle strategy:** Collection may not exist yet. Build it locally first, commit, redeploy.  
**If using Chroma Cloud:** Verify `CHROMA_API_KEY` and `CHROMA_SERVER_HOST` in env vars.

### App crashes after deploy
Check Railway logs:
1. Railway dashboard → Service → Logs
2. Scroll to latest deploy
3. Look for Python errors or missing imports
4. Common: missing `sentence-transformers` or `pymupdf`

### Streamlit not accessible
```
Connection refused at localhost:PORT
```
**Fix:** Ensure `--server.address=0.0.0.0` in Procfile (not localhost)

---

## 9. Cost Estimation (10-50 users/month)

### Scenario A: Chroma Cloud + Railway Hobby
| Service | Cost | Notes |
|---------|------|-------|
| Railway Hobby | $5/month | $5 credit included |
| Chroma Starter | $15/month | 100K queries/month |
| **Total** | **$20/month** | Both covered by credits initially |

### Scenario B: Railway Hobby + Bundled ChromaDB
| Service | Cost | Notes |
|---------|------|-------|
| Railway Hobby | $5/month | $5 credit included |
| ChromaDB | Free | Bundled in deployment |
| **Total** | **$5/month** | Includes $5 credit |

### Scenario C: Railway Hobby + Hardcoded OPSS
| Service | Cost | Notes |
|---------|------|-------|
| Railway Hobby | $5/month | $5 credit included |
| **Total** | **$5/month** | Cheapest option |

---

## 10. Next Steps

1. **Decide on ChromaDB strategy** (Cloud vs Bundle vs Hardcoded)
2. **Fix all hardcoded paths** in app.py and related scripts
3. **Test locally** with updated requirements.txt
4. **Push to GitHub** (create private repo if you want)
5. **Create Railway account** and first project
6. **Deploy & test** in Railway sandbox before going live

---

## 11. Railway & Streamlit Integration Notes

### Why Streamlit on Railway works well
- ✅ Stateless app (perfect for serverless)
- ✅ No database connections to manage initially
- ✅ Simple Procfile configuration
- ✅ Hot reload support (on redeploy)

### Why ChromaDB is the blocker
- ❌ Needs persistent storage
- ❌ Can't write to ephemeral filesystem
- ❌ Requires external solution (Cloud or bundled state)

### Best Practice for MESTRE
- **Local dev:** Use local chromadb-store + .env with local API key
- **Production:** Use Chroma Cloud + Railway env vars (no local paths)

---

**Document prepared:** 2026-03-28 20:23 EDT  
**Ready for:** Team review and deployment planning  
**No deployment executed yet** — research only.

---

