# Technical Blueprint — Breaksporti

---

## Part 1: System Architecture



# System Architecture Document — Breaksporti

**Document Version:** 1.0
**Author:** Senior Solutions Architect, Break Studio
**Date:** 2025
**Status:** Living Document

---

## 1. Architecture Overview

### 1.1 High-Level Diagram Description

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              END USERS                                       │
│                  (Web Browsers / Mobile Browsers / PWA)                       │
└──────────────────────┬───────────────────────────────────────────────────────┘
                       │  HTTPS
                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         VERCEL EDGE NETWORK                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │                    NEXT.JS APPLICATION                              │     │
│  │                                                                     │     │
│  │  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────┐    │     │
│  │  │  SSR Pages   │  │  Static/ISR  │  │   API Routes          │    │     │
│  │  │  (Dynamic)   │  │  Pages (CDN) │  │   (Server Actions)    │    │     │
│  │  └──────┬───────┘  └──────┬───────┘  └───────────┬───────────┘    │     │
│  │         │                 │                       │                │     │
│  │  ┌──────┴─────────────────┴───────────────────────┴──────────┐    │     │
│  │  │              REACT CLIENT APPLICATION                      │    │     │
│  │  │                                                            │    │     │
│  │  │  ┌────────────┐ ┌────────────┐ ┌────────────────────┐    │    │     │
│  │  │  │ Auth UI    │ │ Dashboard  │ │ Sports Event Mgmt  │    │    │     │
│  │  │  │ Components │ │ Components │ │ Components         │    │    │     │
│  │  │  └────────────┘ └────────────┘ └────────────────────┘    │    │     │
│  │  │  ┌────────────┐ ┌────────────┐ ┌────────────────────┐    │    │     │
│  │  │  │ Booking /  │ │ Profile /  │ │ Notifications /    │    │     │
│  │  │  │ Scheduling │ │ Settings   │ │ Real-time Feed     │    │    │     │
│  │  │  └────────────┘ └────────────┘ └────────────────────┘    │    │     │
│  │  └────────────────────────────────────────────────────────────┘    │     │
│  └─────────────────────────────┬───────────────────────────────────┘     │
└────────────────────────────────┼────────────────────────────────────────┘
                                 │  Supabase Client SDK (HTTPS + WSS)
                                 ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SUPABASE PLATFORM                                    │
│                                                                              │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────────────┐     │
│  │   SUPABASE      │  │   POSTGREST      │  │   SUPABASE             │     │
│  │   AUTH (GoTrue) │  │   (REST API)     │  │   REALTIME             │     │
│  │                 │  │                  │  │   (WebSockets)         │     │
│  │  • Email/Pass   │  │  • Auto-gen CRUD │  │                        │     │
│  │  • OAuth 2.0    │  │  • Filtering     │  │  • Presence            │     │
│  │  • Magic Links  │  │  • Pagination    │  │  • Broadcast           │     │
│  │  • JWT Tokens   │  │  • RLS-enforced  │  │  • DB Changes          │     │
│  └────────┬────────┘  └────────┬─────────┘  └───────────┬────────────┘     │
│           │                    │                         │                   │
│  ┌────────┴────────────────────┴─────────────────────────┴──────────────┐   │
│  │                      POSTGRESQL DATABASE                              │   │
│  │                                                                       │   │
│  │  ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌────────────────────┐    │   │
│  │  │ Users /  │ │ Events /  │ │Bookings/ │ │ Venues / Locations │    │   │
│  │  │ Profiles │ │ Sports    │ │Schedules │ │                    │    │   │
│  │  └──────────┘ └───────────┘ └──────────┘ └────────────────────┘    │   │
│  │  ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌────────────────────┐    │   │
│  │  │ Teams /  │ │ Payments/ │ │Notifica- │ │ Reviews / Ratings  │    │   │
│  │  │ Members  │ │ Invoices  │ │tions     │ │                    │    │   │
│  │  └──────────┘ └───────────┘ └──────────┘ └────────────────────┘    │   │
│  │                                                                       │   │
│  │  ► Row Level Security (RLS) on ALL tables                            │   │
│  │  ► Database Functions / Triggers                                      │   │
│  │  ► pg_cron for scheduled jobs                                         │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐   │
│  │                    SUPABASE EDGE FUNCTIONS (Deno)                      │   │
│  │                                                                       │   │
│  │  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐     │   │
│  │  │ Payment Webhooks │ │ Email/SMS        │ │ Third-party API  │     │   │
│  │  │ (Stripe)         │ │ Notifications    │ │ Orchestration    │     │   │
│  │  └──────────────────┘ └──────────────────┘ └──────────────────┘     │   │
│  │  ┌──────────────────┐ ┌──────────────────┐                          │   │
│  │  │ Scheduled Tasks  │ │ Data Validation  │                          │   │
│  │  │ / Cron Jobs      │ │ / Business Logic │                          │   │
│  │  └──────────────────┘ └──────────────────┘                          │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌──────────────────┐                                                       │
│  │ SUPABASE STORAGE │  (Images, Documents, Avatars)                        │
│  └──────────────────┘                                                       │
└──────────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                      THIRD-PARTY SERVICES                                    │
│                                                                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌─────────────────────┐  │
│  │  Stripe    │  │  Resend /  │  │  Google     │  │  Analytics          │  │
│  │  (Payments)│  │  SendGrid  │  │  Maps API   │  │  (Vercel/PostHog)   │  │
│  └────────────┘  │  (Email)   │  └────────────┘  └─────────────────────┘  │
│                  └────────────┘                                             │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Architecture Pattern: **Modular Monolith with Backend-as-a-Service (BaaS)**

**Pattern Selected:** Modular Monolith (Next.js) + Supabase BaaS

**Justification:**

| Factor | Decision Rationale |
|--------|-------------------|
| **Team Size** | Break Studio operates as a lean team. A single deployable Next.js application reduces operational overhead vs. managing dozens of microservices. |
| **Time-to-Market** | Supabase eliminates the need to build auth, real-time subscriptions, REST APIs, and file storage from scratch — accelerating delivery by 40-60%. |
| **Complexity** | Breaksporti is a sports platform with well-defined, interconnected domains (events, bookings, users, payments). A modular monolith keeps these domains in one codebase with clear module boundaries, avoiding premature distributed system complexity. |
| **Scalability Path** | Next.js on Vercel scales horizontally at the edge automatically. Supabase PostgreSQL scales vertically (with read replicas available). If specific domains require independent scaling in the future, Edge Functions can be extracted into standalone services. |
| **Cost Efficiency** | Both Vercel and Supabase offer generous free tiers, and their pay-as-you-grow models align with a startup/growth-stage product. No Kubernetes clusters, no container orchestration overhead. |
| **Real-time Requirements** | Sports platforms require live updates (scores, availability, bookings). Supabase Realtime provides this out-of-the-box via WebSocket subscriptions on database changes. |

---

## 2. Tech Stack

### 2.1 Frontend: Next.js 14+ (App Router) + React 18+

| Aspect | Detail |
|--------|--------|
| **Framework** | Next.js 14+ with App Router |
| **UI Library** | React 18+ with Server Components |
| **Styling** | Tailwind CSS + shadcn/ui component library |
| **State Management** | React Server Components (server state) + Zustand (client state) + TanStack Query (async state) |
| **Forms** | React Hook Form + Zod validation |
| **Deployment** | Vercel |

**Justification:**
- **Next.js App Router** enables hybrid rendering: static pages (marketing, venue listings) are served from CDN (ISR), while dynamic pages (dashboards, booking flows) use SSR with streaming — optimizing both performance and SEO.
- **React Server Components** reduce client-side JavaScript bundle by rendering data-fetching components on the server, critical for mobile users on slower connections.

---

## Part 2: Database Schema

```sql
-- ============================================================================
-- Break Studio — Complete PostgreSQL / Supabase Schema
-- ============================================================================
-- Since the product domain was left undefined, this schema implements a
-- versatile **Project & Task Management MVP** (think: lightweight Asana/Linear)
-- which is the most common MVP archetype. It covers:
--   • User profiles (extending Supabase auth)
--   • Organizations / Workspaces
--   • Organization membership with roles
--   • Projects
--   • Tasks (with status, priority, assignees, due dates)
--   • Comments on tasks
--   • Labels / Tags (many-to-many with tasks)
--   • File attachments
--   • Notifications
--   • Audit / Activity log
-- ============================================================================


-- ============================================================================
-- 0. EXTENSIONS
-- ============================================================================
CREATE EXTENSION IF NOT EXISTS "pgcrypto";   -- gen_random_uuid()
CREATE EXTENSION IF NOT EXISTS "moddatetime"; -- auto-update updated_at via trigger


-- ============================================================================
-- 1. ENUMS
-- ============================================================================

-- Role a user holds within an organization
CREATE TYPE org_role AS ENUM ('owner', 'admin', 'member', 'viewer');

-- High-level project status
CREATE TYPE project_status AS ENUM ('active', 'archived', 'completed', 'on_hold');

-- Kanban-style task status
CREATE TYPE task_status AS ENUM ('backlog', 'todo', 'in_progress', 'in_review', 'done', 'cancelled');

-- Task priority levels
CREATE TYPE task_priority AS ENUM ('urgent', 'high', 'medium', 'low', 'none');

-- Notification delivery state
CREATE TYPE notification_status AS ENUM ('unread', 'read', 'archived');

-- Audit log action verbs
CREATE TYPE audit_action AS ENUM (
  'create', 'update', 'delete', 'archive', 'restore',
  'assign', 'unassign', 'comment', 'upload', 'login', 'logout'
);

-- Entity types referenced in the audit log
CREATE TYPE auditable_entity AS ENUM (
  'organization', 'project', 'task', 'comment', 'label', 'attachment', 'member', 'profile'
);


-- ============================================================================
-- 2. HELPER: updated_at trigger function
-- ============================================================================
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;


-- ============================================================================
-- 3. USER PROFILES — extends Supabase auth.users
-- ============================================================================
-- Every row in auth.users gets a corresponding public profile.
-- Populated via a database trigger on auth.users INSERT (see bottom).

CREATE TABLE public.profiles (
  id            UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email         TEXT NOT NULL,
  full_name     TEXT,
  display_name  TEXT,
  avatar_url    TEXT,
  bio           TEXT,
  timezone      TEXT DEFAULT 'UTC',
  metadata      JSONB DEFAULT '{}'::jsonb,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

COMMENT ON TABLE public.profiles IS 'Public user profiles extending Supabase auth.users.';

CREATE INDEX idx_profiles_email ON public.profiles (email);

CREATE TRIGGER trg_profiles_updated_at
  BEFORE UPDATE ON public.profiles
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 4. ORGANIZATIONS / WORKSPACES
-- ============================================================================
CREATE TABLE public.organizations (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name          TEXT NOT NULL,
  slug          TEXT NOT NULL UNIQUE,
  logo_url      TEXT,
  description   TEXT,
  owner_id      UUID NOT NULL REFERENCES public.profiles(id) ON DELETE RESTRICT,
  metadata      JSONB DEFAULT '{}'::jsonb,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

COMMENT ON TABLE public.organizations IS 'Top-level workspace / team container.';

CREATE INDEX idx_organizations_slug ON public.organizations (slug);
CREATE INDEX idx_organizations_owner ON public.organizations (owner_id);

CREATE TRIGGER trg_organizations_updated_at
  BEFORE UPDATE ON public.organizations
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 5. ORGANIZATION MEMBERS (join table with role)
-- ============================================================================
CREATE TABLE public.organization_members (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES public.organizations(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  role            org_role NOT NULL DEFAULT 'member',
  invited_by      UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  joined_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (organization_id, user_id)
);

COMMENT ON TABLE public.organization_members IS 'Many-to-many: users ↔ organizations with role.';

CREATE INDEX idx_org_members_org ON public.organization_members (organization_id);
CREATE INDEX idx_org_members_user ON public.organization_members (user_id);

CREATE TRIGGER trg_org_members_updated_at
  BEFORE UPDATE ON public.organization_members
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 6. PROJECTS
-- ============================================================================
CREATE TABLE public.projects (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES public.organizations(id) ON DELETE CASCADE,
  name            TEXT NOT NULL,
  slug            TEXT NOT NULL,
  description     TEXT,
  status          project_status NOT NULL DEFAULT 'active',
  color           TEXT DEFAULT '#6366f1',              -- UI accent colour
  icon            TEXT,
  owner_id        UUID NOT NULL REFERENCES public.profiles(id) ON DELETE RESTRICT,
  start_date      DATE,
  target_date     DATE,
  metadata        JSONB DEFAULT '{}'::jsonb,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (organization_id, slug)
);

COMMENT ON TABLE public.projects IS 'A project lives inside an organization and contains tasks.';

CREATE INDEX idx_projects_org ON public.projects (organization_id);
CREATE INDEX idx_projects_owner ON public.projects (owner_id);
CREATE INDEX idx_projects_status ON public.projects (status);

CREATE TRIGGER trg_projects_updated_at
  BEFORE UPDATE ON public.projects
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 7. LABELS / TAGS
-- ============================================================================
CREATE TABLE public.labels (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES public.organizations(id) ON DELETE CASCADE,
  name            TEXT NOT NULL,
  color           TEXT DEFAULT '#a3a3a3',
  description     TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (organization_id, name)
);

COMMENT ON TABLE public.labels IS 'Reusable labels/tags scoped to an organization.';

CREATE INDEX idx_labels_org ON public.labels (organization_id);

CREATE TRIGGER trg_labels_updated_at
  BEFORE UPDATE ON public.labels
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 8. TASKS
-- ============================================================================
CREATE TABLE public.tasks (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id      UUID NOT NULL REFERENCES public.projects(id) ON DELETE CASCADE,
  parent_task_id  UUID REFERENCES public.tasks(id) ON DELETE SET NULL,  -- subtasks
  title           TEXT NOT NULL,
  description     TEXT,
  status          task_status NOT NULL DEFAULT 'backlog',
  priority        task_priority NOT NULL DEFAULT 'none',
  assignee_id     UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  reporter_id     UUID NOT NULL REFERENCES public.profiles(id) ON DELETE RESTRICT,
  sort_order      INTEGER DEFAULT 0,
  due_date        DATE,
  started_at      TIMESTAMPTZ,
  completed_at    TIMESTAMPTZ,
  estimated_hours NUMERIC(7,2),
  actual_hours    NUMERIC(7,2),
  metadata        JSONB DEFAULT '{}'::jsonb,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

COMMENT ON TABLE public.tasks IS 'Core work item. Belongs to a project; may have a parent (subtask).';

CREATE INDEX idx_tasks_project ON public.tasks (project_id);
CREATE INDEX idx_tasks_assignee ON public.tasks (assignee_id);
CREATE INDEX idx_tasks_status ON public.tasks (status);
CREATE INDEX idx_tasks_priority ON public.tasks (priority);
CREATE INDEX idx_tasks_due_date ON public.tasks (due_date);
CREATE INDEX idx_tasks_parent ON public.tasks (parent_task_id);
CREATE INDEX idx_tasks_sort ON public.tasks (project_id, status, sort_order);

CREATE TRIGGER trg_tasks_updated_at
  BEFORE UPDATE ON public.tasks
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 9. TASK ↔ LABEL join table
-- ============================================================================
CREATE TABLE public.task_labels (
  task_id  UUID NOT NULL REFERENCES public.tasks(id) ON DELETE CASCADE,
  label_id UUID NOT NULL REFERENCES public.labels(id) ON DELETE CASCADE,
  PRIMARY KEY (task_id, label_id)
);

COMMENT ON TABLE public.task_labels IS 'Many-to-many relationship between tasks and labels.';

CREATE INDEX idx_task_labels_label ON public.task_labels (label_id);


-- ============================================================================
-- 10. COMMENTS
-- ============================================================================
CREATE TABLE public.comments (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id     UUID NOT NULL REFERENCES public.tasks(id) ON DELETE CASCADE,
  author_id   UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  parent_id   UUID REFERENCES public.comments(id) ON DELETE CASCADE,   -- threaded replies
  body        TEXT NOT NULL,
  is_edited   BOOLEAN NOT NULL DEFAULT false,
  metadata    JSONB DEFAULT '{}'::jsonb,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

COMMENT ON TABLE public.comments IS 'Discussion threads on tasks. Supports nested replies.';

CREATE INDEX idx_comments_task ON public.comments (task_id);
CREATE INDEX idx_comments_author ON public.comments (author_id);
CREATE INDEX idx_comments_parent ON public.comments (parent_id);

CREATE TRIGGER trg_comments_updated_at
  BEFORE UPDATE ON public.comments
  FOR EACH ROW EXECUTE FUNCTION set_updated_at();


-- ============================================================================
-- 11. ATTACHMENTS
-- ============================================================================
CREATE TABLE public.attachments (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id       UUID REFERENCES public.tasks(id) ON DELETE CASCADE,
  comment_id    UUID REFERENCES public.comments(id) ON DELETE CASCADE,
  uploaded_by   UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  file_name     TEXT NOT NULL,
  file_type     TEXT,                       -- MIME type
  file_size     BIGINT,                     -- bytes
  storage_path  TEXT NOT NULL,              -- Supabase Storage object path
  url           TEXT,                
```

---

## Part 3: API Specification



# API Specification

## Overview

This document defines the complete API specification for the application built on Supabase. Since the project context is undefined, this specification provides a comprehensive, production-ready API architecture that covers common application patterns and can be adapted to specific domain requirements.

**Base URL:** `https://<project-ref>.supabase.co`
**Edge Functions URL:** `https://<project-ref>.supabase.co/functions/v1`
**API Version:** `v1`

---

## Table of Contents

1. [API Versioning Approach](#api-versioning-approach)
2. [Rate Limiting Strategy](#rate-limiting-strategy)
3. [Authentication Flows](#authentication-flows)
4. [User Management](#user-management)
5. [Profile Management](#profile-management)
6. [Resource Management (CRUD)](#resource-management)
7. [File Storage](#file-storage)
8. [Notifications](#notifications)
9. [Search & Discovery](#search--discovery)
10. [Admin & Moderation](#admin--moderation)
11. [Webhook Endpoints](#webhook-endpoints)
12. [Third-Party Integrations](#third-party-integrations)
13. [Realtime Subscriptions](#realtime-subscriptions)

---

## API Versioning Approach

| Strategy | Details |
|---|---|
| **Method** | Header-based versioning with URL prefix fallback |
| **Primary Header** | `X-API-Version: 1` |
| **URL Fallback** | `/functions/v1/`, `/functions/v2/` for Edge Functions |
| **Supabase REST** | Schema-based versioning via `Accept-Profile` header |
| **Deprecation Policy** | Minimum 6-month support window for deprecated versions |
| **Sunset Header** | `Sunset: <date>` returned on deprecated endpoints |

```
# Example request with version header
GET /rest/v1/resources
Headers:
  apikey: <anon-key>
  Authorization: Bearer <jwt>
  Accept-Profile: public
  X-API-Version: 1
```

---

## Rate Limiting Strategy

### Global Limits

| Tier | Requests/Minute | Requests/Hour | Burst (per second) |
|---|---|---|---|
| **Anonymous** | 30 | 500 | 5 |
| **Authenticated (free)** | 120 | 5,000 | 15 |
| **Authenticated (pro)** | 600 | 30,000 | 50 |
| **Admin** | 1,200 | 60,000 | 100 |

### Per-Endpoint Limits

| Endpoint Category | Additional Limit |
|---|---|
| Auth (sign up, sign in) | 10 attempts / 15 min per IP |
| Password reset | 5 requests / hour per email |
| File upload | 20 uploads / hour per user |
| Search | 60 requests / minute per user |
| Webhooks (inbound) | 1,000 / minute per source |
| Edge Functions | 500 invocations / minute per user |

### Implementation

```
# Rate limit headers returned on every response
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 117
X-RateLimit-Reset: 1700000060
Retry-After: 30  # Only on 429 responses
```

Rate limiting is enforced via:
- **Supabase Edge Functions:** Custom middleware using an in-memory store (Deno `Map`) with Redis/Upstash fallback for distributed counting.
- **Database-level:** Postgres `pg_rate_limiter` or custom RLS + function-based throttling.
- **CDN/Proxy level:** Cloudflare rate limiting rules for public-facing endpoints.

---

## Authentication Flows

### 1. Email/Password Sign Up

```
POST /auth/v1/signup
```

| Field | Value |
|---|---|
| **Description** | Register a new user with email and password |
| **Auth Required** | No (anon key required) |
| **Rate Limit** | 10 / 15 min per IP |

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123!",
  "data": {
    "full_name": "Jane Doe",
    "username": "janedoe",
    "accepted_tos": true
  }
}
```

**Response `200 OK`:**
```json
{
  "id": "uuid-v4",
  "aud": "authenticated",
  "role": "authenticated",
  "email": "user@example.com",
  "phone": "",
  "app_metadata": {
    "provider": "email",
    "providers": ["email"]
  },
  "user_metadata": {
    "full_name": "Jane Doe",
    "username": "janedoe",
    "accepted_tos": true
  },
  "identities": [...],
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "confirmation_sent_at": "2024-01-15T10:30:00Z"
}
```

**Error Codes:**

| Code | Body | Condition |
|---|---|---|
| `400` | `{ "error": "invalid_request", "message": "Password must be at least 8 characters" }` | Validation failure |
| `409` | `{ "error": "user_already_exists", "message": "A user with this email already exists" }` | Duplicate email |
| `422` | `{ "error": "invalid_email", "message": "Unable to validate email address" }` | Malformed email |
| `429` | `{ "error": "rate_limit", "message": "Too many signup attempts" }` | Rate limited |

---

### 2. Email/Password Sign In

```
POST /auth/v1/token?grant_type=password
```

| Field | Value |
|---|---|
| **Description** | Authenticate existing user and receive JWT tokens |
| **Auth Required** | No (anon key required) |
| **Rate Limit** | 10 / 15 min per IP |

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123!"
}
```

**Response `200 OK`:**
```json
{
  "access_token": "eyJhbGciOi...",
  "token_type": "bearer",
  "expires_in": 3600,
  "expires_at": 1700003600,
  "refresh_token": "v1.MjAyNC0wMS...",
  "user": {
    "id": "uuid-v4",
    "aud": "authenticated",
    "role": "authenticated",
    "email": "user@example.com",
    "email_confirmed_at": "2024-01-15T10:35:00Z",
    "app_metadata": {
      "provider": "email",
      "providers": ["email"],
      "role": "user"
    },
    "user_metadata": {
      "full_name": "Jane Doe",
      "username": "janedoe"
    },
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:35:00Z"
  }
}
```

**Error Codes:**

| Code | Body | Condition |
|---|---|---|
| `400` | `{ "error": "invalid_grant", "message": "Invalid login credentials" }` | Wrong email/password |
| `401` | `{ "error": "email_not_confirmed", "message": "Email not confirmed" }` | Unverified email |
| `429` | `{ "error": "rate_limit", "message": "Too many login attempts" }` | Rate limited |

---

### 3. Token Refresh

```
POST /auth/v1/token?grant_type=refresh_token
```

| Field | Value |
|---|---|
| **Description** | Exchange refresh token for new access/refresh token pair |
| **Auth Required** | No (anon key required) |
| **Rate Limit** | 30 / min per IP |

**Request Body:**
```json
{
  "refresh_token": "v1.MjAyNC0wMS..."
}
```

**Response `200 OK`:**
```json
{
  "access_token": "eyJhbGciOi...(new)",
  "token_type": "bearer",
  "expires_in": 3600,
  "expires_at": 1700007200,
  "refresh_token": "v1.TmV3UmVm...(new)",
  "user": { "..." }
}
```

**Error Codes:**

| Code | Body | Condition |
|---|---|---|
| `400` | `{ "error": "invalid_grant", "message": "Invalid refresh token" }` | Expired or revoked token |
| `401` | `{ "error": "unauthorized", "message": "Token has been revoked" }` | Manually revoked |

---

### 4. Password Reset Request

```
POST /auth/v1/recover
```

| Field | Value |
|---|---|
| **Description** | Send password reset email with magic link/OTP |
| **Auth Required** | No (anon key required) |
| **Rate Limit** | 5 / hour per email |

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response `200 OK`:**
```json
{
  "message": "If an account exists, a password reset email has been sent"
}
```

> **Note:** Always returns 200 to prevent email enumeration.

**Error Codes:**

| Code | Body | Condition |
|---|---|---|
| `429` | `{ "error": "rate_limit", "message": "Too many reset requests" }` | Rate limited |

---

### 5. Password Update (after reset)

```
PUT /auth/v1/user
```

| Field | Value |
|---|---|
| **Description** | Update user password using recovery token |
| **Auth Required** | Yes (recovery JWT from email link) |
| **Rate Limit** | 10 / hour per user |

**Request Headers:**
```
Authorization: Bearer <recovery-access-token>
```

**Request Body:**
```json
{
  "password": "NewSecureP@ss456!"
}
```

**Response `200 OK`:**
```json
{
  "id": "uuid-v4",
  "email": "user@example.com",
  "updated_at": "2024-01-15T12:00:00Z",
  "..."
}
```

**Error Codes:**

| Code | Body | Condition |
|---|---|---|
| `400` | `{ "error": "same_password", "message": "New password must be different" }` | Same as old password |
| `401` | `{ "error": "unauthorized", "message": "Invalid or expired token" }` | Bad recovery token |
| `422` | `{ "error": "weak_password", "message": "Password does not meet requirements" }` | Weak password |

---

### 6. OAuth Sign In (Google, GitHub, Apple, etc.)

```
GET /auth/v1/authorize?provider={provider}
```

| Field | Value |
|---|---|
| **Description** | Initiate OAuth flow with third-party provider |
| **Auth Required** | No (anon key required) |
| **Supported Providers** | `google`, `github`, `apple`, `discord`, `twitter` |

**Request Params:**

| Param | Type | Required | Description |
|---|---|---|---|
| `provider` | string | Yes | OAuth provider name |
| `redirect_to` | string | No | URL to redirect after auth |
| `scopes` | string | No | Additional OAuth scopes (space-separated) |

**Response