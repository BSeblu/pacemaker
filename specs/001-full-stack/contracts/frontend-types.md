# Frontend API Contracts

## Authentication Components (Clerk)

### SignIn Component
```typescript
import { SignIn } from "@clerk/clerk-react"

<SignIn 
  path="/sign-in"
  routing="path"
  redirectUrl="/hello-world"
  afterSignInUrl="/hello-world"
/>
```

### SignUp Component
```typescript
import { SignUp } from "@clerk/clerk-react"

<SignUp 
  path="/sign-up"
  routing="path"
  redirectUrl="/hello-world"
  afterSignUpUrl="/hello-world"
/>
```

### UserButton Component
```typescript
import { UserButton } from "@clerk/clerk-react"

<UserButton 
  afterSignOutUrl="/"
  appearance={{
    elements: {
      avatarBox: "w-10 h-10",
    }
  }}
/>
```

## API Client Types

### API Response Interfaces
```typescript
// src/types/api.ts
export interface HelloWorldResponse {
  message: "hello world"
  userId: string
  timestamp: string
  status: "success"
}

export interface ErrorResponse {
  error: string
  message: string
  timestamp: string
}

export interface HealthResponse {
  status: "ok"
  timestamp: string
  version: string
}
```

### API Client Functions
```typescript
// src/services/api.ts
export class ApiClient {
  private baseURL: string

  constructor(baseURL: string = 'http://localhost:3000') {
    this.baseURL = baseURL
  }

  async getHelloWorld(): Promise<HelloWorldResponse> {
    const response = await fetch(`${this.baseURL}/api/hello-world`, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
    })

    if (!response.ok) {
      const error: ErrorResponse = await response.json()
      throw new Error(error.message)
    }

    return response.json()
  }

  async getHealth(): Promise<HealthResponse> {
    const response = await fetch(`${this.baseURL}/api/health`, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
      },
    })

    return response.json()
  }
}
```

## React Hook for API Calls

```typescript
// src/hooks/useHelloWorld.ts
import { useAuth } from '@clerk/clerk-react'
import { useQuery } from '@tanstack/react-query'
import { ApiClient, HelloWorldResponse } from '../services/api'

export function useHelloWorld() {
  const { isSignedIn } = useAuth()
  const apiClient = new ApiClient()

  return useQuery<HelloWorldResponse>({
    queryKey: ['hello-world'],
    queryFn: () => apiClient.getHelloWorld(),
    enabled: isSignedIn, // Only fetch if user is authenticated
    retry: (failureCount, error) => {
      // Don't retry on auth errors
      if (error.message.includes('Unauthorized') || error.message.includes('Forbidden')) {
        return false
      }
      return failureCount < 3
    },
  })
}
```

## Component Props Interfaces

### Hello World Page Component
```typescript
// src/pages/HelloWorldPage.tsx
interface HelloWorldPageProps {
  className?: string
}

export function HelloWorldPage({ className }: HelloWorldPageProps) {
  // Component implementation
}
```

### Navigation Component
```typescript
// src/components/Navigation.tsx
interface NavigationProps {
  className?: string
}

export function Navigation({ className }: NavigationProps) {
  // Component implementation
}
```

## Routing Configuration

```typescript
// src/router/index.tsx
import { createBrowserRouter } from 'react-router-dom'
import { HomePage } from '../pages/HomePage'
import { SignInPage } from '../pages/SignInPage'
import { SignUpPage } from '../pages/SignUpPage'
import { HelloWorldPage } from '../pages/HelloWorldPage'

export const router = createBrowserRouter([
  {
    path: '/',
    element: <HomePage />,
  },
  {
    path: '/sign-in',
    element: <SignInPage />,
  },
  {
    path: '/sign-up', 
    element: <SignUpPage />,
  },
  {
    path: '/hello-world',
    element: <HelloWorldPage />,
  },
])
```

## Error Handling Types

```typescript
// src/types/errors.ts
export enum ApiError {
  NETWORK_ERROR = 'NETWORK_ERROR',
  AUTHENTICATION_ERROR = 'AUTHENTICATION_ERROR',
  AUTHORIZATION_ERROR = 'AUTHORIZATION_ERROR',
  SERVER_ERROR = 'SERVER_ERROR',
  UNKNOWN_ERROR = 'UNKNOWN_ERROR',
}

export interface AppError {
  type: ApiError
  message: string
  timestamp: string
}

export function mapApiErrorToAppError(error: ErrorResponse): AppError {
  switch (error.error) {
    case 'Unauthorized':
      return {
        type: ApiError.AUTHENTICATION_ERROR,
        message: error.message,
        timestamp: error.timestamp,
      }
    case 'Forbidden':
      return {
        type: ApiError.AUTHORIZATION_ERROR,
        message: error.message,
        timestamp: error.timestamp,
      }
    case 'Internal Server Error':
      return {
        type: ApiError.SERVER_ERROR,
        message: error.message,
        timestamp: error.timestamp,
      }
    default:
      return {
        type: ApiError.UNKNOWN_ERROR,
        message: error.message,
        timestamp: error.timestamp,
      }
  }
}
```