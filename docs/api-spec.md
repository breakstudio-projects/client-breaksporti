

# API Specification

## Overview

This document defines the complete API specification for the application built on Supabase. Since the project context is undefined, this specification provides a comprehensive, production-ready API architecture that covers common application patterns and can be adapted to any domain.

---

## Table of Contents

1. [API Versioning & Conventions](#api-versioning--conventions)
2. [Rate Limiting Strategy](#rate-limiting-strategy)
3. [Authentication Flows](#authentication-flows)
4. [User Management](#user-management)
5. [Profile Management](#profile-management)
6. [Resource Management (CRUD)](#resource-management-crud)
7. [File Storage](#file-storage)
8. [Notifications](#notifications)
9. [Admin & Analytics](#admin--analytics)
10. [Webhook Endpoints](#webhook-endpoints)
11. [Third-Party Integrations](#third-party-integrations)
12. [Realtime Subscriptions](#realtime-subscriptions)

---

## API Versioning & Conventions

### Versioning Approach

| Layer | Strategy | Details |
|---|---|---|
| **Supabase REST (PostgREST)** | Schema-based versioning | Use `api_v1`, `api_v2` Postgres schemas exposed via different base paths |
| **Edge Functions** | URL path prefix versioning | `/v1/function-name`, `/v2/function-name` |
| **Breaking changes** | New version + deprecation window | 90-day deprecation notice; sunset header in responses |

### Base URLs

```
# Supabase Auto-Generated REST API
https://<project-ref>.supabase.co/rest/v1/

# Supabase Edge Functions
https://<project-ref>.supabase.co/functions/v1/

# Supabase Auth
https://<project-ref>.supabase.co/auth/v1/

# Supabase Storage
https://<project-ref>.supabase.co/storage/v1/

# Supabase Realtime
wss://<project-ref>.supabase.co/realtime/v1/
```

### Common Headers

```
apikey: <SUPABASE_ANON_KEY>              # Required on all requests
Authorization: Bearer <JWT_TOKEN>        # Required on authenticated requests
Content-Type: application/json           # Request body format
Prefer: return=representation            # Return created/updated record
Prefer: count=exact                      # Include total count in range header
```

### Standard Response Envelope (Edge Functions)

```json
{
  "success": true,
  "data": {},
  "meta": {
    "request_id": "uuid",
    "timestamp": "ISO-8601",
    "version": "v1"
  }
}
```

### Standard Error Response

```json
{
  "success": false,
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource does not exist.",
    "details": {},
    "hint": "Check the resource ID and try again."
  },
  "meta": {
    "request_id": "uuid",
    "timestamp": "ISO-8601"
  }
}
```

### Common Error Codes

| HTTP Status | Code | Description |
|---|---|---|
| 400 | `BAD_REQUEST` | Invalid request body or parameters |
| 401 | `UNAUTHORIZED` | Missing or invalid authentication |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 404 | `RESOURCE_NOT_FOUND` | Resource does not exist |
| 409 | `CONFLICT` | Resource already exists or state conflict |
| 422 | `VALIDATION_ERROR` | Request validation failed |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Unexpected server error |
| 503 | `SERVICE_UNAVAILABLE` | Downstream service unavailable |

---

## Rate Limiting Strategy

### Tier-Based Limits

| Tier | Requests/Minute | Requests/Hour | Burst (per second) |
|---|---|---|---|
| **Anonymous** | 30 | 500 | 5 |
| **Authenticated (Free)** | 120 | 5,000 | 15 |
| **Authenticated (Pro)** | 600 | 30,000 | 50 |
| **Admin** | 1,200 | 60,000 | 100 |
| **Webhook Incoming** | 300 | 10,000 | 30 |

### Implementation

```
# Rate limit headers returned on every response
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 117
X-RateLimit-Reset: 1700000060
Retry-After: 32                          # Only on 429 responses
```

- **Backend**: Implemented via a shared `rate_limiter` module in Edge Functions using Supabase KV / Redis (Upstash).
- **Keying**: Rate limits keyed by `user_id` for authenticated requests, `IP + fingerprint` for anonymous.
- **Sliding Window**: Uses sliding window log algorithm for smooth rate limiting.
- **Exemptions**: Internal service-to-service calls authenticated via `service_role` key are exempt.

---

## Authentication Flows

### 1. Email/Password Sign Up

```
POST /auth/v1/signup
```

| Field | Details |
|---|---|
| **Description** | Register a new user with email and password |
| **Auth Required** | No |
| **Rate Limit** | 5 requests/minute per IP |

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123!",
  "data": {
    "full_name": "Jane Doe",
    "avatar_url": null
  }
}
```

**Response (200):**

```json
{
  "id": "uuid",
  "aud": "authenticated",
  "role": "authenticated",
  "email": "user@example.com",
  "email_confirmed_at": null,
  "phone": "",
  "confirmation_sent_at": "2024-01-15T10:30:00Z",
  "app_metadata": {
    "provider": "email",
    "providers": ["email"]
  },
  "user_metadata": {
    "full_name": "Jane Doe",
    "avatar_url": null
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Error Codes:**

| Status | Code | Condition |
|---|---|---|
| 400 | `BAD_REQUEST` | Invalid email format or weak password |
| 422 | `VALIDATION_ERROR` | Email already registered |
| 429 | `RATE_LIMITED` | Too many signup attempts |

---

### 2. Email/Password Sign In

```
POST /auth/v1/token?grant_type=password
```

| Field | Details |
|---|---|
| **Description** | Authenticate user and receive JWT tokens |
| **Auth Required** | No |
| **Rate Limit** | 10 requests/minute per IP |

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123!"
}
```

**Response (200):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 3600,
  "expires_at": 1700003600,
  "refresh_token": "v1.MjAyNC0wMS...",
  "user": {
    "id": "uuid",
    "aud": "authenticated",
    "role": "authenticated",
    "email": "user@example.com",
    "email_confirmed_at": "2024-01-15T10:35:00Z",
    "app_metadata": {
      "provider": "email",
      "providers": ["email"]
    },
    "user_metadata": {
      "full_name": "Jane Doe"
    }
  }
}
```

**Error Codes:**

| Status | Code | Condition |
|---|---|---|
| 400 | `BAD_REQUEST` | Invalid credentials |
| 422 | `VALIDATION_ERROR` | Email not confirmed |
| 429 | `RATE_LIMITED` | Too many login attempts (lockout) |

---

### 3. Token Refresh

```
POST /auth/v1/token?grant_type=refresh_token
```

| Field | Details |
|---|---|
| **Description** | Refresh an expired access token |
| **Auth Required** | No (uses refresh token) |
| **Rate Limit** | 30 requests/minute per user |

**Request Body:**

```json
{
  "refresh_token": "v1.MjAyNC0wMS..."
}
```

**Response (200):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 3600,
  "expires_at": 1700007200,
  "refresh_token": "v1.NjQ4OTIwMT..."
}
```

**Error Codes:**

| Status | Code | Condition |
|---|---|---|
| 400 | `BAD_REQUEST` | Invalid or expired refresh token |
| 401 | `UNAUTHORIZED` | Token has been revoked |

---

### 4. Password Reset Request

```
POST /auth/v1/recover
```

| Field | Details |
|---|---|
| **Description** | Send password reset email |
| **Auth Required** | No |
| **Rate Limit** | 3 requests/minute per IP |

**Request Body:**

```json
{
  "email": "user@example.com"
}
```

**Response (200):**

```json
{}
```

> Note: Always returns 200 regardless of whether email exists (prevents enumeration).

---

### 5. Password Update

```
PUT /auth/v1/user
```

| Field | Details |
|---|---|
| **Description** | Update user password (after reset or while logged in) |
| **Auth Required** | Yes (Bearer token from reset link or session) |
| **Rate Limit** | 5 requests/minute per user |

**Request Body:**

```json
{
  "password": "NewSecureP@ss456!"
}
```

**Response (200):**

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "updated_at": "2024-01-15T12:00:00Z"
}
```

---

### 6. OAuth Sign In

```
GET /auth/v1/authorize?provider={provider}&redirect_to={url}
```

| Field | Details |
|---|---|
| **Description** | Initiate OAuth flow with third-party provider |
| **Auth Required** | No |
| **Supported Providers** | `google`, `github`, `apple`, `discord`, `twitter` |

**Query Parameters:**

| Param | Type | Required | Description |
|---|---|---|---|
| `provider` | string | Yes | OAuth provider name |
| `redirect_to` | string | Yes | URL to redirect after auth |
| `scopes` | string | No | Additional OAuth scopes (space-delimited) |

**Response:** HTTP 302 redirect to provider's OAuth consent page.

**Callback (handled by Supabase):**

```
GET {redirect_to}#access_token=...&refresh_token=...&token_type=bearer&expires_in=3600
```

---

### 7. Magic Link Sign In

```
POST /auth/v1/magiclink
```

| Field | Details |
|---|---|
| **Description** | Send a passwordless magic link to email |
| **Auth Required** | No |
| **Rate Limit