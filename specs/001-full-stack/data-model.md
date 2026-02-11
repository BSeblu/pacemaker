# Data Model: Full Stack Scaffolding - Serverless Architecture

**Project**: 001-full-stack  
**Date**: 2026-02-11  
**Status**: Draft

## Overview

This data model defines the entities for the serverless full-stack application with Neon PostgreSQL persistence and Clerk authentication.

## Entities

### User (Clerk-Managed)

**Source**: Clerk authentication service  
**Storage**: External (Clerk) - not in PostgreSQL  

**Fields**:
- `userId` (string, required) - Unique identifier from Clerk
- `email` (string, required) - User email address
- `firstName` (string, optional) - User first name
- `lastName` (string, optional) - User last name
- `isSignedIn` (boolean, required) - Authentication status
- `sessionToken` (string, required) - JWT token from Clerk

**Relationships**: 
- One-to-Many with AuditLog (user actions)
- One-to-Many with UserSettings (user preferences)

### AuditLog

**Source**: Application events  
**Storage**: Neon PostgreSQL  
**Table**: `audit_logs`

**Fields**:
- `id` (UUID, primary key) - Unique log entry identifier
- `user_id` (string, required, indexed) - Clerk user ID
- `action` (string, required) - Action performed (e.g., "hello_world_accessed")
- `timestamp` (timestamp, required) - When action occurred
- `ip_address` (string, optional) - Client IP address
- `user_agent` (string, optional) - Client user agent

**SQL Schema**:
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(255) NOT NULL,
    action VARCHAR(100) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent TEXT,
    CONSTRAINT valid_action CHECK (action IN ('hello_world_accessed', 'user_signin', 'user_signout'))
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
```

**Validation Rules**:
- user_id must not be empty
- action must be from allowed set
- timestamp auto-generated if not provided

### UserSettings

**Source**: User preferences  
**Storage**: Neon PostgreSQL  
**Table**: `user_settings`

**Fields**:
- `id` (UUID, primary key) - Unique settings identifier
- `user_id` (string, required, unique) - Clerk user ID
- `theme` (string, default: 'light') - UI theme preference
- `notifications_enabled` (boolean, default: true) - Notification preference
- `created_at` (timestamp, required) - When settings created
- `updated_at` (timestamp, required) - When settings last updated

**SQL Schema**:
```sql
CREATE TABLE user_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(255) NOT NULL UNIQUE,
    theme VARCHAR(20) DEFAULT 'light' CHECK (theme IN ('light', 'dark', 'system')),
    notifications_enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_settings_user_id ON user_settings(user_id);
```

### HelloWorldResponse (API Response)

**Source**: Backend API  
**Storage**: Stateless response (not persisted)  

**Fields**:
- `message` (string, required) - "hello world" message
- `userId` (string, required) - ID of authenticated user
- `timestamp` (string, required) - ISO 8601 timestamp
- `status` (string, required) - Response status ("success")
- `requestCount` (integer, optional) - Number of times user accessed endpoint (from AuditLog)

**Type Definition**:
```typescript
interface HelloWorldResponse {
  message: "hello world"
  userId: string
  timestamp: string
  status: "success"
  requestCount?: number
}
```

## Entity Relationships

```
User (Clerk) ──1:N──> AuditLog
  └─ Tracks all user actions in PostgreSQL

User (Clerk) ──1:1──> UserSettings
  └─ Stores user preferences in PostgreSQL

User ──generates──> HelloWorldResponse
  └─ Stateless API response with optional request count
```

## Database Connection Strategy

### Neon Serverless Driver

**Approach**: HTTP-based connections (no pooling)

**Configuration**:
```python
from neon_api import NeonAPI

# Connection string from environment
DATABASE_URL = os.environ.get('DATABASE_URL')

# Serverless driver handles HTTP requests
# Each query is independent HTTP POST
```

**Connection Pattern**:
```python
async def get_db():
    """Get database connection for serverless environment"""
    import psycopg2
    # HTTP-based connection - no pooling needed
    conn = psycopg2.connect(DATABASE_URL)
    try:
        yield conn
    finally:
        conn.close()
```

**Key Characteristics**:
- No connection pooling required (stateless HTTP)
- Each request opens/closes connection automatically
- Latency: ~20-50ms for simple queries
- Retry logic built into driver for transient failures

## API Interface Contracts

### Authentication Endpoints

**Note**: Handled by Clerk UI components, not direct API endpoints.

### Protected API Endpoints

#### GET /api/hello-world
**Description**: Returns hello world message and logs access  
**Authentication**: Required (valid Clerk JWT)  
**Database Operations**: 
- INSERT into audit_logs (user action)
- SELECT COUNT from audit_logs (optional request count)

**Request**: None (authenticated via JWT token)  
**Response**: HelloWorldResponse

#### GET /api/health
**Description**: Health check with database connectivity test  
**Authentication**: None  
**Database Operations**: Simple SELECT 1 query  
**Response**: Health status with database connectivity

#### GET /api/user/settings
**Description**: Get user preferences  
**Authentication**: Required  
**Database Operations**: SELECT from user_settings  
**Response**: UserSettings object

#### PUT /api/user/settings
**Description**: Update user preferences  
**Authentication**: Required  
**Database Operations**: UPSERT into user_settings  
**Request**: Partial UserSettings object  
**Response**: Updated UserSettings object

## Frontend State Management

### Authentication Context
```typescript
interface AuthContext {
  isSignedIn: boolean
  isLoaded: boolean
  user: User | null
  token: string | null
}
```

### API Response Types
```typescript
interface HelloWorldResponse {
  message: "hello world"
  userId: string
  timestamp: string
  status: "success"
  requestCount?: number
}

interface UserSettings {
  theme: "light" | "dark" | "system"
  notificationsEnabled: boolean
}
```

## Security Considerations

### Database Security
- Use row-level security (RLS) for user-specific data
- Never store Clerk secrets in database
- Audit log tracks all sensitive operations
- Database connections use SSL/TLS only

### Data Retention
- Audit logs: 90 days (configurable)
- User settings: Persistent until account deletion
- Implement data export/deletion for GDPR compliance

## Performance Optimization

### Database Indexes
- Primary keys (automatic)
- user_id indexes for AuditLog and UserSettings
- timestamp indexes for time-based queries

### Query Optimization
- Use LIMIT for large audit log queries
- Implement pagination for user history
- Cache user settings in frontend (short TTL)

## Migration Strategy

### Initial Schema Setup
```sql
-- Run on database creation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create tables
CREATE TABLE audit_logs (...);
CREATE TABLE user_settings (...);

-- Create indexes
CREATE INDEX ...;
```

### Schema Evolution
- Use Alembic for Python/FastAPI migrations
- Each deployment runs migrations automatically
- Backward-compatible changes only

## No-Persistence Clarification

The original specification mentioned "no database for now" but this was superseded by the serverless architecture clarification (2026-02-11). The current implementation uses:
- **Neon PostgreSQL** for persistent data (user settings, audit logs)
- **Clerk** for user authentication data (external, not in our database)
- **Stateless API responses** (computed on-demand, not cached)