# Research Document: Full Stack Scaffolding - Serverless Architecture

**Project**: 001-full-stack  
**Date**: 2026-02-11  
**Status**: Complete

## Architecture Overview

Cost-optimized serverless architecture with scale-to-zero capabilities for both compute and database layers.

## Technology Decisions

### Backend Framework

**Decision**: Python with FastAPI (Serverless Deployment)  
**Rationale**: FastAPI provides excellent async support required for serverless environments. Its dependency injection system works well with Neon's serverless driver. Python's ecosystem has mature support for serverless deployment via Vercel, AWS Lambda, and Railway.  
**Alternatives considered**: Node.js/Hono (better cold starts but less mature ecosystem), Node.js/Fastify (heavier bundle size), Go (steeper learning curve).

**Serverless Considerations**:
- Cold start latency: ~200-500ms (acceptable for this use case)
- Bundle size: FastAPI + dependencies ~15-20MB
- Deployment: Vercel Functions or Railway provide seamless serverless deployment
- HTTP-based database connections eliminate connection pooling complexity

### Database

**Decision**: Neon Serverless PostgreSQL  
**Rationale**: Purpose-built for serverless architectures with HTTP-based connections (no TCP connection pooling needed). Generous free tier includes 500MB storage and 190 compute hours. Auto-suspends after 5 minutes of inactivity (scale-to-zero).  
**Alternatives considered**: Supabase (requires connection pooling), PlanetScale (MySQL-based), AWS Aurora Serverless (vendor lock-in, more complex).

**Connection Strategy**:
- Use `@neondatabase/serverless` driver (HTTP-based)
- No connection pooling required (stateless HTTP requests)
- Each database operation is independent HTTP request
- Auto-scaling handled by Neon platform

### Frontend Framework

**Decision**: React with Vite  
**Rationale**: Unchanged from original plan. Vite's fast HMR and minimal bundle size align with ADHD-friendly development. React has excellent Clerk integration.  
**Alternatives considered**: Same as original research (SvelteKit, Nuxt.js, Next.js).

### Testing Strategy

**Decision**: pytest (backend) + Vitest (frontend)  
**Rationale**: pytest is the standard for Python with excellent async testing support. For serverless functions, we test the handler functions directly. Vitest provides fast feedback for frontend development.  
**Serverless Testing Approach**:
- Unit tests: Test FastAPI endpoints directly using TestClient
- Integration tests: Test with mocked Neon driver
- E2E tests: Full flow with real database (test branch)

### Deployment Platform

**Decision**: Vercel (primary) with Railway fallback  
**Rationale**: Vercel has first-class support for Python serverless functions with FastAPI. Zero-config deployment from Git. Built-in CI/CD. Generous free tier (hobby plan).  
**Alternatives considered**: Railway (excellent but slightly more expensive), AWS Lambda (more complex configuration), Netlify (good but Python support less mature).

## Updated Technical Context

**Language/Version**: Python 3.11+, TypeScript/Node.js 20+  
**Primary Dependencies**: FastAPI, @neondatabase/serverless, React, Vite, Clerk, pytest  
**Storage**: Neon serverless PostgreSQL (HTTP-based connections via serverless driver)  
**Testing**: pytest (backend), Vitest + React Testing Library (frontend), Playwright (e2e)  
**Target Platform**: Vercel (serverless deployment), Docker (local development)  
**Project Type**: web (serverless backend + frontend)  
**Performance Goals**: 
- Warm API response: <100ms
- Cold start: <1000ms
- Database operations: <200ms  
**Constraints**: 
- Setup <10 minutes
- $0 cost when idle (scale-to-zero)
- Serverless architecture (stateless functions)  
**Scale/Scope**: Development to production-ready serverless application

## Implementation Approach

### Setup Workflow
1. Clone repository and run setup script (3 minutes)
2. Configure environment variables (Clerk + Neon) (2 minutes)
3. Start Docker Compose for local development (1 minute)

Total setup time: **~6 minutes**

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

**Total Idle Cost**: $0/month

### Cold Start Mitigation

**Strategies**:
1. Use lazy imports where possible
2. Minimize dependencies in requirements.txt
3. Use connection reuse within same invocation
4. Consider Vercel's "Fluid Compute" for consistent performance
5. Warm-up pings for critical endpoints (optional)

**Expected Cold Start Times**:
- Python + FastAPI: ~300-500ms
- Database query (first): +100-200ms
- Total cold start: <1000ms (meets SC-003)

## Migration Notes

This replaces the original Node.js/Fastify architecture with Python/FastAPI serverless approach:
- **Changed**: Backend language (Node.js → Python)
- **Changed**: Architecture (container-based → serverless functions)
- **Changed**: Database (none → Neon PostgreSQL)
- **Changed**: Connection strategy (none → HTTP-based serverless driver)
- **Unchanged**: Frontend (React + Vite + Clerk)
- **Unchanged**: Testing philosophy (fast feedback loops)
- **Unchanged**: Deployment target (Vercel/Railway)

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