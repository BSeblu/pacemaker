# Research Document: Full Stack Scaffolding - Serverless Architecture with Next.js

**Project**: 001-full-stack  
**Date**: 2026-02-11  
**Status**: Complete

## Architecture Overview

Cost-optimized serverless architecture with scale-to-zero capabilities for both compute and database layers, leveraging Next.js with SSR and FastAPI with Neon PostgreSQL.

## Technology Decisions

### Backend Framework

**Decision**: Python with FastAPI (Serverless Deployment)  
**Version**: Python 3.11+ (preferably 3.12), FastAPI 0.104+  
**Rationale**: FastAPI provides excellent async support required for serverless environments. Python 3.11 offers 20-30% performance improvement over 3.10. AWS Lambda, Vercel, and Railway support Python 3.11+ natively with 15-25% performance gains.  
**Alternatives considered**: Node.js/Hono (better cold starts but less mature ecosystem), Node.js/Fastify (heavier bundle size), Go (steeper learning curve).

**Key Dependencies**:
- `fastapi[standard]` - Optimized dependency bundle
- `mangum>=0.17.0` - AWS Lambda adapter
- `neon-api` - HTTP-based database connections
- `pydantic>=2.0` - Type validation

### Frontend Framework

**Decision**: Next.js with SSR (Server-Side Rendering)  
**Version**: Next.js 15+ with Node.js 20+ LTS  
**Rationale**: Next.js provides built-in SSR, excellent serverless integration, and superior developer experience with TypeScript support. Next.js 15 includes Server Components, improved Edge Runtime support, and better performance.  
**Alternatives considered**: React + Vite SPA (rejected due to user preference for SSR and Next.js), SvelteKit (smaller ecosystem), Nuxt.js (Vue-based, less familiar).

**Key Dependencies**:
- `next@15+` - React framework with SSR
- `@clerk/nextjs` - Authentication integration
- `@tanstack/react-query` - API state management
- `vitest` - Modern testing framework

### Database Solution

**Decision**: Neon Serverless PostgreSQL  
**Connection Strategy**: HTTP-based connections (no traditional pooling)  
**Rationale**: Purpose-built for serverless architectures with HTTP-based connections. Generous free tier (500MB storage, 190 compute hours). Auto-suspends after 5 minutes of inactivity (scale-to-zero).  
**Alternatives considered**: Supabase (requires connection pooling), PlanetScale (MySQL-based), AWS Aurora Serverless (vendor lock-in, more complex).

**Connection Optimization**:
- Use `neon.tech?sslmode=require&pool_mode=transaction` for optimal performance
- Disable SQLAlchemy pooling: `NullPool` to delegate to Neon's PgBouncer
- Up to 10,000 concurrent connections vs 100-200 direct connections

### Testing Strategy

**Decision**: pytest (backend) + Vitest (frontend)  
**Backend Stack**: pytest + pytest-asyncio + httpx + pytest-cov  
**Frontend Stack**: Vitest + React Testing Library + Playwright (E2E)  
**Rationale**: pytest is the standard for Python with excellent async testing support. Vitest is 10x faster than Jest in watch mode with native TypeScript and ESM support for Next.js projects.  

**Serverless Testing Approach**:
- Unit tests: Test FastAPI endpoints directly using TestClient
- Integration tests: Test with mocked Neon driver
- E2E tests: Full flow with real database (test branch)

### Deployment Platform

**Decision**: Vercel (primary) with framework-agnostic configuration  
**Target**: Serverless functions with Edge Runtime optimization  
**Rationale**: Vercel has first-class support for both Next.js and Python serverless functions. Zero-config deployment from Git. Generous free tier.  
**Alternatives considered**: Railway (excellent but slightly more expensive), AWS Lambda (more complex configuration), Netlify (good but Python support less mature).

**Performance Targets**:
- **Edge Runtime**: 9x faster cold starts vs traditional serverless
- **Node.js Runtime**: Full API access, 50MB bundle limit
- **Geographic distribution**: Automatic with Vercel Edge Network

## Updated Technical Context

**Language/Version**: Python 3.11+ (preferably 3.12), TypeScript/Node.js 20+ LTS, Next.js 15+  
**Primary Dependencies**: FastAPI, Next.js, Neon serverless PostgreSQL, Clerk authentication, pytest, Vitest  
**Storage**: Neon serverless PostgreSQL (HTTP-based connections with pooling)  
**Testing**: pytest + pytest-asyncio (backend), Vitest + React Testing Library (frontend)  
**Target Platform**: Vercel (Edge/Node.js serverless), framework-agnostic deployment compatibility  
**Project Type**: web (full-stack with serverless backend + frontend)  
**Performance Goals**: API <100ms warm, <1000ms cold start, database <200ms, bundle <1MB  
**Constraints**: $0 idle cost, scale-to-zero architecture, 3-step setup, <10ms dev server restarts  
**Scale/Scope**: Development environment to production-ready serverless application

## Performance Optimization

### Cold Start Mitigation

**Backend Strategies**:
1. Use `fastapi[standard]` for minimal dependency bundle
2. Lazy imports for heavy modules
3. Connection reuse within same serverless invocation
4. ARM64 (Graviton2) runtime for 20% cost reduction

**Expected Performance**:
- Python + FastAPI: ~300-500ms cold start
- Database query (first): +100-200ms
- Total cold start: <1000ms (meets SC-003)

**Frontend Strategies**:
1. Next.js Edge Runtime for geo-distributed functions
2. Incremental Static Regeneration (ISR) for semi-static content
3. Bundle optimization with automatic code splitting

**Expected Performance**:
- Edge functions: ~50-100ms cold start
- SSR pages: <500ms first render
- CSR interactions: <100ms after initial load

### Database Optimization

**Neon-Specific Features**:
- Serverless driver handles HTTP-based connections automatically
- PgBouncer connection pooling built-in
- Automatic retry logic for transient failures
- Geographic proximity deployment reduces latency

**Query Optimization**:
- Use LIMIT for large audit log queries
- Implement pagination for user history
- Cache user settings in frontend (short TTL)

## Implementation Approach

### Setup Workflow
1. Clone repository and run setup script (3 minutes)
2. Configure environment variables (Clerk + Neon) (2 minutes)
3. Start Docker Compose for local development (1 minute)

Total setup time: **~6 minutes** (meets SC-001 and SC-006)

### Development Flow
- Local: Docker Compose with hot reload
- Database: Connect to Neon (can use local Postgres for offline dev)
- Testing: pytest with mocked database, Vitest for frontend
- Deployment: Git push triggers Vercel deployment

### Cost Optimization

**Compute Costs**:
- Vercel Hobby: $0 (500GB bandwidth, 100GB-hours serverless functions)
- Cold starts mean $0 when idle

**Database Costs**:
- Neon Free Tier: $0 (500MB storage, 190 compute hours/month)
- Auto-suspend after 5min inactivity = $0 when not in use

**Total Idle Cost**: $0/month (meets SC-011)

## ADHD-Friendly Features

### 3-Step Setup
1. Clone repository
2. Configure environment variables  
3. Run `docker compose up`

**Immediate Value**:
- Hello world endpoint provides instant feedback
- Next.js hot reload: <2 seconds
- Clear error messages with actionable steps

**Development Experience**:
- Next.js provides superior TypeScript support and plugin ecosystem
- Vitest offers 10x faster test feedback than Jest
- Serverless functions scale-to-zero when not in use

## Migration Notes

This replaces the original React/Vite SPA architecture with Next.js SSR approach:
- **Changed**: Frontend architecture (SPA → SSR)
- **Enhanced**: Server-side rendering for better performance and SEO
- **Improved**: Developer experience with Next.js optimizations
- **Maintained**: Backend (FastAPI), Database (Neon), Deployment (serverless)

## Risk Mitigation

**Cold Start Latency**:
- Mitigation: Acceptable for development/MVP use case
- Fallback: Can migrate to Railway for consistent warm performance

**Database Connection Reliability**:
- Mitigation: HTTP-based connections are stateless and retry-friendly
- Fallback: Implement exponential backoff in database layer

**Vendor Lock-in**:
- Mitigation: FastAPI is portable, Neon uses standard PostgreSQL
- Fallback: Can migrate to any PostgreSQL provider

**Performance Variability**:
- Mitigation: Next.js Edge Runtime provides geographic optimization
- Monitoring: Built-in Vercel analytics and error tracking

## Best Practices Documentation

### Backend FastAPI
```python
# Optimal dependencies
fastapi[standard]>=0.104.0
mangum>=0.17.0
neon-api>=0.2.0
pydantic>=2.0
sqlmodel>=0.0.14
pytest-asyncio>=0.21.0
```

### Frontend Next.js
```javascript
// Optimal dependencies
next@15.0.0
@clerk/nextjs@5.0.0
@tanstack/react-query@5.0.0
vitest@1.0.0
@testing-library/react@14.0.0
```

### Development Environment
```yaml
# Docker Compose with hot reload
services:
  frontend:
    build: ./frontend
    volumes: ['./frontend:/app']
    ports: ['3000:3000']
  
  backend:
    build: ./backend  
    volumes: ['./backend:/app']
    ports: ['3001:3000']
```

## Deployment Configuration

### Vercel.json
```json
{
  "functions": {
    "backend/api/*.py": {
      "runtime": "python3.11",
      "maxDuration": 10
    }
  },
  "framework": null,
  "buildCommand": "npm run build",
  "outputDirectory": ".next"
}
```

### Next.js Configuration
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    runtime: 'edge',
  },
  env: {
    DATABASE_URL: process.env.DATABASE_URL,
  }
}

module.exports = nextConfig
```

This research provides the foundation for implementing the full-stack serverless application with Next.js SSR, meeting all performance, cost, and ADHD-friendly requirements while maintaining framework-agnostic deployment capabilities.