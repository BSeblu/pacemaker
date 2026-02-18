
# Architecture Overview

## Phase 1 (No AI)
- Deterministic behavior core
- Fully functional without AI

## Tech Stack
- Frontend: Next.js
- Backend: Nest.js
- Database: Serverless SQLite (Turso cloud / drizzle)
- Authentication: Clerk
- Optional: serverless deployment (Vercel / Netlify / Cloudflare)

## Contexts
- Action Core Context: manages actions, tasks, projects
- Cadence Context: manages routines and triggers
- Engagement Context: user-facing orchestration
- AI / Insight Context: future, optional
