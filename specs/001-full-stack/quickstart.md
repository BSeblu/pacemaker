# Quick Start Guide: Full Stack Scaffolding - Serverless

**Project**: 001-full-stack  
**Architecture**: Serverless Python/FastAPI + Neon PostgreSQL + React  
**Setup Time**: ~6 minutes  
**Difficulty**: Beginner

## Prerequisites

1. **Python 3.11+** - [Download Python](https://www.python.org/)
2. **Node.js 20+** - [Download Node.js](https://nodejs.org/)
3. **Git** - [Download Git](https://git-scm.com/)
4. **Docker & Docker Compose** - [Download Docker](https://www.docker.com/products/docker-desktop/)
5. **Clerk Account** - [Create free account](https://clerk.com/)
6. **Neon Account** - [Create free account](https://neon.tech/)
7. **Vercel Account** (for deployment) - [Create free account](https://vercel.com/)

## Step 1: Get Credentials (3 minutes)

### Clerk Setup
1. Sign in to your [Clerk Dashboard](https://dashboard.clerk.com/)
2. Create a new application
3. Go to **API Keys** section
4. Copy **Publishable Key** and **Secret Key**
5. Go to **Domains** and add `http://localhost:5173` and your Vercel preview/production URLs

### Neon Setup
1. Sign in to your [Neon Console](https://console.neon.tech/)
2. Create a new project
3. Create a database (e.g., `pacemaker_dev`)
4. Copy the **Connection String** (will look like: `postgresql://user:pass@host.neon.tech/dbname?sslmode=require`)
5. Save this for Step 3

## Step 2: Clone and Setup (2 minutes)

```bash
# Clone the repository
git clone <repository-url>
cd pacemaker

# Switch to the feature branch
git checkout 001-full-stack

# Setup environment variables
cp .env.example .env
```

## Step 3: Configure Environment (1 minute)

Edit `.env` file with your credentials:

```env
# Clerk Configuration
VITE_CLERK_PUBLISHABLE_KEY=pk_test_your_publishable_key_here
CLERK_SECRET_KEY=sk_test_your_secret_key_here
CLERK_WEBHOOK_SECRET=whsec_your_webhook_secret_here

# Neon Database Configuration
DATABASE_URL=postgresql://user:password@host.neon.tech/dbname?sslmode=require

# API Configuration
VITE_API_URL=http://localhost:3000
BACKEND_URL=http://localhost:3000
```

## Step 4: Start Development Environment

### Docker Compose (Recommended for consistency)

```bash
# Start all services
docker compose up --build

# This will start:
# - Backend API on http://localhost:3000 (FastAPI with hot reload)
# - Frontend app on http://localhost:5173 (Vite with HMR)
# - Automatic environment loading
```

### Manual Setup (Alternative)

```bash
# Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python -m uvicorn src.main:app --reload --port 3000

# Frontend setup (new terminal)
cd frontend
npm install
npm run dev
```

## Step 5: Initialize Database

The application will auto-create tables on first run, but you can also run migrations manually:

```bash
cd backend
python -m alembic upgrade head
```

Or use the Docker approach:
```bash
docker compose exec backend python -m alembic upgrade head
```

## Step 6: Test the Application

1. **Open http://localhost:5173** in your browser
2. **Sign up** using the Clerk authentication form
3. **Navigate to Hello World page** after authentication
4. You should see:
   - "hello world" message
   - Your user ID
   - Request count (increments with each visit)
   - Settings page to customize preferences

## Development Workflow

### Running Tests

```bash
# Backend tests (with mocked database)
cd backend
pytest

# Frontend tests
cd frontend
npm test

# End-to-end tests
npm run test:e2e
```

### Database Operations

```bash
# Access database shell
docker compose exec backend python -c "from src.database import get_db; ..."

# View logs
docker compose logs -f backend

# Reset database (careful!)
docker compose down -v
```

### Making Changes

**Backend** (FastAPI):
- Edit files in `backend/src/`
- Auto-reload enabled (fastest feedback loop)
- Tests run automatically with pytest watch mode

**Frontend** (React + Vite):
- Edit files in `frontend/src/`
- Hot Module Replacement (HMR) provides instant updates
- Tests run automatically with Vitest

## Project Structure

```
pacemaker/
├── backend/                    # Python FastAPI (Serverless)
│   ├── api/
│   │   └── hello.py           # Serverless function entry point
│   ├── src/
│   │   ├── database.py        # Neon serverless driver config
│   │   ├── auth.py            # Clerk JWT validation
│   │   └── main.py            # FastAPI application
│   ├── migrations/            # Alembic database migrations
│   ├── tests/
│   │   ├── test_api.py
│   │   └── test_database.py
│   ├── requirements.txt
│   ├── vercel.json            # Serverless deployment config
│   └── Dockerfile.dev
├── frontend/                   # React + Vite
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   └── App.tsx
│   ├── tests/
│   └── Dockerfile.dev
├── docker-compose.yml         # Local development
├── vercel.json               # Root deployment config
└── README.md
```

## Serverless Architecture Details

### Cost Optimization

**Zero Cost When Idle**:
- Vercel Functions: Scale to zero, pay only for execution time
- Neon Database: Auto-suspends after 5 minutes of inactivity
- Total idle cost: $0/month

**Usage Limits (Free Tiers)**:
- Vercel: 100GB-hours serverless functions, 500GB bandwidth
- Neon: 500MB storage, 190 compute hours/month
- Perfect for development and small production workloads

### Cold Start Behavior

**Expected Latency**:
- Warm invocation: <100ms (meets SC-003)
- Cold start: 300-500ms (acceptable for this use case)
- First database query: +50-100ms

**Mitigation Strategies**:
- Use dependency caching (Vercel handles this)
- Lazy imports for heavy libraries
- Consider warm-up pings for critical endpoints

### Database Connection Strategy

**HTTP-Based Connections**:
- No connection pooling needed
- Each query is independent HTTP POST request
- Neon serverless driver handles retries automatically
- Latency: ~20-50ms per query

```python
# Example usage in FastAPI
from neon_api import NeonAPI

async def get_user_count():
    # HTTP-based query - no persistent connection
    result = await neon.query("SELECT COUNT(*) FROM users")
    return result[0][0]
```

## Deployment to Vercel

### One-Time Setup

1. Connect your GitHub repository to Vercel
2. Set environment variables in Vercel dashboard:
   - `DATABASE_URL` (Neon connection string)
   - `CLERK_SECRET_KEY`
   - `CLERK_WEBHOOK_SECRET`
3. Deploy!

### Environment Variables

In Vercel dashboard, add:
```
DATABASE_URL=postgresql://...
CLERK_SECRET_KEY=sk_...
CLERK_WEBHOOK_SECRET=whsec_...
```

### Deployment Workflow

```bash
# Push to main branch triggers automatic deployment
git push origin 001-full-stack

# Vercel will:
# 1. Detect Python/FastAPI backend
# 2. Build serverless functions
# 3. Deploy frontend (if in same repo)
# 4. Apply environment variables
```

## Common Issues & Solutions

### Database Connection Errors

**Error**: `Connection refused` or `timeout`

**Solutions**:
1. Verify DATABASE_URL is correct
2. Check Neon project is active (not paused)
3. Ensure SSL mode is enabled: `?sslmode=require`
4. Try reconnecting: `docker compose restart backend`

### Cold Start Timeouts

**Error**: 504 Gateway Timeout on first request

**Solutions**:
1. This is normal for cold starts
2. Increase timeout in `vercel.json`:
   ```json
   {
     "functions": {
       "api/*.py": {
         "maxDuration": 10
       }
     }
   }
   ```
3. Implement health check endpoint to warm up

### Clerk Authentication Issues

**Error**: `Unauthorized` or redirect loops

**Solutions**:
1. Verify Clerk keys are correctly set
2. Check allowed origins in Clerk dashboard
3. Clear browser cookies/cache
4. Ensure frontend and backend keys match environment

### Database Migrations

**Running migrations locally**:
```bash
docker compose exec backend alembic upgrade head
```

**Creating new migration**:
```bash
cd backend
alembic revision --autogenerate -m "add user preferences"
```

## Development Tips

### ADHD-Friendly Features

- **3-step setup**: Clone → Configure → Run
- **Instant feedback**: Hot reload on both frontend and backend
- **Clear error messages**: Structured error responses with actionable steps
- **Visual progress**: Request counter shows database is working
- **Fast tests**: pytest and Vitest provide immediate feedback

### Performance Monitoring

```bash
# Check API response times
curl -w "@curl-format.txt" http://localhost:3000/api/health

# Monitor cold starts
vercel logs --json | grep "duration"

# Database query performance
# Check Neon dashboard for query analytics
```

### Local vs Production

| Feature | Local (Docker) | Production (Vercel) |
|---------|---------------|---------------------|
| Backend | FastAPI dev server | Serverless functions |
| Database | Neon (dev branch) | Neon (prod branch) |
| Hot reload | Yes | No (redeploy) |
| Cold starts | No | Yes |
| Cost | $0 | $0 (within limits) |

## Next Steps

1. **Customize the schema**: Add new tables in `backend/migrations/`
2. **Add API endpoints**: Create new files in `backend/api/`
3. **Build UI components**: Add to `frontend/src/components/`
4. **Write tests**: Follow existing patterns in `tests/` directories
5. **Deploy**: Push to trigger automatic Vercel deployment

## Getting Help

1. **Check logs**: `docker compose logs [service]`
2. **Verify health**: `curl http://localhost:3000/api/health`
3. **Test database**: Check Neon dashboard connection status
4. **Review contracts**: See `/contracts/api.yaml` for API specs

Enjoy your cost-optimized serverless development environment! 🚀

---

**Total Cost When Idle**: $0/month  
**Setup Time**: ~6 minutes  
**Architecture**: Serverless Python + Neon PostgreSQL + React