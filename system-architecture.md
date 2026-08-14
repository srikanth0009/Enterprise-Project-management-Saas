# Enterprise Project Management SaaS — System Architecture

## 1. Purpose

This document explains how the frontend, backend, database, cache, file storage, WebSockets, and deployment pieces work together as one system.

## 2. High-Level Architecture

```text
Users
  ↓
React + TypeScript
  ↓ HTTPS / REST / WebSocket
Node.js + Express
  ├── Authentication / Authorization
  ├── Business Services
  ├── Validation
  └── API Controllers
        │
        ├──────────────→ MongoDB
        │
        ├──────────────→ Redis
        │
        └──────────────→ Cloudinary

WebSocket events ← Node.js → Connected users
```

## 3. Technology Responsibilities

| Component | Responsibility |
|---|---|
| React | User interface |
| TypeScript | Type safety |
| Tailwind + ShadCN | UI styling/components |
| React Query | Server state, caching, mutations |
| Zustand | Client/UI state |
| Node.js + Express | REST API and backend |
| MongoDB | Persistent application data |
| Redis | Cache, rate limiting, coordination |
| WebSockets | Real-time events |
| Cloudinary | File/image storage |
| Docker | Reproducible environments |

## 4. Frontend Architecture

```text
src/
├── app/
│   ├── router/
│   ├── providers/
│   └── layouts/
├── components/
│   ├── ui/
│   └── shared/
├── features/
│   ├── auth/
│   ├── organizations/
│   ├── teams/
│   ├── workspaces/
│   ├── projects/
│   ├── issues/
│   ├── comments/
│   ├── notifications/
│   └── dashboard/
├── hooks/
├── lib/
│   ├── api/
│   └── websocket/
├── stores/
└── types/
```

Feature-based organization keeps related UI, hooks, API calls, and types together.

### React Query vs Zustand

```text
React Query → server state
               Projects
               Issues
               Comments
               Notifications

Zustand     → client state
               Sidebar
               Modal state
               UI preferences
               Temporary selections
```

Do not put all server data into Zustand.

## 5. Backend Architecture

```text
HTTP Request
     ↓
Router
     ↓
Middleware
     ↓
Controller
     ↓
Service
     ↓
Repository / Data Access
     ↓
MongoDB
```

Middleware handles:

- Authentication
- Authorization
- Validation
- Rate limiting
- Logging
- Error handling

Controllers should remain thin. Business rules belong in services.

## 6. MongoDB

MongoDB is the primary source of truth.

It stores:

```text
Users
Organizations
Memberships
Teams
Workspaces
Projects
Issues
Comments
Labels
Attachment metadata
Notifications
Activities
Invitations
```

Most relationships are represented using IDs/references as defined in `database-design.md`.

## 7. Redis

Redis is not the primary database.

It can support:

```text
Caching
Rate limiting
Temporary data
Session-related state where appropriate
WebSocket coordination
Pub/Sub
```

### Cache-aside strategy

```text
Read
 ↓
Redis?
 ├─ HIT  → return cached data
 └─ MISS → MongoDB → store in Redis → return

Write
 ↓
MongoDB
 ↓
Invalidate relevant Redis key
```

For the MVP, cache only data where it provides clear value, such as project details and dashboard statistics.

## 8. Cloudinary

Cloudinary stores uploaded files.

```text
React
  ↓ multipart/form-data
Backend
  ↓ validate permission/type/size
Cloudinary
  ↓ file URL
MongoDB
  ↓ metadata
React
```

MongoDB stores metadata such as:

```text
fileName
fileUrl
fileType
fileSize
uploadedBy
issueId
createdAt
```

## 9. Authentication Flow

```text
Login
 ↓
POST /api/v1/auth/login
 ↓
Express
 ↓
Validate credentials
 ↓
Verify password hash
 ↓
Create access token
 ↓
Create refresh session/token
 ↓
Frontend
```

Protected requests use:

```http
Authorization: Bearer <accessToken>
```

When the access token expires:

```text
Frontend
 ↓
POST /api/v1/auth/refresh
 ↓
Refresh token
 ↓
New access token
```

The refresh token should preferably use a secure HTTP-only cookie.

## 10. Authorization Flow

Authentication answers:

> Who are you?

Authorization answers:

> Are you allowed to do this?

Example:

```text
Authenticated user
      ↓
Organization membership
      ↓
Project membership
      ↓
Required role
      ↓
Permission granted
```

The backend must enforce permissions. Hiding a button in React is not a security mechanism.

## 11. Multi-Tenant Architecture

Each organization is a tenant.

```text
Organization A
 ├── Members
 ├── Workspaces
 ├── Projects
 └── Issues

Organization B
 ├── Members
 ├── Workspaces
 ├── Projects
 └── Issues
```

A user must never be able to access another organization's data.

For every protected resource, the backend verifies the relationship between:

```text
User → Organization → Resource
```

## 12. WebSockets

REST handles normal CRUD operations:

```text
Create
Read
Update
Delete
```

WebSockets handle live events:

```text
issue.created
issue.updated
issue.deleted
comment.created
notification.created
member.added
member.removed
```

Example:

```text
User A
 ↓
PATCH /issues/123
 ↓
MongoDB update
 ↓
Publish issue.updated
 ↓
WebSocket
 ↓
User B
```

WebSockets are an event/synchronization mechanism, not a database.

## 13. WebSockets + React Query

```text
WebSocket event
      ↓
React Query
      ↓
Update/invalidate relevant cache
      ↓
UI updates
```

For example:

```javascript
queryClient.invalidateQueries({
  queryKey: ["issues", projectId]
});
```

This keeps React Query responsible for server state while WebSockets provide real-time synchronization.

## 14. Redis Pub/Sub for Scaling

With one backend instance, WebSockets can work directly.

If we later have:

```text
Server 1
Server 2
Server 3
```

Redis Pub/Sub can distribute events:

```text
Server 1
   ↓
Redis Pub/Sub
   ↓
Server 2 + Server 3
```

This allows connected users on different backend instances to receive the same events.

## 15. Example: Create Issue

```text
User clicks Create Issue
        ↓
React form
        ↓
Client validation
        ↓
React Query mutation
        ↓
POST /projects/:id/issues
        ↓
Authentication
        ↓
Authorization
        ↓
Backend validation
        ↓
Issue Service
        ↓
MongoDB
        ↓
Activity created
        ↓
Notification created if needed
        ↓
WebSocket event published
        ↓
API response
        ↓
React Query cache update
        ↓
Kanban UI updates
```

## 16. Example: Kanban Drag and Drop

```text
User moves ECOM-42
TODO → IN_PROGRESS
        ↓
PATCH /api/v1/issues/:id
        ↓
Issue Service
        ↓
MongoDB
        ↓
issue.updated
        ↓
WebSocket
        ↓
Other connected users
```

If explicit ordering is implemented, the request can also contain a position value.

## 17. Example: Dashboard

```text
React Dashboard
      ↓
GET /api/v1/projects/:id/dashboard
      ↓
Authentication
      ↓
Authorization
      ↓
Dashboard Service
      ↓
Redis?
 ├── HIT  → return cached statistics
 └── MISS → MongoDB aggregation
              ↓
           Redis cache
              ↓
           Response
```

## 18. Docker Architecture

For local development:

```text
Docker Compose
├── frontend
├── backend
├── mongodb
└── redis
```

Cloudinary remains an external service.

Docker provides a reproducible development environment so the project can be started consistently.

## 19. Production Architecture

```text
                 Internet
                    ↓
          Reverse Proxy / Load Balancer
                    ↓
          ┌─────────┴─────────┐
          ↓                   ↓
     Backend 1           Backend 2
          └─────────┬─────────┘
                    ↓
             ┌──────┴──────┐
             ↓             ↓
          MongoDB         Redis

             + Cloudinary
```

The backend should remain sufficiently stateless to support multiple instances.

## 20. Failure Handling

### Redis unavailable

The application should continue using MongoDB where possible; caching/coordination features may temporarily degrade.

### WebSocket unavailable

REST continues working. Users may temporarily lose live updates.

### Cloudinary unavailable

File uploads fail gracefully while normal issue/project operations continue.

### MongoDB unavailable

Core application operations cannot continue and should return controlled errors.

## 21. Security Architecture

Important controls:

```text
HTTPS
Password hashing
JWT verification
HTTP-only refresh cookies
RBAC
Tenant isolation
Backend validation
Rate limiting
Request-size limits
Secure CORS
Security headers
File validation
```

Never trust:

```text
Frontend user ID
Frontend organization ID
Frontend role
Frontend permissions
```

The server must verify these values.

## 22. CORS

Only known frontend origins should be allowed.

Development:

```text
http://localhost:5173
```

Production:

```text
https://your-domain.com
```

Avoid permissive authenticated production CORS configurations.

## 23. Observability

The application should eventually provide:

```text
Request logs
Error logs
Performance metrics
Database monitoring
Health checks
```

Basic health endpoint:

```http
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

## 24. Data Ownership

```text
MongoDB
→ persistent application data

Redis
→ cache / temporary / coordination data

Cloudinary
→ uploaded files

WebSocket
→ real-time event delivery

React Query
→ frontend server-state cache

Zustand
→ frontend client state
```

This separation prevents responsibilities from becoming mixed.

## 25. Architecture Decisions

### Why MongoDB?

The domain contains many related but flexible documents, and MongoDB works well with document-oriented data and iterative product development.

### Why Redis?

It provides very fast access for caching, rate limiting, and future distributed event coordination.

### Why WebSockets?

Project-management users benefit from seeing issue, comment, and notification changes without manually refreshing.

### Why Cloudinary?

Dedicated object/media storage is better suited to uploaded files than storing large binary files in MongoDB.

### Why Docker?

It makes local infrastructure reproducible and prepares the project for consistent deployment.

### Why layered backend architecture?

It keeps HTTP concerns, business rules, and persistence concerns separate and testable.

## 26. Final System Flow

```text
                           USER
                            │
                            ▼
                    ┌───────────────┐
                    │ React Client  │
                    │ TypeScript    │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
             REST API             WebSocket
                 │                     │
                 └──────────┬──────────┘
                            ▼
                    ┌───────────────┐
                    │ Node/Express  │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           MongoDB        Redis       Cloudinary
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                    Real-time Events
                            │
                            ▼
                    Connected Users
```

## 27. Design Progress

```text
1. Product Requirements       ✅
2. User Flows                 ✅
3. Domain Model               ✅
4. Database Design            ✅
5. API Design                 ✅
6. System Architecture        ✅

7. Frontend Architecture      ← NEXT
8. UI Design
9. Implementation
```

The next document is `frontend-architecture.md`. It will define the React application in detail: routes, pages, layouts, feature modules, components, React Query, Zustand, forms, authentication state, API client, WebSocket client, protected routes, loading/error states, and the Kanban architecture.
